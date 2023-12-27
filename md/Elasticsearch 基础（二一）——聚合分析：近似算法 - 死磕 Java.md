> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1157126200)

> 本章，我们来介绍下聚合分析底层用到的算法：  近似算法 & nbsp;。  什么是近似算法？  一般来讲，有些聚合分析的 metric 操作，是很容易在多个 sh

本章，我们来介绍下聚合分析底层用到的算法： **近似算法** 。

_**什么是近似算法？**_

一般来讲，有些聚合分析的 metric 操作，是很容易在多个 shard 中并行执行的，比如`max`、`min`、`avg`这种，coordinate node 拿到各个 shard 的返回结果后，只需要经过简单计算就能得出最终结果：

1.  coordinate node 把请求广播到所有 shard；
2.  每个分片计算本地最大的字段值，返回给 coordinate node；
3.  coordinate node 选出所有 shard 返回的最大值，这就是最终的最大值。

上面这类算法可以随着机器数的线性增长而横向扩展，无须任何协调操作（机器之间不需要讨论中间结果），而且内存消耗很小（一个整型就能代表最大值）。

但是还有些算法，是很难并行执行的，比如说`count(distinct)`，并不是说在每个 shard 上直接过滤出 distinct value 就可以了，因为 coordinate node 需要拿到各个 shard 返回的结果，在内存中进行筛选操作，如果数据量非常大，这个过程非常耗时。

所以，Elasticsearch 为了提升性能，采用了近似算法，它们会提供准确但不是 100% 精确的结果， 以牺牲一点小小的估算错误为代价，这些算法可以为我们换来高速的执行效率和极小的内存消耗。

一、近似算法
------

### 1.1 基本思想

近似算法的基本思想就是在 _**大数据**_ 、 _**精确性**_ 和 _**实时性**_ 这三者之间做出权衡，一般只能选择其中的 2 个，有点类似于 CAP。 因为对于很多应用，能够实时返回高度准确的结果要比 100% 精确结果重要得多：

**精确 + 实时**

数据可以存入单台机器的内存之中，我们可以随心所欲，使用任何想用的算法。结果会 100% 精确，响应会相对快速。

**大数据 + 精确**

传统的 Hadoop，可以处理 PB 级的数据并且为我们提供精确的答案，但它可能需要几周的时间才能为我们提供这个答案。

**大数据 + 实时**

近似算法为我们实时提供准确但不精确的结果。

![](http://image.skjava.com/article/series/elasticsearch/202308082147457861.png)

Elasticsearch 目前支持两种近似算法（ **cardinality** 和 **percentiles** ）。 它们会提供准确但不是 100% 精确的结果，以牺牲一点小小的估算错误为代价，这些算法可以为我们换来高速的执行效率和极小的内存消耗。

二、Cardinality 算法
----------------

### 2.1 基本使用

Cardinality 算法用于统计某个字段的不同值的个数，也就是去重统计。比如，我们需要统计每个月销售的不同品牌数量，那么可以像下面这样构造查询：

```
    GET /tvs/_search
    {
      "size" : 0,
      "aggs" : {
          "months" : {
            "date_histogram": {
              "field": "sold_date",
              "interval": "month"
            },
            "aggs": {
              "distinct_brand" : {
                  "cardinality" : {
                    "field" : "brand"
                  }
              }
            }
          }
      }
    }


```

### 2.2 算法优化

Cardinality 算法的统计结果并不一定精确，但是速度非常快，我们还可以通过调整参数来进一步优化。

#### precision_threshold

`precision_threshold`可以控制 Cardinality 算法的精确度和内存消耗，它接受 0–40000 之间的数字，更大的值还是会被当作 40000 来处理。

比如，`precision_threshold`设置为 100，那么 Elasticsearch 会确保当字段唯一值在 100 以内时，会得到非常准确的结果，这个准确率几乎 100%。但是，如果字段唯一值的数目高于`precision_threshold`，ES 就会开始节省内存而牺牲准确度。

> 根据 Elasticsearch 的官方统计，`precision_threshold`设置为 100 时，对于 100 万个不同的字段值，统计结果的误差可以维持在 5% 以内。

```
    GET /tvs/_search
    {
        "size" : 0,
        "aggs" : {
            "distinct_brand" : {
                "cardinality" : {
                  "field" : "brand",
                  "precision_threshold" : 100 
                }
            }
        }
    }


```

#### HyperLogLog

Cardinality 算法的底层是基于 HyperLogLog++ 算法（简称 HLL）实现的，HLL 算法会对所有 unique value 取 hash 值，通过 hash 值近似求 distinct count。

默认情况下，如果我们的请求里包含 cardinality 统计，ELasticsearch 会实时对所有的 field value 取 hash 值。所以，一种优化思路就是在建立索引时，就将所有字段值的 hash 建立好。

比如，我们对 brand 字段再内建一个字段——名为 “hash”，它的类型是`murmur3`，是一种计算 hash 值的算法：

```
    PUT /tvs/
    {
      "mappings": {
        "sales": {
          "properties": {
            "brand": {
              "type": "text",
              "fields": {
                "hash": {
                  "type": "murmur3" 
                }
              }
            }
          }
        }
      }
    }


```

当我们需要统计字段的 distinct value 时，直接对内置字段进行 cardinality 统计即可：

```
    GET /tvs/_search
    {
        "size" : 0,
        "aggs" : {
            "distinct_brand" : {
                "cardinality" : {
                  "field" : "brand.hash",
                  "precision_threshold" : 100 
                }
            }
        }
    }


```

三、Percentiles 算法
----------------

Percentiles 算法可以按照百分比来统计某个字段的聚合信息。

### 3.1 基本使用

比如，我们有一个网站，记录了每次请求的访问耗时，需要统计 tp50、tp90、tp99，那么用 percentiles 实现就非常方便。

> tp50：50% 的请求的最长耗时  
> tp90：90% 的请求的最长耗时  
> tp99：99% 的请求的最长耗时

我们创建一个示例来进行演示：

```
    
    PUT /website
    {
        "mappings": {
            "properties": {
                "latency": {
                    "type": "long"
                },
                "province": {
                    "type": "keyword"
                },
                "timestamp": {
                    "type": "date"
                }
            }
        }
    }
    
    
    POST /website/logs/_bulk
    { "index": {}}
    { "latency" : 105, "province" : "江苏", "timestamp" : "2016-10-28" }
    { "index": {}}
    { "latency" : 83, "province" : "江苏", "timestamp" : "2016-10-29" }
    { "index": {}}
    { "latency" : 92, "province" : "江苏", "timestamp" : "2016-10-29" }
    { "index": {}}
    { "latency" : 112, "province" : "江苏", "timestamp" : "2016-10-28" }
    { "index": {}}
    { "latency" : 68, "province" : "江苏", "timestamp" : "2016-10-28" }
    { "index": {}}
    { "latency" : 76, "province" : "江苏", "timestamp" : "2016-10-29" }
    { "index": {}}
    { "latency" : 101, "province" : "新疆", "timestamp" : "2016-10-28" }
    { "index": {}}
    { "latency" : 275, "province" : "新疆", "timestamp" : "2016-10-29" }
    { "index": {}}
    { "latency" : 166, "province" : "新疆", "timestamp" : "2016-10-29" }
    { "index": {}}
    { "latency" : 654, "province" : "新疆", "timestamp" : "2016-10-28" }
    { "index": {}}
    { "latency" : 389, "province" : "新疆", "timestamp" : "2016-10-28" }
    { "index": {}}
    { "latency" : 302, "province" : "新疆", "timestamp" : "2016-10-29" }


```

下面的请求，按照 latency 字段的记录数百分比进行分组，然后统计组内的平均延时信息：

```
    GET /website/_search 
    {
      "size": 0,
      "aggs": {
        "latency_percentiles": {
          "percentiles": {
            "field": "latency",
            "percents": [
              50,
              95,
              99
            ]
          }
        },
        "latency_avg": {
          "avg": {
            "field": "latency"
          }
        }
      }
    }


```

响应如下：

```
    {
      "took": 31,
      "timed_out": false,
      "_shards": {
        "total": 5,
        "successful": 5,
        "failed": 0
      },
      "hits": {
        "total": 12,
        "max_score": 0,
        "hits": []
      },
      "aggregations": {
        "latency_avg": {
          "value": 201.91666666666666
        },
        "latency_percentiles": {
          "values": {
            "50.0": 108.5,
            "95.0": 508.24999999999983,
            "99.0": 624.8500000000001
          }
        }
      }
    }


```

### 3.2 percentile_ranks

percentile_ranks 可以按照字段值的区间进行分组，然后统计出每个区间的占比。

比如，我们需要统计：对于每个省份，有多少请求（百分比）的延时分别在 200ms 以内、1000ms 以内？就可以像下面这样构造请求：

```
    GET /website/_search 
    {
      "size": 0,
      "aggs": {
        "group_by_province": {
          "terms": {
            "field": "province"
          },
          "aggs": {
            "latency_percentile_ranks": {
              "percentile_ranks": {
                "field": "latency",
                "values": [
                  200,
                  1000
                ]
              }
            }
          }
        }
      }
    }


```

响应如下：

```
    {
      "took": 38,
      "timed_out": false,
      "_shards": {
        "total": 5,
        "successful": 5,
        "failed": 0
      },
      "hits": {
        "total": 12,
        "max_score": 0,
        "hits": []
      },
      "aggregations": {
        "group_by_province": {
          "doc_count_error_upper_bound": 0,
          "sum_other_doc_count": 0,
          "buckets": [
            {
              "key": "新疆",
              "doc_count": 6,
              "latency_percentile_ranks": {
                "values": {
                  "200.0": 29.40613026819923,
                  "1000.0": 100
                }
              }
            },
            {
              "key": "江苏",
              "doc_count": 6,
              "latency_percentile_ranks": {
                "values": {
                  "200.0": 100,
                  "1000.0": 100
                }
              }
            }
          ]
        }
      }
    }


```

### 3.3 算法优化

percentile 底层采用了 TDigest 算法，该算法会使用很多节点来执行百分比的计算，但是存在误差，参与计算的节点越多就越精准。

percentile 有一个参数`compression`可以用来控制节点数量，默认值是 100，所以如果想要 percentile 算法更精准，compression 值可以设置得越大。

> 注意：compression 值越大越消耗内存，一般 compression=100 时，内存占用大约为：100 x 20 x 32 = 64KB。

四、总结
----

本章，我介绍了 Elasticsearch 中两种常用的近似算法：cardinality 和 percentile。它们的思想都是在 _**大数据**_ 、 _**精确性**_ 和 _**实时性**_ 这三者之间做出权衡。Elasticsearch 为了提升性能，采用近似算法，它们会提供准确但不是 100% 精确的结果， 以牺牲一点小小的估算错误为代价，这些算法可以为我们换来高速的执行效率和极小的内存消耗。
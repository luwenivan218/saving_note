> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1154354581)

> Elasticsearch 的聚合分析是有范围的，这个作用范围叫做 AggregationScope。默认情况下，这个 scope 就是检索出的所有记录。如果我们希望对部分记录进行聚合

Elasticsearch 的聚合分析是有范围的，这个作用范围叫做 Aggregation Scope。默认情况下，这个 scope 就是检索出的所有记录。如果我们希望对部分记录进行聚合分析，就要结合 query 来使用。

一、Aggregation Scope
-------------------

### 1.1 结合普通 query

我们可以将聚合分析与全文检索结合起来使用，Elasticsearch 中的所有聚合都会在一个 scope 下执行，结合普通搜索请求后，这个 scope 就是检索出的结果。

比如，我们需要统计指定品牌下每个颜色的销量：

```
    GET /tvs/_search 
    {
      "size": 0,
      "query": {
        "term": {
          "brand": {
            "value": "小米"
          }
        }
      },
      "aggs": {
        "group_by_color": {
          "terms": {
            "field": "color"
          }
        }
      }
    }


```

结果如下：

```
    {
      "took" : 34,
      "timed_out" : false,
      "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : {
          "value" : 2,
          "relation" : "eq"
        },
        "max_score" : null,
        "hits" : [ ]
      },
      "aggregations" : {
        "group_by_color" : {
          "doc_count_error_upper_bound" : 0,
          "sum_other_doc_count" : 0,
          "buckets" : [
            {
              "key" : "绿色",
              "doc_count" : 1
            },
            {
              "key" : "蓝色",
              "doc_count" : 1
            }
          ]
        }
      }
    }


```

### 1.2 结合过滤

除了和 query 结合外，聚合分析还可以跟 filter 结合使用。比如下面的请求是统计价格大于 1200 的所有电视机的平均价格。

```
    GET /tvs/_search 
    {
      "size": 0,
      "query": {
        "constant_score": {
          "filter": {
            "range": {
              "price": {
                "gte": 1200
              }
            }
          }
        }
      },
      "aggs": {
        "avg_price": {
          "avg": {
            "field": "price"
          }
        }
      }
    }


```

有时候，我们可能不想进行全局的 filter，而是针对某个 bucket 进行精细化的 filter，那么就可以使用`aggs.filter`。比如下面的请求，我们统计长虹电视最近 1 个月、最近 3 个月、最近 6 个月的平均值：

```
    GET /tvs/_search 
    {
      "size": 0,
      "query": {
        "term": {
          "brand": {
            "value": "长虹"
          }
        }
      },
      "aggs": {
        "recent_1m": {
          "filter": {
            "range": {
              "sold_date": {
                "gte": "now-1m"
              }
            }
          },
          "aggs": {
            "recent_1m_avg_price": {
              "avg": {
                "field": "price"
              }
            }
          }
        },
        "recent_3m": {
          "filter": {
            "range": {
              "sold_date": {
                "gte": "now-3m"
              }
            }
          },
          "aggs": {
            "recent_3m_avg_price": {
              "avg": {
                "field": "price"
              }
            }
          }
        },
        "recent_6m": {
          "filter": {
            "range": {
              "sold_date": {
                "gte": "now-6m"
              }
            }
          },
          "aggs": {
            "recent_6m_avg_price": {
              "avg": {
                "field": "price"
              }
            }
          }
        }
      }
    }


```

### 1.3 global bucket

有时候，我们可能有下面这样的聚合分析需求，对于一次聚合分析请求，我们希望给出两个结果：

1.  指定 scope 范围内的聚合结果；
2.  不限定范围的聚合结果。

对于这种需求，就要用到 global bucket 了。比如，我们希望对比长虹牌电视机的平均销售额和所有品牌电视机的平均销售额：

```
    GET /tvs/_search 
    {
      "size": 0, 
      "query": {
        "term": {
          "brand": {
            "value": "长虹"
          }
        }
      },
      "aggs": {
        "single_brand_avg_price": {
          "avg": {
            "field": "price"
          }
        },
        "all": {
          "global": {},
          "aggs": {
            "all_brand_avg_price": {
              "avg": {
                "field": "price"
              }
            }
          }
        }
      }
    }


```

上述请求中的 “query” 用于限定 scope，对该 scope 范围内的 doc 执行聚合分析，而内部的 “global” 关键字会将聚合分析的范围指定为所有 doc。

请求结果如下：

```
    {
      "took" : 35,
      "timed_out" : false,
      "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : {
          "value" : 3,
          "relation" : "eq"
        },
        "max_score" : null,
        "hits" : [ ]
      },
      "aggregations" : {
        "all" : {
          "doc_count" : 8,
          "all_brand_avg_price" : {
            "value" : 2650.0
          }
        },
        "single_brand_avg_price" : {
          "value" : 1666.6666666666667
        }
      }
    }


```

二、总结
----

本章，我们介绍了聚合分析中的 Aggregation Scope，其实就是限定进行聚合分析的 doc 范围。此外，聚合分析也可以和 query、filter 结合使用。
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1130345981)

> 从本章开始，我将介绍 Elasticsearch 强大的聚合分析功能，聚合分析和 SQL 中的 GROUPBY 非常类似。掌握聚合分析，最重要的还是实战练习，所以从本章开始，我将会通过一个

从本章开始，我将介绍 Elasticsearch 强大的聚合分析功能，聚合分析和 SQL 中的`GROUP BY`非常类似。掌握聚合分析，最重要的还是实战练习，所以从本章开始，我将会通过一个案例，来介绍聚合分析的各种使用方式。

我们先来了解下聚合分析中的两个核心概念：bucket 和 metric。

一、核心概念
------

### 1.1 bucket

bucket 代表一个数据分组。举个例子，我们有以下数据：

<table><thead><tr><th>CITY</th><th>NAME</th></tr></thead><tbody><tr><td>北京</td><td>小李</td></tr><tr><td>北京</td><td>小王</td></tr><tr><td>上海</td><td>小张</td></tr><tr><td>上海</td><td>小丽</td></tr><tr><td>上海</td><td>小陈</td></tr></tbody></table>

如果我们基于 CITY 划分 buckets，就划分出来两个 bucket：

*   北京 bucket：包含了 2 个人，小李、小王
*   上海 bucket：包含了 3 个人，小张、小丽、小陈

### 1.2 metric

当我们对数据进行 bucket 分组之后，就可以对每个 bucket 进行统计分析了，比如说计算一个 bucket 内所有数据的数量，或者计算一个 bucket 内所有数据的平均值、最大值、最小值。

metric，就是对一个 bucket 执行的某种聚合分析操作，比如说求平均值、求最大值 / 最大值、求数量、请和等。

二、案例背景
------

我们的案例以家电卖场中的电视销售为背景，这个案例将会贯彻整个聚合分析系列的所有章节。我们会对各种品牌、颜色的电视机销量和销售额进行聚合分析。

首先构建一个索引`tvs`：

```
    PUT /tvs
    {
        "mappings": {
          "properties": {
                    "price": {
                        "type": "long"
                    },
                    "color": {
                        "type": "keyword"
                    },
                    "brand": {
                        "type": "keyword"
                    },
                    "sold_date": {
                        "type": "date"
                    }
                }
        }
    }


```

批量插入一些数据：

```
    POST /tvs/_bulk
    { "index": {}}
    { "price" : 1000, "color" : "红色", "brand" : "长虹", "sold_date" : "2016-10-28" }
    { "index": {}}
    { "price" : 2000, "color" : "红色", "brand" : "长虹", "sold_date" : "2016-11-05" }
    { "index": {}}
    { "price" : 3000, "color" : "绿色", "brand" : "小米", "sold_date" : "2016-05-18" }
    { "index": {}}
    { "price" : 1500, "color" : "蓝色", "brand" : "TCL", "sold_date" : "2016-07-02" }
    { "index": {}}
    { "price" : 1200, "color" : "绿色", "brand" : "TCL", "sold_date" : "2016-08-19" }
    { "index": {}}
    { "price" : 2000, "color" : "红色", "brand" : "长虹", "sold_date" : "2016-11-05" }
    { "index": {}}
    { "price" : 8000, "color" : "红色", "brand" : "三星", "sold_date" : "2017-01-01" }
    { "index": {}}
    { "price" : 2500, "color" : "蓝色", "brand" : "小米", "sold_date" : "2017-02-12" }


```

三、实战
----

### 3.1 metric：按数量分组

我们需要统计哪种颜色的电视机销量最高：

```
    GET /tvs/_search
    {
        "size" : 0,
        "aggs" : { 
            "popular_colors" : { 
                "terms" : { 
                  "field" : "color"
                }
            }
        }
    }


```

查询请求的各个关键字含义如下：

*   **size：** 只获取聚合结果，而不需要返回执行聚合的那些原始数据；
*   **aggs：** 固定语法，表示要对一份数据执行分组聚合操作；
*   **popular_colors：** 每个 aggs 的名字，自定义；
*   **terms：** 根据字段值进行分组；
*   **field：** 进行分组的字段。

上述请求的返回结果如下：

```
    {
      "took" : 7,
      "timed_out" : false,
      "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : {
          "value" : 8,
          "relation" : "eq"
        },
        "max_score" : null,
        "hits" : [ ]
      },
      "aggregations" : {
        "popular_colors" : {
          "doc_count_error_upper_bound" : 0,
          "sum_other_doc_count" : 0,
          "buckets" : [
            {
              "key" : "红色",
              "doc_count" : 4
            },
            {
              "key" : "绿色",
              "doc_count" : 2
            },
            {
              "key" : "蓝色",
              "doc_count" : 2
            }
          ]
        }
      }
    }


```

我们来看下返回结果中的一些核心关键字的含义：

*   **hits.hits：** 我们在请求中指定了 size=0，所以 hits.hits 就是空的，否则会把执行聚合的那些原始数据返回；
*   **aggregations：** 聚合结果；
*   **popular_color：** 自定义的聚合名称；
*   **buckets：** 根据我们指定的 field 划分出的 buckets；
*   **key：** field 的值
*   **doc_count：** 这个 bucket 分组内的 doc 条数

> 按数量分组其实并不算是一个 metric 操作，它是 Elasticsearch 对聚合分析的一种默认操作，利用 term 实现。

### 3.2 metric：统计平均值

这一节，我们来学习第一个 metric 操作——统计平均值。假设我们需要统计每种颜色电视机的平均价格：

```
    GET /tvs/_search
    {
       "size" : 0,
       "aggs": {
          "colors": {
             "terms": {
                "field": "color"
             },
             "aggs": { 
                "avg_price": { 
                   "avg": {
                      "field": "price" 
                   }
                }
             }
          }
       }
    }


```

上述请求，我们嵌套了一个”aggs“，这个”aggs“和 “terms” 平级，会对每个 bucket 执行一次 metric 操作：

```
    "aggs": { 
        "avg_price": { 
            "avg": {
              "field": "price" 
            }
        }
    }


```

返回结果如下：

```
    {
      "took" : 12,
      "timed_out" : false,
      "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : {
          "value" : 8,
          "relation" : "eq"
        },
        "max_score" : null,
        "hits" : [ ]
      },
      "aggregations" : {
        "colors" : {
          "doc_count_error_upper_bound" : 0,
          "sum_other_doc_count" : 0,
          "buckets" : [
            {
              "key" : "红色",
              "doc_count" : 4,
              "avg_price" : {
                "value" : 3250.0
              }
            },
            {
              "key" : "绿色",
              "doc_count" : 2,
              "avg_price" : {
                "value" : 2100.0
              }
            },
            {
              "key" : "蓝色",
              "doc_count" : 2,
              "avg_price" : {
                "value" : 2000.0
              }
            }
          ]
        }
      }
    }


```

可以看到，每个 bucket 内部多了”avg_price“，其 value 就是我们的 metric 计算的结果——每个 bucket 中的所有 doc 的 price 字段值的平均值。

### 3.3 下钻分析

所谓下钻分析，就是对每个 bucket 再进行分组，然后对每个最小粒度的分组再执行聚合分析操作。比如，我们已经按照颜色对电视机进行分组了，但是还想统计下每种颜色下的各个品牌的电视机平均价格：

```
    GET /tvs/_search 
    {
      "size": 0,
      "aggs": {
        "group_by_color": {
          "terms": {
            "field": "color"
          },
          "aggs": {
            "color_avg_price": {
              "avg": {
                "field": "price"
              }
            },
            "group_by_brand": {
              "terms": {
                "field": "brand"
              },
              "aggs": {
                "brand_avg_price": {
                  "avg": {
                    "field": "price"
                  }
                }
              }
            }
          }
        }
      }
    }


```

可以看到，上述请求在最内部又嵌套了一个 "group_by_brand"，按照 band 字段进行分组，然后求品牌的平均价格：

```
    "group_by_brand": {
      "terms": {
        "field": "brand"
      },
      "aggs": {
        "brand_avg_price": {
          "avg": {
            "field": "price"
          }
        }
      }
    }


```

返回结果如下：

```
    {
      "took" : 54,
      "timed_out" : false,
      "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : {
          "value" : 8,
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
              "key" : "红色",
              "doc_count" : 4,
              "color_avg_price" : {
                "value" : 3250.0
              },
              "group_by_brand" : {
                "doc_count_error_upper_bound" : 0,
                "sum_other_doc_count" : 0,
                "buckets" : [
                  {
                    "key" : "长虹",
                    "doc_count" : 3,
                    "brand_avg_price" : {
                      "value" : 1666.6666666666667
                    }
                  },
                  {
                    "key" : "三星",
                    "doc_count" : 1,
                    "brand_avg_price" : {
                      "value" : 8000.0
                    }
                  }
                ]
              }
            },
            {
              "key" : "绿色",
              "doc_count" : 2,
              "color_avg_price" : {
                "value" : 2100.0
              },
              "group_by_brand" : {
                "doc_count_error_upper_bound" : 0,
                "sum_other_doc_count" : 0,
                "buckets" : [
                  {
                    "key" : "TCL",
                    "doc_count" : 1,
                    "brand_avg_price" : {
                      "value" : 1200.0
                    }
                  },
                  {
                    "key" : "小米",
                    "doc_count" : 1,
                    "brand_avg_price" : {
                      "value" : 3000.0
                    }
                  }
                ]
              }
            },
            {
              "key" : "蓝色",
              "doc_count" : 2,
              "color_avg_price" : {
                "value" : 2000.0
              },
              "group_by_brand" : {
                "doc_count_error_upper_bound" : 0,
                "sum_other_doc_count" : 0,
                "buckets" : [
                  {
                    "key" : "TCL",
                    "doc_count" : 1,
                    "brand_avg_price" : {
                      "value" : 1500.0
                    }
                  },
                  {
                    "key" : "小米",
                    "doc_count" : 1,
                    "brand_avg_price" : {
                      "value" : 2500.0
                    }
                  }
                ]
              }
            }
          ]
        }
      }
    }


```

### 3.4 metric：统计极值

我们现在需要统计每种颜色的电视机的最高价和最低价：

```
    GET /tvs/_search
    {
       "size" : 0,
       "aggs": {
          "colors": {
             "terms": {
                "field": "color"
             },
             "aggs": {
                "min_price" : { "min": { "field": "price"} }, 
                "max_price" : { "max": { "field": "price"} }
             }
          }
       }
    }


```

返回结果就不贴了，通过上述的几个示例我们可以看到，90% 的常见数据分析操作，无非就是 count、avg、max、min、sum 之类的 metric 操作。

四、总结
----

本章，我介绍了 Elasticsearch 中的聚合分析操作，聚合分析的核心就是先分组，然后对组内的记录执行 metric 操作，也可以分组之后再细粒度分组。常用的 metric 操作无非就是些求总数、平均值、极值之类的操作。
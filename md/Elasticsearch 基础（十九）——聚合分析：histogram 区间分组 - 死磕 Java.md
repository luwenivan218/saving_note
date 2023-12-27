> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1152502831)

> 我们在上一章讲到了 Elasticsearch 在聚合分析中，可以使用 “term” 对指定 field 按照数量分组。本章，我们将介绍另一种区间分组的方式——histogram。一、简介

我们在上一章讲到了 Elasticsearch 在聚合分析中，可以使用 “term” 对指定 field 按照数量分组。本章，我们将介绍另一种区间分组的方式——histogram。

一、简介
----

### 1.1 histogram

histogram，它会接收一个 field，然后按照这个 field 值的各个范围区间，进行 bucket 分组操作：

```
    GET /tvs/_search
    {
       "size" : 0,
       "aggs":{
          "price":{
             "histogram":{ 
                "field": "price",
                "interval": 2000
             }
          }
       }
    }


```

上述请求中，我们对 “price” 字段进行区间分组，区间间隔为 2000，返回结果如下：

```
    {
      "took" : 3,
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
        "price" : {
          "buckets" : [
            {
              "key" : 0.0,
              "doc_count" : 3
            },
            {
              "key" : 2000.0,
              "doc_count" : 4
            },
            {
              "key" : 4000.0,
              "doc_count" : 0
            },
            {
              "key" : 6000.0,
              "doc_count" : 0
            },
            {
              "key" : 8000.0,
              "doc_count" : 1
            }
          ]
        }
      }
    }


```

按照区间分组之后，我们就可以对各个 bucket 执行 metric 操作了，比如计算总和：

```
    GET /tvs/_search
    {
       "size" : 0,
       "aggs":{
          "price":{
             "histogram":{ 
                "field": "price",
                "interval": 2000
             },
             "aggs":{
                "revenue": {
                   "sum": { 
                     "field" : "price"
                   }
                 }
             }
          }
       }
    }


```

### 1.2 date_histogram

如果我们希望的按区间分组的字段是 date 类型的，那么需要用到`date_histogram`关键字。比如：

```
    GET /tvs/_search
    {
       "size" : 0,
       "aggs": {
          "sales": {
             "date_histogram": {
                "field": "sold_date",
                "interval": "month", 
                "format": "yyyy-MM-dd",
                "min_doc_count" : 0, 
                "extended_bounds" : { 
                    "min" : "2016-01-01",
                    "max" : "2017-12-31"
                }
             }
          }
       }
    }


```

解释下上述几个关键参数：

*   min_doc_count：某个日期区间内的 doc 数量至少要等于这个参数，这个区间才会返回；
*   extended_bounds：划分 bucket 的时候，会限定在这个起始日期和截止日期内。

二、实战
----

### 2.1 date_histogram

假设我们现在的需求是：统计每季度每个品牌的电视销售额，那么可以这样构造请求：

```
    GET /tvs/_search 
    {
      "size": 0,
      "aggs": {
        "group_by_sold_date": {
          "date_histogram": {
            "field": "sold_date",
            "interval": "quarter",
            "format": "yyyy-MM-dd",
            "min_doc_count": 0,
            "extended_bounds": {
              "min": "2016-01-01",
              "max": "2017-12-31"
            }
          },
          "aggs": {
            "total_sum_price": {
              "sum": {
                "field": "price"
              }
            },  
            "group_by_brand": {
              "terms": {
                "field": "brand"
              },
              "aggs": {
                "sum_price": {
                  "sum": {
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

上述请求其实就是先按日期进行分组，然后下钻到组内再按照品牌分组，最后对每个子组执行求和 metric 操作，结果如下：

```
    {
      "took" : 97,
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
        "group_by_sold_date" : {
          "buckets" : [
            {
              "key_as_string" : "2016-01-01",
              "key" : 1451606400000,
              "doc_count" : 0,
              "total_sum_price" : {
                "value" : 0.0
              },
              "group_by_brand" : {
                "doc_count_error_upper_bound" : 0,
                "sum_other_doc_count" : 0,
                "buckets" : [ ]
              }
            },
            {
              "key_as_string" : "2016-04-01",
              "key" : 1459468800000,
              "doc_count" : 1,
              "total_sum_price" : {
                "value" : 3000.0
              },
              "group_by_brand" : {
                "doc_count_error_upper_bound" : 0,
                "sum_other_doc_count" : 0,
                "buckets" : [
                  {
                    "key" : "小米",
                    "doc_count" : 1,
                    "sum_price" : {
                      "value" : 3000.0
                    }
                  }
                ]
              }
            },
            {
              "key_as_string" : "2016-07-01",
              "key" : 1467331200000,
              "doc_count" : 2,
              "total_sum_price" : {
                "value" : 2700.0
              },
              "group_by_brand" : {
                "doc_count_error_upper_bound" : 0,
                "sum_other_doc_count" : 0,
                "buckets" : [
                  {
                    "key" : "TCL",
                    "doc_count" : 2,
                    "sum_price" : {
                      "value" : 2700.0
                    }
                  }
                ]
              }
            },
            {
              "key_as_string" : "2016-10-01",
              "key" : 1475280000000,
              "doc_count" : 3,
              "total_sum_price" : {
                "value" : 5000.0
              },
              "group_by_brand" : {
                "doc_count_error_upper_bound" : 0,
                "sum_other_doc_count" : 0,
                "buckets" : [
                  {
                    "key" : "长虹",
                    "doc_count" : 3,
                    "sum_price" : {
                      "value" : 5000.0
                    }
                  }
                ]
              }
            },
            {
              "key_as_string" : "2017-01-01",
              "key" : 1483228800000,
              "doc_count" : 2,
              "total_sum_price" : {
                "value" : 10500.0
              },
              "group_by_brand" : {
                "doc_count_error_upper_bound" : 0,
                "sum_other_doc_count" : 0,
                "buckets" : [
                  {
                    "key" : "三星",
                    "doc_count" : 1,
                    "sum_price" : {
                      "value" : 8000.0
                    }
                  },
                  {
                    "key" : "小米",
                    "doc_count" : 1,
                    "sum_price" : {
                      "value" : 2500.0
                    }
                  }
                ]
              }
            },
            {
              "key_as_string" : "2017-04-01",
              "key" : 1491004800000,
              "doc_count" : 0,
              "total_sum_price" : {
                "value" : 0.0
              },
              "group_by_brand" : {
                "doc_count_error_upper_bound" : 0,
                "sum_other_doc_count" : 0,
                "buckets" : [ ]
              }
            },
            {
              "key_as_string" : "2017-07-01",
              "key" : 1498867200000,
              "doc_count" : 0,
              "total_sum_price" : {
                "value" : 0.0
              },
              "group_by_brand" : {
                "doc_count_error_upper_bound" : 0,
                "sum_other_doc_count" : 0,
                "buckets" : [ ]
              }
            },
            {
              "key_as_string" : "2017-10-01",
              "key" : 1506816000000,
              "doc_count" : 0,
              "total_sum_price" : {
                "value" : 0.0
              },
              "group_by_brand" : {
                "doc_count_error_upper_bound" : 0,
                "sum_other_doc_count" : 0,
                "buckets" : [ ]
              }
            }
          ]
        }
      }
    }


```

三、总结
----

本章，我介绍了聚合分析中的区间分组，Elasticsearch 采用`histogram`关键字来完成对指定字段值的区间分组，如果我们想要分组的字段类型为日期，则需要使用`date_histogram`关键字。
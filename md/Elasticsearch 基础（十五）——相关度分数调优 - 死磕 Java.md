> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1123880375)

> 上一章，我对 Elasticsearch 中的整个相关度分数算法的核心思想和原理进行了讲解，包括 TF/IDF，vectorspacemodel，booleanmodel 等等。本章，

转

 2023-08-08  阅读 (137)  点赞 (0)

原文地址：https://www.tpvlog.com/article/146

上一章，我对 Elasticsearch 中的整个相关度分数算法的核心思想和原理进行了讲解，包括 TF/IDF，vector space model，boolean model 等等。  
本章，我们就来看看，实际在使用 Elasticsearch 的过程中，如何对相关度分数进行调优。综合来讲，主要有如下四种方法，我们一一来看下：

*   query-time boost
*   negative boost
*   constant_score
*   function_score

一、query-time boost
------------------

query-time boost 就是利用`boost`增强某个 query 的权重，比如下面的查询有两个搜索条件，针对 title 字段的查询由于添加了`boost`参数，使其权重更大，所以 title 在匹配 doc 中分数占比会更大：

```
    GET /forum/_search
    {
      "query": {
        "bool": {
          "should": [
            {
              "match": {
                "title": {
                  "query": "java spark",
                  "boost": 2
                }
              }
            },
            {
              "match": {
                "content": "java spark"
              }
            }
          ]
        }
      }
    }


```

二、negative boost
----------------

negative boost，主要用于减少某些字段的权重，可以看成是 query-time boost 的反向参数。

比如我们有这样一个查询需求：搜索 content 字段中包含 "java"，但不包含 spark 的 "document"。那么，可能会像下面这样构造查询请求：

```
    GET /forum/_search 
    {
      "query": {
        "bool": {
          "must": [
            {
              "match": {
                "content": "java"
              }
            }
          ],
          "must_not": [
            {
              "match": {
                "content": "spark"
              }
            }
          ]
        }
      }
    }


```

但是有时候，我们不希望完全排除某个关键字，可能只是希望如果字段中包含某个关键字，就降低它的分数，比如上面的 spark。

对于这种需求，可以使用`negative_boost`，包含了 negative term 的 document，其分数会乘以`negative boost`：

```
    GET /forum/_search 
    {
      "query": {
        "boosting": {
          "positive": {
            "match": {
              "content": "java"
            }
          },
          "negative": {
            "match": {
              "content": "spark"
            }
          },
          "negative_boost": 0.2
        }
      }
    }


```

三、constant_score
----------------

如果我们压根不需要相关度评分，就直接用`constant_score`加 filter，这样所有的 doc 分数都是 1，没有评分的概念：

```
    GET /forum/_search 
    {
      "query": {
        "bool": {
          "should": [
            {
              "constant_score": {
                "query": {
                  "match": {
                    "title": "java"
                  }
                }
              }
            },
            {
              "constant_score": {
                "query": {
                  "match": {
                    "title": "spark"
                  }
                }
              }
            }
          ]
        }
      }
    }


```

四、function_score
----------------

我们还可以使用`function_score`，自定义相关度分数的算法。

比如我们有这样一个需求：希望看某个帖子的人越多，那么该帖子的分数就越高，帖子浏览数可以定义为一个`follower_num`字段。那么可以像下面这样使用`function_score`：

```
    GET /forum/_search
    {
      "query": {
        "function_score": {
          "query": {
            "multi_match": {
              "query": "java spark",
              "fields": ["tile", "content"]
            }
          },
          "field_value_factor": {
            "field": "follower_num",
            "modifier": "log1p",
            "factor": 0.5
          },
          "boost_mode": "sum",
          "max_boost": 2
        }
      }
    }


```

上述请求中：

*   `log1p`是一个函数，用于对字段分数进行修正：`new_score = old_score * log(1 + factor * follower_num)`；
*   `boost_mode`，用于决定最终 doc 分数与指定字段的值如何计算：multiply，sum，min，max，replace；
*   `max_boost`，用于限制计算出来的分数不要超过 max_boost 指定的值。

五、总结
----

本章，我介绍了如何对相关度分数进行调优。综合来讲，主要有如下四种方法：

*   query-time boost
*   negative boost
*   constant_score
*   function_score
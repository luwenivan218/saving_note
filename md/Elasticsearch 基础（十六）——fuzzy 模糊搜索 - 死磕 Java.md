> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1126643346)

> 本章，我们来介绍下 Elasticsearch 中的 fuzzy 模糊搜索。什么是模糊搜索？举个例子，我们输入的搜索关键字可能会出现拼写错误的情况，比如在下面的 documents 中搜索

转

 2023-08-08  阅读 (226)  点赞 (0)

原文地址：https://www.tpvlog.com/article/146

本章，我们来介绍下 Elasticsearch 中的 fuzzy 模糊搜索。什么是模糊搜索？举个例子，我们输入的搜索关键字可能会出现拼写错误的情况，比如在下面的 documents 中搜索 “hello world”，结果拼写成了 “hallo world”：

```
    
    hello world
    
    hello java


```

fuzzy 模糊搜索技术，会自动将拼写错误的搜索文本进行纠正，纠正以后去尝试匹配索引中的数据。

一、使用示例
------

我们可以像下面这样构造模糊搜索请求：

```
    GET /my_index/_search 
    {
      "query": {
        "fuzzy": {
          "text": {
            "value": "surprize",
            "fuzziness": 2
          }
        }
      }
    }


```

`fuzziness`表示模糊度，其数值就是最多可以纠正几个字母。

另一种使用 fuzzy 模糊搜索的语法如下，也是比较常用的语法：

```
    GET /my_index/_search 
    {
      "query": {
        "match": {
          "text": {
            "query": "SURPIZE ME",
            "fuzziness": "AUTO",
            "operator": "and"
          }
        }
      }
    }


```

二、总结
----

本章，我们介绍了 Elasticsearch 中的 fuzzy 模糊搜索，主要有两种使用的方式。一般来说，实际更多的是直接在 match query 中使用 fuzziness。
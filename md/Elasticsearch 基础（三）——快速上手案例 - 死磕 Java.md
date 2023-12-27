> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/6124711740)

> 本章，我将通过引入一个实战案例，带领新手童鞋了解下 Elasticsearch 的基本使用，请先确保按照上一章的讲解搭建好了 Elasticsearch 开发环境。一、案例背景假设我们

本章，我将通过引入一个实战案例，带领新手童鞋了解下 Elasticsearch 的基本使用，请先确保按照上一章的讲解搭建好了 Elasticsearch 开发环境。

一、案例背景
------

假设我们有一个电商网站，现在需要基于 Elasticsearch 构建一个商品管理后台系统。这个系统需要提供以下功能：

1.  对商品信息进行 CRUD（增删改查）操作；
2.  执行简单的结构化查询；
3.  执行简单的全文检索，以及复杂的 phrase（短语）检索；
4.  对于全文检索的结果，进行高亮显示；
5.  对数据进行简单的聚合分析

我将基于 Elasticsearch，带领新手童鞋实现上述功能，从而掌握 ES 的基本使用。

二、CRUD 操作
---------

我们先来进行商品的增删改查操作。

### 2.1 新增商品

新增商品数据，其实就是新增一条 document，ES 的语法如下：

```
    
    PUT /{index}/{type}/{id}
    {
        
    }


```

我们新建三条 document：

```
    PUT /ecommerce/product/1
    {
        "name" : "gaolujie yagao",
        "desc" :  "gaoxiao meibai",
        "price" :  30,
        "producer" :      "gaolujie producer",
        "tags": [ "meibai", "fangzhu" ]
    }
    
    PUT /ecommerce/product/2
    {
        "name" : "jiajieshi yagao",
        "desc" :  "youxiao fangzhu",
        "price" :  25,
        "producer" :      "jiajieshi producer",
        "tags": [ "fangzhu" ]
    }
    
    PUT /ecommerce/product/3
    {
        "name" : "zhonghua yagao",
        "desc" :  "caoben zhiwu",
        "price" :  40,
        "producer" :      "zhonghua producer",
        "tags": [ "qingxin" ]
    }


```

> Elasticsearch 会自动建立 Index 和 Type，同时，ES 默认会对 document 的每个 field 都建立倒排索引，让其可以被搜索。

### 2.2 查询商品

查询商品数据，ES 语法如下：

```
    
    GET /{index}/{type}/{id}


```

比如，我们执行`GET /ecommerce/product/1`，返回如下信息：

```
    {
      "_index" : "ecommerce",
      "_type" : "product",
      "_id" : "1",
      "_version" : 1,
      "_seq_no" : 0,
      "_primary_term" : 1,
      "found" : true,
      "_source" : {
        "name" : "gaolujie yagao",
        "desc" : "gaoxiao meibai",
        "price" : 30,
        "producer" : "gaolujie producer",
        "tags" : [
          "meibai",
          "fangzhu"
        ]
      }
    }


```

### 2.3 修改商品

修改商品数据，有两种方式： **替换** 、 **更新** 。

**替换方式** 的 ES 语法如下：

```
    
    PUT /{index}/{type}/{id}
    {
        
    }


```

比如，我们执行：

```
    PUT /ecommerce/product/1
    {
        "name" : "jiaqiangban gaolujie yagao",
        "desc" :  "gaoxiao meibai",
        "price" :  30,
        "producer" :      "gaolujie producer",
        "tags": [ "meibai", "fangzhu" ]
    }


```

返回如下信息：

```
    {
      "_index" : "ecommerce",
      "_type" : "product",
      "_id" : "1",
      "_version" : 2,
      "result" : "updated",
      "_shards" : {
        "total" : 2,
        "successful" : 1,
        "failed" : 0
      },
      "_seq_no" : 2,
      "_primary_term" : 1
    }


```

> 注意，使用替换方式去修改商品信息时，document 的所有 field 都需要带上。

我们再来看下更新方式。 **更新方式** 的语法如下：

```
    
    POST /{index}/{type}/{id}/_update
    {
        "doc":{
            
        }
    }


```

比如，我们执行：

```
    POST /ecommerce/product/1/_update
    {
      "doc": {
        "name": "jiaqiangban gaolujie yagao"
      }
    }


```

返回如下信息：

```
    {
      "_index" : "ecommerce",
      "_type" : "product",
      "_id" : "1",
      "_version" : 4,
      "result" : "noop",
      "_shards" : {
        "total" : 0,
        "successful" : 0,
        "failed" : 0
      },
      "_seq_no" : 4,
      "_primary_term" : 1
    }


```

### 2.4 删除商品

删除商品数据，ES 语法如下：

```
    
    DELETE /{index}/{type}/{id}


```

比如，我们执行`DELETE /ecommerce/product/1`，返回如下信息：

```
    {
      "_index" : "ecommerce",
      "_type" : "product",
      "_id" : "1",
      "_version" : 5,
      "result" : "deleted",
      "_shards" : {
        "total" : 2,
        "successful" : 1,
        "failed" : 0
      },
      "_seq_no" : 5,
      "_primary_term" : 1
    }


```

三、数据检索
------

本节，我们来讲解下 Elasticsearch 中的各种常见的数据检索方式。

### 3.1 query string search

_**语法：**_ `GET /{index}/{type}/_search`

我们在很多网站进行搜索的时候，搜索词一般都是跟在 search 参数后面，以 query string 的 http 请求形式发起检索。比如，我们要搜索商品名中包含 “yagao” 关键子的所有商品，并将结果按照售价排序。那么就可以执行以下语句：

```
    GET /ecommerce/product/_search?q=name:yagao&sort=price:desc


```

返回结果：

```
    {
      "took" : 530,
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
        "hits" : [
          {
            "_index" : "ecommerce",
            "_type" : "product",
            "_id" : "3",
            "_score" : null,
            "_source" : {
              "name" : "zhonghua yagao",
              "desc" : "caoben zhiwu",
              "price" : 40,
              "producer" : "zhonghua producer",
              "tags" : [
                "qingxin"
              ]
            },
            "sort" : [
              40
            ]
          },
          {
            "_index" : "ecommerce",
            "_type" : "product",
            "_id" : "1",
            "_score" : null,
            "_source" : {
              "name" : "gaolujie yagao",
              "desc" : "gaoxiao meibai",
              "price" : 30,
              "producer" : "gaolujie producer",
              "tags" : [
                "meibai",
                "fangzhu"
              ]
            },
            "sort" : [
              30
            ]
          },
          {
            "_index" : "ecommerce",
            "_type" : "product",
            "_id" : "2",
            "_score" : null,
            "_source" : {
              "name" : "jiajieshi yagao",
              "desc" : "youxiao fangzhu",
              "price" : 25,
              "producer" : "jiajieshi producer",
              "tags" : [
                "fangzhu"
              ]
            },
            "sort" : [
              25
            ]
          }
        ]
      }
    }


```

我们对上面的一些关键返回字段的含义说明下：  
**took：** 耗费时长（毫秒）  
**timed_out：** 是否超时  
**_shards** ：数据拆成的分片数，所以对于搜索请求，会打到所有的 primary shard（或者是 primary shard 对应的某个 replica shard）；  
**hits.total** ：查询结果的数量，3 个 document；  
**hits.max_score** ：score 的含义，就是 document 对于一个 search 的相关度的匹配分数，越相关，就越匹配，分数也高；  
**hits.hits** ：包含了匹配搜索的 document 的详细数据。

> query string search 适用于一些临时、快速的检索请求，如果查询请求很复杂，那么 query string search 是不太适用的，所以在生产环境中，几乎很少使用 **query string search** 。

### 3.2 query DSL

所谓 DSL，就是 _**Domain Specified Language**_ ，是 Elasticsearch 中很常用的一种检索方式。

_**语法：**_

```
    GET /{index}/{type}/_search
    {
        "query":{}
    }


```

我们通过 “查询商品名中包含 yagao 关键子的所有商品，并将结果按照售价排序” 这个示例来看下如何使用：

```
    GET /ecommerce/product/_search
    {
        "query" : {
            "match" : {
                "name" : "yagao"
            }
        },
        "sort": [
            { "price": "desc" }
        ]
    }


```

可以看到，所有查询请求参数全部放到了一个 http requstbody 里面，所以可以构建复杂的查询，适合生产环境使用。

### 3.3 query filter

query filter 主要用于对数据进行过滤。比如，我们要搜索商品名称包含”yagao“，且售价大于 25 元的商品：

```
    GET /ecommerce/product/_search
    {
        "query" : {
            "bool" : {
                "must" : {
                    "match" : {
                        "name" : "yagao" 
                    }
                },
                "filter" : {
                    "range" : {
                        "price" : { "gt" : 25 } 
                    }
                }
            }
        }
    }


```

上述 bool 里面可以封装多个查询条件。

### 3.4 full-text search

full-text search，就是全文检索。比如我们要查询 producer 字段中包含”producer“或”jiajieshi“的所有商品：

```
    GET /ecommerce/product/_search
    {
        "query" : {
            "match" : {
                "producer" : "producer jiajieshi"
            }
        }
    }


```

可以看到，我们在上述的”match“里面有空格分隔开了 "producer jiajieshi"，这样只要 producer 字段中包含了上述任意一个字符，就都会被检索到。结果如下：

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
          "value" : 3,
          "relation" : "eq"
        },
        "max_score" : 1.1143606,
        "hits" : [
          {
            "_index" : "ecommerce",
            "_type" : "product",
            "_id" : "2",
            "_score" : 1.1143606,
            "_source" : {
              "name" : "jiajieshi yagao",
              "desc" : "youxiao fangzhu",
              "price" : 25,
              "producer" : "jiajieshi producer",
              "tags" : [
                "fangzhu"
              ]
            }
          },
          {
            "_index" : "ecommerce",
            "_type" : "product",
            "_id" : "1",
            "_score" : 0.13353139,
            "_source" : {
              "name" : "gaolujie yagao",
              "desc" : "gaoxiao meibai",
              "price" : 30,
              "producer" : "gaolujie producer",
              "tags" : [
                "meibai",
                "fangzhu"
              ]
            }
          },
          {
            "_index" : "ecommerce",
            "_type" : "product",
            "_id" : "3",
            "_score" : 0.13353139,
            "_source" : {
              "name" : "zhonghua yagao",
              "desc" : "caoben zhiwu",
              "price" : 40,
              "producer" : "zhonghua producer",
              "tags" : [
                "qingxin"
              ]
            }
          }
        ]
      }
    }


```

注意，采用全文检索时，返回的每一项数据中有一个 “_score” 字段，这个就是 “相关度分数” 的意思，分数越高，则越接近检索词。

### 3.5 phrase search

phrase search（短语搜索），跟全文检索刚好相反。在全文检索会将输入的搜索串拆解开来，去倒排索引里面去一一匹配，只要能匹配上任意一个拆解后的单词，就可以作为结果返回。而 phrase search 则要求输入的关键字必须在出现指定的文本中，一模一样才算匹配。

举个例子，我们要查询 producer 字段中包含关键字”jiajieshi producer“的所有商品：

```
    GET /ecommerce/product/_search
    {
        "query" : {
            "match_phrase" : {
                "producer" : "jiajieshi producer"
            }
        }
    }


```

返回结果如下，可以看到，只包含一条 producer 字段为 “jiajieshi producer” 的记录：

```
    {
      "took" : 48,
      "timed_out" : false,
      "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : {
          "value" : 1,
          "relation" : "eq"
        },
        "max_score" : 1.1143606,
        "hits" : [
          {
            "_index" : "ecommerce",
            "_type" : "product",
            "_id" : "2",
            "_score" : 1.1143606,
            "_source" : {
              "name" : "jiajieshi yagao",
              "desc" : "youxiao fangzhu",
              "price" : 25,
              "producer" : "jiajieshi producer",
              "tags" : [
                "fangzhu"
              ]
            }
          }
        ]
      }
    }


```

### 3.6 highlight search

highlight search（高亮搜索结果）。举个例子，我们希望检索 name 字段包含 “yaogao” 的所有商品，然后对检索结果中的 “yaogao” 文本进行高亮：

```
    GET /ecommerce/product/_search
    {
        "query" : {
            "match" : {
                "name" : "yagao"
            }
        },
        "highlight": {
            "fields" : {
                "name" : {}
            }
        }
    }


```

返回结果如下，可以看到，匹配项的结果里多了个 “highlight” 字段，里面用`<em>`高亮了匹配的文本：

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
          "value" : 3,
          "relation" : "eq"
        },
        "max_score" : 0.13353139,
        "hits" : [
          {
            "_index" : "ecommerce",
            "_type" : "product",
            "_id" : "1",
            "_score" : 0.13353139,
            "_source" : {
              "name" : "gaolujie yagao",
              "desc" : "gaoxiao meibai",
              "price" : 30,
              "producer" : "gaolujie producer",
              "tags" : [
                "meibai",
                "fangzhu"
              ]
            },
            "highlight" : {
              "name" : [
                "gaolujie <em>yagao</em>"
              ]
            }
          },
          {
            "_index" : "ecommerce",
            "_type" : "product",
            "_id" : "2",
            "_score" : 0.13353139,
            "_source" : {
              "name" : "jiajieshi yagao",
              "desc" : "youxiao fangzhu",
              "price" : 25,
              "producer" : "jiajieshi producer",
              "tags" : [
                "fangzhu"
              ]
            },
            "highlight" : {
              "name" : [
                "jiajieshi <em>yagao</em>"
              ]
            }
          },
          {
            "_index" : "ecommerce",
            "_type" : "product",
            "_id" : "3",
            "_score" : 0.13353139,
            "_source" : {
              "name" : "zhonghua yagao",
              "desc" : "caoben zhiwu",
              "price" : 40,
              "producer" : "zhonghua producer",
              "tags" : [
                "qingxin"
              ]
            },
            "highlight" : {
              "name" : [
                "zhonghua <em>yagao</em>"
              ]
            }
          }
        ]
      }
    }


```

四、数据分析
------

这一节，我们来看下如何利用 Elasticsearch 进行数据分析。首先，我们还是录入以下 document 数据：

```
    PUT /ecommerce/product/1
    {
        "name" : "gaolujie yagao",
        "desc" :  "gaoxiao meibai",
        "price" :  30,
        "producer" :      "gaolujie producer",
        "tags": [ "meibai", "fangzhu" ]
    }
    
    PUT /ecommerce/product/2
    {
        "name" : "jiajieshi yagao",
        "desc" :  "youxiao fangzhu",
        "price" :  25,
        "producer" :      "jiajieshi producer",
        "tags": [ "fangzhu" ]
    }
    
    PUT /ecommerce/product/3
    {
        "name" : "zhonghua yagao",
        "desc" :  "caoben zhiwu",
        "price" :  40,
        "producer" :      "zhonghua producer",
        "tags": [ "qingxin" ]
    }


```

注意，要使用聚合分析，需要将文本 field 的`fielddata`属性设置为 true，所以还要执行：

```
    PUT /ecommerce/_mapping
    {
      "properties": {
        "tags": {
          "type": "text",
          "fielddata": true
        }
      }
    }


```

> 关于 mapping，我后续章节会详细讲解，这里读者只要跟着这么操作就行了。

### 4.1 聚合分析

**语法：**

```
    GET /{index}/{type}/_search
    {
      "aggs": {
        "聚合名称": {
          "terms": { "field": "分组字段" }
        }
      }
    }


```

我们的第一个需求是：计算每个 tag 下的商品数量。比如，"fangzhu" 这个 tag，一共有两个商品包含它。

请求如下，意思是新建一个名称为 “group_by_tags” 的聚合，按照 tags 这个字段进行分组：

```
    GET /ecommerce/product/_search
    {
      "aggs": {
        "group_by_tags": {
          "terms": { "field": "tags" }
        }
      }
    }


```

响应如下：

```
    {
      "took" : 6,
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
        "max_score" : 1.0,
        "hits" : [
          {
            "_index" : "ecommerce",
            "_type" : "product",
            "_id" : "1",
            "_score" : 1.0,
            "_source" : {
              "name" : "gaolujie yagao",
              "desc" : "gaoxiao meibai",
              "price" : 30,
              "producer" : "gaolujie producer",
              "tags" : [
                "meibai",
                "fangzhu"
              ]
            }
          },
          {
            "_index" : "ecommerce",
            "_type" : "product",
            "_id" : "2",
            "_score" : 1.0,
            "_source" : {
              "name" : "jiajieshi yagao",
              "desc" : "youxiao fangzhu",
              "price" : 25,
              "producer" : "jiajieshi producer",
              "tags" : [
                "fangzhu"
              ]
            }
          },
          {
            "_index" : "ecommerce",
            "_type" : "product",
            "_id" : "3",
            "_score" : 1.0,
            "_source" : {
              "name" : "zhonghua yagao",
              "desc" : "caoben zhiwu",
              "price" : 40,
              "producer" : "zhonghua producer",
              "tags" : [
                "qingxin"
              ]
            }
          }
        ]
      },
      "aggregations" : {
        "group_by_tags" : {
          "doc_count_error_upper_bound" : 0,
          "sum_other_doc_count" : 0,
          "buckets" : [
            {
              "key" : "fangzhu",
              "doc_count" : 2
            },
            {
              "key" : "meibai",
              "doc_count" : 1
            },
            {
              "key" : "qingxin",
              "doc_count" : 1
            }
          ]
        }
      }
    }


```

我们关键看：

```
    "aggregations" : {
      "group_by_tags" : {
        "doc_count_error_upper_bound" : 0,
        "sum_other_doc_count" : 0,
        "buckets" : [
          {
            "key" : "fangzhu",
            "doc_count" : 2
          },
          {
            "key" : "meibai",
            "doc_count" : 1
          },
          {
            "key" : "qingxin",
            "doc_count" : 1
          }
        ]
      }
    }


```

上面的 “buckets” 就是按照 tags 进行分组的结果，key 和 doc_count 就表示每个 tag 对应的数量。

* * *

我们再来个复杂点的需求：按照指定的价格范围区间进行分组，然后在每组内再按照 tag 进行分组，最后再计算组内的平均价格。

请求：

```
    GET /ecommerce/product/_search
    {
      "size": 0,
      "aggs": {
        "group_by_price": {
          "range": {
            "field": "price",
            "ranges": [
              {
                "from": 0,
                "to": 20
              },
              {
                "from": 20,
                "to": 40
              },
              {
                "from": 40,
                "to": 50
              }
            ]
          },
          "aggs": {
            "group_by_tags": {
              "terms": {
                "field": "tags"
              },
              "aggs": {
                "average_price": {
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

响应：

```
    {
      "took" : 11,
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
        "group_by_price" : {
          "buckets" : [
            {
              "key" : "0.0-20.0",
              "from" : 0.0,
              "to" : 20.0,
              "doc_count" : 0,
              "group_by_tags" : {
                "doc_count_error_upper_bound" : 0,
                "sum_other_doc_count" : 0,
                "buckets" : [ ]
              }
            },
            {
              "key" : "20.0-40.0",
              "from" : 20.0,
              "to" : 40.0,
              "doc_count" : 2,
              "group_by_tags" : {
                "doc_count_error_upper_bound" : 0,
                "sum_other_doc_count" : 0,
                "buckets" : [
                  {
                    "key" : "fangzhu",
                    "doc_count" : 2,
                    "average_price" : {
                      "value" : 27.5
                    }
                  },
                  {
                    "key" : "meibai",
                    "doc_count" : 1,
                    "average_price" : {
                      "value" : 30.0
                    }
                  }
                ]
              }
            },
            {
              "key" : "40.0-50.0",
              "from" : 40.0,
              "to" : 50.0,
              "doc_count" : 1,
              "group_by_tags" : {
                "doc_count_error_upper_bound" : 0,
                "sum_other_doc_count" : 0,
                "buckets" : [
                  {
                    "key" : "qingxin",
                    "doc_count" : 1,
                    "average_price" : {
                      "value" : 40.0
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

五、总结
----

本章，我通过一个简单的案例讲解了 Elasticsearch 的数据增删改查，基本的数据检索和数据分析，相信新手童鞋们对 Elasticsearch 的基本操作已经掌握了。下一章，我将讲解 Elasticsearch 的分布式架构。
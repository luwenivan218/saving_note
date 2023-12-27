> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1095244379)

> 本章，我们来讲下全文检索（FullTextQuery）中的多字段搜索。和上一章 termfilter 不一样的是：全文检索不是搜索 exactvalue，而是对检索关键字进行分词后，

本章，我们来讲下全文检索（Full Text Query）中的多字段搜索。和上一章 term filter 不一样的是：全文检索不是搜索 exact value，而是对检索关键字进行分词后，实现倒排索引检索。多字段搜索，说白了，就是希望在多个不同的 field 中检索关键字。

一、案例实战
------

### 1.1 数据准备

我们假设已经录入了以下 4 条文章数据：

```
    {
        "articleID" : "XHDK-A-1293-#fJ3",
        "userID" : 1,
        "hidden" : false,
        "postDate" : "2017-01-01",
        "tag" : [
            "java",
            "hadoop"
        ],
        "view_cnt" : 30,
        "title" : "this is java and elasticsearch blog"
    },
    {
        "articleID" : "KDKE-B-9947-#kL5",
        "userID" : 1,
        "hidden" : false,
        "postDate" : "2017-01-02",
        "tag" : [
            "java"
        ],
        "view_cnt" : 50,
        "title" : "this is java blog"
    },
    {
        "articleID" : "JODL-X-1937-#pV7",
        "userID" : 2,
        "hidden" : false,
        "postDate" : "2017-01-01",
        "tag" : [
            "hadoop"
        ],
        "view_cnt" : 100,
        "title" : "this is elasticsearch blog"
    },
    {
        "articleID" : "QQPX-R-3956-#aD8",
        "userID" : 2,
        "hidden" : true,
        "postDate" : "2017-01-02",
        "tag" : [
        "java",
            "elasticsearch"
        ],
        "view_cnt" : 80,
        "title" : "this is java, elasticsearch, hadoop blog"
    }


```

### 1.2 full text query 示例

我们先来看下全文检索的基本使用方式：

```
    
    GET /forum/_search
    {
        "query": {
            "match": {
                "title": "java elasticsearch"
            }
        }
    }


```

全文检索时，会对搜索关键字进行拆分，上述 "title" 字段默认就是 text 类型，所以最终会以倒排索引的方式查询，只有记录中的 “title” 包含了 “java” 或“elasticsearch”，都会被检索出来。

我们也可以用 bool 组合多个搜索条件：

```
    GET /forum/_search
    {
      "query": {
        "bool": {
          "must":     { "match": { "title": "java" }},
          "must_not": { "match": { "title": "spark"  }},
          "should": [
                      { "match": { "title": "hadoop" }},
                      { "match": { "title": "elasticsearch"   }}
          ]
        }
      }


```

#### minimum_should_match

如果我们希望指定的关键字中，必须至少匹配其中的多少个关键字，才能作为结果返回，可以利用`minimum_should_match`参数：

```
    GET /forum/article/_search
    {
      "query": {
        "match": {
          "title": {
            "query": "java elasticsearch spark hadoop",
            "minimum_should_match": "75%"
          }
        }
      }
    }


```

上述查询到的结果中，至少会包含 “java“、“elasticsearch“、“spark“、“hadoop” 中的三个。

#### boost 权重

我们可以通过`boost`进行权重控制，也就是对于检索关键字，我们希望拆分后的某些词被优先检索。Elasticsearch 进行相关度分数计算时，权重越大，相应的 relevance score 会越高，也就会优先被返回。默认情况下，搜索条件的权重都是 1。

举个例子，假设我们希望检索出 title 包含 hadoop 或 elasticsearch 的记录，但是希望 hadoop 优先搜索出来，那么可以设置 hadoop 的权重更大些：

```
    GET /forum/_search 
    {
      "query": {
        "bool": {
          "should": [
            {
              "match": {
                "title": {
                  "query": "hadoop",
                  "boost": 5
                }
              }
            },
            {
              "match": {
                "title": {
                  "query": "elasticsearch",
                  "boost": 2
                }
              }
            }
          ]
        }
      }
    }


```

> 注意：如果一个 index 有多个 shard 的话，搜索结果可能不准确。因为对于一个搜索请求，coordinate node 可能会将其转发给任意一个 shard。Elasticsearch 在计算相关度分数时，采用了 TF/IDF 算法，该算法需要知道关键字在所有 document 中出现的次数，而每个 shard 只包含了部分 document，TF/IDF 算法计算时只采用了当前 shard 中的所有 document 数，所以对于不同 shard 计算出的相关度分数可能都是不同的。

### 1.3 match query 底层原理

当我们使用 match query 进行检索时，Elasticsearch 底层会转换成 term 形式。比如针对下面这种检索：

```
    GET /forum/_search
    {
        "query": {
            "match": {
                "title": {
                    "query": "java elasticsearch",
                    "operator": "and"
                   }
            }
        }
    }


```

Elasticsearch 会将其转换成如下 term 形式：

```
    {
      "bool": {
        "should": [
          { "term": { "title": "java" }},
          { "term": { "title": "elasticsearch"   }}
        ]
      }
    }


```

二、best_fields 策略
----------------

所谓 best_fields 策略，就是对多个 filed 进行搜索匹配时，挑选某个 field 匹配度最高的那个分数，同时在多个 query 最高分相同的情况下，在一定程度上考虑其他 query 的分数。简单来说，就是对多个 field 进行搜索时，就想搜索到某一个 field 包含更多关键字的数据。

### 2.1 multi-field 搜索

语言描述实在太绕，我们通过一个例子来理解下，假设有五条 doc 记录：

```
    
    { "doc" : {"title" : "this is java and elasticsearch blog","content" : "i like to write best elasticsearch article"} }
    
    { "doc" : {"title" : "this is java blog","content" : "i think java is the best programming language"} }
    
    { "doc" : {"title" : "this is elasticsearch blog","content" : "i am only an elasticsearch beginner"} }
    
    { "doc" : {"title" : "this is java, elasticsearch, hadoop blog","content" : "elasticsearch and hadoop are all very good solution, i am a beginner"} }
    
    { "doc" : {"title" : "this is spark blog","content" : "spark is best big data solution based on scala ,an programming language similar to java"} }


```

我们希望搜索 title 或 content 中包含 “java” 或“solution”关键字的帖子，这其实就是典型的 multi-field 搜索，我们一般会像下面这样构建请求：

```
    
    GET /forum/_search
    {
        "query": {
            "bool": {
                "should": [
                    { "match": { "title": "java solution" }},
                    { "match": { "content":  "java solution" }}
                ]
            }
        }
    }


```

如果按照正常的思维，匹配度最高的应该是 doc5，因为只有它的 content 字段既包含 “java” 又包含“solution”。但事实上，doc5 的相关度分数（relevance score）并不是最高的，因为默认情况下，对于这种 multi-field 搜索，Elasticsearch 采用的是 **most_fields 策略** ，其算法大致是这样的：

1.  计算每个 query 的分数，然后求和。对于上述搜索，就是 “should” 中的两个 field 检索条件，比如 doc4 计算的结果分别是 1.1 和 1.2，相加为 2.3；
2.  计算 matched query 的数量，比如对于 doc4，两个 field 都能匹配到，数量就是 2；
3.  sum(每个 query 的分数）x count(matched query) / count(总 query 数量) 作为最终相关度分数。

对于 doc4，上述算法的计算结果就是：(1.1+1.2) x 2/2=2.3；而对于 doc5，title 字段是匹配不到结果的，所以 matched query=1，doc5 的最终分数可能是 (0+2.3) x 1/2=1.15，所以检索结果排在了 doc4 后面。

### 2.2 dis_max

我们希望的搜索结果应该是：某一个 field 匹配到了尽可能多的关键词，其分数更高；而不是尽可能多的 field 匹配到了少数的关键词，却排在了前面。

Elasticsearch 提供了`dis_max`语法，可以直接取多个 query 中，分数最高的那一个 query 的分数，比如像下面这样构建请求，doc5 的相关度分数就会上去：

```
    GET /forum/_search
    {
        "query": {
            "dis_max": {
                "queries": [
                    { "match": { "title": "java solution" }},
                    { "match": { "content":  "java solution" }}
                ]
            }
        }
    }


```

比如对于上述的 doc4，两个 field 检索的最终分数分别为 1.1 和 1.2，那就取最大值 1.2：

```
    { "match": { "title": "java solution" }} -> 1.1
    { "match": { "content":  "java solution" }} -> 1.2


```

对于 doc5，针对 “title” 的检索没有匹配结果，分数为 0，但 “content” 的分数为 2.3，所以取最大值 2.3：

```
    { "match": { "title": "java solution" }} -> 0
    { "match": { "content":  "java solution" }} -> 2.3


```

### 2.3 tie_breaker

`dis_max`只取多个 query 中，分数最高的那一个 query 的分数，而完全不考虑其它 query 的分数。但有时这并不能满足我们的需求，举个例子，我们希望检索 title 字段包含 “java solution” 或 "content" 字段包含 “java solution” 的帖子，最终满足条件的每个 doc 的匹配结果如下：

*   doc1，title 中包含 “java“，content 不包含 “java“、“solution“任何一个关键词；
*   doc2，title 中不包含任何一个关键词，content 中包含 “solution”；
*   doc3，title 中包含 “java“，content 中包含 “solution“。

最终搜索结果是，doc1 和 doc2 排在了 doc3 的前面，而不是我们期望的 doc3 排在最前面。此时我们可以利用`tie_breaker`参数将其他 query 的分数也考虑进去：

```
    GET /forum/_search
    {
        "query": {
            "dis_max": {
                "queries": [
                    { "match": { "title": "java solution" }},
                    { "match": { "content":  "java solution" }}
                ],
                "tie_breaker": 0.3
            }
        }
    }


```

`` `tie_breaker``的值，在 0-1 之间，其意义在于：将其他 query 的分数，乘以`tie_breaker`，然后再与最高分数的那个 query 进行计算，得到最终分数。

### 2.4 multi_match 搜索

上面我们讲的`dis_max`和`tie_breaker`其实就是 bese_fields 策略的核心实现原理了。Elasticsearch 还提供了一种 multi_match 搜索，来简化实现 bese_fields 策略：

```
    GET /forum/_search
    {
      "query": {
        "multi_match": {
            "query":                "java solution",
            "type":                 "best_fields", 
            "fields":               [ "title", "content" ],
            "tie_breaker":          0.3,
            "minimum_should_match": "50%" 
        }
      } 
    }


```

如果要用`dis_max`和`tie_breaker`和来实现同样的效果，则是下面这样，可以看到`multi_match`确实简化了编码：

```
    GET /forum/_search
    {
      "query": {
        "dis_max": {
          "queries":  [
            {
              "match": {
                "title": {
                  "query": "java solution",
                  "minimum_should_match": "50%"
                }
              }
            },
            {
              "match": {
                "body": {
                  "query": "java solution",
                  "minimum_should_match": "50%"
                }
              }
            }
          ],
          "tie_breaker": 0.3
        }
      } 
    }


```

### 2.5 优缺点

best_fields 策略是最常用，也是最符合人类思维的搜索策略。Google、Baidu 之类的搜索引擎，默认就是用的这种策略。

**优点：** 通过 best_fields 策略，以及综合考虑其他 field，还有`minimum_should_match`支持，可以尽可能精准地将匹配的结果推送到最前面。  
**缺点：** 除了那些精准匹配的结果，其他差不多大的结果，排序结果不是太均匀，没有什么区分度了。

三、most_fields 策略
----------------

most_fields 策略，也是 Elasticsearch 进行 multi-field 搜索时的默认策略，其实就是综合多个 field 一起进行搜索，尽可能多地让所有 query 参与到总分的计算中，结果不一定精准。

比如，某个 document 的一个 field 虽然包含更多的关键字，但是因为其他 document 有更多 field 匹配到了，所以其它的 doc 会排在前面。

我们可以通过以下方式显式使用 most_fields 策略：

```
    GET /forum/_search
    {
      "query": {
        "multi_match": {
            "query":                "java solution",
            "type":                 "most_fields", 
            "fields":               [ "title", "content" ]
        }
      } 
    }


```

### 3.1 优缺点

**优点：** 将尽可能匹配更多 field 的结果推送到最前面，整个排序结果是比较均匀的。  
**缺点：** 可能那些精准匹配的结果，无法推送到最前面。

四、cross-fields 策略
-----------------

cross-fields 搜索，就是跨多个 field 去搜索一个标识。比如姓名字段可以散落在多个 field 中，first_name 和 last_name，地址字段可以散落在 country、province、city 中，那么搜索人名或者地址，就是 cross-fields 搜索。

要进行 cross-fields 搜索，我们可能会立马想到使用上面讲的 **most_fields 策略** ，因为 multi_fields 会考虑多个 field 匹配的分数，而 cross-fields 搜索本身刚好就是多个 field 检索的问题。

我们通过示例来看下 cross-fields 搜索，假设有以下用户信息：

```
    
    { "doc" : {"author_first_name" : "Peter", "author_last_name" : "Smith"} }
    
    { "doc" : {"author_first_name" : "Smith", "author_last_name" : "Williams"} }
    
    { "doc" : {"author_first_name" : "Jack", "author_last_name" : "Ma"} }
    
    { "doc" : {"author_first_name" : "Robbin", "author_last_name" : "Li"} }
    
    { "doc" : {"author_first_name" : "Tonny", "author_last_name" : "Peter Smith"} }


```

我们希望检索姓名中包含 “Peter Smith” 的用户信息，一般会像下面这样构造请求：

```
    GET /forum/_search
    {
      "query": {
        "multi_match": {
          "query":       "Peter Smith",
          "type":        "most_fields",
          "fields":      [ "author_first_name", "author_last_name" ]
        }
      }
    }


```

检索出的结果包含：doc1、doc2、doc5，我们希望的结果应该是 doc5 排在最前面，然后是 doc1，最后才是 doc2，即 doc5>doc1>doc2， **但事实上，doc5 可能会排在最后** 。之所以会出现这种情况，跟 TF/IDF 算法有关，我这边不作赘述，后面会讲 TF/IDF 算法原理。

所以，如果我们需要进行 cross-fields 搜索，应该直接使用 multi_match 提供的 **cross-fields 策略** ：

```
    GET /forum/_search
    {
      "query": {
        "multi_match": {
          "query": "Peter Smith",
          "type": "cross_fields", 
          "operator": "and",
          "fields": ["author_first_name", "author_last_name"]
        }
      }
    }


```

使用 cross-fields 策略进行多字段检索时，会要求关键字拆分后的每个 term 必须出现在被检索的字段中。比如上面我们检索 “Peter Smith” 时，会拆成 “Peter” 和“Smith”两个 term，那就要求：

*   Peter 必须在 author_first_name 或 author_last_name 中出现；
*   Smith 必须在 author_first_name 或 author_last_name 中出现。

五、总结
----

全文检索时，如果需要针对多个 field 进行检索，我们一般会使用 match query 或 multi_match 语法。默认情况下，Elasticsearch 进行这类多字段检索的策略是 most_fields。读者要理解 most_fields 策略和 best_fields 策略的内容及其优缺点，根据自己的实际需求选择合适的策略。
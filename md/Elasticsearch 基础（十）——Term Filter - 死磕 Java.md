> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1073082040)

> 本章，我将通过示例讲解下 Elasticsearch 中 TermFilter 的使用。所谓 TermFilter，就是不会对搜索关键词进行分词操作，根据 exactvalue 进行搜索，数

本章，我将通过示例讲解下 Elasticsearch 中 Term Filter 的使用。所谓 Term Filter，就是不会对搜索关键词进行分词操作，根据 exact value 进行搜索，数字、boolean、date 天然支持这种方式。但是对于 text 类型的字段需要特别注意，ES 默认会对 text 字段先分词，再检索。

一、案例实战
------

### 1.1 数据准备

先批量插入一些论坛帖子信息，索引为`forum`：

```
    POST /forum/_bulk
    { "index": { "_id": 1 }}
    { "articleID" : "XHDK-A-1293-#fJ3", "userID" : 1, "hidden": false, "postDate": "2017-01-01" }
    { "index": { "_id": 2 }}
    { "articleID" : "KDKE-B-9947-#kL5", "userID" : 1, "hidden": false, "postDate": "2017-01-02" }
    { "index": { "_id": 3 }}
    { "articleID" : "JODL-X-1937-#pV7", "userID" : 2, "hidden": false, "postDate": "2017-01-01" }
    { "index": { "_id": 4 }}
    { "articleID" : "QQPX-R-3956-#aD8", "userID" : 2, "hidden": true, "postDate": "2017-01-02" }


```

#### mapping 结构

我们看下这个索引的 mapping：

```
    
    GET /forum/_mapping
    
    
    {
      "forum" : {
        "mappings" : {
          "properties" : {
            "articleID" : {
              "type" : "text",
              "fields" : {
                "keyword" : {
                  "type" : "keyword",
                  "ignore_above" : 256
                }
              }
            },
            "hidden" : {
              "type" : "boolean"
            },
            "postDate" : {
              "type" : "date"
            },
            "userID" : {
              "type" : "long"
            }
          }
        }
      }
    }


```

可以看到 postDate 默认就是`date`类型。这里关键说下 articleID，它的类型是`text`，Elasticsearch 默认会对`text`类型的字段进行分词，建立倒排索引；其次，还会生成一个 keyword 字段，这个 keyword 就是 articleID 的内容，不会分词，用于建立正排索引，上述 256 的意思是：如果内容过长，只保留 256 个字符。

我们通过`_analyze`命令来理解下，先看下默认的 articleID 字段：

```
    
    GET /forum/_analyze
    {
      "field": "articleID",
      "text":"XHDK-A-1293-#fJ3" 
    }
    
    
    {
      "tokens" : [
        {
          "token" : "xhdk",
          "start_offset" : 0,
          "end_offset" : 4,
          "type" : "<ALPHANUM>",
          "position" : 0
        },
        {
          "token" : "a",
          "start_offset" : 5,
          "end_offset" : 6,
          "type" : "<ALPHANUM>",
          "position" : 1
        },
        {
          "token" : "1293",
          "start_offset" : 7,
          "end_offset" : 11,
          "type" : "<NUM>",
          "position" : 2
        },
        {
          "token" : "fj3",
          "start_offset" : 13,
          "end_offset" : 16,
          "type" : "<ALPHANUM>",
          "position" : 3
        }
      ]
    }


```

再来看下 articleID.keyword 字段：

```
    
    GET /forum/_analyze
    {
      "field": "articleID.keyword",
      "text":"XHDK-A-1293-#fJ3"
    
    }
    
    {
      "tokens" : [
        {
          "token" : "XHDK-A-1293-#fJ3",
          "start_offset" : 0,
          "end_offset" : 16,
          "type" : "word",
          "position" : 0
        }
      ]
    }


```

### 1.2 term filter 示例

接着，我们来看下如何使用 Term Filter。首先看一个示例：根据帖子 ID 搜索帖子。由于用了`term filter`语法，所以不会对搜索关键字进行分词：

```
    
    GET /forum/_search
    {
        "query" : {
            "constant_score" : { 
                "filter" : {
                    "term" : { 
                        "articleID" : "XHDK-A-1293-#fJ3"
                    }
                }
            }
        }
    }
    
    {
      "took" : 5,
      "timed_out" : false,
      "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : {
          "value" : 0,
          "relation" : "eq"
        },
        "max_score" : null,
        "hits" : [ ]
      }
    }


```

我们直接对 articleID 进行 term filter 是查不出结果的，需要使用 articleID.keyword：

```
    
    GET /forum/_search
    {
        "query" : {
            "constant_score" : { 
                "filter" : {
                    "term" : { 
                        "articleID.keyword" : "XHDK-A-1293-#fJ3"
                    }
                }
            }
        }
    }
    
    {
      "took" : 0,
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
        "max_score" : 1.0,
        "hits" : [
          {
            "_index" : "forum",
            "_type" : "article",
            "_id" : "1",
            "_score" : 1.0,
            "_source" : {
              "articleID" : "XHDK-A-1293-#fJ3",
              "userID" : 1,
              "hidden" : false,
              "postDate" : "2017-01-01"
            }
          }
        ]
      }
    }


```

二、Filter 底层原理
-------------

### 2.1 匹配 document

term filter 在执行时，首先会在倒排索引中匹配 filter 条件，也就是我们的搜索关键字，用于获取 document list。我们以`postDatge`来举个例子，如果查找 “2017-02-02”，会发现”2017-02-02“对应的 document list 是 doc2、doc3：

<table><thead><tr><th>word</th><th>doc1</th><th>doc2</th><th>doc3</th></tr></thead><tbody><tr><td>2017-01-01</td><td>Y</td><td>Y</td><td>N</td></tr><tr><td>2017-02-02</td><td>N</td><td>Y</td><td>Y</td></tr><tr><td>2017-03-03</td><td>Y</td><td>Y</td><td>Y</td></tr></tbody></table>

### 2.2 构建 bitset

接着，为每个 fiter 条件构建一个 bitset。bitset 就是一个二进制的数组，数组每个元素都是 0 或 1，用来标识某个 doc 对这个 filter 条件是否匹配，匹配就是 1，否则为 0。比如，doc1 不匹配 "2017-02-02"，而 doc2 和 do3 是匹配的，所以 "2017-02-02" 这个 filter 条件的 bitset 就是 [0, 1, 1]。

### 2.3 遍历 bitset

由于在一个 search 请求中，可以有多个 filter 条件，而 filter 条件都会对应一个 bitset。所以这一步，ES 会从最稀疏的 bitset 开始遍历，优先过滤掉尽可能多的数据。比如我们的 filter 条件是 postDate=2017-01-01，userID=1，对应的 bitset 是：  
postDate: [0, 0, 1, 1, 0, 0]  
userID: [0, 1, 0, 1, 0, 1]

那么遍历完两个 bitset 之后，找到匹配所有 filter 条件只有 doc4，就将其作为结果返回给 client 了。

### 2.4 缓存 bitset

Elasticsearch 会将一些频繁访问的 filter 条件和它对应的 bitset 缓存在内存中，这样就可以提高检索效率了。

注意，如果 document 保存在某个很小的 segment 上的话（segment 记录数 < 1000，或 segment 大小 < index 总大小的 3%），Elasticsearch 就不会对其缓存。因为 segment 很小的话，会在后台被自动合并，那么缓存也没有什么意义了，因问 segment 很快就消失了。

> 这里就可以看出，filter 为什么比 query 的性能更好了，filter 除了不需要计算相关度分数并按其排序外，filter 还会缓存检索结果对应的 bitset。

### 2.5 bitset 更新

如果 document 有新增或修改，那么 filter 条件对应的 cached bitset 会被自动更新。举个例子，假设 postDate=2017-01-01 对应的 bitset 为 [0, 0, 1, 0]。

*   如果新增一条 doc5：id=5，postDate=2017-01-01，那 postDate=2017-01-01 这个 filter 的 bitset 会全自动更新成 [0, 0, 1, 0, 1]；
*   如果修改 doc1：id=1，postDate=2016-12-30，那 postDate=2016-01-01 这个 filter 的 bitset 会全自动更新成 [1, 0, 1, 0, 1]；

三、总结
----

本章，我介绍了 term filter 的基本使用和底层的 bitset 原理。关于 term filter 的更多 API，读者可以参考官方文档，本系统的目的是介绍 Elasticsearch 中各种检索功能的基本使用，重点关注的是底层的实现原理。
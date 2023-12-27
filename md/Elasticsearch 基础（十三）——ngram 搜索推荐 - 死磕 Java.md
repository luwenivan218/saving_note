> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1100787424)

> 上一章，我们讲近似匹配时，提到如果要实现搜索推荐功能，最好不要用 match_phrase_prefix 进行实时的前缀匹配，因为这样性能很差。本章，我们就来介绍下 ngram 分词机

转

 2023-08-08  阅读 (261)  点赞 (0)

原文地址：https://www.tpvlog.com/article/146

上一章，我们讲近似匹配时，提到如果要实现搜索推荐功能，最好不要用`match_phrase_prefix`进行实时的前缀匹配，因为这样性能很差。本章，我们就来介绍下 ngram 分词机制，通过它，我们可以在建立索引阶段就完成 “搜索推荐”。

一、ngram 机制
----------

什么是 ngram？_N-Gram_ 是大词汇连续语音识别中常用的一种语言模型。

举个例子，对于单词 “quick”，我们可以对其做如下拆分，那么，“quick” 这个 term 就被拆分成了 5 种长度下的 ngram，每种长度下的拆分项都是一个 ngram：

```
    
    q u i c k
    
    qu ui ic ck
    
    qui uic ick
    
    quic uick
    
    quick


```

Elasticsearch 使用了一种 “edge ngram” 的分词方法，比如我们有两个下面这样的 document：

```
    
    hello world
    
    hello what


```

Elasticsearch 会对文本中的每个 term，按照 edge ngram 机制建立倒排索引：

<table><thead><tr><th>term</th><th>doc1</th><th>doc2</th></tr></thead><tbody><tr><td>h</td><td>Y</td><td>Y</td></tr><tr><td>he</td><td>Y</td><td>Y</td></tr><tr><td>hel</td><td>Y</td><td>Y</td></tr><tr><td>hell</td><td>Y</td><td>Y</td></tr><tr><td>hello</td><td>Y</td><td>Y</td></tr><tr><td>w</td><td>Y</td><td>Y</td></tr><tr><td>wo</td><td>Y</td><td>N</td></tr><tr><td>wor</td><td>Y</td><td>N</td></tr><tr><td>worl</td><td>Y</td><td>N</td></tr><tr><td>world</td><td>Y</td><td>N</td></tr><tr><td>wh</td><td>N</td><td>Y</td></tr><tr><td>wha</td><td>N</td><td>Y</td></tr><tr><td>what</td><td>N</td><td>Y</td></tr></tbody></table>

当我们检索 “hello w” 时，首先会对 “hello” 这个 term 检索，发现 doc1 和 doc2 都有，然后对 “w” 这个 term 检索，发现 doc1 和 doc2 也都有，所以 doc1 和 duc2 都会被返回，这样就实现了搜索推荐。

由于检索时，完全利用到了倒排索引，并没有去做前缀匹配，所以 ngram 机制实现的搜素推荐效率非常高。

二、使用示例
------

接着，我们来看看如何使用 ngram 进行分词。首先，建立索引，min_gram 和 max_gram 用于控制 ngram 的长度：

```
    PUT /my_index
    {
        "settings": {
            "analysis": {
                "filter": {
                    "autocomplete_filter": { 
                        "type":     "edge_ngram",
                        "min_gram": 1,
                        "max_gram": 20
                    }
                },
                "analyzer": {
                    "autocomplete": {
                        "type":      "custom",
                        "tokenizer": "standard",
                        "filter": [
                            "lowercase",
                            "autocomplete_filter" 
                        ]
                    }
                }
            }
        }
    }


```

我们可以通过以下命令查看下分词结果：

```
    GET /my_index/_analyze
    {
      "analyzer": "autocomplete",
      "text": "quick brown"
    }


```

最后，只要对那些想要实现搜索推荐的字段，修改其字段使用的分词器就完成了：

```
    PUT /my_index/_mapping
    {
      "properties": {
          "title": {
              "type":     "string",
              "analyzer": "autocomplete",
              "search_analyzer": "standard"
          }
      }
    }


```

> 上面`analyzer`的意思是对 title 字段的内容建立索引时，使用 autocomplete 这个分词器，也就是 ngram 分词；`search_analyzer`的意思是，对于我们的检索词，比如 “hello w”，还是用标准的 standard 分词器拆分。

三、总结
----

本章，我讲解了如何使用 Elasticsearch 实现 index-time 搜索推荐。
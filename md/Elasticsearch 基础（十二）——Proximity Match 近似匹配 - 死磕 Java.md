> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1098019901)

> ProximityMatch（近似匹配），包含两种类型： phrasematch  和 & nbsp;proximitymatch 。什么情况下需要使用

转

 2023-08-08  阅读 (144)  点赞 (0)

原文地址：https://www.tpvlog.com/article/146

Proximity Match（近似匹配），包含两种类型： **phrase match** 和 **proximity match** 。什么情况下需要使用近似匹配？我们先看下下面的检索需求：

```
    # doc1
    java is my favourite programming language, and I also think spark is a very good big data system.
    # doc2
    java spark are very related, because scala is spark


```

假设我们有上述两条 document，希望检索出包含 “java spark” 关键字的 doc，但是必须满足以下任一条件：

*   java spark，就靠在一起，中间不能插入任何其他字符；
*   java 和 spark 两个单词靠的越近，doc 的分数越高，排名越靠前。

思考一下，如果使用全文检索中的 math query，能否满足我们的需求？答案是不能，此时就要用到近似匹配了。

一、phrase match
--------------

Phrase Match（短语匹配），就是将搜索词中的多个 term 作为一个短语，一起去搜索，只有包含这个短语的 doc 才会作为结果返回，不像是 match query，任何一个 term 匹配就会返回结果。

Phrase Match 的基本语法如下，只有包含 “java spark” 这个短语的 doc 才会被返回，对于本章开头的那两条 doc，只有 doc2 满足条件：

```
    GET /forum/_search
    {
        "query": {
            "match_phrase": {
                "title": {
                    "query": "java spark",
                    "slop":  1
                }
            }
        }
    }


```

### 1.1 基本原理

Elasticsearch 在建立倒排索引时，会记录每个 term 在文本内容中的位置。比如我们有下面两条 doc：

```
    doc1: hello world, java spark        
    doc2: hi, spark java


```

建立完的倒排索引包含以下内容，hello 这个 term 出现在 doc1 的 position 0，以此类推：

<table><thead><tr><th>Term</th><th>Doc1 中的位置</th><th>Doc2 中的位置</th></tr></thead><tbody><tr><td>hello</td><td>doc1(0)</td><td>N</td></tr><tr><td>wolrd</td><td>doc1(1)</td><td>N</td></tr><tr><td>java</td><td>doc1(2)</td><td>doc2(2)</td></tr><tr><td>spark</td><td>doc1(3)</td><td>doc2(1)</td></tr></tbody></table>

当使用 Phrase Match（短语匹配）时，步骤如下：

1.  Elasticsearch 会首先对短语分词，"java spark" 拆分为 "java" 和 "spark" ；
2.  筛选出 “java” 和“spark”都存在的 doc，也就是说 doc 必须包含短语中的所有 term，那 doc1 和 doc2 都满足；
3.  后一个短语的 position 必须比前一个大 1，即 “spark” 的 position 要比 java 的 position 大 1，那只有 doc1 满足条件。

### 1.2 slop 参数

Phrase Match 有一个很重要的参数——slop，表示短语中的 term，最多经过几次移动才能与一个 document 匹配，这个移动次数，就是 slop。

举个例子，假如有下面这样一条 doc，搜索的短语是 "spark data"：

```
    spark is best big data solution based on scala.


```

那么 slop=3 时，就可以匹配到，可以看下面的移动步骤：

```
    spark         is         best         big         data...
    spark        data
                     -->data
                                 -->data
                                              -->data


```

注意，移动的方向可以是双向的，比如搜索的短语是 "data spark"，那么 slop=5 时，也可以匹配到：

```
    spark         is         best         big         data...
    data        spark
    spark  <-->    data
    spark             -->data
    spark                          -->data
    spark                                       -->data


```

> 当使用 Phrase Match 时，term 靠的越近，相关度分数会越高。

### 1.3 rescore

rescore(重打分），通常与 match query 配合使用。比如，我们有以下请求：

```
    GET /forum/_search 
    {
      "query": {
        "match": {
          "content": "java spark"
        }
      },
      "rescore": {
        "window_size": 50,
        "query": {
          "rescore_query": {
            "match_phrase": {
              "content": {
                "query": "java spark",
                "slop": 50
              }
            }
          }
        }
      }
    }


```

使用`rescore`参数后，会对 math query 匹配到的结果重新打分，上述`window_size`表示取前 50 条记录进行打分，配合 phrase match 可以使 term 越接近的短语分数更高，从而既提供了精确度，又提升了召回率。

> 所谓召回率，就是进行检索时，返回的 document 的数量，数量越多，召回率越高。近似匹配通常会和 match query 搭配使用以提升召回率。

### 1.4 搜索推荐

我们还可以利用`match_phrase_prefix`进行搜索推荐。搜索推荐的原理跟`match_phrase`类似，唯一的区别就是把检索词中的最后一个 term 作为前缀去搜索。

```
    GET /forum/_search 
    {
      "query": {
        "match_phrase_prefix": {
          "content": {
              "query": "java s",
              "slop": 10,
              "max_expansions": 5
          }
        }
      }
    }


```

默认情况下，前缀要扫描所有的倒排索引中的 term，去查找 "s" 打头的单词，但是这样性能太差，所以可以用`max_expansions`限定，"s" 前缀最多匹配多少个 term，就不再继续搜索倒排索引了。

> 一般来讲，不推荐使用`match_phrase_prefix`来实现搜索推荐，因为 Elasticsearch 会在扫描倒排索引时实时进行前缀匹配，性能很差。如果要实现搜索推荐功能，建议使用 [ngram 分词机制](https://www.tpvlog.com/article/159)。

二、proximity match
-----------------

讲解完了 phrase match，proximity match 其实就没什么可讲了，底层原理都是一样的，proximity match 可以看成是加了`slop`参数的 phrase match。

三、总结
----

本章，主要介绍了 Elasticsearch 中的近似匹配搜索，近似匹配搜索分为 **phrase match** 和 **proximity match** ，底层的原理是一样的，都是通过倒排索引去匹配搜索关键词中的各个 term 的位置。
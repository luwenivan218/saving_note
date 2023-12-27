> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1102635397)

> 在使用 Elasticsearch 检索的过程中，结果都会包含一个相关度分数（relevancescore），本章我们就来看看 Elasticsearch 到底是如何计算这个分数的。E

在使用 Elasticsearch 检索的过程中，结果都会包含一个相关度分数（relevance score），本章我们就来看看 Elasticsearch 到底是如何计算这个分数的。

Elasticsearch 进行相关度分数计算时，核心步骤就包含三点：

1.  利用 boolean model 进行 document 过滤；
2.  利用 TF/IDF 算法计算单个 term 的分数；
3.  利用 vector space model 整合最终的相关度分数。

一、bool model
------------

所谓 bool model，就是 Elasticsearch 中的各种 doc 过滤语法，比如 bool 命令中的`must`/`should`/`must not`等等。核心目的就是过滤出包含检索关键字的 document，提升后续分数计算的性能。

这一过程仅仅是过滤，不进行分数计算。

二、TF/IDF 算法
-----------

TF/IDF（term frequency/inverse document frequency）算法，是 Elasticsearch 计算相关度分数的基础， **用于计算单个 term 在 document 中的分数** 。

_**单个 term 是什么意思？**_

比如，我们的检索关键字是”hello world“，index 中包含如下的 document：

```
    
    hello you, and world is very good


```

那么 TF/IDF 算法会对 "hello" 这个 term 计算出一个 doc1 分数，对”world“再计算出一个 doc1 分数。至于”hello world“这整个关键字在 doc1 中的综合分数，TF/IDF 是不管的。

TF/IDF 算法的核心思想就三点：

*   **term frequency：** 表示检索的 term，在单个 document 中的各个词条中出现的频次，出现的次数越多，该 document 相关度越高；
*   **inverse document frequency：** 表示检索的 term，在该索引的所有 document 中出现的频次，出现的次数越少，包含该 term 的 document 相关度越高；
*   **Field-length norm：** 表示 document 的 field 内容长度越短，相关度越高。

### 2.1 term frequency

还是通过示例来理解，假设我们有一个搜索请求，关键字是 “hello world”，索引中包含下面两条 document：

```
    
    hello you, and world is very good
    
    hello, how are you


```

“hello world”进行分词后，拆成两个 term——“hello”和 “world”，显然 doc1 既包含“hello” 又包含“world”，doc2 只包含“hello”，所以 doc1 更相关。

### 2.2 inverse document frequency

再来看下 inverse document frequency，检索请求还是 “hello world”，假设索引中一共包含 1000 条 document，我们只列出其中两条：

```
    
    hello, today is very good
    
    hi world, how are you


```

如果根据 term frequency 规则，doc1 和 doc2 的相关度应该是相同的，但是如果‘hello“在该索引的 1000 条 document 中，有 800 条 document 都包含它，而‘world“只有 200 条 document 包含，那么 doc2 的相关度就比 doc1 更高。

> 这里好好思考下，为什么 term 在整个 document 列表中出现的次数越多，包含它的 doc 相关度反而越低。因为出现的次数越少，说明包含那个 term 的 doc 的区分度越高。

### 2.3 Field-length norm

举个例子，检索请求还是 “hello world”，假设索引中一共包含 2 条 document：

```
    
    { "title": "hello article", "content": "babaaba 1万个单词" }
    
    { "title": "my article", "content": "blablabala 1万个单词，hi world" }


```

doc1 中，”hello“出现在 title 字段，doc2 中，”world“出现在 content 字段，显然 title 字段的内容长度远小于 content 字段的内容长度，所以 doc1 的相关度比 doc2 更高。

### 2.4 示例

我们可以通过`explain`命令，查看 Elasticsearch 对某个 query 的评分到底是如何计算的：

```
    GET /test_index/_search?explain
    {
      "query": {
        "match": {
          "test_field": "test hello"
        }
      }
    }


```

也可以通过如下命令查看某个 document 是如何被一个 query 匹配上，比如下面是查看 id 为 6 的 document 是如何被匹配上的：

```
    GET /test_index/6/_explain
    {
      "query": {
        "match": {
          "test_field": "test hello"
        }
      }
    }


```

三、向量空间模型算法
----------

TF/IDF 算法只能计算单个 term 在 document 中的分数。那么， **如何计算整个搜索关键词在各个 doc 中的综合分数呢** ？这就要靠 vector space model 了。

vector space model 的核心思想其实就是计算两个向量，然后相乘得到最终分：

1.  每个 term 在所有 document 的分数——query vector；
2.  每个 term 在各个 document 的分数——doc vector；
3.  计算 doc vector 对于 query vector 的弧度（其实就是线性代数中的向量运算）。

### 3.1 query vector

vector space model 算法，会首先根据 TF/IDF 算法的结果，计算出一个 **query vector** ，这个 query vector 就是每一个 term 对所有 document 的综合评分。

举个例子，假设 index 中包含 3 条 document，搜索关键字是”hello world“：

```
    
    hello, today is very good
    
    hi world, how are you
    
    hello world


```

那么，对于 hello 这个 term，vector space model 算法会算出它对所有 doc 的评分，比如等于 2；world 这个 term，基于所有 doc 的评分是 5，那么`query vector = [2, 5]`。

> query vector 的计算过程不用去深究，底层涉及线性代数之类的高等数学知识，我们只要知道 vector space model 会计算出这样一个 vector，vector 包含了每个 term 对所有 document 的评分就行了。

### 3.2 doc vector

doc vector，就是每个 term 在各个 document 中的分数组成的一个向量。

比如，”hello“在 doc1 中的分数是 2，doc2 中是 0，doc3 中是 2；"world" 在 doc1 中的分数是 0，doc2 中是 5，doc3 中是 5，那么最终计算出的 doc vector 是下面这样的：

```
    [2 , 0]
    [0 , 5]
    [2 , 5]


```

### 3.3 弧度计算

所谓弧度计算，就是根据 doc vector 和 query vector 进行向量运算，最终得到每个 doc 对多个 term 的总分数。涉及大量数学知识，此处就不再展开了。

四、Lucene 的相关度分数函数
-----------------

最后，我们来看下 Lucene 计算相关度分数的算法，Lucene 中有一个叫做`practical scoring`的函数，综合了上面我们讲的 TF/IDF 算法和 vector space model：

```
    score(q,d)  =  
                queryNorm(q)  
              · coord(q,d)    
              · ∑ (           
                    tf(t in d)   
                  · idf(t)2      
                  · t.getBoost() 
                  · norm(t,d)    
                ) (t in q)


```

这个公式的最终结果，就是一个 query（入参 q），对一个 doc（入参 d）的最终总评分，也就是搜索关键字对某个 document 的相关度分数：

*   **queryNorm(q)：** 用来让一个 doc 的分数处于一个合理的区间内，不要太离谱；
*   **coord(q,d)：** 对更加匹配的 doc，进行一些分数上的成倍奖励；
*   **tf(t in d)：** 计算每个 term 对 doc 的分数，就是 TF/IDF 算法中的 term frequency 步骤；
*   **idf(t)2：** 计算每个 term 对 doc 的分数，就是 TF/IDF 算法中的 inverse document frequency 步骤；
*   **t.getBoost()：** 计入字段权重；
*   **norm(t,d)：** 计算每个 term 对 doc 的分数，就是 TF/IDF 算法中的 Field-length norm 步骤。

五、总结
----

本章，我讲解了 Elasticsearch 进行相关度分数计算的核心原理，相关度分数算法基于 TF/IDF 和 vector space model 实现。Elasticsearch 的底层其实调用了 Lucene 的`practical scoring`函数来完成分数的计算。
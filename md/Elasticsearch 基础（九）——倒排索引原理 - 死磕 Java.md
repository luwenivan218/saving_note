> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1071233850)

> 我们在第一章中简单介绍过倒排索引，本章我们来看下倒排索引的底层原理。先来回顾下什么是倒排索引，假设我们向某个索引里写入了下面两条 document：document 某字段内容 do

转

 2023-08-08  阅读 (164)  点赞 (0)

原文地址：https://www.tpvlog.com/article/146

我们在第一章中简单介绍过倒排索引，本章我们来看下倒排索引的底层原理。先来回顾下什么是倒排索引，假设我们向某个索引里写入了下面两条 document：

<table><thead><tr><th>document</th><th>某字段内容</th></tr></thead><tbody><tr><td>doc1</td><td>Ireallylikedmysmalldogs,andIthinkmymomalsolikedthem.</td></tr><tr><td>doc2</td><td>Heneverlikedanydogs,soIhopethatmymomwillnotexpectmetolikedhim.</td></tr></tbody></table>

Elasticsearch 会对 document 的字段内容进行分词，然后构成倒排索引，比如可能是下面这个样子：

<table><thead><tr><th>word</th><th>doc1</th><th>doc2</th></tr></thead><tbody><tr><td>I</td><td>Y</td><td>Y</td></tr><tr><td>really</td><td>N</td><td>Y</td></tr><tr><td>liked</td><td>Y</td><td>Y</td></tr><tr><td>省略其它分词....</td><td></td><td></td></tr></tbody></table>

> 解释一下，Y 表示这个 word 存在于 document 中，N 表示不存在。

然后，当客户端进行搜素时，Elasticsearch 也会对搜索关键字进行分词，比如关键字是 “I liked her”，那么就会拆分成`I`、`liked`、`her`，这样 Elasticsearch 就能快速根据分词结果找到对应的 document，doc1 和 doc2 中都包含`I`和`liked`，就会都被检索出来。

> 即使输入 “I like her” 也能被检索出来，这跟分词器的行为有关。

一、分词器
-----

建立倒排索引最关键的部分就是 **分词器** 。分词器会对文本内容进行一些特定处理，然后根据处理后的结果再建立倒排索引，主要的处理过程一般如下：

1.  **character filter：** 符号过滤，比如`<span>hello<span>`过滤成`hello`，`I&you`过滤成`I and you` ；
2.  **tokenizer：** 分词，比如，将`hello you and me`切分成`hello`、`you`、`and`、`me`；
3.  **token filter：** 比如，`dogs`替换为`dog`，`liked`替换为`like`，`Tom` 替换为 `tom`，`small` 替换为 `little`等等。

不同分词器的行为是不同的，Elasticsearch 主要内置了以下几种分词器： **standard analyzer** 、 **simple analyzer** 、 **whitespace analyzer** 、 **language analyzer** 。

我们可以通过以下命令，看下分词器的分词效果：

```
    GET /{index}/_analyze
    {
      "analyzer": "standard", 
      "text": "a dog is in the house"
    }
    


```

> 分词器的各种替换行为，也叫做 _**normalization**_ ，本质是为了提升命中率，官方叫做 recall 召回率。

对于 document 中的不同字段类型，Elasticsearch 会采用不同的分词器进行处理，比如 date 类型压根就不会分词，检索时就是完全匹配，而对于 text 类型则会进行分词处理。

> Elasticsearch 通过`_mapping`元数据来定义不同字段类型的建立索引的行为，这块内容官方文档已经写得非常清楚了，我不再赘述。

### 1.1 定制分词器

我们可以修改分词器的默认行为。比如，我们修改`my_index`索引的分词器，启用 english 停用词：

```
    PUT /my_index
    {
      "settings": {
        "analysis": {
          "analyzer": {
            "es_std": {
              "type": "standard",
              "stopwords": "_english_"
            }
          }
        }
      }
    }


```

然后可以通过以下命令，查看分词器的分词效果：

```
    GET /my_index/_analyze
    {
      "analyzer": "es_std", 
      "text": "a dog is in the house"
    }


```

也可以完全定制自己的分词器，更多分词器的用法读者可以参考 Elasticsearch 官方文档：

```
    PUT /my_index
    {
      "settings": {
        "analysis": {
          "char_filter": {
            "&_to_and": {
              "type": "mapping",
              "mappings": ["&=> and"]
            }
          },
          "filter": {
            "my_stopwords": {
              "type": "stop",
              "stopwords": ["the", "a"]
            }
          },
          "analyzer": {
            "my_analyzer": {
              "type": "custom",
              "char_filter": ["html_strip", "&_to_and"],
              "tokenizer": "standard",
              "filter": ["lowercase", "my_stopwords"]
            }
          }
        }
      }
    }


```

二、总结
----

在 Elasticsearch 中建立的索引时，一旦建立完成，索引就不可变，主要是出于性能考虑。关于倒排索引，最核心的一些东西就是上述文章所示的，更多内容建议读者自己去看官方文档。Elasticsearch 的使用大多都是些 API 的调来调去，核心的东西其实就那么点。
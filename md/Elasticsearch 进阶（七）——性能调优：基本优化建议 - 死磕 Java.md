> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/3440825813)

> 本章，我将介绍对 Elasticsearch 进行性能优化的一些最基本的建议。一、优化建议 1.1 不要返回过大的结果集 Elasticsearch 是一个搜索引擎，所以如果用这个搜索引擎

转

 2023-08-08  阅读 (234)  点赞 (0)

原文地址：https://www.tpvlog.com/article/170

本章，我将介绍对 Elasticsearch 进行性能优化的一些最基本的建议。

一、优化建议
------

### 1.1 不要返回过大的结果集

Elasticsearch 是一个搜索引擎，所以如果用这个搜索引擎对大量的数据进行搜索，并且返回搜索结果中排在最前面的少数结果，是非常合适的。然而，如果要做成类似数据库的东西，每次都进行大批量的查询，是很不合适的。如果真的要做大批量结果的查询，记得考虑用 _scroll api_。

### 1.2 避免超大的 document

Elasticsearch 有一个参数：`http.max_context_length`，表示 document 的最大内容大小，默认值是 100MB。

超大的 document 在实际生产环境中是很不好的，会耗费更多的网络、内存、磁盘资源。对于超大 document，我们需要考虑一下，我们到底需要其中的哪些部分。

举例来说，如果我们要对一些书进行搜索，那么我们并不需要将整本书的内容就放入 es 中吧。我们可以仅仅使用每一篇章或者一个段落作为一个 document，然后给一个 field 标识出来这些 document 属于哪本书，这样每个 document 的大小不就变小了么。这就可以避免超大 document 导致的各种开销，同时可以优化搜索的体验。

### 1.3 避免稀疏数据

Lucene 的内核结构，跟稠密的数据配合起来性能会更好。什么叫稀疏的数据，什么叫稠密的数据？

比如有 100 个 document，每个 document 都有 20 个 field，20 个 field 都有值，这就是稠密的数据。但是如果 100 个 document，每个 document 的 field 都不一样，有的 document 有 2 个 field，有的 document 有 50 个 field，这就是稀疏的数据。

下面有一些避免稀疏数据的办法：

**避免将没有任何关联性的数据写入同一个索引**

我们必须避免将结构完全不一样的数据写入同一个索引中，因为结构完全不一样的数据，field 是完全不一样的，会导致 index 数据非常稀疏。最好将这种数据写入不同的索引中，如果这种索引数据量比较少，可以考虑给其很少的 primary shard，比如 1 个，避免资源浪费。

**对 document 的结构进行规范化 / 标准化**

即使我们真的要将不同类型的 document 写入相同的索引中，还是有办法可以避免稀疏性，那就是对不同类型的 document 进行标准化。比如说，如果所有的 document 都有一个时间戳 field，不过有的叫做 timestamp，有的叫做 creation_date，那么可以将不同 document 的这个 field 重命名为相同的字段，尽量让 documment 的结构相同。另外比如有的 document 有一个字段，叫做 goods_type，但是有的 document 没有这个字段，此时可以对没有这个字段的 document，补充一个 goods_type 给一个默认值，比如 default。

**对稀疏的 field 禁用 norms 和 doc_values**

如果上面的步骤都没法做，那么只能对那种稀疏的 field，禁止 norms 和 doc_values 字段，因为这两个字段的存储机制类似，都是每个 field 有一个全量的存储，对存储浪费很大。如果一个 field 不需要考虑其相关度分数，那么可以禁用 norms，如果不需要对一个 field 进行排序或者聚合，那么可以禁用 doc_values 字段。

二、总结
----

本章，我介绍了一些 Elasticsearch 性能优化的基本建议，下一章我们来看下 Elasticsearch 写入数据时有哪些需要注意的点。
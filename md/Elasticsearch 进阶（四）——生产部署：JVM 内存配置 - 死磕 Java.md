> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/3182250408)

> 本章，我们来介绍下 Elasticsearch 的 JVM 和服务器内存分配。前面的章节，我其实已经讲过，Elasticsearch 不建议我们去动 JVM 相关的默认配置。还有线程池数量的

转

 2023-08-08  阅读 (495)  点赞 (0)

原文地址：https://www.tpvlog.com/article/170

本章，我们来介绍下 Elasticsearch 的 JVM 和服务器内存分配。前面的章节，我其实已经讲过，Elasticsearch 不建议我们去动 JVM 相关的默认配置。还有线程池数量的配置，很多人都喜欢去调优线程池，但是在 Elasticsearch 中，默认的 threadpool 设置是非常合理的，对于所有的 threadpool 来说，除了搜索的线程池，都是线程数量设置得跟 cpu core 一样多。

所以，我们不应该去修改 Elasticsearch 的这些默认配置。但是，在生产环境部署 ES，不可避免要回答一个问题：我的机器上有 64G 的内存，或者 32G 的内存，那么一般来说我应该分配多少内存给 Elasticsearch 的 jvm heap 呢？

一、JVM 堆内存分配
-----------

Elasticsearch 默认会给 jvm heap 分配 2 个 G 的大小，生产环境肯定不能用这个配置，要重新设置。一般可以通过`jvm.options`这个配置文件进行设置。

### 1.1 分配建议

Elasticsearch 官方给出的建议是，将机器 50% 的内存分配给 jvm heap，然后留 50% 的内存给 os cache。留给 os cache 的内存会被 Lucene 全部用光，用来缓存 segment file。

如果我们不需要对任何分词 field 进行聚合操作，那就不需要使用 fielddata，此时可以考虑给 os cache 更多的内存，因为 fielddata 是要用 jvm heap。如果我们给 jvm heap 更少的内存，那么实际上 es 的性能反而会更好，因为更多的内存留给了 lucene 用 os cache 提升索引读写性能。

### 1.2 控制堆内存大小

一般来讲， **给 Elasticsearch 的 heap 内存大小最好不要超过 32G** 。因为只有当 heap 内存小于 32G 时，JVM 才会用一种 _compressed oops_ 技术来压缩对象指针（object pointer），解决 object pointer 耗费过大空间的问题。

> object pointer 耗费过大空间：object pointer 指向 heap 中的对象，引用的是二进制格式的地址。对于 64 位的系统来说，当堆内存很大时，64 位的 object pointer 会耗费更多的空间，因为 object pointer 更大了。同时，过大的 object pointer 会在 CPU，main memory 和 LLC、L1 等多级缓存间移动数据的时候，消耗更多的带宽。

实际上，不用 compressed oops 时，你如果给 jvm heap 分配了一个 40~50G 的内存的可用空间，实际上被 object pointer 可能都要占据十几 G 的内存空间，可用的空间量，可能跟使用了 compressed oops 时的 32GB 内存的可用空间，20 多个 G，几乎是一样的。

因此，即使我们有很多内存，但是还是要分配给 heap 在 32GB 以内，否则的话浪费更多的内存，降低 cpu 性能。

综上所述，如果你的 Elasticsearch 要处理的数据量是十亿级以内的话，建议就是用 64G 内存的机器比较合适，差不多 5 台就够了，给 jvm heap 分配 32G，留下 32G 给 os cache。

### 1.3 超大内存机器

如果我们的机器就是一台超级服务器，内存资源甚至达到了 1TB，或者 512G，128G，该怎么办？

首先，Elasticsearch 官方是建议避免用这种超级服务器来部署 ES 集群的，但是如果我们只有这种机器可以用的话，我们要考虑以下几点：

1.  是否需要做大量的全文检索？如果是的话，可以分配 32G 的内存给 ES 进程，同时给 Lucene 留下其余所有的内存，这些剩余内存都会用来缓存 segment file，可以提供非常高性能的搜索，几乎所有的数据都是可以在内存中缓存的，ES 集群的性能会非常高。
2.  是否需要做大量的排序或聚合操作？且聚合操作针对数字、日期等不分词字段？如果是的话，那么还是给 ES32G 的内存即可，其它的留给 os cache。
3.  是否需要针对分词的字段做大量的排序或聚合操作？如果是的话，那么就需要使用 fielddata，这就得给 jvm heap 分配更大的内存了。此时建议运行多个节点在一台机器上。

二、总结
----

本章，我介绍了 Elasticsearch 集群部署过程中的 JVM 堆内存分配的最佳实践。官方建议是 JVM 堆内存和 filesystem os cache 各占一半总内存。
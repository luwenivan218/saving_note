> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1068470138)

> 一、SegmentFile 本章，我们来看下 Elasticsearch 的持久化原理，也就是数据在底层究竟是如何写入到磁盘上的。开始之前，先来了解一个基本概念—— Seg

转

 2023-08-08  阅读 (244)  点赞 (0)

原文地址：https://www.tpvlog.com/article/146

一、Segment File
--------------

本章，我们来看下 Elasticsearch 的持久化原理，也就是数据在底层究竟是如何写入到磁盘上的。开始之前，先来了解一个基本概念—— **Segment** 。

我们知道，在 Elasticsearch 中创建一个 index 索引时，需要指定 shard 分片，一个 shard 分片在底层其实是一个 Lucene 索引，它 **由若干个 segment 文件和对应的`commit point`（提交点文件）构成** 。

![](http://image.skjava.com/article/series/elasticsearch/202308082147076821.png)

segment 文件可以理解成底层存储 document 数据的文件，ES 进行检索时最终是从它里面检索出数据的；而`commit point`文件可以理解成一个保存了若干 segment 信息的列表，它标识着这个 commit point 前所有旧的 segment file 文件。

举个例子，当 Elasticsearch 创建`commit point`文件时，会已有一些 segment 文件是已经存在的，那`commit point`就保存着这些旧的 segment 文件的信息：

![](http://image.skjava.com/article/series/elasticsearch/202308082147082182.png)

二、写入流程
------

一条 document 数据被写入磁盘会经历以下几个过程：

### 1.1 write

首先，document 会被写入 **in-memory buffer** 中，所谓 **in-memory buffer** 其实就是应用内存。同时，document 数据会被写入 **translog** 日志文件。

![](http://image.skjava.com/article/series/elasticsearch/202308082147089563.png)

### 1.2 refresh

每隔 1 秒，Elasticsearch 会执行一次 _**refresh**_ 操作：将 buffer 中的数据 refresh 到 filesystem cache 中的一个新 segment file 中。filesystem cache 其实就是 os cache，性能非常好。注意，此时的 segment file 仅仅是存在于 os cache 中的缓存数据，并不存在于磁盘上。

![](http://image.skjava.com/article/series/elasticsearch/202308082147096414.png)

refresh 操作完成后，buffer 会被清空。另外，如果 buffer 满了也会执行 refresh。 **当数据进入 filesystem cache 后，其实就可以被检索到了** 。

> 为什么说 Elasticsearch 是准实时（NRT，near real-time）的？因为默认每隔 1 秒 refresh 一次，所以写入的数据 1 秒之后才能被检索到。可以通过 index 的`index.refresh_interval`参数配置 refresh 的时间间隔。

### 1.3 flush

上述过程中，segment file 一直存在于 os cache 中，如果发生宕机，cache 中的数据就会丢失。所以需要有一种机制能将 os cache 中的数据写入磁盘文件。这一过程在 Elasticsearch 中就叫做 _**flush**_ 。

在 refresh 的过程中，os cache 中的 segment file 会越来越多（每次 refresh 都会创建一个新的 segment file）， **translog** 日志文件也会越来越大。当 translog 大到一定程度的时候，就会触发 **flush** 操作：

1.  flush 的第一步，将 buffer 中的现有数据 refresh 到 os cache 中，清空 buffer；
2.  在磁盘上写入一个 commit point 文件，里面标识着这个 commit point 前 os cache 中的所有旧 segment file 文件数据；
3.  强行将 os cache 中的所有数据`fsync`到磁盘文件中去；
4.  最后将 translog 清空，因为此时里面记录的数据此时都已经被 fsync 到磁盘了。

![](http://image.skjava.com/article/series/elasticsearch/202308082147108135.png)

> 默认每隔 30 分钟会自动执行一次 **flush** 操作，但是如果 translog 过大，也会提前触发。

_**这里来思考下，translog 日志文件的作用是什么？**_

在执行 flush 操作之前，数据要么是停留在 buffer 中，要么是停留在 os cache 中，此时一旦这台机器宕机，数据就全丢了。所以，需要将数据写入到一个专门的日志文件中，此时即使机器宕机了，再次重启时，Elasticsearch 会自动读取 translog 日志文件中的数据，恢复到内存 buffer 和 os cache 中去。

另外，translog 中的数据，本身也是先写入 os cache，然后默认每隔 5 秒刷一次到磁盘中。所以默认情况下，即使有 translog 保证可用性，也可能丢失 5s 的数据。此时数据仅仅停留在 buffer 或者 translog 文件的 os cache 中，如果此时机器挂了，会丢失这 5 秒钟的数据。

![](http://image.skjava.com/article/series/elasticsearch/202308082147119676.png)

> 如果你希望一定不能丢失数据的话，可以设置 index 的`index.translog.durability`参数，使每次写入一条数据，都是写入 buffer，同时 fsync 写入磁盘上的 translog 文件，但是这会导致写性能、吞吐量严重下降。

### 1.4 merge

由于每 refresh 一次，就会 os cache 中产生一个新的 segment file，所以随着 segment file 越来越多，搜索的性能会降低。此时，Elasticsearch 会定期执行 _**merge**_ 操作。

每次 merge 时，ES 会选择一些相似大小的 segment 进行合并，同时会将那些标识为`deleted`的 document 物理删除掉。合并后新的 segment file 会被写入磁盘，同时会新建 commit point 文件，里面标识着所有合并后新的 segment file。

三、总结
----

本章，我们介绍了 Elasticsearch 的持久化原理，核心就是 refresh、flush、merge 这三个操作。目前很多开源分布式框架都采用了这种数据持久化的思路。
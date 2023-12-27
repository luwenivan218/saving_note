> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/5626001081)

> 本章，我将从分布式系统的几个核心要点去讲解 Elasticsearch 的基本架构。通过本章的学习，我们会看到，分布式框架的很多架构设计理念都是相通的，无外乎就是围绕着 & nbsp;

转

 2023-08-08  阅读 (191)  点赞 (0)

原文地址：https://www.tpvlog.com/article/146

本章，我将从分布式系统的几个核心要点去讲解 Elasticsearch 的基本架构。通过本章的学习，我们会看到，分布式框架的很多架构设计理念都是相通的，无外乎就是围绕着 **高可用** 、 **可扩展** 、 **高性能** 、 **数据一致性** 这四方面展开的。

一、基本架构
------

### 1.1 进程节点

我们在生产环境部署 Elasticsearch 时，都是一台机器上启动一个 Elasticsearch 进程实例，比如我们有三台机器，那么三个 ES 进程实例就构成了一个 ES 集群，每个实例我们把它叫做一个节点：

![](http://image.skjava.com/article/series/elasticsearch/202308082146413281.png)

### 1.2 负载均衡

当我们建立 Index 索引时，必须指定索引有几个 primary shard，以及 primary shard 的 replica shard 数量。

比如，创建一个名称为`test_index`的索引，共有 3 个 primary shard，每个 primary shard 都有 2 个 replica shard，那总共就是 9 个 shard：

```
    PUT /test_index
    {
       "settings" : {
          "number_of_shards" : 3,
          "number_of_replicas" : 2
       }
    }


```

`test_index`索引的数据会被均匀分布到 3 个 primary shard——`P1、P2、P3`上，而 3 个 primary shard 又会被负载均衡到 ES 节点上。我们现在有三个 ES 进程节点，那么 primary shard 的分布就是类似下面这样的，每个 ES 节点上分布 1 个 primary shard：

![](http://image.skjava.com/article/series/elasticsearch/202308082146423542.png)

每个 primary shard 都有一个 replica shard，replica shard 其实就是个数据备份，类似于 msater/slave 模式中的 slave 节点。所以 6 个 replica shard 也会被负载均衡到各个 ES 进程节点上，最终的 shard 分布可能是下面这样的：

![](http://image.skjava.com/article/series/elasticsearch/202308082146432943.png)

上图中，颜色相同的代表一个 primary shard 和它的所有 replica shard，比如对于`P1`这个 primary shard，它有两个 replica shard：`R1-1`和`R1-2`。

> Elasticsearch 有一个原则： **primary shard 和它对应的 replica shard 不能全部署在同一个节点上** ，否则节点挂了的话，数据和拷贝都会丢失。

二、高可用
-----

Elasticsearch 保证集群高可用的方式和大多数分布式框架类似，就是采用 **主从模式 + 选举** 的方式。每个 primary shard 都有其所属的 replica shard 作为副本，如果 primary shard 挂了，就会从所有 replica shard 中选举出一个新的 Leader 作为 primary shard。

我们以下图的部署来具体看下 Elasticsearch 是如何保证集群的高可用的：

![](http://image.skjava.com/article/series/elasticsearch/202308082146441394.png)

初始情况下，三个 ES 进程节点都正常运行，此时集群的状态就是 **green** ，也就是完全正常状态。

假设 ES 进程 1 节点宕机了，我们来看下整个集群的可用性：

![](http://image.skjava.com/article/series/elasticsearch/202308082146450325.png)

上图中，节点 1 宕机了，所以对于 P1 这个 primary shard，就是非 active 状态，此时集群状态转为 **red** 。但是节点 2 和节点 3 上仍然保存着完整的数据，就算再挂掉一个节点，只剩下一个节点 2 或节点 3，整个集群也还是可用的。

当集群状态变为 **red** 后，P1 的两个副本——R1-1 和 R1-2 就会开始一轮选举，胜出者成为新的 primary shard，比如我们假设 R1-1 胜出：

![](http://image.skjava.com/article/series/elasticsearch/202308082146457976.png)

选举完成后，`test_index`索引的三个 primary shard 都是存活的，但是 P1 只有一个 repilca shard 是 active 状态，所以此时集群的状态变成 **yellow** 。

最后，当宕机的那个节点 1 恢复后，上面的 shard 又会重新加入到集群中，原来的 P1 发现节点 2 上已经有了新的 primary shard，自己就会变成 repilca shard，并进行数据同步：

![](http://image.skjava.com/article/series/elasticsearch/202308082146466247.png)

三、可扩展
-----

Elasticsearch 实现水平扩展的方式就是[数据分散集群模式](https://www.tpvlog.com/article/76)，也就是数据分片。

一个索引的数据会被均匀分布到它的 primary shard 中，比如`test_index`索引一共有 3T 的数据，那么每个 primary shard 就有 1T 数据。replica shard 是 primary shard 的副本，所以也包含 1T 数据，此时集群的情况可能是下面这样的：

![](http://image.skjava.com/article/series/elasticsearch/202308082146475698.png)

如果每个 ES 节点的磁盘容量总共也就 3T 多怎么办？此时所有节点的磁盘都几乎撑满了。

Elasticsearch 可以透明的实现节点水平扩展，只要再找台机器，启动一个 Elasticsearch 进程，将其加入到当前集群中，那么上面的 shard 重新在节点上自动均匀分布。比如我们再加三台机器：

![](http://image.skjava.com/article/series/elasticsearch/202308082146482869.png)

通过这种方式，最多可以加到 9 台机器，刚好对应 9 个 shard：

![](http://image.skjava.com/article/series/elasticsearch/2023080821464913310.png)

四、总结
----

本章，我们对 Elasticsearch 的基本架构进行了讲解，后续章节深入其底层实现细节。
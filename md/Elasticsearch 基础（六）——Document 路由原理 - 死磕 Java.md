> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1039833268)

> 在 Elasticsearch 中，记录是以 document 为单位存储在 shard 上的。比如，一个名称为 test_index 的索引，共有 3 个 primaryshard，那么对于任意一

转

 2023-08-08  阅读 (186)  点赞 (0)

原文地址：https://www.tpvlog.com/article/146

在 Elasticsearch 中，记录是以 document 为单位存储在 shard 上的。比如，一个名称为`test_index`的索引，共有 3 个 primary shard，那么对于任意一条 document 记录，只能存在于其中的某一个 primary shard 上，如下图（为简便起见，省略了 replica shard）：

![](http://image.skjava.com/article/series/elasticsearch/202308082146553191.png)

那么，问题来了，对于任意一条 document 记录，到底该分配到哪个 shard 上呢？

一、数据路由
------

如果读者看过我的[《分布式系统从理论到实战系列》](https://www.tpvlog.com/article/62)，那么对分布式系统中的这种数据分散集群下的数据路由一定不会陌生。

事实上，Elasticsearch 就是采用了 hash 路由算法，对 document 记录的 id 标识进行计算，产生了一个 shard 序号，通过这个 shard 序号就可以立即确认写到哪个 shard 里面。

举个例子，假设我们往`test_index`这个索引里面写入了一条 document 记录（id=1024），然后按照路由算法`shard = hash(routing_key) % number_of_primary_shards`，计算出 shard=1，那么就写到序号为 1 的那个 primary shard 中。

> 我们也可以手动指定 document 的 routing_key 值，那么 routing_key 相同的 document 就会路由到同一个 shard 中。

二、数据写入
------

了解 Elasticsearch 进行数据路由的基本原理，我们就来完整的看下 document 写入（增删改）的整个流程，以便大家整个数据路由过程有个更清晰的认识。

假设我们 ES 集群是下面这个样子的，3 个 primary shard，每个 primary shard 都有一个副本。初始时，客户端（集成了 Elasticsearch Client SDK）发起了一条 document 的写入请求，请求可能 hit 到任意某个 ES 节点上，hit 到的这个节点也叫做 _**coordinate node（协调节点）**_ ：

![](http://image.skjava.com/article/series/elasticsearch/202308082146568822.png)

> 由于 ES 进程 1、2、3 构成了一个集群，所以每个 ES 节点其实都知道集群中的其它节点的信息，包括集群中一共有多少 primary/replica shard，每个节点上分配着哪些 primary/replica shard。

假设 ES 进程 2 节点（协调节点）接受到了请求，于是根据 document 的 id 进行 hash 计算，发现结果是 3，也就是应该由 P3 这个 primary shard 处理这个请求，所以就会把请求转发给 ES 进程 3 节点上的 P3：

![](http://image.skjava.com/article/series/elasticsearch/202308082146578843.png)

primary shard 3 处理完请求后，会将数据同步到自己的 replica shard（R3-1），同步完后响应 ES 进程 2：

![](http://image.skjava.com/article/series/elasticsearch/202308082146594824.png)

最后，ES 进程 2（协调节点）收到响应后，返回给 ES client 结果：

![](http://image.skjava.com/article/series/elasticsearch/202308082147004705.png)

> 从上述流程也可以看出，Elasticsearch 对于写请求，最终都是转交给 primary shard 去处理的。

三、数据查询
------

document 数据查询的原理基本和写入类似，只不过查询请求既可以由 primary shard 处理，也可以由 replica shard 处理，这样就提高了系统的吞吐量和性能。

_**coordinate node（协调节点）**_ 在接受到查询请求后，会采用 round-robin 算法，在对应的 primary shard 及其所有 replica 中随机选择一个发送请求，以达到读请求负载均衡的目的。

我们继续通过流程图来看，首先客户端发起查询某个 document 的请求，假设命中到 ES 进程 2，ES 进程 2 根据 document Id 计算出应该由 primary shard 3 来处理：

![](http://image.skjava.com/article/series/elasticsearch/202308082147015946.png)

primary shard 3 有一个 replica，所以协调节点会采用 round-robin 算法选取其中一个转发请求，比如选择了 R3-1，然后将请求转发给它，R3-1 查询得到结果后返回，最终 ES 进程 2 将结果返回给客户端：

![](http://image.skjava.com/article/series/elasticsearch/202308082147022527.png)

四、总结
----

本章，我们对 document 数据读写的路由机制进行了讲解。Elasticsearch 的这种路由机制其实就是把每个节点都看成是对等数据路由中心。事实上，在很多开源分布式框架中，还有一种做法是引入外部的数据路由中心，比如 Zookeeper，或者像 RocketMQ 那样自己实现一个路由中心——NameServer，感兴趣的童鞋可以看看我写的[《分布式系统从理论到实战系列》](https://www.tpvlog.com/article/62)。
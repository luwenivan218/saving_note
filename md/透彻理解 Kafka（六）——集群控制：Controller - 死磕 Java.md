> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/4111203600)

> Kafka 集群启动时，会自动选举出一个 Broker，承担 Controller 的责任。所谓 Controller，就是 Kafka 集群的一个总控组件，负责管理整个集群，包括 Leade

转

 2023-07-31  阅读 (147)  点赞 (0)

原文地址：https://www.tpvlog.com/article/279

Kafka 集群启动时，会自动选举出一个 Broker，承担 Controller 的责任。所谓 Controller，就是 Kafka 集群的一个总控组件，负责管理整个集群，包括 Leader Partition 选举、分区负载均衡、管理集群元数据等等。

那么，本章我们就来看看，Controller 的核心工作机制。

一、Controller 选举
---------------

首先，我们来看下，Kafka 是如何进行 Controller 选举的。

在 Kafka 集群启动的时候，每一个 Broker 都会尝试去 Zookeeper 创建一个`/controller`临时节点，Zookeeper 会保证只有一个 Client 可以创建成功，创建成功的那个 Broker 就成为了 Controller，集群中的其它 Broker 会监听这个节点。

根据 Zookeeper 的会话保持机制，一旦 Controller 所在的 Broker 宕机了，那么临时节点就会消失，由于集群的其它 Broker 会一直监听这个临时节点，所以一旦发现临时节点消失了，就会再次争抢创建临时节点，从而保证有一个新的 Broker 会成为 Controller 角色。

二、Partition Leader 选举
---------------------

Kafka 在创建 Topic 时，一般都会指定 Partition 分区，每个分区都有一个 Leader，N 个 Follower，那么 Kafka 是如何实现 Partition Leader 选举的呢？

1.  首先，在创建 Topic 时，Kafka 就会往 Zookeeper 中注册 Topic 的元数据：包括分区数，每个分区有几个副本，每个副本的状态等等，分区副本的状态初始时都是`NonExistentReplica`；
2.  Kafka Controller 会监听 Zookeeper 的数据变更，当监听到 Topic 变动时，会从 Zookeeper 加载该 Topic 所有分区的副本到内存里，然后把这些副本的状态变更为`NewReplica`；
3.  最后，从中选择第一个副本作为 Leader，其他都是 Follower，并且把它们都加入到分区的 ISR 列表中，同时设置整个 Partition 的状态为`OnlinePartition`。

**举个例子来理解下：**

比如创建了一个`order_topic`，一共 3 个分区，每个分区共 2 个副本（一个 Leader，一个 Follower）。Kafka 会将`order_topic`的元数据信息写入 Zookeeper 中：

```
    /topics/order_topic
    
    partitions = 3, replica_factor = 2
    
    [partition0_1, partition0_2]
    [partition1_1, partition1_2]
    [partition2_1, partition2_2]


```

Kafka Controller 监听到变化后，会从每个 Partition 的副本列表中取第一个作为 Leader，其它的就是 follower，然后全部加入到该 Partition 对应的 ISR 列表中。

接着，Controller 会根据一些算法让 Partition 的每个副本都均匀分布到不同机器，同时还会设置整个 Partition 的状态为`OnlinePartition`。

最后，Controller 还会把这个 Partition 和副本所有的信息（包括谁是 Leader，谁是 Follower，ISR 列表），都发送给所有 Broker 让他们知晓。所以，在 Kafka 集群中，每个 Broker 都有一份各个 Partition 的元数据。

三、Topic 删除
----------

当我们删除一个 Topic 时，Kafka Controller 会发送请求给这个 Topic 的所有 Partition 所在的 Broker 机器，通知它们设置所有 Partition 副本的状态为`OfflineReplica`，也就是让这个 Topic 的所有分区副本下线。

接着，Controller 会将全部副本状态变为`ReplicaDeletionStarted`，然后发送请求给 Broker，把 Partition 副本的数据删除，也就是删除磁盘上的日志文件，删除成功后副本状态会变为`ReplicaDeletionSuccessful`。

最后，副本状态会变为`NonExistentReplica`，同时设置分区状态为`Offline`。
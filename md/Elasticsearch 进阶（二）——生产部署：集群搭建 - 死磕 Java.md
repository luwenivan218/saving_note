> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/2932898808)

> 本章，我将讲解如何搭建一个 Elasticsearch 集群。一般来说，一个 Elasticsearch 集群至少建议有 4~5 个节点。我会在本章中，通过虚拟机部署一个 4 节点的 ES 集群，

转

 2023-08-08  阅读 (210)  点赞 (0)

原文地址：https://www.tpvlog.com/article/170

本章，我将讲解如何搭建一个 Elasticsearch 集群。一般来说，一个 Elasticsearch 集群至少建议有 4~5 个节点。我会在本章中，通过虚拟机部署一个 4 节点的 ES 集群，每个节点都是 2 核 4G 的机器。

这里思考下，每台机器上启动一个 ES 进程，那么怎么让多台机器上的 ES 进程之间互相发现对方，然后组成一个 ES 集群呢？这就涉及 Elasticsearch 的集群发现机制。

一、zen discovery 集群发现
--------------------

默认情况下，Elasticsearch 进程启动后会绑定在`127.0.0.1`上，然后扫描本机上的`9300-9305`端口，尝试跟那些端口上启动的其他 ES 进程通信，组成一个集群。这对于在本机上搭建 ES 集群是很方便的，但是对于生产环境是不行的。在生产环境中的多台机器上部署 ES 集群，就涉及到了 _**zen discovery**_ 机制。

Elasticsearch 是一种 p2p 分布式架构，集群中的每个节点都可以直接跟其它节点进行通信。在 Elasticsearch 集群中，有两种角色： _**master node**_ 和 _**data node**_ ，一个集群只有一个 master node，其它都是 data node。

> 事实上 Elasticsearch 集群中，有三种节点：_master node_、_master eligible node_、_data node_。master eligible node 表示 master 候选节点，ES 会从这些候选节点中选举出一个作为 master node，其它 master eligible node 只是在 master node 故障的时候有接替他的资格，但是还是作为 data node 去使用。

### 1.1 master node

master node 的责任就是 **负责维护集群状态信息（cluster state），也就是集群元数据信息** 。当 cluster state 发生变化时，master node 会将最新的 cluster state 推送给集群中所有的 data node，所以每一个 data node 都有一份完整的 cluster state。

### 1.2 data node

集群中，除了 master node 以外的节点，都是 data node。data node 就是负责数据的读写。

### 1.3 unicast

zen discovery 底层实际上是采用了 _unicast discovery_ 集群发现机制，_unicast discovery_ 会提供一个种子 node 作为中转路由节点。简单来说，如果要让多个节点发现对方并组成一个集群，那么就得有一个中间的公共节点，然后不同的节点就发送请求到这些公共节点，接着通过公共节点交换各自的信息，让所有的 node 感知到其他的 node 存在，并且进行通信，最后组成一个集群。这其实就是基于 _**gossip 流言通信协议**_ 的 unicast 集群发现机制。

unicast 要求配置一个主机列表，用来作为 gossip 通信协议的路由器。这些机器如果通过 hostname 来指定，那么在 ping 的时候会被解析为 ip 地址。unicast discovery 机制最重要的两个配置如下：

*   discovery.zen.ping.unicast.hosts：用逗号分割的主机列表，比如 ["host1", "host2"]；
*   hosts.resolve_timeout：hostname 被 DNS 解析为 ip 地址的 timeout 时长。

当一个 node 与 unicast node list 中的一个成员通信之后，就会接收到一份完整的集群状态，里面包含集群中所有的 node。

### 1.4 master 选举

每个 node 都会通过 ping 跟其它 node 通信，在这个过程中，ES 集群会自动完成 Master 选举。在完成集群的 master 选举之后，如果有新的 node 加入集群，都会发送一个 join request 到 master node。如果 master node 宕机，那么集群中的 node 会再次进行 ping 过程，并且选举出一个新的 master。

以下几个参数在 Master 选举中非常重要：

*   discovery.zen.master_election.ignore_non_master_pings：如果设为 true，会强制区分 master 候选节点，如果 node 的`node.master`设置为了 false，还来发送 ping 请求参与 master 选举，那么这些 node 会被忽略掉，因为他们没有资格参与；
*   discovery.zen.minimum_master_nodes：用于设置对于一个新选举的 master，要求必须有多少个 master 候选 node 去连接这个新选举的 master，如果这些要求没有满足，那么 master node 就会被停止，然后重新选举新 master。这个参数必须设置为候选 node 的 quorum 数量。一般避免只有两个候选 node，因为 2 的 quorum 还是 2，在这种情况下，任何一个候选节点宕机，集群就无法正常运作了；
*   discovery.zen.ping_timeout：用来设置 ping 超时时间，默认 3s。如果因为网络慢或拥塞，导致 master 选举超时，那么可以增加这个参数值，确保集群启动的稳定性。

### 1.5 故障发现

Elasticsearch 有两种集群故障发现机制：

1.  第一种是通过 master 进行的，master 会 ping 集群中所有的其他 node，确保它们是否是存活着的；
2.  第二种是每个 node 都会去 ping master node 来确保 master node 是存活的，否则就会发起一个选举过程。

有下面三个参数用来配置集群故障发现：

*   **ping_interval：** 每隔多长时间 ping 一次 node，默认 1s；
*   **ping_timeout：** 每次 ping 的 timeout 时长，默认是 30s；
*   **ping_retries：** 如果一个 node 被 ping 多少次都失败了，就会认为 node 故障，默认是 3 次。

二、集群部署
------

### 2.1 机器配置

虚拟机如何安装我就不赘述了，读者可以采用 VMware+Centos 安装。假设我们已经安装好了 4 台 Linux 虚拟机，并配置了 hostname 分别为`elasticsearch01`、`elasticsearch02`、`elasticsearch03`、`elasticsearch04`，也配置好了各种运行环境，比如 JRE 环境。

### 2.2 ES 节点部署

首先下载 Elasticsearch 的安装包并解压，包内目录说明如下：

```
    bin：存放es的一些可执行脚本，比如用于启动es进程的脚本，以及用于安装插件的elasticsearch-plugin脚本
    conf：存放es的配置文件，比如elasticsearch.yml
    data：存放es的数据文件，就是每个索引的shard的数据文件
    logs：存放es的日志文件
    plugins：存放es的插件
    script：存放一些脚本文件


```

关于 master node 和 data node 设置，可以通过以下参数：

```
    
    node.master: true
    node.data: false


```

关于 master node 和 data node 的数量分配，一般有如下建议：

*   如果是小集群（node 数量小于 10），那就所有的 node 作为 master eligible node；
*   如果大集群，建议单独拆分 master node 和 data node。

三、总结
----

本章，我主要介绍了 Elasticsearch 集群的 discover 机制，下一章，我将对集群部署中的重要参数做讲解。
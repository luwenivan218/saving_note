> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/3145346162)

> 在生产环境进行 Elasticsearch 集群部署时，涉及很多的参数配置，其中又涉及 Elasticsearch 的底层原理。本章，我们就来聊聊生产环境中这些集群参数的含义和配置。首

转

 2023-08-08  阅读 (185)  点赞 (0)

原文地址：https://www.tpvlog.com/article/170

在生产环境进行 Elasticsearch 集群部署时，涉及很多的参数配置，其中又涉及 Elasticsearch 的底层原理。本章，我们就来聊聊生产环境中这些集群参数的含义和配置。

首先说明一点，Elasticsearch 的默认参数已经经过了相当好的优化，适合绝大多数的情况，尤其是一些性能相关的配置。因此，生产环境下的 ES 集群，几乎所有的配置参数都可以用默认的。也许 MySQL 或 Oracle 这种关系型数据库，非常看重调优，但是 Elasticsearch 真的不需要。如果我们真的面临一些 Elasticsearch 的性能问题，通常建议的解决方案是更好的进行数学建模，或者增加更多的机器资源。

但是在生产环境中，还是有一些业务相关的配置是需要我们修改的，这些配置往往和具体的业务场景相关联，是没法预先给予最好的默认配置的。

一、名称配置
------

### 1.1 集群名称

我们首先来看下集群名称的配置，可以在`elasticsearch.yml`中配置 Elasticsearch 集群的名称：

```
    cluster.name: your_es_name


```

默认情况下，Elasticsearch 会启动一个名称为`elasticsearch`的集群，建议一定要将集群名称重新进行命名，主要是避免公司网络环境中，也许某个开发人员的开发机无意中加入你的集群。

### 1.2 节点名称

每个 node 启动的时候，Elasticsearch 会分配一个随机的名称，这个不适合在生产环境中使用，因为这会导致我们没法记住每台机器，而且每次重启节点都会随机分配，就导致 node 名称每次重启都会变化。因此，我们在生产环境中是需要给每个 node 都分配一个名称的，可以在`elasticsearch.yml`中配置 node 节点的名称：

```
    node.name: es_node_name


```

二、路径配置
------

### 2.1 目录

默认情况下，Elasticsearch 会将 plugin、log、data、config 等都放在安装目录中。这有一个问题，就是在进行 ES 升级的时候，这些目录可能会被覆盖掉，导致我们丢失之前安装好的 plugin、log、data、config 等。

所以建议在生产环境中，必须将这些重要的文件路径都重新设置一下，放在 ES 安装目录以外的地方，可以通过配置`elasticsearch.yml`来改变这些目录位置：

*   path.data：用于设置数据文件的目录，可以指定多个目录，用逗号分隔即可，如果多个目录在不同的磁盘上，那么这就是一个最简单的 RAID 0 方式；
*   path.logs：用于设置日志文件的目录；
*   path.plugins：用于设置插件存放的目录。

一般建议的目录地址是：

```
    path.logs: /var/log/elasticsearch
    path.data: /var/data/elasticsearch
    path.plugins: /var/plugin/elasticsearch


```

### 2.2 配置文件

我们还可以自定义 Elasticsearch 的配置文件的目录，可以通过以下命令来重新设置：

```
    ./bin/elasticsearch -Epath.conf=/etc/elasticsearch/


```

三、脑裂配置
------

所谓脑裂问题，其实就是因为网络问题导致的网络分区，比如集群被划分成了两片，每片都有多个 data node 以及一个 master node，那么整个集群中就出现了两个 master node。但是因为 master node 是集群中非常重要的一个角色，主宰了集群状态的维护、shard 分配，所以如果有两个 master 的话就会造成数据不一致。

### 3.1 最少 master 候选节点

Elasticsearch 中有一个很重要的参数：`discovery.zen.minimum_master_nodes`。

这个参数告诉 Elasticsearch：只有 master 候选节点的数量满足这个值时，才可以进行 master 选举。这个参数的值，必须设置为集群中 master 候选节点的 quorum 数量，也就是大多数（master 候选节点数量 / 2 + 1）。

所以， **一个生产环境的 Elasticsearch 集群，至少要有 3 个节点，同时将这个参数设置为 master 候选节点数的 quorum，就可以防止脑裂问题的产生** 。

但是因为 es 集群是可以动态增加和下线节点的，所以可能随时会改变 quorum。所以这个参数也是可以通过 API 随时修改的，特别是在节点上线和下线的时候，都需要作出对应的修改。而且一旦修改过后，这个配置就会持久化保存下来：

```
    PUT /_cluster/settings
    {
        "persistent" : {
            "discovery.zen.minimum_master_nodes" : 2
        }
    }


```

四、shard recovery 配置
-------------------

在集群启动的时候，有一些配置会影响 shard 恢复的过程。首先，我们需要理解在默认配置下，shard 恢复的过程会发生什么事情：

首先，假设我们有 10 个 node，每个 node 都有一个 shard（总共 5 个 primary shard，5 个 replica shard）。如果我们将集群关闭后进行一些维护性的操作，当我们重启集群的时候，此时可能会出现 5 个节点先启动了，然后剩下 5 个节点还没启动。

接着，这 5 个启动的节点会通过 gossip 协议互相通信，选举出一个 master，然后组成一个集群。他们会发现数据没有被均匀的分布，因为有 5 个节点没有启动，那么那 5 个节点上的 shard 就是不可用的。此时在线的 5 个 node 就会将部分 replica shard 提升为 primary shard，同时为每个 primary shard 复制足够的 replica shard。

最后，剩下的 5 个节点加入了集群。但是这些节点发现本应该由他们持有的 shard 已经被复制了，此时他们就会删除自己本地的数据。然后集群又会开始进行 shard 的 rebalance 操作，将最早启动的 5 个 node 上的 shard 均匀分布到后来启动的 5 个 node 上去。

在这个过程中，shard 重新复制、移动、删除、再次移动的过程，会耗费大量的网络和磁盘资源。对于数据量庞大的集群来说，可能导致每次集群重启时都有 TB 级别的数据移动，导致集群启动会耗费很长时间。

### 4.1 gateway.expected_nodes

如果可以让整个集群中的所有节点都完全上线之后，再进行复制和移动 shard，那情况就会好很多。所以，Elasticsearch 提供了一个`gateway.recover_after_nodes`参数，这个参数可以让 ES 等待足够的 node 都上线之后，再开始进行 shard recovery 的过程。

`gateway.recover_after_nodes`参数一般和`gateway.recover_after_time`、`gateway.expected_nodes`配合使用，比如进行如下设置：

```
    gateway.recover_after_nodes: 8
    gateway.recover_after_time: 5m
    gateway.expected_nodes: 10


```

上述配置后，Elasticsearch 集群的行为会变成这样：等待至少 8 个节点在线，然后等待最多 5 分钟或者 10 个节点都在线，才开始进行 shard recovery 的过程。这样就可以避免少数 node 启动时，就立即开始 shard recovery，消耗大量的网络和磁盘资源，甚至可以将 shard recovery 过程从数小时缩短为数分钟（大数据量场景下）。

五、总结
----

本章，我主要介绍了 Elasticsearch 的一些通用参数配置，包括集群的名称配置、node 名称配置，以及配置文件的默认路径修改。
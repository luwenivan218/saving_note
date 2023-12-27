> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1041688026)

> 我们上一章在讲解 Document 数据写入时，提到真正处理请求的那个 primaryshard 会把数据同步给自己的 replicashard，同步成功后才返回响应。事实上，这么说并不

转

 2023-08-08  阅读 (198)  点赞 (0)

原文地址：https://www.tpvlog.com/article/146

我们上一章在讲解 Document 数据写入时，提到真正处理请求的那个 primary shard 会把数据同步给自己的 replica shard，同步成功后才返回响应。

事实上，这么说并不完全准确，Elasticsearch 其实提供了三种数据同步机制： _**one**_ ， _**all**_ ， _**quorum**_ 。我们可以在请求时带上`consistency`参数表明采用哪种模式，默认是 _**quorum**_ 。例如：

```
    put /index/type/id?consistency=quorum


```

一、数据一致性
-------

### 1.1 one 模式

所谓 one 模式，就是对于 document 的写操作（增删改），只要有一个 primary shard 是 active 活跃可用的，操作就可以执行。

### 1.2 all 模式

所谓 all 模式，就是对于 document 的写操作（增删改），要求必须所有的 primary shard 和 replica shard 都是活跃的，才可以执行这个写操作。

### 1.3 quorum 模式

所谓 quorum 模式，就是对于 document 的写操作（增删改），写之前必须确保大多数 shard 都可用，当不满足 “大多数” 这个条件时，请求就会默认等待 1 分钟，超过时间就会报 timeout 错误。我们可以在写操作的时候，加一个`timeout`参数，比如：

```
    PUT /index/type/id?timeout=30


```

这样就可以自己控制超时时间。

那么，何谓大多数 shard 呢？事实上，Elasticsearch 是通过一个公式去计算的：

```
    (primary + number_of_replicas) / 2 + 1


```

举个例子，我们有个索引`test_index`，一共是 1 个 primary shard，3 个 replica shard：

```
    PUT /test_index
    {
       "settings" : {
          "number_of_shards" : 1,
          "number_of_replicas" : 3
       }
    }


```

假设我们有两个 ES 进程节点，那么 shard 的分布可能是下面这样的：

![](http://image.skjava.com/article/series/elasticsearch/202308082147052941.png)

此时，`(primary + number_of_replicas) / 2 + 1 = (1+3)/2+1 = 3`，那么只有当 3 个 shard 都是 active 状态时，写操作才能执行，如果任意一个 ES 节点挂了，写操作就不能执行了（此时只有 2 个 active shard）。

> 注意，quorum 模式只有当`number_of_replicas>1`时才生效，如果不满足条件，请求就会一直 wait。

四、总结
----

本章，我们介绍了 Elasticsearch 的写一致性原理，特别要注意 quorum 模式时对于 active shard 数量的要求，我们在对 index 进行 shard 拆分时也要仔细考虑清楚。
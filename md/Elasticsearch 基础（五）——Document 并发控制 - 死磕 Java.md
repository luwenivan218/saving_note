> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/5598324267)

> 我们知道，在数据库中，如果多个客户端线程同时对一条记录进行增删改查，数据库会有锁机制保证并发访问的数据一致性。在 Elasticsearch 中，document 就表示一条数据记录

我们知道，在数据库中，如果多个客户端线程同时对一条记录进行增删改查，数据库会有锁机制保证并发访问的数据一致性。在 Elasticsearch 中，document 就表示一条数据记录，那么 Elasticsearch 是如何对 document 的并发访问进行控制的呢？

本章，我们就以一个电商下单的案例为背景，介绍 document 的并发控制机制。

一、案例背景
------

我们现在有一个商品管理系统，底层采用 Elasticsearch 存储商品信息（包括商品 ID、商品库存等）。现在，用户需要进行下单操作，流程如下：

1.  用户浏览商品，此时发送请求到商品管理系统，查询商品信息；
2.  用户下单购买；
3.  支付成功后，发送请求到商品管理系统，扣减商品库存（库存 - 1）。

二、version 版本号
-------------

Elasticsearch 内部采用了版本号机制对 document 的并发修改进行控制 。所谓版本号，本质是一种乐观锁。

### 3.1 _version

我们往 Elasticsearch 写入 document 数据时，ES 会自动为 document 生成一个版本号`_version`，比如我们插入一条 id 为 6 的数据，返回结果里的`_version`就表示这条数据的版本号：

```
    PUT /test_index/test_type/6
    {
      "test_field": "test test"
    }
    
    返回：
    {
      "_index": "test_index",
      "_type": "test_type",
      "_id": "6",
      "_version": 1,
      "result": "created",
      "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
      },
      "created": true
    }


```

第一次创建 document 时，它的`_version`版本号是 1；以后，每次对这个 document 修改或删除时，`_version`版本号会自动加 1（哪怕是删除，也会对这条数据的版本号加 1）。

### 3.2 版本控制

我们来看下 ES 的数据版本控制流程，整个业务执行流程如下：

1.  假设我们现在有两个线程 A 和 B，同时读取到了商品信息，此时获取到的 document 的`_version`版本号是相同的，假设都是 1；
2.  A 线程在本地将库存减 1 后，发送 PUT 更新请求，请求里带上了版本号`PUT /product/storage/1?version=1`；
3.  ES 收到请求后，对数据进行更新，然后将版本号 + 1；
4.  B 线程在本地将库存减 1 后，发送 PUT 更新请求，请求里也带上了版本号 1，`PUT /product/storage/1?version=1`；
5.  ES 收到请求后，发现 id 为 1 的 document 的版本号已经是 2 了，而 B 线程的更新请求里带的版本号还是 1，两者不一样，于是拒绝更新，返回报错。

### 3.3 external version

Elasticsearch 还提供了一个外部版本号的功能，就是说可以不用它提供的内部`_version`版本号来进行并发控制，可以基于你自己维护的一个版本号来进行并发控制。

内部版本语法：

```
    ?version=1


```

自定义版本号语法：

```
    ?version=1&version_type=external


```

注意，当`version_type=external`的时，只有当提供的 version 比 Elasticsearch 中的`_version`大的时候，才能完成修改。

我们来看个例子，比如现在写入了这样一条记录,，可以看到`_version`版本号是 1：

```
    PUT /test_index/test_type/8
    {
      "test_field": "test"
    }
    
    {
      "_index": "test_index",
      "_type": "test_type",
      "_id": "8",
      "_version": 1,
      "result": "created",
      "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
      },
      "created": true
    }


```

然后，A 线程先进行修改，带上了外部版本号 2：

```
    PUT /test_index/test_type/8?version=2&version_type=external
    {
      "test_field": "test client 1"
    }
    
    {
      "_index": "test_index",
      "_type": "test_type",
      "_id": "8",
      "_version": 2,
      "result": "updated",
      "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
      },
      "created": false
    }


```

接着，B 线程也进行修改，带上了外部版本号 2，可以看到报错了，因为当`version_type=external`的时，只有当提供的 version 比 Elasticsearch 中的`_version`大的时候，才能完成修改：

```
    PUT /test_index/test_type/8?version=2&version_type=external
    {
      "test_field": "test client 2"
    }
    
    {
      "error": {
        "root_cause": [
          {
            "type": "version_conflict_engine_exception",
            "reason": "[test_type][8]: version conflict, current version [2] is higher or equal to the one provided [2]",
            "index_uuid": "6m0G7yx7R1KECWWGnfH1sw",
            "shard": "1",
            "index": "test_index"
          }
        ],
        "type": "version_conflict_engine_exception",
        "reason": "[test_type][8]: version conflict, current version [2] is higher or equal to the one provided [2]",
        "index_uuid": "6m0G7yx7R1KECWWGnfH1sw",
        "shard": "1",
        "index": "test_index"
      },
      "status": 409
    }


```

三、总结
----

本章，我们讲解了 document 的并发控制原理，其核心就是基于内部的版本号机制，下一章，我们来看下 document 数据的路由原理。
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/6420178954)

> 本章，我将搭建一个 Elasticsearch 开发环境，作为后续章节讲解的基础，各位可以跟着我一起进行手动搭建。一、环境准备 1.1ES 版本选择我们先去 Elasticsearch 的

转

 2023-08-08  阅读 (234)  点赞 (0)

原文地址：https://www.tpvlog.com/article/146

本章，我将搭建一个 Elasticsearch 开发环境，作为后续章节讲解的基础，各位可以跟着我一起进行手动搭建。

一、环境准备
------

### 1.1 ES 版本选择

我们先去 Elasticsearch 的 GitHub 主页，看下它的版本变迁：[https://github.com/elastic/elasticsearch/releases：](https://github.com/elastic/elasticsearch/releases%EF%BC%9A)

![](http://image.skjava.com/article/series/elasticsearch/202308082146223331.png)

可以看到，现在最新的稳定版本是`v7.6.0`。我们也可以去的官方主页看下，`v7.x`版本相对于以前都哪些大的变动，然后根据自己的需求选择适合的版本：[https://www.elastic.co/guide/en/elasticsearch/reference/7.6/breaking-changes-7.0.html。](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/breaking-changes-7.0.html%E3%80%82)

我这里直接以最新版本作为环境搭建的基础。

### 1.2 下载 ES

前往 Elasticsearch 的官网：[https://www.elastic.co/start 进行下载。考虑到大多数童鞋用的是 windows，我这里选择 windows 版本：](https://www.elastic.co/start%E8%BF%9B%E8%A1%8C%E4%B8%8B%E8%BD%BD%E3%80%82%E8%80%83%E8%99%91%E5%88%B0%E5%A4%A7%E5%A4%9A%E6%95%B0%E7%AB%A5%E9%9E%8B%E7%94%A8%E7%9A%84%E6%98%AFwindows%EF%BC%8C%E6%88%91%E8%BF%99%E9%87%8C%E9%80%89%E6%8B%A9windows%E7%89%88%E6%9C%AC%EF%BC%9A)

![](http://image.skjava.com/article/series/elasticsearch/202308082146235862.png)

考虑到后面要用到 Kibana 作为图形界面展示，我们顺带把它也一起下载了：

![](http://image.skjava.com/article/series/elasticsearch/202308082146244413.png)

下载完成后，自己找个目录解压缩。

### 1.3 JDK 环境

Elasticsearch 源码采用 Java 编写，所以运行时依赖于 JVM，`v7.6.0`版本在根目录下有自带的 openJDK 供其使用。

所以如果想要使用自己的 JDK，需要配置好`JAVA_HOME`环境变量，同时最好用 JDK11 + 版本，因为 Elasticsearch7.x 推荐使用 LTS 版本的 JDK。

> Support Matrix：[https://www.elastic.co/cn/support/matrix#matrix_jvm](https://www.elastic.co/cn/support/matrix#matrix_jvm)

二、启动 Elasticsearch
------------------

### 2.1 启动 ES

上面步骤都完成后，进入`/bin`目录，执行`elasticsearch.bat`启动 Elasticsearch。启动完成后，在浏览器输入：[http://localhost:9200/?pretty，若显示以下 JSON 报文，则说明启动成功：](http://localhost:9200/?pretty%EF%BC%8C%E8%8B%A5%E6%98%BE%E7%A4%BA%E4%BB%A5%E4%B8%8BJSON%E6%8A%A5%E6%96%87%EF%BC%8C%E5%88%99%E8%AF%B4%E6%98%8E%E5%90%AF%E5%8A%A8%E6%88%90%E5%8A%9F%EF%BC%9A)

```
    {
      "name" : "DESKTOP-JGDSLQG",
      "cluster_name" : "elasticsearch",
      "cluster_uuid" : "M7Hak1h6RRSeQyajXPr71g",
      "version" : {
        "number" : "7.6.0",
        "build_flavor" : "default",
        "build_type" : "zip",
        "build_hash" : "7f634e9f44834fbc12724506cc1da681b0c3b1e3",
        "build_date" : "2020-02-06T00:09:00.449973Z",
        "build_snapshot" : false,
        "lucene_version" : "8.4.0",
        "minimum_wire_compatibility_version" : "6.8.0",
        "minimum_index_compatibility_version" : "6.0.0-beta1"
      },
      "tagline" : "You Know, for Search"
    }


```

上述报文可以看到 ES 集群名称、ES 版本等信息。

### 2.2 启动 Kibana

接着，我们启动 Kibana，使用里面的 UI 界面，去操作 Elasticsearch。Kibana 的启动方式和 ES 类似，进入`/bin`目录，执行`kibana.bat`，启动完成后，在浏览器输入：[http://localhost:5601，就可以进入 Kibana 的控制台了：](http://localhost:5601，就可以进入Kibana的控制台了：)

![](http://image.skjava.com/article/series/elasticsearch/202308082146253494.png)

我们进入 “Dev Tool” 界面，输入`GET _cluster/health`测试下：

![](http://image.skjava.com/article/series/elasticsearch/202308082146264395.png)

三、简单上手
------

搭建完了环境，我们就来上手体验下。Elasticsearch 提供了一套 Cat API，可以以 _RESTful_ 的方式进行各种操作。

### 3.1 集群健康状况

默认情况下，我们创建的索引归属于 “elasticsearch” 集群，我们也可以自定义集群。我们先来看看如何了解集群的健康状况？在 Elasticsearch 中，集群的健康状况一共三种：green、yellow、red。

**green：** 每个索引的 primary shard 和 replica shard 都是 active 状态的；  
**yellow：** 每个索引的 primary shard 都是 active 状态的，但是部分 replica shard 不是 active 状态（即不可用的状态）；  
**red：** 部分索引的 primary shard 不是 active 状态，也就是部分索引有数据丢失了。

通过`GET /_cat/health?v`命令看到集群现在的健康状况：

```
    epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
    1583314274 09:31:14  elasticsearch green           1         1      3   3    0    0        0             0                  -                100.0%


```

### 3.2 索引操作

#### 查询索引

`GET /_cat/indices?v`

可以看到，默认有三个索引：

```
    health stats index                    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
    green  open   .kibana_task_manager_1   iA5Q_umzQ1ySMSyiLOCI7w   1   0          2            0     57.1kb         57.1kb
    green  open   .apm-agent-configuration DE8HMOigSEaiTHmdqtxERg   1   0          0            0       283b           283b
    green  open   .kibana_1                T8O3ZQROT_C2hcdJtTOj_Q   1   0         10            0     36.2kb         36.2kb


```

#### 创建索引

`PUT /test_index?pretty`

其中`test_index`是我们创建的索引名称，pretty 参数表示显示风格，可以忽略。响应如下：

```
    {
      "acknowledged" : true,
      "shards_acknowledged" : true,
      "index" : "test_index"
    }


```

#### 删除索引

`DELETE /test_index?pretty`

响应如下：

```
    {
      "acknowledged" : true
    }


```

四、总结
----

本章，我们搭建了一个 Elasticsearch+Kibana 的开发环境，作为后续学习的基础。下一章节，我将引入一个示例，对 Elasticsearch 的基本功能进行实战讲解，便于大家快速上手 Elasticsearch。
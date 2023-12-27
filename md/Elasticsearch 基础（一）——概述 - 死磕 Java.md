> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/6688002443)

> 本系列的第一篇，我们先来了解下 Elasticsearch 到底是个东西？本章的目的，是让从来没有接触过 Elasticsearch 的同学对它有一个感性的认识。Elasticsear

本系列的第一篇，我们先来了解下 Elasticsearch 到底是个东西？本章的目的，是让从来没有接触过 Elasticsearch 的同学对它有一个感性的认识。

Elasticsearch 不是什么新技术，主要是将全文检索、数据分析以及分布式技术整合在了一起，可以作为一个大型分布式集群（数百台服务器），对 PB 级的数据进行检索和分析，而且开箱即用，上手非常简单。

我们先来看下，到底为什么会出现 Elasticsearch？

一、 起源
-----

### 1.1 全文检索

**传统数据库**

我们先来看下，如果用传统数据库，怎么做检索。假设有下面这样一张 “商品表”：

<table><thead><tr><th>商品 ID</th><th>商品名称</th><th>商品描述</th></tr></thead><tbody><tr><td>1</td><td>高露洁牙膏</td><td>我是高露洁，balabalabla...</td></tr><tr><td>2</td><td>中华牙膏</td><td>我是中华，balabalabla...</td></tr><tr><td>3</td><td>佳洁士牙膏</td><td>我是佳洁士，balabalabla...</td></tr></tbody></table>

用户输入了 “牙膏” 关键字，我们的后台程序可能会像下面这样查询：

```
    SELECT * FROM products WHERE product_name LIKE '%牙膏%';


```

缺点：

1.  数据库中字段所包含记录的可能会很长，比如 “商品描述” 这个字段，可能有上千个字段，这个时候，每次都要对每条记录的所有文本进行扫描，性能很差；
2.  不能模糊匹配，比如输入 “中华牌”，就搜索不出来 “中华牙膏”。

**倒排索引**

另一种思路是，我们先将每条记录拆分成词条：

*   高露洁牙膏 -> 高露洁、牙膏
*   中华牙膏 -> 中华、牙膏
*   佳洁士牙膏 -> 佳洁士、牙膏

这样就组成下面这样的一章索引表：

<table><thead><tr><th>词条</th><th>商品 ID</th></tr></thead><tbody><tr><td>牙膏</td><td>1、2、3</td></tr><tr><td>高露洁</td><td>1</td></tr><tr><td>中华</td><td>2</td></tr><tr><td>佳洁士</td><td>3</td></tr></tbody></table>

这样，当我们输入 “中华牌” 检索时，”中华牌 “就被拆成“中华”、“牌” 这两个词条，我们马上就可以根据词条去索引表找到商品的 ID，进而找出商品信息。

上述这种思路就叫做 “倒排索引”。假设有一百万条记录，最终词条其实可能就几十万条，另外倒排索引底层会有一些数据结构和算法保证来检索效率，所以整体效率相比传统的数据库检索方式会大幅提升。

### 1.2 Lucene

Lucene，是 Apache 提供的一个开源 Jar 包，是目前最先进、功能最强大的搜索库，里面封装了各种建立倒排索引、全文检索的算法。我们可以在自己的项目中基于 Lucene 的 api 对已有的数据建立索引，Lucene 会在本地磁盘上组织索引的数据结构。

但是，Lucene 有个缺点，上手难度大，API 非常复杂，需要深入研究各种索引结构的底层原理才能用好。其次，Lucene 只能单机开发部署，而大数据场景下索引是非常消耗磁盘的，可用性也无法保证，所以并不适合分布式环境下的数据检索。

### 1.3 整合

Elasticsearch 在底层整合了 Lucene，提供了简单易用的 Restful API 接口，同时提供了分布式的数据检索和分析，基本就是开箱即用。这就是 Elasticsearch 诞生的初衷。

二、适用场景
------

介绍完了 Elasticsearch 的诞生初衷，我们来具体看看 Elasticsearch 到底能做什么？

### 2.1 数据搜索

比如百度、各类网站的站内搜索等等，Elasticsearch 支持各种各样的检索：全文检索、结构化检索、部分匹配、自动完成、搜索纠错、搜索推荐等等。

### 2.2 数据分析

比如日志数据分析，logstash 采集日志，ES 进行复杂的数据分析（ELK 技术，Elasticsearch+Logstash+Kibana）。

比如 BI（Business Intelligence）系统，分析某区域最近 3 年的用户消费金额的趋势以及用户群体的组成构成，产出相关的数张报表。一般用 ES 执行数据分析和挖掘，Kibana 进行数据可视化。

### 2.3 大数据近实时处理

Elasticsearch 可以将海量数据分散到多台服务器上去存储和检索，在秒级别对数据进行搜索和分析。

三、核心概念
------

本节，我们先来了解下 Elasticsearch 中的一些核心概念，后续章节我们会针对每个概念深入展开讲解。

### 3.1 Document（文档）

在 Elasticsearch 中，数据以 JSON 格式进行存储，最小数据单元就叫 **document** 。一个 document 可以是一条客户数据、一条商品信息数据，一条订单数据等等。比如，下面就是一个商品信息 document：

```
    {
      "product_id": "1",
      "product_name": "高露洁牙膏",
      "product_desc": "高效美白",
      "category_id": "2",
      "category_name": "日化用品"
    }


```

### 3.2 Index（索引）

Index，指包含一堆有相似结构的 document 数据，比如一个客户索引，商品索引，订单索引等等。一个 Index 包含很多 document，代表了一类相似或者相同的 document。

### 3.3 Type（分类）

Type，表示 Index 中的一个 **逻辑数据分类** 。每个 Index 可以有多个 type，而每个 type 又可以存储多个 document。

### 3.4 Field（字段）

一个 document 里面有多个 field，每个 field 就是一个数据字段。

以关系型数据库来类比理解：

<table><thead><tr><th>Elasticsearch</th><th>关系型数据库</th></tr></thead><tbody><tr><td>Index</td><td>数据库实例</td></tr><tr><td>Type</td><td>表</td></tr><tr><td>Document</td><td>行</td></tr><tr><td>Field</td><td>列</td></tr></tbody></table>

还是不理解？没关系，我再来举个示例，假设我们的系统要存储商品信息。

*   首先 “商品” 这个概念就可以作为一个 Index，比如我们叫它`mer_idx`；
*   然后，商品有很多分类，比如日化商品、电器商品、生鲜商品，每一个分类我们把它作为一个 Type，那么就有`mer_normal_type`、`mer_elc_type`、`mer_sea_type`。
*   每一种 Type 分类下都有自己的一些商品信息，这些商品信息的大部分 Field 都是相同的，比如商品 ID、商品名称、数量、描述等等，但是每个分类又略有差别，比如电器商品类相比生鲜商品类，可能多了些电器参数字段。

```
    
    mer_elc_type(电器商品类型)：product_id，product_name，product_desc，electic_power
    mer_sea_type(生鲜商品类型)：product_id，product_name，product_desc，expired_date
    
    
    {
      
      "product_id": "2",
      "product_name": "长虹电视机",
      "product_desc": "4k高清",
      "electic_power": "250V"
    }
    {
      
      "product_id": "3",
      "product_name": "基围虾",
      "product_desc": "纯天然，冰岛产",
      "expired_date": "3天"
    }


```

### 3.5 Shard（分片）

单台机器无法存储大量数据，Elasticsearch 可以将一个 Index（索引）中的数据切分为多个 Shard，分布在不同服务器节点上，这样就可以实现水平扩展，提升吞吐量和性能。这其实是一种[数据分散集群](https://www.tpvlog.com/article/76#menu_1)的架构模式。

> 每个 Shard 都是一个 Lucene index，可以看成是一个 Lucene 实例，后续章节我会专门讲解 Shard。

### 3.6 Replica（副本）

任何一台服务器随时都可能故障或宕机，此时 Shard 中的数据就可能丢失，因此可以为每个 Shard 创建多个备份——Replica 副本。Replica 可以在 Shard 故障时提供备用服务，保证数据不丢失，多个 Replica 还可以提升搜索的吞吐量和性能。这其实是一种[主从架构模式](https://www.tpvlog.com/article/75)。

> 每个索引的 Primary Shard 数量，需要在索引建立时就设置，一旦设置完不能修改，而 Replica Shard 可以随时修改数量。

四、关于 Type 的补充说明
---------------

Elasticsearch 从`7.0`版本开始，移除了 Type 这个概念。主要原因是在同一个索引中，存储仅有小部分字段相同或者全部字段都不相同的 document，会导致数据稀疏，影响 Lucene 有效压缩数据的能力。

但是，我在本系列的讲解中仍然沿用`Type`这个概念，主要是考虑到向前兼容，因为很多公司都是用的 5.x、6.x 版本。事实上，这也引入`Type`讲解也不会造成什么影响，因为目前在`7.x`中，还是兼容 Type 的。

五、总结
----

本章，我们介绍了 Elasticsearch 是什么，以及它的一些基本概念，帮助新手童鞋有一个感性的认识。下一章，我将带着大家搭建好 Elasticsearch 环境，体验下 Elasticsearch 的基本使用。
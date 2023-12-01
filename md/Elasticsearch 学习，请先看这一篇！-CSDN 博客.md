> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/laoyang360/article/details/52244917)

[全网最新 | Elasticsearch 7.X 进阶实战视频课](https://edu.csdn.net/course/detail/35551)

[Elasticsearch](https://so.csdn.net/so/search?q=Elasticsearch&spm=1001.2101.3001.7020) 最少必要知识实战教程直播回放

题记：
---

Elasticsearch 研究有一段时间了，现特将 Elasticsearch 相关核心知识、原理从初学者认知、学习的角度，从以下 9 个方面进行详细梳理。欢迎讨论…

0. 带着问题上路——ES 是如何产生的？
---------------------

### （1）思考：大规模数据如何检索？

如：当系统数据量上了 10 亿、100 亿条的时候，我们在做系统架构的时候通常会从以下角度去考虑问题：  
1）用什么数据库好？(mysql、sybase、[oracle](https://so.csdn.net/so/search?q=oracle&spm=1001.2101.3001.7020)、达梦、神通、mongodb、hbase…)  
2）如何解决单点故障；(lvs、F5、A10、Zookeep、MQ)  
3）如何保证数据安全性；(热备、冷备、异地多活)  
4）如何解决检索难题；(数据库代理中间件：mysql-proxy、Cobar、MaxScale 等;)  
5）如何解决统计分析问题；(离线、近实时)

### （2）传统数据库的应对解决方案

对于关系型数据，我们通常采用以下或类似架构去解决查询瓶颈和写入瓶颈：  
解决要点：  
1）通过主从备份解决数据安全性问题；  
2）通过数据库代理中间件心跳监测，解决单点故障问题；  
3）通过代理中间件将查询语句分发到各个 slave 节点进行查询，并汇总结果  
![](https://img-blog.csdnimg.cn/img_convert/fb6715313b9546ac263b6abf83397269.png)

### （3）非关系型数据库的解决方案

对于 Nosql 数据库，以 mongodb 为例，其它原理类似：  
解决要点：  
1）通过副本备份保证数据安全性；  
2）通过节点竞选机制解决单点问题；  
3）先从配置库检索分片信息，然后将请求分发到各个节点，最后由路由节点合并汇总结果  
![](https://img-blog.csdnimg.cn/img_convert/3572f966207da7bf1c929fcb08c13fb3.png)

### 另辟蹊径——完全把数据放入内存怎么样？

我们知道，完全把数据放在内存中是不可靠的，实际上也不太现实，当我们的数据达到 PB 级别时，按照每个节点 96G 内存计算，在内存完全装满的数据情况下，我们需要的机器是：1PB=1024T=1048576G  
节点数 = 1048576/96=10922 个  
实际上，考虑到数据备份，节点数往往在 2.5 万台左右。成本巨大决定了其不现实！

从前面讨论我们了解到，把数据放在内存也好，不放在内存也好，都不能完完全全解决问题。  
全部放在内存速度问题是解决了，但成本问题上来了。  
为解决以上问题，从源头着手分析，通常会从以下方式来寻找方法：  
1、存储数据时按有序存储；  
2、将数据和索引分离；  
3、压缩数据；  
这就引出了 Elasticsearch。

1. ES 基础一网打尽
------------

### 1.1 ES 定义

ES=elaticsearch 简写， Elasticsearch 是一个开源的高扩展的分布式全文检索引擎，它可以近乎实时的存储、检索数据；本身扩展性很好，可以扩展到上百台服务器，处理 PB 级别的数据。  
Elasticsearch 也使用 Java 开发并使用 Lucene 作为其核心来实现所有索引和搜索的功能，但是它的目的是通过简单的 RESTful API 来隐藏 Lucene 的复杂性，从而让全文搜索变得简单。

### 1.2 Lucene 与 ES 关系？

1）Lucene 只是一个库。想要使用它，你必须使用 Java 来作为开发语言并将其直接集成到你的应用中，更糟糕的是，Lucene 非常复杂，你需要深入了解检索的相关知识来理解它是如何工作的。

2）Elasticsearch 也使用 Java 开发并使用 Lucene 作为其核心来实现所有索引和搜索的功能，但是它的目的是通过简单的 RESTful API 来隐藏 Lucene 的复杂性，从而让全文搜索变得简单。

### 1.3 ES 主要解决问题：

1）检索相关数据；  
2）返回统计结果；  
3）速度要快。

### 1.4 ES 工作原理

当 ElasticSearch 的节点启动后，它会利用多播 (multicast)(或者单播，如果用户更改了配置) 寻找集群中的其它节点，并与之建立连接。这个过程如下图所示：  
![](https://img-blog.csdnimg.cn/img_convert/3e7f4c840221802537f16118d77ff221.png)

### 1.5 ES 核心概念

#### 1）Cluster：集群。

ES 可以作为一个独立的单个搜索服务器。不过，为了处理大型数据集，实现容错和高可用性，ES 可以运行在许多互相合作的服务器上。这些服务器的集合称为集群。

#### 2）Node：节点。

形成集群的每个服务器称为节点。

#### 3）Shard：分片。

当有大量的文档时，由于内存的限制、磁盘处理能力不足、无法足够快的响应客户端的请求等，一个节点可能不够。这种情况下，数据可以分为较小的分片。每个分片放到不同的服务器上。  
当你查询的索引分布在多个分片上时，ES 会把查询发送给每个相关的分片，并将结果组合在一起，而应用程序并不知道分片的存在。即：这个过程对用户来说是透明的。

#### 4）Replia：副本。

为提高查询吞吐量或实现高可用性，可以使用分片副本。  
副本是一个分片的精确复制，每个分片可以有零个或多个副本。ES 中可以有许多相同的分片，其中之一被选择更改索引操作，这种特殊的分片称为主分片。  
当主分片丢失时，如：该分片所在的数据不可用时，集群将副本提升为新的主分片。

#### 5）全文检索。

全文检索就是对一篇文章进行索引，可以根据关键字搜索，类似于 mysql 里的 like 语句。  
全文索引就是把内容根据词的意义进行分词，然后分别创建索引，例如”你们的激情是因为什么事情来的” 可能会被分词成：“你们 “，” 激情 “，“什么事情“，” 来“ 等 token，这样当你搜索“你们” 或者 “激情” 都会把这句搜出来。

### 1.6 ES 数据架构的主要概念（与关系数据库 Mysql 对比）

![](https://img-blog.csdnimg.cn/img_convert/526c8b7e4770eaa79928a401c63b8eb0.png)  
（1）关系型数据库中的数据库（DataBase），等价于 ES 中的索引（Index）  
（2）一个数据库下面有 N 张表（Table），等价于 1 个索引 Index 下面有 N 多类型（Type），  
（3）一个数据库表（Table）下的数据由多行（ROW）多列（column，属性）组成，等价于 1 个 Type 由多个文档（Document）和多 Field 组成。  
（4）在一个关系型数据库里面，schema 定义了表、每个表的字段，还有表和字段之间的关系。 与之对应的，在 ES 中：Mapping 定义索引下的 Type 的字段处理规则，即索引如何建立、索引类型、是否保存原始索引 JSON 文档、是否压缩原始 JSON 文档、是否需要分词处理、如何进行分词处理等。  
（5）在数据库中的增 insert、删 delete、改 update、查 search 操作等价于 ES 中的增 PUT/POST、删 Delete、改_update、查 GET.

### 1.7 ELK 是什么？

ELK=elasticsearch+Logstash+kibana  
elasticsearch：后台分布式存储以及全文检索  
logstash: 日志加工、“搬运工”  
kibana：数据可视化展示。  
ELK 架构为数据分布式存储、可视化查询和日志解析创建了一个功能强大的管理链。 三者相互配合，取长补短，共同完成分布式大数据处理工作。

2. ES 特点和优势
-----------

1）分布式实时文件存储，可将每一个字段存入索引，使其可以被检索到。  
2）实时分析的分布式搜索引擎。  
分布式：索引分拆成多个分片，每个分片可有零个或多个副本。集群中的每个数据节点都可承载一个或多个分片，并且协调和处理各种操作；  
负载再平衡和路由在大多数情况下自动完成。  
3）可以扩展到上百台服务器，处理 PB 级别的结构化或非结构化数据。也可以运行在单台 PC 上（已测试）  
4）支持插件机制，分词插件、同步插件、Hadoop 插件、可视化插件等。

3、ES 性能
-------

### 3.1 性能结果展示

（1）硬件配置：  
CPU 16 核 AuthenticAMD  
内存 总量：32GB  
硬盘 总量：500GB 非 SSD

（2）在上述硬件指标的基础上测试性能如下：  
1）平均索引吞吐量： 12307docs/s（每个文档大小：40B/docs）  
2）平均 CPU 使用率： 887.7%（16 核，平均每核：55.48%）  
3）构建索引大小： 3.30111 GB  
4）总写入量： 20.2123 GB  
5）测试总耗时： 28m 54s.

### 3.2 性能 esrally 工具（推荐）

使用参考：http://blog.csdn.net/laoyang360/article/details/52155481

4、为什么要用 ES？
-----------

### 4.1 ES 国内外使用优秀案例

1） 2013 年初，GitHub 抛弃了 Solr，采取 ElasticSearch 来做 PB 级的搜索。 “GitHub 使用 ElasticSearch 搜索 20TB 的数据，包括 13 亿文件和 1300 亿行代码”。

2）维基百科：启动以 elasticsearch 为基础的核心搜索架构。  
3）SoundCloud：“SoundCloud 使用 ElasticSearch 为 1.8 亿用户提供即时而精准的音乐搜索服务”。  
4）百度：百度目前广泛使用 ElasticSearch 作为文本数据分析，采集百度所有服务器上的各类指标数据及用户自定义数据，通过对各种数据进行多维分析展示，辅助定位分析实例异常或业务层面异常。目前覆盖百度内部 20 多个业务线（包括 casio、云分析、网盟、预测、文库、直达号、钱包、风控等），单集群最大 100 台机器，200 个 ES 节点，每天导入 30TB + 数据。

### 4.2 我们也需要

实际项目开发实战中，几乎每个系统都会有一个搜索的功能，当搜索做到一定程度时，维护和扩展起来难度就会慢慢变大，所以很多公司都会把搜索单独独立出一个模块，用 ElasticSearch 等来实现。

近年 ElasticSearch 发展迅猛，已经超越了其最初的纯搜索引擎的角色，现在已经增加了数据聚合分析（aggregation）和可视化的特性，如果你有数百万的文档需要通过关键词进行定位时，ElasticSearch 肯定是最佳选择。当然，如果你的文档是 JSON 的，你也可以把 ElasticSearch 当作一种 “NoSQL 数据库”， 应用 ElasticSearch 数据聚合分析（aggregation）的特性，针对数据进行多维度的分析。

【知乎：热酷架构师潘飞】ES 在某些场景下替代传统 DB  
个人以为 Elasticsearch 作为内部存储来说还是不错的，效率也基本能够满足，在某些方面替代传统 DB 也是可以的，前提是你的业务不对操作的事性务有特殊要求；而权限管理也不用那么细，因为 ES 的权限这块还不完善。  
由于我们对 ES 的应用场景仅仅是在于对某段时间内的数据聚合操作，没有大量的单文档请求（比如通过 userid 来找到一个用户的文档，类似于 NoSQL 的应用场景），所以能否替代 NoSQL 还需要各位自己的测试。  
如果让我选择的话，我会尝试使用 ES 来替代传统的 NoSQL，因为它的横向扩展机制太方便了。

5. ES 的应用场景是怎样的？
----------------

### 5.1 通常我们面临问题有两个：

1）新系统开发尝试使用 ES 作为存储和检索服务器；  
2）现有系统升级需要支持全文检索服务，需要使用 ES。  
以上两种架构的使用，以下链接进行详细阐述。  
http://blog.csdn.net/laoyang360/article/details/52227541

### 5.2 一线公司 ES 使用场景：

1）新浪 ES 如何分析处理 32 亿条实时日志 http://dockone.io/article/505  
2）阿里 ES 构建挖财自己的日志采集和分析体系 http://afoo.me/columns/tec/logging-platform-spec.html  
3）有赞 ES 业务日志处理 http://tech.youzan.com/you-zan-tong-ri-zhi-ping-tai-chu-tan/  
4）ES 实现站内搜索 http://www.wtoutiao.com/p/13bkqiZ.html

6. 如何部署 ES？
-----------

### 6.1 ES 部署（无需安装）

1）零配置，开箱即用  
2）没有繁琐的安装配置  
3）java 版本要求：最低 1.7  
我使用的 1.8  
[root@laoyang config_lhy]# echo $JAVA_HOME  
/opt/jdk1.8.0_91  
4）下载地址：  
https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/zip/elasticsearch/2.3.5/elasticsearch-2.3.5.zip  
5）启动  
cd /usr/local/elasticsearch-2.3.5  
./bin/elasticsearch  
bin/elasticsearch -d(后台运行)

### 6.2 ES 必要的插件

必要的 Head、kibana、IK（中文分词）、graph 等插件的详细安装和使用。  
http://blog.csdn.net/column/details/deep-elasticsearch.html

### 6.3 ES windows 下一键安装

自写 bat 脚本实现 windows 下一键安装。  
1）一键安装 ES 及必要插件（head、kibana、IK、logstash 等）  
2）安装后以服务形式运行 ES。  
3）比自己摸索安装节省至少 2 小时时间，效率非常高。  
脚本说明：  
http://blog.csdn.net/laoyang360/article/details/51900235

7. ES 对外接口（开发人员关注）
------------------

### 1）JAVA API 接口

http://www.ibm.com/developerworks/library/j-use-elasticsearch-java-apps/index.html

### 2）RESTful API 接口

常见的增、删、改、查操作实现：  
http://blog.csdn.net/laoyang360/article/details/51931981

8.ES 遇到问题怎么办？
-------------

1）国外：https://discuss.elastic.co/  
2）国内：http://elasticsearch.cn/

参考：
---

[1] http://www.tuicool.com/articles/7fueUbb  
[2] http://zhaoyanblog.com/archives/495.html  
[3]《Elasticsearch 服务器开发》  
[4]《实战 Elasticsearch、Logstash、Kibana》  
[5]《Elasticsearch In Action》  
[6]《某 ES 大牛 PPT》

9、还有吗？
------

《死磕 Elasticsearch 方法论》：普通程序员高效精进的 10 大狠招！（免费完整版）  
https://blog.csdn.net/laoyang360/article/details/79293493  
——————————————————————————————————  
**更多 ES 相关实战干货经验分享**，请扫描下方【铭毅天下】微信公众号二维码关注。  
（每周至少更新一篇！）

![](https://img-blog.csdnimg.cn/img_convert/4baf67e1fa5e747dd5cc7ea584eca2a5.png)  
和你一起，**死磕 Elasticsearch**！  
——————————————————————————————————

2016-08-18 21:10 思于家中床前

作者：铭毅天下  
转载请标明出处，原文地址：  
http://blog.csdn.net/laoyang360/article/details/52244917  
如果感觉本文对您有帮助，请点击‘顶’支持一下，您的支持是我坚持写作最大的动力，谢谢！
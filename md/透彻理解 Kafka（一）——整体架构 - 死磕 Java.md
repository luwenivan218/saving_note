> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/5200958357)

> 从本系列开始，我将对 Kafka 这一流行的分布式消息中间件进行透彻分析。我之前在 MQ 系列中，对 RocketMQ 的核心原理进行过讲解，其实它俩在设计上有很多相似地方。Kafka 是 L

从本系列开始，我将对 Kafka 这一流行的分布式消息中间件进行透彻分析。我之前在 [MQ 系列](https://www.tpvlog.com/article/125)中，对 RocketMQ 的核心原理进行过讲解，其实它俩在设计上有很多相似地方。

Kafka 是 Linkedin 采用 Scala 语言开发的 一个 **多分区** 、 **多副本** 、 **基于 ZooKeeper** 的分布式消息系统，现己被捐献给 Apache 基金会 。 目前 Kafka 已经定位为一个分布式流式处理平台，它以高吞吐低延迟、可持久化、可水平扩展、支持流数据处理等多种特性而被广泛使用。

本章，我会从整体上对 Kafka 的基本架构和一些核心概念做阐述，便于读者初步了解 Kafka，后续章节再对细节作详述。

一、核心组件
------

在 Kafka 中，一共有四个核心组件： _**Zookeeper 集群**_ 、 _**Broker**_ 、 _**Producer**_ 、 _**Customer**_ ，它们之间的基本关系可以用下面这张图表示：

![](http://image.skjava.com/article/series/kafka/202307312118594601.png)

- **Zookeeper 集群：** 负责 Kafka 集群的元数据管理、控制器选举等操作；  
- **Producer：** 生产者，也就是发送消息的一方，负责创建消息，然后将其投递到 Kafka 中；  
- **Consumer** ：消费者，也就是接收消息的一方，消费者连接到 Kafka 上并接收消息，进行相应的业务逻辑处理；  
- **Broker** ：Broker 可以简单地看作一个独立的 Kafka 服务节点或 Kafka 服务进程实例，一个或多个 Broker 组成了 一个 Kafka 集群。一般而言，生产环境一台服务器运行一个 Broker 进程。

## 二、高性能

Kakfa 之所以具有_高吞吐低延迟_的特性，核心有四个点：

- 顺序读写磁盘；  
- MMap 内存映射技术；  
- 零拷贝技术；  
- 数据批处理。

![](http://image.skjava.com/article/series/kafka/202307312119009082.png)

### 2.1 顺序读写磁盘

Kafka 在持久化消息的时候，仅仅是将消息追加到日志文件的末尾，也就是 **磁盘顺序写** ，性能极高。

### 2.2 MMap 内存映射

**MMAP** 也就是 **内存映射文件** ，在 64 位操作系统中一般可以表示 20G 的数据文件，它的工作原理是直接利用操作系统的 Page 来实现文件到物理内存的直接映射，完成映射之后对物理内存的操作会被同步到硬盘上。

![](http://image.skjava.com/article/series/kafka/202307312119024743.png)

通过 **MMAP** 技术，进程可以像读写硬盘一样读写内存（逻辑内存），不必关心内存的大小，因为有虚拟内存兜底。这种方式可以极大的提升 I/O 能力，省去了数据从用户空间到内核空间复制的开销。

但是这里要注意：写到 **MMAP** 中的数据并没有真正写到硬盘，操作系统会定时进行刷盘操作，将 Page Cache 中的数据 flush 到磁盘。

> Kafka 提供了`producer.type`参数，来控制是否同步刷盘。

### 2.3 零拷贝

Consumer 在消费消息时，会请求 Kafka Broker 从磁盘文件里读取消息，然后通过网络发送出去，整个流程涉及到了 **零拷贝** ，如下图：

![](http://image.skjava.com/article/series/kafka/202307312119032404.png)

可以看到，Kafka Broker 利用了 Linux 的`sendfile`函数，直接把读取操作交给 OS，OS 会查看 Page Cache 中是否有数据，如果没有就从磁盘上读取并缓存到 Page Cache，如果有就直接将 OS Cache 里的数据拷贝给网卡引擎，这样就减少了上下文切换和数据复制的开销。

### 2.4 数据批处理

当 Consumer 需要消费数据时，首先想到的是消费一条，Kafka 发送一条。但实际上，Kafka 会把一批消息压缩存储，当消费者拉取数据时，实际上是拉到一批数据。比如说 100 万条消息压缩放到一个文件中可能就是 10M 的数据量，如果消费者和 Kafka 之间网络良好，10MB 大概 1 秒就能发送完，即 Kafka 每秒处理了 100 万条消息。

正是因为这种批处理的方式，Kafka 才有了极高的吞吐量。

三、可扩展
-----

### 3.1 数据分片

Kafka 中的消息以 **主题（Topic）** 为单位进行归类，生产者发送消息时，必须指定消息的主题，而消费者负责订阅主题并进行消费。

主题（Topic）是一个 **逻辑** 上的概念，它可以细分为多个分区，同一主题下的不同分区包含的消息是不同的，各个分区构成了一个完整的数据集，这实际上就是 **数据分散集群** 的模式。

**分区（Partition）** 在存储层面可以看作一个可追加的日志（ Log ）文件，消息在被追加到分区日志的时候都会分配一个特定的偏移量（ offset ）。 offset 是消息在分区中的唯一标识， Kafka 通过它来保证消息在分区内的顺序性，不过 offset 并不跨越分区，也就是说， **Kafka 保证的是分区有序而不是主题有序** 。

四、高可用
-----

### 4.1 多副本冗余

Kafka 为了维持集群的高可用，引入了 **副本（Replica）** 机制，每个分区都可以有多个副本，同一分区的不同副本中保存的消息是相同的。 Kafka 会选举出一个 Leader 副本，负责处理 **读写** 请求 ，其它 Follower 副本只负责与 Leader 进行消息同步。

Kafka 可以通过多副本机制实现故障的自动转移，当 Kafka 集群中某个 broker 挂掉时，各个副本重新选举 leader，最终保证服务可用。

我通过下面一张图来总结下主题（Topic）、分区（Partition）、副本（Replica）这三个概念。下图中，两个 Broker 进程组成了一个 Kafka 集群，然后创建了一个主题，这个主题有 3 个分区，每个分区有 2 个副本。Kafka 会自动均匀分布同一个分区的不同副本到不同的 Broker 上：

![](http://image.skjava.com/article/series/kafka/202307312119038925.png)

### 4.2 故障自动转移

Kafka 基于 Zookeeper 实现了故障节点的发现与自动转移。当一个分区的 Leader 副本挂掉后，其它 Follower 副本会选举出新的 Leader，并往 Zookeeper 中注册信息。

五、数据同步
------

### 5.1 ISR/OSR

每个分区的 follower 副本都会从 leader 同步消息数据，既然是同步，就一定有滞后性。这里引申出 Kafka 中的三个概念：

*   **AR ( Assigned Replicas ）：** 在 Kafka 中，分区中的所有副本（包含 Leader）统称为 AR；
*   **ISR (On-Sync Replicas)：** 所有与 Leader 保持一定程度同步的副本（包括 leader 在内）组成 ISR；
*   **OSR ( Out-of-Sync Replicas）：** 与 Leader 副本同步滞后过多的副本（不包括 leader 副本）组成 OSR 。

通过上面的描述可以看出： **AR = ISR + OSR** 。

Leader 负责维护和跟踪 ISR 集合中所有 follower 副本的滞后状态（Leader 会维护每个 Follower 的 LEO，Follower 来拉取消息时会带上自己的 LEO），当 follower 副本落后太多或失效时，leader 会把它从 ISR 集合中剔除，转移到 OSR。默认情况下， 当 leader 副本发生故障时，只有在 ISR 集合中的副本才有资格被选举为新的 leader。

### 5.2 HW 和 LEO

那么，到底怎么样才算同步滞后过多呢？这就又涉及到 _HW(High Watermark，高水位)_ 和 *LEO(Log End Offset)* 两个概念了。

*   **HW(High Watermark，高水位)** ： 标识了 一个特定的消息偏移量（ offset ），消费者只能拉取到这个 offset 之前的消息；
*   **LEO(Log End Offset)** ：标识了当前日志文件中的下一条待写入消息的 offset。

我通过下面这张图来讲解上述的概念：

![](http://image.skjava.com/article/series/kafka/202307312119046946.png)

上图是某个分区的某个副本中的日志文件，这个日志文件中一共有 9 条消息，第一条消息的 offset=0，最后一条消息的 offset=8，offset 为 9 的位置用虚线框表示，代表下一条待写入的消息。

上述日志文件的 HW=6，表示 Consumer 只能拉取到 offset 在 0 至 5 之间的消息，offset 为 6~8 之间的消息对 Consumer 是不可见的；offset = 9 的位置即为当前日志文件的 LEO，所以 LEO 的大小相于当前日志中的最后一条消息的 offset +1。

ISR 集合中的每个副本都会维护自身的 LEO ，而 **ISR 集合中最小的 LEO 即为这个分区的 HW** ，对于消费者而言，只能消费 HW 之前的消息。

> Kafka 之所以引入 ISR 机制，是为了有效地权衡 **数据可靠性** 和 **性能** 之间的关系，因为完全同步复制影响性能，完全异步复制又可能因为 follower 落后太多宕机后导致消息丢失。我后面章节会再对 ISR 机制进行更详细的分析。

六、总结
----

本章，我先从整体上对 Kafka 的核心功能和架构进行了分析，从后续章节开始，我将对 Kafka 每一个组件进行深入剖析。
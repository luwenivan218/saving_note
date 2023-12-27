> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/4360611674)

> 我在《整体架构》这一章中，已经比较清楚的介绍了 Kafka 的 ISR 机制。每个分区中的 Leader 会负责维护和跟踪 ISR 集合中所有 follower 的滞后状态（Leader 会维护每个

我在[《整体架构》](https://www.tpvlog.com/article/279)这一章中，已经比较清楚的介绍了 Kafka 的 ISR 机制。每个分区中的 Leader 会负责维护和跟踪 ISR 集合中所有 follower 的滞后状态（Leader 会维护每个 Follower 的 LEO，Follower 来拉取消息时会带上自己的 LEO），当 follower 副本落后太多或失效时，leader 会把它从 ISR 集合中剔除，转移到 OSR。默认情况下， 当 Leader 发生故障时，只有在 ISR 集合中的副本才有资格被选举为新的 leader。

但是，在一些极端情况下，ISR 机制可能会引发一些问题，本章我就对这些问题进行一一分析。

一、存在的问题
-------

在老版本的 Kafka 中，ISR 机制可能引发 **数据丢失** 和 **数据不一致** 问题。

### 1.1 数据丢失

我们先来看第一种情况——数据丢失。什么情况下会出现数据丢失呢？

举个例子，假设某分区一共有 2 个副本，一个 Leader，一个 Follower。生产者（Producer）的`min.insync.replicas`参数设置为 1，也就是说，生产者发送消息给 Leader，Leader 写 Log 成功后，生产者就会认为发送成功了：

1.  生产者先发送了 1 条消息给 Leader，此时 Leader 的信息：`LEO = 1, HW = 0`；
2.  Follower 发送 Fetch 请求同步数据，上送`LEO = 0`；
3.  Leader 会维护所有 Follower 的 LEO 信息，取最小值作为 HW，此时`HW = 0`并响应；
4.  Follower 同步到消息，更新自身信息：`LEO = 1, HW = 0`；
5.  Follower 再次发送 Fetch 请求，上送`LEO = 1`；
6.  Leader 维护所有 Follower 的 LEO 信息，取最小值作为 HW，此时`HW = 1`并响应；
7.  正常情况下，Follower 会接收到响应，更新自身信息：`LEO = 1, HW = 1`；

上述第 7 步，如果 Follower 还没接收到响应就挂掉了，此时它的`LEO = 1, HW = 0`，那么 Follower 重启后，会依据 HW 来调整 LEO，LEO 会自动被调整为 0，也就是说已经同步的消息会被从日志文件里删除。

接着，如果此时 Leader 也挂了，然后 Follower 当选为新 Leader，由于它的`HW = 0`，那么原来的 Leader 同步到数据后，会截断自己的日志，发生数据丢失。

### 1.2 数据不一致

我们再来看第二种情况——数据不一致，假设某分区一共有 2 个副本，一个 Leader，一个 Follower。生产者（Producer）的`min.insync.replicas`参数设置为 1，也就是说，生产者发送消息给 Leader，Leader 写 Log 成功后，生产者就会认为发送成功了：

1.  生产者发送了 2 条消息给 Leader，此时 Leader 的信息：`LEO = 2, HW = 0`；
2.  Follower 发送 Fetch 请求同步数据，先同步第一条，上送`LEO = 0`；
3.  Leader 会维护所有 Follower 的 LEO 信息，取最小值作为 HW，此时`HW = 0`并响应；
4.  Follower 同步到消息，更新自身信息：`LEO = 1, HW = 0`；
5.  Follower 再次发送 Fetch 请求，同步第二条消息，上送`LEO = 1`；
6.  Leader 维护所有 Follower 的 LEO 信息，取最小值作为 HW，此时`HW = 1`并响应；
7.  正常情况下，Follower 会接受到响应，更新自身信息：`LEO = 1, HW = 1`；

上述第 7 步完成后，假设 Leader 宕机了，Follower 成为新的 Leader，此时它的`HW = 1`，就一条数据，然后生产者又发了一条数据给新 leader，此时`HW = 2`，但是第二条数据是新的数据。接着老 leader 重启变为 follower，这个时候发现两者的 HW 都是 2。这个时候他俩数据是不一致的，本来合理的应该是新的 follower 要删掉自己原来的第二条数据，跟新 leader 同步的，让他们俩的数据一致，但是因为 HW 一样，所以就不会截断数据了。

二、解决方案
------

上述数据丢失的场景是一种非常极端的场景，一般只会在`0.11.x`版本之前出现。`0.11.x`版本时，Kafka 引入了 _**Leader Epoch**_ 机制。所谓 Leader Epoch，大致可以理解为每个 Leader 的版本号，以及自己是从哪个 offset 开始写数据的，类似`[epoch = 0, offset = 0]`。

三、ISR 工作原理
----------

Kafka 到底是如何维护 ISR 列表的？什么样的 Follower 才有资格放到 ISR 列表里呢？

### 3.1 replica.lag.max.messages

在`0.9.x`之前的版本里，Kafka Broker 有一个核心的参数：`replica.lag.max.messages`，默认值 4000，表示如果 Follower 落后 Leader 的消息数量超过了这个参数值，就认为 Follower 是 _out-of-sync_，就会从 ISR 列表里移除。

我们通过一个例子来理解下，假设一个分区有 3 个副本：一个 Leader，两个 Follower，配置是：`replica.lag.max.messages = 3`，`min.insync.replicas = 2`，`ack = -1`。

也就是说，生产者发送一条消息后，当 ISR 中至少存在 2 个副本（包含 Leader）且这些副本都写成功后，生产者才会收到写入成功的响应。那么每个 Follower 会不断地发送 Fetch 请求拉取消息（上送自己的 LEO），此时 Kafka 会判断 Leader 和 Follower 的 LEO 相差多少，如果差的数量超过了`replica.lag.max.messages`参数值，就会把 Follower 踢出 ISR 列表。

**存在的问题**  
`replica.lag.max.messages`这一机制，在瞬间高并发访问的情况下会出现问题：比如 Leader 瞬间接收到几万条消息，然后所有 Follower 还没来得及同步过去，此时所有 follower 都会被踢出 ISR 列表，然后同步完成之后，又会再被加入 ISR 列表。

也就是说，这种依靠同步消息数量来判断 Follower 是否落后的机制，可能会导致在系统高峰时期，Follower 被频繁踢出 ISR 列表，然后再回到 ISR 列表，这种操作是完全无意义的。

### 3.2 replica.lag.time.max.ms

Kafka 从`0.9.x`版本开始，引入了`replica.lag.max.ms`参数，默认值 10 秒，表示如果某个 Follower 的 LEO 一直落后 Leader 超过了 10 秒，那么才判定这个 Follower 是 _out-of-sync_，就会从 ISR 列表里移除。

这样的话，即使出现瞬间的流量洪峰，一下子导致几个 Follower 都落后了不少数据，但是只要在限定的时间内尽快追上来，别一直落后，就不会认为是 _out-of-sync_。

* * *

上面就是 ISR 的核心工作机制了，一般导致 Follower 同步数据较慢的原因主要有以下三种：

1.  Follower 所在机器的性能变差，比如网络负载过高，I/O 负载过高，CPU 负载过高等等；
2.  Follower 所在机器的 Kafka Broker 进程出现卡顿，最常见的就是发生了 Full GC；
3.  动态增加了 Partition 的副本，此时新加入的 Follower 会拼命从 Leader 上同步数据，但是这个是需要时间的，所以如果参数配置不当，会导致生产者 hang 等待同步完成。
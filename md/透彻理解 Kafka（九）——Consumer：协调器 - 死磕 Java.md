> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/8034274402)

> 每个消费者组都会选择一个 Broker 作为自己的 Coordinator，这个 GroupCoordinator 协调器负责监控这个消费组里的各个消费者的心跳，判断它们是否宕机，如果宕

转

 2023-07-31  阅读 (176)  点赞 (0)

原文地址：https://www.tpvlog.com/article/279

每个消费者组都会选择一个 Broker 作为自己的 Coordinator，这个 GroupCoordinator 协调器负责监控这个消费组里的各个消费者的心跳，判断它们是否宕机，如果宕机则进行 Rebalance。

那么，这里就要思考几个问题：

1.  消费者组应该选择哪一个 Broker 作为 GroupCoordinator？
2.  GroupCoordinator 是如何为它的消费者组进行 Rebalance 的？

一、GroupCoordinator
------------------

### 1.1 选择 GroupCoordinator

我们先来看下消费者组是如何选择 GroupCoordinator 的。

我之前说过，每个 Consumer 在提交 offset 时，会将 offset 提交到`__consumer_offsets`这个内部 Topic 的某个分区上，默认分区数是 50，事实上， **Consumer 提交 offset 的那个 Leader 分区所在的 Broker 就是 GroupCoordinator** 。

举个例子，假设有一个消费者组，`groupId = membership-consumer-group`：

1.  Consumer 启动的时候，会对 groupId 进行 hash 运算，然后与`__consumer_offsets`的分区数取模，得到一个数值；
2.  消费者组下的所有 Consumer 在提交 offset 时，会提交到这个数值对应的`__consumer_offsets`的那个分区；
3.  最终，上述 Leader 分区所在的 Broker 就是这个消费者组的 GroupCoordinator，组里的消费者会维护 Socket 连接与这个 Broker 进行通信。

### 1.2 分区分配

确认 GroupCoordinator 后，我们再来看下分区分配是如何进行的。

1.  初始时，消费者组内的每个 Consumer 都会发送`JoinGroup`请求到 GroupCoordinator；
2.  GroupCoordinator 会从消费组内选择一个 Consumer 作为 Leader，然后把消费者组的情况发送给这个 Leader；
3.  消费者 Leader 会负责制定分区方案，并通过`SyncGroup`请求告知 GroupCoordinator；
4.  最后，GroupCoordinator 会把分区方案下发给所有 Consumer，各个 Consumer 就会跟指定 Leader 分区所在的 Broker 建立 Socket 连接，开始消费消息。

二、Reblance 策略
-------------

当消费者组中的某个 Consumer 宕机或者增减分区时，GroupCoordinator 会负责分区重分配，也就是所谓的 **reblance** 。在 reblance 期间，消费者组会变得 **不可用** 。另外，reblance 可能会引发 “重复消费” 的问题，比如 Consumer 消费完某个分区中的一部分消息后还没有来得及提交 offset，此时发生了 reblance，然后这个分区被分配给了消费组内的另一个 Consumer，这样原来被消费完的那部分消息又会被新的 Consumer 重新消费一遍，即发生了重复消费。

Kafka 一共提供了三种 Rebalance 的策略： _**Range**_ 、 _**Round-Robin**_ 、 _**Sticky**_ 。

### 2.1 Range 策略

Range 策略就是按照 Partition 的序号范围进行分配，也是 **默认策略** 。

举个例子，某个主题有 8 个分区：partition0-partition7。那么分区 partition0-2 分配给 Consumer1，partition3-5 给一个 Consumer2，partition6-7 给一个 consumer3。

### 2.2 Round-Robin 策略

Round-Robin 策略，就是轮询分配。

举个例子，某个主题有 8 个分区：partition0-partition7。那么 partiton0 给 Consumer1，partiton1 给另一个 Consumer2，依此类推…… 最后 Consumer1 分配到 partiton0、3、6，Consumer2 分配到 partition1、4、7，Consumer3 分配到 partition2、5。

### 2.3 Sticky 策略

Range 策略和 Round-Robin 策略都存在一个问题，就是发生 Rebalance 的时候会导致分区被频繁重新分配。

举个例子，比如 Consumer2 挂掉了，那么会导致原本分配给 Consumer1 和 Consumer3 的分区也要被重新分配，这种分配很多时候是没必要的。

所以，Kafka 又新增了一种 **Sticky 策略** ，就是说在发生 Rebalance 时，尽量让原本属于这个 Consumer 的分区不变动，再把多余的分区均匀分配出去，这样就能尽可能维持原来的分区分配策略。
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/7175339780)

> Kafka 中存在大量的延时操作，比如延时生产、延时拉取和延时删除等。Kafka 并没有使用 JDK 自带的 Timer 或 DelayQueue 来实现延时的功能，而是基于 & nbsp; 时间轮

Kafka 中存在大量的延时操作，比如延时生产、延时拉取和延时删除等。Kafka 并没有使用 JDK 自带的 Timer 或 DelayQueue 来实现延时的功能，而是基于 **时间轮** 的概念实现了一个用于延时功能的定时器（ SystemTimer ）。

JDK 中 Timer 和 DelayQueue 的插入和删除操作的平均时间复杂度为 *O(nlogn）* 并不能满足 Kafka 的高性能要求，而基于时间轮的插入和删除操作的时间复杂度为 _**0(1)**_ 。时间轮的设计思想在很多开源框架中都有应用，比如 Netty 、ZooKeeper 等等。

一、设计思想
------

### 1.1 基本结构

Kafka 中的时间轮（ TimingWheel ）是一个存储定时任务的 **环形队列** ， 底层采用数组实现，数组中的每个元素可以存放一个定时任务列表（ TimerTaskList ）。 TimerTaskList 是一个环形的 **双向链表** ，链表中的每一项就是定时任务项（ TimerTaskEntry ），其中封装了真正的定时任务（ TimerTask ）。

![](http://image.skjava.com/article/series/kafka/202307312120308261.png)

如上图，时间轮由多个时间格组成， 每个时间格就是基本时间跨度`tickMs` 。时间轮的格数是固定的，用`wheelSize`表示，那么整个时间轮的总体时间跨度`interval = tickMs * wheelSize`。

举个例子：若时间轮的 tickMs = 1ms ，wheelSize = 20 ，那么总体时间跨度 interval 就是 20ms，可以用来存放延时时间在 0-20ms 内的定时任务。

1.  初始情况下，表盘指针 currentTime 指向时间格 0；
2.  有一个定时为 2ms 的任务插进来会存放到时间格为 2 的 TimerTaskList 中；
3.  随着时间推移 ， 指针 currentTime 不断向前推进，过了 2ms 之后，当到达时间格 2 时，就需要将时间格 2 对应的 TimeTaskList 中的任务进行相应的到期操作；
4.  此时若又有一个定时为 8ms 的任务插进来，则会存放到时间格 10 中，currentTime 再过 8ms 后会指向时间格 10 ；
5.  如果同时有一个定时为 19ms 的任务插进来，新来的 TimerTaskEntry 会复用原来的 TimerTaskList，所以它会插入原本己经到期的时间格 1 。

以上就是整个时间轮的运行机制，总之，整个时间轮的总体跨度是不变的，随着指针 currentTime 的不断推进，当前时间轮所能处理的时间段也在不断后移，总体时间范围在 currentTime 和 currentTime+interval 之间 。

### 1.2 多层级

那么这里有个问题，若整个时间轮的总体时间跨度`interval = tickMs * wheelSize`，比如 20ms，那么对于定时为 350ms 的任务该如何处理？此时已经超出了时间轮能表示的时间跨度。

Kafka 为此引入了 **层级时间轮** 的概念，当任务的到期时间超过了当前时间轮所表示的时间范围时，就会尝试添加到上层时间轮中。比如对于 20ms 跨度的时间轮，它的上级是`interval = 20 * 20 = 400ms`，对于 400ms 跨度的时间轮，它的上级是`interval = 400 * 20 = 8000ms`，以此类推：

![](http://image.skjava.com/article/series/kafka/202307312120387192.png)

举个例子，对于 450ms 的定时任务：

1.  首先，会升级存放到第三层时间轮中，被插入到第三层时间轮的时间格 1 所对应的 TimerTaskList；
2.  随着时间的流逝，当此 TimerTaskList 到期之时，原本定时为 450ms 的任务还剩下 50ms 的时间，还不能执行这个任务的到期操作；
3.  于是执行 **时间轮降级** ，将剩余时间为 50ms 的定时任务重新提交到第二层到期时间为 ［40ms,60ms）的时间格中；
4.  再经历 40ms 之后，此时这个任务又被 “察觉”，不过还剩余 10ms ，所以还要降级一次，放到第一层时间轮的［ 10ms, 11ms）的时间格中；
5.  最后，经历 l0ms 后，此任务真正到期，最终执行相应的到期操作。

### 1.3 DelayQueue

Kafka 使用了 JDK 中的 DelayQueue 来推进时间轮。具体做法是将每个使用到的 **TimerTaskList** 都加入到一个 DelayQueue 中，DelayQueue 会根据 TimerTaskList 的超时时间来排序，最短超时时间的 TimerTaskList 会被排在 DelayQueue 的队头。

然后，Kafka 中会有一个线程`ExpiredOperationReaper`来获取 DelayQueue 中到期的任务列表，这样这个线程既可以对任务进行时间轮降级，也可以直接执行这个任务。

总结一下，Kafka 中的时间轮（ TimingWheel ）专门用来插入和删除延时任务，而 DelayQueue 则专门负责时间推进。
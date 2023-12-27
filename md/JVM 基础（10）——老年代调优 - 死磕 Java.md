> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/8789937013)

> 一、简介上一章中，我们通过一个实际案例讲解了如何进行新生代的 JVM 参数调优，本章我们来继续分析这个问题，在上一章优化好的背景下，讲解如何进行老年代的调优。我们先来回顾下，每隔 2

一、简介
----

上一章中，我们通过一个实际案例讲解了如何进行新生代的 JVM 参数调优，本章我们来继续分析这个问题，在上一章优化好的背景下，讲解如何进行老年代的调优。

我们先来回顾下，每隔 20s，新生代经历 Minor GC 后，会产生 100MB 存活对象：

![](http://image.skjava.com/article/series/jvm/202308102128179631.png)

JVM 的参数配置如下：

`-Xms3072M -Xmx3072M -Xmn2048M -Xss1M -XX:PermSize=256M -XX:MaxPermSize=256M -XX:SurvivorRatio=8 -XX:MaxTenuringThreshold=5 -XX:PretenureSizeThreshold=1M -XX:+UseParNewGC -XX:+UseConcMarkSweepGC`

二、老年代调优
-------

### 2.1 何时进入老年代

#### 晋升年龄

首先，由于我们设置的晋升年龄为 5，所以躲过 5 次 Minor GC 的存活对象会进入老年代，这些对象一般是系统的业务逻辑组件，不会很大，基本也就几十 MB，它们会长期存活在老年代中：

![](http://image.skjava.com/article/series/jvm/202308102128185562.png)

#### 大对象

按照我们的 JVM 参数，超过 1MB 的大对象会直接进入老年代。但是我们的系统中假设是不存在这种大对象的，所以可以忽略这块内容。

#### Survivor 空间不足

Minor GC 过后，存活对象大小可能超过 Suvivor 空间大小（200MB），虽然我们之前已经对此进行了优化，调整过 S 区的大小，但是在一些突发场景下，比如大促期间，仍然可能出现超过 200MB 的存活对象。

我们现在就假设每隔 5min，就会有一批超过 200MB 的存活对象进入老年代：

![](http://image.skjava.com/article/series/jvm/202308102128192293.png)

那么多久会触发 Full GC 呢？之前分析过，触发 Full GC 一共有以下几种情况（排除 JDK1.6 及以前的情况）：

1.  老年代可用内存 < 历代晋升老年代的平均对象大小；
2.  老年代可用内存 < 本次晋升老年代的存活对象大小；
3.  设置了`-XX:CMSInitiatingOccupancyFaction`参数，当老年代内存占用达到该比例时，也会触发 Full GC；

其实在生产环境下，只要对新生代进行过上一章节的优化，那对象进入老年代的速度是非常慢的。很可能在系统运行了大约半小时~ 1 小时之后，才会有接近 1G 的对象进入老年代，而且上述 Full GC 的情况一般需要在老年代近乎快占满的情况下才可能触发。

#### Concurrent Mode Failure

经过前面的推算，我们基本可以知道，系统运行 1 小时后，老年代的存活对象大概有 900MB，此时就会触发一次 Full GC：

![](http://image.skjava.com/article/series/jvm/202308102128199074.png)

但是由于老年代只剩下约 100MB 的可用空间，所以在 CMS 进行并发清理的时候，如果有新的对象（假设大约 200MB）进入老年代，就会出现 “Concurrent Mode Failure”：

![](http://image.skjava.com/article/series/jvm/202308102128205305.png)

当出现 “Concurrent Mode Failure” 时，JVM 会立即进入“Stop the World”，然后切换成 Serial Old 垃圾回收器，采用单线程方式进行老年代的垃圾回收，回收掉 900MB 对象后，再恢复系统运行：

![](http://image.skjava.com/article/series/jvm/202308102128213496.png)

> Concurrent Mode Failure 出现的概率其实是非常小的，首先必须是在 CMS 触发 Full GC 期间，其次还需要在此期间有对象晋升到老年代，且这些对象的大小要大于老年代可用空间。

经过上述调优后，JVM 的参数配置如下：

`-Xms3072M -Xmx3072M -Xmn2048M -Xss1M -XX:PermSize=256M -XX:MaxPermSize=256M -XX:SurvivorRatio=8 -XX:MaxTenuringThreshold=5 -XX:PretenureSizeThreshold=1M -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFaction=92`

### 2.2 内存碎片整理

在 CMS 完成 Full GC 后，会对老年代进行内存碎片整理。我们可以通过配置来设置经过多少次 Full GC 后进行一次内存碎片整理。

但是，通过上述分析，我们知道，Full GC 的频率本身并不高，在高峰时期也就一个多小时一次，高峰过去后，很可能几小时才会触发一次 Full GC。所以，对于内存碎片整理，保持默认值就可以，即每次 Full GC 完成后进行一次整理。

经过上述调优后，JVM 的参数配置如下：

`-Xms3072M -Xmx3072M -Xmn2048M -Xss1M -XX:PermSize=256M -XX:MaxPermSize=256M -XX:SurvivorRatio=8 -XX:MaxTenuringThreshold=5 -XX:PretenureSizeThreshold=1M -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFaction=92 -XX:+UseCMSCompactAtFullCollection -XX:CMSFullGCsBeforeCompaction=0`

三、总结
----

本章，我们讲解了针对老年代的调优，其实可以看到，老年代调优的根本目的就是降低 Full GC 频次，而前提就是对 Minor GC 进行优化，以减少对象进入老年代。

对于很多 Java 系统而言，只要对系统运行期间的内存模型做好预估，然后合理分配 JVM 的各内存区域，尽量让 Minor GC 后的存活对象留在 Survivor 不要去老年代，那么即使其余的 JVM 参数不做优化，系统性能基本上也能满足要求。
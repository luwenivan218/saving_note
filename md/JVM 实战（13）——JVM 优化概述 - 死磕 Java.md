> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1994752242)

> 一、简介本章，我们先来对系统运行过程中可能会遇到的各种 JVM 性能问题作个概述，以此为引子，作为后续实战篇的铺垫。JVM 性能优化其实就是针对 JVM 内存分配、参数设置进行优化，目的

一、简介
----

本章，我们先来对系统运行过程中可能会遇到的各种 JVM 性能问题作个概述，以此为引子，作为后续实战篇的铺垫。

JVM 性能优化其实就是针对 JVM 内存分配、参数设置进行优化，目的是减少 GC 次数，避免对象频繁进入老年代。所以，我们来先来回顾下新生代和老年代的垃圾回收过程，并看下可能会引发的各种 JVM 性能问题。

在正式开始之前，我先给出一份 JVM 调优模板，这份模板基本上涵盖了 JVM 调优所需的所有核心参数，后续我们所有的调优也会围绕它展开：  
`-Xms4096M -Xmx4096M -Xmn3072M -Xss1M -XX:MetaspaceSize=256M -XX:MaxMetaspaceSize=256M -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFaction=92 -XX:UseCMSCompactAtFullCollection -XX:CMSFullGCsBeforeCompaction=0 -XX:CMSParallellInitialMarkEnabled -XX:CMSScavengeBeforeRemark -XX:DisableExplicitGC -XX:PrintGCDetail -Xloggc:gc.log -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath:/usr/local/app/oom.hprof`

上述有些参数看不懂没关系，我们在后续的各个实战章节中都会陆续提到，这里先简要说下：  
首先，JVM 中各块内存区域大小的分配是根据系统运行模型来配置的，然后是 ParNew 和 CMS 这两种垃圾回收器的配置，特别注意 CMS 的一些参数，主要是提升 CMS 的效率和性能，还有就是打印 GC 日志，日志可以借助后续章节会讲到的 jstat 工具进行分析，最后两个参数是在发生内存溢出异常时，自动 dump 出内存快照，然后就可以通过 MAT 等工具进行分析了。

二、JVM 性能问题
----------

JVM 运行时，最核心的区域就是 Java 堆内存，这里会存放我们系统创建出来的各种对象。而且堆内存通常划分为新生代和老年代，新生代存放新创建出来的各种对象。所以，我们先来看下新生代 GC 会有哪些问题。

### 2.1 新生代 GC

随着系统的不断运行，新生代中的对象会越来越多，直到快被塞满。此时会根据 GC Roots 去寻找存活的对象。GC Roots 一般是类静态变量或方法的局部变量。由于我们创建对象最多的地方是在方法内，方法运行完毕，局部变量就没有了，所以新生代中这种对象其实占了 99%，这也是新生代对象存活率低的原因。

![](http://image.skjava.com/article/series/jvm/202308102131263721.png)

新生代进行 Minor GC 时，会采用 **复制算法** ，将 Eden 区和一块 Survivor 区的存活对象复制到另一块 Survivor 区，然后清空 Eden 和之前的 Survivor。同时，新生代 GC 期间会”Stop the World“，即只允许 GC 线程进行回收工作，其它工作线程都会被挂起。

![](http://image.skjava.com/article/series/jvm/202308102131271852.png)

假设一次新生代的 GC 需要 20ms，那么此时对于用户发送的请求，这 20ms 内是无法处理的，系统会卡顿 20ms。但是新生代的 GC 速度非常快，所以只要不频繁 GC，其实对系统是没什么影响的。所以，新生代 GC 其实没什么好调优的，只要多分配点堆内存，保证 Survivor 区空间充足，那么低峰时期一般几小时才有一次新生代 GC，高峰期也最多几分钟一次新生代 GC。

_那么，什么时候新生代 GC 会对系统产生很大的影响呢？_

当系统部署在大内存机器上时，比如 32 核 64G 的机器，新生代的 Eden 区可能有 32G 以上的内存。

此时，如果系统的负载特别高（比如部署了 Kafka、Elasticsearch 每秒处理上万的请求），那么可能导致 Eden 区的几十 G 空间在短短几分钟内被塞满。而此时进行新生代 GC 会停止系统的运行，由于新生代空间非常大，GC 时间会很长，可能长达数秒钟。

对于一个高负载高并发的系统，每隔几分钟就停顿几秒去进行新生代 GC，是不可接受的。

解决方案一般就是使用 G1 垃圾回收器，因为 G1 可以设置一个预期停顿时间（比如 20ms），那么 G1 基于它的 Region 内存划分原理，就可以在运行一段时间之后，回收一部分 Region，控制时间在 20ms 内，然后再运行再回收。

所以， **G1 天生就适合在这种大内存的机器上运行，可以完美解决大内存垃圾回收时间过长的问题。**

### 2.2 老年代 GC

之前给大家讲过新生代中对象晋升到老年代的几个可能条件：

*   年龄太大
*   符合动态年龄判断规则
*   大对象
*   新生代 GC 后存活的对象放不下 Survivor 区

上述条件中，关键是动态年龄判断和对象放不下 Survivor 区，从而导致大量对象频繁进入老年代：

![](http://image.skjava.com/article/series/jvm/202308102131284063.png)

老年代 GC 非常耗时，无论是 CMS 还是 G1。通常老年代 GC 要比新生代 GC 慢十倍以上，所以针对老年代 GC 的优化还是要先从新生代 GC 入手，合理分配内存和设置 JVM 参数，尽量让对象不要频繁进入老年代。

三、各种 GC 分类
----------

在基础篇，我们介绍过各种 GC 类型，Minor GC、Full GC、Mixed GC、Young GC 等等。本节我们就来统一梳理下。

### 3.1 Minor GC/Young GC

当新生代的 Eden 区域被占满后，实际就需要触发新生代的 GC，这就是所谓的”Minor GC“，也可以称之为”Young GC“。后续章节，我们统一用 Young GC 指代新生代的 GC。

**触发时机：** 新生代的 Eden 区域被占满后。

### 3.2 Full GC/Old GC

Old GC 是仅仅针对老年代区域进行垃圾回收。而 Full GC 则是针对新生代、老年代、永久代的全体内存空间进行垃圾回收。后续章节，我们统一用 Old GC 指代老生代的 GC。

**触发时机：** 老年代空间不够。具体时机可细分为以下几种：

1.  进行 Young GC 之前：如果老年代的连续可用内存空间 < 新生代历次晋升的平均大小，此时先触发一次 Old GC 清理老年代，然后再执行 Young GC。
2.  进行 Young GC 之后：如果存活对象要进入老年代，但是老年代的连续可用内存空间 < 存放对象的大小，此时必须触发一次 Old GC。
3.  老年代的内存使用率超过了 92%，此时也会触发 Old GC。

> 在很多 JVM 的实现机制里，当上述几种条件达到时，实际触发的其实是 Full GC，这个 Full GC 会包含 Young GC、Old GC 和永久代 GC。

### 3.3 Mixed GC

Mixed GC 是 G1 垃圾回收器中特有的概念，在 G1 中，一旦老年代占据了 Java 堆内存的 45%，就会触发 Mixed GC，此时对新生代和老年代都进行垃圾回收。

**触发时机：** G1 特有，老年代空间占据到 Java 堆内存的 45%。

### 3.4 永久代 GC

永久代一般存放着类信息、常量池等等。在进行 Full GC 的时候，会顺带对永久代进行 GC，一般来说永久代里的东西是不需要回收的，如果永久代真的满了，回收之后也没腾出足够的空间来，就会抛出 OOM 异常。
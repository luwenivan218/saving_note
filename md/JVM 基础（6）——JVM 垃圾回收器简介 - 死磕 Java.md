> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1760896723)

> 一、简介在新生代和老年代进行垃圾回收的时候，都需要使用回收器进行回收，不同的 JVM 垃圾回收器会有所不同，不同区域一般也采用不同的垃圾回收器。JVM 常见的垃圾回收器有以下几种，我

一、简介
----

在新生代和老年代进行垃圾回收的时候，都需要使用回收器进行回收，不同的 JVM 垃圾回收器会有所不同，不同区域一般也采用不同的垃圾回收器。JVM 常见的垃圾回收器有以下几种，我们先来简要看下，后续会针对每一种 GC 专门详细讲解：

#### Serial/Serial Old

Serial/Serial Old 收集器是最基本也是最古老的垃圾收集器，它是一个单线程收集器，并且在它进行垃圾收集时，必须暂停所有用户线程，也就是发生 “Stop the World”。一般 JVM 都不再使用该收集器。

#### ParNew

ParNew 收集器是 Serial 收集器的多线程版本。新生代并行回收，采用复制算法，老年代串行回收，采用标记整理算法。所以，该收集器一般只用于新生代。

#### CMS

CMS（Current Mark Sweep）收集器，目标是使回收停顿时间最短，也是多线程机制，采用标记整理算法，该回收器一般用于老年代，生产环境上也经常会使用该垃圾回收器与其它 GC 搭配使用。

#### G1

G1 最大的特点是没有物理上的新生代和老年代，它们是逻辑的，G1 将整个 Java 堆划分为多个大小相等的独立区域（Region），对新生代和老年代进行统一收集，并且采用了更加优秀的算法和设计机制。

二、Stop the World
----------------

### 2.1 何谓 Stop the World

上一节，我们提到过 JVM 在进行垃圾回收时，会挂起除 GC 线程以外的所有其它线程，这就会导致系统出现卡顿，这就是 JVM 所谓的 _**Stop the World**_ 机制。

假设当前 JVM 新生代的状态如下，Eden 区已被填满，Survivor1 区存放着上一次 Minor GC 后的存活对象，这时候即将触发 Minor GC，由垃圾回收线程使用垃圾回收器（新生代一般使用 ParNew，采用复制算法），使用特定的垃圾回收算法进行回收，如下图：

![](http://image.skjava.com/article/series/jvm/202308102127502831.png)

回收时，Eden 区和 Survivor1 区的存活对象会被转移到 Survivor2 区，接着 Eden 和 Survivor1 中的垃圾对象都会被回收掉：

![](http://image.skjava.com/article/series/jvm/202308102127512602.png)

JVM 在进行垃圾回收时，会让我们的系统暂停，不再创建新的对象，同时让 GC 线程尽快完成垃圾回收的工作——即标记和转移存活对象：

![](http://image.skjava.com/article/series/jvm/202308102127521033.png)

一旦本轮 GC 结束，就可以恢复我们的系统程序，继续在 Eden 区创建新的对象了，如下图：

![](http://image.skjava.com/article/series/jvm/202308102127530824.png)

三、总结
----

上面就是 _**Stop the World**_ 的整体流程，Stop the World 会导致客户端的请求出现卡顿，短则几百毫秒，长则几秒甚至几分钟。所以，无论是 Minor GC 还是 Full GC，都要避免频率过高，这也是使用 JVM 过程中最需要关注和优化的地方。

从下一章开始，我们将详细讲解常见的三种垃圾回收器的工作原理，不同回收器 _**Stop the World**_ 的流程也有所区别，GC 的核心目标其实就是降低 _**Stop the World**_ 的总体时间，这也是 JVM 不断演化的终极思路。
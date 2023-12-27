> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1511189794)

> 一、简介本章，我们将讲解一个使用 Jetty 作为 Web 容器的应用的内存溢出问题，该内存溢出问题发生的区域是堆外内存，主要原因是 JVM 内存区域划分不合理，我们先来看下系统的背景。1

一、简介
----

本章，我们将讲解一个使用 Jetty 作为 Web 容器的应用的内存溢出问题，该内存溢出问题发生的区域是堆外内存，主要原因是 JVM 内存区域划分不合理，我们先来看下系统的背景。

### 1.1 系统背景

生产环境的一个系统发生告警，拿到生产日志后出现如下字样：

```
    nio handle failed java.lang.OutOfMemoryError: Direct buffer memory
        at org.eclipse.jetty.io.nio.xxxx
        at org.eclipse.jetty.io.nio.xxxx
        at org.eclipse.jetty.io.nio.xxxx


```

通过日志，我们可以知道是`Direct buffer memory`这块区域发生了内存溢出异常，而且下面还有一大堆 Jetty 相关的调用栈。

_**Direct buffer memory 是什么？**_ 我们先来了解下这块区域。

### 1.2 堆外内存

Direct buffer memory——堆外内存，顾名思义是 Java 堆内存以外的一块内存区域， **这块区域不受 JVM 管理，而由操作系统管理** 。我们的程序里并没有直接使用堆外内存，而且通过日志中的调用栈看到，是由 Jetty 引起的。也就是说，Jetty 服务器可能在不停的使用堆外内存，然后堆外空间不足了，此时就抛出了内存溢出异常：

![](http://image.skjava.com/article/series/jvm/202308102133264371.png)

Jetty 是采 Java 编写的 Web 容器，它的一些底层机制要求它需要使用到堆外内存。在 Java 中，要使用堆外内存，必须要用到`DirectByteBuffer`这个类，构建 DirectByteBuffer 对象的同时（DirectByteBuffer 对象的引用本身在 Java 堆分配空间），就会在 Java 堆以外的内存空间划出一块区域，然后跟 DirectByteBuffer 对象关联起来：

![](http://image.skjava.com/article/series/jvm/202308102133272602.png)

当 DirectByteBuffer 对象失去所有引用，被垃圾回收器判定为垃圾对象时，就会在 Young GC 或 Full GC 时被回收掉，回收时也会将与它关联的那块堆外内存释放：

![](http://image.skjava.com/article/series/jvm/202308102133282483.png)

二、问题分析
------

了解了系统的大致情况以及堆外内存的基本原理，我们大致可以推测出正是因为 DirectByteBuffer 对象长期没有被回收，导致堆外内存被大量占用，从而引发内存溢出。

那么， **什么情况下会出现大量的 DirectByteBuffer 对象一直存活，导致大量的堆外内存也无法被释放呢？** 根据我们之前的学习经验，有三种可能：

1.  系统承载着超高并发，瞬间大量请求过来，创建了过多的 DirectByteBuffer 对象，来不及回收掉下一次请求又过来的，导致内存溢出。
2.  处理请求速度过慢或超时。
3.  JVM 中某些区域划分不合理，导致对象大量存活。

根据监控系统的分析，系统的并发度并不高，程序日志显示也没有很多超时，所以很可能是因为 JVM 内存区域划分不合理或处理请求速度过慢导致的。

### 2.1 jstat 分析

我们通过 jstat 分析发现，Jetty 会不断的创建 DirectByteBuffer 对象，直到新生代 Eden 区满了，就会触发 Young GC。但是，往往垃圾回收的一瞬间，很多请求还没处理完，所以只有部分 DirectByteBuffer 对象被回收，存活下来的 DirectByteBuffer 对象需要转移到 Survivor 区，但是 Survivor 区的大小只有 10MB！所以，只能将 DirectByteBuffer 对象转移到老年代：

![](http://image.skjava.com/article/series/jvm/202308102133290184.png)

按道理说，即使因为程序处理过慢，导致 Young GC 不能回收掉 DirectByteBuffer 对象，那么 DirectByteBuffer 对象进入到老年代后，等程序处理完了，下次 Full GC 时也会被回收掉。但问题就出在了 JVM 内存空间划分不合理，我们发现系统上线时的 JVM 配置是这样的：新生代一共 200MB 左右的空间，其中每个 Survivor 区就 10MB，老年代反而有 800MB 左右。

Survivor 区的空间不足，导致 DirectByteBuffer 对象进入老年代，随着老年代中的 DirectByteBuffer 对象越来越多，这些 DirectByteBuffer 对象关联的堆外内存占用也会越来越多，此时很多老年代中的 DirectByteBuffer 对象已经是垃圾对象了，但是由于一直没达到触发老年代回收的阈值，所以也就没法 Full GC，堆外内存也就是没法释放，最终导致堆外内存溢出。

### 2.2 SystemGC

Java NIO 其实已经考虑到了上述 DirectByteBuffer 垃圾对象一直无法被回收的问题，它在每次分配堆外内存时，都会调用下`System.gc()`方法，提醒 JVM 主动去回收那些没人引用的 DirectByteBuffer 对象，从而释放其关联的堆外内存。

但是，我们的系统上线时设置了参数`-XX:+DisableExplictGC`，也就是屏蔽了程序中的`System.gc()`方法，最终导致了堆外内存溢出的发生。

三、系统优化
------

分析清楚了原因，主要从两方面进行优化：

1.  根据系统运行模型，重新合理分配 JVM 内存，给新生代更多内存，特别是 Survivor 区，保证能够容纳每次 Young GC 后的存活对象；
2.  去掉参数`-XX:+DisableExplictGC`，让 System.gc() 生效。

> 生产环境原则上是要开启`-XX:+DisableExplictGC`的，但是如果能够保证自己程序里不出现`System.gc()`，则可以关闭。

四、总结
----

本章中，我们的案例之所以发生堆外内存溢出，其实是很多因素综合的结果。包括 JVM 内存划分不合理、处理请求速度较慢、屏蔽了 System.gc()。

所以，生产环境一旦发生 OOM 异常，除去一些程序 bug 等很明显的原因，往往是比较难排查的，可能是很多因素综合在一起导致了内存异常，我们要做的就是抓住主要矛盾，先按照最基本的优化思路去分析。另外，从这个案例和之前的 tomcat 案例也可以看出，我们平时还是要多去了解一些开源框架的底层原理，这样才能在出现问题时直击要点并解决问题。
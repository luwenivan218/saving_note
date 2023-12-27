> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1765523009)

> 一、简介 ParNew 是目前最常用的 JVM 垃圾回收器之一，主要应用在 & nbsp; 新生代 & nbsp; 的 GC 中。ParNew 采用了多线程垃圾回收机制，ParNew 垃圾回收器在执行 Mi

一、简介
----

ParNew 是目前最常用的 JVM 垃圾回收器之一，主要应用在 **新生代** 的 GC 中。ParNew 采用了多线程垃圾回收机制，ParNew 垃圾回收器在执行 Minor GC 时，会将 JVM 中的工作线程全部停止掉，禁止程序在运行时创建新的 Java 对象，然后自己用多个垃圾回收线程去进行垃圾回收，回收的机制和算法跟前面章节介绍的一样。

![](http://image.skjava.com/article/series/jvm/202308102127573631.png)

二、使用方式
------

通过 JVM 的参数设置，可以显式指定使用 ParNew 作为新生代的垃圾回收器。`-XX:+UseParNewGC`，只要加入这个选项，JVM 启动之后就会使用 ParNew 进行新生代的垃圾对象回收。

默认情况下，ParNew 设置的回收线程数与机器的 CPU 核心数相同。也就是说，假如我们线上的机器是 8 核，那么此时 ParNew 的垃圾回收线程数就是 8 个。比如下图，每个 GC 线程都通过一个 CPU 在运行：

![](http://image.skjava.com/article/series/jvm/202308102127586842.png)

> 线程数一般不需要手动设置，保持跟 CPU 核心数一致就可以充分进行并行处理。假如一定要手动设置，则使用`-XX:ParallelGCThreads`参数设置。

三、适用场景
------

我们上一章还提到过一种垃圾回收器——Serial，Serial 是一种单线程的垃圾收集器，生产环境的服务端应用基本不会使用该收集器，但是有一类 Java 应用还是可能会用到 Serial，即客户端 Java 应用，比如百度云盘的 Windows 客户端、印象笔记的 Windows 客户端等等。

由于客户端 Java 应用一般运行在 PC 上，且很多 PC 还是单核的，所以如果采用 ParNew 进行垃圾回收，就会导致一个 CPU 运行多个线程，此时会出现频繁的上下文切换，反而加重了性能开销，可能效率还不如单线程好。

所以，在使用 ParNew 作为生产环境的垃圾回收器时，记得使用`-server`指定为服务端模式。
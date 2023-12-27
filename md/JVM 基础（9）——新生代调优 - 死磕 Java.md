> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/9048504010)

> 一、简介本章和下一章，我们将通过一个实际案例讲解如何进行 JVM 参数调优：合理优化新生代、老年代、Eden 和 Survivor 各个区域的内存大小，接着再尽量优化参数避免新生代的对象

一、简介
----

本章和下一章，我们将通过一个实际案例讲解如何进行 JVM 参数调优：合理优化新生代、老年代、Eden 和 Survivor 各个区域的内存大小，接着再尽量优化参数避免新生代的对象进入老年代，尽量让对象留在新生代里被回收掉。

本章先针对新生代调优进行讲解，后续章节，我们会针对老年代调优再专门讲解，新生代调优的整个过程都围绕以下几点来思考：

*   每秒占用多少内存？
*   多长时间触发一次 Minor GC？
*   每次 Minor GC 后，存活对象的平均大小是多少？
*   Survivor 能否容纳存活对象？
*   会不会因为 Survivor 无法容纳频繁进入老年代？
*   会不会因为动态年龄判断规则进入老年代？

我们先来看下案例的背景。

### 1.1 案例背景

假设生产环境有一个每日上亿访问量的电商系统，日平均订单量 50 万。在双 11 等大促场景下，峰值为每秒 1000 笔订单。现在部署 3 台 4 核 8G 的机器，每台每秒抗 300 笔订单请求。

我们的目标就是对 JVM 有限的内存资源进行合理的分配和优化，让 JVM 的 Minor GC 次数尽量少，同时尽量避免 Full GC。

### 1.2 内存使用模型估算

交代完了背景，我们再来估算下高峰时期的 JVM 内存使用模型：

一笔订单一般 20 个左右的字段，按 1kb 算，那 1 秒钟 300 笔订单就是 300kb，然后还需要算上其它业务对象（比如库存、积分、商户等等），所以放大 20 倍。此外，订单系统还有其它操作，特别是查询操作，同样耗费内存，所以再放大 10 倍，那么每秒钟的内存总开销就是：  
$$  
300kb_20_10=60MB  
$$

> 我们假设 1s 过后，这 60MB 的对象都会失去引用变成垃圾。

![](http://image.skjava.com/article/series/jvm/202308102128112661.png)

二、JVM 内存分配
----------

### 2.1 初始情况

4 核 8G 的机器，一般分配 4G 给 JVM。假设最初状态下，Java 堆内存分配 3G（新生代 1.5G，老年代 1.5G），每个线程的栈空间 1MB，一个 JVM 大概几百个线程，所以总共给虚拟机栈几百兆，然后再给永久代 256MB：

`-Xms3072M -Xmx3072M -Xmn1536M -Xss1M -XX:PermSize=256M -XX:MaxPermSize=256M -XX:SurvivorRatio=8`

参数`-XX:SurvivorRatio`采用默认值 8，即 Eden 和 Survivor 按照默认的 8:1:1 分配内存。

> 空间担保机制的 JVM 参数`-XX:HandlePromotionFailure`在 JDK1.6 以后就被废弃了，JDK1.6 以后只需要判断 “老年代可用空间> 新生代对象总和” 或“老年代可用空间 > 历次 Minor GC 晋升到老年代的对象平均大小”，这两个条件满足任意一个，就可以直接进行 Minor GC，不会提前触发 Full GC。

![](http://image.skjava.com/article/series/jvm/202308102128119642.png)

### 2.2 优化 Survivor 空间

系统运行期间，每秒都会在新生代产生 60MB 对象，然后 1 秒后就失去引用，大约 25 左右，新生代中 Eden 区的 1.2G 空间就会被占满：

![](http://image.skjava.com/article/series/jvm/202308102128126183.png)

此时，就会触发是否进行 Minor GC 的判断，由于老年代可用空间（1.5G） > 历次 Minor GC 晋升到老年代的对象平均大小（0G），所以 Minor GC 直接运行，一下子回收 99% 的新生代中的垃圾，除了最近 1 秒的还在处理，剩下的都处理完了，总共余留大约 100MB 存活对象，存放对象放入 S1 区，然后清空 Eden：

![](http://image.skjava.com/article/series/jvm/202308102128132294.png)

然后，再运行 20 秒后，Eden 区再次被占满，再次触发并执行 Minor GC，存放对象放入 S2，同时清空 Eden 和 S1：

![](http://image.skjava.com/article/series/jvm/202308102128140325.png)

按照上面的测算，每次存活对象的平均大小为 100MB，Survivor 区的大小为 150MB，非常接近，很可能出现某次 Minor GC 后存活对象超过了 150MB，导致 Survivor 区无法容纳，使得存活对象晋升到老年代。

此外，每次存活对象的年龄是相同的，根据动态年龄规则，100MB 超过了 Survivor 的 50%，所以很可能这 100MB 存放对象会直接晋升到老年代。

所以，Survivor 区只分配 150MB 空间是不够的。所以可以考虑 **把新生代调整为 2G，老年代调整为 1G，这样每个 Survivor 就有 200MB 空间** ：

![](http://image.skjava.com/article/series/jvm/202308102128147596.png)

此时，JVM 参数如下：

`-Xms3072M -Xmx3072M -Xmn2048M -Xss1M -XX:PermSize=256M -XX:MaxPermSize=256M -XX:SurvivorRatio=8`

> 对于任何系统，应尽量让每次 Minor GC 后的对象都留在 Survivor 中，不要进入老年代，所以 Survivor 区是首要优化点。

### 2.3 是否优化晋升年龄

默认情况，新生代存活对象的年龄到达 15 后，就会进入老年代，那么我们是否要通过参数`-XX:MaxTenuringThreshold`来调整年龄的上限值，以避免对象进行老年代呢？

答案是否定的，一般来说，经历了 15 次 Minor GC 还存在于新生代的，都是那些需要长期存活的核心业务对象，比如 @Service、@Controller 等注解的对象。对于这些对象，它们就应该进入老年代，况且这类对象的数量不会很多，一般一个系统也就是几十 MB 而已。

所以，反而可以降级晋升年龄，我们这里设置成 5，即 - XX:MaxTenuringThreshold=5。

### 2.4 是否优化大对象

大对象是可以直接进入老年代的，一般来说`-XX:PretenureSizeThreshold`这个阈值设置成 1MB 足以，因为很少有超过 1MB 的大对象。如果有，可能是你提前分配了一个大数组或集合之类的对象，用以存放缓存。

### 2.5 指定垃圾回收器

最后，还要指定垃圾回收器，新生代用 ParNew，老年代用 CMS，最终，JVM 的参数设置如下：

`-Xms3072M -Xmx3072M -Xmn2048M -Xss1M -XX:PermSize=256M -XX:MaxPermSize=256M -XX:SurvivorRatio=8 -XX:MaxTenuringThreshold=5 -XX:PretenureSizeThreshold=1M -XX:+UseParNewGC -XX:+UseConcMarkSweepGC`

ParNew 垃圾收集器的核心参数，其实就是配套的新生代内存大小，Eden 和 Survivor 的比例，只要你设置合理，避免存活对象放不下 Survivor 而进入老年代，或者是动态年龄判断后进入老年代，那么 Minor GC 一般不会有什么问题。

三、总结
----

本章通过一个电商示例，分析了 JVM 新生代的参数优化，核心优化目的就是尽量让每次存活的对象都进入 Survivor，避免进入老年代。

本章主要针对新生代进行优化，下一章我们将结合案例分析老年代的垃圾回收和 JVM 参数优化。
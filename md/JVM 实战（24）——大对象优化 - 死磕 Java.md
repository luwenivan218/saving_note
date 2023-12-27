> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1715972882)

> 一、案例背景本章将介绍一个因为大对象而导致的频繁 GC 问题，其本质也是开发童鞋在写程序代码时存在 bug，进而导致出现 JVM 性能问题。首先，这个系统上线之后发现一天的 FullGC 次

一、案例背景
------

本章将介绍一个因为大对象而导致的频繁 GC 问题，其本质也是开发童鞋在写程序代码时存在 bug，进而导致出现 JVM 性能问题。

首先，这个系统上线之后发现一天的 Full GC 次数多达几十次，通常来说，我们建议的一个比较良好的 JVM 性能，应该是 Full GC 几天才发生一次，或者最多一天发生几次而已。

生产环境这个系统部署在 2 核 4G 的机器上，JVM 参数如下：  
`-Xms=1536M -Xmx=1536M -Xmn=512M -Xss=256K -XX:SurvivorRatio=5 -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=68 -XX:CMSParallelRemarkEnable -XX:UseCMSInitiatingOccupancyOnly -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:PrintHeapAtGC -Xloggc:gc.log`

比较关键的几个设置是：

*   -XX:SurvivorRatio=5：表示 Eden:Survivor:Survivor=5:1:1
*   -XX:CMSInitiatingOccupancyFraction=68：表示一旦老年代内存使用达到 68%，就会触发 Full GC

此时，整个系统对应的 JVM 内存模型如下：

![](http://image.skjava.com/article/series/jvm/202308102132504021.png)

### 1.1 存在问题

我们通过 jstat 进行分析，统计出的 JVM 性能如下：

*   系统运行时间：6 天
*   6 天内的 Full GC 次数和总耗时：250 次，70 秒
*   6 天内的 Young GC 次数和总耗时：26000 次，1400 秒

也就是说，平均每 20s 触发一次 Young GC，每 30 分钟触发一次 Full GC。根据 Eden 区和老年代的空间可以估算，系统每秒钟会产生约 20MB 对象进入 Eden，每 30 分钟会有约 600MB 对象进入老年代：

![](http://image.skjava.com/article/series/jvm/202308102132510752.png)

根据参数`-XX:CMSInitiatingOccupancyFraction=68`，老年代内存使用达到 68%，就会触发 Full GC，一次 Full GC 时间约 300ms。

二、大对象
-----

### 2.1 优化前

通过上述的案例背景介绍，我们首先想到的是会不会因为 Survivor 区太小，导致 Young GC 后的存活对象太多，放不下 Survivor 了，所以就一直有对象流入老年代，进而导致 30 分钟后触发 Full GC？

但这只是推论，因为对象进入老年代也可能是因为动态年龄判断规则，所以我们就需要通过工具在高峰期观察 JVM 的内存使用情况。

事实上，我们观察到每次 Young GC 后进入老年代的对象非常少，而且一次 Young GC 的存活对象也就是几十 MB，Survior 区可以容纳，偶尔触发动态年龄判断规则时，才有几十 MB 对象进入老年代：

![](http://image.skjava.com/article/series/jvm/202308102132518313.png)

因此，分析到这里就很奇怪了，因为通过 jstat 追踪，并不是每次 Young GC 后都有几十 MB 对象进入老年代，而是偶尔才有对象进入老年代，记住，是偶尔。

_那么老年代里面到底为什么会有那么多对象呢？_

我们观察发现， **系统运行一段时间后，突然间老年代中的对象就会增加五六百 MB** 。

答案已经很明显了—— **大对象** ！一定是系统运行时，每隔一段时间就会产生几百兆的大对象，直接进入老年代，不会走年轻代的 Eden 区，然后配合年轻代还偶尔会有几十 MB 对象进入老年代，所以才 30 分钟触发一次 Full GC：

![](http://image.skjava.com/article/series/jvm/202308102132524414.png)

### 2.2 优化后

了解了问题的所在，我们就开始针对这个案例进行优化。首先，就是要确定这个大对象到底是什么？

我们采用 jmap 工具导出一份 JVM 内存快照，然后通过 jhat 或者 Visual VM 之类的可视化工具进行分析，发现那几百兆大对象就是从数据库中查出的记录，然后对 SQL 进行排查，发现某个 SQL 在一种特殊场景下会执行类似`SELECT * FROM`的语句，导致一次性从数据库中查出几十万条数据。

针对该问题，主要做以下几点优化：

1.  解决程序 bug，不允许全表查询，这样就避免了大对象直接进入老年代的问题；
2.  Survivor 区明显不够，70MB 的空间很容易触发动态年龄判断，所以为其分配更多空间。

优化后的 JVM 参数如下：

`-Xms=1536M -Xmx=1536M -Xmn=1024M -Xss=256K -XX:SurvivorRatio=5 -XX:PermSize=256M -XX:MaxPermSize=256M -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=92 -XX:CMSParallelRemarkEnable -XX:UseCMSInitiatingOccupancyOnly -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:PrintHeapAtGC -Xloggc:gc.log`

可以看到，新生代直接分配 1G 空间，其中 Survivor 各占 150MB 左右，此时 Young GC 过后的几十 MB 存活对象一般就不会进入老年代了。

同时，调整参数`-XX:CMSInitiatingOccupancyFraction=92`，将比例提高至 92%，避免老年代仅占 62% 就触发 Full GC。

最后，还设置永久代大小为 256MB，因为默认永久代就几十 MB，如果程序使用了反射等机制，很容  
易被占满。

经过上述优化，系统基本上每分钟一次 Young GC，几天才会发生一个 Full GC。

三、总结
----

本章，我们通过示例分析了大对象导致的频繁 Full GC 问题，并一步一步展现了发现问题、分析问题、解决问题的思路。当我们发现 Young GC 过后并不是每次都有很多存活对象进入老年代的时候，就要从别的角度考虑下到底为什么会有那么多存活对象进入老年代。
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/3156591571)

> 一、简介前面章节，我们已经介绍了如何通过 GC 日志去分析系统的运行情况。本章，我们将带领大家运行一些 JVM 调优 / 检测工具来分析运行中的系统。我们常用的调优 / 检测工具有三种：jst

一、简介
----

前面章节，我们已经介绍了如何通过 GC 日志去分析系统的运行情况。本章，我们将带领大家运行一些 JVM 调优 / 检测工具来分析运行中的系统。我们常用的调优 / 检测工具有三种：_jstat_、_jmap_、_jhat_，我们来一一看下。

_jstat(JVM statistics Monitoring)_：用于 **监视 JVM 运行时状态信息** 的命令，它可以显示出虚拟机进程中的类装载、内存、垃圾收集、JIT 编译等运行数据。

_jmap(JVM Memory Map)_：用于生成 **heap dump** 文件，jmap 可以查询当前 Java 堆内存的详细信息，比如当前各个区域使用率（总容量、已使用、未使用）、当前使用的是哪种收集器等。

_jhat(JVM Heap Analysis Tool)_：，一般与 jmap 搭配使用，用来 **分析 jmap 生成的 dump 文件** ，jhat 内置了一个微型的 HTTP/HTML 服务器，生成 dump 的分析结果后，可以在浏览器中查看。

> 当然，除了上述介绍的最基本的工具外，还有很多图形化的工具，比如 VisualVM、MAT 等等。我们的目的是介绍使用这些工具进行调优的思路，在理解了思想之后，运用任何工具，都可以轻松把 JVM 的运行情况分析清楚，一通百通。

二、jstat
-------

jstat 可以检查 JVM 的整体运行情况，包括 JVM 内的 Eden、Survivor、老年代的内存使用情况，以及 Young GC 和 Full GC 的频率及耗时。通过这些指标，我们可以分析当前系统的运行状况，判断当前系统的内存使用压力、GC 频次是否太高、内存分配是否合理。

### 2.1 基本用法

jstat 的基本用法如下：

_**jstat [option] LVMID [interval] [count]**_

*   [option]：操作参数
*   LVMID：JVM 进程 ID
*   [interval]：连续输出的时间间隔
*   [count]：连续输出的次数

接下来，我们就介绍 jstat 的一些常用命令。

### 2.2 jstat -gc PID

_jstat -gc PID_，该命令可以查看 JVM 的内存和 GC 情况，PID 就是 JVM 的进程 ID。运行命令后可以看到如下信息：

S0C：From Survivor 区的总大小  
S1C：To Survivor 区的总大小  
S0U：From Survivor 区目前已使用空间  
S1U：To Survivor 区目前已使用空间  
EC：Eden 区的总大小  
EU：Eden 区目前已使用空间  
OC：老年代的总大小  
OU：老年代目前已使用空间  
MC：方法区（永久代、元数据区）的总大小  
MU：方法区（永久代、元数据区）目前已使用空间  
YGC：系统运行迄今为止的 Young GC 次数  
YGCT：系统运行迄今为止的 Young GC 总耗时  
FGC：系统运行迄今为止的 Full GC 次数  
FGCT：系统运行迄今为止的 Full GC 总耗时  
GCT：系统运行迄今为止的所有 GC 总耗时

> jstat -gc PID 是最常用的命令，基本足够我们分析 JVM 的运行情况，jstat 还有许多其它命令，读者可以参考 Oracle 官方文档： [https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstat.html](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstat.html) 。

### 2.3 分析思路

当我们使用 jstat 来分析 JVM 的运行情况时，我们最关注以下信息：

*   新生代对象的增长速率
*   Young GC 的触发频率
*   Young GC 的耗时
*   每次 Young GC 后的新生代存活对象大小
*   每次 Young GC 后的晋升老年代对象大小
*   老年代对象的增长速率
*   Full GC 的触发频率
*   Full GC 的耗时

只要知道了这些信息，就可以结合前几章的分析方法对 JVM 优化：合理分配内存空间，减少新生代对象频繁进入老年代，避免频繁 Full GC。

#### 新生代对象的增长速率

根据前面几章的案例分析，我们首先需要对系统的内存使用模型进行估算，也就是分析 **每秒钟会在 Eden 分配多少对象** 。

可以通过 **jstat -gc PID 1000 10** 进行分析，即每隔 1s 更新一行 jstat 统计信息，一共执行 10 次。

举个例子：假如执行这个命令后，第 1s 先显示出来 Eden 区使用了 200MB 内存，第 2s 显示出来的那行统计信息里，发现 Eden 区使用了 205MB，第 3s 显示出来的那行，发现 Eden 区使用了 209MB 内存。以此类推，可以推断出，系统大概每秒新增 5MB 左右的对象。

另外，一般系统有高峰和日常两种状态，高峰时期执行上述命令可以看到高峰期的对象增长速率。非高峰期可能系统负载比较低，不一定每秒都有请求，所以可以把上面的 1 秒钟调整成 1 分钟，甚至 10 分钟。

按照上述思路，基本可以对线上系统的高峰和日常两个时段内的对象增长速率有很清晰的了解。

#### Young GC 的触发频率

通过新生代对象的增长速率，可以很容易推测出 Young GC 的触发频率。比如 Eden 区总共有 800MB 内存，高峰期每秒新增 5MB 对象，那么高峰期大概 3 分钟就会触发一次 Young GC。日常期以次类推。

#### Young GC 耗时

jstat -gc 会告诉我们从 JVM 启动至今一共发生了多少次 Young GC 以及总耗时。比如系统运行了 24 小时后共发生了 260 次 Young GC，总耗时 20s。那么平均下来，每次 Young GC 大概就耗时几十毫秒的时间。

#### Young GC 存活 / 晋升对象大小

每次 Young GC 过后，有多少对象会存活下来，这个没法直接看出来，但是可以根据 Young GC 的触发频率推断出来。

比如，我们可以每隔 3 分钟统计一次（jstsat -g PID 180000 ），此时可以观察，Eden、Survivor、老年代的已使用空间的变化情况。正常来说，Eden 区在经历 Young GC 后会从接近占满到变得很少，Survivor 区会放入一些存活对象，老年代可能会增长一些对象占用。

所以，每次 Young GC 过后的存活对象大小，就是 Survivor 区的对象大小和本次老年代增长的大小；晋升对象的大小就是本次老年代增长的大小。

#### Full GC 的触发频率 / 耗时

只要知道了老年代的增长速率，那么 Full GC 的触发时机就可以推断出来。比如，老年代总共 800MB 内存，每隔 3 分钟新增 50MB，那么大概 1 小时触发一次 Full GC，这就是 Full 的触发频率。

至于 Full GC 的平均耗时，可以通过 jstat 命令打印出来的 JVM 启动以来的 Full GC 次数和总耗时计算出来。比如迄今一共执行了 10 次 Full GC，总耗时 30s，那么 Full GC 平均耗时就是 3s 左右。

三、jmap
------

如果只是需要了解 JVM 的运行情况，然后进行 JVM GC 优化，那 jstat 完全够用了。但是有时候，我们会发现 JVM 新增对象的速度很快，然后就想看看， **到底什么对象占据了那么多的内存** 。比如，我们之前的[模拟对象晋升](https://www.tpvlog.com/article/101)一章中，总有几百 KB 的未知对象占据着空间，jmap 就可以帮助我们解决这个问题。

### 3.1 基本用法

jmap（JVM Memory Map），用于生成 **heap dump** 文件，可以查询当前 Java 堆的详细信息，比如当前各个区域使用率（总容量、已使用、未使用）、当前使用的是哪种收集器等。其基本用法如下：

_**jmap [option] LVMID**_

*[options]* 命令参数：

*   dump：生成堆转储快照
*   finalizerinfo：显示在 F-Queue 队列等待 Finalizer 线程执行 finalizer 方法的对象
*   heap：显示 Java 堆详细信息
*   histo：显示堆中对象的统计信息
*   permstat：打印永久代（元数据区、方法区）中的
*   F：当 dump 没有响应时，强制生成 dump 快照

### 3.2 jmap -heap PID

该命令用于显示 Java 堆内存的详细信息，比如 Eden 区总容量、已使用的容量、剩余容量，两个 Survivor 区的总容量、已使用容量、剩余容量，老年代的总容量、已使用容量、剩余容量。

但是，这些信息一般 jstat 命令就可以显示，所以一般不会用 jmap 去看这些信息。

### 3.3 jmap -histo PID

`jmap -histo`会打印出类似以下的信息，即当前 JVM 中的对象占用情况（按空间占用从大到小排序）：

![](http://image.skjava.com/article/series/jvm/202308102132019571.png)

所以，通过该命令可以了解到当前内存里到底是哪个对象占用了大量空间

### 3.4 jmap -dump PID

`jmap -dump`可以生成一个 Java 堆转储快照。比如`jmap -dump:live,format=b,file=dump.hprof PID`，这个命令会在当前目录下生成一个 dump.hprof 二进制文件，它会把这一时刻 Java 堆内存中的所有对象的快照放到文件中去，供后续分析。

四、jhat
------

jhat（JVM Heap Analysis Tool），一般与 jmap 搭配使用，用来 **分析 jmap 生成的 Java 堆转储快照文件** 。

jhat 内置了一个微型的 HTTP/HTML 服务器，生成 dump 的分析结果后，可以在浏览器中查看。

> 一般不会直接在服务器上进行分析，因为 jhat 是一个耗时并且耗费硬件资源的过程，一般把服务器生成的 dump 文件复制到本地或其他机器上进行分析。另外，分析同样一个 dump 快照， **MAT** 需要的额外内存比 jhat 要小的多的多，所以建议使用 MAT 来进行分析，当然也看个人偏好。

### 4.1 基本用法

jstat 的基本用法如下：

_**jhat [dumpfile]**_

比如，可以使用命令`jhat dump.hprof -port 7000`启动 jhat 服务器，当通过浏览器访问 7000 端口时，就可以通过图形化的方式去分析堆内存里的对象分布情况了。

五、总结
----

本章，我们介绍了 _jstat_、_jmap_、_jhat_ 这三种命令行工具的基本用法。系统开发完毕后，一般要经过 **预估性优化** 、 **压测优化** 、 **线上监控** 这三个过程。

_预估性优化_ ：本质就是 **估算系统内存使用模型，然后合理分配 Java 堆内存，尽量让每次 Young GC 后的存活对象小于 Survivor 区，避免存活对象频繁进入老年代引发 Full GC** 。

_压测优化_ ：是对预估性优化的检验，通常这个环境会使用一些压测工具模拟高并发的访问，看看系统能否撑住请求压力、响应延时是否在正常范围内，保持稳定运行。压测环节需要借助 jstat 等工具分析 JVM 运行情况，然后合理调整堆内存分布。

_线上监控_ ：是系统上线之后对 JVM 的监控，最简单的方式是在每天的高峰期和日常期，用 _jstat_、_jmap_、_jhat_ 等命令查看 JVM 情况。更常见的做法是引入专门的监控系统，比如 Zabbix、OpenFalcon、Ganglia 等。业务系统会将 JVM 统计项发给这些监控系统，然后监控系统会进行分析并以图形化方式动态展现，还可以制定监控规则，让其对频繁 GC 的情况进行告警。

下一章，我们将通过实际案例讲解如何通过 _jstat_、_jmap_、_jhat_ 这三种命令行工具进行优化。
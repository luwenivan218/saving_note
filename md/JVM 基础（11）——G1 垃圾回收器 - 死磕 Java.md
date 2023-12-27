> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/8466706298)

> 一、简介经过前面章节的讲解，大家应该对 ParNew+CMS 这个 GC 组合的执行原理非常清楚了。但是，“StoptheWorld” 这个最根本的问题并没有解决。无论是新生代的回收，还

一、简介
----

经过前面章节的讲解，大家应该对 ParNew+CMS 这个 GC 组合的执行原理非常清楚了。但是，“Stop the World”这个最根本的问题并没有解决。无论是新生代的回收，还是老年代的回收，都会或多或少发生 “Stop the World” 现象，对系统的运行产生影响。

所以，后续各种垃圾回收器的优化，都是奔着减少 “Stop the World” 这个目标去的。在此基础上，G1 垃圾回收器就诞生了，它可以提供比 “ParNew+CMS” 组合更好的垃圾回收性能。

### 1.1 什么是 G1

G1 垃圾回收器是 Jdk1.7 的新特性之一，在 Jdk1.7 + 版本都可以自主配置 G1 作为 JVM GC 选项。G1 垃圾回收器可以同时回收新生代和老年代的对象，它一个人就可以搞定所有的垃圾回收。

**G1 将整个 Java 堆划分为多个大小相等的独立区域（Region）** ：

![](http://image.skjava.com/article/series/jvm/202308102128247861.png)

并且，虽然 G1 还保留着新生代和老年代的概念，但它们只是逻辑上的，新生代和老年代不再是物理上隔阂的，而只是一部分 Region 的集合，每一个 Region 既可能属于新生代，也可能属于老年代：

![](http://image.skjava.com/article/series/jvm/202308102128253772.png)

刚开始时 Region 谁都不属于，然后会先分配给新生代，当对象越来越多后，可能触发 G1 对这个 Region 进行垃圾回收：

![](http://image.skjava.com/article/series/jvm/202308102128259953.png)

然后下一次，这个 Region 可能又被分配给了老年代，用来存放长期存活对象：

![](http://image.skjava.com/article/series/jvm/202308102128268784.png)

### 1.2 预期停顿时间

G1 最大的特点就是，可以让我们 **设置一个垃圾回收的预期停顿时间** 。比如我们可以指定：G1 进行垃圾回收时，保证 “Stop the World” 的时间不超过 1 分钟。

之前，我们采用 ParNew+CMS 时，为了尽量减少 GC 次数，需要对 JVM 内存空间合理划分，还要配置各种 JVM 参数。但是现在，我们可以直接给 G1 指定一个预期停顿时间，告诉它一段时间内因垃圾回收导致的系统停顿时间不能超过多久，剩下的全部交给 G1 全权负责，这样就相当于 **我们可以直接控制 GC 对系统性能的影响** 。

> 通过`-XX:MaxGCPauseMills`参数可以设定预期停顿时间，表示 G1 执行 GC 时最多让系统停顿多长时间，默认 200ms。

### 1.3 回收价值

G1 之所以能够做到控制停顿时间，是因为它会追踪每个 Region 里的 **回收价值** 。所谓回收价值，是指每个 Region 里有多少垃圾对象，如果进行回收，耗时多长，能够回收掉多少。

大家看下下图，G1 通过追踪发现，1 个 Region 中的垃圾有 10MB，回收需要 1s，另一个 Region 中的垃圾有 20MB，回收需要 200ms：

![](http://image.skjava.com/article/series/jvm/202308102128275475.png)

然后在垃圾回收的时候，G1 就会判断哪个 Region 更有回收价值，显然 20MB/200ms 那个更有回收价值，因为所需的时间更短，能回收的垃圾也更多。

**根据回收价值进行 GC，这个就是 G1 的核心设计思路** 。

二、Region
--------

### 2.1 Region 大小设置

首先，G1 的堆内存中，各个 Region 的大小是相同的，那么要分配多少个 Region 呢？每个 Region 的大小为多少？

其实是自动设置的，我们通过`-Xms`和`-Xmx`来设置堆内存的大小，然后 JVM 启动时发现如果采用了 G1 作为垃圾回收器（通过参数`-XX:UseG1GC`指定），会用堆内存大小除以 2048，得到每个 Region 的大小。比如堆大小为 4096MB，默认 2048 个 Region，那每个 Region 就是 2MB：

![](http://image.skjava.com/article/series/jvm/202308102128285406.png)

> Region 的大小必须为 2 的整数倍，如 2MB、4MB、6MB 等，可以通过`-XX:G1HeapRegionSize`参数手动指定。

### 2.2 动态 Region

初始情况下，堆内存的 5% 空间为新生代的大小，以 4G 堆内存来算，就是 200MB 的新生代，约 100 个 Region。但是在系统运行期间，Region 的数量是动态变化的，不过新生代最多占比也不会超过 60%。

另外，一旦 Region 进行了垃圾回收，此时新生代的 Region 数量还会减少，这些其实都是动态的。

> 可以通过参数`-XX:G1NewSizePercent`来设置新生代的初始占比，默认 5%；通过参数`-XX:G1MaxNewSizePercent`来设置新生代的最大占比，默认 60%。

### 2.3 Eden 和 Survivor

G1 垃圾回收器的新生代也有 Eden 和 Survivor 的划分，同样通过`-XX:SurvivorRatio=8`设置比例。比如说，新生代最初有 100 个 Region，那 Eden 就占 80 个，两个 Survivor 各占 10 个。

随着对象不停的在新生代分配，属于新生代的 Region 会不断增加，Eden 和 Survivor 对应的 Region 也会不断增加。

三、垃圾回收原理
--------

接着，我们来看下 G1 进行垃圾回收的整个流程。

### 3.1 新生代回收

G1 对新生代的垃圾回收思路和 ParNew 其实是类似的，随着不停的在新生代的 Eden 区中分配对象，JVM 会不停的给新生代增加更多的 Region，直到新生代的占比达到堆内存的 60%。

假设此时新生代的 Region 数目为 1200 个，其中 Eden 占了 1000 个 Region（两个 Survivor 各 100 个），并且已经分配满了对象：

![](http://image.skjava.com/article/series/jvm/202308102128292207.png)

这个时候，就会触发新生代的 GC，进入 “Stop the World” 状态，G1 垃圾回收器会使用 **复制算法** ，将存活的对象放入其中一块空的 Survivor 区中（上图中的 S1），然后回收掉 Eden 中的垃圾对象：

![](http://image.skjava.com/article/series/jvm/202308102128298038.png)

> 这个过程和 ParNew 还是有区别的，因为 G1 可以设置预期停顿时间（`-XX:MaxGCPauseMills`参数，默认 200ms），它会对每个 Region 进行追踪，估算回收该 Region 的价值，然后选择一部分 Region，保证系统停顿时间在指定的控制范围内。

### 3.2 晋升到老年代

默认新生代最多只能占用 60%Region，老年代最多可以占用 40%Region，以 4G 堆内存算就是 800 个 Region。那么新生代中的对象何时会晋升到老年代呢？这个机制和之前章节讲解的 ParNew+CMS 几乎一样：

1.  对象躲过了多次 GC，达到一定的年龄（`-XX:MaxTenuringThrehold`参数设置）；
2.  符合动态年龄判断规则，即某次新生代 GC 后，各年龄存活对象的累加大小超过了 Survivor 的 50%；

除了上述两种情况外，还有一种大对象直接晋升的规则，不过和之前有所区别：

**G1 提供了专门的 Region 存放大对象，而不是让大对象进入老年代的 Region。在 G1 中，大对象的判断规则就是超过了一个 Region 的 50%，而且如果对象太大，会跨多个 Region 存储。**

![](http://image.skjava.com/article/series/jvm/202308102128307459.png)

> 新生代和老年代在进行垃圾回收的时候，会顺带将大对象 Region 一起回收。

### 3.3 混合回收

随着系统的运行，会有越来越多的存活对象进入老年代，因次会动态给老年代分配更多的 Region。当老年代的 Region 数量达到堆内存的 45% 时（通过参数`-XX:InitiatingHeapOccupancyPercent`可以配置），会触发一次 **混合回收（Mixed GC）** ，即对新生代和老年代一起回收。

比如，假设堆内存总共 2048 个 Region，如果老年代占到 45%，即约 1000 个 Region 时，就会触发混合回收，如下图：

![](http://image.skjava.com/article/series/jvm/2023081021283185010.png)

#### 初始标记

混合回收的第一步，就是进行 **初始标记** 。初始标记需要 “Stop the World”，然后标记 GC Roots 能够 **直接引用** 到的对象，这个过程是非常快的。

如下图，初始标记时先挂起工作线程，然后对各个线程内的局部变量代表的 GC Roots，以及方法区中的类静态变量代表的 GC Roots 进行扫描，标记出来它们直接引用的那些对象：

![](http://image.skjava.com/article/series/jvm/2023081021283263311.png)

> 初始标记阶段，需要 “Stop the World”，但是并不耗时。

#### 并发标记

接着，进入 **并发标记** 阶段。该阶段工作线程和 GC 线程并行运行，GC 线程会从 GC Roots 开始追踪所有存活的对象。这里举个例子来更好的理解下 **GC Roots 追踪** 。

先来看下示例代码：

```
    public class Kafka{
        public static ReplicaManager replicaManager = new ReplicaManager();
    }
    public class ReplicaManager{
        public static ReplicaFetcher replicaFetcher = new ReplicaFetcher();
    }


```

上述代码中，类静态变量 replicaManager 是一个 GC Root。初始标记阶段，仅仅标记它指向的 ReplicaManager 对象。在并发标记阶段，进行 GC Roots 追踪时，会从 GC Root 的直接关联对象（ReplicaManager）开始往下追踪，然后追踪到 replicaFetcher 实例变量，该变量关联到了对象 ReplicaFetcher，所以最终就把 ReplicaFetcher 对象并发标记为存活对象。

![](http://image.skjava.com/article/series/jvm/2023081021283324512.png)

> 并发标记阶段，不需要 “Stop the World”，需要追踪全部存活对象，所以非常耗时。但是，由于 GC 线程与工作线程并行运行，所以对系统程序的影响不大。

#### 最终标记

在并发标记期间，JVM 会记录系统程序对一些对象的修改，比如新创建一些对象，又有一些对象失去引用等等。在最终标记阶段，会进入 “Stop the World” 状态，然后根据并发标记阶段记录的修改，再次标记下哪些是存活对象，哪些是垃圾对象：

![](http://image.skjava.com/article/series/jvm/2023081021283435713.png)

> 并发标记阶段，需要 “Stop the World”，但是并不耗时。

#### 执行回收

执行回收阶段，会计算老年代中的每个 Region 中的存活对象数量、存活对象占比，以及执行 GC 的预期性能和效率，然后会 “Stop the World”，开始进行垃圾回收，回收时采用 **复制算法** ，把要回收的 Region 里的存活对象放入其他 Region，然后清掉这个 Region。

执行回收阶段，只会选择部分 Region 进行回收，因为系统停顿时间必须控制在我们指定的范围内。

比如说，此时老年代的 1000 个 Region 已经占满了对象，根据设置的 **预期停顿时间（200ms）** ，那么通过之前的计算得知，可能回收其中 800 个 Region 差不多刚好 200ms，那么就只会回收 800 个 Region，把系统停顿之间控制在指定范围内：

![](http://image.skjava.com/article/series/jvm/2023081021283525714.png)

**注意：**  
执行回收时，并不是一次性就全部回收掉了，“执行回收 “这个动作会反复执行好多次，也就是说先停止系统一会儿，然后回收掉一些 Region，再让系统运行一会儿，然后再停止再回收 Region，如此反复。参数`-XX:G1MixedGCCountTarget`可以控制次数，默认为 8 次。

之所以这样做，本质还是要控制每次 GC 的时间再预期停顿时间内。

> 执行回收时，会 “Stop the World”，而且不仅仅只回收老年代，新生代和大对象的 Region 也都会进行回收。
> 
> 另外，只有 Region 中的存活对象大小 < Region 空间的 85% 时，才会对这个 Region 进行回收，可以通过参数`-XX:G1MixedGCLiveThredholdPercent`来设置这个比例。

#### 停止回收

由于在执行回收阶段，基于 **复制算法** ，那就会不断的空出一些 Region，一旦空闲的 Region 数据量达到了堆内存的 5%，就会立即停止回收，那么本轮混合回收（Mixed GC）就结束了。

> 可以通过参数`-XX:G1HeapWastePercent`配置这个空闲 Region 的占比，默认为 5%。

#### 回收失败

由于在执行回收时，需要将存活对象拷贝到其他 Region 中，如果万一在次过程中没有空闲的 Region 可以承载存活对象，就会触发 Full GC。

此时，JVM 会立即停止程序，然后采用 Serial Old 收集器进行单线程标记、清除、压缩整理，空出一批 Region，这个过程是非常缓慢的。

四、总结
----

本章，我们对 G1 垃圾回收的基本原理和垃圾回收的整个流程做了讲解。其实可以发现，G1 的最大特点其实就是 **把每次执行回收的时间控制在我们设置的预期停顿时间范围内。**

G1 非常适合大内存的机器，比如 16G、32G 这种，另外对于响应时效性要求高的系统，G1 也非常合适，因为 G1 可以控制每次 GC 的时间。

此外，G1 的整个 GC 过程其实和 CMS 非常类似，下一章我们将通过一个实际案例来讲解如何对 G1 进行优化，及其背后的原理。
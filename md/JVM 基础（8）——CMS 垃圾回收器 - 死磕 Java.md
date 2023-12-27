> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/9297883209)

> 一、简介理想情况下，我们都希望自己的系统能在每次 MinorGC 后，存活对象都能转移到 Survivor 区，避免进入老年代。但真实情况是，线上系统很可能因为各种各样的情况，导致很多

一、简介
----

理想情况下，我们都希望自己的系统能在每次 Minor GC 后，存活对象都能转移到 Survivor 区，避免进入老年代。但真实情况是，线上系统很可能因为各种各样的情况，导致很多对象进入老年代，甚至频繁触发老年代的 Full GC。所以，我们必须对老年代的垃圾回收器的执行原理有一个清晰的认识，才能在出现问题时及时应对。

二、CMS 基本原理
----------

老年代的垃圾回收，我们一般会选择 CMS 垃圾回收器，它采用的是 **标记清除（mark-sweep）** 算法。关于标记清除算法，我们已经在 [JVM 垃圾回收算法](https://www.tpvlog.com/article/89)一文中详细介绍过了标记清除算法，标记清除与标记整理的区别就是它标记完垃圾对象后直接清理掉，这个会造成内存碎片过多的问题，后面会讲到。

针对老年代进行垃圾回收时，CMS 也会出现”Stop the World“现象，之前的文章也说过，如果挂起一切工作线程，然后慢慢地去执行”mark-sweep“算法，会导致系统” 卡顿 “时间过长，很多响应无法处理。CMS 采取的策略是： **垃圾回收线程和系统工作线程尽量并行执行** 。

CMS 在执行一次垃圾回收的过程一共为 4 个阶段：

*   初始标记
*   并发标记
*   重新标记
*   并发清理

下面我们来对每一阶段进行分析。

### 2.1 初始标记

CMS 进行垃圾回收时，首先会执行初始标记，这个阶段会让系统的工作线程全部挂起，进入 “Stop the World” 状态，初始标记，就是标记 “GC Roots” 能够引用到的对象。如下图：

![](http://image.skjava.com/article/series/jvm/202308102128027401.png)

我们还是以示例代码来看下整个过程：

```
    public class Kafka {
        public static ReplicaManager replicaManager = new ReplicaManager();
    }
    public class ReplicaManager {
        public ReplicaFetcher replicaFetcher = new ReplicaFetcher();
    }


```

假设上面代码对应的 JVM 内存结构如下图：

![](http://image.skjava.com/article/series/jvm/202308102128032962.png)

那么初始标记时，静态变量 replicaManager 所引用的 ReplicaManager 对象就会被标记出来，但 ReplicaFetcher 对象不会被标记，因为类的静态字段和方法的局部变量可以作为 “GC Roots”，而类的普通实例字段不能。

> 初始标记阶段，虽然会出现 “Stop the World”，但其实影响不大，因为从“GC Roots” 去标记存活对象的效率很高。另外， 可以通过设置参数`-XX:+CMSParallelInitialMarkEnabled`开始多线程的初始标记，减少 STW 时间，以提升性能。

### 2.2 并发标记

接着，进入并发标记阶段，该阶段系统的工作线程可以随时创建各种对象。与此同时，垃圾回收线程会尽可能的对已有的对象进行 GC Roots 追踪。

所谓 GC Roots 追踪，就是对老年代里的 ReplicaFetcher 这类对象，看看被谁引用了？比如 ReplicaFetcher 对象被 ReplicaManager 对象的 replicaFetcher 字段引用，而 ReplicaManager 对象又被 Kafka 类的静态字段 replicaManager 引用。

那么此时，CMS 就会认为 ReplicaFetcher 这个对象其实是被 GC Roots 间接引用的，所以不需要回收它，可以把它标记存活，如下图：

![](http://image.skjava.com/article/series/jvm/202308102128039733.png)

但是在并发标记的过程中，由于系统在不停的工作，可能会创建出来新的对象，也可能一些旧的对象失去引用，如下图：

![](http://image.skjava.com/article/series/jvm/202308102128048564.png)

并发标记的时候，需要对 GC Roots 进行深度追踪，看所有对象里到底有多少是存活的，而老年代中的对象存活率又比较高，所以这个过程会追踪大量的对象，比较耗时。

> 并发标记阶段，其本质就是追踪老年代中的所有对象能否直接或间接被 GC Roots 引用，这个过程是跟垃圾回收线程并行进行的，所以虽然很耗时，但不会对系统运行造成较大影响。

### 2.3 重新标记

在第二阶段，一边是 GC 线程标记着存活对象和垃圾对象，另一边是工作线程不停的创建新对象和让老对象失去引用，所以当第二阶段结束后，会有很多存活对象和垃圾对象是没有被标记出来的：

![](http://image.skjava.com/article/series/jvm/202308102128054745.png)

所以，重新标记阶段的工作，就是让系统停下来，进入 “Stop the World”，然后再重新标记一下第二阶段里新创建和新失去引用的那些对象：

![](http://image.skjava.com/article/series/jvm/202308102128064176.png)

> 重新标记阶段，虽然会发生 “Stop the World”，但速度是很快的，因为只是对第二阶段中因为并行而变动过的少数对象进行标记。另外，可以通过设置参数`-XX:+CMSScavengeBeforeMark`，让重新标记之前尽量先先执行一次 Young GC，那么重新标记的时候就可以少扫描一些对象，以提升该阶段的性能。

### 2.4 并发清理

重新标记完成后，就会恢复工作线程，进入最后的并发清理阶段。该阶段垃圾回收线程和工作线程是并行运行的，垃圾回收线程会清理之前标记为垃圾的对象：

![](http://image.skjava.com/article/series/jvm/202308102128073377.png)

并发清理需要将垃圾对象从各种随机的内存位置清理掉，所以也比较耗时。

> 并发清理阶段和并发标记阶段一样，是比较耗时的，但是不会挂起工作线程，所以基本不影响系统的运行。

三、性能问题
------

CMS 为了避免长时间的”Stop the World“，采用了 4 个阶段进行垃圾回收，其中初始标记和重新标记阶段虽然会”Stop the World“，但是耗时很短，所以影响不大；并发标记和并发清理阶段虽然耗时较长，但是可以跟工作线程并行执行，所以影响也不大。

那么，CMS 就很完美了吗？显示不是，我们来看下 CMS 的这种垃圾回收方式可能会出现什么样的问题。

### 3.1 CPU 消耗

CMS 垃圾回收器有一个最大的问题，就是并发标记和并发清理阶段，工作线程和垃圾回收线程同时运行，而这两个阶段又比较耗时，所以会导致有限的 CPU 资源长时间被垃圾回收线程占用。

CMS 启动时默认的垃圾回收线程数量是：`(CPU核数+3)/4`。假设我们用最普通的机器（2 核 4G）来测算，（2+3）/4=1，也就是说 GC 线程会占去一个 CPU。

### 3.2 Concurrent Mode Failure

CMS 垃圾回收器的另一个问题就是 _Concurrent Mode Failure_。所谓 _Concurrent Mode Failure_，就是指在并发清理阶段，有一些原来是新生代的对象将晋升到老年代，而此时老年代的可用空间又不够了，就会发生 _Concurrent Mode Failure_。我们来看下整个过程：

首先，在并发清理阶段，由于工作线程也在并行运行，一些新生代对象晋升到了老年代，随后失去了引用，那这些对象就变成了老年代的” **浮动垃圾** “，因为它们已经错过了并发标记，只能等到下一次 GC 时被回收：

![](http://image.skjava.com/article/series/jvm/202308102128081068.png)

上图中的红圈部分就是一个浮动垃圾。为了应对出现” 浮动垃圾 “这种情况，CMS 会在老年代预留一定的空间，如果 CMS 在垃圾清理期间，出现了浮动垃圾，且垃圾大小大于老年代中的预留空间，就会出现 _Concurrent Mode Failure_。

当出现 _Concurrent Mode Failure_ 时，CMS 会自动让 Serial Old 垃圾收集器来替代自己，强行进行 “Stop the World”，并重新进行 GC Roots 追踪，标记垃圾对象并清除，完事后才恢复工作线程运行。

综上，生产环境下，这个自动触发 CMS 垃圾回收的比例需要合理优化下，避免出现 _Concurrent Mode Failure_ 问题。

> CMS 触发垃圾回收的时机，其中一个就是当老年代内存占用达到一定的比例，通过 `-XX:CMSInitiatingOccupancyFaction`参数可以设置这个比例，JDK1.6 中默认是 92%。

### 3.3 内存碎片

由于 CMS 采用标记整理算法对老年代的垃圾对象进行回收，所以会产生大量的内存碎片。如果内存碎片太多，会导致后续对象进入老年代找不到可用的连续空间，触发 Full GC。

CMS 有一个参数`-XX:+UseCMSCompactAtFullCollection`（默认打开），表示是否要在 Full GC 之后进行 Stop the World，停止工作线程，然后进行老年代的内存碎片整理。

还有另外一个参数`-XX:CMSFullGCsBeforeCompaction`，意思是执行多少次 Full GC 之后再执行一次内存碎片整理工作，默认是 0，即每次 Full GC 之后都会进行一次内存碎片整理。

四、总结
----

通过上述对 CMS 垃圾回收器的执行流程分析，我们其实已经知道 CMS 专门针对 “Stop the World” 进行了优化：

*   最耗时的两个阶段——并发标记和并发清理，其实都不需要挂起工作线程，所以对系统整体性能影响不大；
*   两个需要”Stop the World“的阶段——初始标记和重新标记，仅仅是简单的标记而已，速度非常快，所以基本上对系统的整体性能影响也不大。

但是，用 CMS 进行老年代的垃圾回收还是要比新生代的 Minor GC 慢十倍以上，原因很简单：

*   新生代执行速度快，是因为新生代的存活对象很少，从 GC Roots 出发马上能追踪到哪些对象是活的，压根不需要追踪多少对象；
*   老年代中对象的存活率很高，所以并发标记阶段很慢，此外并发清理阶段是去找各种零零散散的垃圾对象，速度也很慢，最后 Full GC 完还得内存整理，所以非常耗时。

我们这里对 **Full GC 的所有情况** 做一个总结：

1.  老年代的可用连续内存空间 < 新生代全部对象的大小，且未开启空间担保；
2.  老年代的可用连续内存空间 < 新生代全部对象的大小，且开启了空间担保，但是老年代的可用连续内存空间 < 历代晋升到老年代的平均对象大小；
3.  新生代 Minor GC 后，存活对象大小 > Survivor 大小，而老年代的可用连续内存空间不足；
4.  老年代已经使用的内存空间超过了`-XX:CMSInitiatingOccupancyFaction`参数指定的比例。

最后，CMS 也存在一些问题，比如 CPU 消耗、Concurrent Mode Failure、内存碎片等，需要通过一些参数进行合理配置，我们后面章节会通过案例进行讲解如何优化。
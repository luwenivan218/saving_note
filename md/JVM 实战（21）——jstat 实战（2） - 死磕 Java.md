> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/2602533778)

> 一、简介上一章，我们通过 jstat 命令分析了 BI 系统中新生代对象的 GC 情况，也就是 YoungGC。本章，我们再来通过 jstat 命令分析下 FullGC 的情况。1.1 案例背景假设现

一、简介
----

上一章，我们通过 jstat 命令分析了 BI 系统中新生代对象的 GC 情况，也就是 Young GC。本章，我们再来通过 jstat 命令分析下 Full GC 的情况。

### 1.1 案例背景

假设现在生产环境有一套 “数据计算系统”，不停地从 MySQL 等各类数据源提取数据到内存中进行计算，系统是分布式的。

每个节点（机器）每分钟执行 100 次操作（提取数据并计算，每次操作耗时 10s），每次操作 1 万条数据，每条数据大小为 1KB 左右，那么每次操作的数据大小就是 10MB：

![](http://image.skjava.com/article/series/jvm/202308102132140941.png)

> 每台机器的配置是 4 核 8G，JVM 分配 4G 内存，其中新生代 1.5G，老年代 1.5G。

### 1.2 内存使用模型估算

每次操作会在 Eden 区分配 10MB 对象，以 1 分钟 100 次操作来算，那么 **Eden 区 1 分钟内就会被占满** ：

![](http://image.skjava.com/article/series/jvm/202308102132145692.png)

每个计算任务处理 1 万条数据耗时 10s，假设此时 80 个计算任务都结束了，还有 20 个计算任务共计 200MB 正在计算中，那么此时 **200MB 对象是存活的** ，不会被 Young GC 回收掉：

![](http://image.skjava.com/article/series/jvm/202308102132150383.png)

由于任何一块 Survivor 区只有 100MB，所以新生代中这存活的 200MB 对象会晋升到老年代，然后清空 Eden：

![](http://image.skjava.com/article/series/jvm/202308102132156674.png)

如此反复，大约经过 7 分钟后，也就是经历了 7 次 Young GC，此时大概有 1.4G 对象在老年代中：

![](http://image.skjava.com/article/series/jvm/202308102132161675.png)

再经过 1 分钟，也就是第 8 分钟结束时，新生代又满了，此时发现老年代可用空间已经不足（剩余 100MB），比历代平均的晋升对象大小（200MB）要小，所以会直接触发一次 Full GC。

Full GC 会先把老年代的垃圾回收了（假设能全部回收），然后执行一次 Young GC，此时 Eden 区存活的对象会进入老年代：

![](http://image.skjava.com/article/series/jvm/202308102132168166.png)

按照这种情况， **每隔 8 分钟左右就会发生一次 Full GC** 。Full GC 的性能是很差的，所以必须进行优化，最基本的优化思路就是扩大 Survivor 区的内存，比如扩到 200MB。这样基本就能避免对象频繁进入老年代，将 Full GC 频率降低到几个小时一次。

二、代码示例
------

我们通过一段代码来模拟下上述情况，先来看下 JVM 参数配置。

### 2.1 JVM 内存参数

`-XX:NewSize=104857600 -XX:MaxNewSize=104857600 -XX:InitialHeapSize=209715200 -XX:MaxHeapSize=209715200 -XX:SurvivorRatio=8 -XX:MaxTenuringThreshold=15 -XX:PretenureSizeThreshold=20971520 -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:gc.log`

上述，我们把 Java 堆内存设置为 200MB，其中年轻代 100MB，Eden 占 80MB，Survivor 各占 10MB，老年代 100MB，大对象阈值为 20MB。

### 2.2 程序源码

```
    public class Demo1 {
        public static void main(String[] args) throws InterruptedException {
            Thread.sleep(30000);     
    
            while (true) {
                loadData();
            }
        }
    
        private static void loadData() throws InterruptedException {
            byte[] data = null;
            for (int i = 0; i < 4; i++) {
                data = new byte[10 * 1024 * 1024];    
            }
            data = null;
    
            byte[] data1 = new byte[10 * 1024 * 1024];
            byte[] data2 = new byte[10 * 1024 * 1024];
    
            byte[] data3 = new byte[10 * 1024 * 1024];
            data3 = new byte[10 * 1024 * 1024];
    
            Thread.sleep(1000);         
        }
    }


```

上述程序代码中，每秒都会执行一次`loadData()`，它会首先分配 4 个 10MB 数组对象，但是立马变成垃圾；然后会有 data1 和 data2 两个 10MB 的数组对象被创建并一直被引用；最后，data3 指向两个新创建的 10MB 数组对象。

总之，`loadData()`的目的就是为了模拟 1s 内创建接近 80MB 对象，触发 Young GC 的。

### 2.3 jstat 分析

当我们启动程序后，main 线程会阻塞 30s，此时我们可以先通过`jps`命令查找当前 JVM 的进程 ID——13740：

![](http://image.skjava.com/article/series/jvm/202308102132174547.png)

然后在 30s 内执行下述命令，统计 JVM 状态，每隔 1s 打印一次，共打印 1000 次：  
`jstat -gc 13740 1000 1000`

我们来看下输出结果：

![](http://image.skjava.com/article/series/jvm/202308102132177898.png)

首先，看下 **EU** 那列，表示 Eden 区的内存使用情况，刚开始一直都是 6MB 多的使用量，此时 main 线程还在阻塞中，当 main 线程恢复后，1 秒钟就发生一次 Young GC，因为 Eden 区只有 80MB。

通过 **OU** 列，明显可以看到老年代新增了 30MB 对象，这就是程序中 data1、data2、data3 引用的存活对象，因为 Eden 区放不下，所以触发了 Young GC，然后又发现存活对象在 Survivor 区也放不下，所以将转移到了老年代：

![](http://image.skjava.com/article/series/jvm/202308102132183989.png)

可以看到，Young GC 和 Full GC 都特别频繁，Full GC 几乎两三秒就会出现一次，而且从耗时看，Full GC 平均耗时 2ms 左右，但是 Young GC 竟然又 7ms，比 Full GC 还高：

![](http://image.skjava.com/article/series/jvm/2023081021321909310.png)

因为上述的每次 Full GC 都是由 Young GC 的，Young GC 时发现存活对象放不进 Survivor，先尝试转移到老年代，但当老年代空间也不足时就会联动触发 Full GC，必须等到 Full GC 完成后，才能将存活对象转移过去，Young GC 才算完成。

三、优化
----

我们来对上述示例进行下优化，主要就是调整 Survivor 区的大小：

`-XX:NewSize=209715200 -XX:MaxNewSize=209715200 -XX:InitialHeapSize=314572800 -XX:MaxHeapSize=314572800 -XX:SurvivorRatio=2 -XX:MaxTenuringThreshold=15 -XX:PretenureSizeThreshold=20971520 -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:gc.log`

上述 JVM 参数，我们把 Java 堆内存调整为 300MB，新生代占 200MB，其中 Eden 区 100MB，Survivor 区各 50MB，老年代 100MB。

### 3.1 jstat 分析

我们根据上述 JVM 参数再重新运行程序，输出结果如下：

![](http://image.skjava.com/article/series/jvm/2023081021321959111.png)

可以看到，Young GC 频率每秒 1 次，每次存活对象大小约 20MB，Survivor 区足够容纳，所以没有触发过 Full GC。而且 15 次 Young GC 耗时才 120ms，也就是平均每次 8ms，所以对系统的运行几乎没有影响。

四、总结
----

本章，我们通过一个示例引出频繁 Full GC 的问题，并通过 jstat 命令观察 JVM 运行情况，然后对 JVM 进行调优，最后再通过 jstat 观察优化后的 JVM 运行情况，将系统的运行效率提升了，避免了频繁 Full GC。

通过本章和前一章的两个示例，相信读者已经掌握了 jstat 的核心用法。从下一章开始，我们会用一系列的真实生产案例还原出各种不同的 JVM 优化场景，帮助大家强化对 JVM 性能问题进行分析和处理的能力。
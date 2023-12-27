> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/2870383999)

> 一、简介本章和下一章，我们将通过之前讲过的两个案例，看看如何在生产环境下使用 jstat 对 JVM 运行情况进行分析。我们先来回顾下商户 BI 系统。1.1 案例背景假设生产环境有一个商户

一、简介
----

本章和下一章，我们将通过之前讲过的两个案例，看看如何在生产环境下使用 jstat 对 JVM 运行情况进行分析。我们先来回顾下[商户 BI 系统](https://www.tpvlog.com/article/98)。

### 1.1 案例背景

假设生产环境有一个商户 BI 系统，用于商户日常经营数据的分析和报表输出，其大致运行逻辑如下：

1.  商户会在业务平台上进行运营，产生各种各样的业务数据；
2.  Hadoop、Spark 等会对这些业务数据进行计算，然后放入 MySQL、HBase 之类的存储中；
3.  最后，我们的 BI 系统会把各种存储好的数据暴露给前端，允许前端基于各种搜索条件筛选和展示。

![](http://image.skjava.com/article/series/jvm/202308102132057641.png)

系统刚上线时，商户数量只有几万家，生产机器配置是 4 核 8G，新生代分配 1.5G，Eden 区有 1G：

![](http://image.skjava.com/article/series/jvm/202308102132063912.png)

### 1.2 内存使用模型估算

每个商户的主页，前端每隔几秒钟就会发送一个请求给 BI 系统，用于生成一种实时报表。每台机器差不多每秒抗 500 个请求，由于报表需要的数据量比较大，一般每个请求需要加载约 100KB 的数据到内存中，每秒 500 个请求总共就是 50MB 数据，每次 Young GC 过后存活对象也就几十 MB：

![](http://image.skjava.com/article/series/jvm/202308102132070503.png)

根据上述内存使用模型的估算，每秒需加载 50MB 数据到 Eden 区，那 3 分钟左右就会将 Eden 区占满，触发 Young GC。在 1G 的内存空间中进行 Young GC 的效率是很高的，基本上 10ms 左右就可以搞定，所以 BI 系统 **每运行几分钟就会出现 10ms 左右的卡顿** ，但是对终端用户和系统运行基本没有影响：

![](http://image.skjava.com/article/series/jvm/202308102132078624.png)

二、代码示例
------

我们通过一段代码来模拟下上述情况，先来看下 JVM 参数配置。

### 2.1 JVM 内存参数

`-XX:NewSize=104857600 -XX:MaxNewSize=104857600 -XX:InitialHeapSize=209715200 -XX:MaxHeapSize=209715200 -XX:SurvivorRatio=8 -XX:MaxTenuringThreshold=15 -XX:PretenureSizeThreshold=3145728 -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:gc.log`

上述，我们把 Java 堆内存设置为 200MB，其中年轻代 100MB，Eden 占 80MB，Survivor 各占 10MB，老年代 100MB。

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
            for (int i = 0; i < 50; i++) {
                data = new byte[100 * 1024];    
            }
            data = null;
            Thread.sleep(1000);         
        }
    }


```

上述程序代码中，`while(true)`用来模拟每秒 50 次请求，每次请求加载 100KB 数据，也就是每秒 5MB 数据。

### 2.3 jstat 分析

当我们启动程序后，main 线程会阻塞 30s，此时我们可以先通过`jps`命令查找当前 JVM 的进程 ID——2236：

![](http://image.skjava.com/article/series/jvm/202308102132084915.png)

然后在 30s 内执行下述命令，统计 JVM 状态，每隔 1s 打印一次，共打印 1000 次：  
`jstat -gc 2236 1000 1000`

我们来看下输出结果：

![](http://image.skjava.com/article/series/jvm/202308102132089426.png)

首先，看下 **EU** 那列，表示 Eden 区的内存使用情况，刚开始一直都是 6MB 多的使用量，此时 main 线程还在阻塞中：

![](http://image.skjava.com/article/series/jvm/202308102132098737.png)

之后，线程恢复运行，Eden 区的使用空间每秒钟都在增长，根据差值计算大概就是每秒 5MB，与我们的代码逻辑吻合：

![](http://image.skjava.com/article/series/jvm/202308102132105918.png)

当 Eden 区使用接近 80MB 时，再要分配就失败了，此时触发了一次 Young GC，Eden 区的使用空间降低到 4573.3KB：

![](http://image.skjava.com/article/series/jvm/202308102132114969.png)

针对上述示例，我们可以通过 jstat 命令清晰的看出，新生代对象增速为 5MB/s 左右，大约十几秒就会触发一次 Young GC，每次 Young GC 回收大约 70MB 空间，耗时 1ms，所以 Young GC 的速度是很快的，即使回收 800MB 空间，也就耗时 10ms 左右。

三、总结
----

本章，我们通过 jstat 命令分析了 BI 系统中新生代对象的 GC 情况。通过 jstat 命令，我们清晰的看到虽然系统每隔十几秒就会进行一次 Young GC，但是 Young GC 耗时很小，而且没有存活对象进入老年代，所以系统运行的效率还是挺高的。
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1938419357)

> 一、简介本章，我们将通过示例代码演示 YoungGC 是如何发生的。同时，我们会讲解如何通过 JVM 参数去配置打印 GC 日日，然后通过 GC 日志分析 JVM 中的 GC 到底是如何运行的。1.1

一、简介
----

本章，我们将通过示例代码演示 Young GC 是如何发生的。同时，我们会讲解如何通过 JVM 参数去配置打印 GC 日日，然后通过 GC 日志分析 JVM 中的 GC 到底是如何运行的。

### 1.1 JVM 内存参数

我们的示例程序基于 JDK1.8，JVM 参数如下：  
`-XX:NewSize=5242880 -XX:MaxNewSize=5242880 -XX:InitialHeapSize=10485760 -XX:MaxHeapSize=10485760 -XX:SurvivorRatio=8 -XX:PretenureSizeThreshold=10485760 -XX:+UseParNewGC -XX:+UseConcMarkSweepGC`

上述参数中：

*   -XX:NewSize=5242880 -XX:MaxNewSize=5242880：新生代大小 5MB
*   -XX:InitialHeapSize=10485760 -XX:MaxHeapSize=10485760：Java 堆内存 10MB
*   -XX:SurvivorRatio=8：Eden 区占 4MB，Survivor 各占 0.5MB
*   -XX:PretenureSizeThreshold=10485760：大对象阈值 10MB
*   -XX:+UseParNewGC -XX:+UseConcMarkSweepGC：新生代使用 ParNew，老年代使用 CMS

![](http://image.skjava.com/article/series/jvm/202308102131446941.png)

### 1.2 GC 日志参数

我们需要在系统的 JVM 参数中加入 GC 日志的打印选型：

*   -XX:+PrintGCDetails：打印详细的 GC 日志
*   -XX:+PrintGCTimeStamps：打印每次 GC 发生的时间
*   -Xloggc:gc.log：设置将 GC 日志写入一个磁盘文件

加入日志参数后，JVM 的参数如下：

`-XX:NewSize=5242880 -XX:MaxNewSize=5242880 -XX:InitialHeapSize=10485760 -XX:MaxHeapSize=10485760 -XX:SurvivorRatio=8 -XX:PretenureSizeThreshold=10485760 -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:gc.log`

二、示例程序
------

### 2.1 程序源码

示例程序代码如下：

```
    public class Demo1 {
        public static void main(String[] args) {
            byte[] array1 = new byte[1024 * 1024];
            array1 = new byte[1024 * 1024];
            array1 = new byte[1024 * 1024];
            array1 = null;
    
            byte[] array2 = new byte[2 * 1024 * 1024];
        }
    }


```

### 2.2 JVM 内存模型

我们根据上述代码来分析下对象是如何在 Eden 区分配的。

首先，main 方法里的第一行代码会为 Eden 区创建一个 1MB 的 byte 数组，第 2-3 行代码 array1 局部变量重新赋值引用：

![](http://image.skjava.com/article/series/jvm/202308102131453082.png)

第 4 行代码`array1=null`，这时之前的 3 个数组对象都失去了引用：

![](http://image.skjava.com/article/series/jvm/202308102131462033.png)

第 5 行代码`byte[] array2 = new byte[2 * 1024 * 1024]`，尝试创建一个 2MB 的对象放入 Eden 区，但是 Eden 区已经空间不足了，所以这时就会触发新生代的 Young GC。

### 2.3 程序执行

当使用 IDE 执行程序时，我们先要进行 JVM 参数配置，以 IDEA 为例，配置如下：

![](http://image.skjava.com/article/series/jvm/202308102131467334.png)

运行完成后，工程目录下会出现 GC 日志文件：gc.log，内容如下：

```
    Java HotSpot(TM) 64-Bit Server VM (25.111-b14) for windows-amd64 JRE (1.8.0_111-b14), built on Sep 22 2016 19:24:05 by "java_re" with MS VC++ 10.0 (VS2010)
    Memory: 4k page, physical 12470176k(6211924k free), swap 14370720k(6127712k free)
    CommandLine flags: -XX:InitialHeapSize=10485760 -XX:MaxHeapSize=10485760 -XX:MaxNewSize=5242880 -XX:NewSize=5242880 -XX:OldPLABSize=16 -XX:PretenureSizeThreshold=10485760 -XX:+PrintGC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:SurvivorRatio=8 -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseConcMarkSweepGC -XX:-UseLargePagesIndividualAllocation -XX:+UseParNewGC 
    0.242: [GC (Allocation Failure) 0.242: [ParNew: 4094K->512K(4608K), 0.0018168 secs] 4094K->1696K(9728K), 0.0020491 secs] [Times: user=0.05 sys=0.02, real=0.00 secs] 
    Heap
     par new generation   total 4608K, used 3702K [0x00000000ff600000, 0x00000000ffb00000, 0x00000000ffb00000)
      eden space 4096K,  77% used [0x00000000ff600000, 0x00000000ff91d978, 0x00000000ffa00000)
      from space 512K, 100% used [0x00000000ffa80000, 0x00000000ffb00000, 0x00000000ffb00000)
      to   space 512K,   0% used [0x00000000ffa00000, 0x00000000ffa00000, 0x00000000ffa80000)
     concurrent mark-sweep generation total 5120K, used 1184K [0x00000000ffb00000, 0x0000000100000000, 0x0000000100000000)
     Metaspace       used 3484K, capacity 4498K, committed 4864K, reserved 1056768K
      class space    used 387K, capacity 390K, committed 512K, reserved 1048576K


```

三、日志分析
------

本节，我们来分析下上述的 gc.log 日志。

### 3.1 GC 情况概览

我们先来看下日志中的如下行，这是本次 GC 情况的概要说明：

```
    0.242: [GC (Allocation Failure) 0.242: [ParNew: 4094K->512K(4608K), 0.0018168 secs] 4094K->1696K(9728K), 0.0020491 secs] [Times: user=0.05 sys=0.02, real=0.00 secs]


```

**GC (Allocation Failure) ：** 说明了为啥发生 GC，因为对象分配失败，也就是上述的 Eden 区空间不足了；

**0.242：** 系统运行了 0.242 秒以后，发生了本次 GC；

**ParNew: 4094K->512K(4608K), 0.0018168 secs：** 使用 ParNew 进行新生代的 GC，GC 前新生代使用了 4094K，GC 完成后新生代使用了 512K，4608K 表示年轻代的总空间（Eden+1 个 Survivor），本次 GC 耗时 0.0018168 秒；

**4094K->1696K(9728K), 0.0020491 secs：** Java 堆内存的总空间为 8728K，GC 前使用了 4094K，GC 后使用了 1696K；

**Times: user=0.05 sys=0.02, real=0.00 secs：** 本次 GC 消耗的时间，与元数据区有关，后续讲解。

根据 GC 日志可以看出，本轮 Young GC 有 512K 对象存活下来了，从 Eden 区转移到了 Survivor 区：

![](http://image.skjava.com/article/series/jvm/202308102131473165.png)

### 3.2 JVM 退出时堆内存

我们接着看下面的 GC 日志，这是 **JVM 退出时当前 Java 堆内存的使用情况** ：

```
    Heap
     par new generation   total 4608K, used 3702K [0x00000000ff600000, 0x00000000ffb00000, 0x00000000ffb00000)
      eden space 4096K,  77% used [0x00000000ff600000, 0x00000000ff91d978, 0x00000000ffa00000)
      from space 512K, 100% used [0x00000000ffa80000, 0x00000000ffb00000, 0x00000000ffb00000)
      to   space 512K,   0% used [0x00000000ffa00000, 0x00000000ffa00000, 0x00000000ffa80000)


```

**par new generation total 4608K, used 3702K：** ParNew 负责的新生代总共有 4608K 内存，目前使用了 3702K；

**eden space 4096K, 77% used：** Eden 总共使用了 4096K

**from space 512K, 100% used：** From Survivor 区使用了 100%（存放转移过来的未知存活对象）

**to space 512K, 0% used：** To Survivor 区未使用

我们接着看：

```
    concurrent mark-sweep generation total 5120K, used 1184K [0x00000000ffb00000, 0x0000000100000000, 0x0000000100000000)
    Metaspace       used 3484K, capacity 4498K, committed 4864K, reserved 1056768K
    class space    used 387K, capacity 390K, committed 512K, reserved 1048576K


```

**concurrent mark-sweep generation total 5120K, used 1184K：** 使用 CMS 管理的老年代总空间为 5210K，已使用 1184K

**Metaspace used 3484K, capacity 4498K, committed 4864K, reserved 1056768K：** 元数据区的空间信息

**class space used 387K, capacity 390K, committed 512K, reserved 1048576K：** Class 空间信息

> JDK1.8 开始，取消了方法区，取而代之的是 Metaspace。Metaspace 直接使用本地内存。默认情况下，其大小会根据使用情况动态调整，也可以使用`-XX:MaxMetaspaceSize`来控制最大内存。
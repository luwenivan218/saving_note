> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1915323613)

> 一、简介上一章，我们已经进行了一次 YoungGC 日志的分析，本章我们继续结合代码示例做实验，来看看对象是如何从新生代进入老年代的。我们之前讲过新生代对象晋升到老年代的几种场景：

一、简介
----

上一章，我们已经进行了一次 Young GC 日志的分析，本章我们继续结合代码示例做实验，来看看对象是如何从新生代进入老年代的。我们之前讲过新生代对象晋升到老年代的几种场景：

*   躲过 15 次 GC
*   符合动态年龄判断规则
*   Young GC 后存活对象放不进 Survivor
*   大对象直接进入老年代

本章，我们通过示例代码模拟最常见的一种场景——Young GC 后存活对象放不进 Survivor。

### 1.1 JVM 内存参数

我们的示例程序基于 JDK1.8，JVM 参数如下：  
`-XX:NewSize=10485760 -XX:MaxNewSize=10485760 -XX:InitialHeapSize=20971520 -XX:MaxHeapSize=20971520 -XX:SurvivorRatio=8 -XX:MaxTenuringThreshold=15 -XX:PretenureSizeThreshold=10485760 -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:gc.log`

上述给新生代分配了 10MB 空间，老年代也是 10MB，参数注意两点：

*   -XX:PretenureSizeThreshold=10485760：超过 10MB 的大对象直接进入老年代
*   -XX:MaxTenuringThreshold=15：对象年龄到达 15 时进入老年代

![](http://image.skjava.com/article/series/jvm/202308102131504531.png)

二、示例程序
------

### 2.1 程序源码

示例程序代码如下：

```
    public class Demo1 {
        public static void main(String[] args) {
            byte[] array1 = new byte[2 * 1024 * 1024];
            array1 = new byte[2 * 1024 * 1024];
            array1 = new byte[2 * 1024 * 1024];
    
            byte[] array2 = new byte[128 * 1024];
            array2 = null;
    
            byte[] array3 = new byte[2 * 1024 * 1024];
        }
    }


```

### 2.2 JVM 内存模型

我们根据上述代码来分析下内存中的对象分配。首先连续创建了三个 2MB 的数组对象，将 array1 指向最后一个数组对象，然后创建了一个 128KB 的数组，将 array2 赋 null：

![](http://image.skjava.com/article/series/jvm/202308102131509362.png)

注意，Eden 区里会有一些 “未知对象”，根据[模拟 Young GC](https://www.tpvlog.com/article/100) 一文中的分析，对象大小在 500KB 左右，我们后续会通过工具分析这些 “未知对象” 到底是什么。

然后，执行代码`byte[] array3 = new byte[2 * 1024 * 1024]`，希望在 Eden 区继续创建一个 2MB 的数组。显然，Eden 区的空间不足了，此时就会触发 Young GC。

### 2.3 程序执行

我们执行程序，得到以下 GC 日志：

```
    0.352: [GC (Allocation Failure) 0.353: [ParNew: 8106K->623K(9216K), 0.0021991 secs] 8106K->2673K(19456K), 0.0033689 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
    Heap
     par new generation   total 9216K, used 2837K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
      eden space 8192K,  27% used [0x00000000fec00000, 0x00000000fee297c0, 0x00000000ff400000)
      from space 1024K,  60% used [0x00000000ff500000, 0x00000000ff59be50, 0x00000000ff600000)
      to   space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
     concurrent mark-sweep generation total 10240K, used 2050K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
     Metaspace       used 3147K, capacity 4496K, committed 4864K, reserved 1056768K
      class space    used 343K, capacity 388K, committed 512K, reserved 1048576K


```

三、日志分析
------

我们先来看下日志中的下面这行，这是本次 GC 情况的概要说明：

```
    0.352: [GC (Allocation Failure) 0.353: [ParNew: 8106K->623K(9216K), 0.0021991 secs] 8106K->2673K(19456K), 0.0033689 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]


```

**ParNew: 8106K->623K(9216K)：** 可以看到，本次 Young GC 后，新生代只剩下了 623KB（未知对象）。但是明明 array1 还引用着一个 2MB 的数组：

![](http://image.skjava.com/article/series/jvm/202308102131516093.png)

我们注意下 Survivor 的大小，只有 1MB，是容纳不下 2MB 数组和未知对象的。根据 “Young GC 后存活对象放不进 Survivor 会进入老年代” 规则，ParNew 会将 2MB 数组转移到老年代，未知对象转移到 Survivor：

![](http://image.skjava.com/article/series/jvm/202308102131522734.png)

通过观察 GC 日志，也印证了这一点：  
**from space 1024K, 60% used：** Survivor 中有 600 多 KB 的数据，就是未知对象；  
**concurrent mark-sweep generation total 10240K, used 2050K：** 老年代中的 2MB 对象就是 array3 引用的数组对象。

四、总结
----

本章通过 GC 日志分析了一种新生代对象进入老年代的示例，即 Young GC 后存活对象放不进 Survivor，则会进行老年代。  
需要注意的是，并不是所有存活对象都会进入老年代，可能会有部分对象留在 Survivor 区，部分对象进入老年代。
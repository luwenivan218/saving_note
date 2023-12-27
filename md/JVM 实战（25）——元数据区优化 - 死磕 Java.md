> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1669770806)

> 一、案例背景本章介绍的案例比较特殊，是由于人为设置 JVM 参数错误，而导致的 JVM 性能问题。首先，生产环境有一个新上线的系统，频繁触发 FullGC 告警。通过 GC 日志，我们发现日志

一、案例背景
------

本章介绍的案例比较特殊，是由于人为设置 JVM 参数错误，而导致的 JVM 性能问题。

首先，生产环境有一个新上线的系统，频繁触发 Full GC 告警。通过 GC 日志，我们发现日志中有大量以下字样：  
`【Full GC(Metadata GC Threshold) xxxxx,xxxxx】`

从这里就知道，频繁的 Full GC 是因为 Metadata 区域（JDK1.8+）被占满。

### 1.1 存在问题

Metadata 区域，也就是元数据区，一般存放着类信息，为什么会被频繁占满，进而触发 Full GC 呢？我们通过工具分析元数据区的内存占用情况，发现元数据区域的内存使用波动曲线类似于下面这样：

![](http://image.skjava.com/article/series/jvm/202308102132557351.png)

也就是说，元数据区的内存占用不断增加，当达到一个顶点后（快占满）就会触发 Full GC，Full GC 会对元数据区域进行垃圾回收，所以接下来元数据区的内存占用就又变小了。

二、元数据区占满
--------

### 2.1 优化前

通过上述的案例背景介绍，已经可以很明显的看出，系统的问题就是 **不断有新的类被加载到元数据区，导致不断地触发 Full GC** ：

![](http://image.skjava.com/article/series/jvm/202308102132562042.png)

_那到底是什么类在不断地被加载呢？_

我们可以通过在 JVM 启动参数中加上以下配置，然后观察 GC 日志：  
`-XX:TraceClassLoading -XX:TraceClassUnloading`

这两个参数，用来追踪类加载和类卸载的情况，GC 日志中会打印出 JVM 加载了哪些类、卸载了哪些类：  
`【Loaded sun.reflect.GeneratedSerializationConstructorAccessor from _JVM_Defined_Class】`

可以看到，JVM 在运行期间不断地加载了大量的 “GeneratedSerializationConstructorAccessor” 类到元数据区，这些类是在程序中使用 Java 的反射时加载的，比如像下面这样：

```
    Method method = XXX.class.getDeclaredMethod(xxx, xxx);
    method.invoke(target, params);


```

**在执行这类反射代码时，JVM 会动态生成一些类放入元数据区：**

![](http://image.skjava.com/article/series/jvm/202308102132568083.png)

_那 JVM 又为什么会不停的创建这些类对象呢？_

首先，我们要明白 Class 对象都是 SoftRefence（软引用），软引用在正常情况下不会被回收。JVM 在 GC 时判断是否回收软引用对象时，采用了一个公式：  
$$  
clock-timestamp ≤ freespace * SoftRefLRUPolicyMSPerMB  
$$  
这个公式的意思是说：clock-timestamp 表示对象最近一次被访问距当前的时间差，freespace 表示 JVM 中的空闲内存大小，SoftRefLRUPolicyMSPerMB 表示每 1MB 空间可以允许 SoftReference 对象存活多久。

举个例子，假如 JVM 中的空闲内存大小为 3000MB，SoftRefLRUPolicyMSPerMB 设置为 1000ms，那么 Class 对象就可以存活：3000*1000=3000 秒，也就是 50 分钟。

SoftRefLRUPolicyMSPerMB 可以通过 JVM 启动参数配置，在上述案例中，这个值被配置成了 0，于是 freespace * SoftRefLRUPolicyMSPerMB=0。这就导致程序通过反射创建出 Class 对象后，立马被回收了，接着 JVM 在反射代码的执行过程中，继续创建这种反射代理类，在 JVM 的机制下，这种类对象会越来越多，直到将元数据区占满。

![](http://image.skjava.com/article/series/jvm/202308102132574924.png)

> 这里其实大家会有个疑问，为什么软引用的 class 对象被回收后，就会导致 JVM 不断的创建更多的新 class 对象。其实这是 JDK 内部的一个缺陷，需要分析 JDK 源码，不再赘述。

### 2.2 优化后

了解了问题的所在，我们就开始针对这个案例进行优化，其实非常简单，之所以出现元数据区的频繁 GC 就是因为 JVM 参数设置不合理，只要把`-XX:SoftRefLRUPolicyMSPerMB=0`这个参数的值设置大一点就可以了，比如 1000、2000、5000。

提高这个数值，就是让反射过程中 JVM 自动创建的软引用的一些 class 对象不要被随便回收，那元数据区中的内存占用也就基本稳定了。

三、总结
----

本章，我们通过示例分析了元数据区被占满导致的频繁 Full GC 问题，通过参数`-XX:SoftRefLRUPolicyMSPerMB`可以配置软引用对象的平均存活时间，从而避免了元数据区频繁被占满。
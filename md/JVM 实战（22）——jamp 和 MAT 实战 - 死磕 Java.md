> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/2020736693)

> 一、简介我们通过 jstat 进行分析，发现 FullGC 非常频繁，基本上每隔两分钟就会执行一次，而且每次 FullGC 的时间长达 10 秒。1.1 案例背景系统的 JVM 内存模型如下，当时给

一、简介
----

我们通过 jstat 进行分析，发现 Full GC 非常频繁，基本上每隔两分钟就会执行一次，而且每次 Full GC 的时间长达 10 秒。

### 1.1 案例背景

系统的 JVM 内存模型如下，当时给 Java 堆内存分配了 20G，其中年轻代 10G，老年代 10G：

![](http://image.skjava.com/article/series/jvm/202308102132230101.png)

事实上，虽然分配了那么大的内存空间给年轻代和老年代，但是通过 jstat 分析发现，Eden 区大概 1 分钟就会被占满，然后触发一次 Young GC，而且 Young GC 过后有几个 G 的对象都会存活并进入老年代：

![](http://image.skjava.com/article/series/jvm/202308102132240862.png)

这说明系统代码运行时会产生大量对象，经常在 1 分钟过后就塞满 Eden，然后会触发 Young GC，但是由于程序处理极慢，导致大量存活对象 Survivor 区无法容纳，从而进入老年代。

由于老年代的内存有 10GB，所以在没有采用 G1 的情况下，一次 Full GC 的回收速度很慢，长达 10s，这就直接导致了工作线程无法正常运行，对于用户来说就是系统卡死。

二、JVM 优化
--------

### 2.1 优化思路

通过上述分析，我们可以判断一定是程序代码的某处在不断生成各种对象，导致系统加载过多数据到内存中。所以，要对这个案例进行优化，就必须分析到底是程序哪里在源源不断地创建对象。

我们可以先通过 jmap 生成一个 JVM 内存快照文件，然后通过 MAT 进行分析。下面我们通过一段示例代码来排查：

```
    public class Demo1{
        public static void main(String[] args){
            List<Data> datas = new ArrayList<>();
            for(int i=0; i<10000; i++){
                datas.add(new Data());
            }
            Thread.sleep(1 * 60 * 60 * 1000);
        }
    }


```

### 2.2 生成 JVM 内存快照

首先执行上述这段程序，通过`jps`获取 JVM 进程 ID——1177：

![](http://image.skjava.com/article/series/jvm/202308102132255233.png)

然后执行 jmap 命令导出 JVM 内存快照：  
`jmap -dump:live,format=b,file=dump.hprof 1177`

### 2.3 MAT 分析

线上 dump 出来的内存快照一般都有几个 G，比如我们上述的程序就有 8 个多 G 的内存快照，所以运行 MAT 时，务必将`MemoryAnalyzer.ini`中的启动堆大小设置为 8G 以上：

![](http://image.skjava.com/article/series/jvm/202308102132262604.png)

启动 MAT 后，选择 “Leak Suspects”，也就是内存泄漏分析，接着我们会看到下面的图：

![](http://image.skjava.com/article/series/jvm/202308102132266735.png)

![](http://image.skjava.com/article/series/jvm/202308102132281946.png)

“Problem Suspect1” 告诉我们：main 线程通过局部变量引用占据内存 24.97% 的对象，而且占据内存的是一个`java.lang.Object[]`数组。

我们可以通过 “Detail” 链接进去查看这个数组到底是什么，通过这个详细说明，我们可以看到 mian 线程中引用的是一个`java.util.ArrayList`，里面的每个元素都是`Demo1$Data`对象：

![](http://image.skjava.com/article/series/jvm/202308102132337227.png)

然后，知道了这些不断创建的对象是什么后，我们还希望知道程序是在哪段代码创建了这些对象。如下图所示，先点击页面中的 “See stacktrace” 链接，就会进入一个线程执行代码堆栈的调用链：

![](http://image.skjava.com/article/series/jvm/202308102132359008.png)

![](http://image.skjava.com/article/series/jvm/202308102132393889.png)

可以看到，问题定位到了 Demo1 类的 main 方法内的第 12 行，最终发现是这个线程执行了`String.split()`方法导致产生了大量的对象。

### 2.4 问题解决

_为什么`String.split()`方法会造成内存泄漏呢？_

在 JDK1.6 以前，`String.split()`方法对于 “Hello World Ressmix” 这种字符串，底层是基于一个数组来存放的，比如[H,e,l,l,o, ,W,o,r,l,d, ,R,e,s,s,m,i,x]，当基于空格切割时，比如“Hello”，不会存到一个新的数组中，而是采用偏移量来表明是对应原数组中的那一段。

但是 JDK1.7 以后，每个切割出来的子字符串都对应一个全新的数组。

所以，上述案例中程序的问题就是加载了大量数据出来，可能一次几十万条，然后通过 split 对这些字符串进行切割，导致字符串数组对象暴增几十倍，这就是为什么系统会频繁产生大量对象的原因。

解决方案就是对`String.split()`处的代码进行优化，避免同时加载大量数据并进行切割。

三、总结
----

本章通过一个内存泄漏的案例，讲解了分析此类问题的思路和解决方法。jmap 和 MAT 经常组合在一起使用，用于线上问题此类的排查。
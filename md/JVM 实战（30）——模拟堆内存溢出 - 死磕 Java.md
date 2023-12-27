> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1459474513)

> 一、简介本章，我们将通过示例代码演示 Java 堆内存区域是如何发生内存溢出的，并根据内存快照进行分析。我们回顾下堆内存溢出的一个场景：系统负载很高，不停的在 Eden 区创建新对象，

一、简介
----

本章，我们将通过示例代码演示 Java 堆内存区域是如何发生内存溢出的，并根据内存快照进行分析。

我们回顾下堆内存溢出的一个场景：系统负载很高，不停的在 Eden 区创建新对象，直到触发 Young GC，但是由于并发太高，Young GC 发现 Eden 区存活对象非常多，Survivor 无法容纳，只能把大批存活对象转移到老年代。经过几次这种 Young GC 之后，老年代也满了，于是触发 Full GC，但是 Full GC 之后老年代里还是塞满了对象，导致 Young GC 过后的存活对象无处可安放，最终引发堆内存溢出。

二、示例程序
------

### 2.1 程序源码

```
    package com.ressmix.jvm;
    
    import java.util.ArrayList;
    import java.util.List;
    
    public class Demo3 {
        public static void main(String[] args) {
            long counter = 0L;
            List<Object> list = new ArrayList<Object>();
    
            while (true) {
                list.add(new Object());
                System.out.println("当前创建了第" + ++counter + "个对象");
            }
        }


```

上述代码很简单，就是不停的创建对象，由于对象由一个 while 循环外部的 List 引用着，且 main() 方法是无限执行的，所以这些创建的对象始终不会被回收掉。这样最终 Eden 区、老年代的空间都会被占满。

### 2.2 JVM 参数

我们设置下堆内存的总大小为 5MB：`-Xms5m -Xmx5m -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:+PrintGCDetails -Xloggc:gc.log -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=./`，这样就可以快速触发堆内存溢出。程序执行后的打印日志输出如下：

```
    当前创建了第160063个对象
    当前创建了第160064个对象
    当前创建了第160065个对象
    java.lang.OutOfMemoryError: Java heap space
    Dumping heap to ./\java_pid6412.hprof ...
    Heap dump file created [7250823 bytes in 0.058 secs]
    Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
        at java.util.Arrays.copyOf(Arrays.java:3210)
        at java.util.Arrays.copyOf(Arrays.java:3181)
        at java.util.ArrayList.grow(ArrayList.java:261)
        at java.util.ArrayList.ensureExplicitCapacity(ArrayList.java:235)
        at java.util.ArrayList.ensureCapacityInternal(ArrayList.java:227)
        at java.util.ArrayList.add(ArrayList.java:458)
        at com.ressmix.jvm.Demo3.main(Demo3.java:12)


```

可以看到，在 5MB 的堆内存中，不断创建 Obejct 对象，当创建到第 160065 个对象时，堆内存实在放不下了，从而引发`java.lang.OutOfMemoryError`。

三、问题分析
------

我们通过程序日志中的报错`“java.lang.OutOfMemoryError: Java heap space”`，知道发生了 Java 堆内存溢出。针对 Java 堆内存溢出，一般不用看 GC 日志，因为堆内存溢出伴随的 GC 日志内容会非常多，我们直接分析 dump 出的内存快照即可。

采用 MAT 打开内存快照`java_pid6412.hprof`（进程名不同，文件名也略有区别）：

![](http://image.skjava.com/article/series/jvm/202308102133156321.png)

可以看到一大堆对象占据了 81.19% 的堆内存，我们直接点击`"See stacktrace"`，看看这些对象是在程序哪里创建出来的：

![](http://image.skjava.com/article/series/jvm/202308102133160782.png)

很显然，main 方法的第 12 行，一直调用`list.add(new Object())`，由此直接引发了内存溢出，我们只要有针对性的修复我们代码的 bug 即可。

四、总结
----

本章，我们通过代码示例模拟了堆内存溢出的场景，Java 堆内存也是最容易出现内存溢出的区域。基本的分析思路就是 dump 出事发现场的内存快照，然后通过 MAT 进行查看，分析出内存占用最多的对象，然后分析线程调用栈，找到代码位置，最后进行优化即可。

从下一章开始，我们将给出真实的生产环境案例，看看这些案例是如何引起 OOM 以及排查和解决的思路方法。
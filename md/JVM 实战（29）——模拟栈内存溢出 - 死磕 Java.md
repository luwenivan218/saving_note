> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1455771773)

> 一、简介本章，我们将通过示例代码演示 Java 虚拟机栈区域是如何发生内存溢出的，并根据内存快照进行分析。我们回顾下栈内存溢出的一个场景：每个线程的栈内存是固定的，如果某个线程不停

一、简介
----

本章，我们将通过示例代码演示 Java 虚拟机栈区域是如何发生内存溢出的，并根据内存快照进行分析。

我们回顾下栈内存溢出的一个场景：每个线程的栈内存是固定的，如果某个线程不停的无限制调用方法，每次方法调用都会有一个栈帧入栈，此时就会导致线程的栈内存被耗尽。

二、示例程序
------

### 2.1 程序源码

```
    package com.ressmix.jvm;
    
    public class Demo2 {
        public static long counter = 0L;
    
        public static void main(String[] args) {
            work();
        }
    
        private static void work() {
            System.out.println("第" + ++counter + "次调用work方法");
            work();
        }
    }


```

上述代码非常简单，就是无限制的递归调用 work 方法。

### 2.2 JVM 参数

我们设置 JVM 栈内存大小为 1MB：`-XX:ThreadStackSize=1m`，然后执行程序，输出打印日志如下：

```
    第6226次调用work方法
    Exception in thread "main" java.lang.StackOverflowError
        at sun.nio.cs.UTF_8$Encoder.encodeLoop(UTF_8.java:691)
        at java.nio.charset.CharsetEncoder.encode(CharsetEncoder.java:579)
        at sun.nio.cs.StreamEncoder.implWrite(StreamEncoder.java:271)
        at sun.nio.cs.StreamEncoder.write(StreamEncoder.java:125)
        at java.io.OutputStreamWriter.write(OutputStreamWriter.java:207)
        at java.io.BufferedWriter.flushBuffer(BufferedWriter.java:129)
        at java.io.PrintStream.write(PrintStream.java:526)
        at java.io.PrintStream.print(PrintStream.java:669)
        at java.io.PrintStream.println(PrintStream.java:806)
        at com.ressmix.jvm.Demo2.work(Demo2.java:11)
        at com.ressmix.jvm.Demo2.work(Demo2.java:12)
        at com.ressmix.jvm.Demo2.work(Demo2.java:12)
        at com.ressmix.jvm.Demo2.work(Demo2.java:12)
        at com.ressmix.jvm.Demo2.work(Demo2.java:12)
        at com.ressmix.jvm.Demo2.work(Demo2.java:12)
        at com.ressmix.jvm.Demo2.work(Demo2.java:12)
        at com.ressmix.jvm.Demo2.work(Demo2.java:12)
        at com.ressmix.jvm.Demo2.work(Demo2.java:12)
        at com.ressmix.jvm.Demo2.work(Demo2.java:12)


```

可以看到，当执行到第 5931 次递归调用时，发生了栈内存溢出——`java.lang.StackOverflowError`。

三、问题分析
------

首先明确一点，GC 日志和 dump 快照仅仅对 Java 堆内存的问题分析有效，就线程的栈内存和栈帧而言，是不存在所谓的 GC 的。所以，分析栈内存溢出最直接有效的方法就是看程序的本地日志：

```
    at com.ressmix.jvm.Demo2.work(Demo2.java:11)
    at com.ressmix.jvm.Demo2.work(Demo2.java:12)
    at com.ressmix.jvm.Demo2.work(Demo2.java:12)
    at com.ressmix.jvm.Demo2.work(Demo2.java:12)
    at com.ressmix.jvm.Demo2.work(Demo2.java:12)
    at com.ressmix.jvm.Demo2.work(Demo2.java:12)
    at com.ressmix.jvm.Demo2.work(Demo2.java:12)
    at com.ressmix.jvm.Demo2.work(Demo2.java:12)


```

程序日志大量报错`at com.ressmix.jvm.Demo2.work(Demo2.java:12)`，其实已经告诉我们了程序的问题所在——无限次调用 work 方法。

四、总结
----

本章，我们通过代码示例模拟了栈内存溢出的场景，大家可以看到 1MB 的栈内存大约可以支撑 5000 次的递归调用，这个数量已经很高了，一般的方法根本不可能出现连续几千次的调用。所以，栈内存溢出在生产环境是很少出现的，即使有，一般都是程序 bug 导致的。

我们在给 Java 虚拟机栈分配内存的时候，要根据 JVM 的线程数合理分配，一般来说每个线程 1MB 的栈内存是足够了，剩下的就是合理预估总线程数。基本上，线程主要来自以下几部分：

*   JVM 进程自带的一些后台线程
*   程序依赖的第三方组件创建的后台线程
*   Web 容器的工作线程
*   程序自己创建的一些额外线程

一般来说，一个 JVM 中上述这些线程总数不会超过 1000 个，我们以 1000 个来算，每个线程 1MB 栈内存，总共分配 1G 的空间给 JVM 栈内存就足够了。
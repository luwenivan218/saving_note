> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1642687841)

> 一、简介我们在 JVM 垃圾回收机制一章中，简单介绍了 JVM 的垃圾回收机制，先来回顾下，系统运行时创建的对象优先在 Java 堆内存区域分配：然后新生代里的对象越来越多，当快满了的时候

一、简介
----

我们在 [JVM 垃圾回收机制](https://www.tpvlog.com/article/87)一章中，简单介绍了 JVM 的垃圾回收机制，先来回顾下，系统运行时创建的对象优先在 Java 堆内存区域分配：

![](http://image.skjava.com/article/series/jvm/202308102126598921.png)

然后新生代里的对象越来越多，当快满了的时候就会触发 “Minor GC”，把新生代中的一些对象回收掉：

![](http://image.skjava.com/article/series/jvm/202308102127005722.png)

那么这里就涉及一个问题： _**JVM 如何知道要去回收哪些对象？**_ 这其实就是 JVM 的对象存活判定机制，主要涉及两种算： **可行性分析算法** 和 **引用计数算法** 。

> 引用计数算法，是给对象添加一个引用计数器，每当有一个地方引用它时，计数器就加 1，当引用失效时，计数器值就减 1，任何时刻计数器为 0 的对象就是可以被回收的。
> 
> 由于 Java 语言没有选用引用计数法来管理 JVM 内存，所以本文不赘述，而且引用计数法不能很好的解决循环引用的问题（Python 采用的是引用计数法）。

二、可达性分析算法
---------

可达性分析算法（GC Root Tracing ），其基本思路就是通过一系列的名为 " **GC Roots** " 的对象作为起始点，从这些起始点开始搜索，搜索所走过的路径称为引用链（Reference Chain），当一个对象到 GC Roots 没有任何引用链相连时（即从 **GC Roots** 到这个对象不可达），则证明此对象是不可用的，就可以被回收。

GC Roots 包括：

*   Java 虚拟机栈中的局部变量（指向着 GC 堆里的对象）；
*   VM 的一些静态数据结构里指向 GC 堆里的对象的引用，例如 HotSpot VM 里的 Universe 里有很多这样的引用；
*   所有当前被加载的 Java 类（看情况）；
*   Java 类的运行时常量池里的引用类型常量；
*   String 常量池（StringTable）里的引用。

可达性分析算法最难理解的就是该选取哪些对象作为 GC Roots，我们通过两个示例来看下。

### 2.1 示例一

下面是最常见的一种情况：

```
    public class Kafka {
        public static void main(String[] args) {
            loadReplicasFromDisk();
        }
        public static void loadReplicasFromDisk(){
            ReplicaManager replicaManager = new ReplicaManager();
        }
    }


```

当执行到 loadReplicasFromDisk() 时，对应的 JVM 内存数据结构如下图：

![](http://image.skjava.com/article/series/jvm/202308102127016903.png)

假如此时新生代的内存已经快满了，发生了 “Minor GC”，那么 JVM 会分析 ReplicaManager 对象的可达性，发现它被“replicaManager” 这个局部变量引用着，在 JVM 规范中， **局部变量是可以作为 GC Roots 的** ，所以就不会被回收。

### 2.2 示例二

另一种比较常见的情况，是下面这种样子：

```
    public class Kafka {
        public static ReplicaManager replicaManager = new ReplicaManager();
    }


```

对应的 JVM 内存数据结构如下图：

![](http://image.skjava.com/article/series/jvm/202308102127069794.png)

假如此时新生代的内存已经快满了，发生了 “Minor GC”，那么 JVM 会分析 ReplicaManager 对象的可达性，发现它被“replicaManager” 这个方法区中的静态变量引用着，在 JVM 规范中， **静态变量是可以作为 GC Roots 的** ，所以就不会被回收。

三、Java 引用类型
-----------

可达性分析与 Java 的引用类型有关联，为了更好的管理对象的内存，更好的进行垃圾回收，JVM 团队扩展了引用类型，从最早的强引用类型增加到 **强引用** 、 **软引用** 、 **弱引用** 、 **虚引用** 四个引用类型：

![](http://image.skjava.com/article/series/jvm/202308102127089035.png)

### 3.1 强引用（StrongReference）

默认的对象都是强引用类型，如果 JVM 在对象存活判定时，通过 GC Roots 可达性分析结果为可达，表示引用类型仍然被引用着，这类对象始终不会被垃圾回收器回收。比如下面这段代码：

```
    public class Kafka {
        public static ReplicaManager replicaManager = new ReplicaManager();
    }


```

### 3.2 软引用（SoftReference）

在 JVM 内存充足的情况下，软引用是不会被 GC 回收的， **只有在 JVM 内存不足的情况下，才会被 GC 回收** 。比如下面这段代码：

```
    public class Kafka {
        public static SoftReference<ReplicaManager> replicaManager = new SoftReference<ReplicaManager>(new ReplicaManager());
    }


```

**适用场景：** 网页缓存、图片缓存

### 3.3 弱引用（WeakReference）

不论当前 JVM 内存是否充足，都 **只能存活到下一次垃圾收集之前** ，说的直白点，只要发生 GC 弱引用对象就会被回收，比如下面这段代码：

```
    public class Kafka {
        public static WeakReference<ReplicaManager> replicaManager = new WeakReference<ReplicaManager>(new ReplicaManager());
    }


```

> ThreadlLocal 中定义的 ThreadLocalMap 就使用到的弱引用。ThreadLocalMap 的 Entry，其 Key 就是一个弱引用对象，读者可以参考我的[《Java 多线程系列》](https://www.tpvlog.com/article/17)。

### 3.4 虚引用（PhantomReference）

虚引用，不会影响对象的生命周期，所持有的引用就跟没持有一样，随时都能被 GC 回收。在使用虚引用时，必须和 **引用队列** 关联使用。其使用场景是用来跟踪对象被垃圾回收器回收的活动。

在对象的垃圾回收过程中，如果 GC 发现一个对象还存在虚引用，则会把这个 **虚引用加入到与之关联的引用队列** 中。

程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收。如果程序发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象内存被回收之前采取必要的行动防止被回收。

四、finalize 方法
-------------

大家理解完了 GC Roots 和引用类型的概念，基本就都知道了哪些对象可以被回收，哪些对象不可以被回收：

**有 GC Roots 引用的对象不能回收，没有 GC Roots 引用的对象，如果是软引用或弱引用，可能会被回收。**

真正的回收环节，待被回收的对象其实还有一次机会拯救自己，那就是对象的 finalize() 方法。我们通过一段代码示例来看下：

```
    public class ReplicaManager {
        public static ReplicaManager instance;
    
        @Override
        protected void finalize() throws Throwable {
            ReplicaManager.instance = this;
        }
    }


```

假如有一个 ReplicaManager 对象马上就要被回收了（此时已经没有 GC Roots 到达它的链路），此时 GC 会首先调用下该对象的 finalize() 方法，看看它是否找了一个新的 GC Roots 来引用自己，比如上述代码中，GC 发现有个静态变量 instance 引用了该实例，那 GC 就不会去回收它。

> finalize 方法没事不要去重写，这都是 GC 内部的机制，平时也几乎不用。
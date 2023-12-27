> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1232066187)

> 一、简介 JVM 会加载类到内存中，所以 JVM 中必然会有一块内存区域来存放我们写的那些类。Java 中有类对象、普通对象、本地变量、方法信息等等各种对象信息，所以 JVM 会对内存区域进

一、简介
----

JVM 会加载类到内存中，所以 JVM 中必然会有一块内存区域来存放我们写的那些类。Java 中有类对象、普通对象、本地变量、方法信息等等各种对象信息，所以 JVM 会对内存区域进行划分：

![](http://image.skjava.com/article/series/jvm/202308102125454151.png)

> JDK1.8 及以后，上图中的方法区变成了 Metaspace——元数据区。

我们本章的目的，就是介绍 JVM 中各块内存区域的功能，其中都是存放的哪些 java 对象信息。

二、方法区
-----

方法区只存在于 JDK1.8 以前的版本，主要是存储从”.class“文件里加载进来的类，包括 **类的名称** 、 **方法信息** 、 **字段信息** 、 **静态变量** 、 **常量** 以及 **编译器编译后的代码** 等。从 JDK1.8 开始，这块区域的名字改成了元数据区（Metaspace），元数据区直接使用本地内存。

默认情况下，元数据区会根据使用情况动态调整，避免了在 JDK1.8 以前由于加载类过多从而出现 java.lang.OutOfMemoryError: PermGen。但也不能无限扩展，因此可以使用 -XX:MaxMetaspaceSize 来控制最大内存。

以上一章的示例来看，Kafka.class 和 ReplicaManager.class 加载到 JVM 后，会放到方法区中：

```
    public class Kafka {
        public static void main(String[] args) {
            ReplicaManager manager = new ReplicaManager();
        }
    }


```

方法区 / 元数据区是所有线程共享的：

![](http://image.skjava.com/article/series/jvm/202308102125462792.png)

三、程序计数器
-------

程序计数器，用来记录当前线程正在执行的字节码指令。我们还是继续以上一章的代码作为示例来讲解：

```
    public class Kafka {
        public static void main(String[] args) {
            ReplicaManager manager = new ReplicaManager();
            manager.loadReplicaFromDisk();
        }
    }


```

首先，上面这段. java 源程序会被编译成. class 文件，.class 中存放的是 JVM 可以读懂的字节码，比如下面这样

```
    public java.lang.String getName();
        descriptor: ()Ljava/lang/String;
        flags: ACC_PUBLIC
        Code:
            stack=1, locals=1, args_size=1
                0: aload_0
                1: get_field    
                4: areturn


```

当 JVM 加载类信息到内存之后，实际就会使用自己的 _**字节码执行引擎**_ ，去执行这些字节码指令，如下图：

![](http://image.skjava.com/article/series/jvm/202308102125476513.png)

程序计数器的作用就在这里，它会 **记录当前执行的字节码指令的位置** ，如下图：

![](http://image.skjava.com/article/series/jvm/202308102125501264.png)

程序计数器是 **线程私有** 的，也就是说每个线程都有个自己的程序计数器，记录当前线程执行到了哪一条字节码指令：

![](http://image.skjava.com/article/series/jvm/202308102125508555.png)

四、Java 虚拟机栈
-----------

Java 虚拟机栈，其实是一种表示 Java 方法执行的数据结构。每个方法被执行的时候，都会创建一个栈帧（Stack Frame）用于存储 **局部变量表** 、 **操作栈** 、 **动作链接** 、 **方法出口** 等信息。每个方法从被调用到执行完成的过程，其实就是一个栈帧在虚拟机栈中从入栈到出栈的过程。

下面的这段程序，肯定有一个 main 线程来执行 main() 方法里面的代码，方法内部我们通常会定义一些局部变量，比如 manager，JVM 中必须有一块区域来保存方法中的这些数据，这个就是 Java 虚拟机栈，Java 虚拟机栈是 **线程私有** 的。

```
    public class Kafka {
        public static void main(String[] args) {
            ReplicaManager manager = new ReplicaManager();
            manager.loadReplicaFromDisk();
        }
    }


```

```
    public class ReplicaManager {
        public static void loadReplicaFromDisk() {
            Boolean hashFinishedLoad = false;
        }
    }


```

比如 main 线程执行了 main() 方法，那么就会创建一个栈帧（里面存放 manager 局部变量），并将其压入 main 线程自己的 Java 虚拟机栈中，如下图：

![](http://image.skjava.com/article/series/jvm/202308102125516786.png)

然后 main 线程继续执行 loadReplicaFromDisk 方法，遇到方法内部的 hashFinishedLoad 局部变量，就会再创建一个栈帧，压入自己的虚拟机栈中：

![](http://image.skjava.com/article/series/jvm/202308102125530467.png)

上述就是 JVM 中的”Java 虚拟机栈 “这个组件的作用： **调用任何方法时，为方法创建栈帧然后入栈，栈帧里存放了这个方法对应的局部变量之类的数据（也包括方法执行的其它相关信息），方法执行完毕后就出栈。**

![](http://image.skjava.com/article/series/jvm/202308102125547138.png)

五、Java 堆内存
----------

Java 堆内存，这是 JVM 内存区域中最重要的一块区域，存放着各种 Java 对象，是线程共享区域。

下面代码中，new ReplicaManager() 创建了一个对象实例，这个对象实例的相关信息就存放在 Java 堆内存中：

```
    public class Kafka {
        public static void main(String[] args) {
            ReplicaManager manager = new ReplicaManager();
            manager.loadReplicaFromDisk();
        }
    }


```

main 线程在执行 main() 方法时，会为其创建一个栈帧并入栈，栈帧中的局部变量 manager 存放着 ReplicaManager 对象实例在 Java 堆内存中的地址：

![](http://image.skjava.com/article/series/jvm/202308102126012599.png)

六、本地方法栈
-------

本地方法栈，其作用和 Java 虚拟机栈类似，区别在于本地方法栈是为虚拟机所使用到的 **Native 方法** 服务，而 Java 虚拟机栈为虚拟机执行 Java 方法 (也就是字节码) 服务。本地方法栈也是线程私有的。

JDK 中的很多底层 API，比如 IO、NIO、网络等，如果大家去看它的源码，会发现很多地方是调用的 native 修饰的方法，比如下面这样：

```
    public native int hashCode();


```

在调用 native 方法时，也会有线程对应的栈来保存 native 方法底层用到的局部变量表之类的信息，这就是本地方法栈的作用。

七、总结
----

本章，我们通过代码的执行流程讲解了 JVM 的内存模型，读者需要重点关注方法区、程序计数器、Java 虚拟机栈、Java 堆内存与程序执行逻辑的关系，其中 Java 堆内存是我们后面章节要关注的重点区域。

![](http://image.skjava.com/article/series/jvm/2023081021260414010.png)
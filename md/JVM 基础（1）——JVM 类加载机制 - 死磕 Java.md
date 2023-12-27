> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1784752032)

> 一、简介首先，我们来简单看下 Java 程序的执行流程：上图中，典型的 Java 程序执行流程如下：我们在本地编写完 Java 源程序；IDE 自动帮我们编译成. class 文件（也可以手动通

一、简介
----

首先，我们来简单看下 Java 程序的执行流程：

![](http://image.skjava.com/article/series/jvm/202308102125242341.png)

上图中，典型的 Java 程序执行流程如下：

1.  我们在本地编写完 Java 源程序；
2.  IDE 自动帮我们编译成. class 文件（也可以手动通过 javac 命令编译），然后打包成 jar 包或者 war 包；
3.  接着，执行 java -jar 命令或直接部署到 web 容器中来运行程序；
4.  运行时，OS 会启动一个 JVM 进程，JVM 会采用 **类加载器** 将各种. class 文件中包含的 Java 类加载到内存中；
5.  最后，JVM 基于自己的 **字节码执行引擎** ，来执行加载到内存中的那些类。

二、类加载机制
-------

Java 的类加载机制远没有第一节中描述的那么简单，上述只是让读者了解下整体流程，本节，我们就深入内部，讲解下 Java 的类加载机制的内部原理。

### 2.1 完整流程

类从. class 二进制数据被加载到 JVM 内存中开始，到卸载出内存为止，它的整个生命周期包括：

_加载（Loading）_、_验证（Verification）_、_准备 (Preparation)_、_解析 (Resolution)_、_初始化 (Initialization)_、_使用 (Using)_、_卸载 (Unloading)_，共 7 个阶段。

![](http://image.skjava.com/article/series/jvm/202308102125256122.png)

* 加载（Loading）* 阶段很简单，当程序执行到需要的类时，JVM 就会通过 _**类加载器**_ 将其加载到内存中。接下来，我们先看下什么是类加载器，然后详细讲解整个类加载流程。

### 2.2 类加载器

类加载器可以大致划分为以下三类：

#### Bootstrap ClassLoader

主要负责加载 JDK 安装目录下的核心类库（比如 / lib 目录下的类），这些核心类库是 JVM 运行时自身需要用到的。

> Bootstrap ClassLoader 采用 C++ 语言实现，也是 JVM 自身的一部分，开发者不能直接在 Java 程序中使用。

#### Extension ClassLoader

主要负责加载 JDK 安装目录下的扩展类库（比如 / lib/ext 目录下的类），这些扩展类库是 JDK 按照功能进行模块划分的，一般也是 Java 程序运行所必需的。

> 开发者可以在 Java 程序中直接使用 Extension ClassLoader。

#### Application ClassLoader

负责加载用户类路径（classpath）所指定的类，可以简单的理解成负责加载用户自己开发的 Java 类。

> 开发者可以在 Java 程序中直接使用 Extension ClassLoader，这也是默认的类加载器。

除了上述提供到三种类加载器外，开发者也可以自定义类加载器，根据自己的需求去加载类。

### 2.3 双亲委派机制

JVM 的类加载器是有亲子层级结构的，层级结构如下图：

![](http://image.skjava.com/article/series/jvm/202308102125263163.png)

当我们的类加载器需要加载一个类时，首先会委派给自己的父类加载器去加载，最终传到到顶层 的类加载器去加载；如果某个父类加载器发现在 **自己负责的范围内** 并没有找到这个类，就会下推加载权力给自己的子类加载器。

以上图为例：

1.  当 Application ClassLoader 加载一个 class 时，它首先不会自己去尝试加载这个类，而是把类加载请求委派给父类加载器 Extension ClassLoader 去完成；
2.  当 Extension ClassLoader 加载一个 class 时，它首先也不会自己去尝试加载这个类，而是把类加载请求委派给 Bootstrap ClassLoader 去完成；
3.  如果 Bootstrap ClassLoader 加载失败（例如在 $JAVA_HOME/jre/lib 里未查找到该 class），会使用 Extension ClassLoader 来尝试加载；
4.  Extension ClassLoader 也加载失败，则会使用 Application ClassLoader 来加载，如果 Application ClassLoader 也加载失败，则会报出 ClassNotFoundException 异常。

#### 优点

双亲委派机制的优点很明显，可以 **避免类的重复加载** ，当父亲已经加载了该类时，子 ClassLoader 就没有必要再加载一次。

另外，考虑到 **安全因素** ，Java 核心 api 中的类不会被随意替换：假设通过网络传递一个名为 java.lang.Integer 的类，通过双亲委托模式传递到 Bootstrap ClassLoader，发现在核心 Java API 中已经有这个类了，就并不会重新加载网络传递过来的 java.lang.Integer，而直接返回已加载过的 Integer.class，这样便可以防止核心 API 库被随意篡改。

### 2.4 设计类加载器

下面我们通过一个示例，更好地理解下双亲委派机制。Tomcat 是常用的 web 容器，本身是用 Java 实现的，当我们的程序以 war 包部署到 tomcat 后，tomcat 启动后的内部 JVM 需要加载我们程序中的. class 文件。那么 Tomcat 的类加载机制应该如何设计，才能动态加载我们 war 包中的类到 tomcat 自身的 JVM 中去呢？

首先，Tomcat 的类加载体系如下图，蓝色部分是 Tomcat 继承 Application ClassLoader 实现的自定义类加载器：

![](http://image.skjava.com/article/series/jvm/202308102125280304.png)

Common、Catalina、Shared 类加载器用来加载 Tomcat 自身的一些核心基础类库。同时，Tomcat 为每一个部署在其内的 web 应用都分配了一个对应的 WebApp 类加载器，就是这个类加载器负责加载我们部署的这个 web 应用的类，每一个 WebApp 只负责加载自己对应的那个 web 应用的 class 文件，不会传导给上层类加载器去加载。所以， **Tomcat 的类加载器设计其实是打破了双亲委派机制的** 。

至于 Jsp 类加载器，则是给每一个 JSP 都准备了一个 Jsp 类加载器。

三、类加载过程
-------

### 3.1 验证阶段

根据 [Java 虚拟机规范](https://docs.oracle.com/javase/specs/)，需要对加载进来的 “.class” 文件的内容进行校验，包括验证文件格式、元数据、字节码、符号引用等各种信息，以确认是否符合指定的规范。

验证阶段就是用来做这个事情的，来看下下面的代码：

```
    public class Kafka {
        public static void main(String[] args) {
            ReplicaManager manager = new ReplicaManager();
        }
    }


```

代码示例中，Kafka 类用到了 ReplicaManager 类，所以它们都会在被加载进 JVM 后进行验证：

![](http://image.skjava.com/article/series/jvm/202308102125285205.png)

### 3.2 准备阶段

准备阶段，主要是为类及其静态字段分配内存，并将其初始化为默认值。比如，下面的 ReplicaManager 类：

```
    public class ReplicaManager {
        public static int flushInterval;
    }


```

当加载阶段、验证阶段都执行完成后，JVM 会给类的静态字段分配内存空间，上述代码就是给 flushInternal 字段赋默认值 0，整个过程如下图：

![](http://image.skjava.com/article/series/jvm/202308102125291656.png)

### 3.3 解析阶段

解析阶段，实际上是把 **类的符号引用替换为直接引用** 的过程，这一过程底层非常复杂，我们后续章节将进行专门讲解。

![](http://image.skjava.com/article/series/jvm/202308102125298287.png)

### 3.4 初始化阶段

之前说过，JVM 会在准备阶段给类的静态字段分配空间和默认值。而在初始化阶段，就会正式执行类的初始化代码，对类进行初始化操作。什么是初始化代码？我们来看下下面这段代码理解下：

```
    public class ReplicaManager {
        public static int flushInterval = Configuration.getInt("replica.flush.interval");
        public static Map<String,Replica> replicas;
    
        static {
            loadReplicaFromDish():
        }
    
        public static void loadReplicaFromDish(){
            this.replicas = new HashMap<String,Replica>();
        }
    }


```

对于 flushInternal 变量，我们通过一个 getInt 方法从配置中获取值并进行赋值，这个赋值动作在 **准备阶段** 是不会执行的，而是在 **初始化阶段** 执行。另外，对于 static 静态代码块，也是在这个阶段执行的。

![](http://image.skjava.com/article/series/jvm/202308102125303398.png)

> 在初始化阶段，如果 JVM 初始化某个类时，发现其父类还没有初始化完成的话，会首先去加载其父类，加载策略就是上一节提到的双亲委派机制。

### 3.5 使用阶段

没啥好说的，就是在程序中使用类或对象。

### 3.6 卸载阶段

卸载阶段，就是当对象不再需要使用时，JVM 需要进行垃圾回收，这一阶段涉及两个核心过程：_存活判定_和_垃圾回收_，我们会在后续章节详细讲解。

四、总结
----

本章，我们介绍了 Java 的类加载机制及其整个流程，JVM 底层的类加载过程的细节非常多，十分复杂，读者如果想要深入，可以参阅 [The Java Virtual Machine Specification](https://docs.oracle.com/javase/specs/jvms/se8/jvms8.pdf)。下一章，我们将看看 JVM 是如何进行内存区域划分的。
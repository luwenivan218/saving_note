> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/4397535146)

> Kafka 具有 & nbsp; 高吞吐低延迟 & nbsp; 的特性。那么 Kafka 是如何实现的呢？这就涉及到消息的持久化机制了。Kafka 会将消息追加到分区日志文件中，并且仅仅是追加数据

Kafka 具有 _**高吞吐低延迟**_ 的特性。那么 Kafka 是如何实现的呢？这就涉及到消息的持久化机制了。Kafka 会将消息追加到分区日志文件中，并且仅仅是追加数据到文件末尾，也就是采用了 **顺序写** 的机制。

但是，光顺序写其实还是不够的，Kafka 同时利用了操作系统的 Page Cache，也就是说消息不是写到磁盘上，而是写到缓存中，正是依靠了 **顺序写 + Page Cache + 零拷贝** 的机制，Kafka 才能有超高的写入性能，单物理机可以做到每秒 10W 级别的消息写入。

![](http://image.skjava.com/article/series/kafka/202307312119264691.png)

> 从写 Page Cache 这个特性也可以看出，虽然 Kafka Broker 自身是一个 JVM 进程，但其实不会占用过多 JVM 内存，而是需要 OS 分配更多的 page cache，以此来缓存更多的消息并异步刷盘。

一、零拷贝
-----

我们来回顾一下使用 Kafka 的整个流程：

1.  Producer 发送一个消息给 Kafka Broker；
2.  Kafka 接受到消息后，将其持久化到磁盘（可能只是写入 Page Cache）；
3.  Consumer 拉取消息，Kafka 读取磁盘上的消息，然后通过网络发送给 Consumer。

整个流程涉及多次磁盘读写，如果 Kafka 真的这么干，就不会有 _**高吞吐低延迟**_ 的特性了。事实上，Kafka 大量使用了 **零拷贝** 技术，使得消息存储和消费的性能极高。

在了解什么是零拷贝之前，我们先来看下传统的 I/O 方式。

### 1.1 普通 I/0 过程

假设我们有下面的几行代码，JVM 程序先从磁盘上读文件，然后通过 Socket 发送给其它 JVM 程序：

```
    
    File file = new File("xxx.txt");
    RandomAccessFile raf = new RandomAccessFile(file, "rw");         
    byte[] arr = new byte[(int) file.length()];        
    raf.read(arr);   
    
    
    Socket socket = new ServerSocket(8080).accept();        
    socket.getOutputStream().write(arr);


```

上述`[读磁盘数据 -> 网络发送]`整个流程一共发生了四次数据拷贝，如下图：

![](http://image.skjava.com/article/series/kafka/202307312119276172.png)

1.  从用户态切换到内核态，将磁盘上的数据通过 **DMA 拷贝** 到内核缓冲区；
2.  从内核态切换到用户态，将内核缓冲区的数据拷贝到用户缓冲区（CPU 拷贝）；
3.  从用户态切换到内核态，将用户缓冲区的数据拷贝到 Socket 缓冲区；
4.  最后，还有一个异步化的过程，将 Socket 缓冲区数据通过 DMA 拷贝到网络引擎，发送出去；
5.  全部完成后，从内核态切换到用户态。

所以说，从本地磁盘读取数据，发生了 2 次数据拷贝，然后通过网络发送出去，又发生了 2 次数据拷贝。期间用户态和内核态之间要发生 4 次切换。所以说，普通的 IO 操作性能是较低的。

### 1.2 零拷贝过程

我们再来看下 Kafka 是如何实现消息的存储和消费的，假设此时消息已经通过异步刷盘，从 os cache 刷到了磁盘上：

![](http://image.skjava.com/article/series/kafka/202307312119294243.png)

我来说明下上图的流程：

1.  首先，Kafka Broker 从用户态切换到内核态，将磁盘上的数据通过 **DMA 拷贝** 到内核缓冲区；
2.  从内核缓冲区拷贝数据的一些 offset 和 length 文件描述符到 Socket 缓冲区；
3.  接着，直接把数据从内核缓冲区拷贝到网络引擎（网卡）里。同时，还从 Socket 缓冲区里拷贝一些 offset 和 length 到网络引擎里去，但是这个 offset 和 length 的量很少，几乎可以忽略。

综上所述，通过 sendfile 方式进行零拷贝时，数据传送只发生在内核态，而且整个过程只进行了 2 次数据拷贝。

> Linux 2.1 版本提供了 sendFile 函数，也就是零拷贝技术，对应于 Java 语言，`FileChannal.transferTo()`方法的底层实现就是 sendfile 方法。
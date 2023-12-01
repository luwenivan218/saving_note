> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/excellentyuxiao/article/details/53390408)

### 1. Netty 简介

Netty 是一个高性能、异步事件驱动的 NIO [框架](https://so.csdn.net/so/search?q=%E6%A1%86%E6%9E%B6&spm=1001.2101.3001.7020)，基于 JAVA NIO 提供的 API 实现。它提供了对 TCP、UDP 和文件传输的支持，作为一个异步 NIO 框架，Netty 的所有 IO 操作都是异步非阻塞的，通过 Future-Listener 机制，用户可以方便的主动获取或者通过通知机制获得 IO 操作结果。 作为当前最流行的 NIO 框架，Netty 在互联网领域、大数据分布式计算领域、游戏行业、通信行业等获得了广泛的应用，一些业界著名的开源组件也基于 Netty 的 NIO 框架构建。

### 2. Netty 线程模型

在 JAVA NIO 方面 Selector 给 Reactor 模式提供了基础，Netty 结合 Selector 和 Reactor 模式设计了高效的线程模型。先来看下 Reactor 模式：

2.1 Reactor 模式

Wikipedia 这么解释 Reactor 模型：“The reactor design pattern is an event handling pattern for handling service requests delivered concurrently by one or more inputs. The service handler then demultiplexes the incoming requests and dispatches them synchronously to associated request handlers.”。首先 Reactor 模式首先是事件驱动的，有一个或者多个并发输入源，有一个 Server Handler 和多个 Request Handlers，这个 Service Handler 会同步的将输入的请求多路复用的分发给相应的 Request Handler。可以如下图所示：

![](https://img-blog.csdn.net/20161129103112729)

从结构上有点类似生产者和消费者模型，即一个或多个生产者将事件放入一个 Queue 中，而一个或者多个消费者主动的从这个队列中 poll 事件来处理；而 Reactor 模式则没有 Queue 来做缓冲，每当一个事件输入到 Service Handler 之后，该 Service Handler 会主动根据不同的 Evnent 类型将其分发给对应的 Request Handler 来处理。

2.2 Reator 模式的实现

关于 Java NIO 构造 Reator 模式，Doug lea 在《Scalable IO in Java》中给了很好的阐述，这里截取 PPT 对 Reator 模式的实现进行说明

1. 第一种实现模型如下：  
![](https://img-blog.csdn.net/20161129103222584)

这是最简单的 Reactor 单线程模型，由于 Reactor 模式使用的是异步非阻塞 IO，所有的 IO 操作都不会被阻塞，理论上一个线程可以独立处理所有的 IO 操作。这时 Reactor 线程是个多面手，负责多路分离套接字，Accept 新连接，并分发请求到处理链中。

对于一些小容量应用场景，可以使用到单线程模型。但对于高负载，大并发的应用却不合适，主要原因如下：

1.  当一个 NIO 线程同时处理成百上千的链路，性能上无法支撑，即使 NIO 线程的 CPU 负荷达到 100%，也无法完全处理消息
2.  当 NIO 线程负载过重后，处理速度会变慢，会导致大量客户端连接超时，超时之后往往会重发，更加重了 NIO 线程的负载。
3.  可靠性低，一个线程意外死循环，会导致整个通信系统不可用

为了解决这些问题，出现了 Reactor 多线程模型。

2.Reactor 多线程模型：  
![](https://img-blog.csdn.net/20161129103519887)

相比上一种模式，该模型在处理链部分采用了多线程（线程池）。

在绝大多数场景下，该模型都能满足性能需求。但是，在一些特殊的应用场景下，如服务器会对客户端的握手消息进行安全认证。这类场景下，单独的一个 Acceptor 线程可能会存在性能不足的问题。为了解决这些问题，产生了第三种 Reactor 线程模型

3.Reactor 主从模型  
![](https://img-blog.csdn.net/20161129103725841)

该模型相比第二种模型，是将 Reactor 分成两部分，mainReactor 负责监听 server socket，accept 新连接；并将建立的 socket 分派给 subReactor。subReactor 负责多路分离已连接的 socket，读写网络数据，对业务处理功能，其扔给 worker 线程池完成。通常，subReactor 个数上可与 CPU 个数等同。

2.3 Netty 模型

2.2 中说完了 Reactor 的三种模型，那么 Netty 是哪一种呢？其实 Netty 的线程模型是 Reactor 模型的变种，那就是去掉线程池的第三种形式的变种，这也是 Netty NIO 的默认模式。Netty 中 Reactor 模式的参与者主要有下面一些组件：

1.  Selector
2.  EventLoopGroup/EventLoop
3.  ChannelPipeline

Selector 即为 NIO 中提供的 SelectableChannel 多路复用器，充当着 demultiplexer 的角色，这里不再赘述；下面对另外两种功能和其在 Netty 之 Reactor 模式中扮演的角色进行介绍。

### 3.EventLoopGroup/EventLoop

当系统在运行过程中，如果频繁的进行线程上下文切换，会带来额外的性能损耗。多线程并发执行某个业务流程，业务开发者还需要时刻对线程安全保持警惕，哪些数据可能会被并发修改，如何保护？这不仅降低了开发效率，也会带来额外的性能损耗。

为了解决上述问题，Netty 采用了串行化设计理念，从消息的读取、编码以及后续 Handler 的执行，始终都由 IO 线程 EventLoop 负责，这就意外着整个流程不会进行线程上下文的切换，数据也不会面临被并发修改的风险。这也解释了为什么 Netty 线程模型去掉了 Reactor 主从模型中线程池。

EventLoopGroup 是一组 EventLoop 的抽象，EventLoopGroup 提供 next 接口，可以总一组 EventLoop 里面按照一定规则获取其中一个 EventLoop 来处理任务，对于 EventLoopGroup 这里需要了解的是在 Netty 中，在 Netty 服务器编程中我们需要 BossEventLoopGroup 和 WorkerEventLoopGroup 两个 EventLoopGroup 来进行工作。通常一个服务端口即一个 ServerSocketChannel 对应一个 Selector 和一个 EventLoop 线程，也就是说 BossEventLoopGroup 的线程数参数为 1。BossEventLoop 负责接收客户端的连接并将 SocketChannel 交给 WorkerEventLoopGroup 来进行 IO 处理。

EventLoop 的实现充当 Reactor 模式中的分发（Dispatcher）的角色。

### 4.ChannelPipeline

ChannelPipeline 其实是担任着 Reactor 模式中的请求处理器这个角色。

ChannelPipeline 的默认实现是 DefaultChannelPipeline，DefaultChannelPipeline 本身维护着一个用户不可见的 tail 和 head 的 ChannelHandler，他们分别位于链表队列的头部和尾部。tail 在更上层的部分，而 head 在靠近网络层的方向。在 Netty 中关于 ChannelHandler 有两个重要的接口，ChannelInBoundHandler 和 ChannelOutBoundHandler。inbound 可以理解为网络数据从外部流向系统内部，而 outbound 可以理解为网络数据从系统内部流向系统外部。用户实现的 ChannelHandler 可以根据需要实现其中一个或多个接口，将其放入 Pipeline 中的链表队列中，ChannelPipeline 会根据不同的 IO 事件类型来找到相应的 Handler 来处理，同时链表队列是责任链模式的一种变种，自上而下或自下而上所有满足事件关联的 Handler 都会对事件进行处理。

ChannelInBoundHandler 对从客户端发往服务器的报文进行处理，一般用来执行半包 / 粘包，解码，读取数据，业务处理等；ChannelOutBoundHandler 对从服务器发往客户端的报文进行处理，一般用来进行编码，发送报文到客户端。

下图是对 ChannelPipeline 执行过程的说明：  
![](https://img-blog.csdn.net/20161129104246082)

关于 Pipeline 的更多知识可参考：[浅谈管道模型 (Pipeline)](http://blog.csdn.net/yanghua_kobe/article/details/7561016%20%E6%B5%85%E8%B0%88%E7%AE%A1%E9%81%93%E6%A8%A1%E5%9E%8B%28Pipeline%29)

### 5.Buffer

Netty 提供的经过扩展的 Buffer 相对 NIO 中的有个许多优势，作为数据存取非常重要的一块，我们来看看 Netty 中的 Buffer 有什么特点。

1.ByteBuf 读写指针

*   在 ByteBuffer 中，读写指针都是 position，而在 ByteBuf 中，读写指针分别为 readerIndex 和 writerIndex，直观看上去 ByteBuffer 仅用了一个指针就实现了两个指针的功能，节省了变量，但是当对于 ByteBuffer 的读写状态切换的时候必须要调用 flip 方法，而当下一次写之前，必须要将 Buffe 中的内容读完，再调用 clear 方法。每次读之前调用 flip，写之前调用 clear，这样无疑给开发带来了繁琐的步骤，而且内容没有读完是不能写的，这样非常不灵活。相比之下我们看看 ByteBuf，读的时候仅仅依赖 readerIndex 指针，写的时候仅仅依赖 writerIndex 指针，不需每次读写之前调用对应的方法，而且没有必须一次读完的限制。

2. 零拷贝

*   Netty 的接收和发送 ByteBuffer 采用 DIRECT BUFFERS，使用堆外直接内存进行 Socket 读写，不需要进行字节缓冲区的二次拷贝。如果使用传统的堆内存（HEAP BUFFERS）进行 Socket 读写，JVM 会将堆内存 Buffer 拷贝一份到直接内存中，然后才写入 Socket 中。相比于堆外直接内存，消息在发送过程中多了一次缓冲区的内存拷贝。
*   Netty 提供了组合 Buffer 对象，可以聚合多个 ByteBuffer 对象，用户可以像操作一个 Buffer 那样方便的对组合 Buffer 进行操作，避免了传统通过内存拷贝的方式将几个小 Buffer 合并成一个大的 Buffer。
*   Netty 的文件传输采用了 transferTo 方法，它可以直接将文件缓冲区的数据发送到目标 Channel，避免了传统通过循环 write 方式导致的内存拷贝问题。

3. 引用计数与池化技术

*   在 Netty 中，每个被申请的 Buffer 对于 Netty 来说都可能是很宝贵的资源，因此为了获得对于内存的申请与回收更多的控制权，Netty 自己根据引用计数法去实现了内存的管理。Netty 对于 Buffer 的使用都是基于直接内存（DirectBuffer）实现的，大大提高 I/O 操作的效率，然而 DirectBuffer 和 HeapBuffer 相比之下除了 I/O 操作效率高之外还有一个天生的缺点，即对于 DirectBuffer 的申请相比 HeapBuffer 效率更低，因此 Netty 结合引用计数实现了 PolledBuffer，即池化的用法，当引用计数等于 0 的时候，Netty 将 Buffer 回收致池中，在下一次申请 Buffer 的没某个时刻会被复用。

### 总结

Netty 其实本质上就是 Reactor 模式的实现，Selector 作为多路复用器，EventLoop 作为转发器，Pipeline 作为事件处理器。但是和一般的 Reactor 不同的是，Netty 使用串行化实现，并在 Pipeline 中使用了责任链模式。

Netty 中的 buffer 相对有 NIO 中的 buffer 又做了一些优化，大大提高了性能。
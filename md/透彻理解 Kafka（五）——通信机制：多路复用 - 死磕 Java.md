> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/4323681022)

> KafkaBroker 基于 Reactor 模式，通过 I/O 多路复用来完成请求的处理，所以具有极高的吞吐量。关于 I/O 多路复用，我在其它的专栏里反复讲解过多次了。本章，我再来针对 K

Kafka Broker 基于 Reactor 模式，通过 I/O 多路复用来完成请求的处理，所以具有极高的吞吐量。关于 I/O 多路复用，我在其它的专栏里反复讲解过多次了。本章，我再来针对 Kafka 讲解下它是如何实现多路复用的。

一、工作流程
------

每个 Kafka Broker 上都有一个 Acceptor 线程和多个 Processor 线程：

1.  Kafka Broker 通过 Acceptor 监听每个新的 Socket 连接，建立连接成功后，会采用 Round Robin 的轮询方式，将 Socket 连接分配给 Processor 线程；
2.  Processor 线程负责处理这个 Socket 连接，每一个 Processor 都有一个 Selector，可以非阻塞的处理多个客户端的读写请求，包括读取数据和将响应返回给对应 Client，但是 Processor 本身不处理具体的业务逻辑；
3.  所有 Processor 都会把请求放入一个 Broker 全局唯一的请求队列，默认大小是 500，可以通过`queued.max.requests`参数设置；
4.  接着，有一个 **KafkaRequestHandler** 线程池负责不停的从队列中获取请求来处理，这个线程池大小默认是 8 个，可通过`num.io.threads`参数控制，处理完请求后的响应，会放入每个 Processor 自己的响应队列 ResponseQueue 里；
5.  最后，Processor 会监听自己的响应队列，把响应拿出来通过 Socket 连接返回给客户端。

整个流程如下图：

![](http://image.skjava.com/article/series/kafka/202307312119375031.png)

> 通过参数`num.network.threads`可以设置 processor 线程的数量，默认值是 3。
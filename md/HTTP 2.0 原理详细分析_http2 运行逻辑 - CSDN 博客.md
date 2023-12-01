> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/zhuyiquan/article/details/69257126)

HTTP 2.0 是在 SPDY（An experimental protocol for a faster web, The Chromium Projects）基础上形成的下一代互联网[通信协议](https://so.csdn.net/so/search?q=%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE&spm=1001.2101.3001.7020)。HTTP/2 的目的是通过支持请求与响应的多路复用来较少延迟，通过压缩 HTTPS 首部字段将协议开销降低，同时增加请求优先级和服务器端推送的支持。  
本文目的是学习 HTTP 2.0 的原理并研究其通信的详细细节。大部分知识点源于《Web 性能权威指南》。

*   [1. 二进制分帧层](#1-二进制分帧层)
    *   [1.1 帧（frame）](#11-帧frame)
    *   [1.2 消息（message）](#12-消息message)
    *   [1.3 流（stream）](#13-流stream)
*   [2. 多路复用共享连接](#2-多路复用共享连接)
*   [3. 请求优先级](#3-请求优先级)
*   [4. 服务端推送](#4-服务端推送)
*   [5. 首部压缩](#5-首部压缩)
*   [6. 一个完整的 HTTP 2.0 通信过程](#6-一个完整的http-20通信过程)
    *   [6.1 基于 ALPN 的协商过程](#61-基于alpn的协商过程)
    *   [6.2 基于 HTTP 的协商过程](#62-基于http的协商过程)
    *   [6.3 完整通信过程](#63-完整通信过程)
*   [7. HTTP 2.0 性能瓶颈](#7-http-20性能瓶颈)
*   [参考文献](#参考文献)

**1. 二进制分帧层**
-------------

**二进制分帧层，是 HTTP 2.0 性能增强的核心。**  
HTTP 1.x 在应用层以纯文本的形式进行通信，而 HTTP 2.0 将所有的传输信息分割为更小的消息和帧，并对它们采用二进制格式编码。这样，客户端和服务端都需要引入新的二进制编码和解码的机制。  
如下图所示，HTTP 2.0 并没有改变 HTTP 1.x 的语义，只是在应用层使用二进制分帧方式传输。  
![](https://img-blog.csdn.net/20170405172818292?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemh1eWlxdWFu/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  
因此，也引入了新的通信单位：

### **1.1 帧（frame）**

HTTP 2.0 通信的最小单位，包括帧首部、流标识符、优先值和帧净荷等。  
![](https://img-blog.csdn.net/20170405153816267?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemh1eWlxdWFu/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

其中，帧类型又可以分为：

*   **DATA**：用于传输 HTTP 消息体；
*   **HEADERS**：用于传输首部字段；
*   **SETTINGS**：用于约定客户端和服务端的配置数据。比如设置初识的双向流量控制窗口大小；
*   **WINDOW_UPDATE**：用于调整个别流或个别连接的流量
*   PRIORITY： 用于指定或重新指定引用资源的优先级。
*   RST_STREAM： 用于通知流的非正常终止。
*   PUSH_ PROMISE： 服务端推送许可。
*   PING： 用于计算往返时间，执行 “活性” 检活。
*   GOAWAY： 用于通知对端停止在当前连接中创建流。

标志位用于不同的帧类型定义特定的消息标志。比如 DATA 帧就可以使用`End Stream: true`表示该条消息通信完毕。流标识位表示帧所属的流 ID。优先值用于 HEADERS 帧，表示请求优先级。R 表示保留位。  
下面是 Wireshark 抓包的一个 DATA 帧：  
![](https://img-blog.csdn.net/20170405170457247?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemh1eWlxdWFu/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### **1.2 消息（message）**

消息是指逻辑上的 HTTP 消息（请求 / 响应）。一系列数据帧组成了一个完整的消息。比如一系列 DATA 帧和一个 HEADERS 帧组成了请求消息。

### **1.3 流（stream）**

流是连接中的一个虚拟信道，可以承载双向消息传输。每个流有唯一整数标识符。为了防止两端流 ID 冲突，客户端发起的流具有奇数 ID，服务器端发起的流具有偶数 ID。  
所有 HTTP 2. 0 通信都在一个 TCP 连接上完成， 这个连接可以承载任意数量的双向数据流 Stream。 相应地， 每个数据流以 消息的形式发送， 而消息由一 或多个帧组成， **这些帧可以乱序发送， 然后根据每个帧首部的流标识符重新组装。**  
![](https://img-blog.csdn.net/20170405172729697?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemh1eWlxdWFu/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  
二进制分帧层保留了 HTTP 的语义不受影响，包括首部、方法等，在应用层来看，和 HTTP 1.x 没有差别。同时，所有同主机的通信能够**在一个 TCP 连接 </**
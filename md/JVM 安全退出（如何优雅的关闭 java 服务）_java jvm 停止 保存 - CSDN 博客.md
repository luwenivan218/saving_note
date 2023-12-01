> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/u011001084/article/details/73480432)

https://tech.imdada.cn/2017/06/18/[jvm](https://so.csdn.net/so/search?q=jvm&spm=1001.2101.3001.7020)-safe-exit/?utm_source=tuicool&utm_medium=referral

背景
--

> 用户：货都到了，购物车里怎么还有刚买的东西，what?  
> 产品：有用户反映，提单完成了，怎么没清购物车，研发赶紧看看是不是有 bug 啊？  
> 研发：恩，我看看，！@#￥%……&*（）一顿狂查，搜嘎，当时在上线，重启应用，异步任务丢了……  
> 产品：能不能行，上线你就丢任务，丢不丢人啊！  
> 研发：…………

上线! 重启! 你还在为丢失任务而烦恼么？看这里看这里，从此不再丢任务，JVM 可以安全退出的

在交易流程中，为了提升服务的性能，我们做了一些异步化的优化，比如更新用户最近使用的收货地址、提单完成后通过 MQ 去发送各种通知类消息、清理用户的购物车等等这些操作，异步化加快了应用的响应速度同时也带来一个隐患，如何保障异步操作的执行？这个场景主要发生在应用重启时，对于通过线程或线程池进行的异步化，JVM 重启时，后台执行的异步操作可能尚未完成。这时，需要通过 JVM 安全关闭来保证异步操作进行完成后，JVM 再执行关闭。  
更广泛的说，在 Linux 上很多应用通常会通过 kill -9 pid 的方式强制将进程杀掉，这种方式简单高效，因此很多应用的停止脚本经常会选择使用 kill -9 pid 的方式。强制进程退出, 会带来一些副作用，对应用程序而言其效果等同于突然掉电，可能会导致如下一些问题：

1.  缓存中的数据尚未持久化到磁盘中，导致数据丢失；
2.  正在进行文件的 write 操作，没有更新完成，突然退出，导致文件损坏；
3.  线程池的任务队列中尚有接收到的任务还没来得及处理，导致任务丢失；
4.  数据库操作已经完成，例如账户余额更新，准备返回应答消息给客户端时，消息尚在通信线程的发送队列中排队等待发送，进程强制退出导致应答消息没有返回给客户端，客户端发起超时重试，会带来重复更新问题；
5.  其它问题等…

这些问题都有可能对我们的业务产生影响，造成不必要的损失，为了避免这些问题，我们需要在 JVM 关闭时做些扫尾的工作，为此 JVM 提供了关闭钩子（shutdown hooks）来做这些事情。本文探讨了利用关闭钩子的相关内容。

[](https://tech.imdada.cn/2017/06/18/jvm-safe-exit/?utm_source=tuicool&utm_medium=referral#JVM__u5173_u95ED "JVM 关闭")JVM 关闭
---------------------------------------------------------------------------------------------------------------------------

首先，我们了解下哪些情况会导致 JVM 关闭，如下图

[![](http://img10.360buyimg.com/n7/s523x355_jfs/t6595/305/70607884/742714/c76b7170/59390c8aNf12b52ac.bmp)](http://img10.360buyimg.com/n7/s523x355_jfs/t6595/305/70607884/742714/c76b7170/59390c8aNf12b52ac.bmp)

对于强制关闭的几种情况，系统关机，操作系统会通知 JVM 进程关闭并等待，一旦等待超时，系统会强制中止 JVM 进程；kill -9、Runtime.halt()、断电、系统 crash 这些种方式会直接无商量中止 JVM 进程，JVM 完全没有执行扫尾工作的机会。因此对用应用程序而言，我们强烈不建议使用 kill -9 这种暴力方式退出。  
而对于正常关闭、异常关闭的几种情况，JVM 关闭前，都会调用已注册的 shutdown hooks，基于这种机制，我们可以将扫尾的工作放在 shutdown hooks 中，进而使我们的应用程序安全的退出。基于平台通用性的考虑，我们更推荐应用程序使用 System.exit(0) 这种方式退出 JVM。

JVM 与 shutdown hooks 交互流程如下图所示，可以对照源码进一步的学习 shutdown hooks 工作原理。  
[![](http://img10.360buyimg.com/n7/s1306x1261_jfs/t6205/257/91070679/64131/65b88829/5939100fNaf1f9cbe.png)](http://img10.360buyimg.com/n7/s1306x1261_jfs/t6205/257/91070679/64131/65b88829/5939100fNaf1f9cbe.png)

[](https://tech.imdada.cn/2017/06/18/jvm-safe-exit/?utm_source=tuicool&utm_medium=referral#Jvm_u5B89_u5168_u9000_u51FA "Jvm安全退出")Jvm 安全退出
-----------------------------------------------------------------------------------------------------------------------------------------

对于 tomcat 类 Web 应用，我们可以直接通过 Runtime.addShutdownHook(Thread hook) 注册自定义钩子，在钩子中实现资源的清理；而对于 worker 类应用，我们可以采用如下的方式安全的退出应用。

### [](https://tech.imdada.cn/2017/06/18/jvm-safe-exit/?utm_source=tuicool&utm_medium=referral#u57FA_u4E8E_u4FE1_u53F7_u7684_u8FDB_u7A0B_u901A_u77E5_u673A_u5236 "基于信号的进程通知机制")基于信号的进程通知机制

信号是在软件层次上对中断机制的一种模拟，在原理上，一个进程收到一个信号与处理器收到一个中断请求可以说是一样的。通俗来讲，信号就是进程间的一种异步通信机制。信号具有平台相关性，Linux 平台支持的一些终止进程信号如下所示：

<table><thead><tr><th>信号名称</th><th>用途</th></tr></thead><tbody><tr><td>SIGKILL</td><td>终止进程，强制杀死进程</td></tr><tr><td>SIGTERM</td><td>终止进程，软件终止信号</td></tr><tr><td>SIGTSTP</td><td>停止进程，终端来的停止信号</td></tr><tr><td>SIGPROF</td><td>终止进程，统计分布图用计时器到时</td></tr><tr><td>SIGUSR1</td><td>终止进程，用户定义信号 1</td></tr><tr><td>SIGUSR2</td><td>终止进程，用户定义信号 2</td></tr><tr><td>SIGINT</td><td>终止进程，中断进程</td></tr><tr><td>SIGQUIT</td><td>建立 CORE 文件终止进程，并且生成 core 文件</td></tr></tbody></table>

Windows 平台存在一些差异，它的一些信号举例如下所示：

<table><thead><tr><th>信号名称</th><th>用途</th></tr></thead><tbody><tr><td>SIGINT</td><td>Ctrl+C 中断</td></tr><tr><td>SIGTERM</td><td>kill 发出的软件终止</td></tr><tr><td>SIGBREAK</td><td>Ctrl+Break 中断</td></tr></tbody></table>

信号选择：为了不干扰正常信号的运作，又能模拟 Java 异步通知，在 Linux 上我们需要先选定一种特殊的信号。通过查看信号列表上的描述，发现 SIGUSR1 和 SIGUSR2 是允许用户自定义的信号, 我们可以选择 SIGUSR2，在 Windows 上我们可以选择 SIGINT。

通过这种信号机制，对应用程序 JVM 发送特定信号, JVM 可以感知并处理该信号，进而可以接受程序退出指令。

### [](https://tech.imdada.cn/2017/06/18/jvm-safe-exit/?utm_source=tuicool&utm_medium=referral#u5B89_u5168_u9000_u51FA_u5B9E_u73B0 "安全退出实现")安全退出实现

首先看下通用的 JVM 安全退出的流程图：

[![](http://img10.360buyimg.com/n7/s866x766_jfs/t6460/82/83995300/36603/9e53dc3c/5939103eN931a1818.png)](http://img10.360buyimg.com/n7/s866x766_jfs/t6460/82/83995300/36603/9e53dc3c/5939103eN931a1818.png)

第一步，应用进程启动的时候，初始化 Signal 实例，它的代码示例如下：

```
Signal sig = new Signal(getOSSignalType());
```

其中 Signal 构造函数的参数为 String 字符串，也就上文介绍的信号量名称。

第二步，根据操作系统的名称来获取对应的信号名称，代码如下：

```
private String getOSSignalType()
   {
       return System.getProperties().getProperty("os.name").
                 toLowerCase().startsWith("win") ? "INT" : "USR2";
    }
```

判断是否是 windows 操作系统，如果是则选择 SIGINT，接收 Ctrl+C 中断的指令；否则选择 USR2 信号，接收 SIGUSR2（等价于 kill -12 pid）指令。

第三步，将实例化之后的 SignalHandler 注册到 JVM 的 Signal，一旦 JVM 进程接收到 kill -12 或者 Ctrl+C 则回调 handle 接口，代码示例如下：

```
Signal.handle(sig, shutdownHandler);
```

其中 shutdownHandler 实现了 SignalHandler 接口的 handle(Signal sgin) 方法，代码示例如下：

```
public class ShutdownHandler implements SignalHandler {
    /**
     * 处理信号
     *
     * @param signal 信号
     */
    public void handle(Signal signal) {
    }
}
```

第四步，在接收到信号回调的 handle 接口中，初始化 JVM 的 ShutdownHook 线程，并将其注册到 Runtime 中，示例代码如下：

```
private void registerShutdownHook()
 {
        Thread t = new Thread(new ShutdownHook(), "ShutdownHook-Thread");
        Runtime.getRuntime().addShutdownHook(t);
 }
```

第五步，接收到进程退出信号后，在回调的 handle 接口中执行虚拟机的退出操作，示例代码如下：

```
Runtime.getRuntime().exit(0);
```

JVM 退出时，底层会自动检测用户是否注册了 ShutdownHook 任务，如果有，则会自动执行注册钩子的 Run 方法，应用只需要在 ShutdownHook 中执行扫尾工作即可，示例代码如下：

```
class ShutdownHook implements Runnable
{
        @Override
        public void run() {
                System.out.println("ShutdownHook execute start...");
                try {
                   TimeUnit.SECONDS.sleep(10);//模拟应用进程退出前的处理操作
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("ShutdownHook execute end...");
                }
}
```

通过以上的几个步骤，我们可以轻松实现 JVM 的安全退出，另外，通常安全退出需要有超时控制机制，例如 30S，如果到达超时时间仍然没有完成退出，则由停机脚本直接调用 kill -9 强制退出。

[](https://tech.imdada.cn/2017/06/18/jvm-safe-exit/?utm_source=tuicool&utm_medium=referral#u4F7F_u7528_u5173_u95ED_u94A9_u5B50_u7684_u6CE8_u610F_u4E8B_u9879 "使用关闭钩子的注意事项")使用关闭钩子的注意事项
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

*   关闭钩子本质上是一个线程（也称为 Hook 线程），对于一个 JVM 中注册的多个关闭钩子它们将会并发执行，所以 JVM 并不保证它们的执行顺序；由于是并发执行的，那么很可能因为代码不当导致出现竞态条件或死锁等问题，为了避免该问题，强烈建议在一个钩子中执行一系列操作。
    
*   Hook 线程会延迟 JVM 的关闭时间，这就要求在编写钩子过程中必须要尽可能的减少 Hook 线程的执行时间，避免 hook 线程中出现耗时的计算、等待用户 I/O 等等操作。
    
*   关闭钩子执行过程中可能被强制打断, 比如在操作系统关机时，操作系统会等待进程停止，等待超时，进程仍未停止，操作系统会强制的杀死该进程，在这类情况下，关闭钩子在执行过程中被强制中止。
*   在关闭钩子中，不能执行注册、移除钩子的操作，JVM 将关闭钩子序列初始化完毕后，不允许再次添加或者移除已经存在的钩子，否则 JVM 抛出 IllegalStateException。
*   不能在钩子调用 System.exit()，否则卡住 JVM 的关闭过程，但是可以调用 Runtime.halt()。
*   Hook 线程中同样会抛出异常，对于未捕捉的异常，线程的默认异常处理器处理该异常，不会影响其他 hook 线程以及 JVM 正常退出。

[](https://tech.imdada.cn/2017/06/18/jvm-safe-exit/?utm_source=tuicool&utm_medium=referral#u603B_u7ED3 "总结")总结
--------------------------------------------------------------------------------------------------------------

为了保障应用重启过程中异步操作的执行，避免强制退出 JVM 可能产生的各种问题，我们可以采用关闭钩子、自定义信号的方式，主动的通知 JVM 退出，并在 JVM 关闭前，执行应用程序的一些扫尾工作，进一步保证应用程序可以安全的退出。
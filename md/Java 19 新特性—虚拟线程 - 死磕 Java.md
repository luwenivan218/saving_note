> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1865448759)

> 很多语言都有类似于 “虚拟线程” 的技术，比如 Go、C#、Erlang、Lua 等，他们称之为“协程”。曾经我们 Java 开发者面对这种平凡而又高级的技术只能干瞪眼，然而现在我们 JavaBoy 也能挺起腰杆子说

很多语言都有类似于 “虚拟线程” 的技术，比如 Go、C#、Erlang、Lua 等，他们称之为“协程”。曾经我们 Java 开发者面对这种平凡而又高级的技术只能干瞪眼，然而现在我们 Java Boy 也能挺起腰杆子说：协程算个の。

![](https://sike.skjava.com/java-features/202312041000001.png)

为什么引入虚拟线程
---------

小明同学作为一个初入职场的小菜鸟，某天接到老板的任务，要他写一个简单的需求：前端传一个 fileId 过来，需要去文件服务器下载下来并解析，然后针对解析的内容做一些 xxx 的事情，需求很简单，小明哥三下五除二就搞定了：

```
    public void doSomething(String fileId) {
        String filePath = downloadFile(Field);
        List<XxxDTO> list = readFile(filePath);
        doSomething(list);
    }


```

随着业务增长，请求量越来越大，小明同学的接口响应速度越来越慢了，而这个时候小小明同学又恰好学习了多线程，知道多线程能够提高系统吞吐量，然后小试牛刀一把：

```
    public void doSomething(String fileId) {
        new Thread(()->{
            String filePath = downloadFile(Field);
            List<XxxDTO> list = readFile(filePath);
            doSomething(list);
        }).start();
    }


```

但是在测试环境压测时，测试同学测着测着发现不对劲了，怎么越搞越慢，而且内存都快爆表了，小明同学只好请教旁边的 Java 高手大明哥了，大明哥一看，惊呆了，大惊：小伙子，不错啊，会用线程了，但不是你这么搞的，去请测试吃顿饭吧，否则年底你要背绩效了。大明哥告诉小明同学：你这样一个请求创建一个线程，成本很高，资源会被耗尽的，你要池化。然后小明同学又花了一个晚上学习了线程池，改造如下：

```
    public void doSomething(String fileId) {
        executor.submit(() -> {
            String filePath = downloadFile(Field);
            List<XxxDTO> list = readFile(filePath);
            doSomething(list);
        });
    }


```

这段代码足够小明同学顶很长时间了。但是架不住公司快速发展，某天老板跟小明同学说，它的那个接口还需要优化下。小明同学懵逼了，这还怎么优化啊？只有硬着头皮去找大明哥帮助了，大明哥跟你说，你这个接口是 I/O 密集型接口，主要耗时在下载文件和读取文件上，所以要针对它们做优化。大明哥告诉你，刚刚好项目组已完成 Java 21 版本的升级（虚拟线程在 Java 19 为预览特性 ，Java 21 转正），里面有一个**虚拟线程是处理 I/O 密集型操作的神器**。

什么是虚拟线程
-------

众所周知，JVM 是一个多线程环境，它通过 `java.lang.Thread` 为我们提供了对操作系统线程的抽象，但是 Java 中的线程都只是对操作系统线程的一种简单封装，我们可以称之为 “平台线程”。

然后，就平台线程而言，在某些场景下，它是有一些缺陷的，具体体现在：

*   **代价昂贵**：创建平台线程的成本很高。每当创建一个平台线程时，操作系统都必须在堆栈中分配大量内存来存储线程的上下文、原生调用堆栈和 Java 调用堆栈。由于堆栈大小是固定的，这就导致了高昂的内存开销。
*   **上下文切换成本高**：在多线程环境下，需要在不同线程间切换，这种上下文切换会消耗时间和资源。
*   **线程数量有限**：Java 线程仅仅只是对操作系统线程的封装，而操作系统线程的数量是有限的，这就限制了 Java 同时运行的线程数量，从而限制了应用程序的并发能力。

然后虚拟线程就没有这个烦恼。

那什么是虚拟线程呢？

> **虚拟线程是 Java 中的一种轻量级线程，它旨在解决传统线程模型中的一些限制，提供了更高效的并发处理能力，允许创建数千甚至数万个虚拟线程，而无需占用大量操作系统资源。**

虚拟线程与传统线程具有如下几个优势：

1.  **更加轻量级**：虚拟线程相比传统线程更加轻量级。因为它们不是直接映射到操作系统线程上，而是在用户空间内被管理。这种设计减少了线程创建和销毁的开销，允许在同一应用中运行成千上万的线程。
2.  **资源消耗更少**：由于不是直接映射到操作系统线程，虚拟线程显著降低了内存和其他资源的消耗。这使得在有限资源下可以创建更多的线程。
3.  **上下文切换开销更低**：由于虚拟线程在用户空间，而不是通过操作系统，所以它的上下文切换开销更低。
4.  **改善阻塞操作的处理**：在传统线程模型中，阻塞操作（如 I/O）会导致整个线程被阻塞，浪费宝贵的系统资源。然而当一个虚拟线程阻塞时，它可以被挂起，底层的操作系统线程则可以用来运行其他虚拟线程。
5.  **简化并发编程**：可以像编写普通顺序代码一样编写并发代码，而不需要过多考虑线程管理和调度。它简化了 Java 的并发编程模型。
6.  **提升性能**：在 I/O 密集型应用中，虚拟线程能够显著地提升性能。而且由于它们的创建和销毁成本低，能够更加高效地利用系统资源。

虽然虚拟线程这么厉害，但是它不做以下几件事：

*   **不替代传统线程**：虚拟线程并不旨在完全替代传统的操作系统线程，而是作为一个补充。对于需要密集计算和精细控制线程行为的场景，传统线程仍然是主流。
*   **非针对最低延迟**：虚拟线程主要针对高并发和高吞吐量，而不是最低延迟。对于需要极低延迟的应用，传统线程可能是更好的选择。
*   **不改变基本的线程模型**：虚拟线程改进了线程的实现方式，但并未改变 Java 基本的线程模型和同步机制。锁和同步仍然是并发控制的重要工具。

下图是虚拟线程的架构图，

![](https://sike.skjava.com/java-features/202312031000001.png)

如何使用虚拟线程
--------

虚拟线程的使用和普通线程几乎一模一样，他们唯一区别在于创建虚拟线程只能通过特定的方法。

> **一、创建虚拟线程并运行**

创建虚拟线程最简单的方式就是使用 `Thread.startVirtualThread(Runnable)`，`startVirtualThread()` 是 Thread 静态方法，它用于创建一个新的虚拟线程，并立即执行给定的 `Runnable` 任务。

```
    @Test
    public void virtualThreadTest() {
        Thread.startVirtualThread(() -> {
            
            System.out.println("这是死磕 Java 新特性的文章—Java 19 新特性：虚拟线程");
        });
    } 


```

> 二、**使用 Thread Builder**

`Thread.Builder`是一个流式 API，用于构建和配置线程。它提供了设置线程属性（如名称、守护状态、优先级、未捕获异常处理器等）的方法。相比直接使用 Thread 来构建线程，`Thread.Builder`提供了更多的灵活性和控制力。

使用`ofVirtual()` 可以创建一个虚拟线程：

```
    @Test
    public void virtualThreadTest() {
        Thread.ofVirtual()
                .name("virtualThreadTest")
                .uncaughtExceptionHandler((t,e)-> System.out.println("线程[" + t.getName() + "发生了异常。message:" + e.getMessage()))
                .start(()->{
                    System.out.println("这是死磕 Java 新特性的文章—Java 19 新特性：虚拟线程");
                });
    }


```

使用 `ofVirtual()` 创建一个虚拟线程，然后调用 `name()` 为虚拟线程设置一个名称，`uncaughtExceptionHandler()` 用来处理线程执行异常。

`Thread.Builder` 还支持设置线程其他属性：

*   `daemon(boolean)`：设置线程是否为守护线程。
*   `priority(int)：`设置线程优先级。

同时，`Thread.Builder` 可以用于生成平台线程，使用 `ofPlatform()` 。

> **三、创建虚拟线程，但不执行**

上面两种方式都是创建后运行，我们也可以创建虚拟线程后，不运行，通过手动调用 `start()` 来运行：

```
    @Test
    public void virtualThreadTest() {
        var vt = Thread.ofVirtual()
                .unstarted(()->{
                    System.out.println("这是死磕 Java 新特性的文章—Java 19 新特性：虚拟线程");
                });
        vt.start();
    }


```

> **四、 使用 ExecutorService**

与平台线程异常，虚拟线程也支持线程池，我们可以使用`Executors.newVirtualThreadPerTaskExecutor()` 创建一个线程池，该线程池会给每个任务分配一个虚拟线程。这意味着每个提交给线程池的任务都会在自己的虚拟线程上异步执行。

```
ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();


```

一旦我们创建了虚拟线程池就可以向其提交任务，任务可以是`Runnable` 或 `Callable`。

```
executor.submit(() -> {
    System.out.println("这是死磕 Java 新特性的文章—Java 19 新特性：虚拟线程");
});
or
Future<String> future = executor.submit(() -> {
    return "这是死磕 Java 新特性的文章—Java 19 新特性：虚拟线程";
});




```

使用完线程池后，我们可以使用`shutdown()` 来关闭线程池，它会等待正在执行的任务完成，但不会接受新的任务。如果需要立即停止所有任务，可以使用 `shutdownNow()`。

```
executor.shutdown();


```

对于虚拟线程而言，虚拟线程池不一定是必须的，如果任务生命周期简单，不需要复杂的管理，并行任务也不是特别多，其实没有必要使用虚拟线程池，直接使用 `Thread.startVirtualThread` 或 `Thread.ofVirtual().start()` 来创建和启动虚拟线程可能还更加简单些。

使用场景 & 注意情况
-----------

*   虚拟线程是一种轻量级的线程，它适用于 I/O 密集型的任务。对于 CPU 密集型任务（如大量计算），平台线程可能是更好的选择，因为虚拟线程在这种情况下可能不会带来性能上面的优势。
*   虽然虚拟线程在资源的消耗上面比平台线程少，但是我们仍然需要合理管理，不要以为资源少就可以随便创建虚拟线程。创建过多的虚拟线程可能会导致内存消耗的增加，尤其是当每个线程都需要维护自己的栈空间时。
*   虚拟线程可能会使得调试和性能监控更加复杂。确保有适当的工具和策略来监控虚拟线程的行为和性能。
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/5892329907)

> Java19，于 2022-09-20 发布 JEP405：Record 模式（预览）Java14 引入预览特性 Record 旨在提供一种简洁的语法来声明类似数据的小型不可变对象，主要是为了解决长期以来在 Java

原

 2023-12-24  阅读 (24)  点赞 (0)

> Java 19， 于 2022-09-20 发布

![](https://sike.skjava.com/java-features/202311292000001.png)

**JEP 405**：Record 模式（预览）
-------------------------

Java 14 引入预览特性 Record 旨在提供一种简洁的语法来声明类似数据的**小型不可变对象**，主要是为了解决长期以来在 Java 中定义纯数据载体类时，代码过于繁琐的问题。在 Java 16 中转为正式特性。

`instanceof` 模式匹配也是在 Java 14 作为预览特性引入的，主要是为了解决 `instanceof` 在做类型匹配时需要进行强制类型转换而导致的代码冗余。

Java 19 引入 Record 模式作为预览特性，它允许在`instanceof`操作中使用记录模式，直接解构和匹配记录中的字段。

比如有一个记录`Record Point(int x, int y)`，可以使用 Record 模式直接检查和提取 `x` 和 `y` 值：

```
if (obj instanceof Point(int x, int y)) {
  System.out.println(x+y);
}


```

可以看出，类型判断、类型转换、Record 值的结构一气呵成，极大地简化了代码结构。

该特性使 Java 模式匹配能力得到进一步扩展。

更多阅读：

*   [Java 14 新特性—模式匹配的 instanceof](https://www.skjava.com/series/article/3436282193)[](https://www.skjava.com/series/article/3436282193)
*   [Java 14 新特性—新增 Record 类型](https://www.skjava.com/series/article/8932587293)
*   [Java 19 新特性—Record 模式](https://www.skjava.com/series/article/9674439600)

**JEP 422**：JDK 移植到 Linux/RISC-V
--------------------------------

`RISC-V` 是一个开源的基于 RISC 的指令集架构。通过`Linux/RISC-V`移植，Java 将获得对硬件指令集的支持。目前该端口支持以下的`HotSpot VM`选项：

*   模板解释器
*   客户端 JIT 编译器
*   服务端 JIT 编译器
*   包括 ZGC 和 Shenandoah 在内的主流垃圾收集器

该特性通过为 `Linux/RISC-V` 提供支持，增强了 Java 在多种硬件平台上的可用性和适应性。

**JEP 424**：外部函数和内存 API（预览）
---------------------------

外部函数和内存 API 是 Java 17 作为预览特性引入的，它的核心在于提供一种安全、高效的方法来访问本地代码（例如 C 或 C++ 库）和内存。主要是通过两个组件实现的：

1.  **Foreign Function Interface (FFI)**: 允许 Java 代码直接调用非 Java 代码，特别是用 C/C++ 编写的代码。这可以通过定义一种类型安全的方式来实现，同时避免了 Java 本地接口（JNI）的复杂性和开销。
2.  **Foreign Memory Access API**：提供了一种安全的方法来访问不受 JVM 管理的内存。这对于需要操作系统级别内存操作或者直接与外部设备交互的应用程序非常重要。

<table><thead><tr><th>Java 版本</th><th>更新类型</th><th>JEP</th><th>更新内容</th></tr></thead><tbody><tr><td>Java 14</td><td>孵化器</td><td>JEP 370</td><td>引入了外部内存访问 API</td></tr><tr><td>Java 15</td><td>第二孵化器</td><td>JEP 383</td><td>优化外部内存访问 API</td></tr><tr><td>Java 16</td><td>孵化器</td><td>JEP 389</td><td>引入了外部链接器 API</td></tr><tr><td>Java 16</td><td>第三孵化器</td><td>JEP 393</td><td>继续优化</td></tr><tr><td>Java 17</td><td>孵化器</td><td>JEP 412</td><td>引入了外部函数和内存 API</td></tr><tr><td>Java 18</td><td>二次孵化器</td><td>JEP 419</td><td>在此版本中，API 再次进行了一些改进和扩展。</td></tr><tr><td>Java 19</td><td>第一次预览</td><td>JEP 424</td><td></td></tr></tbody></table>

JEP 425：虚拟线程（预览）
----------------

虚拟线程是一种轻量级的线程，也就是我们俗称的协程。它的资源分配和调度由`VM`实现，而不是操作系统。虚拟线程的主要特点包括：

1.  **轻量级**：与传统线程相比，它更轻量，创建和销毁的成本较低。
2.  **简化并发编程**：由于不受操作系统线程数量的限制，我们可以为每个独立的任务创建一个虚拟线程，简化并发编程模型。

Java 19 引入虚拟线程的主要目的是为了解决以下问题：

*   **解决并发编程复杂性**：传统的线程模型在处理大量并发任务时复杂且效率低下。虚拟线程简化了并发编程，因为它们更加轻量级，并且易于管理。
*   **资源限制**：操作系统线程是有限的资源，大量线程的创建和管理可能会导致性能下降和资源耗尽。虚拟线程通过减轻这些限制，使得创建和管理大量线程成为可能。

虚拟线程在其他多线程语言中已经被证实是十分有用的，在知乎上有一个关于 Java 19 虚拟线程的讨论，感兴趣的可以去看看：[https://www.zhihu.com/question/536743167](https://www.zhihu.com/question/536743167)。

同时，这篇文章对虚拟线程做了非常详细的介绍：[https://mp.weixin.qq.com/s/yyApBXxpXxVwttr01Hld6Q](https://mp.weixin.qq.com/s/yyApBXxpXxVwttr01Hld6Q)

> 更多阅读：[Java 19 新特性—虚拟线程](https://www.skjava.com/series/article/1865448759)

**JEP 426**：向量 API（第四次孵化）
-------------------------

向量 API 是在 Java 16 中作为孵化器引入的，其目的是提供一个表达式丰富、编译时性能可预测的平台，用于编写复杂的向量计算，以充分利用现代处理器的 SIMD（单指令多数据）指令。

它经历了 Java 16 、Java 17 、Java 18 三次孵化，这是第四次孵化：

<table><thead><tr><th>Java 版本</th><th>更新类型</th><th>JEP</th><th>更新内容</th></tr></thead><tbody><tr><td>Java 16</td><td>第一次孵化</td><td>JEP 338</td><td>提供一个平台无关的方式来表达向量计算，能够充分利用现代处理器上的向量硬件指令。</td></tr><tr><td>Java 17</td><td>第二次孵化</td><td>JEP 414</td><td>对 API 进行了改进，增加了性能优化和新的功能。</td></tr><tr><td>Java 18</td><td>第三次孵化</td><td>JEP 417</td><td>进一步增强 API，改进了性能和易用性。</td></tr><tr><td>Java 19</td><td>第四次孵化</td><td>JEP 426</td><td>进一步增强 API</td></tr></tbody></table>

**JEP 427**：模式匹配的 Switch（第三次预览）
-------------------------------

模式匹配的 Switch 首次是在 Java 17 中作为预览特性引入，其主要目的是为了简化在`case` 标签中进行类型检查和类型转换，减少冗余的代码。

<table><thead><tr><th>Java 版本</th><th>更新类型</th><th>JEP</th><th>更新内容</th></tr></thead><tbody><tr><td>Java 17</td><td>第一次预览</td><td>JEP 406</td><td>引入模式匹配的 Swith 表达式作为预览特性。</td></tr><tr><td>Java 18</td><td>第二次预览</td><td>JEP 420</td><td>对其做了改进和细微调整</td></tr><tr><td>Java 19</td><td>第三次预览</td><td>JEP 427</td><td>进一步优化模式匹配的 Swith 表达式</td></tr></tbody></table>

> 更多阅读：[Java 17 新特性—模式匹配的 Switch 表达式](https://www.skjava.com/series/article/9479813794)

JEP 428：结构化并发（孵化功能）
-------------------

传统的并发编程模型往往让开发者面对复杂的线程管理和错误处理问题，尤其是在多线程和异步编程场景中。Java 19 引入结构化并发，其主要目的是为了改善 Java 并发模型，简化 Java 中的并发编程。

结构化并发是一种新的并发编程范式，旨在使并发操作更容易管理和维护。它通过引入一种称为 “结构化并发” 的概念来实现，该概念强调将并发任务组织在一起，使它们的生命周期更加清晰和可控。

目前该特性还处于孵化阶段。
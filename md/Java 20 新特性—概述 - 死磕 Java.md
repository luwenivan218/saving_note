> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1519146949)

> Java20，于 2023-03-21 日发布。JEP429：作用域值（第一次孵化）在多线程环境下，正确管理上下文数据是一项有挑战的事情，传统的解决方案（如 ThreadLocal）在某些场景下并不是很适用

原

 2023-12-23  阅读 (31)  点赞 (0)

> Java 20，于 2023-03-21 日发布。

![](https://sike.skjava.com/java-features/202311301000001.png)

**JEP 429**：作用域值（第一次孵化）
-----------------------

在多线程环境下，正确管理上下文数据是一项有挑战的事情，传统的解决方案（如 ThreadLocal）在某些场景下并不是很适用或者效率比较低下。Java 20 引入作用域值（Scoped Values），它可以在线程内和线程间共享不可变的数据，并且优于线程局部变量。

Scoped Values 是一种新的机制，它主要用于在线程或虚拟线程中安全地传递和访问数据，它允许在代码的不同部分之间传递信息，例如，在多线程应用中从父线程向子线程传递数据。它的引入解决了两个问题：

1.  **线程局部存储的限制**：ThreadLocal 可能会导致内存泄漏，并且不适合于虚拟线程。Scoped Values 提供了一种更安全和高效的方式来处理线程局部数据。

*   **上下文传递的复杂性**：在复杂的多线程环境中传递上下文信息通常都很复杂。Scoped Values 简化了这一过程。

目前该特性还处于孵化阶段。

**JEP 432**：Record 模式（第二次预览）
----------------------------

Record 模式 是 Java 19 作为预览特性引入的，它允许在 `instanceof` 表达式中使用记录模式，这样我们就可以在单个 `instanceof` 检查中同时测试一个对象的类型并提取其组件。

该特性与 Java 14 中引入的记录类（Record Classes）结合使用，提供了一种更直观、更简洁的方式来处理数据对象。

此次改进：

*   添加对通用记录模式类型参数推断的支持，
*   添加对记录模式的支持以出现在增强语句的标题中`for`
*   删除对命名记录模式的支持。

<table><thead><tr><th>Java 版本</th><th>更新类型</th><th>JEP</th><th>更新内容</th></tr></thead><tbody><tr><td>Java 19</td><td>第一次预览</td><td>JEP 405</td><td>引入 Record 模式作为预览特性</td></tr><tr><td>Java 20</td><td>第二次预览</td><td>JEP 432</td><td>优化 Record 模式</td></tr></tbody></table>

**JEP 433**：模式匹配的 Switch 表达式（第四次预览）
-----------------------------------

模式匹配的 Switch 表达式首次在 Java 17 中作为预览特性引入，它允许在 `switch` 语句和表达式中使用模式匹配，这样就直接在 `case` 标签中匹配对象的类型。

<table><thead><tr><th>Java 版本</th><th>更新类型</th><th>JEP</th><th>更新内容</th></tr></thead><tbody><tr><td>Java 17</td><td>第一次预览</td><td>JEP 406</td><td>引入模式匹配的 Swith 表达式作为预览特性。</td></tr><tr><td>Java 18</td><td>第二次预览</td><td>JEP 420</td><td>对其做了改进和细微调整</td></tr><tr><td>Java 19</td><td>第三次预览</td><td>JEP 427</td><td>进一步优化模式匹配的 Swith 表达式</td></tr><tr><td>Java 20</td><td>第四次预览</td><td>JEP 433</td><td></td></tr></tbody></table>

> 更多阅读：[Java 17 新特性—模式匹配的 Switch 表达式](https://www.skjava.com/series/article/9479813794)

JEP 434：外部函数与内存 API（第二次预览）
--------------------------

外部函数和内存 API 是 Java 17 作为预览特性引入的，它的核心在于提供一种安全、高效的方法来访问本地代码（例如 C 或 C++ 库）和内存。主要是通过两个组件实现的：

1.  **Foreign Function Interface (FFI)**: 允许 Java 代码直接调用非 Java 代码，特别是用 C/C++ 编写的代码。这可以通过定义一种类型安全的方式来实现，同时避免了 Java 本地接口（JNI）的复杂性和开销。
2.  **Foreign Memory Access API**：提供了一种安全的方法来访问不受 JVM 管理的内存。这对于需要操作系统级别内存操作或者直接与外部设备交互的应用程序非常重要。

JDK 20 中是第二次预览，这次的改进包括：

*   `MemorySegment` 和 `MemoryAddress` 抽象的统一
*   增强的 `MemoryLayout` 层次结构
*   `MemorySession`拆分为`Arena`和`SegmentScope`，以促进跨维护边界的段共享。

<table><thead><tr><th>Java 版本</th><th>更新类型</th><th>JEP</th><th>更新内容</th></tr></thead><tbody><tr><td>Java 14</td><td>孵化器</td><td>JEP 370</td><td>引入了外部内存访问 API</td></tr><tr><td>Java 15</td><td>第二孵化器</td><td>JEP 383</td><td>优化外部内存访问 API</td></tr><tr><td>Java 16</td><td>孵化器</td><td>JEP 389</td><td>引入了外部链接器 API</td></tr><tr><td>Java 16</td><td>第三孵化器</td><td>JEP 393</td><td>继续优化</td></tr><tr><td>Java 17</td><td>孵化器</td><td>JEP 412</td><td>引入了外部函数和内存 API</td></tr><tr><td>Java 18</td><td>二次孵化器</td><td>JEP 419</td><td>在此版本中，API 再次进行了一些改进和扩展。</td></tr><tr><td>Java 19</td><td>第一次预览</td><td>JEP 424</td><td></td></tr><tr><td>Java 20</td><td>第二次预览</td><td>JEP 434</td><td>对 API 进行了一些改进</td></tr></tbody></table>

**JEP 436**：虚拟线程（第二次预览）
-----------------------

虚拟线程是 Java 19 作为预览特性首次引入，其主要目的是解决 Java 并发编程模型的复杂性，使其更加简单和高效。它解决了两个问题：

*   **解决并发编程复杂性**：传统的线程模型在处理大量并发任务时复杂且效率低下。虚拟线程简化了并发编程，因为它们更加轻量级，并且易于管理。
*   **资源限制**：操作系统线程是有限的资源，大量线程的创建和管理可能会导致性能下降和资源耗尽。虚拟线程通过减轻这些限制，使得创建和管理大量线程成为可能。

<table><thead><tr><th>Java 版本</th><th>更新类型</th><th>JEP</th><th>更新内容</th></tr></thead><tbody><tr><td>Java 19</td><td>第一次预览</td><td>JEP 425</td><td>引入虚拟线程作为预览特性</td></tr><tr><td>Java 20</td><td>第二次预览</td><td>JEP 437</td><td>优化虚拟线程</td></tr></tbody></table>

> 更多阅读：[Java 19 新特性—虚拟线程](https://www.skjava.com/series/article/1865448759)

**JEP 437**：结构化并发（第二次孵化）
------------------------

结构化并发是在 Java 19 中孵化的，它的主要目的是使并发操作更容易管理和维护。它通过引入一种称为 “结构化并发” 的概念来实现，该概念强调将并发任务组织在一起，使它们的生命周期更加清晰和可控。

<table><thead><tr><th>Java 版本</th><th>更新类型</th><th>JEP</th><th>更新内容</th></tr></thead><tbody><tr><td>Java 19</td><td>第一次孵化</td><td>JEP 428</td><td>引入结构化并发作为孵化特性</td></tr><tr><td>Java 20</td><td>第二次孵化</td><td>JEP 437</td><td>优化增强结构化并发</td></tr></tbody></table>

**JEP 438**：向量 API（第五次孵化）
-------------------------

向量 API 是在 Java 16 中作为孵化器引入的，其目的是提供一个表达式丰富、编译时性能可预测的平台，用于编写复杂的向量计算，以充分利用现代处理器的 SIMD（单指令多数据）指令。它目前是第五次孵化。

<table><thead><tr><th>Java 版本</th><th>更新类型</th><th>JEP</th><th>更新内容</th></tr></thead><tbody><tr><td>Java 16</td><td>第一次孵化</td><td>JEP 338</td><td>引入向量 API 作为孵化器</td></tr><tr><td>Java 17</td><td>第二次孵化</td><td>JEP 414</td><td>对 API 进行了改进，增加了性能优化和新的功能。</td></tr><tr><td>Java 18</td><td>第三次孵化</td><td>JEP 417</td><td>进一步增强 API，改进了性能和易用性。</td></tr><tr><td>Java 19</td><td>第四次孵化</td><td>JEP 426</td><td>进一步增强 API</td></tr><tr><td>Java 20</td><td>第五次孵化</td><td>JEP 438</td><td>进一步增强 API</td></tr></tbody></table>
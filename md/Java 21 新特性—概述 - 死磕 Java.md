> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1967910684)

> Java21，发布于 2023-09-19。JEP430：字符串模板（预览）字符串的拼接和格式化是我们日常比较常见的一种操作，我们一般有如下几种解决方法：使用 + 来拼接使用 StringBuffer 和 Spr

> Java 21 ，发布于 2023-09-19 。

![](https://sike.skjava.com/java-features/202311302000001.png)

JEP 430：字符串模板 （预览）
------------------

字符串的拼接和格式化是我们日常比较常见的一种操作，我们一般有如下几种解决方法：

1.  使用 `+` 来拼接
2.  使用 `StringBuffer`和`SpringBuilder` 来拼接字符串
3.  使用`String::format` 和 `String::formatted` 来格式化字符串
4.  使用`java.text.MessageFormat` 格式化字符串

这几种传统的方式都比较麻烦，过程有很繁琐，比如：`int x = 20;int y = 3;`，如何输出 `20 + 3 = 21`？：

```
String s = x + " + " + y + " = " + (x + y);
或者
new StringBuilder().append(x).append(" + ").append(y).append(" = ").append(x + y).toString();



```

为了简化字符串的构造和格式化，Java 21 引入字符串模板功能，该特性主要目的是为了提高在处理包含多个变量和复杂格式化要求的字符串时的可读性和编写效率。

它允许**在字符串中直接嵌入表达式**来字符串的构造和格式化。比如上面可以简写为：

```
StringTemplate.STR."\{x} + \{y} = \{x+y}";


```

目前该特性为预览特性，请谨慎使用。

> 更多阅读：[Java 21 新特性—字符串模板](https://www.skjava.com/series/article/1880587197)

JEP 431：有序集合
------------

Java 中的集合框架一直以来缺少一种原生的、高效的有序集合实现。传统的集合类如 `TreeSet` 或 `LinkedHashSet` 虽然在某种程度上提供了有序性，但它们的设计并不专注于维护严格的元素顺序。

Java 21 引入有序集合，提供一种新的集合类型，旨在解决访问 Java 中各种集合类型的第一个和最后一个元素需要非统一且麻烦处理场景。该集合具有如下特点：

1.  **严格的顺序保持**：集合将确保元素的顺序，无论是基于元素插入的顺序还是基于某种排序规则。
2.  **性能优化**：集合将针对快速访问、插入和删除操作进行优化，特别是在大量元素的情况下。
3.  **易于使用和理解**：与现有的集合类型一致，有序集合也将提供一个直观和易于使用的 API 接口。

> 更多阅读：[Java 21 新特性—有序集合](https://www.skjava.com/series/article/1895017259)

JEP 439：分代 ZGC
--------------

ZGC 自推出以来（Java 11 引入，Java 15 成为正式特性），已经被证明是一个高效的、可扩展的、低延迟的垃圾收集器。然而，它在处理大量的小对象和短生命周期的对象时效率不是最优的。Java 21 引入分代 ZGC ，通过采用分代收集策略，提高垃圾收集的效率和应用程序的性能。

其主要内容：

1.  **分代收集策略**：在分代 ZGC 中，堆内存被分为几个不同的区域（代）：年轻代和老年代。不同的对象根据它们的年龄和大小被分配到不同的代中。
2.  **优化小对象处理**：轻代专门用于存放新创建的小对象。由于大多数小对象很快就会变得不可达，所以可以在年轻代中更高效地进行垃圾收集。
3.  **减少垃圾收集延迟**：通过频繁但快速地收集年轻代中的对象，可以减少整个堆垃圾收集的延迟，从而提高应用程序的响应性。

分代策略使得 ZGC 能更高效地处理那些生命周期短的对象，有助于减少应用程序遇到的长时间停顿，从而降低延迟。

JEP 440：Record 模式
-----------------

Java 14 引入预览特性 Record 旨在提供一种简洁的语法来声明类似数据的**小型不可变对象**，主要是为了解决长期以来在 Java 中定义纯数据载体类时，代码过于繁琐的问题。在 Java 16 中转为正式特性。

Java 19 引入 Record 模式作为预览特性，它允许在`instanceof`操作中使用 Record，并直接解构和匹配记录中的字段。在 Java 21 中成为正式特性。

<table><thead><tr><th>Java 版本</th><th>更新类型</th><th>JEP</th><th>更新内容</th></tr></thead><tbody><tr><td>Java 19</td><td>第一次预览</td><td>JEP 405</td><td>引入 Record 模式作为预览特性</td></tr><tr><td>Java 20</td><td>第二次预览</td><td>JEP 432</td><td>优化 Record 模式</td></tr><tr><td>Java 21</td><td>正式特性</td><td>JEP 440</td><td>成为正式特性</td></tr></tbody></table>

> 更多阅读：[Java 19 新特性—Record 模式](https://www.skjava.com/series/article/9674439600)

JEP 441：switch 模式匹配
-------------------

模式匹配的 Switch 表达式首次在 Java 17 中作为预览特性引入，它允许在 `switch` 语句和表达式中使用模式匹配，这样就可以直接在 `case` 标签中匹配对象的类型。

<table><thead><tr><th>Java 版本</th><th>更新类型</th><th>JEP</th><th>更新内容</th></tr></thead><tbody><tr><td>Java 17</td><td>第一次预览</td><td>JEP 406</td><td>引入模式匹配的 Swith 表达式作为预览特性。</td></tr><tr><td>Java 18</td><td>第二次预览</td><td>JEP 420</td><td>对其做了改进和细微调整</td></tr><tr><td>Java 19</td><td>第三次预览</td><td>JEP 427</td><td>进一步优化模式匹配的 Swith 表达式</td></tr><tr><td>Java 20</td><td>第四次预览</td><td>JEP 433</td><td></td></tr><tr><td>Java 21</td><td>正式特性</td><td>JEP 441</td><td>成为正式特性</td></tr></tbody></table>

> 更多阅读：[Java 17 新特性—模式匹配的 Switch 表达式](https://www.skjava.com/series/article/9479813794)

JEP 442：外部函数和内存 API （第三次预览）
---------------------------

外部函数和内存 API 是 Java 17 作为预览特性引入的，它的核心在于提供一种安全、高效的方法来访问本地代码（例如 C 或 C++ 库）和内存。主要是通过两个组件实现的：

1.  **Foreign Function Interface (FFI)**: 允许 Java 代码直接调用非 Java 代码，特别是用 C/C++ 编写的代码。这可以通过定义一种类型安全的方式来实现，同时避免了 Java 本地接口（JNI）的复杂性和开销。
2.  **Foreign Memory Access API**：提供了一种安全的方法来访问不受 JVM 管理的内存。这对于需要操作系统级别内存操作或者直接与外部设备交互的应用程序非常重要。

Java 21 为第三次预览，此次对其进行了性能、通用性、安全性、易用性上的优化。

<table><thead><tr><th>Java 版本</th><th>更新类型</th><th>JEP</th><th>更新内容</th></tr></thead><tbody><tr><td>Java 14</td><td>孵化器</td><td>JEP 370</td><td>引入了外部内存访问 API</td></tr><tr><td>Java 15</td><td>第二孵化器</td><td>JEP 383</td><td>优化外部内存访问 API</td></tr><tr><td>Java 16</td><td>孵化器</td><td>JEP 389</td><td>引入了外部链接器 API</td></tr><tr><td>Java 16</td><td>第三孵化器</td><td>JEP 393</td><td>继续优化</td></tr><tr><td>Java 17</td><td>孵化器</td><td>JEP 412</td><td>引入了外部函数和内存 API</td></tr><tr><td>Java 18</td><td>二次孵化器</td><td>JEP 419</td><td>在此版本中，API 再次进行了一些改进和扩展。</td></tr><tr><td>Java 19</td><td>第一次预览</td><td>JEP 424</td><td></td></tr><tr><td>Java 20</td><td>第二次预览</td><td>JEP 434</td><td>对 API 进行了一些改进</td></tr><tr><td>Java 21</td><td>第三次预览</td><td>JEP 442</td><td></td></tr></tbody></table>

JEP 443：未命名模式和变量 （预览）
---------------------

在 Java 21 之前，任何时候我们都需要明确指出变量的名称。这种方式虽然清晰明了，但在某些情况下可能显得繁琐，特别是在变量实际上并不需要一个明确的名称时。

Java 21 引入特性：未命名模式的变量，该特性通过允许使用未命名变量来简化代码。

主要内容就是，可以选择不为某些变量命名，而是使用一种特殊的标记（例如 `_`），表示这是一个未命名的变量。简而言之就是如果我们在代码中声明了一个变量，但又不打算使用它。这个时候，就可以将其替换为下划线字符`_`。它可以应用于各种场景，例如`try-catch`块、`for`循环等。

> 更多阅读：[Java 21 新特性—未命名模式和变量](https://www.skjava.com/series/article/1961861176)

JEP 444：虚拟线程（正式特性）
------------------

虚拟线程是在 Java 19 中作为预览特性引入的，其主要目的是为了简化和改善 Java 的并发模型，主要解决以下两个问题：

*   **解决并发编程复杂性**：传统的线程模型在处理大量并发任务时复杂且效率低下。虚拟线程简化了并发编程，因为它们更加轻量级，并且易于管理。
*   **资源限制**：操作系统线程是有限的资源，大量线程的创建和管理可能会导致性能下降和资源耗尽。虚拟线程通过减轻这些限制，使得创建和管理大量线程成为可能。

它功能非常强大，终于在 Java 21 中成为正式特性：

<table><thead><tr><th>Java 版本</th><th>更新类型</th><th>JEP</th><th>更新内容</th></tr></thead><tbody><tr><td>Java 19</td><td>第一次预览</td><td>JEP 425</td><td>引入虚拟线程作为预览特性</td></tr><tr><td>Java 20</td><td>第二次预览</td><td>JEP 437</td><td>优化虚拟线程</td></tr><tr><td>Java 21</td><td>正式特性</td><td>JEP 444</td><td>成为正式特性</td></tr></tbody></table>

更多阅读：[Java 19 新特性—虚拟线程](https://www.skjava.com/series/article/1865448759)

JEP 445：未命名类和 main 方法 （预览）
--------------------------

在 Java 21 之前，我们创建一个可以运行的类，需要经历创建包，新建类，写 main 方法。这个步骤显得非常繁琐，这对于快速原型制作、教育目的或小型工具开发而言，这些往往都是不必要的。同时，这种也不利于新手学习。Java 21 引入该特性来简化开发流程，降低入门门槛。

> 更多阅读：[Java 21 新特性—未命名类和 main 方法](https://www.skjava.com/series/article/6780334511)

JEP 446：作用域值 （预览）
-----------------

作用域值是在 Java 20 中引入的，它主要用于在线程或虚拟线程中安全地传递和访问数据，它允许在代码的不同部分之间传递信息，例如，在多线程应用中从父线程向子线程传递数据。

它的引入解决了两个问题：

1.  **线程局部存储的限制**：ThreadLocal 可能会导致内存泄漏，并且不适合于虚拟线程。Scoped Values 提供了一种更安全和高效的方式来处理线程局部数据。

*   **上下文传递的复杂性**：在复杂的多线程环境中传递上下文信息通常都很复杂。Scoped Values 简化了这一过程。

<table><thead><tr><th>Java 版本</th><th>更新类型</th><th>JEP</th><th>更新内容</th></tr></thead><tbody><tr><td>Java 20</td><td>孵化器</td><td>JEP 429</td><td>引入作用域作为孵化器</td></tr><tr><td>Java 21</td><td>预览特性</td><td>JEP 446</td><td></td></tr></tbody></table>

> **TIPS**：作用域值是一个很重要的特性，目前还处于预览节点，等转正时大明哥再做详细介绍

JEP 448：向量 API（第六次孵化）
---------------------

向量 API 是在 Java 16 中作为孵化器引入的，其目的是提供一个表达式丰富、编译时性能可预测的平台，用于编写复杂的向量计算，以充分利用现代处理器的 SIMD（单指令多数据）指令。它目前是第六次孵化。

<table><thead><tr><th>Java 版本</th><th>更新类型</th><th>JEP</th><th>更新内容</th></tr></thead><tbody><tr><td>Java 16</td><td>第一次孵化</td><td>JEP 338</td><td>引入向量 API 作为孵化器</td></tr><tr><td>Java 17</td><td>第二次孵化</td><td>JEP 414</td><td>对 API 进行了改进，增加了性能优化和新的功能。</td></tr><tr><td>Java 18</td><td>第三次孵化</td><td>JEP 417</td><td>进一步增强 API，改进了性能和易用性。</td></tr><tr><td>Java 19</td><td>第四次孵化</td><td>JEP 426</td><td>进一步增强 API</td></tr><tr><td>Java 20</td><td>第五次孵化</td><td>JEP 438</td><td>进一步增强 API</td></tr><tr><td>Java 21</td><td>第六次孵化</td><td>JEP 448</td><td>进一步增强 API</td></tr></tbody></table>

JEP 449：弃用 Windows 32 位 x86 端口
------------------------------

因为 Windows 10 是最后一个支持 32 位操作的 Windows 操作系统，将于 2025 年 10 月终止生命周期。且虚拟线程在 x86-32 上无法实现。所以 Java 21 弃用 Windows 32 位 x86 端口 ，意味着在 Windows 平台上，32 位版本的 Java 虚拟机（JVM）将被弃用。

JEP 451：准备禁止动态加载代理
------------------

在某些情况下动态加载代理可能导致安全问题，比如未经授权的代码执行和权限提升。而且还存在一定的性能影响。为了增强 Java 平台的安全性和稳定性，特别是在处理代理类时，Java 21 准备禁止动态加载代理，未来当代理程序动态加载到正在运行的 JVM 中时，将会发出警告。

JEP 452：密钥封装机制 API 安全库
----------------------

密钥封装是保护密钥不被非法访问的重要手段，在传输和存储密钥时尤其重要。该特性提供了一种标准化的方法来封装（加密）和解封（解密）密钥。

该 API 是一种针对密钥封装机制（KEMs）的 API，是一种使用公钥密码学来保护对称密钥的加密技术。

JEP 453：结构化并发（预览）
-----------------

结构化并发是在 Java 19 中孵化的，它的主要目的是使并发操作更容易管理和维护。它通过引入一种称为 “结构化并发” 的概念来实现，该概念强调将并发任务组织在一起，使它们的生命周期更加清晰和可控。

<table><thead><tr><th>Java 版本</th><th>更新类型</th><th>JEP</th><th>更新内容</th></tr></thead><tbody><tr><td>Java 19</td><td>第一次孵化</td><td>JEP 428</td><td>引入结构化并发作为孵化特性</td></tr><tr><td>Java 20</td><td>第二次孵化</td><td>JEP 437</td><td>优化增强结构化并发</td></tr><tr><td>Java 21</td><td>预览</td><td>JEP 453</td><td></td></tr></tbody></table>
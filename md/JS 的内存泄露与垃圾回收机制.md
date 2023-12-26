> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7315302731684642854)

1、内存管理
======

程序的运行需要占用内存，JavaScript 是在创建变量（对象，字符串等）时自动进行了内存分配，并且在不使用它们时 “自动” 释放。释放的过程称为垃圾回收（Garbage Collection，简称 GC）。这个 “自动” 是混乱的根源，并让开发者错误的感觉他们可以不关心内存管理，这个 “自动” 并不足够智能，那些很占空间的值一旦不再用到，必须检查是否还存在对它们的引用。如果是的话，就需要手动解除引用。

2、内存分配
======

在 js 中，将数据分为`原始数据类型`和`引用数据类型`，分别存储在栈和堆中，那么，栈内存和堆内存有什么区别呢？

堆内存（Heap Memory）
----------------

*   **动态分配**：堆内存是动态分配的，通常用于存储应用程序中的对象和其他复杂数据结构。分配和释放的顺序不固定。
*   **垃圾回收**：垃圾回收器会定期扫描堆内存，查找那些不再被任何变量或对象引用的内存块，然后释放这些内存以供再次使用。
*   **全局访问**：堆内存中的对象可以在应用程序的任何地方被访问。

栈内存（Stack Memory）
-----------------

*   **静态分配**：栈内存是静态或自动分配的，主要用于存储局部变量和函数调用的上下文。分配和释放的顺序是严格定义的，遵循后进先出（LIFO）的原则。
*   **无需垃圾回收**：当函数执行结束，其作用域中的局部变量会自动被清除。因此，栈内存不需要垃圾回收器来管理。
*   **作用域限制**：栈内存中的变量只能在其声明的函数内部访问。

由于栈内存的分配和回收是自动和确定的，因此`垃圾回收器主要关注堆内存的管理`。在栈上，当函数调用返回时，它的栈帧（包括所有局部变量）就会被移除，这一过程是由 `CPU 的调用堆栈机制自动管理`的，不需要垃圾回收器参与。

然而，需要注意的是，虽然栈内存的局部变量在函数返回后会被清除，但如果这些局部变量持有对堆内存中对象的引用，那么这些对象的生命周期可能会超过函数调用本身。在这种情况下，只有当没有任何引用指向堆内存中的对象时，垃圾回收器才会回收该对象占用的内存。

3、什么样的内存称为垃圾？
=============

程序不再使用的内存（如：不再被引用的对象）或一些不可达的对象（不能从根上访问）

4、如何识别内存泄漏？
===========

`内存泄漏`通常指的是程序中已分配的内存由于某些原因没有被释放或无法被释放，长时间运行后会导致应用程序使用的内存不断增加，最终可能耗尽系统资源，导致程序性能下降甚至崩溃。在 JavaScript 环境中，内存泄漏通常发生在堆内存中，因为这是大部分动态分配内存的地方，比如对象、数组和函数闭包等。

[经验法则](https://link.juejin.cn?target=https%3A%2F%2Fwww.toptal.com%2Fnodejs%2Fdebugging-memory-leaks-node-js-applications "https://www.toptal.com/nodejs/debugging-memory-leaks-node-js-applications")是：如果连续五次垃圾回收之后，内存占用一次比一次大，就有内存泄漏。这就要求实时查看内存占用。

1）浏览器

Chrome 浏览器查看内存占用，如果内存占用基本平稳，接近水平，就说明不存在内存泄漏

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/110c82fc61b64d769ea21ab54a8afc97~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=600&h=307&s=61671&e=png&b=414141)

反之则是内存泄漏

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b2bca17bded244b19682429079ca402c~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=599&h=307&s=53248&e=png&b=414141)

2）命令行：使用 Node 提供的 [`process.memoryUsage`](https://link.juejin.cn?target=https%3A%2F%2Fnodejs.org%2Fapi%2Fprocess.html%23process_process_memoryusage "https://nodejs.org/api/process.html#process_process_memoryusage")方法

该方法会返回一个对象：

*   rss（resident set size）：所有内存占用，包括指令区和堆栈。
    
*   heapTotal："堆" 占用的内存，包括用到的和没用到的。
    
*   heapUsed：V8 引擎为 JavaScript 分配的堆内存中已经使用的部分
    
*   external： V8 引擎内部的 C++ 对象占用的内存。
    
    判断内存泄漏我们一般是以`heapUsed`为准，需要注意的是，heapUsed 是评估 JavaScript 内存泄漏的一个重要指标，但它不是唯一的指标。对于内存泄漏的全面诊断，可能还需要结合其他工具和方法，比如剖析（profiling）工具、内存快照（memory snapshots）和堆分析（heap analysis）等。
    

5、垃圾回收的触发时机？
============

垃圾回收可以周期性地运行，也可能在达到内存分配阈值时触发。因为垃圾回收可能会暂停程序执行（称为 “停顿”），现代的垃圾回收器会尽量优化这些停顿，使其对程序的影响最小。

6、垃圾回收
======

常见的方法是以下两种：

1)、标记 - 清除 (Mark-and-Sweep)
---------------------------

这是最常见的垃圾回收算法。它的基本思路分为两个阶段：

`标记阶段`：垃圾回收器会从根（通常是全局对象）开始，遍历所有从根开始的引用，递归标记所有可达的（reachable）对象 (在应用中仍然可以访问到的对象)。

`清除阶段`：垃圾回收器接着会查找所有未被标记的对象，并释放它们的内存。由于这些对象不可达，它们被认为是 “垃圾”。

`缺陷-内存碎片`：被释放的内存位置是不变的，它不会重新排序内存空间，而我们在插入引用类型的数据时它们需要是**连续的内存空间**，故对内存空间的利用率并不高

`标记整理（Mark-Compact）算法`（旨在解决标记清除算法带来的内存碎片问题），在整理阶段，它会将所有活跃的对象向一端移动，从而清理掉未标记对象所占用的空间，并消除内存中的碎片。在整理过程中，活跃对象的引用地址可能会发生变化，所以需要更新原来指向这些对象的引用，确保它们指向新的地址。由此我们会发现，虽然移动的操作提高了内存分配的效率，但移动对象并更新引用这一过程可能会导致**较大的性能开销**。由于整理阶段要暂停程序执行（Stop-The-World），因此可能会影响到程序的响应时间和吞吐量，尤其是在堆内存比较大的情况下。

现代垃圾回收器通常使用多种算法组合来管理内存，例如，将堆内存分成几个区域（如新生代、老年代），并针对不同的区域使用不同的垃圾回收策略，以达到高效和低延迟的垃圾回收效果。标记整理算法通常用于老年代垃圾回收，这是因为老年代中对象的存活率较高，移动存活对象并整理内存可以有效地减少碎片，为新对象的分配提供更大的连续空间。

2)、引用计数 (Reference Counting)
----------------------------

这是另一种垃圾回收算法，其思路相对简单：语言引擎有一张 "引用表"，保存了内存里面所有的资源（通常是各种值）的引用次数。如果一个值的引用次数是`0`，就表示这个值不再用到了，因此可以将这块内存释放

但存在这样的情况：比如一个值不被需要，引用数却不为`0`，垃圾回收机制无法释放这块内存，从而导致内存泄漏

```
const arr = [1, 2, 3, 4];
console.log('hello world');


```

以上，数组`[1, 2, 3, 4]`是一个值，会占用内存。变量`arr`是仅有的对这个值的引用，因此引用次数为`1`。尽管后面的代码没有用到`arr`，但它仍然会持续占用内存。

故我们需要手动释放，将`arr`重置为`null`，解除`arr`对`[1, 2, 3, 4]`的引用，引用次数变成了`0`，这块内存就可以被垃圾回收机制释放了。

```
let arr = [1, 2, 3, 4];
console.log('hello world');
arr = null; // 手动解除


```

因此，并不是说有了垃圾回收机制，程序员就无需关注内存占用了～

`致命的问题: 循环引用`。如果两个或两个以上的对象相互引用，但它们都不再被其他活跃对象或根引用，那么它们的引用次数永远不会达到 0，即使它们实际上是不可访问的。

```
function createExample() { 
    var objectA = {}; 
    var objectB = {};
    // 创建循环引用
    objectA.a = objectB; 
    objectB.a = objectA;
} 
createExample();


```

函数执行完毕，并且外部都不存在对 objectA 和 objectB 的引用了，它们的引用计数仍然为 1，因为它们相互引用。故在纯粹的引用计数垃圾回收机制中，这将导致 objectA 和 objectB 永远不会被回收。所以现代浏览器不会使用纯粹的引用计数法

7、WeakMap
=========

如果能有一种方法，在新建引用的时候就声明，哪些引用必须手动清除，哪些引用可以忽略不计，当其他引用消失以后，垃圾回收机制就可以释放内存。那么就能减少由于程序员疏忽而导致的内存泄漏了。ES6 考虑到这一点，推出了两种新的数据结构：[WeakSet](https://link.juejin.cn?target=http%3A%2F%2Fes6.ruanyifeng.com%2F%23docs%2Fset-map%23WeakSet "http://es6.ruanyifeng.com/#docs/set-map#WeakSet") 和 [WeakMap](https://link.juejin.cn?target=http%3A%2F%2Fes6.ruanyifeng.com%2F%23docs%2Fset-map%23WeakMap "http://es6.ruanyifeng.com/#docs/set-map#WeakMap")。

`WeakMap` 是一种**键值对**的集合，其中的键必须是对象或[非全局注册的符号](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FSymbol%23%25E5%2585%25A8%25E5%25B1%2580%25E5%2585%25B1%25E4%25BA%25AB%25E7%259A%2584_symbol "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol#%E5%85%A8%E5%B1%80%E5%85%B1%E4%BA%AB%E7%9A%84_symbol")，且值可以是任意的 [JavaScript 类型](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FData_structures "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Data_structures")，并且不会创建对它的键的强引用。换句话说，一个对象作为 `WeakMap` 的键存在，不会阻止该对象被垃圾回收。一旦一个对象作为键被回收，那么在 `WeakMap` 中相应的值便成为了进行垃圾回收的候选对象，只要它们没有其他的引用存在。

`WeakMap` 允许将数据与对象相关联，而不阻止键对象被垃圾回收，即使值引用了键

参考文献：

[MDN - 内存管理](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FMemory_management "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Memory_management")

[阮一峰 - JavaScript 内存泄漏教程](https://link.juejin.cn?target=https%3A%2F%2Fwww.ruanyifeng.com%2Fblog%2F2017%2F04%2Fmemory-leak.html%3Futm_source%3Dtuicool%26utm_medium%3Dreferral%2F*%26%255E%2525%24 "https://www.ruanyifeng.com/blog/2017/04/memory-leak.html?utm_source=tuicool&utm_medium=referral/*&%5E%25$")

[MDN-WeakMap](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FWeakMap "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/WeakMap")
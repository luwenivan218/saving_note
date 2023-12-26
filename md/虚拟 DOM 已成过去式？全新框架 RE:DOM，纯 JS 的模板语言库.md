> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7316145140073775167)

> 文章首发自公众号：程序员 Sunday

Hello，大家好，我是 Sunday。

如果你在近几年使用过 JavaScript 框架，你可能会听到 “虚拟 DOM 很快” 的说法，这通常意味着虚拟 DOM 比真实 DOM 更快。然而，这其实是一个关于虚拟 DOM 强大之处的误解。有些人可能会问：“为什么 Svelte 不使用虚拟 DOM，却仍然能运行得如此之快？”

现在看来，我们有必要深入探讨一下这个问题了。

什么是 “虚拟 DOM”
------------

虚拟 DOM（Virtual DOM）是一个概念，用于优化 Web 应用程序的性能。它是一个虚拟表示真实 DOM 树的内存中的对象树。通过对比前后两个虚拟 DOM 树的差异，可以最小化实际 DOM 操作的次数，从而提高渲染效率。

假设你有一个页面，其中有一个列表需要更新。使用虚拟 DOM，Vue 和 React 会创建一个虚拟的树结构来表示这个列表。当数据发生变化时，Vue 和 React 不会直接操作真实的 DOM。相反，它们会生成一个新的虚拟 DOM 树，并与之前的虚拟 DOM 树进行比较。

例如，假设你有一个待办事项列表，用 Vue 编写的代码可能是这样的：

```
<div id="app">
  <ul>
    <li v-for="todo in todos" :key="todo.id">{{ todo.text }}</li>
  </ul>
</div>


```

当 `todos` 数组中的待办事项发生变化时，Vue 会生成一个新的虚拟 DOM 树，然后与之前的虚拟 DOM 树进行比较。它会找出变化的部分，并只更新那些需要改变的地方，而不是重新渲染整个列表。

React 中也有类似的机制。比如，使用 JSX 编写的 React 代码可能如下所示：

```
function TodoList({ todos }) {
  return (
    <div>
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </div>
  );
}


```

当 `todos` 数组发生变化时，React 也会生成新的虚拟 DOM 树，并通过其 diff 算法找出变化，然后只更新必要的部分，而不是重新渲染整个列表。

这种机制使得 Vue 和 React 能够在数据变化时更高效地更新页面。

为什么 Svelte 不使用虚拟 DOM？
---------------------

Svelte 不使用虚拟 DOM 的主要原因是它采用了一种不同的编译思路，即 “编译时” 而非 “运行时” 的方法来处理组件更新和 DOM 操作。

虚拟 DOM 的概念在 React 和 Vue 等框架中被广泛应用，它通过比较前后两个虚拟 DOM 树的差异，并将变化应用到实际的 DOM 上，从而提高了页面更新的性能。然而，这种机制也带来了一些额外的开销和复杂性。

相比之下，Svelte 采用了一种 “编译时” 方法。在编译阶段，Svelte 会分析组件中的代码，并生成针对性的原生 JavaScript 代码，这些代码负责在运行时直接操作 DOM。换句话说，Svelte 在组件构建阶段就知道如何更新 DOM，而不需要在运行时执行虚拟 DOM 的 diff 操作。

这种静态的编译方式让 Svelte 具有更高的性能，因为它避免了在运行时进行虚拟 DOM 操作所带来的开销。Svelte 的这种设计思想允许它生成更为精准和高效的代码，让开发者可以专注于编写简洁、易读的组件代码，同时保持出色的性能。

虚拟 DOM 是唯一快速的方式吗？
-----------------

显然不是的。

虚拟 DOM 的速度通常被认为相对较快，因为它可以最小化实际 DOM 操作的次数，通过比较前后两个虚拟 DOM 树的差异，只更新必要的部分。这种优化确实提高了页面渲染的效率，特别是在大型应用中或者有复杂 UI 的情况下。

然而，虚拟 DOM 的速度快慢还取决于多个因素：

1.  **应用规模：** 对于小型应用，虚拟 DOM 的优势可能不太显著，因为在简单场景下直接操作 DOM 可能更高效。
    
2.  **数据变更频率：** 如果应用中数据变更频繁，虚拟 DOM 可以帮助减少不必要的 DOM 操作，提高性能。但若数据变更不频繁，虚拟 DOM 的优势可能不那么明显。
    
3.  **算法和优化：** 虚拟 DOM 的 diff 算法影响着更新速度。不同的框架可能采用不同的 diff 算法，一些框架可能在这方面进行了更多的优化，提高了速度。
    
4.  **浏览器兼容性：** 虽然大多数现代浏览器都能很好地处理虚拟 DOM，但在某些旧版本浏览器中，虚拟 DOM 可能不如预期快速。
    

所以说，虚拟 DOM 的速度只能说是相对较快，但并非适用于所有场景。

一些新兴的框架（如 Svelte）采用了不同的方法，通过编译时生成高效的代码来实现更快的页面渲染，显示出与虚拟 DOM 相媲美甚至更快的性能。因此，虽然虚拟 DOM 提供了性能优势，但并不是唯一且适用于所有情况的解决方案。

什么是 RE:DOM？
-----------

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2f055fc4fa7a4c239e7df84ae63eff2c~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2738&h=1246&s=168678&e=png&a=1&b=ffffff)

当讨论前端库时，性能和易用性是至关重要的。

`RE:DOM` 是一个不使用虚拟 DOM 的库，它比许多基于虚拟 DOM 的库，如 React，速度更快，同时占用更少内存。它还支持轻松创建可重用组件，这是它的另一个优点。

另一个 RE:DOM 的显著优势是，**开发者只需使用纯 JavaScript，无需学习和使用复杂的模板语言。**

当前，RE:DOM 是一个 MIT 许可的开源项目，拥有超过 3.3k 的 star 和 1.0k 的项目依赖量，是一个备受关注的前端开源项目。

接下来，让我们来看看如何使用 RE:DOM。

最简单的方式是从 Hello World 示例开始。对于下面的 HTML 标记，使用 RE:DOM 极为简单：

```
<!DOCTYPE html>
<html lang="en">
    <body>
        <script src="https://redom.js.org/redom.min.js"></script>
        <script>
            import { el, mount } from "redom";
            const hello = el("h1", "Hello world!");
            mount(document.body, hello);
        </script>
    </body>
</html>


```

在 ES6 环境中，可以直接使用 import 语法：

```
import { el, mount } from 'https://redom.js.org/redom.es.min.js';


```

接下来，让我们了解一些 RE:DOM 的常见方法。

### Element

使用 `el()` 创建元素并使用 `mount()` 挂载，几乎就像使用纯 JavaScript 一样：

```
import { el, mount } from 'redom';

const hello = el('h1', 'Hello RE:DOM!');
mount(document.body, hello);


```

生成的 HTML 示例为：

```
<body>
  <h1>Hello RE:DOM!</h1>
</body>


```

### Text reference

使用 `text()` 帮助器生成文本节点的引用：

```
import { text, mount } from 'redom';
const hello = text('hello');
mount(document.body, hello);
hello.textContent = 'hi!';


```

### ID 和类名

可以使用 `#` 和 `.` 作为定义元素 ID 和类名称的快捷方式：

```
el('');
el('#hello');
el('.hello');
el('span.hello');


```

生成的 HTML 示例为：

```
<div></div>
<div id="hello"></div>
<div class="hello"></div>
<span class="hello"></span>


```

### 样式处理

可以使用字符串或对象定义样式：

```
el("div", { style: "color: red;" });
el("div", { style: { color: "red" } });


```

生成的 HTML 格式如下：

```
<div style="color: red;"></div>
<div style="color: red;"></div>


```

RE:DOM 提供了简洁而强大的方法来处理 DOM，让开发变得更加便捷和高效。

> **前端训练营：1v1 私教，终身辅导计划，帮你拿到满意的 `offer`。** 已帮助数百位同学拿到了中大厂 `offer`。欢迎来撩~~~~~~~~
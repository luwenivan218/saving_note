> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7314608415531089972)

Vue3 的到来开启了一系列增强和突破性功能的新时代。从 Vue2 到 Vue3 的这一进化步骤为开发者提供了一系列机会，能够让我们站在 Web 开发趋势的前沿，充分发挥这个卓越框架的潜力。今天我将与大家分享，Vue3 带来的宏伟进步。在迁移过程中如何处理已弃用和被淘汰的部分, 以及如何丝滑的从 Vue2 向 Vue3 过渡。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/adda512000884c76904de066a6f83260~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=736&h=414&s=8394&e=webp&b=4a077d)

Vue3 的新特性都有哪些呢？
===============

**1. Composition API（组合式 API）**

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/70141f842b174f0b8f0111a34004f781~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=460&h=300&s=5264&e=webp&b=343643)

组合式 API 提供了一种灵活的方式来组织组件逻辑，更易于通过更好的类型检查在组件之间重用代码。与选项式 API 相比，组合式 API 更易于维护。

**2. 更好的类型支持**

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/47698d76c5374f67a18d47fc9ee391a7~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=736&h=385&s=3998&e=webp&b=082f62) Vue3 的代码基于 TypeScript 完全编写，具有自动生成的类型定义。它具有更好的类型推断和对 TypeScript 的可选链和空值合并运算符的支持。

**3. 改进的虚拟 DOM 算法，多根节点**

Vue3 中的虚拟 DOM 已经完全重新设计。它利用了一个新的差异算法，并通过基于编译器的优化加速渲染过程。 在 Vue3 中，我们可以为单个组件创建多个根节点，而在 Vue2 中这是不可行的。

**4. 传送门（Teleport）**

`<Teleport>` 是一个内置组件，允许我们将组件模板的一部分 “传送” 到存在于该组件的 DOM 层次结构之外的 DOM 节点中。`<Teleport>` 的 `to` 属性期望一个 CSS 选择器字符串或一个实际的 DOM 节点。当`<Teleport>` 组件被挂载时，传送到目标的节点必须已经存在于 DOM 中。如下面代码所示：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cc17c0de13e84b9c862ddda1c065def7~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1242&h=512&s=34496&e=webp&b=151718) 这段代码使用了 Vue3 的组件 `<demo-component>` 包含了一个 `<teleport>` 组件，而 `<teleport>` 又包裹了一个名为 `<pop-up>` 的子组件。这段代码的主要目的是将 `<pop-up>` 组件的内容传送（teleport）到页面中具有 `id="teleport-area"` 的 DOM 节点内。

Vue 3 已弃用和移除的内容
===============

**Filters:**

在 Vue 3 中，已移除 Filters。相反，我们可以使用方法或计算属性。您可以参考下面的示例：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e60632a464c240cb98b4028ada567aed~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1242&h=825&s=51382&e=webp&b=161819)

vue2 中的 filters 函数

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4be1687690d049cebb4be214c7bcfa84~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1242&h=825&s=52520&e=webp&b=161819)

Vue3 使用计算属性进行了替换

**v-model**

在 Vue 3 中，v-model 的 prop 和事件默认名称已更改：

*   PROP: value ⟶ modelValue；
*   EVENT: input ⟶ update:modelValue；
*   破坏性的变更: v-bind 的. sync 修饰符和组件模型选项已被移除，用 v-model 上的参数替代
*   新增: 可以在同一组件上进行多个 v-model 绑定 ；创建自定义 v-model 修饰符的能力

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d9e374013a4d4a0093f12b4267e3859c~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1242&h=429&s=37286&e=webp&b=151718)

Vue2 中的 v-model

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/35b055877aa3428db88a8149a3fc5f30~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1242&h=825&s=94164&e=webp&b=161819)

Vue3 中 V-model

**作用域插槽**

在 Vue 3 中，作用域插槽具有不同的语法。要识别插槽名称，请使用 v-slot 指令或其 #简写。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fb2dfe2f6d4c4617be9e426f3d232393~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1242&h=489&s=39246&e=webp&b=151718)

Vue2 slot 的使用

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6bc65f609df14d338e6519164d6f0a8a~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1242&h=489&s=35948&e=webp&b=151718)

Vue3 slot 的使用

**Key 属性**

Vue3 在 v-if/v-else/v-else-if 分支上不再需要 key，因为 Vue 现在会自动生成唯一的 key。 `在<template v-for>上的key应该放在<template>标签上（而不是其子元素上）。`

**v-if 与 v-for 的优先级**

*   在 Vue 2 中: 在同一个元素上同时使用 v-if 和 v-for 时，v-for 会优先于 v-if。
*   在 Vue 3 中: v-if 将始终优先于 v-for。

以上是今天的分享内容，由于篇幅太长的原因，下一篇将继续分享 如何丝滑的从 Vue2 向 Vue3 过渡。如果你感兴趣，欢迎评论 点赞和收藏！
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7314642642868224034)

Vue3 的到来开启了一系列增强和突破性功能的新时代。从 Vue2 到 Vue3 的这一进化步骤为开发者提供了一系列机会，能够让我们站在 Web 开发趋势的前沿，充分发挥这个卓越框架的潜力。上篇与大家分享了，Vue3 引入的新特性。以及向 Vue3 迁移过程中如何处理已弃用和被淘汰的部分。今天用一个简单的 TodoList 项目实例分享如何从 Vue2 向 Vue3 过渡。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1f6861bf377f43e58b97de9774ca8181~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=735&h=490&s=29700&e=webp&b=e5e0df)

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f4bf3dfbaf7543878056eeb86930a857~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1242&h=1104&s=39722&e=webp&b=fefefe)

安装 Vue 迁移工具
===========

**安装迁移工具前准备工作：**

*   将任何已弃用的命名 / 作用域插槽语法更新至最新版本。
*   如果使用自定义的 webpack，请将 vue-loader 更新到最新版本。
*   对于使用 vue-cli ，升级 @vue/cli-service 到最新版本。

**安装迁移构建：**

*   将 vue 升级到 ^3.1.0；
*   安装 @vue/compat（Vue 迁移构建，版本要与 vue 版本相同）；
*   运行时报错先尝试一下 npm cache clean --force 清除 npm 缓存再进行安装；
*   将 package.json 中的 vue-template-compiler 替换为 @vue/compiler-sfc@^3.1.0；

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1cbb857cd4384703a6ae34444ed83f38~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1242&h=581&s=41730&e=webp&b=151718)

*   在 *.config.js 中配置 @vue/compat;

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/413b0264b95a42fb861f56e8f4e3fe9e~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1242&h=885&s=54730&e=webp&b=151718)

vue-cli 中配置

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8efd83f3c4724eb290bf9f56d42716d4~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1242&h=977&s=45342&e=webp&b=151718)

webpack 中配置

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/008bb50e8d184132940a78ec1c4e3533~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1242&h=856&s=38280&e=webp&b=161819)

Vite 中配置

解决迁移构建过程的报错
===========

存在构建迁移时与 Vue3 不兼容的问题, npm 运行代码时出现报错, 本实例项目中 Vue2 使用了 filters 函数而 Vue3 废弃了 filter 函数：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c6dfe959d4cb4dea9b7a0694fba5d0af~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1242&h=569&s=49658&e=webp&b=202020) 以及在控制台遇到下面这样的 warning 提示：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e2ec49b1d664636ba422a3b0f1e4780~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1242&h=500&s=66042&e=webp&a=1&b=fcf5f1) 需要修改一些配置文件，解决 Vue3 和 Vue2 的区别：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/35f4abbb0686460da420c4667900eeb7~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1242&h=644&s=50774&e=webp&a=1&b=202020)

main.js 在 vue2 和 vue3 的配置区别

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75955418ea9b4d58919a81a99ab46da7~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1242&h=632&s=39204&e=webp&b=202020)

store.js 在 vue2 和 vue3 的配置区别

还需要修改 Vue3 组合式 API 和 Vue2 之间的区别：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e3771a3348ce40eb86a95f3337a8444b~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1242&h=560&s=39926&e=webp&b=20201f)

测试你的程序运行并删除迁移构建
===============

首先测试已迁移的组件，确保它们在 Vue 3 环境中的功能正常。可以利用 Vue Devtools 扩展来在迁移过程中调试和检查你的应用程序。一旦测试完成，确保你的应用在 Vue 3 中运行正常，就可以卸载 @vue/compat 并删除在. config.js 中的配置。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0dac4c69939e4df2967791e06ce04b29~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1242&h=428&s=30622&e=webp&b=201f1f)

删除 @vue/compat 配置

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9156617726da46be98dcc4e2e12005ee~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1242&h=388&s=34278&e=webp&b=272727)

删除在. config.js 中的配置

今天以一个简单的实例分享 Vue2 向 Vue3 过渡，当然复杂的项目过渡中可能会有更多需要一一解决的问题和 BUG！
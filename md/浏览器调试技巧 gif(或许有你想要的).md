> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7315990049891450932)

简介
==

本文主要介绍了一些谷歌浏览器的使用小技巧，来方便大家平时的开发和调试，有需要的朋友可以参考下。

**本文共分为以下几部分：**

*   [调试压缩文件](#id1 "#id1")
*   [观察某个对象实时变化](#id2 "#id2")
*   [快速复制请求](#id3 "#id3")
*   [寻找含有指定数据的请求](#id4 "#id4")
*   [触发断点的其他方式](#id5 "#id5")
*   [寻找忘记释放的定时器](#id6 "#id6")
*   [DOM 断点](#id7 "#id7")
*   [请求断点](#id8 "#id8")
*   [查找未使用的代码](#id9 "#id9")
*   [定位页面卡顿原因](#id10 "#id10")
*   [快速拿到指定 dom](https://link.juejin.cn?target=id11 "id11")
*   [定位请求所走的方法栈路径](https://link.juejin.cn?target=id12 "id12")
*   [分析请求网络](https://link.juejin.cn?target=id13 "id13")
*   [调试悬浮样式](https://link.juejin.cn?target=id14 "id14")
*   [多窗口调试同域站点](https://link.juejin.cn?target=id15 "id15")
*   [只看某一域下面请求](https://link.juejin.cn?target=id16 "id16")

主体内容
====

[调试压缩文件](https://link.juejin.cn?target=)
----------------------------------------

想要调试线上环境的时候，代码经常是压缩成一行，无法进行断点和修改，利用下面的操作便可以简单进行压缩文件调试。

```
source => 左上角Overrides => 开启Enable LocalOverrides(本地代理) => 找到调试源文件 => 右键Save for overrides => Pretty print(格式化代码) => 刷新页面


```

这里以掘金为例，测试跟踪掘金的登录流程。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c87ee559fab04dcab1976d4e91ecf004~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1735&h=874&s=2677510&e=gif&f=432&b=fcfbfb)

[观察某个对象实时变化](https://link.juejin.cn?target=)
--------------------------------------------

有时候想观察某个元素的变化规律，特别是鼠标拖动时，可以用 Create live expression

```
Console => Create live expression（灰色眼睛）


```

例如实时观察 body 元素的宽度

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6795bbd4f19542a28ca0a1b52cc5ea8d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1735&h=874&s=3153014&e=gif&f=246&b=fcfcfc)

**[快速复制请求](https://link.juejin.cn?target=)**
--------------------------------------------

有时候想用 postman 调试接口参数，复制 cookie 步骤太繁琐？可以试试将请求直接复制成 Curl 命令

```
NetWork => Name => 右键某条请求copy => copy as cURL(bash)


```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7f1598d1c87742d9bacd679e3124e8bb~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1735&h=874&s=1264432&e=gif&f=300&b=fcfbfb)

**[寻找含有指定数据的请求](https://link.juejin.cn?target=)**
-------------------------------------------------

想要找某个字段是哪个接口返回的，可以用 network 隐藏的搜索栏

```
Network => 进入面板按下 Ctrl + F => 左上角搜索栏


```

例如搜索 article_id

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3c54bfbbc25343a6a2841e502ad0380f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1735&h=874&s=363315&e=gif&f=118&b=fbfbfb)

**[触发断点的其他方式](https://link.juejin.cn?target=)**
-----------------------------------------------

可以试试定时器写 debug

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/58a6cb34407c45078488afec6b58e3f3~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1735&h=874&s=262078&e=gif&f=117&b=fdfdfd)

**[找到忘记释放的定时器](https://link.juejin.cn?target=)**
------------------------------------------------

忘记释放的定时器会对性能造成影响，想要快速找到，可以试试定时器 Breakpoints

```
Sources => Event Listener Breakpoints => Timer => setInterval fired


```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/41c71e722c4f47fba96780bec4d3eacf~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1735&h=874&s=383314&e=gif&f=149&b=fcfcfc)

**[DOM 断点](https://link.juejin.cn?target=)**
--------------------------------------------

找到元素是在哪条 js 语句被修改

```
Elements => 右键元素 => Break on => subtree modification(子节点被修改时触发) || attribute modification(该节点属性被修改时触发) || node removal(该节点被移除时触发)


```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f25af5bb89dc4c79914b1d52092b86e2~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1735&h=874&s=1141079&e=gif&f=116&b=080808)

**[请求断点](https://link.juejin.cn?target=)**
------------------------------------------

想要对某条特点请求发送时进行断点，可以用 XHR/fetch Breakpoints

```
Sources => XHR/fetch Breakpoints => Add breakpoint(加号)


```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1a1e985dc18d4307937ea30fe2020c0f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1777&h=874&s=627546&e=gif&f=160&b=fbfafa)

**[查找未使用的 js 和 css 代码](https://link.juejin.cn?target=)**
--------------------------------------------------------

Chrome DevTools 中的 Coverage 标签可帮助您查找未使用的 JavaScript 和 CSS 代码，它会实时更新 js css 覆盖率，并帮你定位到未使用的代码

```
ctrl + shift + p => 输入 show coverage => 点击黑色圈圈（instrucment coverage）开始记录


```

例如：有如下代码，每次点击 button 按钮便使用一个 css 样式和一个 js 方法，观察 coverage 记录的 js 和 css 使用情况变化 ![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e2e0e941fd11447cb50fa4c17951cc10~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=803&h=522&s=144936&e=png&b=1f1f1f)

注意下面 git 图的中间区域（我们的代码区，coverage 会帮我们标注未使用和已使用的样式）和下边区域（统计区，统计已使用的总字节数）

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/70d604ae90c24fab952a88c77cc3aa22~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1777&h=874&s=188876&e=gif&f=75&b=fbfbfb)

**[定位页面卡顿原因](https://link.juejin.cn?target=)**
----------------------------------------------

通过 perfomance 选项卡的录制功能记录下卡顿时期浏览器运行状况，然后通过分析 Main(主线程) 的带红色三角形的 Task，找到耗时的 js 和 dom 操作。

```
perfomance => Record(黑色圈圈)


```

从下图可以看到，有两块带三角形的 Task, 一个是我们的点击事件（在里面生成 3 万个 dom 节点），另一个是浏览器事件（将 3 万个 dom 节点渲染到页面上）

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a9ba0b3af314c098d0f03dab399dba5~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1777&h=874&s=655329&e=gif&f=197&b=fdfdfd)

**[快速拿到指定 dom 元素](https://link.juejin.cn?target=)**
---------------------------------------------------

console 面板下，可以通过`$0,$1,$2,$3,$4`拿到最近点击过的五个元素，也可以通过 $(‘选择器’) 拿到元素, $$('选择器') 拿到所有符合条件元素

[![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b2d3dec214cc473d89ef253b19d8bdc0~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=500&h=246&s=33697&e=gif&b=faf9f9)](https://link.juejin.cn?target=https%3A%2F%2Fwww.imagehub.cc%2Fimage%2F1JHzyA "https://www.imagehub.cc/image/1JHzyA")

**[定位请求所走方法栈路径](https://link.juejin.cn?target=)**
-------------------------------------------------

```
NetWork => initiator => 鼠标悬浮指定请求就可查看请求方法栈路径


```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/07f0291400344de18ae38ad54d2b9363~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=870&h=806&s=111520&e=png&b=fbfbfb)

**[分析网络情况](https://link.juejin.cn?target=)**
--------------------------------------------

对某个接口速度进行分析，到底是同时请求过多，还是服务器响应太慢，又或者内容太大

```
 NetWork => waterfall => 点击具体请求


```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d8c9bcca696f4d6d9abba5e04d3f41a6~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2447&h=1283&s=543791&e=png&b=fdfdfd)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/342b198a20d3425d8cd66c478125f5d6~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1359&h=1300&s=288023&e=png&b=fdfdfd)

**[调试悬浮样式](https://link.juejin.cn?target=)**
--------------------------------------------

```
ctrl+shift+p => 禁用JavaScript


```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8e056b1cb3754605b9885b3fd0f2430f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2442&h=1247&s=535995&e=gif&f=39&b=fcfcfc)

**[多窗口调试同域站点](https://link.juejin.cn?target=)**
-----------------------------------------------

应用场景：同域模式下，A 页面在 cookie 种下临时信息（非幂等信息无法二次操作），然后点击按钮打开 B 页面，B 页面利用完临时信息后立马删除，然后进行逻辑操作。需要查看 B 的打开时的逻辑，此时往往来不及打断点或者打开 F12 进入断点。 那么可以尝试下面步骤。利用同域页面共享渲染进程来进行 debug

```
在窗口一打开B页面，并打开F12并加入断点 => 在A种完cookie后进行跳转B之前进行断点阻止下一步执行 => 刷新B页面，放开A页面断点


```

**[只看某一域下面请求](https://link.juejin.cn?target=)**
-----------------------------------------------

```
network => filter => domain:你的域名


```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8b499d49893e4496bf510a5356f1a15c~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2442&h=1296&s=643225&e=gif&f=40&b=fdfdfd)

完
=

更多内容，可参考谷歌开发者文档，基本涵盖了 devTools 的所有操作 [developers.google.cn/web/tools/c…](https://link.juejin.cn?target=https%3A%2F%2Fdevelopers.google.cn%2Fweb%2Ftools%2Fchrome-devtools%2F "https://developers.google.cn/web/tools/chrome-devtools/")
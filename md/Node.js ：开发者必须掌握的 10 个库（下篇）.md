> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7316121883636006931)

Node.js 通过 JavaScript 库的方式提供了代码重用的能力，但是选择合适的库可能会很困难。有用的库可以缩短开发时间，并为您的 Web 应用程序提供多种优势，例如更快的加载时间和较小的应用程序包大小。在选择库时，需要考虑应用程序的复杂性、支持库的社区、更新频率以及文档质量等因素。Node.js 库通过 Node.js 软件包管理器 npm 进行维护，它可以帮助安装各种开源库。紧接上篇今天将继续分享 5 个重要的 Node.js 库，它们使 Web 开发变得更简单。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cf02388d3e7a47e0a8401a3a530c0fd1~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=736&h=481&s=13004&e=jpg&b=000000)

Lodash
======

Lodash 是一个 JavaScript 实用工具包，帮助开发者编写简单且易于维护的代码。它包括了 200 多个实用函数，用于处理常见的编程任务，比如类型检查、简单的数学操作等。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ac947014619b4c15be69fd9db70c99db~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2656&h=1868&s=480032&e=png&b=151718) **Lodash 库的特点与优势**：

*   使用 Polyfills 实现跨浏览器兼容性。
*   在处理对象数组时，提供了内建的解决方案，如 filter、search 和 flatMap。
*   帮助开发者避免冗余代码，保持代码的清晰性。

Axios
=====

Axios 是一个基于 Promise 的 Node.js 和浏览器 HTTP 客户端。它还会根据需要管理浏览器或 Node.js 请求和响应数据的转换。Axios 是同构的，这意味着它可以在服务器和客户端上使用相同的代码库。Axios 在服务器端使用原生的 HTTP 模块，在客户端使用 XMLHttpRequest 进行 HTTP 通信。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b17ca1341bb24e5f98d04478750d93d2~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1192&h=1120&s=206430&e=png&b=161819) **Axios 库的特点与优势**：

*   提供了常见的 HTTP 数据类型（如 GET、PUT、POST 和 DELETE）的 API 方法。
*   在通过互联网执行 HTTP 查询时，通过防范跨站请求伪造（CSRF）来提高安全性。
*   自动进行 JSON 数据翻译，轻松将响应数据转换为 JSON 格式。

Puppeteer
=========

Puppeteer 是一个 Node.js 框架，通过 DevTools 协议提供了一个高级 API，允许你通过控制 Chrome/Chromium 来自动化浏览器。它可以用于自动化前端测试，比如处理请求的测试、识别和比较 UI 组件以及性能测试等。可以通过将 Puppeteer 包导入到他们的代码中来构建一个 Chromium 实例，然后该实例可以与浏览器引擎通信以进行自动化测试。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a81f58a607bc405e984b141b9584d327~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1226&h=1380&s=254747&e=png&b=151718) **Puppeteer 库的优势与特点：**

*   无需设置，配置简单，不需要任何额外的驱动程序。
*   网站被爬取以生成预渲染内容。
*   与诸如 Jest 和 Mocha 等知名测试框架兼容。

Multer
======

Multer 是一个基于 HTML 表单解析器 Busboy 构建的 Node.js 中间件库，支持多部分和多表单数据。在初始化 Multer 实例之后，其中的一个参数是一个测试对象，该对象指定了上传的文件在服务器上的保存位置。Multer 将上传请求传递给一个文件对象，Multer API 解析该对象并将其传输到目标站点。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f996d029e2a34c71bbf07cd7637d5dcd~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1328&h=1902&s=350216&e=png&b=151718) **Multer 库的特点与优势：**

*   通过内置的解析，使原始 HTTP 请求数据更容易用于存储。
*   允许定义文件的编码类型，为上传的文件提供额外的保护程度。
*   可以过滤并限制文件类型和大小的上传选择。

Dotenv
======

Dotenv 是一个 Node.js 实用模块，用于管理应用程序的环境变量并保护关键的配置数据。Dotenv 还帮助应用程序按照十二因素应用程序方法保存环境变量。当 dotenv 库在应用程序早期配置时，来自`.env`文件的环境变量会立即注入到 process.env 中。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8d34d074c604446f8d615ef9b090a1f9~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1244&h=934&s=176835&e=png&b=161819) **Dotenv 库的特点与优势**：

*   允许将秘密信息（如 API 密钥和登录凭证）与源代码分离，并允许每个开发者建立自己的`.env`文件。
*   由于它是一个零依赖的模块，不会增加程序的大小。

在 Node.js 中有许多有用的库，但选择适合你的项目的理想库可能会有些困难。根据自己不同需求选择合适的库使你的 Web 开发变得更简单。
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/2132633297)

> 熟悉 Python 和 Scala 之类的语言的小伙伴应该知道，他们在很早就已经有了交互式编程环境 REPL（Read-Eval-PrintLoop），REPL 以交互式的方式对语句和表达式进行求值。REPL 提供

原

 2023-11-19  阅读 (91)  点赞 (0)

熟悉 `Python` 和 `Scala` 之类的语言的小伙伴应该知道，他们在很早就已经有了交互式编程环境 `REPL`（`Read-Eval-Print Loop`），`REPL` 以交互式的方式对语句和表达式进行求值。`REPL` 提供了一个交互式的方式，允许开发人员输入代码，立即执行它，无需编译，然后查看执行结果。

Java 在 Java 9 引入 Java 版的 `REPL` 工具：`Java Shell`。

什么是 JShell
----------

`Java Shell` 是 Java 在 Java 9 中引入的一个交互式编程工具，它可以让开发人员能够在一个命令行界面即时编写、编辑和执行 Java 代码片段，而无需创建和编译传统的 Java 源代码文件。

`JShell`的目的是提供一个更快速、更便捷的方式来学习和测试 Java 代码，以及进行原型设计和实验性编程。它主要有如下几个特点：

1.  **即时反馈**：可以立即查看代码的输出，不需要等待编译和运行整个 Java 应用程序。
2.  ** 代码片段：** 您可以编写单个 Java 表达式、语句或方法，并在`JShell`中立即执行它们。这对于测试和验证代码的特定部分非常有用。
3.  **自动导入**：`JShell`自动导入常见的 Java 类和包，因此您无需手动导入它们，使代码编写更加简洁。
4.  **脚本化**：我们可以将多个命令保存在脚本文件中，并使用`JShell`执行这些脚本。

`JShell` 通常用于教学、快速原型设计和测试，以及在 Java 开发中进行快速的探索性编程，但是它不适用于开发大型的 Java 应用程序。

为什么需要使用 **JShell**
------------------

我们先看我们以前刚刚学习 Java 时写的第一个 `hello world` 的程序步骤：

1.  打开 idea，并新建一个 `java-study` 的工程
2.  配置该工程的 Java 环境
3.  新建一个 `hello world` 的 Java 类
4.  编译 Java 文件，并修复各种问题
5.  运行程序

该步骤比较繁琐，而使用 `JShell` 工具，我们可以一次输入一个程序元素，立即看到执行结果，并根据需要进行调整。

`JShell`对于初学者非常有用，因为它允许开发者快速编写和测试 Java 代码片段，无需担心完整项目结构和编译过程。这有助于新手更容易地理解 Java 语言的基础概念。同时由于不需要创建完整的应用程序，这可以加速原型设计过程，帮助我们更快地验证概念。

> 但是 `JShell` 不能代替 IDE，它只能作为辅助和便捷的工具而已。

使用 **JShell**
-------------

要使用 `JShell` 必须安装 JDK 9 或更高版本。

> 启动 `JShell`

使用 jshell 命令即可启动 `JShell`，如果启动不了，可能是环境变量配置配置好，可以进入 “ `bin` ” 目录，通过该命令启动 `JShell`。

```
 chenssy@chenssydeMacBook-Pro  ~  jshell
|  欢迎使用 JShell -- 版本 17.0.8
|  要大致了解该版本, 请键入: /help intro


```

启动 `JShell` 我们就可以在该窗口测试代码了。

在 JShell 中我们可以直接**声明变量**、**创建方法**、**调用方法**：

> **声明变量**

```
jshell> int a = 1;
a ==> 1

jshell> int b = 2;
b ==> 2


```

这里声明了两个变量 a 和 b。

> **创建方法**

```
jshell> int add(int x,int y) {
   ...>     return x + y;
   ...> }
|  已创建 方法 add(int,int)


```

新建了一个 `add()` 方法；

> **调用方法**

我们可以直接利用上面的变量 a 、b 来调用 `add()`:

```
jshell> int c = add(a,b)
c ==> 3


```

> **运行表达式**

`JShell` 还可以直接输入 Java 表达式，计算出表达式的结果，然后返回。

```
jshell> 1 + 2
$6 ==> 3

jshell> 1 > 2 ? "死磕 Java 并发" : "死磕 Java 新特性"
$7 ==> "死磕 Java 新特性"

jshell>


```

> **创建 Java 类**

`JShell` 可以新建 Java 类，然后调用它。

```
jshell> public class SKTest{
   ...>     public void test(){
   ...>         System.out.println("死磕 Java 就是牛");
   ...>     }
   ...> }
|  已创建 类 SKTest

jshell> SKTest skTest = new SKTest();
skTest ==> SKTest@17a7cec2

jshell> skTest.test();
死磕 Java 就是牛


```

> **定义接口**

我们还可以定义接口，并实现它。

```
jshell> public interface UserService{
   ...>     void test();
   ...> }
|  已创建 接口 UserService

jshell> public class UserServiceImpl implements UserService{
   ...>     public void test(){
   ...>         System.out.println("死磕 Java 新特性就是牛...");
   ...>     }
   ...> }
|  已创建 类 UserServiceImpl

jshell> UserService userService = new UserServiceImpl();
userService ==> UserServiceImpl@4783da3f

jshell> userService.test();


```

其他的情况大明哥就不一一介绍了，个人感觉这个工具还是显得有点儿鸡肋，比如你新建一个类，语法错了需要重新写，这就很鸡肋了，不过 Mac book 的 iTerm 倒是可以回退整块语句，也算方便了不少。

JShell 的主要命令
------------

JShell 提供了一系列主要命令，用于在交互式环境中执行各种任务和操作。以下表格是一些常用的命令：

<table><thead><tr><th>命令</th><th>描述</th></tr></thead><tbody><tr><td><code>/help</code> (或 <code>/?</code>)</td><td>查看可用命令的帮助信息，或获取特定命令的详细信息。</td></tr><tr><td><code>/list</code> (或 <code>/l</code>)</td><td>列出当前已经输入的代码片段。</td></tr><tr><td><code>/edit</code> (或 <code>/e</code>)</td><td>编辑以前输入的代码片段。</td></tr><tr><td><code>/drop </code>(或<code> /d</code>)</td><td>删除以前输入的代码片段。</td></tr><tr><td><code>/save </code>(或 <code>/s</code>)</td><td>将当前会话中的代码保存到文件中。</td></tr><tr><td><code>/open</code> (或<code>/o</code>)</td><td>从文件中加载代码以继续会话。</td></tr><tr><td><code>/reset</code></td><td>重置<code>JShell</code>环境，清除所有已输入的代码片段。</td></tr><tr><td><code>/vars</code> (或 <code>/v</code>)</td><td>查看当前定义的所有变量。</td></tr><tr><td><code>/methods</code> (或 <code>/m</code>)</td><td>查看当前定义的所有方法。</td></tr><tr><td><code>/types</code> (或 <code>/t</code>)</td><td>查看当前定义的所有类型。</td></tr><tr><td><code>/imports</code> (或<code>/i</code>)</td><td>查看当前导入的包和类。</td></tr><tr><td><code>/set</code> (或 <code>/s</code>)</td><td>设置<code>JShell</code>的各种选项，如类路径、编辑器、输出格式等。</td></tr><tr><td><code>/classpath</code> (或 <code>/cp</code>)</td><td>查看或设置类路径，以便加载外部类和库。</td></tr><tr><td><code>/history </code>(或 <code>/h</code>)</td><td>查看和搜索<code>JShell</code>历史记录。</td></tr><tr><td><code>/save</code> (或 <code>/s</code>)</td><td>将<code>JShell</code>历史记录保存到文件中，以便将其用于以后的会话。</td></tr></tbody></table>
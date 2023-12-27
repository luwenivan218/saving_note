> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/6780334511)

> 引言 Java 初学者你的第一个 Class 类一定是下面这个：publicclassHelloWorld{publicstaticvoidmain(String[]args){System.out.prin

原

 2023-12-23  阅读 (34)  点赞 (0)

引言
--

Java 初学者你的第一个 Class 类一定是下面这个：

```
public class HelloWorld {
  public static void main(String[] args) {
    System.out.println("Hello, World!");
  }
}



```

这个方法对老手来说这个类非常简单，但是对于新手而言，这个类就比较复杂了，因为涉及到了 Java 里面好几个概念，大明哥给你拆解下：

1.  `public class HelloWorld`
    1.  `public`：这是一个访问修饰符，表示这个类对所有类都是可见的。
    2.  `class`：这是一个关键字，用来定义一个类。
    3.  `HelloWorld`：这是类的名称。
2.  `public static void main(String[] args)`
    1.  `public`：访问修饰符，表示这个方法对所有类都是可见的。
    2.  `static`：Java 的一个关键字，表示这个方法可以在没有创建类的对象的情况下被调用。
    3.  `void`：方法的返回类型，`void`表示这个方法不返回任何值。
    4.  `main`：方法的名称，`main()` 是 Java 程序的入口点。
    5.  `String[] args`：`main()` 的参数，`args`是一个字符串数组，可以从命令行接收参数。这个参数非常神秘而且无用，因为它几乎没有用到过。
3.  `System.out.println("Hello, World!")`
    1.  `System`：一个预定义的 Java 类，包含用于标准输入、输出等的方法和变量。
    2.  `out`：`System`类的一个静态成员字段，表示标准输出流。
    3.  `println`：`PrintStream`类的一个方法，用于输出信息并换行。
    4.  `"Hello, World!"`：这是被输出的字符串。

如果你学习 Java 的第一课，你老师把这个拆解来解释给你看，你会不会晕？而且还告诉你这是 Java 中最简单的没有之一，你会不会退却？我学习的时候，我老师就一句话，记住就行，而且那个时候我们还是用记事本敲，经常敲不对。这无形中就提高的初学者的入门门槛。

为了方便入门和教学，Java 21 引入特性：未命名类和实例的 Main 方法。它主要包含两部分内容:

*   **未命名的 Java 类**：指的是没有明确命名的类，它们可以用来创建对象并调用方法。
*   **新的启动协议**：允许更简单地运行 Java 类，并且无需太多样板。

实例 main 方法
----------

Java 21 增强了启动 Java 程序的协议，允许实例直接使用 main 方法。它做了下精简：

1.  实例 main 方法，就意味着可以是 `non-static` 的。
2.  `main()` 的访问修饰符可以不必是 `public` 的，只需要是 `non-private`（也即`public`, `protected` 和 `package-protected`）的即可。
3.  `main()` 中的 `String[] args` 将是可选传入的。

所以一个 main() 可以精简成这样：

```
class HelloWorld { 
    void main() { 
        System.out.println("Hello, World!");
    }
}


```

未命名类
----

Java 21 还引入未命名的类来实现隐式声明，也就是我们可以写类名，也可以不写类名，故而上面的代码可以进一步精简：

```
void main() { 
  System.out.println("Hello, World!");
}


```

当一个类中不包含任何类声明，而仅有方法声明和成员变量声明时，该类便被称为 “未命名类”。由于无类申明，所以是无法使用静态方法的，但是可以使用 `this` 关键字或非静态方法的方法引用。

同时，未命名类是可以拥有成员变量和成员方法的，如下：

```
String skjava = "死磕 Java 新特性";
 
String getSk() { return skjava; }
 
void main() {
    System.out.println(getSk());
}


```

这两个特性还处于预览阶段，而且[大明哥](https://www.skjava.com/series/article/www.skjava.com)认为对于我们在实际开发过程中意义并不大。
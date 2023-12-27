> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1747461365)

> 什么是 Lambda 表达式 Lambda 表达式是在 Java8 中引入，并且被吹捧为 Java8 最大的特性。它是函数式编程的的一个重要特性，标志着 Java 向函数式编程迈出了重要的第一步。它的语法如下：(para

![](https://sike.skjava.com/java-features/202310031000003.png)

什么是 Lambda 表达式
--------------

Lambda 表达式是在 Java 8 中引入，并且被吹捧为 Java 8 最大的特性。它是函数式编程的的一个重要特性，标志着 Java 向函数式编程迈出了重要的第一步。

它的语法如下：

```
(parameters) -> expression


```

或者

```
(parameters) -> { statements; }


```

其中

*   `parameters` ：是 Lambda 表达式的参数列表，可以为空或包含一个或多个参数。
*   `->` ：是 Lambda 操作符，用于将参数和 Lambda 主体分开。
*   `expression` ：是 Lambda 表达式的返回值，或者在主体中执行的单一表达式。
*   `{ statements; }` ：是 Lambda 主体，包含了一系列语句，如果需要执行多个操作，就需要使用这种形式。

Java 8 引入 Lambda 表达式的主要作用是简化部分匿名内部类的写法。使用它可以完成用少量的代码实现复杂的功能，极大的简化代码代码量和代码结构。同时，JDK 中也增加了大量的内置函数式接口供我们使用，使得在使用 Lambda 表达式时更加简单、高效。

下面我们就来看它的一些常见用法。

常见用法
----

### 无参数，无返回值

例如 `Runnable` 接口的 `run()`。

在 Java 8 版本之前的版本，我们一般都是这样用：

```
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("死磕 Java 就是牛逼...");
            }
        }).start();


```

从 Java 8 开始，无参数匿名内部类可以简写成如下这种方式：

```
() -> {
    执行语句
}


```

所以上面代码可以简写成这样的：

```
new Thread(() -> System.out.println("死磕 Java 就是牛逼...")).start();


```

### 单参数，无返回值

只有一个参数，无返回值，如下：

```
(x) -> System.out.println(x);


```

在 Java 8 中，有一个函数式接口 Consumer，它定义如下：

```
@FunctionalInterface
public interface Consumer<T> {

    void accept(T t);
}


```

我们用它来演示下：

```
        Consumer<String> consumer = (String s) -> {
            System.out.println(s);
        };
        
        consumer.accept("死磕 Java 就是牛...");


```

是不是比较简便，但是这段代码还不够简便，它还可以进行多次优化，

*   如果 Lambda 主体只有一条语句，则 `{、}` 可以省略

```
Consumer<String> consumer = (String s) -> System.out.println(s);


```

*   Lambda 表达式有一个依据：**类型推断机制**。在上下文信息足够的情况下，编译器可以推断出参数表的类型，而不需要显式指名。所以 `(String s)` 可以简写为 `(s)`：

```
Consumer<String> consumer = (s) -> System.out.println(s);


```

*   对于只有一个参数的情况，左侧括号可以省略：

```
Consumer<String> consumer = s -> System.out.println(s);



```

### 多参数，有返回值

如 Comparator 接口的 `compare(T o1, T o2)` 方法，在 Java 8 之前，写法如下：

```
        Comparator<Integer> comparator = new Comparator<Integer>() {
            @Override
            public int compare(Integer o1, Integer o2) {
                System.out.println("o1:" + o1);
                System.out.println("o2:" + o2);
                return o1.compareTo(o2);
            }
        };
        
        comparator.compare(12,13);


```

使用 Lambda 表达式后：

```
        Comparator<Integer> comparator = (o1, o2) -> {
            System.out.println("o1:" + o1);
            System.out.println("o2:" + o2);
            return o1.compareTo(o2);
        };

        comparator.compare(12,13);


```

当然，如果去掉 `System.out.println()`，还可以简写为 `Comparator<Integer> comparator = (o1, o2) -> o1.compareTo(o2);` ，这里是可以省略 return 关键字的。

这里就 Lambda 的简写做一个总结：

1.  **类型推断**：编译器可以根据上下文推断 Lambda 表达式的参数类型，从而可以省略参数类型的声明。
2.  **单一参数**：当 Lambda 表达式只有一个参数时，可以省略参数外的括号。如: `(x) → x * 2` 可以简写为 `x → x * 2` 。
3.  **单表达式**：当 Lambda 表达式只有一行代码时，可以省略大括号和 return 关键字。如 `(x,y) → {return x + y}` 可以简写为 `(x,y) → x + y`。

Lambda 简写依据
-----------

Lambda 简写的依据有两个：

> **1、必须有相应的函数式接口**

所谓函数式接口函数式就是指只包含一个抽象方法的接口，它是在 Java 8 版本中引入的，其主要目的是支持函数式编程，有了函数式接口我们可以将函数作为参数传递、将函数作为返回值返回，同时也为使用 Lambda 表达式提供了支持。

函数式接口具有以下特征：

1.  **只包含一个抽象方法**：函数式接口只能有一个抽象方法，但可以包含多个默认方法或静态方法（Java 8 中有另一个新特性：`default` 关键字）。这个唯一的抽象方法通常用来表示某种功能或操作。
2.  **用** **`@FunctionalInterface`** **注解标记**：注解不强制，但通常会使用它来标记该接口为函数式接口。这样做可以让编译器检查接口是否符合函数式接口的定义，以避免不必要的错误。

> **2、类型推断机制**

类型推断机制则是允许编译器根据上下文自动推断 Lambda 表达式的参数类型。这个推断过程包括两个方面：

1.  **目标类型推断**

编译器会根据 Lambda 表达式在赋值、传参等地方的上下文来推断 Lambda 表达式的目标类型。例如，如果 Lambda 表达式被赋值给一个接口类型的变量，编译器会根据该接口的抽象方法来推断 Lambda 表达式的参数类型。

```
Runnable runnable = () -> System.out.println("死磕 Java 就是牛...");


```

Lambda 表达式被赋值给了 Runnable 类型的变量，所以编译器知道 Lambda 表达式需要没有参数且返回类型为`void`的方法。

1.  **参数类型推断**

如果 Lambda 表达式的参数类型可以从上下文中唯一确定，编译器会自动推断参数的类型。例如：

```
List<String> skList = Arrays.asList("死磕 Java 并发", "死磕 Netty", "死磕 NIO","死磕 Spring");
skList.forEach(sk -> System.out.println(sk));


```

`forEach`方法期望一个参数类型为`Consumer<String>`的函数，编译器可以从 `sk`的类型推断出 Lambda 表达式的参数类型为`String`。

虽然类型推断机制允许省略 Lambda 表达式的参数类型，但有时候显式声明参数类型可以增强代码的可读性和处理复杂的泛型情况，这个时候我们还是将参数类型写上会显得更加友好。
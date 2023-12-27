> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1501251408)

> 局部变量类型推断是 Java10 引入的，通过使用 var 关键字允许编译器推断变量的类型，这大大简化了 Java 代码的编写。但是在 Java10 中我们是不能在 Lambda 参数使用 var，这在 Java11 中得到了

原

 2023-11-28  阅读 (63)  点赞 (0)

局部变量类型推断是 Java 10 引入的，通过使用`var`关键字允许编译器推断变量的类型，这大大简化了 Java 代码的编写。但是在 Java 10 中我们是不能在 Lambda 参数使用`var`，这在 Java 11 中得到了改进。

关于局部变量类型推断参考文章：[Java 10 新特性—局部变量类型推断](https://www.skjava.com/series/article/www.skjava.com)

在 Java 11 之前，我们是不能在 Lambda 表达式的参数中使用`var`，例如：

```
Function<String, String> func = (var str) -> str.toUpperCase();


```

这种写法在 Java 11 之前是非法的，但是在 Java 11 中是合法的，在这个例子中，`str` 的类型被推断为`String`。

由于可以在 Lambda 表达式中可以使用 var 了，所以我们还可以将注解与类型推断结合使用。例如：

```
BiFunction<String, String, String> biFunction = (@NonNull var a, @NonNull var b) -> a + b;



```

在这里，`a`和`b`都被推断为`String`类型，且都使用了`@NonNull`注解。
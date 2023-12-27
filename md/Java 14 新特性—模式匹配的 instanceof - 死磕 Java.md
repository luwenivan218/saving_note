> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/3436282193)

> instanceof 是 Java 中的一个关键字，它用于检查一个对象是否是特定类的实例或者该类的子类的实例。它通常用在条件语句中，以确定对象的类型，从而避免在向下转型时发生 ClassCastExcepti

原

 2023-12-20  阅读 (43)  点赞 (0)

`instanceof` 是 Java 中的一个关键字，它用于检查一个对象是否是特定类的实例或者该类的子类的实例。它通常用在条件语句中，以确定对象的类型，从而避免在向下转型时发生 `ClassCastException`。

在使用 `instanceof` 时，如果左边的实例属于右边的类或接口，或者是他们的子类，那么 `instanceof` 就会返回 `true`，否则，返回 `false`。这是一种类型安全的检查方式，用来保证在将对象转型为更具体的类型之前，这个对象确实是这个类型的，如下：

```
if (object instanceof String) {
    
    String str = (String) object;
}


```

虽然这段代码没有错误，但是它存在两个不足：

*   在使用 `instanceof` 检查后，还需要对其进行强制类型转换来访问特定类型的成员，这种方很明显就是多余的，我明明知道它是某种类型，为什么还需要进行强制转换这一步骤呢？
*   同时，有可能会出错，因为它将类型检测和转换分为了两个部分，由于开发者的原因有可能会导致两个步骤的类型不一致，而导致 `ClassCastException`。

为了解决这两个不足，Java 14 引入模式匹配的 `instanceof`。该特性允许在 `instanceof` 操作符的条件判断中直接定义一个变量，如果对象是指定的类型，这个变量会被初始化为被检查的对象，可以立即使用，无需额外的类型转换。

```
if (obj instanceof String str) {
    
}



```

这里 `str` 是一个绑定变量，只有当 `obj` 是一个 `String` 类型的实例时它才会被初始化。

该特性即省略了强制转换的冗余步骤，也保证了类型检查和转换的原子性，避免了 `ClassCastException`。

但是，该特性是预览特性，未来可能会发生变化，虽然很方便，但在成为标准前请谨慎使用。
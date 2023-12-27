> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/2019475189)

> 注解，我相信小伙伴应该都使用过，Java 从 Java5 开始引入该特性，发展到现在已经是遍地开花了，且在很多框架都得到了广泛的使用，用来简化程序中的配置。但是，在 Java8 之前，注解仅能用于声明（如方法、

注解，我相信小伙伴应该都使用过，Java 从 Java 5 开始引入该特性，发展到现在已经是遍地开花了，且在很多框架都得到了广泛的使用，用来简化程序中的配置。

但是，在 Java 8 之前，注解仅能用于声明（如方法、类、字段）。这意味着注解无法直接应用于类型本身（例如，方法的返回类型、变量的类型等等）。这种限制减少了注解在代码分析、检查及处理中的潜在用途（虽然不会产生什么问题）。为了提升注解的功能，使其能够更全面地支持各种编程场景（如增强静态代码分析、提供更丰富的编译时检查等等），Java 8 引入了类型注解。

类型注解可以用在哪些地方？
-------------

类型注解扩展了注解的应用范围，使其不仅能应用于声明，还能应用于任何使用类型的地方，包括：

> 1、**泛型类型参数**

可以在泛型类型中使用注解，如：

```
List<@NonNull String> strings = new ArrayList<>();


```

`@NonNull` 注解表示列表中的字符串不应该为 `null`。

> 2、**类型转换**

在类型转换表达式中使用注解，如：

```
Object obj = "skjava.com";
String str = (@NonNull String) obj;


```

将一个对象转换为字符串，并通过 `@NonNull` 注解标明转换后的字符串不能为 `null`。

> **实现语句**

在实现接口或扩展类时使用注解，如：

```
class CustomList implements @ReadOnly List<String> {
    
}


```

`@ReadOnly` 表示这个实现`List`接口的 `CustomList` 是只读的。

> **方法或构造器的返回类型**

方法声明的返回类型前可以添加注解，如：

```
public @Positive int getPositiveNumber() {
    return 42;
}


```

`@Positive` 注解表示这个方法应该返回一个正数。

> **异常声明**

在方法抛出的异常类型上也可以添加注解，如：

```
public void readFile(String path) throws @Critical IOException {
    
}



```

`@Critical` 注解被用于标记抛出的 `IOException` 是关键的，可能需要特别处理。

> **局部变量**

```
@NonNull String skjava = "死磕 Java 新特性";


```

`@NonNull` 注解声明局部变量 `skjava` 不会为 null。

> **方法或构造器的参数**

```
public void doSomething(@NonEmpty List<String> list) {
    
}



```

`@NonEmpty` 注解表明方法参数 `list` 应该是一个非空的列表。

> **类的实例化**

```
MyObject myObject = new @Interned MyObject();



```

`@Interned` 注解可以用来指示 `MyObject` 的这个实例应该被内部化（interning）处理。

类型注解有什么作用？
----------

Java 8 引入类型注解的主要目的是增强 Java 的类型检查能力，提供更加丰富的代码分析工具，同时帮助避免常见的错误。它的作用包括但不限于数据校验、类型检查、代码分析等等。例如下面一个例子利用类型注解来避免空指针异常：

```
    @Test
    public void test() {
        printLength("skjava.com");
        printLength(null);
    }

    public void printLength(@NonNull String str) {
        System.out.println(str.length());
    }


```

在这个例子中，`printLength()` 的参数 `str`被标记为 `@NonNull`。这意味着在编译时，编译器会检查是否有可能传入一个 `null` 值。如果有这样的可能性，编译器可以生成一个警告或错误，从而防止可能的 `NullPointerException`，例如在 idea 中会有这个告警：

![](https://sike.skjava.com/java-features/202311191000001.png)

通过使用类型注解，我们可以在编写代码的时候就避免潜在的运行时错误，增强代码的健壮性。

虽然使用类型注解有这些优点，但是它会增加代码的复杂度，如果过度使用或使用不当，类型注解可能会使代码变得更加复杂和难以理解，不如这段代码：

```
@NotEmpty List<@NonNull String> strings = new ArrayList<@NonNull String>()>


```

看着就脑袋痛，所以该特性我们需要适当使用，但是在一些核心场景，容易出错的地方，尤其是参数类型检查，大明哥还是推荐使用，可以带来很大的便利。
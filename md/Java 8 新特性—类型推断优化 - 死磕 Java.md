> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/6176475946)

> 理解泛型在讨论类型推断之间，我们有必要先理解下泛型。泛型是 Java1.5 引入的特性，主要目的是增强 Java 程序的类型安全性，同时提高代码的重用性和可读性。比如，在泛型引入之前，所有的集合都是保存 Obj

![](https://sike.skjava.com/java-features/202311232000003.png)

理解泛型
----

在讨论类型推断之间，我们有必要先理解下泛型。

泛型是 Java 1.5 引入的特性，主要目的是增强 Java 程序的类型安全性，同时提高代码的重用性和可读性。比如，在泛型引入之前，所有的集合都是保存 `Object` 类型的，很容易意外地将错误类型的对象放入集合中，这可能导致运行时异常。

```
List list = new ArrayList();
list.add("skjava.com");
list.add(1234);


```

同时，从集合中取出的对象也是 `Object` 类型的，需要我们显式强制转换为适当的类型：

```
String str = (String) list.get(0);



```

通过引入泛型可以解决这两个问题：

*   泛型通过在编译时提供类型检查，能够减少因为存放错误类型的错误。比如：

```
List<String> list = new ArrayList<String>();
list.add("skjava.com");
list.add(1234);    


```

*   泛型消除显示的强制类型转换，因为编译器能够自动地处理类型转换的细节，比如：

```
String str = list.get(0);     


```

当然，除了集合外，泛型还能够泛化方法和类，我们可以创建泛型类或泛型方法，它们可以使用多种类型而不是单一的类型。这样就增加了代码的灵活性和可重用性。

泛型虽好，但是它有一个坑，就是每次定义时都要写明泛型的类型，显得非常冗余，比如：

```
List<String> list = new ArrayList<String>();
list.add("skjava.com");


```

我明明在定义变量的时候就已经指明了参数类型，为什么在初始化的时候还要我指定呢？这不显得多余吗？

Java 7 类型推断优化
-------------

Java 7 为了解决这个冗余的问题，引入 “钻石操作符”（即 `<>`）用来简化泛型实例的创建过程。钻石操作符 `<>` 允许编译器根据上下文推断出泛型的类型，从而避免了重复的类型声明。比如：

```
List<String> list = new ArrayList<String>();


```

可以优化为：

```
List<String> list = new ArrayList<>();


```

在这里，`new ArrayList<>` 中的 `<>` 操作符使得编译器能够自动推断出其类型为 `String`。

一定要注意`new ArrayList` 后面的 “`<>`”，只有加上这个 “`<>`” 才表示是自动类型推断，否则就是非泛型类型的 ArrayList，并且在使用编译器编译源代码时会给出一个警告提示。

但是 Java 7 的类型推断还是有缺陷：

*   只有构造器的参数化类型在上下文中被显著的声明了，才可以使用类型推断，否则不行，例如：

![](https://sike.skjava.com/java-features/202311232000002.png)

这里 `new ArrayList<>()` 需要明确指定类型：

```
List<String> list = new ArrayList<>();
list.add("skjava.com");
list.addAll(new ArrayList<String>());


```

Java 8 类型推断优化
-------------

为了简化代码的编写，减少冗余信息以及 Lambda 表达式，Java 8 对类型推断进行进一步的优化，主要内容有：

1.  与 Lambda 表达式和方法引用的协同
2.  Stream API 中的应用

### **与 Lambda 表达式和方法引用的协同**

Java 8 引入 Lambda 表达式极大地简化代码代码量和代码结构，标志着 Java 向函数式编程迈出了重要的第一步。而简写 Lambda 表示的一个依据就是类型推断，它允许编译器根据上下文自动推断 Lambda 表达式的参数类型，它分为目标类型推断和参数类型推断。

更多关于 Lambda 表达式的请阅读：[Java 8 新特性—Lambda 表达式](https://www.skjava.com/series/article/www.skjava.com)

### Stream API 中的应用

Java 8 引入 Stream API 极大地提高了 Java 对集合的操作能力，类型推断使得在使用 Stream API 时让代码更加简介和阅读，比如，不使用类型推断：

```
Stream<String> stream = list.stream();
stream.filter((String s) -> s.startsWith("sike"));


```

使用类型推断的写法：

```
list.stream().filter(s -> s.startsWith("sike"));


```

在这里，编译器能够从上下文推断出 `s` 的类型是 `String`。

Stream API 支持的链式操作，类型推断在这里也发挥着重要作用，使得这些链式调用更加简洁：

```
List<String> filteredList = list.stream()
                                .filter(s -> s.startsWith("sike"))
                                .map(String::toUpperCase)
                                .collect(Collectors.toList());


```

`filter`、`map` 和 `collect` 操作形成了一个操作链，每个操作的输出类型自动成为下一个操作的输入类型，无需显式指定。你说，如果这里需要强制转换得要多麻烦？
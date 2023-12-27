> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1624717282)

> 在文章 Optional 解决 NullPointerException 大明哥详细的介绍如何利用 Optional 来解决 NullPointerException。然后 Java9 对它又进行了一些增强，以提高其实

在文章 [Optional 解决 NullPointerException](https://www.skjava.com/series/article/1810194665) 大明哥详细的介绍如何利用 `Optional` 来解决 `NullPointerException` 。然后 Java 9 对它又进行了一些增强，以提高其实用性和易用性。主要是增加三个方法：

*   `or()`
*   `ifPresentOrElse()`
*   `stream()`

下面就一一介绍这三个方法。

or()
----

`or()` 方法定义如下：

```
public Optional<T> or(Supplier<? extends Optional<? extends T>> supplier);


```

方法接受一个 `Supplier`，当 `Optional` 为空时，将执行该 `Supplier`，以获取另一个 `Optional` 对象。这使得在处理 `Optional` 时更加灵活，可以轻松地提供备用值或计算。

```
    @Test
    public void orTest() {
        Optional<String> optional1 = Optional.of("死磕 Java 新特性");
        Optional<String> optional2 = Optional.empty();
        System.out.println("optional1 value:" + optional1.or(()->Optional.of("死磕 Java")).get());
        System.out.println("optional2 value:" + optional2.or(()->Optional.of("死磕 Java")).get());
    }

optional1 value:死磕 Java 新特性
optional2 value:死磕 Java



```

ifPresentOrElse()
-----------------

Java 8 有一个 `ifPresent()`，它用于在 `Optional` 包含非空值时执行指定的操作，Java 9 对其的改进就是增加了一个 `else`。

`ifPresentOrElse()` 方法定义如下：

```
public void ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction);


```

它接受两个参数：

1.  `action`：是一个 `Consumer` 函数式接口，用于在 `Optional` 包含非空值时执行的操作。
2.  `emptyAction`：是一个 Runnable 函数式接口，用于在 `Optional` 为空时执行的操作，通常是一些默认操作或错误处理。

使用 `ifPresentOrElse()` 方法，你可以避免手动编写空值检查代码，从而更容易地处理可能为空的情况。例如：

```
    @Test
    public void ifPresentOrElseTest() {
        Optional<String> optional = Optional.empty();
        optional.ifPresentOrElse(value -> System.out.println("optional value:" + optional.get()),
                () -> System.out.println("optional value is null"));
    }

optional value is null



```

该方法可以让我们不需要显式地编写 `if...else` 来检查 `Optional` 是否为空，从而提高了代码的可读性和可维护性，是比较实用的一个方法。

stream()
--------

`stream`() 允许我们将 `Optional` 对象转化为一个流 （`Stream`），以便能够更灵活地处理包含或不包含值的情况。方法定义如下：

```
Stream<T> stream()


```

该方法允许我们执行以下操作：

*   如果 `Optional` 包含非空值 **：** `stream()` 会创建一个包含这个值的流。
*   如果 `Optional` 为空 **：** `stream()` 会创建一个空流。

有了 `Stream` 后，我们就可以利用 `Stream` 的各种功能来处理 `Optional` 中的值，例如映射、过滤、转换等等。例如：

```
    @Test
    public void streamTest() {
        Optional<String> optional = Optional.of("死磕 Java");
        optional.stream()
                .filter(o -> o.startsWith("死磕"))
                .map(val -> val + " —— https://skjava.com/")
                .forEach(System.out::println);
    }

死磕 Java-https:



```

再入：

```
    @Test
    public void streamTest1() {
        List<Optional<String>> list = List.of(
                Optional.of("死磕 java 并发"),
                Optional.of("死磕 java 新特性"),
                Optional.empty(),
                Optional.of("死磕 Netty"),
                Optional.empty()
        );
        list.stream()
                .flatMap(Optional::stream)
                .filter(o -> o.startsWith("死磕"))
                .map(val -> val + " —— https://skjava.com/")
                .forEach(System.out::println);
    }

死磕 java 并发 —— https:
死磕 java 新特性 —— https:
死磕 Netty —— https:



```

通过 `stream()` 可以让我们更加流畅地处理 `Optional` 中的值，而不需要额外的空值检查代码。
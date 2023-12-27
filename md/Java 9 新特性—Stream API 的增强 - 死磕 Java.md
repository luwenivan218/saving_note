> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/2547807010)

> 我们知道 Java8 强势推出 StreamAPI，让我们能够以一种更加简洁、易读、高效的方式来处理集合数据，大大地提高了我们的生产力，在文章 StreamAPI 对元素流进行函数式操作，大明哥对 StreamA

我们知道 Java 8 强势推出 `Stream API`，让我们能够以一种更加简洁、易读、高效的方式来处理集合数据，大大地提高了我们的生产力，在文章 [Stream API 对元素流进行函数式操作](https://www.skjava.com/series/article/6972779809)，大明哥对 `Stream API` 做了非常详细的说明，各位小伙伴可以去阅读阅读。

为了提高 Stream API 的灵活性和性能，Java 9 对 `Stream API` 做了一些增强，主要体现在以下几个方法：

1.  新增 `ofNullable()`
2.  重载 `iterate()`
3.  新增 `dropWhile()` 和 `takeWhile()`

![](https://sike.skjava.com/java-features/202311011000001.png)

新增 ofNullable()
---------------

`ofNullable()` 用于创建一个 `Stream`，其中包含一个非空元素或者为空。该方法的主要目的是简化处理可能包含 `null` 值的集合时的代码，以便避免显式地检查和过滤 `null` 值。其定义如下：

```
    public static<T> Stream<T> ofNullable(T t) {
        return t == null ? Stream.empty()
                         : StreamSupport.stream(new Streams.StreamBuilderImpl<>(t), false);
    }



```

*   如果参数 `t` 不为 null，则返回一个包含该非空元素的单元素 Stream。
*   如果参数 `t` 为 null，则返回一个空的 Stream。

这就意味着我们可以使用 `Stream.ofNullable()` 方法来快速创建一个只包含非空元素的 Stream，且无需担心处理 null 值的边界情况。比如，我们有一个 List 包含一些可能为 null 的字符串，我们可以使用 `Stream.ofNullable()` 来创建一个只包含非空字符串的 Stream：

```
    @Test
    public void ofNullableTest() {
        List<String> list = Arrays.asList("死磕 Java",null,"死磕 Java 新特性","死磕 Netty",null,null);
        list.stream()
            .flatMap(Stream::ofNullable)
            .forEach(System.out::println);
    }


```

重载 iterate()
------------

我们知道 Java 8 中的 `iterate()` 用于创建一个无限流，其元素由给定的初始值和一个生成下一个元素的函数产生，为了终止流我们需要使用一些限制性的函数来操作，例如 `limit()`：

```
    @Test
    public void iterate() {
        Stream.iterate(1,v -> v + 2)
                .limit(10)
                .forEach(System.out::println);
    }


```

在 Java 9 中为了限制该无序流的长度，增加了一个谓词，方法定义如下：

```
static <T> Stream<T> iterate(T seed, Predicate<? super T> hasNext, UnaryOperator<T> next)


```

*   `seed`：初始元素，作为序列的第一个元素。
*   `hasNext`：用于在生成元素时限制序列的长度。如果 `hasNext` 返回 `true`，则`next` 函数就会继续生成下一个元素；一旦 `hasNext` 返回 `false`，序列生成将停止。

例如：

```
    @Test
    public void iterateTest() {
        Stream.iterate(1,n -> n < 20,v -> v + 2)
                .forEach(System.out::println);
    }


```

当生成的值大于等于 20 时，序列生成就停止。

该重载方法允许我们更加方便地生成元素序列，并在需要时限制序列的长度，这在处理无限序列的情况下非常有用。

新增 dropWhile() 和 takeWhile()
----------------------------

Java 9 引入 `dropWhile()` 和 `takeWhile()`，这两个方法允许我们根据谓词条件从流中选择或删除元素，直到遇到第一个不满足条件的元素。

`dropWhile()` 和 `takeWhile()`，用于处理流中的前缀元素，而不必处理整个流，为处理 Stream 流提供了更多的灵活性。

### dropWhile()

`dropWhile()` 方法定义如下：

```
dropWhile(Predicate<? super T> predicate)


```

*   方法接受一个 `Predicate`（谓词）作为参数。
*   方法会从流的开头开始，跳过满足谓词条件的所有元素，直到找到第一个不满足条件的元素。一旦找到第一个不满足条件的元素，`dropWhile()` 将停止跳过元素，并返回一个新的流，其中包含了剩余的元素。
*   如果流的所有元素都满足谓词条件，那么返回一个空流。

示例：

```
    @Test
    public void dropWhileTest() {
        Stream<Integer> stream = Stream.of(1,2,3,4,5,3,2,6,7,8);
        stream.dropWhile(x -> x < 5)
                .forEach(System.out::println);
    }
    }

5
3
2
6
7
8



```

谓词条件 `x < 5`，即 `dropWhile()` 会跳过小于 5 的元素， 一旦找到第一个不小于 5 的元素（5），它将停止跳过并返回包含剩余元素的新流（`5,3,2,6,7,8`）。

### takeWhile()

`takeWhile()` 则与 `dropWhile()` 相反，它是从流的开头开始，选择满足谓词条件的所有元素，直到找到第一个不满足条件的元素，一旦找到该元素，将会停止选择元素，并返回一个新的流，其中包含了前缀元素。注意两个方法的用词区别，一个是跳过（`dropWhile()`），一个是选择（`takeWhile()`）。

如果流的所有元素都满足谓词条件，它将返回原始流的一个副本。

示例：

```
    @Test
    public void takeWhileTest() {
        Stream<Integer> stream = Stream.of(1,2,3,4,5,3,2,6,7,8);
        stream.takeWhile(x -> x < 5)
                .forEach(System.out::println);
    }

1
2
3
4



```

看到没，同样的示例，只是方法不一样，效果就完全相反。
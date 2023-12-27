> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/6358413511)

> 对于大多数小伙伴而言，其实是很少使用只读集合的，甚至有蛮大一部分小伙伴可能都没有听过这个，大部分使用集合无非就是如下两种方式：List<String>list=newArrayList()

对于大多数小伙伴而言，其实是很少使用只读集合的，甚至有蛮大一部分小伙伴可能都没有听过这个，大部分使用集合无非就是如下两种方式：

```
List<String> list = new ArrayList();
or
List<String> list = getXxxList();



```

其实只读集合在编程中还是蛮有用处的，首先只读就意味着不可变，即线程安全，在使用它的过程中我们无需使用同步机制来保护其内部状态。同时只读集合通常比可变集合更加高效。因为它们是不可变的，不需要支持修改操作，因此在内部数据结构上可以进行优化，这可以提高数据访问的速度和降低内存开销。

这篇文章我们来看看 Java 引入的一项重要特性—只读集合和工厂方法。

Java 8 创建不可变集合
--------------

在 Java 9 之前要创建一个只读集合，通常需要通过将可变集合转换为不可变集合的方式来实现，即使用`Collections.unmodifiableXXX()`，其中`XXX`可以是`List`、`Set`或`Map`，例如要创建一个只读 List：

```
    @Test
    public void test() {
        List<String> list = new ArrayList<>();
        list.add("死磕 Java 新特性");
        list.add("死磕 Netty");

        
        list = Collections.unmodifiableList(list);
    }


```

使用 `Collections.unmodifiableList()` 将 List 转换为不可变集合，如果我们使用 `add()` 来添加元素，会报 `UnsupportedOperationException` 异常信息

这种方式虽然有效，但是比较麻烦，它需要额外的步骤来处理，而且容易出错。

Java 9 创建不可变集合
--------------

为了解决 Java 8 的问题，Java 9 引入不可变集合和对应的工厂方法，目的就在于提供更安全、更高效的方式来创建不可变集合，同时确保原始集合无法被修改。

`List`、`Set`、`Map` 都提供了对应的工厂方法来创建不可变集合，我们这里列出 `List` 的：

```
static <E> List<E>  of()
static <E> List<E>  of(E e1)
static <E> List<E>  of(E e1, E e2)
static <E> List<E>  of(E e1, E e2, E e3)
static <E> List<E>  of(E e1, E e2, E e3, E e4)
static <E> List<E>  of(E e1, E e2, E e3, E e4, E e5)
static <E> List<E>  of(E e1, E e2, E e3, E e4, E e5, E e6)
static <E> List<E>  of(E e1, E e2, E e3, E e4, E e5, E e6, E e7)
static <E> List<E>  of(E e1, E e2, E e3, E e4, E e5, E e6, E e7, E e8)
static <E> List<E>  of(E e1, E e2, E e3, E e4, E e5, E e6, E e7, E e8, E e9)
static <E> List<E>  of(E e1, E e2, E e3, E e4, E e5, E e6, E e7, E e8, E e9, E e10)

static <E> List<E>  of(E... elements)


```

看到这么多重载方法是不是有点儿懵逼，感觉是不是只需要有 `of()` 和 `of(E... elements)` 这两个方法就可以了，从实现的效果上来说，确实是可以。`of(E... elements)` 确实是一个非常灵活的方法，可以适用于多种情况，但是 Java 9 引入多个重载的 `List.of()` 方法并不是为了提供不同数量的参数选择的灵活性，而是为了性能和可读性的考虑。每个 `List.of()`方法的重载版本都是针对特定的参数数量进行了优化，以提高性能和代码清晰度。当使用 `of(E... elements)` 时，Java 运行时需要创建一个数组以容纳传递的元素（**Java 的可变参数，会被编译器转型为一个数组**），这可能会引入一些额外的开销，尤其是在创建小型 `List` 时。而多个重载的 `List.of()` 方法避免了这种开销，因为它们直接接受参数，而无需创建数组。所以，看着懵逼，但是性能杠杠的。

不可变的 List 具有如下几个特征：

1.  这些列表是不可变的。调用任何改变 List 的方法（如`add()`、`remove()`、`replaceAll()`、`clear()`），都会抛出 `UnsupportedOperationException`。
2.  它们不允许 `null` 元素。 尝试添加 `null` 元素将导致 `NullPointerException`。
3.  列表中元素的顺序与提供的参数或提供的数组中的元素的顺序相同。

对于 Set 和 Map 大明哥就不介绍了，小伙们自己去探索下！
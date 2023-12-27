> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/6677405627)

> Java9 开始引入不可变集合，我们通过 of() 即可创建一个不可变集合（详情见：Java9 新特性—新增只读集合和工厂方法）。但是有时候我们需要利用现有集合来创建一个不可变副本，然而 Java9 并没有提供该

原

 2023-11-26  阅读 (61)  点赞 (0)

Java 9 开始引入不可变集合，我们通过 `of()` 即可创建一个不可变集合（详情见：[Java 9 新特性—新增只读集合和工厂方法](https://www.skjava.com/series/article/6358413511)）。但是有时候我们需要利用现有集合来创建一个不可变副本，然而 Java 9 并没有提供该方法，所以 Java10 对其进行了增强。

Java 10 新增 `copyOf()` 用于创建现有集合的不可变副本。分为以下两种情况：

1.  如果原集合已经是不可变的，那么则返回原集合，
2.  如果元集合不是不可变的，那么则创建一个新的对象。

如下：

```
    @Test
    public void copyOfTest() {
        var list1 = List.of("死磕 Java 新特性","死磕 Java 并发","死磕 Netty");
        var copyList1 = List.copyOf(list1);
        System.out.println(list1 == copyList1);

        var list2 = Arrays.asList("死磕 Java 新特性","死磕 Java 并发","死磕 Netty");
        var copyList2 = List.copyOf(list2);
        System.out.println(list2 == copyList2);
    }


```

执行结果：

```
true
false


```

`List.copyOf(list1)` 由于 `list1` 本身是不可变集合，所以就直接将 `list1` 返回了，所以 `list1 == copyList1`，而 list2 是可变的，所以需要新建一个不可变对象，所以 `list2 != copyList2`。

从 `copyOf()` 的源码中我们也可以看出：

```
    static <E> List<E> listCopy(Collection<? extends E> coll) {
        if (coll instanceof List12 || (coll instanceof ListN && ! ((ListN<?>)coll).allowNulls)) {
            return (List<E>)coll;
        } else {
            return (List<E>)List.of(coll.toArray()); 
        }
    }


```
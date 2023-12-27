> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1895017259)

> 我们先看在 Java21 之前，访问 Java 中集合的第一个和最后一个元素的方式：集合获取第一个元素获取最后一个元素 Listlist.get(0)list.get(list.size()-1)Dequede

原

 2023-12-23  阅读 (55)  点赞 (0)

我们先看在 Java 21 之前，访问 Java 中集合的第一个和最后一个元素的方式：

<table><thead><tr><th>集合</th><th>获取第一个元素</th><th>获取最后一个元素</th></tr></thead><tbody><tr><td>List</td><td><code>list.get(0)</code></td><td><code>list.get(list.size() - 1)</code></td></tr><tr><td>Deque</td><td><code>deque.getFirst()</code></td><td><code>deque.getLast()</code></td></tr><tr><td>SortedSet</td><td><code>sortedSet.first()</code></td><td><code>sortedSet.last()</code></td></tr></tbody></table>

三个集合提供了三类不同的使用方法，非常混乱。为了解决这种混乱，Java 21 引入有序集合，旨在解决访问 Java 中各种集合类型的第一个和最后一个元素需要非统一且麻烦处理场景。

它新增了 `SequencedCollection`，`SequencedSet`，`SequencedMap` 三个接口，使得 Java 中的有序集合类可以按照统一的方法来进行操作。

*   `SequencedCollection`
*   `SequencedSet`
*   `SequencedMap`

下图很好地展示了新增的有序接口的关系。

![](https://sike.skjava.com/java-features/202312011000001.png)

`SequencedCollection` 接口定义定义了如下方法：

*   `addFirst()`：将元素添加为此集合的第一个元素。
*   `addLast()`：将元素添加为此集合的最后一个元素。
*   `getFirst()`：获取此集合的第一个元素。
*   `getLast()`：获取此集合的最后一个元素。
*   `removeFirst()`：移除并返回此集合的第一个元素。
*   `removeLast()`：移除并返回此集合的最后一个元素。
*   `reversed()`：倒叙此集合。

`SequencedMap` 接口定义了如下方法：

*   `firstEntry()`：返回此 Map 中的第一个 Entry，如果为空，返回 null。
*   `lastEntry()`：返回此 Map 中的最后一个 Entry，如果为空，返回 null。
*   `pollFirstEntry(`)：移除并返回此 Map 中的第一个 Entry。
*   `pollLastEntry()`：移除并返回此 Map 中的最后一个 Entry。
*   `putFirst()`：将 key-value 插入此 Map 中，如果该 key 已存在则会替换。注意，此操作完成后，该 key-value 就已经存在于此 Map 中，并且是第一位。
*   `putLast()`：将 key-value 插入此 Map 中，如果该 key 已存在则会替换。注意，此操作完成后，该 key-value 就已经存在于此 Map 中，并且是最后一位。
*   `reversed()`：倒叙此 Map。
*   `sequencedEntrySet()`：返回此 Map 的 Entry。
*   `sequencedKeySet()`：返回此 Map 的 keySet 的 SequencedSet 视图。
*   `sequencedValues()`：返回此 Map 的 value 集合的 SequencedCollection 视图。

下面来演示下。

*   SequencedCollection

```
    @Test
    public void sequencedCollectionTest() {
        List<String> listTemp = List.of("死磕Java","死磕Java新特性","死磕Netty","死磕Spring","死磕NIO","死磕Redis");

        List<String> list = new ArrayList<>(listTemp);
        Deque<String> deque = new ArrayDeque<>(listTemp);
        LinkedHashSet<String> linkedHashSet = new LinkedHashSet<>(listTemp);
        TreeSet<String> sortedSet = new TreeSet<>(listTemp);

        System.out.println("=========输出第一个元素==============");
        System.out.println("list.getFirst()：" + list.getFirst());
        System.out.println("deque.getFirst()：" + deque.getFirst());
        System.out.println("linkedHashSet.getFirst()：" + linkedHashSet.getFirst());
        System.out.println("sortedSet.getFirst()：" + sortedSet.getFirst());

        System.out.println("=========输出最后一个元素==============");
        System.out.println("list.getLast()：" + list.getLast());
        System.out.println("deque.getLast()：" + deque.getLast());
        System.out.println("linkedHashSet.getLast()：" + linkedHashSet.getLast());
        System.out.println("sortedSet.getLast()：" + sortedSet.getLast());

        Consumer<SequencedCollection> reversedPrint = sequencedCollection -> {
            sequencedCollection.reversed().forEach(x -> System.out.print(x + "  "));
            System.out.println();
        };

        System.out.println("=========reversed 输出==============");
        reversedPrint.accept(list);
        reversedPrint.accept(deque);
        reversedPrint.accept(linkedHashSet);
        reversedPrint.accept(sortedSet);
    }

 =========输出第一个元素==============
list.getFirst()：死磕Java
deque.getFirst()：死磕Java
linkedHashSet.getFirst()：死磕Java
sortedSet.getFirst()：死磕Java
=========输出最后一个元素==============
list.getLast()：死磕Redis
deque.getLast()：死磕Redis
linkedHashSet.getLast()：死磕Redis
sortedSet.getLast()：死磕Spring
=========reversed 输出==============
死磕Redis  死磕NIO  死磕Spring  死磕Netty  死磕Java新特性  死磕Java  
死磕Redis  死磕NIO  死磕Spring  死磕Netty  死磕Java新特性  死磕Java  
死磕Redis  死磕NIO  死磕Spring  死磕Netty  死磕Java新特性  死磕Java  
死磕Spring  死磕Redis  死磕Netty  死磕NIO  死磕Java新特性  死磕Java 


```

*   SequencedMap

```
    @Test
    public void sequencedMapTest() {
        LinkedHashMap<String, String> linkedHashMap = new LinkedHashMap<>();
        linkedHashMap.put("Java","死磕 Java 新特性");
        linkedHashMap.put("Netty","死磕 Netty");
        linkedHashMap.put("Spring","死磕 Spring");
        linkedHashMap.put("Redis","死磕 Redis");

        Consumer<SequencedMap> consumer = sequencedMap -> {
            sequencedMap.forEach((k,v) -> System.out.print(k + "--" + v + ";;"));
            System.out.println();
        };
        consumer.accept(linkedHashMap);

        System.out.println("=========putFirst=========");
        linkedHashMap.putFirst("Redis","死磕 Redis-01");
        consumer.accept(linkedHashMap);

        System.out.println("=========putLast=========");
        linkedHashMap.putLast("Java","死磕 Java 新特性-01");
        consumer.accept(linkedHashMap);

        System.out.println(linkedHashMap.sequencedKeySet());
        System.out.println(linkedHashMap.sequencedValues());
    }

Java--死磕 Java 新特性;;Netty--死磕 Netty;;Spring--死磕 Spring;;Redis--死磕 Redis;;
=========putFirst=========
Redis--死磕 Redis-01;;Java--死磕 Java 新特性;;Netty--死磕 Netty;;Spring--死磕 Spring;;
=========putLast=========
Redis--死磕 Redis-01;;Netty--死磕 Netty;;Spring--死磕 Spring;;Java--死磕 Java 新特性-01;;
[Redis, Netty, Spring, Java]
[死磕 Redis-01, 死磕 Netty, 死磕 Spring, 死磕 Java 新特性-01]



```
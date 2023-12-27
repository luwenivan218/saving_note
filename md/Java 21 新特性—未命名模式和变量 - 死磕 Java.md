> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1961861176)

> 该特性非常简单，就一句话：如果一个变量你用不到，那就用_代替它吧。这个特性的主要目的是提高代码的可读性和可维护性。try{}catch(Exceptione){thrownewRuntimeExcep

原

 2023-12-23  阅读 (39)  点赞 (0)

该特性非常简单，就一句话：**如果一个变量你用不到，那就用 ****`_`**** 代替它吧**。这个特性的主要目的是提高代码的可读性和可维护性。

```
try {

} catch (Exception e) {
  throw new RuntimeException("系统蹦啦！！！！");
}


```

这段代码熟悉吧。这里的 e 就是一个没有使用的变量，按照这个特性，我们可以用 `_` 代替。

```
try {

} catch (Exception _) {
  throw new RuntimeException("系统蹦啦！！！！");
}


```

这个特性使用的场景还是比较多的，比如 for 循环：

```
List<String> list = List.of("死磕Java","死磕Java新特性","死磕Netty","死磕Spring","死磕NIO","死磕Redis");
int skCount = 0;
for (String _ : list) {
  skCount++;
}
System.out.println(skCount);


```

你甚至可以这样写：

```
int _ = 1;
String _ = "死磕Java";
var _ = "死磕 Java 并发";


```

这个，就我个人而言，我觉得还是比较别扭，个人还是比较喜欢那种清晰明了的方式。
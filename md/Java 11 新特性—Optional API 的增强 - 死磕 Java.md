> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1414447929)

> Optional 是在 Java8 中引入用于处理可能为 null 的对象。它提供了一种更优雅的方法来减少 NullPointerException 的可能性。为了我们能够方便地使用 Optional，Java11 对

原

 2023-11-28  阅读 (69)  点赞 (0)

© 版权

版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。 本文链接：[https://www.skjava.com/series/article/1414447929](https://www.skjava.com/series/article/1414447929)

Optional 是在 Java 8 中引入用于处理可能为`null`的对象。它提供了一种更优雅的方法来减少`NullPointerException`的可能性。为了我们能够方便地使用 Optional，Java 11 对它进行了增强，主要是新增了一个 API：

<table><thead><tr><th>方法</th><th>描述</th></tr></thead><tbody><tr><td><code>isEmpty()</code></td><td>判断容器是否为空，如果包含的值不存在，则返回 true。</td></tr></tbody></table>

> `isEmpty()`

如果 Optional 对象为空，`isEmpty()` 返回 true，它是 `isPresent()` 的对立面。

```
Optional<String> optional = Optional.empty();
if (optional.isEmpty()) {
    System.out.println("Optional is empty");
}


```

当我们需要检查 Optional 是否不包含任何值时，这个方法是非常有用的。它提供了一种更直观的方式来表达这个检查，特别是在不包含值的情况需要特别处理时。

文章标签：
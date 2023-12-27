> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/9457186945)

> String 绝对是 Java 中最常用的一个类了，String 类的方法使用率也都非常的高，Java11 为了更好地处理字符串，引入了几个新的 API：方法名描述 isBlank() 检查字符串是否为空或仅包含空白

原

 2023-11-27  阅读 (94)  点赞 (1)

String 绝对是 Java 中最常用的一个类了，String 类的方法使用率也都非常的高，Java 11 为了更好地处理字符串，引入了几个新的 API：

<table><thead><tr><th>方法名</th><th>描述</th></tr></thead><tbody><tr><td><code>isBlank()</code></td><td>检查字符串是否为空或仅包含空白字符</td></tr><tr><td><code>lines()</code></td><td>分割获取字符串流（Stream）</td></tr><tr><td><code>strip()</code></td><td>去除字符串首尾的空白字符</td></tr><tr><td><code>repeat(n)</code></td><td>复制字符串</td></tr></tbody></table>

> `isBlank()`

用于判断字符串是否为空或者只包含空白字符。

```
"".isBlank();   
" ".isBlank();   
" \n\t ".isBlank()   
"skjava.com".isBlank();  


```

该方法是对`isEmpty()`的补充，`isEmpty()` 只能检查字符串长度是否为 0，而`isBlank()`则回进一步检查了字符串的内容是否也只包含空白字符。

> `lines()`

该方法返回一个流（Stream），该流由字符串中的行组成，使用行结束符作为分隔符。

```
"死磕 Java\n死磕 Java 新特性\n死磕 Netty".lines().forEach(System.out::println);

死磕 Java
死磕 Java 新特性
死磕 Netty



```

该方法可以处理不同平台上的换行符，无论是`\n`，`\r`还是`\r\n`。

> `strip()`

去除字符串首尾的空白字符。如果字符串中间有空白字符，它们不会被去掉。

```
System.out.println(" 死磕 Java  ".strip());

死磕 Java



```

在原来老的 Java 版本中有一个方法：`trim()`，该方法也可以去掉空格，但是它只能去掉除半角空格。看下面例子

```
System.out.println("[" + "　死磕 Java 新特性　".strip() + "]");   
System.out.println("[" + "　死磕 Java 新特性　".trim() + "]");    


System.out.println("[" + " 死磕 Java 新特性 ".strip() + "]");    
System.out.println("[" + " 死磕 Java 新特性 ".trim() + "]");     


```

所以：

*   `trim()` 只移除 ASCII 字符集中定义的空白字符。
*   `strip()` 移除所有 Unicode 字符集中定义的空白字符。

strip() 还有两个方法：

*   `stripLeading()`：仅移除开头的空白字符。
*   `stripTrailing()`：仅移除末尾的空白字符。

这里就不做介绍了。

> `repeat()`

将字符串重复指定的次数，并将这些重复的字符串连接为一个新的字符串。

```
System.out.println("死磕 Java 新特性，".repeat(3));

死磕 Java 新特性，死磕 Java 新特性，死磕 Java 新特性，



```

*   如果传入的次数为 0，结果是一个空字符串。
*   如果次数小于 0，会抛出`IllegalArgumentException`。
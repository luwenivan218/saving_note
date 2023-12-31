> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/7430445828)

> 在 Java13 之前，我们有时候需要写多行字符串，比如拼接 HTML、XML 之类的，在处理这种字符串时我们往往需要通过使用 \ n 实现换行，以及使用 + 符号连接多行字符串，例如：Stringhtml="

原

 2023-12-18  阅读 (46)  点赞 (0)

在 Java 13 之前，我们有时候需要写多行字符串，比如拼接 HTML、XML 之类的，在处理这种字符串时我们往往需要通过使用`\n`实现换行，以及使用`+`符号连接多行字符串，例如：

```
        String html = "<html>" +
                "<body>" +
                "<p>skjava.com</p>" +
                "</body>" +
                "</html>";


```

这种方式不仅使代码难以阅读，也增加了维护的难度。而且如果字符串中包括了特殊字符，还需要转义，处理起来非常麻烦。

所以为了解决多行字符串处理的复杂性，提高代码的清晰度和开发效率，Java 13 引入文本块。

> 文本块是 Java 中的一个新的字符串字面量，它通过使用三个双引号（`"""`）来标记字符串的开始和结束，允许字符串跨越多行而无需显式的换行符或字符串连接。

例如上面的例子使用文本块可以写成：

```
String html = """
              <html>
                  <body>
                      <p>skjava.com</p>
                  </body>
              </html>
              """;



```

这样写就比上面的那种拼接的方式更加直观和简洁了。

使用文本块具备如下几个优势：

1.  **多行字符串的简化**：在以往，编写多行字符串时，需要通过使用`\n`来实现换行，或者通过`+`来连接多个字符串。而使用文本块则让他们变得非常简单。
2.  **自动格式化和缩进处理**：采用自动拼接的方式，是无法格式化和缩进处理的，而使用文本块则会自动处理字符串的格式化和缩进，它是基于字符串的起始位置来确定缩进级别。这就意味着代码的可读性得到了极大的提升，同时也保持了字符串内容的预期格式。
3.  **方便的处理特殊字符**：在以前对于特殊字符我们是需要进行转义处理的，但使用文本块后，就不再需要对字符串中的特殊字符进行转义了。

所以，文本块特别适合处理多行文本数据，极大地简化了多行字符串的处理。
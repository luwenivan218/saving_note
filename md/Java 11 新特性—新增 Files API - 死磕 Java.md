> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/8589000958)

> Java11 对 java.nio.file.Files 新增了三个 API：readString()：读取文件内容 writeString()：写入文件内容 isSameFile()：比较两个路径是否指向文件系

原

 2023-11-27  阅读 (83)  点赞 (0)

Java 11 对`java.nio.file.Files`新增了三个 API：

*   `readString()`：读取文件内容
*   `writeString()`：写入文件内容
*   `isSameFile()`：比较两个路径是否指向文件系统中的同一个文件

这三个 API 简化了我们对文件的操作，使得操作文件变得更加简便了。

> `readString()`

该方法可以一次性读取文件的全部内容，避免了以前我们使用 BufferedReader 类的繁琐过程。

```
    @Test
    public void readStringTest() throws IOException {
        Path  path = Paths.get("skjava.txt");
        String content = Files.readString(path);
        System.out.println(content);
    }


```

> `writeString()`

该方法用于将字符串内容直接写入文件中，避免了以前我们使用 BufferedWriter 类的繁琐过程。

```
    @Test
    public void writeStringTest() throws IOException {
        Path  path = Paths.get("skjava.txt");
        String content = "www.skjava.com 是大明哥的个人网站，收录了目前最新最全的 Java 系列文章。【死磕 Java】是大明哥打造的进阶类 Java 系列文章，从入门、进阶、实战、源码方方面面阐述 Java 核心原理，助你提升 Java 技术一臂之力。"
        Files.writeString(path,content);
    }


```

> `isSameFile()`

该方法用于比较两个路径是否指向文件系统中的同一个文件，可以用来检查符号链接或文件快捷方式是否指向同一个文件。

```
    @Test
    public void isSameFileTest() throws IOException {
        Path path1 = Paths.get("skjava_01.txt");
        Path path2 = Paths.get("skjava_02.txt");
        boolean result = Files.isSameFile(path1,path2);
        System.out.println(result);
    }


```

文件匹配返回 true，否则返回 false。
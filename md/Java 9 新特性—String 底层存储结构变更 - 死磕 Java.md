> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/3621103566)

> 大明哥相信绝大数小伙伴一定看过 Java8 的 String 源码，对于它的底层存储结构一定不陌生，在 Java9 之前，String 的底层存储结构都是 char[]：publicfinalclassStringi

原

 2023-11-19  阅读 (72)  点赞 (0)

大明哥相信绝大数小伙伴一定看过 Java 8 的 `String` 源码，对于它的底层存储结构一定不陌生，在 Java 9 之前，`String` 的底层存储结构都是 `char[]`：

```
public final class String
     implements java.io.Serializable, Comparable<String>, CharSequence {

  
  private final char value[];

}


```

每个 `char` 都以 2 个字节存储在内存中。然而 Oracle 的 JDK 开发人员调研了成千上万个应用程序的 `heap dump` 信息，他们注意到大多数字符串都是以 `Latin-1` 字符编码表示的，它只需要一个字节存储就够了，两个字节完全是浪费，这比 `char` 数据类型存储少 50%（1 个字节）。

所以，在 Java 9 中将 `String` 的底层存储结构调整为 `byte[]`:

```
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence,
               Constable, ConstantDesc {

    @Stable
    private final byte[] value;

    private final byte coder;
}


```

Java 9 这样调整的目的是减小字符串的内存占用，这样带来了两个好处：

1.  **节省内存**：对于包含大量`ASCII`字符的字符串，内存占用大幅减少，因为每个字符只占用一个字节而不是两个字节。
2.  **提高性能**：由于字符串的存储结构与编码方式更加紧凑，字符串操作的性能也有所提高。

需要注意的是，这仅仅只是底层数据结构的变化，对于我们上层调用者完全是透明的，不会有任何影响，`String` 的方法以前怎么使用，现在还是怎么使用，例如：

```
public class StringTest {
    public static void main(String[] args) {
        String skString1 = "skjava";
        String skString2 = "死磕Java";
        System.out.println(skString1.charAt(0));
        System.out.println(skString2.charAt(0));
    }
}

s
死



```

`charAt()` 源码如下：

```
    public char charAt(int index) {
        if (isLatin1()) {
            return StringLatin1.charAt(value, index);
        } else {
            return StringUTF16.charAt(value, index); 
        }
    }


```

`isLatin1()` 用于判断编码格式是否为 `Latin-1` 字符编码，如果是则调用 `StringLatin1.charAt()`，否则调用 `StringUTF16.charAt()`。这里为什么要判断字符编码呢？`Latin-1` 字符编码也称 `ISO 8859-1`，它包括了拉丁字母（包括西欧、北欧和南欧语言的字母）以及一些常见的符号和特殊字符，但是它并不支持其他非拉丁字母的语言，例如希腊语、俄语或中文，对于这些我们只能使用其他字符编码了。

在 Java 9 中，`String` 支持的字符编码格式有两种：

1.  `Latin-1`：`Latin-1` 编码用于存储只包含拉丁字符的字符串。它采用了一字节编码，每个字符占用一个字节（8 位）。
2.  `UTF-16`：`UTF-16` 编码用于存储包含非拉丁字符的字符串，以及当字符串包含不适合 `Latin-1` 编码的字符时。

在 Java 9 中，`String` 多了一个成员变量 `coder`，它代表编码的格式，0 表示 `Latin-1` ，1 表示 `UTF-16`，我们在看 `skString1` 和 `skString2`：

![](https://sike.skjava.com/java-features/202310312000001.jpg)

从这张图可以清晰地看到 `“skjava”` 的字符编码是 `Latin-1`，而 `“死磕Java”` 的字符编码则是 `UTF-16`。不同的字符编码选择不同的方法来获取。其实你看下 String 里面的方法都是这种模式。

> **所以，Java 9 中的 String 使用 ****`Latin-1`**** 和 ****`UTF-16`**** 两种字符编码方式，根据字符串的内容来选择合适的编码格式，以便在内部存储时提高效率。**

但是，有小伙伴就喜欢硬扛，我就不喜欢 `Latin-1`，可以完全用`UTF-16` 么 ？可以。Java 满足你的一切不合理的要求。

> `-XX:-CompactStrings`**：禁用精简字符串特性**。

1.  如果启用 `Compact Strings`（默认情况），JVM 会根据字符串的内容来选择 `Latin-1` 还是 `UTF-16`，以在内存中有效地存储字符串，减小内存占用。
2.  如果禁用 `Compact Strings`（使用 `-XX:-CompactStrings`），JVM 将始终使用 `UTF-16` 编码来存储字符串。

一般来说，我们是不需要显式设置 `-XX:-CompactStrings`，开启 `Compact Strings` 能够帮组我们节约内存和提高性能。
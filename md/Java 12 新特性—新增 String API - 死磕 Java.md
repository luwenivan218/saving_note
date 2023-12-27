> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/6401260394)

> 为了进一步提高 String 的灵活度，使 String 的处理更加简单高效，Java12 引入三个方法对 StringAPI 进一步增强。indent(intn)transform(Function<?su

原

 2023-12-14  阅读 (51)  点赞 (0)

为了进一步提高 String 的灵活度，使 String 的处理更加简单高效，Java 12 引入三个方法对 String API 进一步增强。

*   `indent(int n)`
*   `transform(Function<? super String,​? extends R> f)`
*   `describeConstable()`和`resolveConstantDesc()`

> `indent(int n)`

该方法用于调整字符串的缩进。它根据指定的值增加或减少每行的空格数。如果 `n` 是正数，它会在每行前面添加相应数量的空格；如果 `n` 是负数，则减少缩进，直到某行的所有前导空格都被移除或达到指定的缩进级别。

```
    @Test
    public void indentTest() {
        String indented = "skjava.com".indent(3); 
        System.out.println(indented);
    }

   skjava.com



```

> `transform(Function<? super String,? extends R> f)`

该方法接受一个函数作为参数，对字符串进行转换，并返回转换的结果。该方法非常强大，强大之处在于它的泛型和功能性特点，它允许我们对字符串执行任意类型的操作并返回所需的类型。例如：

```
    @Test
    public void transformTest() {
        String str = "skjava.com";
        String str1 = str.transform(val -> val.indent(4));
        String str2 = str.transform(val -> {
            String val1 = "'" + val.indent(3) + "'";
            return val1.toUpperCase();
        });
        System.out.println(str1);
        System.out.println(str2);
    }


```

在 `Function` 函数里面，你可以干你任何想干的事情，该方法在对字符串执行复杂转换的场景中特别有用。

> `describeConstable()`与 `resolveConstantDesc()`

该方法要与常量 API 一起工作，提供了对常量的更好支持。方法返回一个描述字符串的 `Optional<String>`。这个 `Optional` 对象可能为空，也可能包含一个 String 实例，表示该字符串可以作为常量描述。如下：

```
    @Test
    public void describeConstableTest() {
        String str = "skjava.com";
        Optional<String> constable = str.describeConstable();

        if (constable.isPresent()) {
            System.out.println(str + "，可以作为字符串常量描述");
        } else {
            System.out.println(str + "，不可以作为字符串常量描述");;
        }
    }


```

`resolveConstantDesc()`，则与 `describeConstable()` 相关，用于解析常量描述。

这两个方法是比较底层的方法，主要主要是为了支持 Java 中的常量描述功能，在日常的 Java 应用程序开发中可能不是经常使用。
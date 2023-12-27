> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1940664105)

> 为了能够以更简洁的方式显示大数字，Java12 引入紧凑数字格式化（CompactNumberFormatting），这是对 NumberFormat 的一个补充。紧凑数字格式化，可以将十进制、货币或百分比

原

 2023-12-14  阅读 (47)  点赞 (0)

为了能够以更简洁的方式显示大数字，Java 12 引入紧凑数字格式化（`Compact Number Formatting`），这是对 NumberFormat 的一个补充。

紧凑数字格式化，可以将十进制、货币或百分比的长数字格式化为短格式，例如将 `1,000` 格式化为 `1K`，`1,000,000` 为 `1M` 等。这种表达的方式特别适合需要在有限空间内显示数字的场景，如图表、用户界面。

使用 NumberFormat 类的 `getCompactNumberInstance()` 就可以实现：

```
    @Test
    public void NumberFormatTest() throws IOException {
        NumberFormat compactFormat = NumberFormat.getCompactNumberInstance(Locale.US, NumberFormat.Style.SHORT);

        System.out.println(compactFormat.format(1000));
        System.out.println(compactFormat.format(10000));
        System.out.println(compactFormat.format(1000000));
        System.out.println(compactFormat.format(1000000000));
    }

1K
10K
1M
1B


```

`getCompactNumberInstance(Locale locale, NumberFormat.Style formatStyle)` 有两个参数：

*   `Locale`： 参数指定了数字格式化时使用的地区设置。不同的地区可能会以不同的方式缩写数字，比如美国，`10000` 表示为 `1K`，但是我国则表示为 `1万`。小伙伴们可以将上面例子的 Local 设置为 `Locale.CHINESE`，看看结果是什么。
*   `LONG`：定义了紧凑数字格式的样式。Java 12 提供了两种主要样式：`SHORT` 和 `LONG`。
    *   `SHORT`** **：提供了最简洁的数字表示形式。例如，1,000 可能被格式化为 “1K”，1,000,000 为 “1M”。这种样式适用于空间有限或需要快速传达数值大小的场景。
    *   `LONG`** **：提供了更详细的格式化方式。在这种样式下，数字可能会被表示为 “1 thousand” 或“1 million”。比如 `1000` 会被定义为`1 thousand`，`10000` 会被定义为 `10 thousand`
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1765935055)

> 通过前一篇文章（日期时间 API）我们知道如何在 Java8 中得到我们需要的日期和时间，但是有时候我们需要将日期和时间对象转换为字符串又或者将字符串解析为日期时间对象，这个时候我们需要用到 Java8 提供的

通过前一篇文章（[日期时间 API](https://www.skjava.com/series/article/1658730277)）我们知道如何在 Java 8 中得到我们需要的日期和时间，但是有时候我们需要将日期和时间对象转换为字符串又或者将字符串解析为日期时间对象，这个时候我们需要用到 Java 8 提供的日期时间格式化工具：DateTimeFormatter。

![](https://sike.skjava.com/java-features/202310101000002.png)

DateTimeFormatter
-----------------

DateTimeFormatter 用于格式化和解析日期和时间，它能够轻松地将日期时间对象转换为字符串以及将字符串解析为日期时间对象。而且它是不可变的，线程安全的。

### **创建 DateTimeFormatter**

DateTimeFormatter 提供了 `ofPattern()` 静态方法用于构建 DateTimeFormatter 对象：

```
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");


```

其他静态方法如下：

*   `ofPattern(String pattern, Locale locale)`：使用给定的模式和区域设置创建格式化器。
*   `ofLocalizedDate(FormatStyle dateStyle)`：创建具有当地特定日期格式的格式化器。FormatStyle 是一个枚举，其值可以是 FULL, LONG, MEDIUM, SHORT。
*   `ofLocalizedDateTime(FormatStyle dateTimeStyle)`：创建具有特定地区日期时间 (date-time) 格式的格式化器。
*   `ofLocalizedDateTime(FormatStyle dateStyle, FormatStyle timeStyle)`： 创建具有特定地区日期时间 (date-time) 格式的格式化器。我们需要为日期和时间分别传递 FormatStyle。例如，日期可以是 LONG，时间可以是 SHORT。
*   `ofLocalizedTime(FormatStyle timeStyle)`： 创建具有当地特定时间格式的格式化器。

我们也可以使用预定义的格式化器，DateTimeFormatter 提供了大量的预定义格式化器，如下：

<table><thead><tr><th>Formatter</th><th>Example</th></tr></thead><tbody><tr><td>BASIC_ISO_DATE</td><td>‘20181203’</td></tr><tr><td>ISO_LOCAL_DATE</td><td>‘2018-12-03’</td></tr><tr><td>ISO_OFFSET_DATE</td><td>‘2018-12-03+01:00’</td></tr><tr><td>ISO_DATE</td><td>‘2018-12-03+01:00’; ‘2018-12-03’</td></tr><tr><td>ISO_LOCAL_TIME</td><td>‘11:15:30’</td></tr><tr><td>ISO_OFFSET_TIME</td><td>‘11:15:30+01:00’</td></tr><tr><td>ISO_TIME</td><td>‘11:15:30+01:00’; ‘11:15:30’</td></tr><tr><td>ISO_LOCAL_DATE_TIME</td><td>‘2018-12-03T11:15:30’</td></tr><tr><td>ISO_OFFSET_DATE_TIME</td><td>‘2018-12-03T11:15:30+01:00’</td></tr><tr><td>ISO_ZONED_DATE_TIME</td><td>‘2018-12-03T11:15:30+01:00[Europe/Paris]’</td></tr><tr><td>ISO_DATE_TIME</td><td>‘2018-12-03T11:15:30+01:00[Europe/Paris]’</td></tr><tr><td>ISO_ORDINAL_DATE</td><td>‘2018-337’</td></tr><tr><td>ISO_WEEK_DATE</td><td>‘2018-W48-6’</td></tr><tr><td>ISO_INSTANT</td><td>‘2018-12-03T11:15:30Z’</td></tr><tr><td>RFC_1123_DATE_TIME</td><td>‘Tue, 3 Jun 2018 11:05:30 GMT’</td></tr></tbody></table>

例如，我们使用 ISO_DATE_TIME 格式化一个日期时间：

```
DateTimeFormatter formatter = DateTimeFormatter.ISO_DATE_TIME;
String dateTime = formatter.format(LocalDateTime.now()); 


```

如果我们需要的不再是简单的 yyyy-MM-dd 格式的日期，而是更加复杂的日期格式，我们可以使用 DateTimeFormatterBuilder 来构建更加复杂的日期格式。

### 格式转换

#### **将日期时间对象转换为字符串**

为了格式化一个日期、时间或日期时间，DateTimeFormatter 提供了两个 format 方法：

*   `format(TemporalAccessor temporal)`：将日期时间对象格式化为字符串，返回的是一个字符串对象。

```
DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
System.out.println(dateTimeFormatter.format(LocalDateTime.now()));   



```

*   `formatTo(TemporalAccessor temporal, Appendable appendable)`：将日期时间对象格式化，但是它没有返回值，而是将结果附加到给定的 Appendable 对象中。Appendable 对象可以是 StringBuffer、StringBuilder 等的实例，这样可以提供性能，因为它避免创建了不必要的字符串对象。

```
DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
StringBuilder stringBuilder = new StringBuilder();
dateTimeFormatter.formatTo(LocalDateTime.now(),stringBuilder);
System.out.println(stringBuilder.toString());                      


```

也可以将日期时间格式的字符串转换为 `LocalDateTime`、`ZonedDateTime` 等日期时间对象：

```
DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
System.out.println(LocalDateTime.parse("2023-10-09 22:12:55",dateTimeFormatter));  


```

#### **将字符串解析为日期时间对象**

DateTimeFormatter 用于将字符串解析为日期时间对象的方法主要是 `parse()`，但是它返回的是 TemporalAccessor，我们需要再次将 TemporalAccessor 进一步转换：

```
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy/MM/dd");
TemporalAccessor temporalAccessor = formatter.parse("2023-10-10");
LocalDate localDate = LocalDate.from(temporalAccessor);


```

有点儿繁琐，所以如果我们明确了要解析的类型，可以直接使用 来将字符串解析为指定的日期时间对象：

```
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy/MM/dd");
LocalDate localDate = formatter.parse("2023-10-10", LocalDate::from);


```

或者直接使用具体的日期时间对象的 parse() 来转换：

```
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy/MM/dd");
LocalDate localDate = LocalDate.parse("2023-10-10",formatter);


```

DateTimeFormatterBuilder
------------------------

DateTimeFormatter 虽然能够帮忙我们格式化标准的日期时间，但是有时候我们需要更加复杂的日期时间格式，比如这种格式：`Day is:17, month is:10, and year:2014 with the time:23:35` ，你使用 DateTimeFormatter 是会报错的：

```
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("Day is:dd, month is:MM, and year:yyyy with the time:HH:mm");
String dateTime = formatter.format(LocalDateTime.now());


```

报错如下：

![](https://sike.skjava.com/java-features/202310101000001.jpg)

碰到这种比较灵活，个性化的日期格式时，DateTimeFormatter 无法处理或者处理起来比较麻烦时，我们可以使用更加强大灵活的 DateTimeFormatterBuilder。

DateTimeFormatterBuilder 是 Java 中用于构建自定义日期时间的格式化神器。它允许我们按照自己的需求来构建日期和时间，包括自定义的日期和时间分隔符时区、日期元素的顺序等等。

DateTimeFormatter 中的预定义格式化器就是采用 DateTimeFormatterBuilder 来构建的，例如 ISO_DATE_TIME：

```
    public static final DateTimeFormatter ISO_DATE_TIME;
    static {
        ISO_DATE_TIME = new DateTimeFormatterBuilder()
                .append(ISO_LOCAL_DATE_TIME)
                .optionalStart()
                .appendOffsetId()
                .optionalStart()
                .appendLiteral('[')
                .parseCaseSensitive()
                .appendZoneRegionId()
                .appendLiteral(']')
                .toFormatter(ResolverStyle.STRICT, IsoChronology.INSTANCE);
    }


```

当我们通过构造函数构建出一个 DateTimeFormatterBuilder 实例对象后，就可以通过它的实例方法来添加不同的元素构造成我们的格式化器。常见的方法有：

<table><thead><tr><th>方法</th><th>描述</th></tr></thead><tbody><tr><td><code>appendPattern(String pattern)</code></td><td>根据传入的模式字符串添加格式化元素。</td></tr><tr><td><code>appendLiteral(char literal)</code></td><td>添加字面文字。</td></tr><tr><td><code>appendValue(TemporalField field, int width)</code></td><td>添加字段值。</td></tr><tr><td><code>appendText(TemporalField field)</code></td><td>添加字段的文本表示。</td></tr><tr><td><code>appendZoneId()</code></td><td>添加时区标识。</td></tr><tr><td><code>appendOffset(String pattern, String noOffsetText)</code></td><td>添加偏移量信息。</td></tr></tbody></table>

示例：

```
builder.appendPattern("yyyy-MM-dd HH:mm:ss");
builder.appendLiteral('T');
builder.appendValue(ChronoField.HOUR_OF_DAY, 2);
builder.appendLiteral(':');
builder.appendValue(ChronoField.MINUTE_OF_HOUR, 2);


```

当我们构建完毕后，使用 `toFormatter()` 方法就可以将其转换为 DateTimeFormatter。下面我们就用 DateTimeFormatterBuilder 来完成上面那个示例。

`Day is:` 这种是字面文字，所以使用 `appendLiteral(char literal)` 直接添加就可以了。年月日这样的实际值我们需要 `appendValue(TemporalField field, int width)` 来添加，该方法中的 `width` 表示字面的宽度，即显示字段的位数，比如 9 月份，如果 width 为 2 ，则显示为 09，如果不设置的话则显示为 9，代码如下：

```
    @Test
    public void test() {
        DateTimeFormatter formatter = new DateTimeFormatterBuilder().appendLiteral("Day is:")
                .appendValue(ChronoField.DAY_OF_MONTH,2)
                .appendLiteral(", month is:")
                .appendValue(ChronoField.MONTH_OF_YEAR,2)
                .appendLiteral(", and year:")
                .appendValue(ChronoField.YEAR,4)
                .appendLiteral(" with the time:")
                .appendValue(ChronoField.HOUR_OF_DAY)
                .appendLiteral(":")
                .appendValue(ChronoField.MINUTE_OF_HOUR)
                .toFormatter();
        LocalDateTime dateTime = LocalDateTime.of(2023,9,13,22,34,25,0);
        String str = dateTime.format(formatter);
        System.out.println(str);
    }

Day is:13, month is:09, and year:2023 with the time:22:34



```

是不是很完美地呈现出来了？

DateTimeFormatterBuilder 是一个非常灵活的格式化工具，我们可以利用它创建各种各样的自定义日期时间格式化器，以满足我们的特定需求。
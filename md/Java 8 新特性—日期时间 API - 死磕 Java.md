> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1658730277)

> 为什么 Java8 要重新设计日期时间 API 作为 Java 开发者你一定直接或者间接使用过 java.util.Date、java.util.Calendar、java.text.SimpleDateForma

![](https://sike.skjava.com/java-features/202310091000001.png)

为什么 Java 8 要重新设计日期时间 API
------------------------

作为 Java 开发者你一定直接或者间接使用过 `java.util.Date` 、`java.util.Calendar`、`java.text.SimpleDateFormat` 这三个类吧，这三个类是 Java 用于处理日期、日历、日期时间格式化的。由于他们存在一些问题，诸如：

1.  **线程不安全**：
    1.  `java.util.Date` 和 `java.util.Calendar` 线程不安全，这就导致我们在多线程环境使用需要额外注意。
    2.  `java.text.SimpleDateFormat` 也是线程不安全的，这可能导致性能问题和日期格式化错误。而且它的模式字符串容易出错，且不够直观。
2.  **可变性**：`java.util.Date`类是可变的，这意味着我们可以随时修改它，如果一不小心就会导致数据不一致问题。
3.  **时区处理困难**：Java 8 版本以前的日期 API 在时区处理上存在问题，例如时区转换和夏令时处理不够灵活和准确。而且时区信息在 `Date` 对象中存储不明确，这使得正确处理时区变得复杂。
4.  **设计不佳**：
    1.  日期和日期格式化分布在多个包中。
    2.  `java.util.Date` 的默认日期，年竟然是从 1900 开始，月从 1 开始，日从 1 开始，没有统一性。而且 `java.util.Date` 类也缺少直接操作日期的相关方法。
    3.  日期和时间处理通常需要大量的样板代码，使得代码变得冗长和难以维护。

基于上述原因，Java 8 重新设计了日期时间 API，以提供更好的性能、可读性和可用性，同时解决了这些问题，使得在 Java 中处理日期和时间变得更加方便和可靠。相比 Java 8 之前的版本，Java 8 版本的日期时间 API 具有如下几个优点：

1.  **不可变性（Immutability）**：Java 8 的日期时间类（如`LocalDate`、`LocalTime`和`LocalDateTime`）都是不可变的，一旦创建就不能被修改。这确保了**线程安全**，避免了并发问题。
2.  **清晰的 API 设计**：Java 8 的日期时间 API 采用了更清晰、更一致的设计，相比于以前版本的 `Date` 和 `Calendar` 更易于理解和使用。而且它们还提供了丰富的方法来执行日期和时间的各种操作，如加减、比较、格式化等。
3.  **本地化支持**：Java 8 的日期时间 API 支持本地化，可以轻松处理不同地区和语言的日期和时间格式。它们能够自动适应不同的时区和夏令时规则。
4.  **新的时区处理**：Java 8 引入了 `ZoneId` 和 `ZoneOffset` 等新的时区类，使时区处理更加精确和灵活。这有助于解决以前版本中时区处理的问题。
5.  **新的格式化 API**：Java 8 引入了 `DateTimeFormatter` 类，用于格式化和解析日期和时间，支持自定义格式和本地化。这提供了更强大和灵活的格式化选项。
6.  **更好的性能**：Java 8 的日期时间 API 比以前的 API 性能更佳。

Instant：时间点
-----------

Instant 用于表示时间线上的点，即一个瞬间。它是不可变的，以纳秒为单位精确表示时间，可以用于在不考虑时区的情况下进行时间的计算和比较。

Instant 参考点是标准的 Java 纪元（epoch），即`1970-01-01T00：00：00Z`（1970 年 1 月 1 日 00:00 GMT）。 Instant 类的 EPOCH 属性返回表示 Java 纪元的 Instant 实例。 在纪元之后的时间是正值，而在此之前的时间即是负值。

*   使用 `Instant.now()` 创建当前的时间点：

```
Instant now = Instant.now();


```

`getEpochSecond()` 返回自纪元以来经过的秒数。 `getNano()` 返回自上一秒开始以来的纳秒数。

*   从 `java.util.Date` 或 `java.util.Calendar` 转换为 `Instant`

```
Instant instant = new Date().toInstant();


```

Instant 在以下场景特别有用：

1.  计算事件发生的时间戳，无论时区如何。
2.  进行时间计算，如计算时间差、定时任务等。

例如：

```
public class InstantTest {

    @test
    public void test() {

        Instant start = Instant.now();

        

        Instant end = Instant.now();

        System.out.println(Duration.between(start, end).toMillis());
    }

}


```

LocalDate ：本地日期
---------------

> LocalDate 用于表示不包含时间信息的日期。它是不可变的。

### 创建 LocalDate

Java 提供了三种方式用来创建一个 LocalDate 对象。

*   使用`LocalDate.now()`方法创建当前日期

```
LocalDate currentDate = LocalDate.now();


```

*   使用 LocalDate.of() 来创建指定年、月、日的 LocalDate 对象

```
LocalDate date = LocalDate.of(2023, 10, 8)


```

*   使用 DateTimeFormatter 解析一个 LocalDate 对象

```
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
LocalDate parsedDate = LocalDate.parse("2023-10-08", formatter);


```

### 日期计算

LocalDate 提供了 plus 和 minus 类方法用于在日期上增加或者减少一定数量的年、月、日：

*   `plusYears()`、`plusMonths()` 和 `plusDays()`：分别用于在日期上增加年、月和日。
*   `minusYears()`、`minusMonths()` 和 `minusDays()`：分别用于从日期中减去年、月和日。

这 6 个方法都是返回一个新的 LocalDate 对象，原始 LocalDate 对象不受影响。

```
    @test
    public void test() {
        LocalDate localDate = LocalDate.now();
        LocalDate plusYears = localDate.plusYears(1);
        LocalDate plusMonths = localDate.plusMonths(1);
        LocalDate plusDays = localDate.plusDays(1);

        LocalDate minusYears = localDate.minusYears(1);
        LocalDate minusMonths = localDate.minusMonths(1);
        LocalDate minusDays = localDate.minusDays(1);

        System.out.println("原始 LocalDate：" + localDate);
        System.out.println("plusYears(1)：" + plusYears);
        System.out.println("plusMonths(1)：" + plusMonths);
        System.out.println("plusDays(1)：" + plusDays);
        System.out.println("minusYears(1)：" + minusYears);
        System.out.println("minusMonths(1)：" + minusMonths);
        System.out.println("minusDays(1)：" + minusDays);
    }
    

原始 LocalDate：2023-10-08
plusYears(1)：2024-10-08
plusMonths(1)：2023-11-08
plusDays(1)：2023-10-09
minusYears(1)：2022-10-08
minusMonths(1)：2023-09-08
minusDays(1)：2023-10-07


```

### 获取日期信息

LocalDate 提供了 get 类方法用于获取日期信息：

*   `getYear()`：获取年份。
*   `getMonth()`：获取月份（返回`Month`枚举类型）。
*   `getDayOfMonth()`：获取月中的天数。
*   `getDayOfWeek()`：获取星期几（返回`DayOfWeek`枚举类型）。

```
    @Test
    public void test() {
        LocalDate localDate = LocalDate.now();
        System.out.println("getYear()：" + localDate.getYear());
        System.out.println("getMonthValue()：" + localDate.getMonthValue());
        System.out.println("getDayOfMonth()：" + localDate.getDayOfMonth());
        System.out.println("getDayOfWeek()：" + localDate.getDayOfWeek());
    }

getYear()：2023
getMonthValue()：10
getDayOfMonth()：8
getDayOfWeek()：SUNDAY



```

### 修改日期

LocalDate 提供了 with 类方法用于修改 LocalDate 对象，它返回的也是一个新的 LocalDate 对象。

*   `withDayOfMonth()`：修改月中的天数字段
*   `withMonth()`：修改月份字段
*   `withYear()`：修改年份字段

```
    @Test
    public void test() {
        LocalDate localDate = LocalDate.now();
        LocalDate withYear = localDate.withYear(2022);
        LocalDate withMonth = localDate.withMonth(12);
        LocalDate withDayOfMonth = localDate.withDayOfMonth(22);

        System.out.println("原始 localDate：" + localDate);
        System.out.println("withYear(2022)：" + withYear);
        System.out.println("withMonth(12)：" + withMonth);
        System.out.println("withDayOfMonth(22)：" + withDayOfMonth);
    }

原始 localDate：2023-10-08
withYear(2022)：2022-10-08
withMonth(12)：2023-12-08
withDayOfMonth(22)：2023-10-22



```

设置的值的时候注意时间范围，你别 `withDayOfMonth(99)` 肯定报异常。

Period：LocalDate 的距离
--------------------

Period 是用于处理日期间隔的类，通常用于计算两个日期之间的间隔，如天数、月数和年数。

`Period.of()` 用于创建一个表示日期间隔的 Period 对象，该方法接受三个参数：年、月、日：

```
Period period = Period.of(5,10,8);


```

同时，Period 也提供了对应的 get 方法用于获取间隔的年、月、日，对应的方法分别为 `getYears()`、`getMonths()`、`getDays()`：

```
    @Test
    public void test() {
        Period period = Period.of(5,10,8);
        int years = period.getYears();
        int months = period.getMonths();
        int days = period.getDays();
    }


```

Period 提供的 `between()` 用于计算两个日期之间的间隔：

```
    @Test
    public void test() {
        LocalDate begin = LocalDate.of(2023,10,8);
        LocalDate end = LocalDate.of(2025,9,12);
        
        Period period = Period.between(begin,end);
        System.out.println("years：" + period.getYears());
        System.out.println("getMonths：" + period.getMonths());
        System.out.println("getDays：" + period.getDays());
    }

years：1
getMonths：11
getDays：4



```

`Period.between()` 返回的是一个 Period 对象，我们可以利用对应的 get 方法获取相应的数据。

LocalTime 本地时间
--------------

LocalTime 用于不包含日期信息的时间，它只表示一天中的时间。

### 创建 LocalTime

LocalTime 提供了四种方式来创建 LocalTime 对象。

*   `LocalTime.now()`：获取当前系统时间。

```
LocalTime localTime = LocalTime.now();      


```

*   `LocalTime.of(int hour, int minute)`：创建指定的小时和分钟的时间

```
LocalTime localTime = LocalTime.of(12,46);    


```

*   `LocalTime.of(int hour, int minute, int second)`：创建指定的小时、分钟和秒的时间。

```
LocalTime localTime = LocalTime.of(12,46,50);     


```

*   `LocalTime.of(int hour, int minute, int second, int nanoOfSecond)`：创建指定的小时、分钟、秒和纳秒的时间。

```
LocalTime localTime = LocalTime.of(12,46,50,500000000);     


```

LocalTime 和 LocalDate 提供的方法都差不多，plus 增加时分秒、minus 减少时分秒，get 获取时分秒，with 修改时分秒，同时 LocalTime 也是不变的，所以也是线程安全的。

### 时间计算

LocalTime 提供了 plus 和 minus 类型方法用于对时间进行加减操作。

*   `plusHours()`、`plusMinutes()`、`plusSeconds()`：分别用于在时间上增加时、分、秒。
*   `minusHours()`、`minusMinutes()`、`minusSeconds()`：分别用于在时间上减少时、分、秒。

```
    @Test
    public void test() {
        LocalTime localTime = LocalTime.now();
        LocalTime plusHours = localTime.plusHours(1);
        LocalTime plusMinutes = localTime.plusMinutes(1);
        LocalTime plusSeconds = localTime.plusSeconds(1);
        LocalTime minusHours = localTime.minusHours(1);
        LocalTime minusMinutes = localTime.minusMinutes(1);
        LocalTime minusSeconds = localTime.minusSeconds(1);

        System.out.println("localTime：" + localTime);
        System.out.println("plusHours(1)：" + plusHours);
        System.out.println("plusMinutes(1)：" + plusMinutes);
        System.out.println("plusSeconds(1)：" + plusSeconds);
        System.out.println("minusHours(1)：" + minusHours);
        System.out.println("minusMinutes(1)：" + minusMinutes);
        System.out.println("minusSeconds(1)：" + minusSeconds);
    }

localTime：15:40:01.160
plusHours(1)：16:40:01.160
plusMinutes(1)：15:41:01.160
plusSeconds(1)：15:40:02.160
minusHours(1)：14:40:01.160
minusMinutes(1)：15:39:01.160
minusSeconds(1)：15:40:00.160 



```

### 获取时间信息

LocalTime 提供了 get 类方法用于获取时间相关的信息。

getHour()、getMinute()、getSecond()、getNano()：分别用于获取时间的时、分、秒、纳秒。

```
    @Test
    public void test() {
        LocalTime localTime = LocalTime.now();

        System.out.println("localTime：" + localTime);
        System.out.println("getHour()：" + localTime.getHour());
        System.out.println("getMinute()：" + localTime.getMinute());
        System.out.println("getSecond()：" + localTime.getSecond());
        System.out.println("getNano()：" + localTime.getNano());
    }

localTime：15:43:38.300
getHour()：15
getMinute()：43
getSecond()：38
getNano()：300000000



```

### 修改时间

LocalTime 提供了 with 类方法用于修改时分秒：

`withHour()`、`withMinute()`、`withSecond()` 分别用于修改时间上的时、分、秒。

```
    @Test
    public void test() {
        LocalTime localTime = LocalTime.now();

        System.out.println("withHour(22)：" + localTime.withHour(22));
        System.out.println("withMinute(22)：" + localTime.withMinute(22));
        System.out.println("withMinute(22)：" + localTime.withSecond(22));
        System.out.println("localTime：" + localTime);
    }

withHour(22)：22:49:41.755
withMinute(22)：15:22:41.755
withMinute(22)：15:49:22.755
localTime：15:49:41.755


```

LocalDateTime 本地日期时间
--------------------

在大部分时间我们不仅仅只是需要日期或者时间，而是日期和时间，这是时候我们就需要使用 LocalDateTime。LocalDateTime 是用于处理日期和时间的，它不包含时区信息，它提供了一种简单、便捷的方式处理日期和时间，可以代替旧的 `java.util.Date` 和 `java.util.Calendar` 类，并且提供更多的功能和灵活性。

LocalDateTime 是表示日期和时间的，包括年、月、日、小时、分钟、秒以及毫秒，但是不包括时区。它里面很多方法和 LocalDate 、LocalTime 一致，所以大明哥在这里就不过多介绍了。

*   `plus` 类：增加日期或时间
*   `minus` 类：减少日期或时间
*   `get` 类：获取日期或时间
*   `with` 类：修改日期或时间
*   `isBefore`、`isAfter`：比较两个 LocalDateTime 对象

ZonedDateTime 带时区的日期时间
----------------------

ZonedDateTime 是一个用于表示包含时区信息的日期和时间对象，它用于解决处理日期和时间时的时区问题，以便更好地支持全球化的时间操作。

有了一个 LocalDateTime 不够，为什么还需要在增加一个 ZonedDateTime 呢？要明白这个问题我们就需要先了解什么是时区，为什么需要它。

### 什么是时区

由于地球自转，不同经度上的地方会在不同时刻经历日出和日落，因此当你在一个地方看到太阳高悬在天空中时，在另一个地方可能是夜晚。当你 9 点起来上班时，别人可能刚刚吃完晚饭准备带老婆孩子去遛弯，所以我们需要创建时区机制，来保证能更合理地安排生产生活。

时区是地球上的不同区域，每个区域都使用统一的时间标准，使人们能够在不同地方协调时间。时区基于经度划分，地球被分成 24 个主要时区，每个时区大致相当于地球上的一个经度带。每个时区都有自己的标准时间，该时间是该时区内所有地方的参考时间。

### ZonedDateTime 的使用方法

ZonedDateTime 和 LocalDateTime 中的方法几乎都一样，知道 LocalDateTime 的方法含义就一定知道 ZonedDateTime 的方法含义。下面大明哥讲解下与 LocalDateTime 不一样的地方。

ZonedDateTime 也是提供了`now()` 和 `of()` 来创建 ZonedDateTime ：

```
ZonedDateTime zonedDateTime = ZonedDateTime.now();



```

使用 `of()` 需要指明时区 ZoneId，ZoneId 是 Java 中用于表示时区的类，目前 Java 8 中共包含 599 个时区。

```
Set<String> availableZoneId = ZoneId.getAvailableZoneIds();   

ZoneId GMT= ZoneId.of("GMT");



```

得到 ZoneId 对象后就可以利用 `of()` 来构建 ZonedDateTime 对象：

```
ZonedDateTime zonedDateTime = ZonedDateTime.of(2023, 10, 9, 21, 10, 0, 0, ZoneId.of("America/New_York"));



```

也可以将一个 LocalDateTime 转换为 ZonedDateTime：

```
LocalDateTime localDateTime = LocalDateTime.now();
ZonedDateTime zonedDateTime = ZonedDateTime.of(localDateTime,ZoneId.of("America/New_York"));


```

ZonedDateTime 也提供了时区转换的功能：

*   `withZoneSameInstant()`：保持时间点不变，但切换到另一个时区
*   `withZoneSameLocal()`：会保持本地时间不变，但根据新时区调整偏移量

```
ZonedDateTime newYorkTime = zonedDateTime.withZoneSameInstant(ZoneId.of("America/New_York"));
ZonedDateTime londonTime = zonedDateTime.withZoneSameInstant(ZoneId.of("Europe/London"));


```
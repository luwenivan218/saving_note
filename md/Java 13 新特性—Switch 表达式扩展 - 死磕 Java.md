> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/9120322209)

> Java12 引入 Switch 表达式作为预览特性，该特性与传统的 Switch 表达式相比，它允许将整个 Switch 结构作为一个表达式，直接返回值，而不是像老版本的 Switch 语句一样不支持返回值。同时它使

原

 2023-12-18  阅读 (37)  点赞 (0)

Java 12 引入 Switch 表达式作为预览特性，该特性与传统的 Switch 表达式相比，它允许将整个 Switch 结构作为一个表达式，直接返回值，而不是像老版本的 Switch 语句一样不支持返回值。同时它使用`->`箭头来代替传统的`":"`，使得代码更加简洁、易读，也避免了使用冗长的`break`语句，杜绝了 "**Fall-through**" 行为。详情参考文章：[Java 12 新特性—Switch 表达式](https://www.skjava.com/series/article/5782529848)

为了进一步扩展 Switch 的功能，Java 13 引入`yield`关键字来处理多分支结构中的返回值。

`yield`用于在 Switch 表达式的每个分支中返回一个值，这点与传统的 Switch 语句需要通过变量赋值来返回值不同。同时在使用`->`的情况下，如果分支逻辑比较复杂，需要多条语句来处理，那么可以在这些语句的最后使用`yield`来返回最终的结果。

在 Java 12 中，虽然他允许直接从 Switch 表达式中返回值，但在处理复杂的逻辑时，仍需依赖外部变量来返回结果。比如下面这段简单的逻辑：

```
    public static String getTypeOfDay(String day) {
        String typeOfDay = switch (day) {
            case "MONDAY", "TUESDAY", "WEDNESDAY", "THURSDAY", "FRIDAY" -> "Weekday";
            case "SATURDAY", "SUNDAY" -> "Weekend";
            default -> "Unknown";
        };
        return typeOfDay;
    }


```

对于这简单的逻辑 ，Java 12 可以直接返回，但是这里我想在 default 处增加一个判断 day 是否为空的逻辑，这个时候就无法简单使用 `->` 直接返回了，我们需要定义一个外部变量来处理：

```
    public static String getTypeOfDay(String day) {
        String typeOfDay = null;
        switch (day) {
            case "MONDAY", "TUESDAY", "WEDNESDAY", "THURSDAY", "FRIDAY" -> typeOfDay  = "Weekday";
            case "SATURDAY", "SUNDAY" -> typeOfDay = "Weekend";
            default -> {
                
                if (day.isEmpty()) {
                    typeOfDay =  "day is empty";
                } else {
                    typeOfDay = "Unknown";
                }
            }
        };
        return typeOfDay;
    }


```

但是，在 Java 13 中，我们可以使用 yield 在 Switch 表达式中直接返回：

```
    public static String getTypeOfDay(String day) {
        return switch (day) {
            case "MONDAY", "TUESDAY", "WEDNESDAY", "THURSDAY", "FRIDAY" ->  "Weekday";
            case "SATURDAY", "SUNDAY" ->  "Weekend";
            default -> {
                if (day.isEmpty()) {
                   yield "day is empty";
                } else {
                    yield "Unknown";
                }
            }
        };
    }


```

是不是简便了很多？

在 Switch 表达式中处理复杂的 case 逻辑时，`yield` 使得在每个分支内部可以执行多条语句，并返回值，从而简化了代码结构并提高了可读性。
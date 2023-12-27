> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/5782529848)

> Switch 语句，这是 Java 入门中控制流程的部分，它是除 if 语句外的另外一种条件判断，提供了比连续的 if-else 语句更加清晰和结构化的选择机制。大明哥相信学过 Java 的小伙伴没有不知道怎么使用它吧

原

 2023-12-14  阅读 (112)  点赞 (0)

Switch 语句，这是 Java 入门中控制流程的部分，它是除 if 语句外的另外一种条件判断，提供了比连续的 `if-else` 语句更加清晰和结构化的选择机制。大明哥相信学过 Java 的小伙伴没有不知道怎么使用它吧。

从最初的 Switch 语句仅支持整型数据类型（如 `byte`, `short`, `char`, 和 `int`），以及枚举类型（Java 5 引入）。到后面的 Java 7 开始支持 String 类型，虽然 Java 7 这一改进极大地增加了 Switch 语句的灵活性和实用性，但是 Switch 语句依然存在一些显著的缺陷：

1.  **"Fall-through" 行为**：在没有显式 `break` 语句的情况下，Switch 语句会从一个 case "穿透" 到下一个 case，忽略了这个会导致不可饶恕的错误。
2.  **代码冗余**：每个 case，我们都需要重复类似的代码结构，增加了代码的冗余和维护难度。

为了解决 Switch 的现存问题，Java 12 引入全新的 Switch 表达式，该表达式不仅增强了 Switch 语句的功能，还大幅提高了其灵活性和表达能力。

> 新的 Switch 表达式引入了 `->` 操作符，用于替代传统的冒号（`:`）。与传统的 Switch 语句不同，使用 `->` 的 case 分支不会出现 `"fall-through"` 现象，因此不需要 break 语句来防止穿透。这减少了代码的复杂性，也降低了编程错误的风险。

通过下面一个例子来说明新的 Switch 表达式和老的 Switch 之间的差异，从这些差异中就可以看出 Java 12 中的 Switch 表达式的增强不分。

*   **老的 Switch 语句**

```
    public static String getTypeOfDay(String day) {
        String typeOfDay;
        switch (day) {
            case "MONDAY":
            case "TUESDAY":
            case "WEDNESDAY":
            case "THURSDAY":
            case "FRIDAY":
                typeOfDay = "Weekday";
                break;
            case "SATURDAY":
            case "SUNDAY":
                typeOfDay = "Weekend";
                break;
            default:
                typeOfDay = "Unknown";
        }
        return typeOfDay;
    }


```

使用传统的 Switch 语句来确定一周中的某一天是工作日还是周末。注意每个 `case` 之后需要 `break` 语句，否则会发生穿透。

*   **新的 Switch 表达式**

对于上面的程序，新的 Switch 表达式写法如下：

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

是不是简洁了很多。从这里可以看出他们之间的差异如下：

*   **返回值**：老的 Switch 仅仅只是语句，用来控制流程的，无法返回值。但是新的 Switch 可以作为表达式使用，支持返回值。
*   **`->`**** 代替 **：老的 Switch 需要在每个案例后面使用 break ，否则会发生 “穿透”，而新的不需要，它会自动终止。
*   **多值匹配**：老的 Switch 无法在一个案例标签中匹配多个值，而新的 Switch 表达式允许一个 `case` 匹配多个值，用`","` 分割即可。
*   **更简洁的语法**: 整体代码更简洁，易于阅读和维护。
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/9479813794)

> Switch 表达式 Java12 引入 Switch 表达式，它解决了传统 Switch 语句的两个缺陷："Fall-through" 行为：在没有显式 break 语句的情况下，Switch 语句会

![](https://sike.skjava.com/java-features/202311291000001.png)

Switch 表达式
----------

Java 12 引入 `Switch` 表达式，它解决了传统 `Switch` 语句的两个缺陷：

1.  **"Fall-through" 行为**：在没有显式 `break` 语句的情况下，`Switch` 语句会从一个 case "穿透" 到下一个 case，忽略了这个会导致不可饶恕的错误。
2.  **代码冗余**：每个 case，我们都需要重复类似的代码结构，增加了代码的冗余和维护难度。

`Switch` 表达式引入了 `->` 操作符，用于替代传统的冒号（`:`）。与传统的 Switch 语句不同，使用 `->` 的 case 分支不会出现 `"fall-through"` 现象，因此不需要 break 语句来防止穿透，如下：

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

它虽然解决这两个问题，但是还是有一个不好的地方，就是返回值，比如在处理复杂的逻辑时，仍需依赖外部变量来返回结果，所以 Java 13 对 `Switch` 表达式进行了扩展，引入 `yield`关键字来处理多分支结构中的返回值，如下：

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

但是，不知道小伙伴们注意没有，`Switch` 表达式只有一种类型，如果我们要处理多种类型呢？我们只能这样处理 ：

```
Object obj = ... 

switch (obj.getClass().getName()) {
    case "java.lang.String":
        String str = (String) obj;
        
        break;
    case "java.lang.Integer":
        Integer i = (Integer) obj;
        
        break;
    case "com.lang.Long":
        Long lon = (Long) obj;
        
        break;
    
}



```

这里面的强制转换是不是比较烦？而且有没有很熟悉 ？Java 14 引入的模式匹配是不是可以解决？

> 更多阅读：J[ava 12 新特性—Switch 表达式](https://www.skjava.com/series/article/5782529848) [](https://www.skjava.com/series/article/www.skjava.com) && [Java 13 新特性—Switch 表达式扩展](https://www.skjava.com/series/article/9120322209)

模式匹配
----

`instanceof` 用于检查一个对象是否是特定类的实例或者该类的子类的实例。它通常用在条件语句中，以确定对象的类型，从而避免在向下转型时发生 `ClassCastException`。在 Java 14 前，我们一般都是这样写：

```
if (object instanceof String) {
    
    String str = (String) object;
}


```

if 语句里面的强制转换显得很是多余，所以 Java 14 引入模式匹配的 `instanceof` 来解决这个问题，它允许在 `instanceof` 操作符的条件判断中直接定义一个变量，如果对象是指定的类型，这个变量会被初始化为被检查的对象，可以立即使用，无需额外的类型转换，如：

```
if (obj instanceof String str) {
    // 可以直接使用 str，而不需要显式的类型转换
}


```

但是这里需要写大量的 `if...else`，如果它和上面的 `Switch` 表达式相结合是不是就可以产生不一样的化学反应？

> 更多阅读：[Java 14 新特性—模式匹配的 instanceof](https://www.skjava.com/series/article/3436282193)

Switch 模式匹配
-----------

用于 `instanceof` 的模式匹配在 Java 16 成为正式特性，到了 Java 17 模式匹配的应用扩展到了`switch`表达式，这标志着 Switch 表达式又得到了一次增强。

在 Java 17 中，`switch` 表达式允许使用模式匹配来处理对象类型，这样就可以直接在 `switch` 语句中检查和转换类型，而不需要额外的 `if...else` 结构和显式类型转换。

`switch` 模式匹配支持以下几种模式：

1.  **类型模式**
2.  **空模式**
3.  **守卫模式**
4.  **常量模式**

### 类型模式

这是一种比较常见的模式，它允许在 `switch` 语句的 case 分支中直接匹配对象的类型。例如，`case String s` 允许你在该分支中直接作为字符串类型的 `s` 来使用，避免了显式的类型检查和强制类型转换。举个例子来说明下：

```
    @Test
    public void switchTest() {
        Object[] objects = { "Hello", 123, "World", "Java", 3.14, "skjava" };
        for (Object obj: objects) {
            if (obj instanceof Integer intR) {
                System.out.println("为整数型：" + intR);
            } else if (obj instanceof Float floatR) {
                System.out.println("为浮点型：" + floatR);
            } else if (obj instanceof Double doubleR) {
            System.out.println("为双精度浮点数：" + doubleR);
            } else if (obj instanceof String str) {
                System.out.println("为字符串：" + str);
            } else {
                System.out.println("其他类型：" + obj);
            }
        }
    }


```

我们用 `Switch` 表达式来改造下：

```
    @Test
    public void switchTest() {
        Object[] objects = { "Hello", 123, "World", "Java", 3.14, "skjava" };
        for (Object obj: objects) {
            switch (obj) {
                case Integer intR -> System.out.println("为整数型：" + intR);
                case Float floatR -> System.out.println("为浮点型：" + floatR);
                case Double doubleR -> System.out.println("为双精度浮点数：" + doubleR);
                case String str -> System.out.println("为字符串：" + str);
                default -> System.out.println("其他类型：" + obj);
            }
        }
    }


```

相比上面的 `if...else` 简洁了很多。同时在 Java 17 之前，Switch 选择器表达式只支持特定类型，即基本整型数据类型`byte`、`short`、`char`和`int`；对应的装箱形式`Byte`、`Short`、`Character`和`Integer`；`String`类；枚举类型。现在有了类型模式，Switch 表达式可以是任何类型啦。

### **空模式**

在 Java17 之前，向`switch`语句传递一个`null`值，会抛出一个`NullPointerException`，现在可以通过类型模式，将 null 检查作为一个单独的 case 标签来处理，如下：

```
    @Test
    public void switchTest() {
        Object[] objects = { "Hello", 123, "World", "Java", 3.14, "skjava" };
        for (Object obj: objects) {
            switch (obj) {
                
                case null -> System.out.println("为空值");
                default -> System.out.println("其他类型：" + obj);
            }
        }
    }


```

`case null` 可以直接匹配值为 `null` 的情况。

### 守卫模式

守卫模式允许我们在 case 标签后添加一个额外的条件。只有当类型匹配并且额外条件为真时，才会进入该 case 块。

比如上面例子，我们要将字符串那块逻辑调整下，比如长度大于 5 的为长字符串，小于 5 的为短字符串，在不使用守卫模式的情况下，我们一般这样写：

```
    @Test
    public void switchTest() {
        Object[] objects = { "Hello", 123, "World", "Java", 3.14, "skjava" };
        for (Object obj: objects) {
            switch (obj) {
                case Integer intR -> System.out.println("为整数型：" + intR);
                case Float floatR -> System.out.println("为浮点型：" + floatR);
                case Double doubleR -> System.out.println("为双精度浮点数：" + doubleR);
                case String str -> {
                    if (str.length() > 5) {
                        System.out.println("为长字符串：" + str);
                    } else {
                        System.out.println("为短字符串：" + str);
                    }
                }
                case null -> System.out.println("为空值");
                default -> System.out.println("其他类型：" + obj);
            }
        }
    }


```

这种写法就显得不是那么友好，使用守卫模式如下：

```
    @Test
    public void switchTest() {
        Object[] objects = { "Hello", 123, "World", "Java", 3.14, "skjava" };
        for (Object obj: objects) {
            switch (obj) {
                case Integer intR -> System.out.println("为整数型：" + intR);
                case Float floatR -> System.out.println("为浮点型：" + floatR);
                case Double doubleR -> System.out.println("为双精度浮点数：" + doubleR);
                case String str && str.length() > 5 -> System.out.println("为长字符串：" + str);
                case String str -> System.out.println("为短字符串：" + str);
                case null -> System.out.println("为空值");
                default -> System.out.println("其他类型：" + obj);
            }
        }
    }


```

使用守卫模式，我们可以编写更灵活和表达性强的代码。
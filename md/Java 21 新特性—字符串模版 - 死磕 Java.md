> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1880587197)

> 引言字符串操作是 Java 中使用最频繁的操作，没有之一。其中非常常见的操作之一就是对字符串的组织，由于常见所以就衍生了多种方案。比如我们要实现 x+y=?，方案有如下几种使用 + 进行字符串拼接 Strings

![](https://sike.skjava.com/java-features/202312021000001.png)

引言
--

字符串操作是 Java 中使用最频繁的操作，没有之一。其中非常常见的操作之一就是对字符串的组织，由于常见所以就衍生了多种方案。比如我们要实现 `x + y = ?`，方案有如下几种

*   使用 `+` 进行字符串拼接

```
String s = x + " + " + y + " = " + (x + y);


```

*   使用 StringBuilder

```
String s = new StringBuilder()
                 .append(x)
                 .append(" + ")
                 .append(y)
                 .append(" = ")
                 .append(x + y)
                 .toString()



```

*   `String::format` 和 `String::formatted` 将格式字符串从参数中分离出来

```
String s = String.format("%2$d + %1$d = %3$d", x, y, x + y);
or 
String s = "%2$d + %1$d = %3$d".formatted(x, y, x + y);



```

*   `java.text.MessageFormat`

```
String s = MessageFormat.format("{0} + {1} = {2}", x,y, x + y);



```

这四种方案虽然都可以解决，但很遗憾的是他们或多或少都有点儿缺陷，尤其是面对 Java 13 引入的文本块（[Java 13 新特性—文本块](https://www.skjava.com/series/article/www.skjava.com)）更是束手无措。

字符串模板
-----

为了简化字符串的构造和格式化，Java 21 引入字符串模板功能，该特性主要目的是为了提高在处理包含多个变量和复杂格式化要求的字符串时的可读性和编写效率。

它的设计目标是：

*   通过简单的方式表达混合变量的字符串，简化 Java 程序的编写。
*   提高混合文本和表达式的可读性，无论文本是在单行源代码中（如字符串字面量）还是跨越多行源代码（如文本块）。
*   通过支持对模板及其嵌入式表达式的值进行验证和转换，提高根据用户提供的值组成字符串并将其传递给其他系统（如构建数据库查询）的 Java 程序的安全性。
*   允许 Java 库定义字符串模板中使用的格式化语法（java.util.Formatter ），从而保持灵活性。
*   简化接受以非 Java 语言编写的字符串（如 SQL、XML 和 JSON）的 API 的使用。
*   支持创建由字面文本和嵌入式表达式计算得出的非字符串值，而无需通过中间字符串表示。

该特性处理字符串的新方法称为：**Template Expressions**，即：模版表达式。它是 Java 中的一种新型表达式，不仅可以执行字符串插值，还可以编程，从而帮助开发人员安全高效地组成字符串。此外，模板表达式并不局限于组成字符串——它们可以根据特定领域的规则将结构化文本转化为任何类型的对象。

### STR 模板处理器

`STR` 是 Java 平台定义的一种模板处理器。它通过用表达式的值替换模板中的每个嵌入表达式来执行字符串插值。使用 STR 的模板表达式的求值结果是一个字符串。

`STR` 是一个公共静态 final 字段，会自动导入到每个 Java 源文件中。

我们先看一个简单的例子：

```
    @Test
    public void STRTest() {
        String sk = "死磕 Java 新特性";
        String str1 = STR."\{sk}，就是牛";
        System.out.println(str1);
    }

死磕 Java 新特性，就是牛



```

上面的 `STR."\{sk}，就是牛"` 就是一个模板表达式，它主要包含了三个部分：

*   模版处理器：`STR`
*   包含内嵌表达式（`\{blog}`）的模版
*   通过`.`把前面两部分组合起来，形式如同方法调用

当模版表达式运行的时候，模版处理器会将模版内容与内嵌表达式的值组合起来，生成结果。

这个例子只是 STR 模版处理器一个很简单的功能，它可以做的事情有很多。

*   数学运算

比如上面的 `x + y = ?`：

```
    @Test
    public void STRTest() {
        int x = 1,y =2;
        String str = STR."\{x} + \{y} = \{x + y}";
        System.out.println(str);
    }


```

这种写法是不是简单明了了很多？

*   调用方法

STR 模版处理器还可以调用方法，比如：

```
String str = STR."今天是：\{ LocalDate.now()} ";


```

当然也可以调用我们自定义的方法：

```
    @Test
    public void STRTest() {
        String str = STR."\{getSkStr()},就是牛";
        System.out.println(str);
    }

    public String getSkStr() {
        return "死磕 Java 新特性";
    }


```

*   访问成员变量

STR 模版处理器还可以访问成员变量，比如：

```
public record User(String name,Integer age) {
}

@Test
public void STRTest() {
  User user = new User("大明哥",18);
  String str = STR."\{user.name()}今年\{user.age()}";
  System.out.println(str);
}



```

需要注意的是，字符串模板表达式中的嵌入表达式数量没有限制，它从左到右依次求值，就像方法调用表达式中的参数一样。例如：

```
    @Test
    public void STRTest() {
        int i = 0;
        String str = STR."\{i++},\{i++},\{i++},\{i++},\{i++}";
        System.out.println(str);
    }

0,1,2,3,4



```

同时，表达式中也可以嵌入表达式：

```
    @Test
    public void STRTest() {
        String name = "大明哥";
        String sk = "死磕 Java 新特性";
        String str = STR."\{name}的\{STR."\{sk},就是牛..."}";
        System.out.println(str);
    }

大明哥的死磕 Java 新特性,就是牛...



```

但是这种嵌套的方式会比较复杂，容易搞混，一般不推荐。

### 多行模板表达式

为了解决多行字符串处理的复杂性，Java 13 引入文本块（[Java 13 新特性—文本块](https://www.skjava.com/series/article/www.skjava.com)），它是使用三个双引号（`"""`）来标记字符串的开始和结束，允许字符串跨越多行而无需显式的换行符或字符串连接。如下：

```
String html = """
              <html>
                  <body>
                      <h2>skjava.com</h2>
                      <ul>
                        <li>死磕 Java 新特性</li>
                        <li>死磕 Java 并发</li>
                        <li>死磕 Netty</li>
                        <li>死磕 Redis</li>
                      </ul>
                  </body>
              </html>
              """;


```

如果字符串模板表达式，我们就只能拼接这串字符串了，这显得有点儿繁琐和麻烦。而字符串模版表达式也支持多行字符串处理，我们可以利用它来方便的组织 html、json、xml 等字符串内容，比如这样：

```
    @Test
    public void STRTest() {
        String title = "skjava.com";
        String sk1 = "死磕 Java 新特性";
        String sk2 = "死磕 Java 并发";
        String sk3 = "死磕 Netty";
        String sk4 = "死磕 Redis";

        String html = STR."""
              <html>
                  <body>
                      <h2>\{title}</h2>
                      <ul>
                        <li>\{sk1}</li>
                        <li>\{sk2}</li>
                        <li>\{sk3}</li>
                        <li>\{sk4}</li>
                      </ul>
                  </body>
              </html>
              """;
        System.out.println(html);
    }


```

如果决定定义四个 `sk` 变量麻烦，可以整理为一个集合，然后调用方法生成 `<li>` 标签。

### FMT 模板处理器

FMT 是 Java 定义的另一种模板处理器。它除了与 STR 模版处理器一样提供插值能力之外，还提供了左侧的格式化处理。下面我们来看看他的功能。比如我们要整理模式匹配的 Switch 表达在 Java 版本中的迭代，也就是下面这个表格

<table><thead><tr><th>Java 版本</th><th>更新类型</th><th>JEP</th><th>更新内容</th></tr></thead><tbody><tr><td>Java 17</td><td>第一次预览</td><td>JEP 406</td><td>引入模式匹配的 Swith 表达式作为预览特性。</td></tr><tr><td>Java 18</td><td>第二次预览</td><td>JEP 420</td><td>对其做了改进和细微调整</td></tr><tr><td>Java 19</td><td>第三次预览</td><td>JEP 427</td><td>进一步优化模式匹配的 Swith 表达式</td></tr><tr><td>Java 20</td><td>第四次预览</td><td>JEP 433</td><td></td></tr><tr><td>Java 21</td><td>正式特性</td><td>JEP 441</td><td>成为正式特性</td></tr></tbody></table>

如果使用 STR 模板处理器，代码如下：

```
    @Test
    public void STRTest() {
        SwitchHistory[] switchHistories = new SwitchHistory[]{
                new SwitchHistory("Java 17","第一次预览","JEP 406","引入模式匹配的 Swith 表达式作为预览特性。"),
                new SwitchHistory("Java 18","第二次预览","JEP 420","对其做了改进和细微调整"),
                new SwitchHistory("Java 19","第三次预览","JEP 427","进一步优化模式匹配的 Swith 表达式"),
                new SwitchHistory("Java 20","第四次预览","JEP 433",""),
                new SwitchHistory("Java 21","正式特性","JEP 441","成为正式特性"),
        };

        String history = STR."""
                Java 版本     更新类型    JEP 更新内容
                \{switchHistories[0].javaVersion()} \{switchHistories[0].updateType()} \{switchHistories[0].jep()} \{switchHistories[0].content()}
                \{switchHistories[1].javaVersion()} \{switchHistories[1].updateType()} \{switchHistories[1].jep()} \{switchHistories[1].content()}
                \{switchHistories[2].javaVersion()} \{switchHistories[2].updateType()} \{switchHistories[2].jep()} \{switchHistories[2].content()}
                \{switchHistories[3].javaVersion()} \{switchHistories[3].updateType()} \{switchHistories[3].jep()} \{switchHistories[3].content()}
                \{switchHistories[4].javaVersion()} \{switchHistories[4].updateType()} \{switchHistories[4].jep()} \{switchHistories[4].content()}
                """;
        System.out.println(history);
    }


```

得到的效果是这样的：

```
Java 版本     更新类型    JEP 更新内容
Java 17 第一次预览 JEP 406 引入模式匹配的 Swith 表达式作为预览特性。
Java 18 第二次预览 JEP 420 对其做了改进和细微调整
Java 19 第三次预览 JEP 427 进一步优化模式匹配的 Swith 表达式
Java 20 第四次预览 JEP 433 
Java 21 正式特性 JEP 441 成为正式特性


```

是不是很丑？完全对不齐，没法看。为了解决这个问题，就可以采用 FMT 模版处理器，在每一列左侧定义格式：

```
   @Test
    public void STRTest() {
        SwitchHistory[] switchHistories = new SwitchHistory[]{
                new SwitchHistory("Java 17","第一次预览","JEP 406","引入模式匹配的 Swith 表达式作为预览特性。"),
                new SwitchHistory("Java 18","第二次预览","JEP 420","对其做了改进和细微调整"),
                new SwitchHistory("Java 19","第三次预览","JEP 427","进一步优化模式匹配的 Swith 表达式"),
                new SwitchHistory("Java 20","第四次预览","JEP 433",""),
                new SwitchHistory("Java 21","正式特性","JEP 441","成为正式特性"),
        };

        String history = FMT."""
                Java 版本     更新类型        JEP             更新内容
                %-10s\{switchHistories[0].javaVersion()}  %-9s\{switchHistories[0].updateType()} %-10s\{switchHistories[0].jep()} %-20s\{switchHistories[0].content()}
                %-10s\{switchHistories[1].javaVersion()}  %-9s\{switchHistories[1].updateType()} %-10s\{switchHistories[1].jep()} %-20s\{switchHistories[1].content()}
                %-10s\{switchHistories[2].javaVersion()}  %-9s\{switchHistories[2].updateType()} %-10s\{switchHistories[2].jep()} %-20s\{switchHistories[2].content()}
                %-10s\{switchHistories[3].javaVersion()}  %-9s\{switchHistories[3].updateType()} %-10s\{switchHistories[3].jep()} %-20s\{switchHistories[3].content()}
                %-10s\{switchHistories[4].javaVersion()}  %-9s\{switchHistories[4].updateType()} %-10s\{switchHistories[4].jep()} %-20s\{switchHistories[4].content()}
                """;
        System.out.println(history);
    }


```

输出如下：

```
Java 版本     更新类型        JEP             更新内容
Java 17     第一次预览     JEP 406    引入模式匹配的 Swith 表达式作为预览特性。
Java 18     第二次预览     JEP 420    对其做了改进和细微调整         
Java 19     第三次预览     JEP 427    进一步优化模式匹配的 Swith 表达式
Java 20     第四次预览     JEP 433                        
Java 21     正式特性       JEP 441    成为正式特性  


```
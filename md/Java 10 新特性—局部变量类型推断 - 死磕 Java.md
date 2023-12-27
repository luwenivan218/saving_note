> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1075067951)

> 局部变量类型推断是 Java10 中引入的一项重要特性，通过使用 var 关键字，允许我们在声明局部变量时省略显式类型。类型推断意味着编译器会查看变量的初始化器并推断出变量的类型。产生背景刚刚学 Java 语法时

局部变量类型推断是 Java 10 中引入的一项重要特性，通过使用`var`关键字，允许我们在声明局部变量时省略显式类型。**类型推断意味着编译器会查看变量的初始化器并推断出变量的类型**。

![](https://sike.skjava.com/java-features/202311051000002.png)

产生背景
----

刚刚学 Java 语法时，我们就被告知：**在 Java 中，所有的变量在使用前必须声明**，所以我们就有了如下代码：

```
int i = 10;
String str = "死磕 Java 新特性";
List<String> list = new ArrayList<>();


```

甚至很多小伙伴已经养成了习惯，在声明变量时永远都是从左写到右，即先写变量类型，然后变量名，最后初始化，这样写有问题吗？没有问题，但是比较繁琐，简单的变量类型还可以接受，但是如果复杂呢？比如这个：

```
Map<String, List<String>> myMap = new HashMap<>();


```

又或者这个：

```
List<Map<String,List<String>>> list = new ArrayList<>();


```

确实很繁琐，对于很多变量声明通常需要重复类型信息，这显得冗余又不增加任何价值。而且许多其他现代编程语言，如 Scala、Kotlin 和 C#，都已经提供了类型推断的能力，Java 作为一门与时俱进的现代化编程语言，怎么能不跟进呢?

好处
--

1.  **减少样板代**：减少了啰嗦和形式的代码，避免了信息冗余，使得代码更易于编写和阅读。例如用`var list = new ArrayList<String>();`代替`ArrayList<String> list = new ArrayList<>();`。
2.  **增强代码的可读性**：很多时候我们在阅读代码时并不关注变量的类型，或者变量类型显而易见，而 `var` 使得变量的意图变得更清晰。
3.  **提高开发效率**：减少编写显式类型所需的时间和努力，可以让开发者更专注于业务逻辑本身，毕竟太多的类型声明只会分散注意力，不会带来额外的好处。

使用场景
----

### 局部变量声明

在局部代码块中声明临时变量，特别是当这些变量的类型很明显时，使用`var`可以减少代码的冗余。比如

```
var i = 10;
var str = "死磕 Java 新特性";


```

这是最简单明了的使用方式 了，尤其是局部变量的类型常常非常长或复杂，使用`var`能够使代码更简洁。。

### **初始化集合和泛型表达式**

在使用集合或泛型类时，类型通常很复杂，使用 `var` 可以避免在声明和初始化时重复复杂的类型。

```
List<String> list = new ArrayList<>();

var list = new ArrayList<String>();

Map<String,List<String>> map = new HashMap<>();

var map = new HashMap<String,List<String>>();



```

在 Java 7 以前，我们声明集合变量时需要这样写：

```
 List<String> list = new ArrayList<String>();    


```

可能你觉得在声明变量的时候已经指明了参数类型，为什么还要在初始化对象时再指定？所以在 Java 7 中引入**泛型类型推断**，于是写法可以简化这样的：

```
List<String> list = new ArrayList<>();    


```

编译器会根据变量声明时的泛型类型自动推断出实例化 List 时的泛型类型。但是我们一定要加上 `“<>”`，只有加上这个 `“<>”` 才表示是自动类型推断。

现在 Java 10 又引入了局部变量类型推断，所以又可以进一步简化：

```
var list = new ArrayList<String>();


```

可能有小伙伴会问，可以写成下面这种格式么：

```
var list = new ArrayList<>();


```

可以，只不过这个时候类型就变成了 Object：

![](https://sike.skjava.com/java-features/202311051000001.jpg)

### 遍历操作

迭代时，使用 `var` 可以避免编写冗长的类型名称。例如：

*   常规写法

```
    @Test
    public void varTest() {
        var list = new ArrayList<String>();
        list.add("死磕 Java 新特性");
        list.add("死磕 Java 并发");
        list.add("死磕 Netty");
        
        for (String str : list) {
            System.out.println(str);
        }
    }


```

`for-each` 里面指明了变量 str 的类型，但是我们明明都知道 list 的类型是 String，为什么还要指明呢？不是多此一举么：

```
    @Test
    public void varTest() {
        

        for (var str : list) {
            System.out.println(str);
        }
    }


```

### 在 try-with-resources 语句中

try-with-resources 要求变量是 final 或事实上的 final，在这里使用`var`可以减少冗余代码。比如：

```
try (var reader = new BufferedReader(new FileReader(path))) {
    
}


```

不适用场景
-----

虽然局部变量类型推断可以提高代码的简洁性，但在某些情况下使用`var`可能并不合适，因为它可能会降低代码的可读性和可维护性。

*   **使用 null 初始化变量时**

不能使用 `var` 来声明一个初始化为 `null` 的变量，因为 `var` 需要足够的信息来推断出变量的类型。

```
var object = null;   


```

*   **需要使用具体类特定方法时**

如果需要调用一个特定类的方法，而这个方法不是由其所有可能的子类共有的，那么最好显式声明这个类类型。比如：

*   **类型信息很重要**

有些时候变量的类型对于我们理解代码很重要，这个时候我们就不能省略变量的类型，尽管可以省略。

*   **数组静态初始化**

数组静态初始化是不能省略的。

```
int[] arr1 = new int[]{1,2,3,4,5}      
int[] arr1 =  {1,2,3,4,5};             
var arr2 = new int[]{1,2,3,4,5};       
var arr3 = {1,2,3,4,5}                 


```

这里这是一些场景，还有其他一些场景也是不能使用的，我们把握一个核心原则就行：**能够让编译器推断出是什么类型的都可以使用**。

注意事项
----

*   使用类型推断时，必须在变量声明时初始化变量，这样编译器才能推断出类型。它只适用于局部变量，不能用于方法参数、返回类型、字段等。
*   类型推断可能使得代码阅读者不清楚变量的实际类型，尤其是当初始化器的表达式不直接显露类型时。**应当避免过度使用** **`var`**。
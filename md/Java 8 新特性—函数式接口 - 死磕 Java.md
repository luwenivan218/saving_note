> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1841959591)

> 在文章 Lambda 表达式提过，Lambda 能够简化的一个依据就是函数式接口，这篇文章我们就来深入了解函数式接口。什么是函数式接口函数式接口是一个只有一个抽象方法的接口，最开始的时候也叫做 SAM 类型的接

在文章 [Lambda 表达式](https://www.skjava.com/series/article/1747461365) 提过，Lambda 能够简化的一个依据就是函数式接口，这篇文章我们就来深入了解函数式接口。

![](https://sike.skjava.com/java-features/202310041000002.png)

什么是函数式接口
--------

函数式接口是一个只有一个抽象方法的接口，最开始的时候也叫做 **SAM 类型**的接口（`Single Abstract Method`）。它具有两个特点：

1.  **只包含一个抽象方法**：函数式接口只能有一个抽象方法，但可以包含多个默认方法或静态方法。
2.  **用** **`@FunctionalInterface`** **注解标记**：该注解不强制，但通常会使用它来标记该接口为函数式接口。这样做可以让编译器检查接口是否符合函数式接口的定义，以避免不必要的错误。

Java 引入函数式接口的主要目的是支持函数式编程范式，也就是 Lambda 表达式。在函数式编程语言中，函数被当做一等公民对待，Lambda 表达式的类型是函数，它可以像其他数据类型一样进行传递、赋值和操作。但是在 Java 中，“**一切皆对象**” 是不可违背的宗旨，所以 Lambda 表达式是对象，而不是函数，他们必须要依附于一类特别的对象类型：函数式接口。

> 所以，**从本质上来说 Lambda 表达式就是一个函数式接口的实例**。这就是 Lambda 表达式和函数式接口的关系。简单理解就是只要一个对象时函数式接口的实例，那么该对象就可以用 Lambda 表达式来表示。

自定义函数式接口
--------

根据函数式接口的定义和特点，我们可以自定义函数式接口：

```
@FunctionalInterface
public interface FunctionInterface {

    
    void doSomething();

    
    default void defaultMethod(String s) {
        System.out.println("默认方法：" + s);
    }

    
    static void staticMethod(String s) {
        System.out.println("静态方法：" + s);
    }
}


```

FunctionInterface 是一个自定义函数式接口，它只包含一个抽象方法 `doSomething()`，还包含一个默认方法 `defaultMethod(String s)` 和一个静态方法 `staticMethod(String s)`，这两个方法都是可选的。

`@FunctionalInterface` 注解是可写可可不写的，但是我们一般都推荐写，写上他可以让编译器检查接口是否符合函数式接口的定义，以避免不必要的错误，比如：

![](https://sike.skjava.com/java-features/202310041000001.jpg)

上面接口定义了两个抽象方法，它会明确告诉你错误了。

使用如下：

```
        FunctionInterface functionInterface = () -> {
            System.out.println("死磕 Java 就是牛...");
        };

        
        functionInterface.doSomething();
        
        functionInterface.defaultMethod("死磕 Netty 就是牛...");
        
        FunctionInterface.staticMethod("死磕 Java 并发就是牛...");


```

执行如下：

![](https://sike.skjava.com/java-features/202310031000001.jpg)

常用函数式接口
-------

其实在 Java 8 之前就已经有了大量的函数式接口，我们最熟悉的就是 `java.lang.Runnable`接口了。Java 8 之前已有的函数式接口：

*   `java.lang.Runnable`
*   `java.util.concurrent.Callable`
*   `java.security.PrivilegedAction`
*   `java.util.Comparator`
*   `java.io.FileFilter`
*   `java.nio.file.PathMatcher`
*   `java.lang.reflect.InvocationHandler`
*   `java.beans.PropertyChangeListener`
*   `java.awt.event.ActionListener`
*   `javax.swing.event.ChangeListener`

而在 Java 8 中，新增的函数式接口都在 `java.util.function` 包中，里面有很多函数式接口，用来支持 Java 的函数式编程，从而丰富了 Lambda 表达式的使用场景。我们使用最多的也是最核心的函数式接口有四个：

*   `java.util.function.Consumer`：消费型接口
*   `java.util.function.Function`：函数型接口
*   `java.util.function.Supplier`：供给型接口
*   `java.util.function.Predicate`：断定型接口

下面我们就来看这四个函数式接口的使用方法

### Consumer 接口

Consumer 代表这一个接受一个输入参数并且不返回任何结果的操作。它包含一个抽象方法 `accept(T t)`，该方法接受一个参数 `t`，并对该参数执行某种操作：

```
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}


```

由于 Consumer 接口中包含的抽象方法不返回结果，所以它通常用于对对象进行一些操作，如修改、输出、打印等。它的使用方法也比较简单，分为两步。

1.  **创建一个 Consumer 对象**：使用 Lambda 表达式来创建一个对象，定义在 `accept(T t)` 中要执行的操作。

```
Consumer<String> consumer = str -> System.out.println(str);


```

2.  **使用 Consumer 对象**

```
consumer.accept("死磕 Java 就是牛...");



死磕 Java 就是牛...



```

在 Consumer 接口中还有一个默认方法 `andThen()`，该方法接受一个 Consumer 实例对象 after，它允许我们将两个 Consumer 对象组合在一起，形成一个新的 Consumer 对象，该新对象按照顺序执行这两个 Consumer 对象的操作。先执行调用`andThen()`接口的`accept()`，然后再执行`andThen()`参数 after 中的`accept()`。

```
Consumer<String> consumer1 = str -> System.out.println("consumer1：" + str);
Consumer<String> consumer2 = str -> System.out.println("consumer2：" + str);

consumer1.andThen(consumer2).accept("死磕 Java 就是牛..");



consumer1：死磕 Java 就是牛..
consumer2：死磕 Java 就是牛..



```

### Function 接口

Function 代表一个接受一个输入参数并且产生一个输出结果的函数。它包含一个抽象方法 `R apply(T t)`，该方法接受一个参数 `t`（类型为 `T`），并返回一个结果（类型为 `R`），我们可以理解为根据一个数据类型 T ，经过一系列的操作后得到类型 R。Function 接口是非常通用的，应该是他们四个当中使用最为广泛的。

> **用途一：函数转换**

Function 可以用于将一个类型的值转换为另一个类型的值。它可以用于各种转换操作，如类型转换、数据映射等。

```
Function<String,Integer> function = str -> Integer.parseInt(str);
int result = function.apply("456");

456


```

> **用途二：数据处理**

Function 可用于对输入数据进行处理并生成输出结果。它可以用于执行各种操作，如过滤、计算、提取、格式化等。

```
Function<List<String>, String> function = list -> {
    StringBuilder result = new StringBuilder();
        for (String str : list) {
            if (str.startsWith("李")) {
                result.append(str).append(",");
            }
        }

        return result.toString();
    };
List<String> list = Arrays.asList("张三","李四","李武","李柳");
System.out.println(function.apply(list));

李四,李武,李柳,   


```

> `andThen()`**：方法链式调用**

`andThen()` 接受一个 Function 作为参数，并返回一个新的 Function，该新函数首先应用当前函数，然后将结果传递给参数函数。这种方法链的方式可以用于将多个函数组合在一起，以执行一系列操作。

```
Function<String,Integer> function1 = t -> Integer.parseInt(t);
Function<Integer,Integer> function2 = t -> t * 10;
System.out.println(function1.andThen(function2).apply("20"));


```

先将 String 转换为 Integer，然后再 `* 10`，利用 `andThen()` 我们可以进行一系列复杂的操作。

> `compose()`**：顺序执行**

`compose()` 与 `andThen()`相反，它首先应用参数函数，然后再应用当前函数，这种可能更加好理解些，常用于一些顺序执行。

```
Function<String,Integer> function1 = t -> {
  System.out.println("function1");
  return Integer.parseInt(t);
};
Function<Integer,Integer> function2 = t -> {
  System.out.println("function2");
  return t * 10;
};
Function<Integer,String> function3 = t -> {
  System.out.println("function3");
  return t.toString();
};
System.out.println(function3.compose(function2.compose(function1)).apply("20"));
        

function1
function2
function3
200




```

从输出结果中可以更加直观地看清楚他们的执行顺序。

> `identity()`：**恒等函数**

`identity()` 返回一个恒等函数，它仅返回其输入值，对输入值不进行任何操作。源码如下：

```
    static <T> Function<T, T> identity() {
        return t -> t;
    }


```

一看感觉 `identity()` 没啥用处，其实它在某些场景大有用处，例如

*   **作为默认函数**

`identity()` 可以作为函数组合链中的起点或默认函数。当我们想构建一个函数组合链时，可以使用 `identity` 作为初始函数，然后使用 `andThen()` 或 `compose()` 方法添加其他函数。这种方式允许您以一种优雅的方式处理链的起点。

```
Function<String,String> function1 = Function.identity();
Function<String,String> function2 = str -> str.toUpperCase();
Function<String,String> function3 = str -> str + " WORLD!!!";

System.out.println(function3.compose(function2.compose(function1)).apply("hello"));


```

*   **保持一致性**

在某些情况下，我们可能需要一个函数，但不需要对输入进行任何操作。使用 `identity()` 可以确保函数的签名（输入和输出类型）与其他函数一致。

### Supplier 接口

Supplier 是一个代表生产（或供应）某种结果的接口，它不接受任何参数，但能够提供一个结果。它定义了一个 `get()` 的抽象方法，用于获取结果。

接口定义简单，使用也简单：

```
Supplier<LocalDate> supplier = () -> LocalDate.now();
LocalDate localDate = supplier.get();



```

Supplier 接口通常用于惰性求值，只有在需要结果的时候才会执行 `get()` 。这对于延迟计算和性能优化非常有用。

### Predicate 接口

Predicate 表示一个谓词，它接受一个输入参数并返回一个布尔值，用于表示某个条件是否满足。抽象方法为 `test()`，使用如下：

```
Predicate<String> predicate = str -> str.length() > 10;
boolean result = predicate.test("www.skjava.com");


```

判断某个字符长度是否大于 10。

> `and()`：表示两个 Predicate 的 与操作

```
Predicate<Integer> predicate1 = x -> x > 10;
Predicate<Integer> predicate2 = x -> x % 2 == 0;
boolean result = predicate1.and(predicate2).test(13);


```

> `or()`：表示两个 Predicate 的或操作

```
Predicate<Integer> predicate1 = x -> x > 10;
Predicate<Integer> predicate2 = x -> x % 2 == 0;
boolean result = predicate1.or(predicate2).test(13);


```

> `negate()`：表示 Predicate 的逻辑非操作

```
Predicate<Integer> predicate1 = x -> x > 10;
boolean result = predicate1.negate().test(14);


```

### 其他函数式接口

除了上面四个常用的函数式接口外，`java.util.function` 包下面还定义了很多函数式接口，下面做一个简单的介绍：

<table><thead><tr><th>接口</th><th>说明</th></tr></thead><tbody><tr><td>BiConsumer&lt;T,U&gt;</td><td>表示接受两个不同类型的参数，但不返回任何结果的操作</td></tr><tr><td>BiFunction&lt;T,U,R&gt;</td><td>表示接受两个不同类型的参数，并返回一个其它类型的结果的操作</td></tr><tr><td>BinaryOperator<t></t></td><td>表示接受两个相同类型的参数，并返回一个同一类型的结果的操作</td></tr><tr><td>BiPredicate&lt;T,U&gt;</td><td>表示接受两个不同诶行的参数，且返回布尔类型的结果的操作</td></tr><tr><td>BooleanSupplier</td><td>不接受任何参数，且返回一个布尔类型的结果的操作</td></tr><tr><td>DoubleBinaryOperator</td><td>表示接受两个 double 类型的参数，并返回 double 类型结果的操作</td></tr><tr><td>DoubleConsumer</td><td>表示接受一个 double 类型的参数，但不返回任何结果的操作</td></tr><tr><td>DoubleFunction<r></r></td><td>表示接受一个 double 类型的参数，且返回一个 R 类型的结果的操作</td></tr><tr><td>DoublePredicate</td><td>表示一个接受两个 double 类型的参数，且返回一个布尔类型的结果的操作</td></tr><tr><td>DoubleSupplier</td><td>表示一个不接受任何参数，但返回布尔类型的结果的操作</td></tr><tr><td>DoubleToIntFunction</td><td>表示接受两个 double 类型的参数，但返回一个 int 类型的结果的操作</td></tr><tr><td>DoubleToLongFunction</td><td>表示接受两个 double 类型的参数，但返回一个 long 类型的结果的操作</td></tr><tr><td>DoubleUnaryOperator</td><td>表示接受一个 double 类型的参数，且返回一个 double 类型的结果的操作</td></tr><tr><td>IntBinaryOperator</td><td>表示一个接受两个 int 类型的参数，且返回一个 int 类型的结果的操作</td></tr><tr><td>IntConsumer</td><td>表示接受一个 int 类型的参数，但不返回任何结果的操作</td></tr><tr><td>IntFunction<r></r></td><td>表示接受一个 int 类型的参数，但返回一个 R 类型的结果的操作</td></tr><tr><td>IntPredicate</td><td>表示接受一个 int 类型的参数，但返回布尔类型的结果的操作</td></tr><tr><td>IntSupplier</td><td>表示不接受任何参数，但返回一个 int 类型的结果的操作</td></tr><tr><td>IntToDoubleFunction</td><td>表示接受一个 int 类型的参数，但返回一个 double 类型的结果的操作</td></tr><tr><td>IntToLongFunction</td><td>表示接受一个 int 类型的参数，但返回一个 long 类型的结果的操作</td></tr><tr><td>IntUnaryOperator</td><td>表示接受一个 int 类型的参数，且返回一个 int 类型的结果的操作</td></tr><tr><td>LongBinaryOperator</td><td>表示接受两个 long 类型的参数，且返回一个 long 类型的结果的操作</td></tr><tr><td>LongConsumer</td><td>表示不接受任何参数，但返回一个 long 类型的结果的操作</td></tr><tr><td>LongFunction<r></r></td><td>表示接受一个 loing 类型的参数，但返回一个 R 类型的结果的操作</td></tr><tr><td>LongPredicate</td><td>表示接受一个 long 类型的参数，但返回布尔类型的结果的操作</td></tr><tr><td>LongSupplier</td><td>表示不接受任何参数，但返回一个 long 类型的结果的操作</td></tr><tr><td>LongToDoubleFunction</td><td>表示接受一个 long 类型的参数，但返回一个 double 类型的结果的函数</td></tr><tr><td>LongToIntFunction</td><td>表示接受一个 long 类型的参数，但返回 int 类型的结果的函数</td></tr><tr><td>LongUnaryOperator</td><td>表示接受一个 long 类型的参数，并返回一个 long 类型的结果的操作</td></tr><tr><td>ObjDoubleConsumer<t></t></td><td>表示接受两个参数，一个为 T 类型的对象，另一个 double 类型，但不返回任何结果的操作</td></tr><tr><td>ObjIntConsumer<t></t></td><td>表示接受两个参数，一个为 T 类型的对象，另一个 int 类型，但不返回任何结果的操作</td></tr><tr><td>ObjLongConsumer<t></t></td><td>表示接受两个参数，一个为 T 类型的对象，另一个 double 类型，但不返回任何结果的操作</td></tr><tr><td>ToDoubleBiFunction&lt;T,U&gt;</td><td>表示接受两个不同类型的参数，但返回一个 double 类型的结果的操作</td></tr><tr><td>ToDoubleFunction<t></t></td><td>表示一个接受指定类型 T 的参数，并返回一个 double 类型的结果的操作</td></tr><tr><td>ToIntBiFunction&lt;T,U&gt;</td><td>表示接受两个不同类型的参数，但返回一个 int 类型的结果的操作</td></tr><tr><td>ToIntFunction<t></t></td><td>表示一个接受指定类型 T 的参数，并返回一个 int 类型的结果的操作</td></tr><tr><td>ToLongBiFunction&lt;T,U&gt;</td><td>表示接受两个不同类型的参数，但返回一个 long 类型的结果的操作</td></tr><tr><td>ToLongFunction<t></t></td><td>表示一个接受指定类型的参数，并返回一个 long 类型的结果的操作</td></tr><tr><td>UnaryOperator<t></t></td><td>表示接受一个参数，并返回一个与参数类型相同的结果的操作</td></tr></tbody></table>

函数式接口使用非常灵活，上面的举例都是很简单的 demo，它需要我们在日常开发过程中多多使用才能灵活地运用它。
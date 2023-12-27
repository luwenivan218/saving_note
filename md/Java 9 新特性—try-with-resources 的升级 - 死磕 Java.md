> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1487108972)

> 只要学过 Java 的小伙伴都需要知道，资源用了是要关闭的，否则会发生大麻烦，轻则被骂，重则滚蛋，不要以为是玩笑话，很严重的！！Java7 之前，资源需要手动关闭我相信有 5 年以上工作经验的小伙伴一定写过这样

原

 2023-11-19  阅读 (95)  点赞 (0)

只要学过 Java 的小伙伴都需要知道，资源用了是要关闭的，否则会发生大麻烦，轻则被骂，重则滚蛋，不要以为是玩笑话，很严重的！！

![](https://sike.skjava.com/java-features/202310311000003.png)

Java 7 之前，资源需要手动关闭
------------------

我相信有 5 年以上工作经验的小伙伴一定写过这样的代码：

```
FileInputStream fileInputStream = null;
try {
    fileInputStream = new FileInputStream("file.txt");
    
} catch (IOException e) {
    
} finally {
    if (fileInputStream != null) {
        try {
            fileInputStream.close();
        } catch (IOException e) {
            
        }
    }
}


```

这是因为，在 Java 7 之前，需要我们手动关闭各种资源，包括文件、数据库连接、网络套接字等等其他可能需要显式关闭的资源。但是这种关闭资源的方式存在几个问题：

*   容易忽略关闭资源的步骤，导致资源泄漏。
*   需要编写冗长的 `try-catch-finally` 代码，降低了代码的可读性。
*   异常处理变得复杂，需要额外的嵌套 `try-catch` 块来处理关闭资源时可能出现的异常。

Java 7 中的 try-with-resources
----------------------------

为了解决上面的问题，Java 7 引入了 `try-with-resources` 语法。`try-with-resources` 是一个用于简化资源管理的语法糖，它可以自动关闭实现了 `AutoCloseable` 或 `Closeable` 接口的资源，无需显式编写资源关闭代码。这种特性有如下几个优点：

*   资源自动化管理，减少了资源泄漏的风险。
*   减少了编写冗长的 `try-catch-finally` ，简化了代码，提高了代码的可读性和可维护性。
*   自动处理资源关闭时的异常。

使用 `try-with-resources` 的步骤如下：

> **1、资源必须实现 ****`AutoCloseable`**** 或 ****`Closeable`**** 接口**

要使用 `try-with-resources`，资源必须是实现了 `AutoCloseable`（Java 7 引入）或 `Closeable`（ Java 5 引入）接口的类的实例。这些接口要求资源类提供一个 `close` 方法，用于在不再需要资源时执行清理操作。例如：

```
public class Resource implements AutoCloseable{
    
    public void doSome() {
        System.out.println("do something....");
    }
    
    @Override
    public void close() throws Exception {
        System.out.println("关闭了 Resource 资源");
    }
}


```

> **2、语法**

`try-with-resources` 使用 `try` 块来声明和初始化资源。资源的声明必须在圆括号内，资源会在 `try` 块退出时自动关闭。此语法还可以包括一个 `catch` 块来处理可能抛出的异常。例如：

```
public class ResourceTest {
    public static void main(String[] args) {
        try(Resource resourceTest = new Resource()) {
            resourceTest.doSome();
        } catch (Exception e) {
            
        }
    }
}



```

> **3、自动关闭资源**

在 `try` 块退出时，资源会自动关闭，即使在 `try` 块内发生异常也会如此。资源的 `close()` 会被调用以释放资源，并且资源的关闭异常会被抛出或被抑制（附加到原始异常中）。例如上诉代码执行结果如下：

```
do something....
关闭了 Resource 资源


```

不管是正常退出还是因为异常退出，都会自动调用资源的 `close()` 来释放资源。

> **4、多个资源**

我们可以在同一个 `try-with-resources` 语句中管理多个资源，它会按照声明的顺序被初始化和关闭。多个资源的声明使用分号分隔。如下：

```
try (Resource1 resource1 = new Resource1();
     Resource2 resource2 = new Resource2();
     Resource3 resource2 = new Resource3()) {
     
} catch (Exception e) {
    
}



```

但是资源的关闭顺序与它们的声明顺序相反，即最后声明的资源最先关闭。

```
public class ResourceTest {
    public static void main(String[] args) {
        try(Resource1 resource1 = new Resource1();
            Resource2 resource2 = new Resource2();
            Resource3 resource3 = new Resource3()) {

        } catch (Exception e) {
            
        }
    }
}

关闭了 Resource3 资源
关闭了 Resource2 资源
关闭了 Resource1 资源



```

Java 9 对 try-with-resources 的改进
-------------------------------

Java 7 虽然利用 `try-with-resources` 来优化了资源的管理，无须我们手动关闭资源，但是它依然是不够完美，它要求管理的资源必须在 `try` 子句中初始化，否则编译不通过，如下：

![](https://sike.skjava.com/java-features/202310311000001.jpg)

为了解决这个问题，Java 9 对其改进了：** 如果你已经有一个资源是 final 或等效于 final 变量, 您可以在 try-with-resources 语句中使用该变量，而无需在 try-with-resources 语句中声明一个新变量。** 例如我们将编译环境调整到 Java 9 及以上去：

![](https://sike.skjava.com/java-features/202310311000002.jpg)

执行结果：

```
关闭了 Resource3 资源
关闭了 Resource2 资源
关闭了 Resource1 资源


```

通过这个例子我们就可以看到 Java 在不断地进步，对 Java 的未来充满信心！！！
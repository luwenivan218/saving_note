> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1690418335)

> 什么是隐藏类隐藏类，是一种不能被其他类直接使用的类。Java15 引入隐藏类主要针对的是库和框架的开发者，而不是直接面向普通 Java 应用程序开发者。它有如下几个特点：不可见性：隐藏类对于 Java 的反射 A

原

 2023-12-20  阅读 (37)  点赞 (0)

什么是隐藏类
------

隐藏类，是一种不能被其他类直接使用的类。Java 15 引入隐藏类主要针对的是库和框架的开发者，而不是直接面向普通 Java 应用程序开发者。它有如下几个特点：

*   **不可见性**：隐藏类对于 Java 的反射 API 是不可见的，这意味着它们不能通过正常的反射机制被发现或访问。但是，这并不是说，他们是完全不可见的，我们需要知道访问他们的 “密码”，只要知道这个密码就可以访问他们。隐藏类与普通 Java 类的最大区别就是隐藏类并不是“广而告之” 的，需要通过特殊的手段来找到他们。
*   **不兼容性**：隐藏类与普通的 Java 类不兼容，这意味着我们不能将一个隐藏类实例转换为任何非隐藏类，也不能将非隐藏类转换为隐藏类。
*   **生命周期管理**：隐藏类的生命周期可以由创建它们的框架或库更精细地控制。当它们不再被需要时，可以被卸载，这有助于资源管理和性能优化。

怎么使用隐藏类
-------

隐藏类不是通过常规的 Java 代码创建的，而是通过特定的 API 调用在运行时动态生成。下面大明哥来演示下如何使用隐藏类。

> **第一步：创建一个普通的 Java 类**

```
public class HiddenClasses {

    public void print() {
        System.out.println("hello，sike java!!!");
    }
}


```

> **第二步：编译 Java 类**

使用 `javac` 命令编译该文件，得到 `.class` 文件，如下：

```
javac HiddenClasses.java


```

> **第三步：读取字节码**

一旦类被编译成 `.class` 文件，我们就可以通过读取这个文件来获取其字节码。

```
Path path = Paths.get("HiddenClasses.class");

byte[] bytes = Files.readAllBytes(path);

System.out.println(Base64.getEncoder().encodeToString(bytes));


```

执行得到结果为：

```
yv66vgAAADQAHAoABgAOCQAPABAIABEKABIAEwcAFAcAFQEABjxpbml0PgEAAygpVgEABENvZGUBAA9MaW5lTnVtYmVyVGFibGUBAAVwcmludAEAClNvdXJjZUZpbGUBABJIaWRkZW5DbGFzc2VzLmphdmEMAAcACAcAFgwAFwAYAQAUaGVsbG/vvIxzaWtlIGphdmEhISEHABkMABoAGwEAJWNvbS9za2phdmEvamF2YS9mZWF0dXJlL0hpZGRlbkNsYXNzZXMBABBqYXZhL2xhbmcvT2JqZWN0AQAQamF2YS9sYW5nL1N5c3RlbQEAA291dAEAFUxqYXZhL2lvL1ByaW50U3RyZWFtOwEAE2phdmEvaW8vUHJpbnRTdHJlYW0BAAdwcmludGxuAQAVKExqYXZhL2xhbmcvU3RyaW5nOylWACEABQAGAAAAAAACAAEABwAIAAEACQAAAB0AAQABAAAABSq3AAGxAAAAAQAKAAAABgABAAAAAwABAAsACAABAAkAAAAlAAIAAQAAAAmyAAISA7YABLEAAAABAAoAAAAKAAIAAAAGAAgABwABAAwAAAACAA0=



```

> **第四步：创建隐藏类并调用其方法**

```
public class Test {
    public static void main(String[] args) throws Throwable {
        
        MethodHandles.Lookup lookup = MethodHandles.lookup();

        
        byte[] classBytes = Base64.getDecoder().decode("yv66vgAAADQAHAoABgAOCQAPABAIABEKABIAEwcAFAcAFQEABjxpbml0PgEAAygpVgEABENvZGUBAA9MaW5lTnVtYmVyVGFibGUBAAVwcmludAEAClNvdXJjZUZpbGUBABJIaWRkZW5DbGFzc2VzLmphdmEMAAcACAcAFgwAFwAYAQAUaGVsbG/vvIxzaWtlIGphdmEhISEHABkMABoAGwEAJWNvbS9za2phdmEvamF2YS9mZWF0dXJlL0hpZGRlbkNsYXNzZXMBABBqYXZhL2xhbmcvT2JqZWN0AQAQamF2YS9sYW5nL1N5c3RlbQEAA291dAEAFUxqYXZhL2lvL1ByaW50U3RyZWFtOwEAE2phdmEvaW8vUHJpbnRTdHJlYW0BAAdwcmludGxuAQAVKExqYXZhL2xhbmcvU3RyaW5nOylWACEABQAGAAAAAAACAAEABwAIAAEACQAAAB0AAQABAAAABSq3AAGxAAAAAQAKAAAABgABAAAAAwABAAsACAABAAkAAAAlAAIAAQAAAAmyAAISA7YABLEAAAABAAoAAAAKAAIAAAAGAAgABwABAAwAAAACAA0=");

        
        Class<?> hiddenClass = lookup.defineHiddenClass(classBytes, true, MethodHandles.Lookup.ClassOption.NESTMATE).lookupClass();

        
        Object hiddenClassInstance = hiddenClass.getConstructor().newInstance();

        
        var printHandle = lookup.findVirtual(hiddenClass, "print", MethodType.methodType(void.class));

        
        printHandle.invoke(hiddenClassInstance);
    }
}


```

执行得到结果：

![](https://sike.skjava.com/java-features/202311221000001.png)

整体就分为两个步骤：

1.  **生成字节码**。可以使用 `ASM` 这样的类库来生成，也可以通过大明哥这种方式，这种方式的优点在于它非常直观。
2.  **反射调用方法**。使用 `Lookup` 、 `MethodHandles` 和 `MethodType` 来创建隐藏类和调用方法。

我们需要注意的，隐藏类，是一个高级特性，主要用于库和框架的开发者，并不是面向普通的应用程序猿。而且，由于涉及到底层的字节码操作和类加载机制，所以在使用这一特性时，我们需要对 Java 的内部工作机制有深入的理解。
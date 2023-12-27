> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/9298099860)

> 接口默认方法在 Java8 之前，接口中可以申明方法和变量的，只不过变量必须是 public、static、final 的，方法必须是 public、abstract 的。我们知道接口的设计是一项巨大的工作，因为

接口默认方法
------

在 Java 8 之前，接口中可以申明方法和变量的，只不过变量必须是 public、static、final 的，方法必须是 public、abstract 的。我们知道接口的设计是一项巨大的工作，因为如果我们需要在接口中新增一个方法，需要对它的所有实现类都进行修改，如果它的实现类比较少还可以接受，如果实现类比较多则工作量就比较大了。

为了解决这个问题，Java 8 引入了默认方法，默认方法允许在接口中添加具有默认实现的方法，它使得接口可以包含方法的实现，而不仅仅是抽象方法的定义。

默认方法允许接口在不破坏实现类的情况下进行演进。这对于标准化库的维护和扩展非常重要，因为可以添加新的方法来满足新的需求，而不会影响已存在的实现。同时默认方法使得接口可以通过通用的方法实现，这可以减少代码的重复性，提供了代码的可维护性，是不是有点儿像抽象类了？

默认方法是通过在接口中使用 `default` 关键字来定义的，后面跟着方法的实现。语法如下：

```
interface MyInterface {
    default void myDefaultMethod() {
        
    }
}


```

实现类可以选择性地重写默认方法，如果它们

实现接口的类可以根据自己的需要重写接口的默认方法。当然如果实现类没有提供自己的实现，将使用默认方法的实现。

使用默认方法非常简单，就当做普通的方法调用即可。

```
public interface MyInterface {

    default String defaultMethod() {
       return "MyInterface-defaultMethod";
    }
}


public class MyInterfaceImpl1 implements MyInterface{
}


public class MyInterfaceImpl2 implements MyInterface{
    @Override
    public String defaultMethod() {
        return "MyInterfaceImpl2-defaultMethod";
    }
}

@Test
public void test() {
  MyInterface myInterface1 = new MyInterfaceImpl1();
  MyInterface myInterface2 = new MyInterfaceImpl2();

  System.out.println(myInterface1.defaultMethod());
  System.out.println(myInterface2.defaultMethod());
}

MyInterface-defaultMethod
MyInterfaceImpl2-defaultMethod



```

MyInterfaceImpl1 调用 MyInterface 接口内的 `defaultMethod()`，而 MyInterfaceImpl2 则是调用自己重写的 `defaultMethod()`。

我们知道 Java 是可以实现多个接口的，如果都包含具有相同方法签名的默认方法呢？这种情况，Java 要求我们实现类必须重写该默认方法来解决冲突，否则编译异常。比如再提供一个接口 MyInterface1 ，MyInterfaceImpl1 实现 MyInterface 和 MyInterface1：

```
public interface MyInterface1 {
    default String defaultMethod() {
        return "MyInterface1-defaultMethod";
    }
}

public class MyInterfaceImpl1 implements MyInterface,MyInterface1{
}



```

编译器会提示你有这个错误：

![](https://sike.skjava.com/java-features/202310111000001.jpg)

这个时候就需要 MyInterfaceImpl1 重写 `defaultMethod()`：

```
public class MyInterfaceImpl1 implements MyInterface,MyInterface1{
    @Override
    public String defaultMethod() {
        
        return "MyInterfaceImpl1-defaultMethod()";
    }
}


```

接口静态方法
------

Java 8 引入了接口静态方法，该特性允许在接口中定义静态方法。

接口静态方法允许在接口中定义静态实用性工具类方法，这些方法与接口的功能相关，但是不依赖于实例，这对于提供该接口的通用方法或工具方法非常有用，可以减少代码重复性，提高代码的可维护性。

接口静态方法的使用和我们平常调用工具类一样的。

```
public interface MyInterface {    
    static String staticMethod() {
        return "MyInterface-staticMethod";
    }
}

@Test
public void test() {
  System.out.println(MyInterface.staticMethod());
}



```

需要注意的是我们是无法在实现类重写接口静态方法的。
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/4535468349)

> 从我们一接触接口开始我们就知道 JavaInterface 接口中是不能定义 private 私有方法的，但是这个在 Java9 中被打破了。为了减少代码重复，提高代码的可维护性，Java9 接口支持私有方法，这样

从我们一接触接口开始我们就知道 Java Interface 接口中是不能定义 private 私有方法的，但是这个在 Java 9 中被打破了。

为了减少代码重复，提高代码的可维护性，Java 9 接口支持私有方法，这样做有几个好处：

1.  **接口更好地演化**：在不破坏现有实现的前提下，我们可以向接口中添加默认方法，这是接口演化的一种方式。然后，如果默认方法之间有重复代码，就会导致代码容易重复。在接口中定义私有方法，可以将这些重复的代码抽离出来，使接口更容易维护和扩展。
2.  **代码复用**：将重复的，通用的功能封装在私有方法中，私有方法可以被接口中的其他方法调用，这有助于减少代码重复和提高代码的可维护性。
3.  **防止子类滥用**：由于私有方法只能在接口内部使用，无法在接口的实现类中被滥用，这有助于保持接口的一致性和约定。

在接口中使用私有方法有如下几个限制：

1.  接口私有方法不能是抽象的。
2.  私有方法只能在接口内部使用，无法被接口的实现类或外部类访问。
3.  私有方法不会继承给接口的子接口，每个接口都必须自己定义自己的私有方法。
4.  私有静态方法可以在其他静态和非静态接口方法中使用。
5.  私有非静态方法不能在私有静态方法内部使用。

下面演示下怎么在接口中使用私有方法。

定义接口 `MyInterface`：

```
public interface MyInterface {

    
    void publicMethod();

    
    default void defaultMethod() {
        System.out.println("defaultMethod");

        
        privateMethod();

        
        privateStaticMethod();
    }

    
    private void privateMethod() {
        System.out.println("privateMethod");
    }

    
    static void staticMethod() {
        System.out.println("staticMethod");

        
        privateStaticMethod();
    }

    
    private static void privateStaticMethod() {
        System.out.println("privateStaticMethod");
    }
}


```

实现类：`MyInterfaceImpl`：

```
public class MyInterfaceImpl implements MyInterface {
    @Override
    public void publicMethod() {
        System.out.println("publicMethod");
    }
}


```

测试类：

```
public class Test {
    public static void main(String[] args) {
        
        MyInterface myInterface = new MyInterfaceImpl();
        myInterface.publicMethod();
        
        
        System.out.println("");
        myInterface.defaultMethod();

        
        System.out.println("");
        MyInterface.staticMethod();
    }
}


publicMethod

defaultMethod
privateMethod
privateStaticMethod

staticMethod
privateStaticMethod



```

> 通过引入私有方法，Java 9 增强了接口的功能，使其更加灵活，能够更好地满足不同的编程需求，特别是在接口的演化和维护方面提供了更多的选项。这有助于减少代码重复，提高代码的可维护性，并使接口的设计更加清晰和一致。
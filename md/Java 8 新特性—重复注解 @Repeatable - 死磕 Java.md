> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/8453320489)

> Java8 之前如何使用重复注解在 Java8 之前我们是无法在一个类型重复使用多次同一个注解，比如我们常用的 @PropertySource，如果我们在 Java8 版本以下这样使用：@PropertySour

原

 2023-11-16  阅读 (120)  点赞 (0)

![](https://sike.skjava.com/java-features/202310102000002.png)

Java 8 之前如何使用重复注解
-----------------

在 Java 8 之前我们是无法在一个类型重复使用多次同一个注解，比如我们常用的 `@PropertySource`，如果我们在 Java 8 版本以下这样使用：

```
@PropertySource("classpath:config.properties")
@PropertySource("classpath:application.properties")
public class PropertyTest {
}


```

编译会报错，错误信息是：`Duplicate annotation`。

那怎么解决这个问题呢？在 Java 8 之前想到一个方案来解决 `Duplicate annotation` 错误：新增一个注解 `@PropertySources`，该注解包裹 `@PropertySource`，如下：

```
public @interface PropertySources {

  PropertySource[] value();
}


```

然后就可以利用 `@PropertySources` 来完成了：

```
@PropertySources({
  @PropertySource("classpath:config.properties"),
  @PropertySource("classpath:application.properties")     
})
public class PropertyTest {
}


```

利用这种嵌套的方式来规避重复注解的问题，怎么获取呢？

```
    @Test
    public void  test() {
        PropertySources propertySources = PropertyTest.class.getAnnotation(PropertySources.class);
        for (PropertySource propertySource : propertySources.value()) {
            System.out.println(propertySource.value()[0]);
        }
    }

classpath:config.properties
classpath:application.properties


```

Java 8 重复注解 @Repeatable
-----------------------

通过上述那种方式确实是可以解决重复注解的问题，但是使用有点儿啰嗦，所以 Java 8 为了解决这个问题引入了注解 `@Repeatable` 来解决这个问题。

> `@Repeatable` 注解允许在同一个类型上多次使用相同的注解，它提供了更灵活的注解使用方式。

下面我们来看看如何使用重复注解。

### 如何使用重复注解

> **1、重复注解声明**

在使用重复注解之前，需要在自定义注解类型上使用`@Repeatable`注解，以指定该注解可重复使用的容器注解类型。容器注解类型本身也是一个注解，通常具有一个 value 属性，其值是一个数组，用于存储重复使用的注解。

```
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Repeatable(MyAnnotations.class)        
public @interface MyAnnotation {
    String name() default "";
}


@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface MyAnnotations {
    MyAnnotation[] value();
}



```

> **2、使用重复注解**

定义了重复注解，我们就可以在一个类型上面使用多个相同的注解，如下：

```
@MyAnnotation(name = "死磕 Java 并发")
@MyAnnotation(name = "死磕 Netty")
@MyAnnotation(name = "死磕 Redis")
@MyAnnotation(name = "死磕 Java 基础")
@MyAnnotation(name = "死磕 Redis")
public class MyAnnotationTest {
}


```

> **3、获取重复注解的值**

使用放射获取元素上面的重复注解，由于我们这里有多个所以需要根据 `getAnnotationsByType()` 来获取所有重复注解的数组：

```
    @Test
    public void test() {
        MyAnnotation[] myAnnotations = MyAnnotationTest.class.getAnnotationsByType(MyAnnotation.class);
        for (MyAnnotation myAnnotation : myAnnotations) {
            System.out.println(myAnnotation.name());
        }
    }





```

![](https://sike.skjava.com/java-features/202310102000001.jpg)

我们还可以直接获取它的容器注解：

```
    @Test
    public void test() {
        MyAnnotations myAnnotations = MyAnnotationTest.class.getAnnotation(MyAnnotations.class);
        for (MyAnnotation myAnnotation : myAnnotations.value()) {
            System.out.println(myAnnotation.name());
        }
    }


```

依然可以获取到值。

重复注解很容易就理解了，知道如何自定义注解，然后变换下思路就行了。
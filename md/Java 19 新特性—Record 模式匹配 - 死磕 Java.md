> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/9674439600)

> Record 是 Java14 引入的，它主要目的是提供一种简洁的语法来声明类似数据的小型不可变对象，主要是为了解决长期以来在 Java 中定义纯数据载体类时，代码过于繁琐的问题。它的本质上是一个不可变的、透明

原

 2023-12-23  阅读 (34)  点赞 (0)

Record 是 Java 14 引入的，它主要目的是提供一种简洁的语法来声明类似数据的小型不可变对象，主要是为了解决长期以来在 Java 中定义纯数据载体类时，代码过于繁琐的问题。它的本质上是一个不可变的、透明的数据载体对象，我们可以理解它是一种特殊类型的 Java 类。定义 Record 方法如下：

```
public record User(String name,Integer age) {
}


```

Record 会自带 `getters`、`equals()`、`hashCode()` 和 `toString()` 方法，无需我们手写。

但是我们在使用类型匹配时仍然需要强制转换下，如下：

```
    public void recordTest(Object obj) {
        if (obj instanceof User) {
            User user = (User) obj;
            String name = user.name();
            Integer age = user.age();
        }
    }


```

这种做法无疑是繁琐的。还记得 Java 14 引入的[模式匹配的 instanceof](https://www.skjava.com/series/article/www.skjava.com) 吗？它允许在 `instanceof` 操作符的条件判断中直接定义一个变量，如果对象是指定的类型，这个变量会被初始化为被检查的对象，可以立即使用，而无需额外的类型转换。这不就解决了我们的问题吗？

Java 19 引入模式匹配（Record Patterns），它允许我们通过模式匹配直接提取组件，而不需要先进行强制类型转换后再提取。它需要与 `instanceof` 或 `switch 模式匹配`一同使用。

我们先看 `instanceof`，改造如下：

```
    public void recordTest(Object obj) {
        if (obj instanceof User user) {
            String name = user.name();
            Integer age = user.age();
        }
    }


```

那在 Switch 表达式中呢，如下：

```
    public void recordTest(Object obj) {
        switch (obj) {
            case null -> throw new NullPointerException("不能为空");
            case User user -> System.out.println("name:" + user.name() + "--age:" + user.age());
            default -> obj.toString();
        }
    }


```

我们还可以将 User 的属性直接提取出来，如下：

```
    public void recordTest(Object obj) {
        switch (obj) {
            case null -> throw new NullPointerException("不能为空");
            case User(String name,Integer age) -> System.out.println("name:" + name + "--age:" + age);
            default -> obj.toString();
        }
    }


```

我们再衍生下，`case User(String name,Integer age)` 一定要这么写吗？我们都知道了它是 User 类型了，他的属性类型是不是就是固定的？还记得 Java 10 引入的局部变量类型推断么？我们可不可以用 var 来代替？其实是可以的：

```
    public void recordTest(Object obj) {
        switch (obj) {
            case null -> throw new NullPointerException("不能为空");
            case User(var name,var age) -> System.out.println("name:" + name + "--age:" + age);
            default -> obj.toString();
        }
    }


```

所以，在声明模式变量时，我们并不需要显式地指定类型，用 var 也行，具体的类型由编译器自动推断。

> 推荐阅读

*   [Java 10 新特性—局部变量类型推断](https://www.skjava.com/series/article/1075067951)
*   [Java 14 新特性—新增 Record 类型](https://www.skjava.com/series/article/8932587293)
*   [Java 14 新特性—模式匹配的 instanceof](https://www.skjava.com/series/article/3436282193)
*   [Java 17 新特性—模式匹配的 Switch 表达式](https://www.skjava.com/series/article/9479813794)
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/8932587293)

> 为什么要引入 Record 我相信很多小伙伴对下面的代码一定非常熟悉：publicclassUserDTO{privateStringuserName;privateIntegerage;publicUs

原

 2023-12-20  阅读 (47)  点赞 (0)

为什么要引入 Record
-------------

我相信很多小伙伴对下面的代码一定非常熟悉：

```
public class UserDTO {

    private String userName;
    
    private Integer age;
    
    public UserDTO(String userName,Integer age) {
        this.userName = userName;
        this.age = age;
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "UserDTO{" +
                "user + userName + '\'' +
                ", age=" + age +
                '}';
    }
}



```

允许使用 Lombok 的小伙伴会这么写：

```
@Data
@AllArgsConstructor
public class UserDTO {
    private String userName;
    
    private Integer age;
}


```

允许使用 Lombok，可以简单很多代码量，但是如果不允许呢？我们是不是就要像上面一样，写构造函数、`setter`

`getters`、`equals()`、`hashCode()`、`toString()`，这种方式不仅使代码变得冗长，而且还容易出错，因为必须手动保证各种方法之间的一致性。

> Java 14 引入新特性：Records，它的主要目的是提供一种简洁的语法来声明类似数据的**小型不可变对象**，主要是为了解决长期以来在 Java 中定义纯数据载体类时，代码过于繁琐的问题。

什么是 Record
----------

**Record 本质上是一个不可变的、透明的数据载体对象。** 是一种特殊类型的 Java 类，具有如下几个特点：

*   **不可变性**：Record 的实例是不可变的，所有字段都是 final 的。
*   **数据驱动**：Record 主要用于封装一组数据，字段在对象构造时通过构造器参数传递并初始化。
*   **简洁性**：自动生成所有字段的 getters 方法，这些方法名称与字段名称相同。
*   **透明性**：自动实现了 `equals()`、`hashCode()` 和 `toString()` 方法，它们基于 Record 的状态，即其字段的值。

UserDTO 使用 `Record` 定义如下：

```
record UserDTO(String userName,Integer age) { }


```

这将自动为 UserDTO 生成一个带有两个字段、一个全参构造器、所有字段的 `getters`、`equals()`、`hashCode()`、`toString()` 方法的类。是不是超级方便，但是需要注意的是该类的实例是一个不可变的。

所以引入 Record 会有如下几个好处：

1.  **减少样板代码**：不需要手动编写 `getters`、`equals()`、`hashCode()`、`toString()` 方法。
2.  **增加代码清晰度**：它清晰地表达了这个类型仅用于数据传输的意图。
3.  **提高开发效率**：我们可以快速定义数据对象，而不必关心常规的、重复性的方法实现。
4.  **不可变性**：由于 Record 是不可变的，它天生就是线程安全的。

Record 非常适合作为在各层间传递的数据载体，比如 DTO（数据传输对象），且由于它是不可变的，在并发变成中可以作为共享数据，来确保线程安全。但是在 **Java 14 中是预览特性**。

参考文章：[https://www.toutiao.com/article/7136183008918602255](https://www.toutiao.com/article/7136183008918602255) 重写
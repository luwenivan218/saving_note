> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1810194665)

> NullPointerException 是我们最常见也是最烦的异常处理，它非常常见，处理起来有很简单，但是你又不得不去处理，超级烦。引言我们先看一个简单的例子：@DatapublicclassUser

`NullPointerException` 是我们最常见也是最烦的异常处理，它非常常见，处理起来有很简单，但是你又不得不去处理，超级烦。

![](https://sike.skjava.com/java-features/202310071000002.png)

引言
--

我们先看一个简单的例子：

```
@Data
public class User {
    private String name;

    private Address address;
}

@Data
public class Address {
    private String province;

    private String city;

    private String area;
}



```

如果我们需要获取用户所在城市，我们会这么写：

```
    public static String getUserCity(User user) {
        return user.getAddress().getCity();
    }
    
    String city = getUserCity(user);


```

这种写法有可能会报 `NullPointerException`，因为 user 可能为 null，user.getAddress() 也有可能为 null，所以为了解决这个问题 ，我们会采用这种写法：

```
    public static String getUserCity(User user) {
        if (user != null) {
            Address address = user.getAddress();
            if (address != null) {
                return address.getCity();
            }
        }
        return null;
    }


```

就问这种写法丑不丑？繁琐不繁琐？为了避免这种丑陋的写法，让丑陋的设计变得优雅，Java 8 提供了 Optional。

Optional 是什么
------------

Optional 是 Java 8 提供了一个类库。被设计出来的目的是为了减少因为`null`而引发的`NullPointerException`异常，并提供更安全和优雅的处理方式。

Java 中臭名昭著的 `NullPointerException` 是导致 Java 应用程序失败最常见的原因，没有之一，没有一个 Java 开发程序员没有遇到这个异常。为了解决 `NullPointerException`，Google Guava 引入了 Optional 类，它提供了一种在处理可能为`null`值时更灵活和优雅的方式，受 Google Guava 的影响，Java 8 引入 Optional 来处理 null 值。

在 Javadoc 中是这样描述它的：**一个可以为 null 的容器对象**。所以 `java.util.Optional<T>` 是一个容器类，它可以保存类型为 T 的值，T 可以是实际 Java 对象，也可以是 null。

API 介绍
------

我们先看 Optional 的定义：

```
public final class Optional<T> {

    
    private final T value;
    
    
 }


```

从这里可以看出，Optional 的本质就是内部存储了一个真实的值 T，如果 T 非空，就为该值，如果为空，则表示该值不存在。

### **构造 Optional 对象**

Optional 的构造函数是 private 权限的，它对外提供了三个方法用于构造 Optional 对象。

> `Optional.of(T value)`

```
    public static <T> Optional<T> of(T value) {
        return new Optional<>(value);
    }
    
    private Optional(T value) {
        this.value = Objects.requireNonNull(value);
    }


```

所以 `Optional.of(T value)` 是创建一个包含非 null 值的 Optional 对象。如果传入的值为`null`，将抛出`NullPointerException` 异常信息。

> `Optional.ofNullable(T value)`

```
    public static <T> Optional<T> ofNullable(T value) {
        return value == null ? empty() : of(value);
    }


```

创建一个包含可能为 null 值的 Optional 对象。如果传入的值为 null，则会创建一个空的 Optional 对象。

> `Optional.empty()`

```
    public static<T> Optional<T> empty() {
        @SuppressWarnings("unchecked")
        Optional<T> t = (Optional<T>) EMPTY;
        return t;
    }
    
    private static final Optional<?> EMPTY = new Optional<>();


```

创建一个空的`Optional`对象，表示没有值。

### **检查是否有值**

Optional 提供了两个方法用来检查是否有值。

> `isPresent()`

`isPresent()` 用于检查 Optional 对象是否包含一个非 null 值，源码如下：

```
    public boolean isPresent() {
        return value != null;
    }


```

示例如下：

```
User user = null;
Optional<User> optional = Optional.ofNullable(user);
System.out.println(optional.isPresent());

false


```

> `ifPresent(Consumer<? super T> action)`

该方法用来执行一个操作，该操作只有在 Optional 包含非 null 值时才会执行。源码如下：

```
    public void ifPresent(Consumer<? super T> consumer) {
        if (value != null)
            consumer.accept(value);
    }


```

需要注意的是，这是 Consumer，是没有返回值的。

示例如下：

```
User user = new User("xiaoming");
Optional.ofNullable(user).ifPresent(value-> System.out.println("名字是:" + value.getName()));


```

### 获取值

获取值是 Optional 中的核心 API，Optional 为该功能提供了四个方法。

> `get()`

`get()` 用来获取 Optional 对象中的值。如果 Optional 对象的值为空，会抛出`NoSuchElementException`异常。源码如下：

```
    public T get() {
        if (value == null) {
            throw new NoSuchElementException("No value present");
        }
        return value;
    }


```

> `orElse(T other)`

`orElse()` 用来获取 Optional 对象中的值，如果值为空，则返回指定的默认值。源码如下：

```
    public T orElse(T other) {
        return value != null ? value : other;
    }


```

示例如下：

```
User user = null;
user = Optional.ofNullable(user).orElse(new User("xiaohong"));
System.out.println(user);

User(name=xiaohong, address=null)



```

> `orElseGet(Supplier<? extends T> other)`

`orElseGet()`用来获取 Optional 对象中的值，如果值为空，则通过 Supplier 提供的逻辑来生成默认值。源码如下：

```
    public T orElseGet(Supplier<? extends T> other) {
        return value != null ? value : other.get();
    }


```

示例如下：

```
User user = null;
user = Optional.ofNullable(user).orElseGet(() -> {
  Address address = new Address("湖南省","长沙市","岳麓区");
  return new User("xiaohong",address);
});
System.out.println(user);

User(name=xiaohong, address=Address(province=湖南省, city=长沙市, area=岳麓区))



```

`orElseGet()` 和 `orElse()`的区别是：当 T 不为 null 的时候，`orElse()` 依然执行 other 的部分代码，而 `orElseGet()` 不会，验证如下：

```
public class OptionalTest {

    public static void main(String[] args) {
        User user = new User("xiaoming");
        User user1 = Optional.ofNullable(user).orElse(createUser());
        System.out.println(user);

        System.out.println("=========================");

        User user2 = Optional.ofNullable(user).orElseGet(() -> createUser());
        System.out.println(user2);
    }

    public static User createUser() {
        System.out.println("执行了 createUser() 方法");
        Address address = new Address("湖南省","长沙市","岳麓区");
        return new User("xiaohong",address);
    }
}


```

执行结果如下：

![](https://sike.skjava.com/java-features/202310071000001.jpg)

是不是 `orElse()` 执行了 `createUser()` ，而 `orElseGet()` 没有执行？一般而言，`orElseGet()` 比 `orElse()` 会更加灵活些。

> `orElseThrow(Supplier<? extends X> exceptionSupplier)`

`orElseThrow()` 用来获取 Optional 对象中的值，如果值为空，则通过 Supplier 提供的逻辑抛出异常。源码如下：

```
    public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
        if (value != null) {
            return value;
        } else {
            throw exceptionSupplier.get();
        }
    }


```

示例如下：

```
User user = null;
user = Optional.ofNullable(user).orElseThrow(() -> new RuntimeException("用户不存在"));


```

### 类型转换

Optional 提供 `map()` 和 `flatMap()` 用来进行类型转换。

> `map(Function<? super T, ? extends U> mapper)`

`map()` 允许我们对 Optional 对象中的值进行转换，并将结果包装在一个新的 Optional 对象中。该方法接受一个 Function 函数，该函数将当前 Optional 对象中的值映射成另一种类型的值，并返回一个新的 Optional 对应，这个新的 Optional 对象中的值就是映射后的值。如果当前 Optional 对象的值为空，则返回一个空的 Optional 对象，且 Function 不会执行，源码如下：

```
    public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Optional.ofNullable(mapper.apply(value));
        }
    }


```

比如我们要获取 User 对象中的 name，如下：

```
User user = new User("xiaolan");
String name = Optional.ofNullable(user).map(value -> value.getName()).get();
System.out.println(name);

xiaolan



```

> `Function<? super T, Optional<U>> mapper`

`flatMap()` 与 `map()` 相似，不同之处在于 `flatMap()`的映射函数返回的是一个 Optional 对象而不是直接的值，它是将当前 Optional 对象映射为另外一个 Optional 对象。

```
    public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Objects.requireNonNull(mapper.apply(value));
        }
    }


```

上面获取 name 的代码如下：

```
String name = Optional.ofNullable(user).flatMap(value -> Optional.ofNullable(value.getName())).get();


```

`flatMap()` 内部需要再次封装一个 Optional 对象，所以 `flatMap()` 通常用于在一系列操作中处理嵌套的`Optional`对象，以避免层层嵌套的情况，使代码更加清晰和简洁。

### 过滤

Optional 提供了 `filter()` 用于在 Optional 对象中的值满足特定条件时进行过滤操作，源码如下：

```
    public Optional<T> filter(Predicate<? super T> predicate) {
        Objects.requireNonNull(predicate);
        if (!isPresent())
            return this;
        else
            return predicate.test(value) ? this : empty();
    }


```

`filter()` 接受 一个 Predicate 来对 Optional 中包含的值进行过滤，如果满足条件，那么还是返回这个 Optional；否则返回 `Optional.empty`。

实战应用
----

这里大明哥利用 Optional 的 API 举几个例子。

*   示例一

Java 8 以前：

```
    public static String getUserCity(User user) {
        if (user != null) {
            Address address = user.getAddress();
            if (address != null) {
                return address.getCity();
            }
        }
        return null;
    }


```

常规点的，笨点的方法：

```
    public static String getUserCity(User user) {
        Optional<User> userOptional = Optional.of(user);
        return Optional.of(userOptional.get().getAddress()).get().getCity();
    }


```

高级一点的：

```
    public static String getUserCity(User user) {
        return Optional.ofNullable(user)
                .map(User::getAddress)
                .map(Address::getCity)
                .orElseThrow(() -> new RuntimeException("值不存在"));
    }


```

是不是比上面高级多了？

*   示例二

比如我们要获取末尾为 "ming" 的用户的 city，不是的统一返回 "深圳市"。

Java 8 以前

```
    public static String getUserCity(User user) {
        if (user != null && user.getName() != null) {
            if (user.getName().endsWith("ming")) {
                Address address = user.getAddress();
                if (address != null) {
                    return address.getCity();
                } else {
                    return "深圳市";
                }
            } else {
                return "深圳市";
            }
        }

        return "深圳市";
    }


```

Java 8

```
    public static String getUserCity2(User user) {
        return Optional.ofNullable(user)
                .filter(u -> u.getName().endsWith("ming"))
                .map(User::getAddress)
                .map(Address::getCity)
                .orElse("深圳市1");
    }


```

这种写法确实是优雅了很多。其余的例子大明哥就不一一举例了，这个也没有其他技巧，唯手熟尔！！
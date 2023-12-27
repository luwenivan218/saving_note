> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1019158498)

> 在前面我们了解了 Lambda 表达式，它能够简化我们的程序，但是它还不是最简单的，Java8 引入了方法引用可以对 Lambda 表达式再进一步简化。什么是方法引用我们先看一个例子。首先定义一个 Student

在前面我们了解了 [Lambda 表达式](https://www.skjava.com/series/article/1747461365)，它能够简化我们的程序，但是它还不是最简单的，Java 8 引入了方法引用可以对 Lambda 表达式再进一步简化。

![](https://sike.skjava.com/java-features/202310051000003.png)

什么是方法引用
-------

我们先看一个例子。首先定义一个 Student 类：

```
public class Student {
    private String name;

    private Integer age;

    public static int compareByAge(Student a,Student b) {
        return a.getAge().compareTo(b.getAge());
    }
}


```

Student 中含有一个静态方法 `compareByAge()`，它是用来比较年龄的。

现在需要实现一个需求，有一批学生我们希望能够根据 age 进行排序。

在没有学习 Lambda 表达式时，我们这样写：

```
public class MethodReferenceTest {
    public static void main(String[] args) {
        List<Student> studentList = Arrays.asList(
                new Student("小明",16),
                new Student("小红",14),
                new Student("小兰",15),
                new Student("小李",18),
                new Student("小张",14),
                new Student("小林",15)
        );

        Collections.sort(studentList, new Comparator<Student>() {
            @Override
            public int compare(Student o1, Student o2) {
                return o1.getAge().compareTo(o2.getAge());
            }
        });
        System.out.println(studentList);
    }
}


```

学习了 Lambda 表达式后，我们知道 Comparator 接口是一个函数式接口，因此我们可以使用 Lambda 表达式，而不需要使用这种匿名内部类的方式：

```
public class MethodReferenceTest {
    public static void main(String[] args) {
        

        Collections.sort(studentList, (o1,o2) -> Student.compareByAge(o1,o2));
        System.out.println(studentList);
    }
}


```

注意，这里我们是使用 Student 类中的静态方法：`compareByAge()`。到这里后其实还有进一步的优化空间：

```
public class MethodReferenceTest {
    public static void main(String[] args) {
        

        Collections.sort(studentList, Student::compareByAge);
        System.out.println(studentList);
    }
}


```

这段代码将 Lambda 表达式 `(o1,o2) -> Student.compareByAge(o1,o2)` 转变为了 `Student::compareByAge` 是不是很懵逼？

`Student::compareByAge` 写法就是我们这篇文章要讲的方法引用。那什么是方法引用呢？

> 方法引用是 Java 8 引入的特性，它提供了一种更加简洁的可用作 Lambda 表达式的表达方式。 定义：**方法引用是用来直接访问类或者实例的已经存在的方法或者构造方法。**

我们可以简单认为，方法引用是一种更加简洁易懂的 Lambda 表达式。当 Lambda 表达式的主体中只有一个执行方法的调用时，我们可以不使用 Lambda 表达式，而是选择更加简洁的方法引用，这样可读性更高一些。

三种方法引用类型
--------

方法引用的标准格式是：`类名::方法名`。它有如下三种类型：

<table><thead><tr><th><strong>类型</strong></th><th><strong>格式</strong></th></tr></thead><tbody><tr><td>引用静态方法</td><td><code>类名::静态方法名</code></td></tr><tr><td>引用对象的实例方法</td><td><code>实例对象::方法名</code></td></tr><tr><td>引用类型的任意对象的实例方法</td><td><code>类名::实例方法名</code></td></tr></tbody></table>

下面我们来看这三种类型的使用方法。

### 引用静态方法

引用静态方法的格式是：`类名::静态方法名`。这个是其实和我们使用静态方法一样，只不过是将 `“.”` 替换成了 `“::”`。其实我们上面那个例子就是引用静态方法的例子，这里大明哥再举一个示例，`java.lang.Math` 中有很多静态方法，比如：

```
Function<Integer,Integer> function1 = t -> Math.abs(t);
int result1 = function1.apply(-123);
        

Function<Integer,Integer> function2 = Math::abs;
int result2 = function2.apply(-123); 


```

### 引用对象的实例方法

引用对象的实力方法格式是：`实例对象名::实例方法名`，这种方式引用的是一个实例方法，所以需要提供一个对象实例来引用，如下：

```
Student student = new Student("小明",15);


Supplier<String> supplier1 = () -> student.getName();
String name1 = supplier1.get();
 

Supplier<String> supplier2 = student::getName;
String name2 = supplier2.get();


```

这种方式在我们使用 Stream 来操作集合时用得非常多。

### 引用类型的任意对象的实例方法

引用类型的任意对象的实例方法的格式是：`类名::实例方法名`，这个有点儿不是很好理解。这种引用方式引用的是一个特定对象的实例方法，通常在函数式接口中作为第一个参数传递给方法引用，怎么理解呢？我们看下面两个例子：

比如 Comparator 中的 `int compare(T o1, T o2)`，我们需要比较两个字符串的大小，使用方式如下：

```
Comparator<String> comparator = (o1,o2) -> o1.compareTo(o2);
System.out.println(comparator.compare("sike","sk"));


```

改成 `类名::实例方法名` 怎么改呢？

```
Comparator<String> comparator = String::compareTo;
System.out.println(comparator.compare("sike","sk"));


```

是不是比较懵逼？再看一个：

```
List<String> list = Arrays.asList("xiaoming","xiaohong","xiaoli","xiaodu");
list.forEach(name -> name.toUpperCase());

List<String> list = Arrays.asList("xiaoming","xiaohong","xiaoli","xiaodu");
list.forEach(String::toUpperCase);



```

是不是比较懵？其实大明哥看到这个也比较懵，确实是不好理解，但是没关系，最后面大明哥教你们一个终极神器，让你使用方法引用不再困难。

方法引用的前提条件
---------

方法引用确实可以极大地降低我们的代码量也更加清晰了，但是并不是所有的 Lambda 表达式都可以转换为方法引用。它有如下几个前提条件。

> **1、Lambda 表达式中只有一个调用方法的代码**

注意这个**一个调用方法**的含义，它包含两重意思。

*   只有一行代码

```
List<String> list = Arrays.asList("xiaoming","xiaohong","xiaoli","xiaodu");
list.forEach(name -> {
    System.out.println("www.skjava.com");
    name.toUpperCase();
});


```

这个 Lambda 中有两行代码，这是无法转换为方法引用的。

*   只有一个调用方法

```
List<String> list = Arrays.asList("xiaoming","xiaohong","xiaoli","xiaodu");
list.forEach(name -> System.out.println(name.toUpperCase()));


```

这种写法也是转换不了的，虽然只有一行代码，但是它调用了两个方法。

> **2、方法引用的目标方法必须与 Lambda 表达式的函数接口的抽象方法参数类型和返回类型相匹配**

这就意味着目标方法的参数数量、类型以及返回类型必须与函数接口的要求一致。但是它只能规范**引用静态方法**和**引用对象的实例方法**，而**引用类型的任意对象的实例方法**这种类型其实是不适用。

> **3、如果方法引用是通过对象引用来实现的，那么 Lambda 表达式中的参数列表的第一个参数必须是方法引用的目标方法的隐式参数，而其余参数（如果有的话）必须与方法引用的目标方法的参数一致。**

比如：

```
BiConsumer<Student,Integer> consumer = (stu,age) -> stu.setAge(age);
改成
BiConsumer<Student,Integer> consumer = Student::setAge;



```

Lambda 表达式有两个参数 `(stu,age)`，第一个参数 `stu` 是目标方法 `setAge()` 的隐式参数，其余参数 （`age`）与方法引用的目标方法 （`setAge(Integer age)` ）的参数 `(Integer age)` 是一致的。这种就可以改写。

又如：

```
Comparator<String> comparator = (o1,o2) -> o1.compareTo(o2);
改为
Comparator<String> comparator = String::compareTo;


```

方法引用简单是简单，就是不好理解，尤其是 `类名::实例方法` 格式的，直接会让人懵逼，还有我们有终极神器。

方法引用的终极神器
---------

这个终极神器其实就是 idea。idea 不管是对于 Lambda 表达式还是方法引用其实都是有提示的，例如：

![](https://sike.skjava.com/java-features/202310051000001.jpg)

idea 会直接提示你该 Lambda 表达式可以简化为 `String::compareTo`，是不是很给力。再如：

![](https://sike.skjava.com/java-features/202310051000002.jpg)

直接提示你可以简化为 Lambda 表达式。所以**工欲善其事必先利其器**。

构造器引用
-----

构造器引用提供了一种更加简介的方式来创建对象，语法格式是 ：`类::new`。调用哪个构造器取决于函数式接口中的方法形参的定义，Lambda 表达式会自动根据接口方法推断出你要调用的构造器。

*   调用无参构造器

```
Supplier<Student> supplier = () -> new Student();

Supplier<Student> supplier = Student::new;



```

*   调用有参构造器

例如：

```
Function<String,Student> function = name -> new Student(name);

Function<String,Student> function = Student::new;



```

这个是调用的构造器为：

```
    public Student(String name) {
        this.name = name;
    }  


```

再如：

```
BiFunction<String,Integer,Student> function = (name,age) -> new Student(name,age);

BiFunction<String,Integer,Student> function = Student::new;


```

到这里各位小伙伴应该明白是怎么回事了吧？但是这里有一个漏洞，因为 Function 只有一个参数，所以它只支持带有一个参数的构造器，BiFunction 有两个参数，所以它只支持带有两个参数的构造器，如果我的 Student 有四个属性呢？怎么办？**自定义函数式接口**。

```
@Data
@AllArgsConstructor
public class Student {
    private String name;

    private Integer age;

    private String birthday;

    private String className;
}


```

我们需要自定义一个函数式接口，它需要有四个参数，一个返回值，如下：

```
@FunctionalInterface
public interface FunctionInterface<T,U,O,P,R> {
    R apply(T t, U u,O o,P p);
}


```

然后就可以利用构造器引用来构造 Student 对象了：

```
FunctionInterface<String,Integer,String,String,Student> functionInterface = Student::new;
System.out.println(functionInterface.apply("xiaoming",8,"06-19","二年三班"));


```

这种方式确实是简单了，但是没有必要为了多个参数来自定义一个函数式接口。在实际项目过程中我觉得还不如 `new Student` 来的直接明了。

数组引用
----

数组引用和构造器引用的语法格式一样，`Type[]::new`。`Type` 是数组元素的类型，后面的`::new`表示引用该类型的数组构造方法来创建新数组。例如：

```
 Function<Integer, int[]> function = int[]::new;
 int[] arrays = function.apply(5);


```

创建一个包含 5 个整数的一维数组。对于多维数组，大明哥其实不是很建议使用这种方式，因为有点儿鸡肋，多维的数组内容还是需要我们处理。
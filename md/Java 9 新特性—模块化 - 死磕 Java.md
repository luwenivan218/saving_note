> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/2044128092)

> Java9 中最大的特性毫无疑问就是模块化，其实模块化的概念在 Java7 的时候就已经提出来了，由于它的复杂性，不断跳票，从 Java7 到 Java8，最后 Java9 终于姗姗来迟，它的出现犹如壮士断腕。那模块

Java 9 中最大的特性毫无疑问就是模块化，其实模块化的概念在 Java 7 的时候就已经提出来了，由于它的复杂性，不断跳票，从 Java 7 到 Java 8 ，最后 Java 9 终于姗姗来迟，它的出现犹如壮士断腕。

那模块化到底是什么呢？在实际开发中又有什么用呢？这篇文章大明哥带你彻底了解 Java 9 的模块化。

![](https://sike.skjava.com/java-features/202310291000007.png)

什么是模块化
------

模块是 Java 9 中新增的一个组件，官方是这么定义它的：** 一个被命名的，代码和数据的自描述集合。（ the module, which is a named, self-describing collection of code and data）。** 怎么理解呢？我们可以简单地将它理解为 `package` 的上一级单位，是多个 `package` 的集合。

我们知道在 `Java` 中， `Java` 文件是最小的可执行文件，为了更好地管理这些 `Java` 文件，我们需要用 `package` 将同一类的 `Java` 文件统一管理起来，多个 `package` 文件、`Java` 文件可以打包成一个 `jar` 文件，现在 Java 9 在 `package` 上面增加 `module`，一个 `module` 可以包含多个 `package`，所以从代码结构上来看层级关系是这样的：`jar > module > package > java 文件`。

所以，从本质上来说，模块就是用来管理各个 `package` 的组件，它的概念，其实就可以理解为在 `package` 上面再包一层，包这一层的主要目的是让我们能够更好地组织和管理 `Java` 应用程序的代码，以及更好地控制代码的可见性和依赖关系。

要掌握模块化，就需要理解它的几个核心概念：

1.  模块（`Module`）：模块是模块化系统的基本单元。它是一个逻辑上独立的代码单元，包括类、接口、资源和`module-info.java`文件。每个模块都有一个唯一的名称，例如："`java.base`"、"`com.example.myapp`" 等。
2.  模块路径（`Module Path`）：模块路径是一组包含模块的路径，用于在运行时指定应用程序所需的模块。类似于类路径，但它是用于模块。
3.  `module-info.java` 文件：每个模块都包含一个特殊的文件，名为`module-info.java`。这个文件描述了模块的信息，包括模块名称、依赖关系、导出的包以及其他模块信息。
4.  模块依赖性（`Module Dependencies`）：在`module-info.java`文件中，可以使用`requires`关键字声明模块之间的依赖关系。
5.  模块导出（`Module Exporting`）：在`module-info.java`文件中，可以使用 `exports` 关键字声明哪些包可以被其他模块访问，这有助于控制包的可见性。

模块化怎么体现的呢？下图是 Java 8 与 Java 9 的目录结构：

![](https://sike.skjava.com/java-features/202310291000001.png)

![](https://sike.skjava.com/java-features/202310291000002.png)

从上图中你会发现 Java 9 中没有`jre`，没有`rt.jar`，没有`tools.jar`，而是多了一个 `jmods`，该文件夹下都是一个一个的模块：

![](https://sike.skjava.com/java-features/202310291000003.jpg)

对于 Java 9 之前的工程，他们都是单体模式，一个简单的 hello world，都需要引入 `rt.jar`，导致这个简单的 hello world 的 jar 变得很大，而 Java 9 引入模块后，它只需要引入它所依赖的即可。

为什么需要模块化
--------

在 Java 9 之前我们没有使用模块化之前用起来还是很顺手的，现在突然在 `package` 上面增加一层 `module`，势必会增加我们编码的复杂度，既然增加了复杂度为什么还要引入呢？其实引入模块化有着几个非常好的优势。

> **1、显式管理依赖**

模块化系统需要我们明确申请模块之间的依赖关系，它减少了传统类路径（classpath）上的混乱和不稳定性。每个模块都需要显示声明自己需暴露的 `package`，而自己所依赖的和自己内部使用的 package，则不会暴露，也不会被外部依赖，这有助于保护内部实现，防止不应该公开的部分被外部模块访问。依赖的模块也需要显示引入需要依赖的 `package`。

> **2、更好地安全性**

模块化系统可以提供更严格的可见性控制，防止私有实现被不应访问的模块访问，从而增强了应用程序的安全性。代码真正意义上可以按照作者的设计思路进行公开和隐藏，同时也限制了反射的滥用，更好的保护了那些不建议被外部直接使用或过时的内部类。

> **3、标准化**

模块化引入了标准化的方式来组织和管理代码。显示的声明暴露的内容，可以让第三方库的开发者更好地管理自己的内部实现逻辑和内部类。

> **4、自定义最小运行时映像**

Java 因为其向后兼容的原则，不会轻易对其内容进行删除，包含的陈旧过时的技术也越来越多，导致`JDK`变得越来越臃肿。而 Java 9 的显示依赖管理使得加载最小所需模块成为了可能，我们可以选择只加载必须的`JDK`模块，抛弃如`java.awt`, `javax.swing`, `java.applet`等这些用不到的模块。这种机制，大大的减少了运行`Java`环境所需要的内存资源，在对于嵌入式系统开发或其他硬件资源受限的场景下的开发非常有用。

> **5、更加适合大型应用程序管理**

于大型应用程序，模块化系统提供更好的组织结构，减少了复杂性，使开发者能够更轻松地管理和扩展应用程序。

> **6、更好的性能**

通过减少不必要的类路径搜索和提供更紧凑的部署单元，模块化系统有助于提高应用程序的性能。

怎么用模块化
------

使用 Java 9 的模块化主要分为以下几个步骤。

> **1、创建模块**

创建一个 `module`，`module` 下面包含该模块的代码和 `module-info.java`文件。`module-info.java`文件是每个模块的关键组成部分，它描述了模块的信息，包括名称、依赖关系和导出的包。

这里我们新建两个 `module`：`java-module-01` 和 `java-module-02`，同时 `java-module-01` 依赖 `java-module-02`，如下：

![](https://sike.skjava.com/java-features/202310291000004.jpg)

这里可能有小伙伴不知道怎么新建 `module-info.java`，其实只需要在 `java` 下面右键 `new` 就可以了：

![](https://sike.skjava.com/java-features/202310291000005.jpg)

如果这里没有，则表示你 `module` 的 `jdk` 没有配置好，配置下就可以了：

![](https://sike.skjava.com/java-features/202310291000006.jpg)

> **2、定义模块信息**

新建模块后，我们就需要在 `module-info.java` 中定义模块信息了，信息主要包括如下几个部分：

1.  使用`module`关键字定义模块，并指定模块的名称，例如：`module java.module01 { }`。
2.  使用`requires`关键字声明模块之间的依赖关系，例如：`requires java.sql;` 表示模块依赖于`java.sql`模块。
3.  使用`exports`关键字声明模块中哪些包可以被其他模块访问，例如：`exports com.skjava.module01.entity;` 表示导出`com.skjava.module01.entity`包。

我们在 `java-module-02` 定义了 `com.skjava.module02.entity` 和 `com.skjava.module02.service` 两个包，同时将 `com.skjava.module02.entity` 暴露出去：

```
module java.module02 {
    exports com.skjava.module02.entity;
}


```

在 `entity package` 新建 `UserEntity` 类，`service package` 中新建 `UserService` 接口：

```
public class UserEntity {
    private String userName;

    private Integer age;

    public UserEntity(String userName,Integer age) {
        this.userName = userName;
        this.age = age;
    }

}


```

如果我们在 `java-module-01` 中不申请引入模块 `java.module02`，我们是无法使用 `UserEntity` 这个类的。所以我们在 `java-module-01` 中的 `module-info.java` 引入 `java.module02`：

```
module java.module01 {
    requires java.module02;
}


```

这个时候我们就可以放心地在 `java-module-01` 中使用 `com.skjava.module02.entity` 的内容了：

```
import com.skjava.module02.entity.UserEntity;

public class UserService {
    public static void main(String[] args) {
        UserEntity user = new UserEntity("大明哥",18);
        System.out.println(user);
    }
}


```

看 UserEntity 导入的包是不是 `com.skjava.module02.entity`。那可以使用 `com.skjava.module02.service` 中的 UserService 呢？不可以，因为 `java-module-02` 并没有将 `com.skjava.module02.service` 暴露出去。

这里只是阐述一种比较简单的使用方式，在实际项目中使用情况会更加复杂，这些都需要我们在工作过程中不断地去探索。
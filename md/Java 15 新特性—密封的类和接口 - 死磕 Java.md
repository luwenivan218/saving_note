> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1777232038)

> 引言继承，作为面向对象语言的三大特性之一，我相信没有小伙伴不知道吧？在工作过程中我们也经常使用继承，我么知道子类继承父类，可以重写父类的方法，编写自己独特的属性与行为，任何依赖父类的业务，子类都可以替

原

 2023-12-20  阅读 (36)  点赞 (0)

引言
--

继承，作为面向对象语言的三大特性之一，我相信没有小伙伴不知道吧？在工作过程中我们也经常使用继承，我么知道子类继承父类，可以重写父类的方法，编写自己独特的属性与行为，任何依赖父类的业务，子类都可以替换掉它。这种情况再绝大多数情况下是非常有价值的，除了少数情况。

> 这少数情况就是：我们需要继承，但是我们又期望能够限制它继承的能力。

是不是很矛盾？我们以加密算法为样例来说明。

我们知道在加密算法的设计中，确保算法的实现不被随意修改或扩展是至关重要的。加入你负责一个内部通信框架，报文都需要加解密。你设计了一套通用的加密算法，其中包含几种标准的加密算法，如 AES、DES 和 RSA，如下：

```
public abstract class Encryptor {
    public abstract byte[] encrypt(byte[] data);
    public abstract byte[] decrypt(byte[] data);
}

public class AesEncryptor extends Encryptor{
    
}

public class DesEncryptor extends Encryptor{
    
}

public class RsaEncryptor extends Encryptor{
    
}




```

你提供给团队使用，但是有一些头铁的团队觉得加密没有必要，或者自己为了 KPI 实现了一套自己的加密算法，通过继承的方式来绕过你框架的加密算法，首先在代码上是检测不出来有问题的，但是他有安全隐患。

怎么杜绝这种情况呢？在 Java 15 之前我们有如下方案：

1.  定义 final 修饰类，这样类就无法被继承了。
2.  `package-private`类（非 public 类），可以控制只能被同一个包下的类继承
3.  依赖沟通与实际的非代码的约束

方案 3 不靠谱，方案 1 和 2 限制的粒度都非常粗，如果我们有更精细化的限制需求的话，就是很难实现了。

密封类
---

为了进一步增强继承的限制能力，Java 15 引入密封类来精确控制类的继承问题 ，目前版本为预览特性。

### 什么是密封类

密封类的主要目的是提供一种更加精确地控制类继承的方法，通过这种方式，类的设计者可以指定一个类它能够被哪些类继承，它增强了类的封装性和安全性。由于密封类限制了类的继承，所以它使得代码更加可预测和易于维护。

*   密封类用 `sealed` 修饰，则它的所有子类都必须在同一个模块或者包内，并且这些子类必须被显式地声明为该密封类的直接子类。
*   密封类的子类可以被声明为`non-sealed`（非密封的）或`final`（最终的）。`non-sealed`的子类可以被进一步继承，而`final`的子类则不能。
*   密封类使用 `permits` 来指定它的子类。

> 与密封类相似，接口也可以被声明为密封的，从而限制哪些其他接口或类可以实现或扩展它。

### 示例代码

假设我们要设计一个游戏，该游戏有三类英雄：战士、法师、射手。按照传统的继承方式就很简单了，大明哥就不演示了，这里我们直接看如何利用密封类来控制我们的设计思路。

游戏初期，我们游戏有且只有这三类英雄，我们这样设计：

```
public class Hero {
}


public class Warrior extends Hero{
}


public class Archer extends Hero{
}


public class Mage extends Hero{
}



```

类的结构如下：

![](https://sike.skjava.com/java-features/202311212000001.png)

开始时，这种写法是没有问题的。但是随着版本的迭代， 游戏英雄需要改造，整体还是分为战士、法师、射手三类，每个类别下面有三个英雄：

*   战士：烈焰勇士（FlameWarrior）、铁甲巨人（IronTitan）、疾风剑士（Swiftblade）
*   射手：寒冰射手（IceArcher）、风暴射手（StormShooter）、影夜射手（ShadowNightArcher）
*   法师：元素法师（ElementMage）、虚空巫师（VoidSorcerer）、光明祭司（LightPriest）

由于英雄类型只有三类，所以我们的 `Hero` 一定要限制，为什么？因为团队技术能力有高有低，总有不怕死的人会用寒冰射手继承 `Hero`。所以我们代码如下。

*   `Hero` 为基类，只允许 `Warrior` 、`Arche`、`Mage` 继承。所以 `Hero` 需要用 sealed 修饰，同时利用 permits 来指定子类为：`Warrior` 、`Arche`、`Mage`，如下：

```
public sealed class Hero permits Warrior,Mage,Archer {
}


```

*   二层英雄类型，他们还需要子类继承，我们同样不希望他的继承被扩散，所以继续使用 `sealed` 修饰

```
public sealed class Warrior extends Hero permits FlameWarrior,IronTitan,Swiftblade{
}


public sealed class Mage extends Hero permits IceArcher,StormShooter,ShadowNightArcher{
}


public sealed class Archer extends Hero permits ElementMage,VoidSorcerer,LightPriest{
}



```

第二层结构就很稳定了。

*   三层为具体英雄，他们继承二层的英雄类型，使用 extends 继承即可，同时需要表示为`non-sealed`或`final`，由于我们不希望类再往下了，所以定义为 `final`：

```
public final class FlameWarrior extends Warrior{
}

public final class IronTitan extends Warrior{
}

public final class Swiftblade extends Warrior{
}




```

整体结构如下：

![](https://sike.skjava.com/java-features/202311212000002.png)

结构是一个非常稳定的三层结构，没有任何类能够继承里面的任一一个类。通过这样的设计，就将这三层保护得非常好了。
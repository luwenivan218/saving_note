> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1777241237)

> 随机数，这个没有小伙伴没有用过吧，Java 提供了几个用于生成随机数的类，他们使用起来是这么地简单，以至于我们很少去认真的对待随机数的具体结果，就好像它是真的随机一样。Java17 之前的伪随机数生成器在

随机数，这个没有小伙伴没有用过吧，Java 提供了几个用于生成随机数的类，他们使用起来是这么地简单，以至于我们很少去认真的对待随机数的具体结果，就好像它是真的随机一样。

![](https://sike.skjava.com/java-features/202311281000003.png)

Java 17 之前的伪随机数生成器
------------------

在 Java 17 之前，Java 的随机数生成主要依赖于下面两个核心类：

*   `java.util.Random`
*   `java.security.SecureRandom`

### **Random**

该类是最最基本的伪随机数生成器，它用于生成一系列不完全是真正随机的数字。`Random` 提供了多种方法来生成不同类型的随机数，包括整数、长整数、浮点数等。

*   **生成随机整数**

```
Random rand = new Random();
int randomInt = rand.nextInt(50); 



```

*   生成随机浮点数

```
Random rand = new Random();
double randomDouble = rand.nextDouble(); 



```

`java.util.Random` 使用起来很简单，但是它有两个很明显的缺陷。

1.  **具备可观测性**：
    1.  `java.util.Random` 使用的是一个线性同余生成器（LCG）算法，该算法简单但效率不高。
    2.  可观测这意味着 `Random` 的输出是完全可预测的，如果我们知道种子和算法，完全可以生成相同的随机序列。这在需要高安全性的随机数生成（如加密）时是不合适的。
2.  **线程不安全**：在多线程环境下，多个线程共享一个 `Random` 实例可能导致竞争条件和数据不一致。

### ThreadLocalRandom

为了解决 `Random` 的线程不安全问题，Java 推出了 `ThreadLocalRandom`。`ThreadLocalRandom` 是一个适用于多线程环境下生成随机数的生成器，它通过为每个线程提供一个独立的随机数生成器实例来解决线程安全问题。

`ThreadLocalRandom` 使用线程局部变量（Thread-Local）的概念，每个线程访问 `ThreadLocalRandom` 时，实际上是访问它自己的一个独立实例，这就着不同线程之间的随机数生成器是完全隔离的，而且他们各自内部的种子是完全隔离的，这就保证了随机数生成的独立性，不会因为其他线程的操作而受到影响。

*   使用方法

```
ThreadLocalRandom random = ThreadLocalRandom.current();   

int randomInt = random.nextInt(10, 50); 
double randomDouble = random.nextDouble(1.0, 5.0); 



```

虽然 `ThreadLocalRandom` 解决了 `Random` 多线程的问题，但是它依然是基于线性同余生成器，导致它生成的随机数序仍然是可预测的。

### SecureRandom

由于 `java.util.Random` 生成的随机数具备可预测性，所以它不适用一些安全要求较高的场景。`java.security.SecureRandom` 是 Java 提供的一个用于生成加密强度随机数的类，它是 `java.security` 包的一部分，专门为需要高安全性的应用场景设计，比如加密、安全令牌生成、会话密钥以及数字签名应用。

`SecureRandom` 生成的随机数具有高安全性，这是因为它使用了更加复杂和不可预测的算法，而且这些算法通常都是基于操作系统提供的随机性源，例如 Unix/Linux 系统的 `/dev/random` 和 `/dev/urandom`，或 Windows 的 `CryptGenRandom API`。所以它更加适用于加密和安全相关的领域。

当然，我们也可以手动设置种子，但是一般不推荐这样做，因为会降低随机数的不可预测性。

`SecureRandom` 也提供了多种方法来生成不同类型的随机数：

```
SecureRandom secureRandom = new SecureRandom();

int randomInt = secureRandom.nextInt();
double randomDouble = secureRandom.nextDouble();



```

我们还可以指定特定的算法来创建 `SecureRandom` 实例：

```
SecureRandom secureRandom = SecureRandom.getInstance("SHA1PRNG");



```

虽然 `SecureRandom` 生成的随机数具有强大的安全性，但也恰恰如此，导致`SecureRandom` 在生成随机数时的性能开销比普通的随机数生成器更高。而且在某些情况下，`SecureRandom` 可能会因为等待足够的熵（随机性）而导致阻塞，尤其是在使用 `/dev/random` 作为随机性源的系统上。

Java 17 新特性：增强型伪随机数生成器
----------------------

### RandomGenerator 接口

在 Java 17 之前，Java 主要依赖 `java.util.Random` 和其子类来生成伪随机数。这些生成器在某些应用场景下不够高效或缺乏必要的特性（例如，长期稳定性、非周期性、能够快速跳跃到序列中的任意位置），为了提供多种新的伪随机数生成器，以满足不同的需求，Java 17 引入增强型伪随机数生成器（PRNG）。

Java 17 为随机数提供了一个全新的接口 `RandomGenerator`，该接口是 Java 生成随机数的顶层接口，用于定义所有伪随机数生成器的标准方法。它是所有新的和旧的随机数的顶层接口，包括 `java.util.Random`。

`RandomGenerator` 作为顶层接口，为了满足不同的随机数应用需求，它有几个子接口，UML 图例如下：

![](https://sike.skjava.com/java-features/202311281000001.png)

*   `StreamableGenerator`：用于那些可以产生随机数流（如整数、长整数、双精度数流）的生成器。实现这个接口的生成器可以创建特定大小或无限大小的随机数流，这对于需要大量随机数的应用（比如模拟或数据分析）特别有用。
*   `JumpableGenerator`：用于那些可以 “跳跃” 到其序列中一个远端点的随机数生成器。这意味着生成器可以从当前状态跳跃到一个预测的未来状态，而不必生成所有中间的随机数。这对于并行计算非常有用，因为它允许并行线程生成不重叠的随机数序列。
*   `LeapableGenerator`：类似于`JumpableGenerator`，但是提供了更精细控制。实现这个接口的生成器可以 “跳跃” 到序列中的一个特定点，然后 “回跳” 到原来的序列。这可以用于更复杂的并行计算场景，其中需要在随机数序列中前进和后退。
*   `ArbitrarilyJumpableGenerator`：用于那些可以跳到其序列中任意点的生成器。它提供了最灵活的跳跃功能，允许生成器跳到序列中任何位置。这对于需要非常特定随机数序列的应用非常有用。
*   `SplittableGenerator`：用于可以分裂成两个或多个独立运行的随机数生成器的情况。它允许每个并行组件拥有自己的随机数生成器实例，而这些生成器彼此独立且状态不相关。

`RandomGenerator` 提供了两类生成随机数的方法：

1.  生成随机数：例如 `nextInt()`、`nextInt(0, 100)`、`nextLong()`
2.  生成随机数流：例如 `ints()`、`longs()`，这些方法返回的是一个 Stream，对于生成大量随机数比较有用。

Java 提供了多种方式用来创建 `RandomGenerator` 实例。

*   使用`RandomGenerator.getDefault()`

这是获取 `RandomGenerator` 实例最简单的方式，它返回默认的`RandomGenerator`实例，适用于大多数用途。

```
RandomGenerator randomGenerator = RandomGenerator.getDefault();



```

*   使用`RandomGenerator.of(String name)`

`RandomGenerator` 提供了 `of()` 用于生成特定的随机数生成器，name 为预定义的生成器名称。

```
RandomGenerator randomGenerator = RandomGenerator.of("Xoshiro256PlusPlus");



```

*   使用 `RandomGeneratorFactory`

`RandomGeneratorFactory` 是生成 `RandomGenerator` 的工厂类，例如：

```
RandomGeneratorFactory<RandomGenerator> factory = RandomGeneratorFactory.of("Xoshiro256PlusPlus");
RandomGenerator randomGenerator = factory.create();



```

得到了 `RandomGenerator` 实例，就可以调用对应的方法获取对应的随机数了。

### RandomGeneratorFactory

`RandomGeneratorFactory` 是创建 `RandomGenerator` 实例的工具类，可以用它来创建不同类型的 `RandomGenerator` 实例，包括但不限于传统的线性同余生成器、梅森旋转算法（Mersenne Twister）、Xoroshiro128++ 算法等。同时它还允许用户根据需求定制随机数生成器的行为，比如设置种子或选择特定的算法。

`RandomGeneratorFactory` 的核心 API 如下：

1.  `of(String name)`：根据指定的名称创建一个 `RandomGenerator` 实例。
2.  `all()`：返回所有可用的 `RandomGeneratorFactory` 实例。
3.  `getDefault()`：返回默认的 `RandomGeneratorFactory` 实例。
4.  `create()`：使用工厂的配置创建一个新的 `RandomGenerator` 实例。

下面演示如何使用 `RandomGeneratorFactory` 创建并使用一个随机数生成器：

```
    @Test
    public void randomGeneratorFactoryTest() {
        
        RandomGeneratorFactory<RandomGenerator> factory = RandomGeneratorFactory.of("Xoroshiro128PlusPlus");
        
        RandomGenerator randomGenerator = factory.create();
        
        randomGenerator.ints().limit(10)
                .forEach(System.out::println);
    }


```

### **RandomGenerator** 类型

Java 17 支持多种 `RandomGenerator` 类型，我们可以通过 `RandomGeneratorFactory.all()` 来获取：

```
    @Test
    public void randomGeneratorFactoryTest() {
        RandomGeneratorFactory.all().forEach(x -> {
            System.out.println(x.name());
        });
    }
 
L32X64MixRandom
L128X128MixRandom
L64X128MixRandom
SecureRandom
L128X1024MixRandom
L64X128StarStarRandom
Xoshiro256PlusPlus
L64X256MixRandom
Random
Xoroshiro128PlusPlus
L128X256MixRandom
SplittableRandom
L64X1024MixRandom



```

其中 SecureRandom 和 Random 是旧的，其余都是 Java 17 提供的，他们位于 `jdk.random` 模块下：

![](https://sike.skjava.com/java-features/202311281000002.png)

这些类型都有自己的名字，用注解标注出来，例如：

```
@RandomGeneratorProperties(
        name = "L32X64MixRandom",
        group = "LXM",
        i = 64, j = 1, k = 32,
        equidistribution = 1
)
public final class L32X64MixRandom extends AbstractSplittableWithBrineGenerator {
    
}


```

我们可以通过 name 属性来获取对应的实例：

```
    @Test
    public void randomGeneratorTest() {
        RandomGenerator randomGenerator = RandomGenerator.of("L32X64MixRandom");
        randomGenerator.ints().limit(10).forEach(System.out::println);
    }


```

下面是 Java 17 提供 10 个不同的随机数生成器：

<table><thead><tr><th>类名</th><th>使用的算法</th><th>特点</th></tr></thead><tbody><tr><td><code>L32X64MixRandom</code></td><td>LXM 算法</td><td>平衡性能和随机性，适用于多种通用应用</td></tr><tr><td><code>L32X64StarStarRandom</code></td><td>StarStar 算法</td><td>提供较好的随机性和性能，适用于需要较快随机数生成的场景</td></tr><tr><td><code>L64X128MixRandom</code></td><td>LXM 算法</td><td>提供更长的周期和更高的随机性，适合更复杂的随机数生成需求</td></tr><tr><td><code>L64X128StarStarRandom</code></td><td>StarStar 算法</td><td>结合了较长周期和良好的随机性，适用于复杂应用</td></tr><tr><td><code>L64X256MixRandom</code></td><td>LXM 算法</td><td>长周期，高随机性，适合于需求严格的随机数生成场景</td></tr><tr><td><code>L64X1024MixRandom</code></td><td>LXM 算法</td><td>高随机性，适用于特别需要长周期和高随机性的应用</td></tr><tr><td><code>L128X128MixRandom</code></td><td>LXM 算法</td><td>极长周期，高随机性，适合于高随机质量要求的应用</td></tr><tr><td><code>L128X256MixRandom</code></td><td>LXM 算法</td><td>提供极长的周期和优秀的随机性，适用于极高随机质量要求的应用</td></tr><tr><td><code>L128X1024MixRandom</code></td><td>LXM 算法</td><td>极高的随机性和更大的状态空间，适用于对随机数质量有极端要求的场合</td></tr><tr><td><code>Xoshiro256PlusPlus</code></td><td>Xoshiro 算法</td><td>高性能，适用于需要快速、高质量随机数的应用</td></tr><tr><td><code>Xoroshiro128PlusPlus</code></td><td>Xoroshiro 算法</td><td>提供良好的性能和随机性平衡，适用于多种应用场景</td></tr></tbody></table>
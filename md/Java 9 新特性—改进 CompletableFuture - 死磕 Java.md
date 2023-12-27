> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1843207164)

> CompletableFuture 是 Java8 中引入用于处理异步编程的核心类，它引入了一种基于 Future 的编程模型，允许我们以更加直观的方式执行异步操作，并处理它们的结果或异常。关于 Completa

`CompletableFuture` 是 Java 8 中引入用于处理异步编程的核心类，它引入了一种基于 Future 的编程模型，允许我们以更加直观的方式执行异步操作，并处理它们的结果或异常。关于 `CompletableFuture` 的核心原理请阅读：[Java 8 新特性—深入理解 CompletableFuture](https://www.skjava.com/series/article/5020516690)。

但是在实际使用过程中，发现 `CompletableFuture` 还有一些改进空间，所以 Java 9 对它做了一些增强，主要内容包括：

*   **新的工厂方法**
*   **支持超时和延迟执行**
*   **支持子类化**

![](https://sike.skjava.com/java-features/202311261000001.png)

**新的工厂方法**
----------

在 Java 9 中 CompletableFuture 新增了三个工厂方法。

> `completedFuture()`

此方法用于创建一个已经完成的 `CompletableFuture` 实例，方法定义如下：

```
public static <U> CompletableFuture<U> completedFuture(U value)


```

该方法允许我们快速创建一个已经有结果的 `CompletableFuture`，这对于单元测试或需要立即返回结果的场景非常有用。

> `failedFuture()`

此方法创建一个异常完成（异常终止）的 `CompletableFuture` 实例，方法定义如下：

```
public static <U> CompletableFuture<U> failedFuture(Throwable ex) 


```

方法接受的参数为 Throwable，表明为异常终止，它为异步编程提供了一种简洁的方式来表示已知的失败情况，这对于错误处理和异常测试场景非常重要。

除上面两个工厂方法外，Java 9 引入了下面这对 `stage-oriented` 工厂方法，返回完成的或异常完成的 completion stages：

*   `CompletionStage completedStage(U value)`: 返回一个新的以指定 value 完成的`CompletionStage` ，并且只支持 `CompletionStage` 里的接口。
*   `CompletionStage failedStage(Throwable ex)`: 返回一个新的以指定异常完成的`CompletionStage` ，并且只支持 `CompletionStage` 里的接口。

**支持超时和延迟执行**
-------------

在 Java 9 中，`CompletableFuture` 引入了支持超时和延迟执行的改进，这两个功能对于控制异步操作的时间行为至关重要。

### 支持超时

> `orTimeout()`

允许为 `CompletableFuture` 设置一个超时时间。如果在指定的超时时间内未完成，`CompletableFuture` 将以 `TimeoutException` 完成，方法定义如下：

```
public CompletableFuture<T> orTimeout(long timeout, TimeUnit unit)


```

该方法为 `CompletableFuture` 提供了超时机制，这对于避免永久挂起的异步操作和保证响应性至关重要。

*   示例：

```
    @Test
    public void orTimeTest() {
       try {
           CompletableFuture completableFuture = CompletableFuture.runAsync(()->{
               System.out.println("异步任务开始执行....");
               try {
                   TimeUnit.SECONDS.sleep(5);
               } catch (InterruptedException e) {
                   throw new RuntimeException(e);
               }
           }).orTimeout(2,TimeUnit.SECONDS);

           completableFuture.join();
       } catch (Exception e) {
           System.out.println(e);
       }
    }


```

*   执行结果

```
异步任务开始执行....
java.util.concurrent.CompletionException: java.util.concurrent.TimeoutException


```

> `completeOnTimeout()`

该允许在指定的超时时间内如果未完成，则用一个默认值来完成 `CompletableFuture`，方法定义如下：

```
public CompletableFuture<T> completeOnTimeout(T value, long timeout, TimeUnit unit)


```

`value` 为默认默认值。该方法提供了一种优雅的回退机制，确保即使在超时的情况下也能保持异步流的连续性和完整性。

*   示例

```
    @Test
    public void completeOnTimeoutTest() {
        CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(()->{
            System.out.println("异步任务开始执行....");
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            return "死磕 Java 新特性";
        }).completeOnTimeout("死磕 Java",2,TimeUnit.SECONDS);

        System.out.println("执行结果为：" + completableFuture.join());
    }


```

*   执行结果

```
异步任务开始执行....
执行结果为：死磕 Java


```

### 支持延迟执行

`CompletableFuture` 提供了`delayedExecutor()` 来支持延迟执行，该方法创建一个延迟执行的 `Executor`，可以将任务的执行推迟到未来某个时间点。方法定义如下：

```
public static Executor delayedExecutor(long delay, TimeUnit unit, Executor executor)


```

它能够让我们更加精确地控制异步任务的执行时机，特别是在需要根据时间安排任务执行的场景中。

*   示例

```
    @Test
    public void completeOnTimeoutTest() {
        
        Executor delayedExecutor = CompletableFuture.delayedExecutor(3, TimeUnit.SECONDS);

        
        CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
            System.out.println("任务延迟后执行...");
        }, delayedExecutor);

        
        future.join();
    }


```

**支持子类化**
---------

在 Java 9 中，CompletableFuture 做了一些改进，使得 **CompletableFuture** 可以被更简单的继承。

> `newIncompleteFuture()`

创建一个新的、不完整的 `CompletableFuture` 实例， 子类可以重写这个方法来返回 `CompletableFuture` 的子类实例，允许在整个 `CompletableFuture` API 中自定义实例的行为。

> `defaultExecutor()`

该方法提供 `CompletableFuture` 操作的默认执行器，子类可以重写此方法以提供不同的默认执行器，这对于定制任务执行策略特别有用。

> `copy()`

创建一个与当前 `CompletableFuture` 状态相同的新实例。子类可以重写此方法以确保复制的实例是特定子类的实例，而不仅仅是 `CompletableFuture`。
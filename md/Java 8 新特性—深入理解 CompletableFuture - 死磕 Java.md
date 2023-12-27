> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/5020516690)

> CompletableFuture 是 Java8 中引入用于处理异步编程的核心类，它引入了一种基于 Future 的编程模型，允许我们以更加直观的方式执行异步操作，并处理它们的结果或异常。Future 的局限性

CompletableFuture 是 Java 8 中引入用于处理异步编程的核心类，它引入了一种基于 Future 的编程模型，允许我们以更加直观的方式执行异步操作，并处理它们的结果或异常。

![](https://sike.skjava.com/java-features/202310211000014.png)

Future 的局限性
-----------

学过 Java 并发或者接触过异步开发的小伙伴应该都知道 Future，通过 Future 我们能够知道异步执行的操作结果，它提供了 `isDone()` 来检测异步是否已经完成，也可以通过 `get()` 方法来获取计算结果。在异步计算中，Future 确实是一个非常优秀的接口，但是它依然存在一些局限性：

1.  **缺乏回调机制**：Future 没有内置的回调机制，这就意味着我们必须轮询 Future 对象来检查任务是否完成，而不是等待通知。
2.  **无法取消任务**：虽然可以通过 `cancel()` 方法来取消 Future 中的任务，但这并不保证任务会被取消。如果任务已经开始执行，那么 `cancel()` 方法可能无法终止任务的执行。
3.  **缺乏异常处理机制**：Future 通过 `get()` 方法返回任务的结果或异常，但它无法提供更多的异常处理功能。如果任务抛出异常，你必须在客户端代码中捕获这些异常。
4.  **单一结果**：每个 Future 对象只能关联一个任务，这就限制了它的使用，如果我们需要并行执行多个任务并收集它们的结果，我们只能自己管理多个 Future 对象。
5.  **无法进行链式调用**：如果我们希望在计算完成后执行特定操作，比如通知用户，这个时候我们就无法使用 Future 来实现了。
6.  **无法组合多任务**：在处理多个任务时，Future 并没有提供很好的组合方式，比如我们需要等待 10 任务全部完成后再执行特定操作，这个时候使用 Future 就不是很好操作了。

什么是 CompletableFuture
---------------------

为了克服 Future 的局限性，Java 8 提供了 CompletableFuture，它构建在 Future 之上，提供了更加强大的异步编程功能，相比 Future 它具备如下优势：

1.  **提供了回调机制**：CompletableFuture 提供了回调功能，我们可以注册回调函数来处理任务完成时的结果，而不必阻塞线程等待任务完成。这样可以提高并发性能，减少线程的阻塞时间。
2.  **提供了异常处理**：CompletableFutur 具备丰富的异常处理机制，可以捕获任务执行中的异常，并允许我们定义自定义的异常处理策略。
3.  **能够取消任务**：我们可以使用 `cancel()`取消任务的执行，同时还可以指定是否中断正在执行的任务。这提供了更好的任务控制能力。
4.  **强大的异步编程能力**：CompletableFuture 提供了丰富的方法来处理异步操作，包括**组合**、**转换**、**处理异常**以及**执行自定义**的操作。这使得异步编程更加灵活，可以更轻松地实现复杂的异步任务组合。
5.  **支持组合、链式操作**：CompletableFutur 提供了一系列支持组合操作的方法，例如 `thenCombine()` ， `thenCompose()`，`thenApplyAsync()`等等，使得多个 CompletableFuture 可以轻松组合成一个新的 CompletableFuture，从而更容易构建复杂的异步操作流。

CompletableFuture 提供了比传统 Future 更加强大、更加灵活的异步编程能力，能够更好地满足复杂异步任务处理的需求，能够更加方便地构建复杂的异步操作流，是 Java 8 及以后的版本中，处理异步操作的首选。

CompletableFuture 的核心 API
-------------------------

### 构建异步操作

CompletableFuture 提供了多种方法用于构建异步操作。

> `runAsync()`： 用于异步执行没有返回值的任务。

它有两个重载方法：

```
public static CompletableFuture<Void> runAsync(Runnable runnable)
public static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor)



```

这两个方法的区别在于：

*   `runAsync(Runnable runnable)` 会使用 ForkJoinPool 作为它的线程池执行异步代码。
*   `runAsync(Runnable runnable, Executor executor)` 则是使用指定的线程池执行异步代码。

示例：

```
    @Test
    public void runAsyncTest(){
        CompletableFuture.runAsync(() ->{
            log.info("死磕 Java 新特性 - 01");
        });

        CompletableFuture.runAsync(() -> {
            log.info("死磕 Java 新特性 - 02");
        }, Executors.newFixedThreadPool(10));
    }



```

结果

![](https://sike.skjava.com/java-features/202310211000001.jpg)

> `supplyAsync()`**：** 用于异步执行有返回值的任务。

`supplyAsync()` 也有两个重载方法，区别 `runAsync()` 和一样：

```
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier)
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor



```

示例：

```
    @Test
    public void supplyAsyncTest() throws Exception {
        CompletableFuture<String> completableFuture1 = CompletableFuture.supplyAsync(() ->{
            log.info("死磕 Java 新特性 - 01");
            return "死磕 Java 新特性 - 01";
        });

        CompletableFuture<String> completableFuture2 = CompletableFuture.supplyAsync(() ->{
            log.info("死磕 Java 新特性 - 02");
            return "死磕 Java 新特性 - 02";
        },Executors.newFixedThreadPool(10));

        log.info(completableFuture1.get());
        log.info(completableFuture2.get());
    }


```

结果：

![](https://sike.skjava.com/java-features/202310211000002.jpg)

> `completedFuture()`： 创建一个已完成的 CompletableFuture，它包含特定的结果。

```
    @Test
    public void completedFutureTest() {
        CompletableFuture<String> completableFuture = CompletableFuture.completedFuture("死磕 Java 就是牛");
        System.out.println(completableFuture.join());
    }

死磕 Java 就是牛



```

> 注意：使用默认线程池会有一个：在主线程任务执行完以后，如果异步线程执行任务还没执行完，它会直接把异步任务线程清除掉，因为默认线程池中的都是守护线程 ForkJoinPool，当没有用户线程以后，会随着 JVM 一起清除。

```
    @Test
    public void runAsyncTest(){
        CompletableFuture.runAsync(() ->{
            log.info("CompletableFuture 任务开始执行...");
            for (int i = 0; i < 100 ; i++) {
                log.info("CompletableFuture 任务执行中[{}]...",i);
            }
            log.info("CompletableFuture 任务执行完毕...");
        });

        log.info("主线程执行完毕...");
    }


```

结果：

![](https://sike.skjava.com/java-features/202310211000004.jpg)

CompletableFuture 任务的 for 循环只执行到 14 就结束了，并没有完成整个任务就被清理掉了。

### 获取结果

CompletableFuture 提供了 `get()` 和 `join()` 方法用于我们获取计算结果：

```
public T get() throws InterruptedException, ExecutionException
public T get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException

public T join()



```

`get()` 有两个重载方法：

*   `get()`：会阻塞当前线程，直到计算完成并返回结果
*   `get(long timeout, TimeUnit unit)`：有阻塞时间，如果在指定的超时时间内未能获取到结果，会抛出 `TimeoutException` 异常。

而 `get()` 和 `join()` 的区别则在于：

*   `get()` 会抛出 `InterruptedException` 和 `ExecutionException` 这两个受检查异常，我们必须显式地在代码中处理这些异常或将它们抛出。
*   `join()` 不会抛出受检查异常，所以在使用过程中代码会显得更加简洁，但是如果任务执行中发生异常，它会包装在 `CompletionException` 中，我们需要在后续代码中处理。

示例：

```
    @Test
    public void completedFutureTest() {
        CompletableFuture<String> completableFuture = CompletableFuture.completedFuture("死磕 Java 就是牛");
        
        System.out.println(completableFuture.join());

        try {
            System.out.println(completableFuture.get());
        } catch (InterruptedException | ExecutionException e) {
            
            
        }
    }


```

### 结果、异常处理

当 CompletableFuture 因为异步任务执行完成或者发生异常而完成时，我们可以执行特定的 Action，主要方法有：

```
public CompletableFuture<T> whenComplete(BiConsumer<? super T,? super Throwable> action)
public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T,? super Throwable> action)
public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T,? super Throwable> action, Executor executor)

public CompletableFuture<T> exceptionally(Function<Throwable, ? extends T> fn) 



```

> `whenComplete(BiConsumer<? super T,? super Throwable> action)`

接受一个 `Consumer` 参数，该参数接受计算的结果（如果成功）或异常（如果发生异常）并执行相应的操作。

```
    @Test
    public void whenCompleteTest() {
        CompletableFuture<String> completableFuture1 = CompletableFuture.supplyAsync(() -> {
            log.info("[completableFuture-1] - www.skjava.com 网站就是牛..");
            return "[completableFuture-1] - 死磕 Java 新特性";
        }).whenComplete((res,ex) -> {
           if (ex == null) {
               System.out.println("结果是:" + res);
           } else {
               System.out.println("发生了异常，异常信息是:" + ex.getMessage());
           }
        });
    }


```

该方法是同步执行，回调函数是在触发它的 CompletableFuture 所在的线程中执行，且它会阻塞当前线程。比如这里我们是在 main 线程去调用它的，所以执行他的线程就是 main 线程，它会阻塞 mian 线程执行。如下：

```
public class WhenCompleteTest {
    private static CompletableFuture<String> future;

    public static void main(String[] args) {
        future = CompletableFuture.supplyAsync(() ->{
            log.info("CompletableFuture 主体执行");
            return "死磕 Java 新特性";
        });

        Thread thread = new Thread(() ->{
           log.info("thread 线程开始执行");

            future.whenComplete((res,ex) -> {
                log.info("whenComplete 主体开始执行");

                sleep(5);

                if (ex == null) {
                    log.info("whenComplete 执行结果:{}",res);
                } else {
                    log.info("whenComplete 执行异常:{}",ex.getMessage());
                }
                log.info("whenComplete 主体执行完毕");
            });

            log.info("thread 线程执行完成");
        });
        thread.setName("test-thread");
        thread.start();

        
        sleep(15);
        log.info("主线程执行完毕");
    }

    public static void sleep(long sleep) {
        try {
            TimeUnit.SECONDS.sleep(sleep);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}


```

结果

![](https://sike.skjava.com/java-features/202310211000006.jpg)

1.  首先 `test-thread` 线程先执行，打印 “thread 线程开始执行”
2.  然后调用 `future.whenComplete()`，这个时候我们看到执行的线程也是 `test-thread`，在这里面它等待了 5 秒
3.  5 秒过后再次打印 “thread 线程执行完成”

从执行结果中可以看出 `whenComplete()` 就是由调用它的线程来执行，且会阻塞当前线程

> `whenCompleteAsync(BiConsumer<? super T,? super Throwable> action)`

异步执行，回调函数会在默认的 ForkJoinPool 的线程中执行，但是它不会阻塞当前线程。我们将上面例子的 `whenComplete()` 改成 `whenCompleteAsync()`，执行结果如下：

![](https://sike.skjava.com/java-features/202310211000007.jpg)

*   `whenCompleteAsync()` 方法的执行线程是 `ForkJoinPool.commonPool-worker-9`
*   没有阻塞 `test-tread` 线程的执行

> `whenCompleteAsync(BiConsumer<? super T,? super Throwable> action, Executor executor)`

它与前一个方法相似，只不过我们可以执行执行 Action 执行的线程池。

```
    @Test
    public void whenCompleteTest() {
        CompletableFuture<String> completableFuture3 = CompletableFuture.supplyAsync(() -> {
            log.info("[completableFuture-3] - www.skjava.com 网站就是牛..");
            return "[completableFuture-2] - 死磕 Java 新特性";
        }).whenCompleteAsync((res,ex) -> {
            if (ex == null) {
                log.info("结果是:{}",res);
            } else {
                log.warn("发生了异常，异常信息是:{}",ex.getMessage());
            }
        },Executors.newFixedThreadPool(4));
    }


```

> `exceptionally(Function<Throwable, ? extends T> fn)`

`exceptionally()` 用于处理异步操作中的异常情况，当异步操作发生异常时，该回调函数将会被执行，我们可以在该回调函数中处理异常情况。`exceptionally`() 返回一个新的 `CompletableFuture` 对象，其中包含了异常处理的结果或者异常对象。

```
    @Test
    public void exceptionallyTest() {
        CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
            int i = 10 / 0;
            return "死磕 Java 并发就是牛";
        });

        CompletableFuture<String> resultFuture = future.exceptionally((ex) -> {
            log.info("发生了异常:{}",ex.getMessage());
            return "死磕 Netty 就是牛";
        });

        try {
            System.out.println(future.join());
        } catch (Exception ex) {
            log.error("异常:{}",ex.getMessage());
        }
        System.out.println(resultFuture.join());
    }


```

结果

![](https://sike.skjava.com/java-features/202310211000008.jpg)

由于 future 抛了异常，所以调用 `future.join()` 会报错，我们需要 `try...catch` 处理下 。

### 结果转换

结果转换，就是将上一段任务的执行结果作为下一阶段任务的入参参与重新计算，产生新的结果。

> `thenApply()`** **和** **`thenApplyAsync()`**：** 用于将一个 CompletableFuture 的结果应用于一个函数，并返回一个新的 CompletableFuture，表示转换后的结果。

```
    @Test
    public void thenApplyTest() {
        CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(() -> {
            log.info("执行第一步...");
            return "死磕 Java";
        }).thenApply(s -> {
            log.info("执行第二步,第一步返回结果:{}",s);
            return s + " 就是牛..";
        });

        log.info("结果为:{}",completableFuture.join());
    }
 
2023-10-22 15:28:26.882 [ForkJoinPool.commonPool-worker-9] INFO  - 执行第一步...
2023-10-22 15:28:26.888 [ForkJoinPool.commonPool-worker-9] INFO  - 执行第二步,第一步返回结果:死磕 Java
2023-10-22 15:28:26.888 [main] INFO  - 结果为:死磕 Java 就是牛.



```

`thenApply()`** **和** **`thenApplyAsync()` 两个方法的区别就不用大明哥再阐述了吧。

> `thenCompose()` 和 `thenComposeAsync()` ：它用于将一个 CompletableFuture 的结果应用于一个函数，该函数返回一个新的 CompletableFuture。

```
    @Test
    public void thenComposeTest() {
        CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(() -> {
            log.info("执行第一步...");
            return "死磕 Java";
        }).thenCompose((s) -> {
            log.info("执行第二步,第一步返回结果:{}",s);
            
            return CompletableFuture.supplyAsync(() -> s + " 就是牛..");
        });

        log.info("结果为:{}",completableFuture.join());
    }


```

`thenCompose()` 与 `thenApply()` 两者的返回值虽然都是新的 CompletableFuture，但是 `thenApply()` 由于它的函数的返回值仅仅只是结果，所以它通常用于对异步操作的结果进行简单的转换，而 `thenCompose()` 则允许我们链式地组合多个异步操作。虽然两者都有可能实现相同的效果（比如上面例子），但是他们的使用场景和意义还是有区别的。

### 结果消费

结果消费则是只对结果执行 Action，而不返回新的计算值。

> `thenAccept()`：用于处理异步操作的结果，但不返回任何结果。

`thenAccept()` 接受一个 Consumer 函数接口。

```
    @Test
    public void thenAcceptTest() throws InterruptedException {
        CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(() ->{
          return "死磕 Java 新特性";
        });

        completableFuture.thenAccept(s ->{
            System.out.println("CompletableFuture 计算结果是:" + s);
        });

        TimeUnit.SECONDS.sleep(5);
    }

CompletableFuture 计算结果是:死磕 Java 新特性



```

> `thenAcceptBoth()`：用于处理两个不同的 CompletableFuture 异步操作的结果，并执行操作，但不返回新的结果。

方法定义如下：

```
public CompletableFuture<Void> thenAcceptBoth(CompletableFuture<? extends U> other, BiConsumer<? super T, ? super U> action)



```

*   other：为另外一个 CompletableFuture，它包含了另一个异步操作的结果。
*   action：类型为 BiConsumer，它接受两个参数，分别表示第一个 CompletableFuture 的结果和第二个 CompletableFuture 的结果。

```
    @Test
    public void thenAcceptBothTest() throws InterruptedException {
        CompletableFuture<String> future1 = CompletableFuture.completedFuture("死磕 Netty");
        CompletableFuture<String> future2 = CompletableFuture.completedFuture("死磕 Java 新特性");

        future1.thenAcceptBoth(future2,(result1,result2) -> {
            System.out.println("future1 的结果是：" + result1);
            System.out.println("future2 的结果是：" + result2);
        });

        TimeUnit.SECONDS.sleep(5);
    }
 
 future1 的结果是：死磕 Netty
future2 的结果是：死磕 Java 新特性



```

> `thenRun()`：用于在一个 CompletableFuture 异步操作完成后执行操作，而不关注计算的结果

`thenRun()` 通常用于执行其他作用的操作、清理工作、或在异步操作完成后触发其他操作。

```
    @Test
    public void thenRunTest() throws InterruptedException {
        CompletableFuture<String> future = CompletableFuture.completedFuture("死磕 Netty");
        future.thenRun(()  ->{
            System.out.println("CompletableFuture 计算执行完成，开始执行后续操作...");
        });

        TimeUnit.SECONDS.sleep(5);
    }


```

### 结果组合

`thenCombine()` 用于将两个不同的 CompletableFuture 异步操作的结果合并为一个新的结果，并执行操作。该方法允许我们在两个异步操作都完成后执行一个操作，它接受两个结果作为参数，并返回一个新的结果。

方法定义如下：

```
public <U, V> CompletableFuture<V> thenCombine(CompletableFuture<? extends U> other, BiFunction<? super T, ? super U, ? extends V> action)



```

*   other：表示另外一个 CompletableFuture，它包含了该 CompletableFuture 的计算结果
*   action：类型是 BiFunction，它接受两个参数，分别是第一个 CompletableFuture 的计算结果和第二个 CompletableFuture 的计算结果。

```
    @Test
    public void thenCombineTest() {
        CompletableFuture<String> future1 = CompletableFuture.completedFuture("死磕 Netty");
        CompletableFuture<String> future2 = CompletableFuture.completedFuture("死磕 Java 新特性");

        CompletableFuture<String> combineFuture = future1.thenCombine(future2,(result1,result2) ->{
            System.out.println("future1 的结果是：" + result1);
            System.out.println("future2 的结果是：" + result2);
            return result1 + "和" + result1 + " 就是牛...";
        });

        System.out.println(combineFuture.join());
    }
 
future1 的结果是：死磕 Netty
future2 的结果是：死磕 Java 新特性
死磕 Netty和死磕 Netty 就是牛...



```

### 任务交互

> `applyToEither()`

`applyToEither()` 用于处理两个不同的 CompletableFuture 异异步操作中的任何一个完成后，将其结果应用于一个函数，并返回一个新的 CompletableFuture 表示该函数的输出结果。该方法允许我们在两个异步操作中的任何一个完成时执行操作，而不需要等待它们都完成。

```
    @Test
    public void applyToEitherTest() {
        CompletableFuture<String> future1 = CompletableFuture.completedFuture("死磕 Netty");
        CompletableFuture<String> future2 = CompletableFuture.completedFuture("死磕 Java 新特性");

        CompletableFuture<String> eitherFuture = future1.applyToEither(future2,res ->{
            System.out.println("接受的结果是:" + res);
            return "eitherFuture 接受的结果是:" +res;
        });

        System.out.println(eitherFuture.join());
    }

接受的结果是:死磕 Netty
eitherFuture 接受的结果是:死磕 Netty


```

> `acceptEither()`

`acceptEither()` 与 `applyToEither()` 一样，也是等待两个 CompletableFuture 中的任意一个执行完成后执行操作，但是它不返回结果。

```
    @Test
    public void acceptEitherTest() {
        CompletableFuture<String> future1 = CompletableFuture.completedFuture("死磕 Netty");
        CompletableFuture<String> future2 = CompletableFuture.completedFuture("死磕 Java 新特性");

        CompletableFuture<Void> eitherFuture = future1.acceptEither(future2,res ->{
            System.out.println("接受的结果是:" + res);
        });

        eitherFuture.join();
    }
 
 接受的结果是:死磕 Java 新特性


```

> `runAfterEither()`

`runAfterEither()`用于在两个不同的 CompletableFuture 异步操作中的任何一个完成后执行操作，而不依赖操作的结果。这个方法通常用于在两个异步操作中的任何一个成功完成时触发清理操作或执行某些操作，而不需要返回值。

```
    @Test
    public void runAfterEitherTest() {
        CompletableFuture<String> future1 = CompletableFuture.completedFuture("死磕 Netty");
        CompletableFuture<String> future2 = CompletableFuture.completedFuture("死磕 Java 并发");
        future1.runAfterEither(future2,() ->{
            System.out.println("已经有一个任务完成了...");
        });
    }


```

> `runAfterBoth()`

`runAfterBoth()` 用于在两个不同的 CompletableFuture 异步操作都完成后执行操作，而不依赖操作的结果。这个方法通常用于在两个异步操作都完成时触发某些操作或清理工作，而不需要返回值。

```
    @Test
    public void runAfterBothTest() {
        CompletableFuture<String> future1 = CompletableFuture.completedFuture("死磕 Netty");
        CompletableFuture<String> future2 = CompletableFuture.completedFuture("死磕 Java 并发");
        future1.runAfterBoth(future2,() ->{
            System.out.println("future1 和 future2 两个异步任务都完成了...");
        });
    }


```

> `anyOf()`

`anyOf()` 是用于处理多个 CompletableFuture 对象的**静态方法**，它允许我们等待多个异步操作中的任何一个完成，并执行相应的操作。它类似于多个异步操作的并发执行，只要有一个操作完成，它就会返回一个新的 CompletableFuture 对象，表示第一个完成的操作。

`anyOf()` 是一个可变参数，我们可以传入任意数量的 CompletableFuture 对象。

```
    @Test
    public void anyOfTest() {
        CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() ->{
            sleep(1);
            log.info("死磕 Netty 执行完成...");
            return "死磕 Netty";
        });

        CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() ->{
            sleep(2);
            log.info("死磕 Java 并发 执行完成...");
            return "死磕 Java 并发";
        });

        CompletableFuture<String> future3 = CompletableFuture.supplyAsync(() ->{
            sleep(3);
            log.info("死磕 Redis 执行完成...");
            return "死磕 Redis";
        });

        CompletableFuture<String> future4 = CompletableFuture.supplyAsync(() ->{
            sleep(4);
            log.info("死磕 Java 新特性 执行完成...");
            return "死磕 Java 新特性";
        });

        CompletableFuture<String> future5 = CompletableFuture.supplyAsync(() ->{
            sleep(5);
            log.info("死磕 Spring 执行完成...");
            return "死磕 Spring";
        });

        CompletableFuture<Object> anyOfFuture = CompletableFuture.anyOf(future1,future2,future3,future4,future5);
        anyOfFuture.thenAccept(result -> {
            log.info("接收到的结果为:" + result);
        });

        sleep(10);
    }



```

结果

![](https://sike.skjava.com/java-features/202310211000009.jpg)

`anyOf()` 比较有用，当我们需要并行执行多个异步操作，并在其中任何一个完成时执行操作时，就可以使用它，大明哥在生产过程中应用过几次。

> `allOf()`

anyOf() 是任一一个异步任务完成就会触发，而 `allOf()` 则需要所有异步都要完成。我们将上面方法改为 `allOf()` 得到结果如下：

![](https://sike.skjava.com/java-features/202310211000010.jpg)

这里得到结果为 null，是因为 `allOf()` 是没有返回值的。

CompletableFuture 的任务编排
-----------------------

著名数学家华罗庚先生在《统筹方法》这篇文章里介绍了一个烧水泡茶的例子，最优解如下：

![](https://sike.skjava.com/java-features/202310211000011.jpg)

但是我们为了能够更好地验证 CompletableFuture 的任务编排功能，我们将其进行扩展：

![](https://sike.skjava.com/java-features/202310211000012.jpg)

```
public class Tea {
    public static void main(String[] args) throws InterruptedException {
        CompletableFuture<String> future1  = CompletableFuture.supplyAsync(()->{
            log.info("拿开水壶");
            return "开水壶";
        });

        CompletableFuture<String> future2  = CompletableFuture.supplyAsync(() -> {
            log.info("拿水壶");
            return "水壶";
        });

        CompletableFuture<String> future3  = CompletableFuture.supplyAsync(() ->{
            log.info("拿茶杯");
            return "茶杯";
        });

        CompletableFuture<String> future4  = CompletableFuture.supplyAsync(() -> {
            log.info("拿茶叶");
            return "西湖龙井";
        });

        CompletableFuture<String> future11 = future1.thenApply((result) -> {
            log.info("拿到" + result + ",开始洗" + result);
            return "干净的开水壶";
        });

        CompletableFuture<String> future12 = future11.thenApply((result) -> {
            log.info("拿到"  + result + ",开始烧开水");
            return "烧开水了";
        });

        CompletableFuture<String> future21 = future2.thenApply((result) -> {
            log.info("拿到" + result + ",开始洗" + result);
            return "干净的水壶";
        });

        CompletableFuture<String> future31 = future3.thenApply((result) -> {
            log.info("拿到" + result + ",开始洗" + result);
            return "干净的茶杯";
        });


        CompletableFuture<Void> future5  = CompletableFuture.allOf(future4,future12,future21,future31);
        future5.thenRun(() -> {
           log.info("泡好了茶，还是喝美味的西湖龙井茶");
        });

        TimeUnit.SECONDS.sleep(5);
    }
}


```

执行结果：

![](https://sike.skjava.com/java-features/202310211000013.jpg)

结果大明哥就不分析了，各位小伙伴好好对照下就明白了。通过这个例子我们清晰的见识到 CompletableFuture 任务编排的能力。

CompletableFuture API 总结
------------------------

CompletableFuture 的 API 比较多，不同的方法有不同的使用场景，大明哥也不可能将所有的 API 都介绍和举一个示例，就简单列一个表格吧。

> **构建异步操作**

<table><thead><tr><th>方法</th><th>说明</th><th>有无返回值</th></tr></thead><tbody><tr><td><code>runAsync</code></td><td>异步执行任务，默认 ForkJoinPool 线程池</td><td>无返回值</td></tr><tr><td><code>supplyAsync</code></td><td>异步执行任务，默认 ForkJoinPool 线程池</td><td>有返回值</td></tr><tr><td><code>completedFuture</code></td><td>创建一个已经完成的 CompletableFuture 对象</td><td>有返回值</td></tr></tbody></table>

> **两个线程依次执行**

<table><thead><tr><th>方法</th><th>说明</th><th>有无返回值</th></tr></thead><tbody><tr><td><code>thenApply</code></td><td>获取前一个线程的执行结果，第二个线程处理该结果，生成一个新的 CompletableFuture 对象</td><td>有返回值</td></tr><tr><td><code>thenAccept</code></td><td>获取前一个线程的执行结果，第二个线程消费结果，不会返还给调用端</td><td>无返回值</td></tr><tr><td><code>thenRun</code></td><td>第一个线程执行完后，再执行，它忽略第一个线程的执行结果，也不返回结果</td><td>无返回值</td></tr><tr><td><code>thenCompose</code></td><td>获取前一个线程的执行结果，对其进行组合，返回新的 CompletableFuture 对象</td><td>有返回值</td></tr><tr><td><code>whenComplete</code></td><td>获取前一个线程的结果或异常，消费</td><td>不影响上一线程返回值</td></tr><tr><td><code>exceptionally</code></td><td>线程异常执行，配合 whenComplete 使用</td><td>有返回值</td></tr><tr><td><code>handle</code></td><td>相当于 whenComplete + exceptionally</td><td>有返回值</td></tr></tbody></table>

> **等待 2 个线程都执行完**

<table><thead><tr><th>方法</th><th>说明</th><th>有无返回值</th></tr></thead><tbody><tr><td><code>thenCombine</code></td><td>2 个线程都要有返回值，等待都结束，结果合并转换</td><td>有返回值</td></tr><tr><td><code>thenAcceptBoth</code></td><td>2 个线程都要有返回值，等待都结束，结果合并消费</td><td>无返回值</td></tr><tr><td><code>runAfterBoth</code></td><td>2 个线程无需要有返回值，等待都结束，执行其他逻辑</td><td>无返回值</td></tr></tbody></table>

> **等待 2 个线程任一执行完**

<table><thead><tr><th>方法</th><th>说明</th><th>有无返回值</th></tr></thead><tbody><tr><td><code>applyToEither</code></td><td>2 个线程都要有返回值，等待任一结束，转换其结果</td><td>有返回值</td></tr><tr><td><code>acceptEither</code></td><td>2 个线程都要有返回值，等待任一结束，消费其结果</td><td>无返回值</td></tr><tr><td><code>runAfterEither</code></td><td>2 个线程无需有返回值，等待任一结束，执行其他逻辑</td><td>无返回值</td></tr></tbody></table>

> **多个线程等待**

<table><thead><tr><th>方法</th><th>说明</th><th>有无返回值</th></tr></thead><tbody><tr><td><code>anyOf</code></td><td>多个线程任一执行完返回</td><td>有返回值</td></tr><tr><td><code>allOf</code></td><td>多个线程全部执行完返回</td><td>无返回值</td></tr></tbody></table>

参考文章
----

*   [https://blog.csdn.net/yy139926/article/details/128917473](https://blog.csdn.net/yy139926/article/details/128917473)
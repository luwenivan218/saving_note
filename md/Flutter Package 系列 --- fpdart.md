> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7316367473317773338)

[fpdart](https://link.juejin.cn?target=https%3A%2F%2Fpub.dev%2Fpackages%2Ffpdart "https://pub.dev/packages/fpdart") 是 Dart 和 Flutter 中函数式编程的软件包。 fpdart 的目标是让开发人员能够在其应用程序中学习和使用函数式编程, 提供了许多其他函数式语言开箱即用的所有主要函数式编程类型。

```
# pubspec.yaml
dependencies: 
  fpdart: ^1.1.0


```

### Option

使用`Option` 处理缺失值。您可以将类型定义为 `Option` ，而不是使用 `null`，会产生更佳简洁的写法

```
/// 一般对于null 的处理
const int? a = null;
int result = 0;
if (a != null) { 
   result = a * 2;
} 
/// 使用Option 对null 的处理
final Option<int> b = none<int>(); 
final result = b.getOrElse(() => 0) * 2;


```

当函数在某些边缘情况下无法返回值时，这非常有用。如果不想抛出异常（不建议抛出异常的做法），可以使用 `Option`。

```
/// 抛出异常 Exception 方式
double divide(int x, int y) { 
  if (y == 0) { 、
    throw Exception('Cannot divide by 0!'); 
  } 
  return x / y;
} 
/// Option 方式
Option<double> divideF(int x, int y) {
  if (y == 0) { 
      return none(); 
  } 
  return some(x / y);
}


```

Either
------

在[使用 Dartz 针对网络请求进行封装](https://juejin.cn/user/175544904460503/posts "https://juejin.cn/user/175544904460503/posts")一文中我们也介绍了这个 Either，这里也简单的说明一下。

 `Either` 用于处理错误。 `Either` 可以是 `Right` 或 `Left`（不能两者兼而有之！）。 `Right` 包含函数成功时返回的值，`Left` 包含函数失败时的一些错误或消息。

```
/// 抛出异常的做法 
double divide(int x, int y) { 
   if (y == 0) { 
      throw Exception('Cannot divide by 0!'); 
   } 
   return x / y;
} 
/// Error 的做法
Either<String, double> divide(int x, int y) { 
   if (y == 0) { 
       return left('Cannot divide by 0'); 
   }
   return right(x / y);
 }


```

Task
----

`Task`允许您以更可组合的方式运行异步函数：

### 正常的异步

```
/// 一般做法
Future<int> async() { 
   return Future.value(10).then((value) => value * 10);
} 
/// Task 做法
Task<int> asyncF() { 
  return Task(() async => 10).map((a) => a * 10);
}


```

### 发生异常的异步

对于可能出现异常的情况，可以使用下面的做法。

```
/// catchError 的做法
Future<int> async() { 
   return Future<int>.error('Some error!') .then((value) => value * 10)    
                      .catchError((dynamic error) => print(error));
} 
/// TaskEither 的做法
TaskEither<String, int> async() { 
return TaskEither<String, int>( () async => left('Some error'), ).map((r) => r * 10);
}


```

结尾
--

更多的使用方法，请查看一下链接 [pub.dev/packages/fp…](https://link.juejin.cn?target=https%3A%2F%2Fpub.dev%2Fpackages%2Ffpdart%23eitherpackagesfpdartlibsrceitherdart "https://pub.dev/packages/fpdart#eitherpackagesfpdartlibsrceitherdart")
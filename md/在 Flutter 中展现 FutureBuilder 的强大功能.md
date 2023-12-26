> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7308623467606671375)

在 Flutter 中，管理异步操作并显示从互联网或其他来源获取的数据是一项常见任务。`FutureBuilder`小部件是处理异步操作并在数据准备好时更新 UI 的强大工具。

FutureBuilder
=============

`FutureBuilder`其核心是一个小部件，它简化了在 Flutter 中使用 Futures 的过程。它允许开发人员处理 一个 `Future`的状态并相应地更新 UI。该小部件采用将一个`Future`作为输入，以及根据 `Future`的当前状态定义要显示的内容的构建器函数。

FutureBuilder 的基本结构
===================

让我们看一下`FutureBuilder`的一个基本使用结构：

```
FutureBuilder<T>( 
  future: myFuture, 
  builder: (BuildContext context, AsyncSnapshot<T> snapshot) { 
    if (snapshot.connectionState == ConnectionState.waiting) { 
      // 显示加载指示器
      return CircularProgressIndicator(); 
    } else  if (snapshot .hasError) { 
      // 显示错误消息
      return Text( 'Error: ${snapshot.error} ' ); 
    } else { 
      // 显示数据
      return Text( 'Data: ${snapshot.data} ' ); 
    } 
  } , 
);


```

*   `future`：`Future`构建器所连接的 。这就是正在观察的异步操作`FutureBuilder`。
*   `builder`：每次状态更改时都会调用的回调函数`Future`。它需要 一个`BuildContext`和 `AsyncSnapshot`作为参数。
*   `BuildContext`：当前的构建上下文，这是创建小部件所需的。
*   `AsyncSnapshot<T>`：异步操作状态的快照。`data`它提供对、`error`和 的访问`connectionState`。

处理不同的状态
=======

1.  连接状态 ( `snapshot.connectionState`):

*   `ConnectionState.none`: 没有执行任何异步操作。
*   `ConnectionState.waiting`：异步操作正在进行中。
*   `ConnectionState.done`：异步操作完成。

2. 错误处理（`snapshot.hasError`）：

*   如果`snapshot.hasError`是`true`，则异步操作期间发生错误。可以使用 访问错误详细信息`snapshot.error`。

3.  数据可用性（`snapshot.data`）：

*   如果异步操作成功完成，则可通过 获取数据`snapshot.data`。

常见用例
====

1. 从 API 获取数据
=============

```
FutureBuilder( 
  future: fetchDataFromAPI(), 
  builder: (BuildContext context, AsyncSnapshot< String > snapshot) { 
    if (snapshot.connectionState == ConnectionState.waiting) { 
      return CircularProgressIndicator(); 
    } else  if (snapshot.hasError) { 
      return Text( '错误：${snapshot.error} ' ); 
    } else { 
      return Text( '来自 API 的数据：${snapshot.data} ' ); 
    } 
  }, 
);


```

2. 从本地存储读取数据
============

```
FutureBuilder( 
  future: readDataFromLocalStorage(), 
  builder: (BuildContext context, AsyncSnapshot< String > snapshot) { 
    if (snapshot.connectionState == ConnectionState.waiting) { 
      return CircularProgressIndicator(); 
    } else  if (snapshot.hasError) { 
      return Text( '错误: ${snapshot.error} ' ); 
    } else { 
      return Text( '本地数据: ${snapshot.data} ' ); 
    } 
  }, 
);


```

3. 身份验证
=======

```
FutureBuilder( 
  future: checkAuthenticationStatus(), 
  builder: (BuildContext context, AsyncSnapshot< bool > snapshot) { 
    if (snapshot.connectionState == ConnectionState.waiting) { 
      return CircularProgressIndicator(); 
    } else  if (snapshot.hasError) { 
      return Text( '错误: ${snapshot.error} ' ); 
    } else { 
      return snapshot.data 
          ? Text( '用户已通过身份验证' ) 
          : Text( '用户未经过身份验证' ); 
    } 
  }, 
);


```

注意事项
====

1.  单独的小部件树：考虑将小部件树分离到不同的小部件中，以保持构建方法的干净和可维护。
2.  错误处理：提供清晰且用户友好的错误消息，帮助用户理解和解决问题。
3.  加载指示器：始终包含加载指示器，以通知用户异步操作正在进行中。
4.  `snapshot.hasData`Snapshot.hasData 与 Snapshot.data：在访问之前进行检查`snapshot.data`以避免空引用错误。
5.  不可变状态：确保异步操作返回的数据或对象是不可变的，以防止意外的副作用。

结尾
==

在 Flutter 开发中，异步操作是不可避免的，并且`FutureBuilder`简化了处理和显示结果的过程。通过了解其结构、处理不同的状态并应用最佳实践，您可以充分利用其潜力来`FutureBuilder`创建响应灵敏且用户友好的 Flutter 应用程序。无论您是从 API 获取数据、从本地存储读取数据，还是管理身份验证状态，`FutureBuilder`Flutter 工具包中的多功能工具都是如此。
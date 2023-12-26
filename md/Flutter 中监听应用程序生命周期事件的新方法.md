> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7314500045738737715)

Flutter 3.13 引入了许多新功能和改进。其中之一是一个名为 `AppLifecycleListener` 的新类，它允许您监听 Flutter 应用程序的生命周期事件。与以前监听应用程序生命周期事件的方式相比，这是一个改进。在本文中，我将比较监听应用生命周期事件的新旧方法，并向您展示如何使用新的 `AppLifecycleListener` 类。

Flutter 3.13 之前
---------------

在 Flutter 3.13 之前，您可以使用 `WidgetsBindingObserver` mixin 监听应用生命周期事件。为此，必须将 `WidgetsBindingObserver` mixin 添加到您的 `State` 类中并重写 `didChangeAppLifecycleState` 方法。在 `didChangeAppLifecycleState` 方法中，您可以使用提供的状态 (`AppLifecycleState`) 值侦听应用生命周期事件。

```
class AppLifecyclePageOld extends StatefulWidget {
  const AppLifecyclePageOld({super.key});
  @override
  State<AppLifecyclePageOld> createState() => _AppLifecyclePageOldState();
}

class _AppLifecyclePageOldState extends State<AppLifecyclePageOld>
    // Use the WidgetsBindingObserver mixin
    with WidgetsBindingObserver {
  @override
  void initState() {
    super.initState();
    // Register your State class as a binding observer
    WidgetsBinding.instance.addObserver(this);
  }

  @override
  void dispose() {
    // Unregister your State class as a binding observer
    WidgetsBinding.instance.removeObserver(this);
    super.dispose();
  }

  // Override the didChangeAppLifecycleState method and
  // listen to the app lifecycle state changes
  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    super.didChangeAppLifecycleState(state);
    switch (state) {
      case AppLifecycleState.detached:
        _onDetached();
      case AppLifecycleState.resumed:
        _onResumed();
      case AppLifecycleState.inactive:
        _onInactive();
      case AppLifecycleState.hidden:
        _onHidden();
      case AppLifecycleState.paused:
        _onPaused();
    }
  }

  void _onDetached() => print('detached');
  void _onResumed() => print('resumed');
  void _onInactive() => print('inactive');
  void _onHidden() => print('hidden');
  void _onPaused() => print('paused');

  @override
  Widget build(BuildContext context) {
    return const Scaffold(
      body: Placeholder(),
    );
  }
}


```

让我们看看如何使用新的 `AppLifecycleListener` 类监听应用生命周期事件。

顺便说一句，“老” 监听应用程序生命周期事件的方式仍然可用。事实上，`AppLifecycleListener` 在底层使用了`WidgetsBindingObserver` mixin。

Flutter 3.13 （含）之后
------------------

`AppLifecycleListener` 类是一种侦听应用生命周期事件的新方法。

您可以使用它来监听应用程序生命周期事件，而无需直接使用 `WidgetsBindingObserver` mixin。为此，创建 `AppLifecycleListener` 类的实例并传递您想要侦听的所有回调。

```
class AppLifecyclePage extends StatefulWidget {
  const AppLifecyclePage({super.key});
  @override
  State<AppLifecyclePage> createState() => _AppLifecyclePageState();
}

class _AppLifecyclePageState extends State<AppLifecyclePage> {
  late final AppLifecycleListener _listener;
  @override
  void initState() {
    super.initState();
    // Initialize the AppLifecycleListener class and pass callbacks
    _listener = AppLifecycleListener(
      onStateChange: _onStateChanged,
    );
  }

  @override
  void dispose() {
    // Do not forget to dispose the listener
    _listener.dispose();
    super.dispose();
  }

  // Listen to the app lifecycle state changes
  void _onStateChanged(AppLifecycleState state) {
    switch (state) {
      case AppLifecycleState.detached:
        _onDetached();
      case AppLifecycleState.resumed:
        _onResumed();
      case AppLifecycleState.inactive:
        _onInactive();
      case AppLifecycleState.hidden:
        _onHidden();
      case AppLifecycleState.paused:
        _onPaused();
    }
  }

  void _onDetached() => print('detached');
  void _onResumed() => print('resumed');
  void _onInactive() => print('inactive');
  void _onHidden() => print('hidden');
  void _onPaused() => print('paused');

  @override
  Widget build(BuildContext context) {
    return const Scaffold(
      body: Placeholder(),
    );
  }
}


```

区别
--

俩者版本监听应用程序生命周期事件的方式非常相似。不过，为了了解 `AppLifecycleListener` 类的主要好处，让我们看一下 Flutter 应用生命周期的状态机图：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d5aa6745759f49c0b1fa1e25e867eb90~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1000&h=360&s=22731&e=png&a=1&b=badcfa)

此图显示了 Flutter 应用程序的所有可能状态。箭头显示状态之间可能的转换。当重写 `didChangeAppLifecycleState` 方法时，您只能监听实际的状态变化，例如当您的应用进入 `resumed` 状态时。但是，您无法收听状态之间的转换，例如您的应用是否从`inactive` 或`detached` 状态进入 `resumed` 状态。现在，`AppLifecycleListener` 类允许您监听状态之间的转换：

```
class AppLifecyclePage extends StatefulWidget {
  const AppLifecyclePage({super.key});
  @override
  State<AppLifecyclePage> createState() => _AppLifecyclePageState();
}

class _AppLifecyclePageState extends State<AppLifecyclePage> {
  late final AppLifecycleListener _listener;

  @override
  void initState() {
    super.initState();
    // Pass all the callbacks for the transitions you want to listen to
    _listener = AppLifecycleListener(
      onDetach: _onDetach,
      onHide: _onHide,
      onInactive: _onInactive,
      onPause: _onPause,
      onRestart: _onRestart,
      onResume: _onResume,
      onShow: _onShow,
      onStateChange: _onStateChanged,
    );
  }

  @override
  void dispose() {
    _listener.dispose();
    super.dispose();
  }

  void _onDetach() => print('onDetach');
  void _onHide() => print('onHide');
  void _onInactive() => print('onInactive');
  void _onPause() => print('onPause');
  void _onRestart() => print('onRestart');
  void _onResume() => print('onResume');
  void _onShow() => print('onShow');
  void _onStateChanged(AppLifecycleState state) {
    // Track state changes
  }

  @override
  Widget build(BuildContext context) {
    return const Scaffold(
      body: Placeholder(),
    );
  }
}


```

这是`AppLifecycleListener` 类的主要优点。它允许您监听应用程序生命周期状态之间的转换，并仅针对您感兴趣的转换执行必要的代码。

`onExitRequested`回调[​](https://link.juejin.cn?target=https%3A%2F%2Fkazlauskas.dev%2Fflutter-app-lifecycle-listener-overview%2F%23the-onexitrequested-callback "https://kazlauskas.dev/flutter-app-lifecycle-listener-overview/#the-onexitrequested-callback")
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

`AppLifecycleListener` 类还有一个名为 `onExitRequested` 的回调。此回调用于询问应用程序是否允许在退出可取消的情况下退出应用程序。例如，它可用于 MacOS 应用程序，其中用户在存在未保存的更改时尝试关闭应用程序：要取消退出，您需要从`onExitRequested`回调中返回 `AppExitResponse.cancel`。否则，返回`AppExitResponse.exit`则是允许应用程序退出。

```
class AppLifecyclePage extends StatefulWidget {
  const AppLifecyclePage({super.key});
  @override
  State<AppLifecyclePage> createState() => _AppLifecyclePageState();
}

class _AppLifecyclePageState extends State<AppLifecyclePage> {
  late final AppLifecycleListener _listener;
  @override
  void initState() {
    super.initState();
    _listener = AppLifecycleListener(
      // Handle the onExitRequested callback
      onExitRequested: _onExitRequested,
    );
  }

  @override
  void dispose() {
    _listener.dispose();
    super.dispose();
  }

  // Ask the user if they want to exit the app. If the user
  // cancels the exit, return AppExitResponse.cancel. Otherwise,
  // return AppExitResponse.exit.
  Future<AppExitResponse> _onExitRequested() async {
    final response = await showDialog<AppExitResponse>(
      context: context,
      barrierDismissible: false,
      builder: (context) => AlertDialog.adaptive(
        title: const Text('Are you sure you want to quit this app?'),
        content: const Text('All unsaved progress will be lost.'),
        actions: [
          TextButton(
            child: const Text('Cancel'),
            onPressed: () {
              Navigator.of(context).pop(AppExitResponse.cancel);
            },
          ),
          TextButton(
            child: const Text('Ok'),
            onPressed: () {
              Navigator.of(context).pop(AppExitResponse.exit);
            },
          ),
        ],
      ),
    );
    return response ?? AppExitResponse.exit;
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('App Lifecycle Demo'),
      ),
      body: Center(
        child: Text(
          '👋',
          style: Theme.of(context).textTheme.displayLarge,
        ),
      ),
    );
  }
}


```

结论[​](https://link.juejin.cn?target=https%3A%2F%2Fkazlauskas.dev%2Fflutter-app-lifecycle-listener-overview%2F%23conclusion "https://kazlauskas.dev/flutter-app-lifecycle-listener-overview/#conclusion")
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

`AppLifecycleListener` 类是一种监听应用生命周期状态的新方法，更重要的是，监听它们之间的转换。此外，`onExitRequested` 回调还简化了在退出可取消的情况下处理退出的过程。
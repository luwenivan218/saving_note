> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7309131109740724259)

响应式编程是一种以对数据随时间变化做出反应为中心的范式。它有助于自动传播更新，从而可确保 UI 和数据保持同步。用 Flutter 术语来说，这意味着只要状态发生变化就会自动触发重建。

### Observables

Observables 是响应式编程的核心。这些数据源会在数据发生变化时向订阅者发出更新。Dart 的核心可观察类型是 Stream。

当状态发生变化时，可观察对象会通知侦听器。从用户交互到数据获取操作的任何事情都可以触发此操作。这有助于 Flutter 应用程序实时响应用户输入和其他更改。

Flutter 有两种类型：`ValueNotifier`和`ChangeNotifier`，它们是类似 observable 的类型，但不提供任何真正的可组合性计算。

### ValueNotifier

Flutter 中的类`ValueNotifier`在某种意义上是响应式的，因为当值发生变化时它会通知观察者，但您需要手动监听所有值的变化来计算完整的值。

1、监听

```
  // 初始化
 final ValueNotifier<String> fristName = ValueNotifier('Tom');
 final ValueNotifier<String> secondName = ValueNotifier('Joy');
 late final ValueNotifier<String> fullName;

  @override
  void initState() {
    super.initState();
    fullName = ValueNotifier('${fristName.value} ${secondName.value}');

    fristName.addListener(_updateFullName);
    secondName.addListener(_updateFullName);
  }
  
  void _updateFullName() {
    fullName.value = '${fristName.value} ${secondName.value}';
  }


  //更改值得时候
  firstName.value = 'Jane'
  secondName.value = 'Jane'


```

2、使用 ValueListenableBuilder 更新 UI

```
 //通知观察者
 ValueListenableBuilder<String>(
   valueListenable: fullName,
   builder: (context, value, child) => Text(
      '${fristName.value} ${secondName.value}',
       style: Theme.of(context).textTheme.headlineMedium,
   ),
),


```

### ChangeNotifier

1、监听

```
  String _firstName = 'Jane';
  String _secondName = 'Doe';

  String get firstName => _firstName;
  String get secondName => _secondName;
  String get fullName => '$_firstName $_secondName';

  set firstName(String newName) {
    if (newName != _firstName) {
      _firstName = newName;
      // Triggers rebuild
      notifyListeners();
    }
  }

  set secondName(String newSecondName) {
    if (newSecondName != _secondName) {
      _secondName = newSecondName;
      // Triggers rebuild
      notifyListeners();
    }
  }
  
  //更改值得时候
  firstName.value = 'Jane'
  secondName.value = 'Jane'


```

2、使用 AnimatedBuilder 更新 UI

```
//通知观察者
AnimatedBuilder(
     animation: fullName,
     builder: (context, child) => Text(
     fullName,
     style: Theme.of(context).textTheme.headlineMedium,
    ),
 ),


```

### [get](https://link.juejin.cn?target=https%3A%2F%2Fpub.dev%2Fpackages%2Fget "https://pub.dev/packages/get")

GetX 将响应式编程变得非常简单。

*   您不需要创建 StreamController。
*   您不需要为每个变量创建一个 StreamBuilder。
*   你不需要为每个状态创建一个类。
*   你不需要创造一个终极价值。

使用 Get 的响应式编程就像使用 setState 一样简单。 让我们想象一下，您有一个名称变量，并且希望每次更改它时，所有使用它的小组件都会自动刷新。

1、监听以及更新 UI

```
//这是一个普通的字符串
var name = 'Jonatas Borges';
为了使观察变得更加可观察，你只需要在它的附加上添加“.obs”。
var name = 'Jonatas Borges'.obs;
而在UI中，当你想显示该值并在值变化时更新页面时，只需这样做。
Obx(() => Text("${controller.name}"));


```

### [Riverpod](https://link.juejin.cn?target=https%3A%2F%2Friverpod.dev%2Fdocs%2Fintroduction%2Fwhy_riverpod "https://riverpod.dev/docs/introduction/why_riverpod")

```
final fristNameProvider = StateProvider<String>((ref) => 'Tom');
final secondNameProvider = StateProvider<String>((ref) => 'Joy');
final fullNameProvider = StateProvider<String>((ref) {
  final fristName = ref.watch(fristNameProvider);
  final secondName = ref.watch(secondNameProvider);
  return '$fristName  $secondName';
});

//更改值得时候
 ref.read(fristNameProvider.notifier).state =
                     'Jane'
 ref.read(secondName.notifier).state =
                     'BB'



```

2、使用 ConsumerWidget 更新 UI

ref.read(surnameProvider) 读取某个值

ref.read(nameProvider.notifier).state 更新某个值的状态

```
class MyHomePage extends ConsumerWidget {
  const MyHomePage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) => 
     Scaffold(
        appBar: AppBar(
          title: const Text('Riverpod Example'),
        ),
        body: Text(
               ref.watch(fullNameProvider),
               style: Theme.of(context).textTheme.headlineMedium,
            ),
      )；
}



```

这里`Consumer`组件是与状态交互所必需的，`Consumer`有一个非标准`build`方法，这意味着如果您需要更改状态管理解决方案，您还必须更改组件而不仅仅是状态。

### [RxDart](https://link.juejin.cn?target=https%3A%2F%2Fpub.dev%2Fpackages%2Frxdart "https://pub.dev/packages/rxdart")

RxDart 将 [ReactiveX](https://link.juejin.cn?target=https%3A%2F%2Freactivex.io%2F "https://reactivex.io/") 的强大功能引入 Flutter，需要明确的逻辑来组合不同的数据流并对其做出反应。

存储计算值：它不会以有状态的方式直接存储计算值，但它确实提供了有用的运算符（例如 [distinctUnique）](https://link.juejin.cn?target=https%3A%2F%2Fpub.dev%2Fdocumentation%2Frxdart%2Flatest%2Frx%2FDistinctUniqueExtension%2FdistinctUnique.html "https://pub.dev/documentation/rxdart/latest/rx/DistinctUniqueExtension/distinctUnique.html")来帮助您最大限度地减少重新计算。

`RxDart` 库还有一个流行的类型被称为`BehaviorSubject`。响应式编程试图解决的核心问题是当依赖图中的任何值（依赖项）发生变化时自动触发计算。如果有多个可观察值，并且您需要将它们合并到计算中，Rx 库自动为我们执行此操作并且自动最小化重新计算以提高性能。

该库向 Dart 的现有流添加了功能。它不会重新发明轮子，并使用其他平台上的开发人员熟悉的模式。

1、监听

```
 final fristName = BehaviorSubject.seeded('Tom');
 final secondName = BehaviorSubject.seeded('Joy');
 
 /// 更新值
  fristName.add('Jane'),
  secondName.add('Jane'),



```

2、使用 StreamBuilder 更新 UI

```
 StreamBuilder<String>(
        stream: Rx.combineLatest2(
            fristName,
            secondName,
            (fristName, secondName) => '$fristName $secondName',
             ),
             builder: (context, snapshot) => Text(
               snapshot.data ?? '',
               style: Theme.of(context).textTheme.headlineMedium,
              ),
    ),


```

### [Signals](https://link.juejin.cn?target=https%3A%2F%2Fpub.dev%2Fpackages%2Fsignals "https://pub.dev/packages/signals")

Signals 以其`computed`功能介绍了一种创新、优雅的解决方案。它会自动创建反应式计算，当任何依赖值发生变化时，反应式计算就会更新。

1、监听

```
  final name = signal('Jane');
  final surname = signal('Doe');
  late final ReadonlySignal<String> fullName =
      computed(() => '${name.value} ${surname.value}');
  late final void Function() _dispose;

 @override
  void initState() {
    super.initState();
    _dispose = effect(() => fullName.value);
  }


```

2、使用 watch 更新 UI

```
Text(
    fullName.watch(context),
    style: Theme.of(context).textTheme.headlineMedium,
 ),


```
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7309693162368630835)

Flutter 介绍
----------

Flutter 是一个由 Google 开发的开源 UI 工具包，用于构建跨平台移动应用、Web 应用和桌面应用。它使用 Dart 编程语言，具有快速的热重载功能和丰富的现代化 UI 组件，使得开发者能够轻松构建高性能、美观的应用程序。

Flutter 是从 React 中汲取灵感的。在 React 中，一切都是组件，而在 Flutter 中，一切都是 Widget，从布局到文本、按钮、图像等都是 Widget。Widget 分为两种类型：StatelessWidget 和 StatefulWidget。StatelessWidget 是不可变的，它的 UI 在整个生命周期中保持不变；而 StatefulWidget 可以持有状态，并且可以在状态发生变化时重新构建 UI。

Flutter 中的 setState 和 React 中的 setState 类似，它们都用于通知框架进行 UI 更新。此外，Flutter 中的 build 方法和 React 中的 render 方法也有相似之处，它们都负责定义用户界面的结构和外观。在两个框架中，当状态发生变化时，都会触发重新构建（或重新渲染），以反映最新的数据和用户界面状态。

与 React Native 等跨平台框架不同，Flutter 不使用 WebView 来渲染，也不使用原生 UI 来渲染，而是自己实现了一套声明式的 UI 体系，使用 Skia 图形库在画布上直接绘制 UI 元素。这个画布在游戏引擎里叫 OpenGL，WebGL，在 Flutter 里，它叫 SGL（SkiaGraphicsLibrary）。Widget 本身既非视图，也不会直接绘制任何内容，而是 UI 及其底层创建真正视图对象的语义的描述。Flutter 的渲染过程是通过将 Widget 转换为一系列绘图指令，然后由 Skia 将这些指令转换为实际的像素，并最终渲染到屏幕上。

### 一些学习资源

Flutter 官网：[docs.flutter.dev/ui](https://link.juejin.cn?target=https%3A%2F%2Fdocs.flutter.dev%2Fui "https://docs.flutter.dev/ui")

练习 flutter：[dartpad.dev/](https://link.juejin.cn?target=https%3A%2F%2Fdartpad.dev%2F "https://dartpad.dev/")

Flutter 实用教程：[flutter.cn/docs/cookbo…](https://link.juejin.cn?target=https%3A%2F%2Fflutter.cn%2Fdocs%2Fcookbook "https://flutter.cn/docs/cookbook")

Widget 目录：[docs.flutter.dev/ui/widgets](https://link.juejin.cn?target=https%3A%2F%2Fdocs.flutter.dev%2Fui%2Fwidgets "https://docs.flutter.dev/ui/widgets")

Material 组件库：[docs.flutter.dev/ui/widgets/…](https://link.juejin.cn?target=https%3A%2F%2Fdocs.flutter.dev%2Fui%2Fwidgets%2Fmaterial "https://docs.flutter.dev/ui/widgets/material")

Flutter 基础命令介绍
--------------

`flutter doctor`: 检查本地 flutter 开发所需的环境

`flutter devices`: 查看连接的设备

`flutter emulators`: 查看已有的模拟器

`flutter emulators --launch <emulator id>`: 运行模拟器

`flutter create projectname`: 创建项目

`flutter run`: 运行项目

`flutter run -d xxx`: 指定某设备运行

`flutter build apk/ipa`: 打包 app

`flutter pub get`: 安装依赖包

`flutter pub add/remove xxx`: 添加 / 移除依赖包

`flutter create --template=package xxx`：创建一个依赖包

`flutter pub publish`：发布包

Widget 介绍及常用 Widget
-------------------

Flutter 是从 React 中汲取灵感的。在 React 中，一切都是组件，而在 Flutter 中，一切都是 Widget。从布局到文本、按钮、图像等都是 Widget。

Widget 分为两种类型：StatelessWidget 和 StatefulWidget。StatelessWidget 是不可变的，它的 UI 在整个生命周期中保持不变；而 StatefulWidget 可以持有状态，并且可以在状态发生变化时重新构建 UI。

在写 Widget 的时候，有时候需要用 const 关键字来创建那些在整个生命周期中保持不变的 Widget，这有助于提高性能。一般用在不需要重新构建或者不依赖于上下文的静态部分。

### 常用 Widget

Text：在应用程序中创建一系列样式文本。

Icon：一个 Material Design 的图标。

Image：展示一张图片。Image 通常使用 AssetImage 或 NetworkImage 等常见的图像加载器，并提供了一些常用的属性，比如 width、height、fit 等。

```
Image(
  image: NetworkImage('https://flutter.github.io/assets-for-api-docs/assets/widgets/owl.jpg'),
)

Image(
  image: AssetImage('assets/my_image.png'),
  width: 100,
  height: 100,
  fit: BoxFit.cover,
)


```

ElevatedButton：一个 Material Design 风格的凸起按钮。它通常用于表示主要操作，比如提交表单或执行重要的用户交互。

```
ElevatedButton(
    onPressed: () {
        // 处理按钮按下事件
        print('ElevatedButton Pressed!');
    },
    child: Text('Press Me'),
);


```

Container：创建矩形视觉元素。Container 可以具有 margin, padding, width, height，也可以使用变换。还可以用 BoxDecoration 装饰，例如背景、边框或阴影。

```
Container(
  width: 100,
  height: 100,
  color: Colors.blue,
  child: Text('Hello Flutter!'),
)


```

Row，Column，Expanded：在水平 (Row) 和垂直 (Column) 方向上创建灵活的布局。这些对象的设计基于 Web 的 Flexbox 布局模型。Expanded 用于在 Row、Column 等 Flex 布局中，使子组件占据剩余可用空间。

```
Row(
  mainAxisAlignment: MainAxisAlignment.center,  // 主轴居中
  crossAxisAlignment: CrossAxisAlignment.center, // 交叉轴居中
  children: [
    Text('Item 1'),
    Text('Item 2'),
    Icon(
      Icons.audiotrack,
      color: Colors.green,
      size: 30.0,
    ),
  ],
);

Column(
  mainAxisAlignment: MainAxisAlignment.center,  // 主轴居中
  crossAxisAlignment: CrossAxisAlignment.center, // 交叉轴居中
  children: [
    Text('Item 1'),
    Text('Item 2'),
  ],
);

// Expanded 充满全部剩余空间
Row(
    children: [
        Container(
            color: Colors.blue,
            height: 100,
            width: 100,
        ),
        Expanded(
            child: Container(
                color: Colors.red,
            ),
        ),
        Container(
            color: Colors.green,
            height: 100,
            width: 100,
        ),
    ],
);


```

Stack：用于堆叠多个 Widget，Stack 的 children 中使用 Positioned，可以将这些 Positioned 相对于 Stack 进行定位。基于 Web 的绝对定位布局模型。

```
Stack(
  children: [
    Positioned(
      top: 10,
      left: 10,
      child: Text('Top Left'),
    ),
    Positioned(
      bottom: 10,
      right: 10,
      child: Text('Bottom Right'),
    ),
  ],
)


```

### 布局 Widget

除了上面的 Container，CenterStack，Row，Column，Center，Positioned，还有一些别的

SizedBox：设置一个固定尺寸的空间，可以用来填充在两个元素之间，来让他们分隔一定的距离。

ListView：显示一个垂直滚动的列表。滚动一般都用这个。在处理大量数据时需要用到 builder 属性。

```
ListView(
  children: [
    ListTile(title: Text('Item 1')),
    ListTile(title: Text('Item 2')),
    ListTile(title: Text('Item 3')),
  ],
)


```

GridView：显示一个二维网格列表。

```
GridView.count(
  crossAxisCount: 2,
  children: [
    Card(child: Text('Item 1')),
    Card(child: Text('Item 2')),
    Card(child: Text('Item 3')),
  ],
)


```

Align：将子 Widget 相对于父 Widget 进行定位。

```
Align(
  alignment: Alignment.center,
  child: Text('Centered Text'),
)


```

Padding：在子 Widget 周围添加内边距。

```
Padding(
  padding: EdgeInsets.all(16.0),
  child: Text('Text with Padding'),
)


```

### 用 HTML 的方式理解 flutter 布局

容器：Container Widget，类似于 div，可以设置宽度，高度，背景颜色 (decoration，color)，子元素 child，padding。

居中：Center Widget 可以将它的子 Widget 同时以水平和垂直方向居中。

文本：Text Widget，其 style 属性可以传入元素 TextStyle 来描述样式，textAlign 设置文本对齐。

定位：如果需要设置绝对定位，指定一个 Widget 的绝对位置，可以把它放在一个 Positioned Widget 中，而 Positioned 则需被放在一个 Stack Widget 的 children 中。

转换：想要旋转、缩放一个 Widget，请将它放在 Transform Widget 中。使用 Transform Widget 的 alignment 和 origin 属性分别来指定转换原点的相对和绝对位置信息。

渐变：想要将线性颜色渐变在 Widget 的背景上应用，请将它嵌套在一个 Container Widget 中。接着将一个 BoxDecoration 对象传递至 Container 的 decoration，然后使用 BoxDecoration 的 gradient 属性来变换背景填充内容。

圆角：请用 BoxDecoration 对象的 borderRadius 属性。

阴影：使用 BoxDecoration 的 boxShadow 属性来生成一系列 BoxShadow widget。

```
final container = Container(
    width: 320,
    height: 240,
    color: Colors.grey[300],
    child: const Center(
        child: Text(
            'hello world',
            style: TextStyle(
                fontFamily: 'Georgia',
                color: Colors.red,
                fontSize: 24,
                fontWeight: FontWeight.bold,
            ),
            textAlign: TextAlign.center,
        ),
    ),
);


```

### StatelessWidget && StatefulWidget

StatelessWidget 是不可变的，其 UI 在整个生命周期中保持不变。

```
class MyStatelessWidget extends StatelessWidget {
  const MyStatelessWidget({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      child: Text('I am a StatelessWidget'),
    );
  }
}


```

StatefulWidget 可以有状态，并在状态发生变化时重新构建 UI。通过 setState 更新状态。

Flutter 中的 setState 和 React 中的 setState 类似，它们都用于通知框架进行 UI 更新。此外，Flutter 中的 build 方法和 React 中的 render 方法也有相似之处，它们都负责定义用户界面的结构和外观。在两个框架中，当状态发生变化时，都会触发重新构建（或重新渲染），以反映最新的数据和用户界面状态。

自定义有状态的 Widget 需要继承 StatefulWidget，并重写 createState 方法。类型是 `State<MyStatefulWidget>`，通过一个自定义的 State 类来创建 State 对象。自定义的 State 类需要继承 State 类 `State<MyStatefulWidget>`。

```
class MyStatefulWidget extends StatefulWidget {
  const MyStatefulWidget({super.key});

  @override
  State<MyStatefulWidget> createState() => _MyStatefulWidgetState();
}

class _MyStatefulWidgetState extends State<MyStatefulWidget> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Counter: $_counter'),
        ElevatedButton(
          onPressed: () {
            setState(() {
              _counter++;
            });
          },
          child: Text('Increment'),
        ),
      ],
    );
  }
}


```

### StatefulWidget 生命周期

StatefulWidget 的生命周期包括以下几个阶段：

1、createState()：在 StatefulWidget 被创建时调用，这个阶段发生在 StatefulWidget 被添加到树中时。通常在这个方法中会创建一个 State 对象并返回。

2、initState()：在 createState 阶段后调用，用于执行一次性的初始化操作。这个方法通常用于设置初始状态值、订阅流、初始化控制器等。

3、didChangeDependencies()：在 initState 之后调用，用于处理 State 对象依赖关系发生变化的情况。

4、build()：构建 Widget 树的方法，这个方法会在 didChangeDependencies 后调用，并且可能会被多次调用。

5、didUpdateWidget()：在 build 阶段后调用，用于处理 Widget 的更新，处理新旧 Widget 的对比操作，通常发生在父组件重建时。

6、setState()：当需要更新 UI 时调用，通常在这个方法中会修改 State 对象的状态并触发 UI 的重建。

7、deactivate()：当 State 对象从树中被移除时调用，通常是在调用 Navigator 进行页面导航时，离开当前页面时，或者在 TabBarView 等切换标签的情况下

8、dispose()：在 State 对象销毁时调用，通常是在页面被销毁时。释放资源、取消订阅流、清理控制器等操作应该在这个方法中执行。

调用了状态类中某个方法，里面调用了 setState (通知 Flutter 框架重新构建 Widget) → build () → didUpdateWidget (处理 Widget 的更新) → dispose(释放资源) → initState (初始化状态) → didChangeDependencies (处理依赖关系的变化) → build(根据新的状态构建 Widget)

父布局是 StatefulWidget，State 每调用一次更新 UI，会间接触发所有子 Widget 的销毁和重建。不要在 build() 的方法内部进行耗时操作。

```
class MyWidget extends StatefulWidget {
  const MyWidget({super.key});

  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  @override
  void initState() {
    super.initState();
    // Initialization code...
  }

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    // Code to handle dependencies change...
  }

  @override
  Widget build(BuildContext context) {
    // Widget creation code...
  }

  @override
  void didUpdateWidget(covariant MyWidget oldWidget) {
    super.didUpdateWidget(oldWidget);
    // Code to handle widget update...
  }

  @override
  void deactivate() {
    super.deactivate();
    // Code when the state is removed from the tree...
  }

  @override
  void dispose() {
    super.dispose();
    // Clean up code...
  }

  void _updateState() {
    setState(() {
      // Code to update state...
    });
  }
}


```

Material 应用程序
-------------

Material 应用程序从 MaterialApp Widget 开始，他包含 title，theme，和 home。home 中又调 Scaffold，有 appBar，body，floatingActionButton 等。

title：用于设置应用程序的标题，通常显示在任务管理器等地方。

home：设置应用程序的主页，即应用程序启动后显示的第一个界面。

theme：定义应用程序的整体主题，包括颜色、字体样式、按钮样式等。

```
MaterialApp(
  title: 'My Flutter App',
  theme: ThemeData(
    // 更多主题配置
  ),
  home: MyHomePage(),
)


```

Scaffold：是一个整体的应用程序框架，用于定义应用程序的基本结构，包括顶部栏 appBar、内容部分 body、底部导航栏 bottomNavigationBar、侧边栏 drawer、悬浮按钮 floatingActionButton 等。大多数应用程序的主页面都是由 Scaffold 构建的。

AppBar：顶部应用栏，通常包含应用程序的标题、图标和操作按钮。

drawer：设置应用程序的侧边栏，用户可以通过滑动屏幕边缘或点击按钮等方式打开侧边栏。

body：是 Scaffold 中用于显示主要内容的区域。可以放置任何 Widget，如 Text、ListView、Column 等。

bottomNavigationBar：设置底部导航栏，允许用户快速导航到不同的页面。

floatingActionButton：悬浮在界面上的圆形按钮，通常用于执行主要的操作，例如添加新内容或打开菜单。

```
Scaffold(
  appBar: AppBar(
    title: Text('My App'),
  ),
  drawer: Drawer(
    child: ListView(
      children: [
        // Drawer items...
      ],
    ),
  ),
  body: Center(
    child: Text('Hello, World!'),
  ),
  bottomNavigationBar: BottomNavigationBar(
    items: [
      BottomNavigationBarItem(
        icon: Icon(Icons.home),
        label: 'Home',
      ),
      BottomNavigationBarItem(
        icon: Icon(Icons.search),
        label: 'Search',
      ),
      // Additional items...
    ],
    // Other properties...
  ),
  floatingActionButton: FloatingActionButton(
    onPressed: () {
      // Handle button press
    },
    child: Icon(Icons.add),
  ),
)


```

处理事件
----

许多按钮类的 Widget 都具有 onPressed 属性，该属性用于指定按钮被按下时要执行的回调函数。

很多 Widget 不支持事件监听，所以我们得使用 GestureDetector widget 包裹，来检测各种输入手势，包括点击、双击、长按、拖动和缩放。

```
GestureDetector(
  onTap: () {
    print('Tapped!');
  },
  child: Container(
    width: 100,
    height: 100,
    color: Colors.green,
    child: Center(
      child: Text('Tap me'),
    ),
  ),
)


```

导航 & 路由
-------

### Navigator

Navigator widget 可以在应用程序中跳转到新的页面。

```
onPressed: () {
  Navigator.of(context).push(
    MaterialPageRoute(
      builder: (context) => const SongScreen(song: song),
    ),
  );
},
child: Text(song.name),


```

### 命名路由

命名路由是一种使用具体的字符串标识符来标记应用程序中不同页面的方法。通过使用命名路由，你可以更清晰地定义和管理导航。

首先，在你的 main.dart 中定义命名路由

```
MaterialApp(
  title: 'Named Routes Demo',
  // Start the app with the "/" named route. In this case, the app starts
  // on the FirstScreen widget.
  initialRoute: '/',
  routes: {
    // When navigating to the "/" route, build the FirstScreen widget.
    '/': (context) => const FirstScreen(),
    // When navigating to the "/second" route, build the SecondScreen widget.
    '/second': (context) => const SecondScreen(),
  },
)


```

导航到下一页，返回上一页，传递参数

```
// Within the `FirstScreen` widget
onPressed: () {
  // Navigate to the second screen using a named route.
  Navigator.pushNamed(
    context,
    '/second',
    arguments: 'Hello from HomePage!', // 跳转时的参数
    );
}

onPressed: () {
  // Navigate back to the first screen by popping the current route
  // off the stack.
  Navigator.pop(context);
}

// 获取参数
final String message = ModalRoute.of(context).settings.arguments as String;


```

### GoRouter

GoRouter 是一个第三方的 Flutter 路由库，提供了更多高级的导航特性。相比于默认的 Navigator，它支持更丰富的路由配置和参数传递。

添加 routes 配置

```
import 'package:go_router/go_router.dart';

// GoRouter configuration
final _router = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => HomeScreen(),
      routes: <RouteBase>[
        GoRoute(
          path: '/second',
          builder: (context, state) => DetailsScreen(),
        ),
      ],
    ),
  ],
);

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: _router,
    );
  }
}


```

页面跳转，传递参数

```
onPressed: () => context.go('/details'),

context.go(Uri(path: '/users/123', queryParameters: {'filter': 'abc'}).toString());


```

Flutter 渲染机制
------------

Widget → 对视图的结构化描述，存储视图渲染相关的 配置信息：布局、渲染属性、事件响应等信息。

Element → Widget 的实例化对象，承载视图构建的上下文数据，连接 Widget 到完成最终 渲染 的 桥梁。

RenderObject → 负责实现视图渲染的对象。

Flutter 的渲染过程简单分成这三步：

1、通过 Widget 树 生成对应的 Element 树；

2、创建相应的 RenderObject 并关联到 Element.renderObject 属性上；

3、构建成 RenderObject 树，深度优先遍历，确定树中各对象的位置和尺寸 (布局) ，把它们绘制到不同图层上。Skia 在 Vsync 信号同步时直接从渲染树合成 Bitmap，最后交给 GPU 渲染。

网络请求 dio
--------

dio 是一个强大的 HTTP 网络请求库，支持全局配置、Restful API、FormData、拦截器、 请求取消、Cookie 管理、文件上传 / 下载、超时、自定义适配器、转换器等。

dio 介绍：[github.com/cfug/dio/bl…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fcfug%2Fdio%2Fblob%2Fmain%2Fdio%2FREADME-ZH.md "https://github.com/cfug/dio/blob/main/dio/README-ZH.md")

### 简单使用

dio 支持 get，post 等请求，也可以并发请求。

```
import 'package:dio/dio.dart';

final dio = Dio();

void request() async {
  // get 请求
  Response response;
  response = await dio.get('/test?id=12&name=dio');
  print(response.data.toString());
  // The below request is the same as above.
  response = await dio.get(
    '/test',
    queryParameters: {'id': 12, 'name': 'dio'},
  );
  print(response.data.toString());

  // post请求
  response = await dio.post('/test', data: {'id': 12, 'name': 'dio'});

  // 并发请求
  response = await Future.wait([dio.post('/info'), dio.get('/token')]);
}


```

### dio 配置 & 拦截器

建议在项目中使用 Dio 单例，这样便可对同一个 dio 实例发起的所有请求进行一些统一的配置， 比如设置公共 header、请求基地址、超时时间等。

可以添加请求响应拦截器，来统一处理请求。添加 LogInterceptor 拦截器来自动打印请求和响应等日志（请不要在生产模式使用，除非你有意输出相关日志）。

```
import 'package:dio/dio.dart';

// dio配置
final dio = Dio();
void dioConfig() {
  dio.options.baseUrl = 'https://api.pub.dev';
  dio.options.connectTimeout = const Duration(seconds: 5);
  dio.options.receiveTimeout = const Duration(seconds: 10);

  dio.interceptors.add(
    InterceptorsWrapper(
      onRequest: (RequestOptions options, RequestInterceptorHandler handler) {
        // 如果你想完成请求并返回一些自定义数据，你可以使用 `handler.resolve(response)`。
        // 如果你想终止请求并触发一个错误，你可以使用 `handler.reject(error)`。
        _logger.d('onRequest');
        return handler.next(options);
      },
      onResponse: (Response response, ResponseInterceptorHandler handler) {
        // 如果你想终止请求并触发一个错误，你可以使用 `handler.reject(error)`。
        _logger.d('onResponse');
        return handler.next(response);
      },
      onError: (DioException error, ErrorInterceptorHandler handler) {
        // 如果你想完成请求并返回一些自定义数据，你可以使用 `handler.resolve(response)`。
        _logger.d('onError');
        return handler.next(error);
      },
    ),
  );

  dio.interceptors.add(
    LogInterceptor(
      logPrint: (o) => debugPrint(o.toString()),
    ),
  );
}


```

数据持久化 Shared preferences
------------------------

使用 Shared preferences 需要借助插件，来将简单的 key-value 数据存储到硬盘。支持的数据类型有 int、double、bool、String、`List<String>`。

数据可能会异步持久化到磁盘，并且不能保证写入返回后会持久化到磁盘，因此该插件不得用于存储关键数据。

```
// 获取 shared preferences.
final SharedPreferences prefs = await SharedPreferences.getInstance();

// 存储数据
// Save an integer value to 'counter' key.
await prefs.setInt('counter', 10);
// Save an boolean value to 'repeat' key.
await prefs.setBool('repeat', true);
// Save an double value to 'decimal' key.
await prefs.setDouble('decimal', 1.5);
// Save an String value to 'action' key.
await prefs.setString('action', 'Start');
// Save an list of strings to 'items' key.
await prefs.setStringList('items', <String>['Earth', 'Moon', 'Sun']);

// 读取数据
// Try reading data from the 'counter' key. If it doesn't exist, returns null.
final int? counter = prefs.getInt('counter');
// Try reading data from the 'repeat' key. If it doesn't exist, returns null.
final bool? repeat = prefs.getBool('repeat');
// Try reading data from the 'decimal' key. If it doesn't exist, returns null.
final double? decimal = prefs.getDouble('decimal');
// Try reading data from the 'action' key. If it doesn't exist, returns null.
final String? action = prefs.getString('action');
// Try reading data from the 'items' key. If it doesn't exist, returns null.
final List<String>? items = prefs.getStringList('items');

// 删除数据
// Remove data for the 'counter' key.
await prefs.remove('counter');


```

插件
--

Flutter 插件是用于与原生平台通信的桥梁，允许 Flutter 应用程序访问设备硬件、原生功能以及其他平台特定的功能。Flutter 插件的主要目的是提供一个接口，让 Flutter Dart 代码与底层原生代码（例如 Android 的 Java 或 Kotlin，iOS 的 Objective-C 或 Swift）进行通信。

### 常用插件

url_launcher：用于打开 URL 的插件，打开系统默认的浏览器或其他应用程序来处理指定的 URL。

webview_flutter：在 Flutter 应用程序中嵌入 WebView，它使用了 WebView 来加载和显示 Web 内容。用户可以直接在应用中浏览网页。

image_picker：用于从 Android 和 iOS 图像库中选择图像，也可以使用相机拍摄新照片。

camera：直接访问相机的预览、拍照和录制视频等功能。

dio：网络请求，上面已经介绍过。

path_provider：用于在不同平台上处理文件和目录路径。这对于在应用程序中读取和写入文件、缓存数据等操作非常有用。

shared_preferences：数据持久化，存储简单数据

sqflite：sqlLite 数据库

permission_handler：用于请求和检查应用程序权限。在移动应用程序中，为了执行某些敏感操作（例如访问相机、位置信息、存储等），需要用户授予应用程序相应的权限。

geolocator，location：获取地理位置信息。location 简单获取位置信息，geolocator 适用于复杂场景。

battery_plus：获取电池状态（full, charging, discharging）

### 开发自定义插件

[docs.flutter.dev/packages-an…](https://link.juejin.cn?target=https%3A%2F%2Fdocs.flutter.dev%2Fpackages-and-plugins%2Fdeveloping-packages%23step-1-create-the-package-1 "https://docs.flutter.dev/packages-and-plugins/developing-packages#step-1-create-the-package-1")

[docs.flutter.dev/platform-in…](https://link.juejin.cn?target=https%3A%2F%2Fdocs.flutter.dev%2Fplatform-integration%2Fplatform-channels "https://docs.flutter.dev/platform-integration/platform-channels")

开发调试
----

在项目中，添加 vscode 的调试脚本，`.vscode/launch.json`，接下来就可以用 vscode 打断点进行调试了

```
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "journey_app_flutter",
            "request": "launch",
            "type": "dart"
        },
        {
            "name": "journey_app_flutter (profile mode)",
            "request": "launch",
            "type": "dart",
            "flutterMode": "profile"
        },
        {
            "name": "journey_app_flutter (release mode)",
            "request": "launch",
            "type": "dart",
            "flutterMode": "release"
        }
    ]
}


```

一个完整的 Flutter 示例
----------------

```
import 'package:flutter/material.dart';
import 'package:logger/logger.dart';
import 'package:go_router/go_router.dart';
import 'package:dio/dio.dart';
import 'package:shared_preferences/shared_preferences.dart';

final _logger = Logger();

// 路由配置
final _router = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const MyHomePage(),
    ),
    GoRoute(
      path: '/second',
      builder: (context, state) => const SecondPage(),
    ),
  ],
);

// dio配置
final dio = Dio();
void dioConfig() {
  dio.options.baseUrl = 'https://lishuxue.site';
  dio.options.connectTimeout = const Duration(seconds: 5);
  dio.options.receiveTimeout = const Duration(seconds: 10);

  dio.interceptors.add(
    InterceptorsWrapper(
      onRequest: (RequestOptions options, RequestInterceptorHandler handler) {
        // 如果你想完成请求并返回一些自定义数据，你可以使用 `handler.resolve(response)`。
        // 如果你想终止请求并触发一个错误，你可以使用 `handler.reject(error)`。
        _logger.d('onRequest');
        return handler.next(options);
      },
      onResponse: (Response response, ResponseInterceptorHandler handler) {
        // 如果你想终止请求并触发一个错误，你可以使用 `handler.reject(error)`。
        _logger.d('onResponse');
        return handler.next(response);
      },
      onError: (DioException error, ErrorInterceptorHandler handler) {
        // 如果你想完成请求并返回一些自定义数据，你可以使用 `handler.resolve(response)`。
        _logger.d('onError');
        return handler.next(error);
      },
    ),
  );
}

void main() {
  dioConfig();
  // 主方法启动app
  runApp(const MyApp());
}

// 主App是个无状态Widget
class MyApp extends StatelessWidget {
  // 构造函数，并且将参数传给父类构造
  const MyApp({super.key});

  // 覆写 build 方法
  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'My Flutter Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple), // 定义颜色
        useMaterial3: true, // 启用Material 3
      ),
      routerConfig: _router, // 路由配置
    );
  }
}

class MyHomePage extends StatefulWidget {
  // 构造方法，并且调用父类构造并传参
  const MyHomePage({super.key});

  // 覆写 createState 方法
  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  // 不需要显式写构造方法

  // 自定义状态
  int _counter = 0;
  final int _currentIndex = 0; // 当前页面的索引，设置导航栏的active

  // 自定义的修改状态的方法
  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }

  void _decrementCounter() {
    setState(() {
      _counter--;
    });
  }

  @override
  void initState() {
    super.initState();
    _logger.d('initState');
  }

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    _logger.d('didChangeDependencies');
  }

  @override
  void didUpdateWidget(MyHomePage oldWidget) {
    super.didUpdateWidget(oldWidget);
    _logger.d('didUpdateWidget');
  }

  @override
  void deactivate() {
    super.deactivate();
    _logger.d('deactivate');
  }

  @override
  void dispose() {
    super.dispose();
    _logger.d('dispose');
  }

  // 覆写 build 方法
  @override
  Widget build(BuildContext context) {
    _logger.d('build');
    return Scaffold(
      // 顶部栏
      appBar: AppBar(
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        title: const Text('Home Page'),
      ),
      // 侧边栏
      drawer: Drawer(
        child: ListView(children: [
          ListTile(
            title: const Text('侧边菜单，Home'),
            selected: _currentIndex == 0, // active 状态
            onTap: () {
              // 处理侧边栏点击事件
              Navigator.pop(context);
              context.go('/');
            },
          ),
          ListTile(
            title: const Text('侧边菜单，Second'),
            selected: _currentIndex == 1,
            onTap: () {
              // 处理侧边栏点击事件
              Navigator.pop(context);
              context.go('/second');
            },
          ),
        ]),
      ),
      // 内容部分
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center, // 主轴居中
          crossAxisAlignment: CrossAxisAlignment.center, // 交叉轴居中
          children: [
            const Text('Home Page'),
            Text('$_counter'),
            ElevatedButton(
              onPressed: () {
                // 处理按钮按下事件
                _logger.d('button pressed');
              },
              child: const Text('Press Me'),
            ),
          ],
        ),
      ),
      // 底部导航栏
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _currentIndex, // active 状态
        items: const [
          BottomNavigationBarItem(
            icon: Icon(Icons.home),
            label: 'Home',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.search),
            label: 'Second',
          ),
        ],
        onTap: (int index) {
          // 处理底部导航栏点击事件
          if (index == 0) {
            context.go('/');
          }
          if (index == 1) {
            context.go('/second');
          }
        },
      ),
      // 悬浮按钮
      floatingActionButton: Column(
        mainAxisAlignment: MainAxisAlignment.end,
        children: [
          // FloatingActionButton.extended是FloatingActionButton的一个扩展版本，它允许在浮动操作按钮上显示文本和图标，以提供更多的信息。
          FloatingActionButton.extended(
            onPressed: _incrementCounter,
            icon: const Icon(Icons.add),
            label: const Text('Increment'),
            // 当我们在一个页面种使用两个FloatingActionButton却不覆写它们的heroTag属性，它们会默认使用同一个标识，以致于冲突报错。
            heroTag: 'Increment',
          ),
          const SizedBox(height: 10),
          FloatingActionButton.extended(
            onPressed: _decrementCounter,
            icon: const Icon(Icons.remove),
            label: const Text('Decrement'),
            heroTag: 'DecrementR',
          )
        ],
      ),
    );
  }
}

class SecondPage extends StatefulWidget {
  const SecondPage({super.key});

  // 覆写 createState 方法
  @override
  State<SecondPage> createState() => _SecondPageState();
}

class _SecondPageState extends State<SecondPage> {
  final int _currentIndex = 1; // 当前页面的索引，设置导航栏的active
  String _text = '初始值，点击发送请求后会变化';
  int _number = 0;

  // 网络请求
  void _request() async {
    try {
      Response response = await dio.get('/blog-api/common/homeinfo');
      var result = response.data;
      String text = result['one']['text'];
      _logger.d(text);
      setState(() {
        _text = text;
      });
    } on DioException catch (e) {
      _logger.d(e.message);
    }
  }

  // 设置持久化数据
  Future<void> _setSharedPreferencesData() async {
    final prefs = await SharedPreferences.getInstance();
    int number = (prefs.getInt('number') ?? 0) + 1;
    await prefs.setInt('number', number);
  }

  // 获取持久化数据
  Future<void> _getSharedPreferencesData() async {
    final prefs = await SharedPreferences.getInstance();
    final int number = prefs.getInt('number') ?? 0;
    setState(() {
      _number = number;
    });
  }

  @override
  void initState() {
    super.initState();
    _getSharedPreferencesData();
  }

  // 覆写 build 方法
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      // 顶部栏
      appBar: AppBar(
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        title: const Text('Second Page'),
      ),
      // 侧边栏
      drawer: Drawer(
        child: ListView(children: [
          ListTile(
            title: const Text('侧边菜单，Home'),
            selected: _currentIndex == 0, // active状态
            onTap: () {
              // 处理侧边栏点击事件
              Navigator.pop(context);
              context.go('/');
            },
          ),
          ListTile(
            title: const Text('侧边菜单，Second'),
            selected: _currentIndex == 1,
            onTap: () {
              // 处理侧边栏点击事件
              Navigator.pop(context);
              context.go('/second');
            },
          ),
        ]),
      ),
      // 内容部分
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center, // 主轴居中
          crossAxisAlignment: CrossAxisAlignment.center, // 交叉轴居中
          children: [
            const Text('Second Page'),
            Text(_text),
            ElevatedButton(
              onPressed: () {
                _request();
              },
              child: const Text('发送请求'),
            ),
            Text('上次存储的数据：$_number'),
            ElevatedButton(
              onPressed: () {
                _setSharedPreferencesData();
              },
              child: const Text('存储数据'),
            ),
            ElevatedButton(
              onPressed: () {
                _getSharedPreferencesData();
              },
              child: const Text('获取持久化的数据'),
            ),
          ],
        ),
      ),
      // 底部导航栏
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _currentIndex, // active状态
        items: const [
          BottomNavigationBarItem(
            icon: Icon(Icons.home),
            label: 'Home',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.search),
            label: 'Second',
          ),
        ],
        onTap: (int index) {
          // 处理底部导航栏点击事件
          if (index == 0) {
            context.go('/');
          }
          if (index == 1) {
            context.go('/second');
          }
        },
      ),
    );
  }
}


```
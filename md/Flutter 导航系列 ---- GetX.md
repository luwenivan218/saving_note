> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7131015737539297287)

携手创作，共同成长！这是我参与「掘金日新计划 · 8 月更文挑战」的第 11 天，[点击查看活动详情](https://juejin.cn/post/7123120819437322247 "https://juejin.cn/post/7123120819437322247")

为了编写更少的代码并在路由之间快速导航，我们将使用 GetX 包。我们将探索命名和未命名的路线。

我们首先在 pubspec.yaml 文件中添加 GetX 包并运行 pub get 命令。

```
dependencies:
 get: version


```

第一步：让我们在应用程序的根目录中使用 **GetMaterialApp() 包装我们的小部件。**

```
import 'package:flutter/material.dart';
import 'package:get/get_navigation/src/root/get_material_app.dart';

import 'home_screen.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
const MyApp({Key? key}) : super(key: key);

// This widget is the root of your application.
@override
Widget build(BuildContext context) {
  return GetMaterialApp(
    title: 'Flutter Demo',
    theme: ThemeData(
    primarySwatch: Colors.*blue*,
  ),
  home: HomeScreen(),
);}
}


```

导航到没有命名路线的不同页面：
---------------

跳转到新屏幕页面：

```
Get.to(NextScreen());


```

要返回上一个屏幕：

```
Get.back();


```

要转到新屏幕并从导航历史记录中删除当前路线：

```
Get.off(NextScreen());


```

要跳转到新屏幕并从导航历史记录中删除所有以前的路线：

```
Get.offAll(NextScreen());


```

从新屏幕获取数据：

```
Get.back(result: 'success');

var data = await Get.to(Payment());



```

导航到具有命名路线的不同页面：
---------------

在 GetMaterialApp() 上定义路线：

```
void main() {
  runApp(
    GetMaterialApp(
      initialRoute: '/',
      unknownRoute: GetPage(name: '/notfound', page: () => UnknownRoutePage()), //like 404 page in web
      getPages: [
        GetPage(name: '/', page: () => MyHomePage()),
        GetPage(name: '/second', page: () => Second()),
        GetPage(
          name: '/third',
          page: () => Third(),
          transition: Transition.zoom  
        ),
      ],
    )
  );
}


```

转到新的命名路线页面：

```
Get.toNamed("/NextScreen");


```

进入有参数的新页面：

```
Get.toNamed("/NextScreen", arguments: 'Get is the best');


```

在下一个屏幕上获取参数：

```
print(Get.arguments);


```

使用路由名称的动态参数转到新页面：

```
Get.offAllNamed("/NextScreen?device=phone&id=354&);


```

在新屏幕上，获取以下参数：

```
print(Get.parameters['id']);
print(Get.parameters['name']);


```

为了进一步简化整个应用程序的导航，我们将创建一个名为 **routes.dart 的文件。** 该文件将包含我们整个应用程序中的路线信息。

```
import 'package:flutter/material.dart';
import 'package:get/get.dart';
import 'package:get_routes/routes/routes.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return GetMaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Get Route Managment',
      initialRoute: '/home',
      getPages: appRoutes(),
    );
  }
}


```

```
import 'package:get/get.dart';
import 'package:get_routes/pages/home.page.dart';
import 'package:get_routes/pages/second.page.dart';
import 'package:get_routes/pages/third.page.dart';

appRoutes() => [
      GetPage(
        name: '/home',
        page: () => HomePage(),
        transition: Transition.leftToRightWithFade,
        transitionDuration: Duration(milliseconds: 500),
      ),
      GetPage(
        name: '/second',
        page: () => SecondPage(),
        middlewares: [MyMiddelware()],
        transition: Transition.leftToRightWithFade,
        transitionDuration: Duration(milliseconds: 500),
      ),
      GetPage(
        name: '/third',
        page: () => ThirdPage(),
        middlewares: [MyMiddelware()],
        transition: Transition.leftToRightWithFade,
        transitionDuration: Duration(milliseconds: 500),
      ),
    ];

class MyMiddelware extends GetMiddleware {
  @override
  GetPage? onPageCalled(GetPage? page) {
    print(page?.name);
    return super.onPageCalled(page);
  }
}


```

这个 Widget 有大量的属性，尽管我们将重点关注我们需要管理路由的属性： **InitialRoute**：我们传递将在应用程序启动时加载的初始路由。  
**GetPages**：我们必须传递应用程序的路由。我建议将路线放在单独的文件中。在这个例子中，我们将它放在 routes.dart 中，其中包含一个函数，该函数返回一个包含不同路由及其屏幕的数组。

GetPage 也接收相当多的属性，虽然唯一需要的是路径的名称（名称）和将被加载的屏幕（页面）。正如您在上面的代码中看到的，我们还可以配置转换，甚至在调用路由时启动 middelware。
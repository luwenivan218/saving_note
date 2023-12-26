> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7128007713501478920)

携手创作，共同成长！这是我参与「掘金日新计划 · 8 月更文挑战」的第 4 天，[点击查看活动详情](https://juejin.cn/post/7123120819437322247 "https://juejin.cn/post/7123120819437322247")

FocusWidget
===========

顾名思义，它管理焦点节点。它是 Flutter 焦点系统中最重要和功能最强大的部分。它管理其拥有的焦点节点与焦点树的附加和分离，管理焦点节点的属性和回调，并具有静态函数以启用附加到小部件树的焦点节点的发现。

这意味着 **Focus** 小部件是包含您想要关注的树中所有子项的小部件。当在传递给它的 **FocusNode 上调用** **requestFocus** 方法时，tab 遍历完成，子树可以获得焦点。传递 **FocusNode** 是可选的，如果我们传递它，我们会获得额外的功能，如果我们不传递，那么 **Focus** 小部件会创建自己的。**FocusNode** 的大部分功能最好通过更改 **Focus** 小部件的属性来访问。

以下示例显示了我们如何创建自定义的可聚焦小部件。**容器**中有一个对焦点事件做出反应的**文本。**

```
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);
  static const String _title = 'Focus Sample';
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: _title,
      home: Scaffold(
        appBar: AppBar(title: const Text(_title)),
        body: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: const <Widget>[MyCustomWidget(), MyCustomWidget()],
        ),
      ),
    );
  }
}

class MyCustomWidget extends StatefulWidget {
  const MyCustomWidget({Key? key}) : super(key: key);@override
  State<MyCustomWidget> createState() => _MyCustomWidgetState();
}

class _MyCustomWidgetState extends State<MyCustomWidget> {
  Color _color = Colors.white;
  String _label = 'Unfocused';
  @override
  Widget build(BuildContext context) {
    return Focus(
      onFocusChange: (bool focused) {
        setState(() {
          _color = focused ? Colors.black26 : Colors.white;
          _label = focused ? 'Focused' : 'Unfocused';
        });
      },
      child: Center(
        child: Container(
          width: 300,
          height: 50,
          alignment: Alignment.center,
          color: _color,
          child: Text(_label),
        ),
      ),
    );
  }
}


```

FocusWidget 键事件
---------------

**可以使用 Focus 小部件的** **onKey** 属性来监听和处理键事件。键事件从具有主要焦点的焦点节点开始。如果该节点未从该属性回调返回 **KeyEventResult.handled**，则为其父焦点节点提供事件，依此类推，直到遇到**焦点** **树的根。** 以下是 **Focus 小部件**，它吸收其子树未处理的每个键，但不能成为主要焦点。

```
Widget build(BuildContext context) {
  return Focus(
    onKey: (FocusNode node, RawKeyEvent event) => KeyEventResult.handled,
    canRequestFocus: false,
    child: child,
  );
}


```

_Focus 键事件在文本输入事件之前_处理，因此当 Focus 小部件围绕文本字段时处理键事件会阻止该键被输入到文本字段中。

下面是一个不允许在文本字段中输入字母 **a 的小部件示例。**

```
Widget build(BuildContext context) {
  return Focus(
    onKey: (FocusNode node, RawKeyEvent event) {
      return (event.logicalKey == LogicalKeyboardKey.keyA) ? KeyEventResult.handled : KeyEventResult.ignored;
},
   child: TextField(),
  );
}


```

**如果我们要做的是输入验证，那么可以使用 TextInputFormatter** 可能会更好地实现此示例的功能。当然该方法仍然很有用，例如，**Shortcuts** 小部件使用此方法在快捷方式成为文本输入之前对其进行处理。

控制 FocusWidget 的焦点
------------------

以下方法用于控制此节点及其后代如何参与 **Focus 小部件**的焦点过程。

1.  如果该`skipTraversal`属性为真，则该焦点节点不参与焦点遍历。如果在其焦点节点上调用它仍然是可`requestFocus`聚焦的，但是当焦点遍历系统正在寻找下一个要关注的事物时会被跳过。
2.  不出所料，该`canRequestFocus`属性控制此小部件管理的焦点节点是否`Focus`可用于请求焦点。如果此属性为 false，则`requestFocus`在节点上调用无效。这也意味着该节点被跳过以进行焦点遍历，因为它无法请求焦点。
3.  该`descendantsAreFocusable`属性控制该节点的后代是否可以接收焦点，但仍允许该节点接收焦点。此属性可用于关闭整个小部件子树的可聚焦性。这就是`ExcludeFocus`小部件的工作方式：它只是`Focus`具有此属性集的小部件。

FocusWidget 中的自动对焦
------------------

顾名思义，当我们想要在**焦点树**中自动聚焦 UI 元素时，此属性很有用。建议我们只给一个孩子在一个**焦点树**中自动对焦。

更改通知焦点小部件
---------

**onFocusChanged** 回调用于监听焦点更改，即使更改是用于添加或删除小部件以及焦点更改。

获取焦点节点
------

您可以通过不同的方法获取 **FocusNode 。**

1.  您可以将其作为属性传递给 **Focus** 小部件，同时从祖先那里获取它。
2.  您可以使用 **Focus.of(context)** 访问它，同时从后代获得它。
3.  当需要从当前上下文中获取它时，我们可以使用 **Builder 方法。** 以下示例显示了此方法。

```
Widget build(BuildContext context) {
  return Focus(
    child: Builder(
    builder: (BuildContext context) {
       final bool hasPrimary = Focus.of(context).hasPrimaryFocus;
       print('Building with primary focus: $hasPrimary');
       return const SizedBox(width: 100, height: 100);
    },),
  );
}


```

定时
--

请求焦点时，仅在当前构建阶段完成后才会生效。这意味着焦点更改总是延迟一帧，因为更改焦点会导致小部件树的任意部分重建，包括当前请求焦点的小部件的祖先。因为后代不能弄脏他们的祖先，所以它必须在帧之间发生，以便任何需要的更改都可以在下一帧发生。
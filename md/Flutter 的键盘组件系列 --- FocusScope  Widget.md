> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7128388355624009742)

携手创作，共同成长！这是我参与「掘金日新计划 · 8 月更文挑战」的第 5 天，[点击查看活动详情](https://juejin.cn/post/7123120819437322247 "https://juejin.cn/post/7123120819437322247")

FocusScope Widget
=================

就像 **FocusWidget** 管理 **FocusNode 一样，**  FocusScopeWidget **管理** FocusScopeNode  **。** 它是焦点树中的一个特殊节点，用作子树中焦点节点的分组机制。除非明确关注范围外的节点，否则焦点遍历将保持在焦点范围内。它还跟踪其子树中聚焦的节点的当前焦点和历史记录。这样，如果一个节点释放焦点或在它有焦点时被移除，焦点可以返回到之前有焦点的节点。如果没有后代有焦点，焦点范围也可用作返回焦点的地方。这允许焦点遍历代码具有用于查找下一个（或第一个）可聚焦控件移动到的起始上下文。

FocusScope 用来操作切换焦点, 隐藏软键盘
==========================

```
 /*设置 获取焦点*/
 FocusScope.of(context).requestFocus(focus1);
 /*传入空的焦点 则隐藏键盘*/
 FocusScope.of(context).requestFocus(FocusNode());


```

`FocusScopeNode`比较重要的方法是`setFirstFocus`，用來设定子作用域节点。

```
  void setFirstFocus(FocusScopeNode scope) {
    if (scope._parent == null) {
      // scope沒有父节点，將scope新增至当前节点下
      _reparent(scope);
    }
    if (hasFocus) {
      // 当前作用域存在焦点，_doRequestFocus將焦点移到scope上，同時记录节点。
      scope._doRequestFocus(findFirstFocus: true);
    } else {
      // 当前作用域不存在焦点，记录节点。
      scope._setAsFocusedChildForScope();
    }
  }


```

如何在 Flutter 中以编程方式关闭键盘
======================

1.  如何获取当前焦点节点：

```
FocusScopeNode currentfocus = FocusScope.of(context); 


```

2.  如何通过从当前焦点节点取消焦点来关闭键盘：

```
FocusScopeNode currentfocus = FocusScope.of(context); 
if (!currentfocus.hasPrimaryFocus &&
            currentFocus.focusedChild !=null) { 
    currentfocus.unfocus();
}


```

在上面的代码片段中，在获取当前的 FocusNode 之后，我们正在检查如果当前的 FocusNode 没有 [hasPrimaryFocus](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FFocusNode%2FhasPrimaryFocus.html "https://api.flutter.dev/flutter/widgets/FocusNode/hasPrimaryFocus.html") 并且它的 [focusedChild](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FFocusScopeNode%2FfocusedChild.html "https://api.flutter.dev/flutter/widgets/FocusScopeNode/focusedChild.html") 不为空，那么我们使用 [Focusmanager](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FFocusManager-class.html "https://api.flutter.dev/flutter/widgets/FocusManager-class.html")`unfocus()`调用当前聚焦的节点来移除焦点。

在 Flutter 中将焦点滑动到 TextField
===========================

```
var focusNode = FocusNode(); 
var textField = TextField(focusNode: focusNode);  FocusScope.of(context).requestFocus(focusNode); 
// or  focusNode.requestFocus(); 


```

*   [docs.flutter.io/flutter/wid…](https://link.juejin.cn?target=https%3A%2F%2Fdocs.flutter.io%2Fflutter%2Fwidgets%2FFocusNode-class.html "https://docs.flutter.io/flutter/widgets/FocusNode-class.html")
*   [flutter.dev/docs/cookbo…](https://link.juejin.cn?target=https%3A%2F%2Fflutter.dev%2Fdocs%2Fcookbook%2Fforms%2Ffocus "https://flutter.dev/docs/cookbook/forms/focus")

TextField 自动聚焦
==============

```
 TextField( autofocus: true, ); 


```

回到上一个焦点
=======

```
FocusScope.of(context).previousFocus();


```

如何取消具有自定义 FocusNode 的 TextField 的焦点？
====================================

一般答案是使用这段代码：`FocusScope.of(context).requestFocus(new FocusNode());` 但是当 TextField 有自定义 focusNode 时，这段代码似乎不起作用。 `SystemChannels.textInput.invokeMethod('TextInput.hide');`仍然有效，但它只删除了键盘，而字段本身仍处于选中状态。
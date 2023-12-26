> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7129488885410693133)

携手创作，共同成长！这是我参与「掘金日新计划 · 8 月更文挑战」的第 8 天，[点击查看活动详情](https://juejin.cn/post/7123120819437322247 "https://juejin.cn/post/7123120819437322247")

FocusableActionDetector 小部件
===========================

这个小部件提供了 **Actions**、**Shortcut**、**MouseRegion** 和 **Focus 小部件**的组合功能。这是 Flutter 控件用来实现控件的所有这些方面的东西。它只是使用组成小部件实现的。

控制焦点遍历
======

焦点遍历意味着让用户通过使用像 **Tab 键**这样的焦点遍历键来与小部件进行交互，这样用户就可以在不使用 **Pointing 输入设备**的情况下聚焦下一个小部件。Flutter 有一个用于此目的的内置机制，叫做 [**ReadingOrderTraversalPolicy**](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FReadingOrderTraversalPolicy-class.html "https://api.flutter.dev/flutter/widgets/ReadingOrderTraversalPolicy-class.html") 。

FocusTraversalGroup 小部件
=======================

这是一个小部件，可用于包装其他小部件，以便我们可以通过**标签遍历**它们。这个小部件几乎解决了几乎所有的问题，但它仍然提供了 [**TabTraversalPolicy**](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FFocusTraversalPolicy-class.html "https://api.flutter.dev/flutter/widgets/FocusTraversalPolicy-class.html") 来确定组内的排序。当我们想要更多地控制遍历操作，在不使用 [**ReadingOrderTraversalPolicy**](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FReadingOrderTraversalPolicy-class.html "https://api.flutter.dev/flutter/widgets/ReadingOrderTraversalPolicy-class.html") 时，也可以使用 [**OrderedTraversalPolicy**](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FOrderedTraversalPolicy-class.html "https://api.flutter.dev/flutter/widgets/OrderedTraversalPolicy-class.html")。

可围绕聚焦组件 [**FocusTraversalOrder**](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FFocusTraversalOrder-class.html "https://api.flutter.dev/flutter/widgets/FocusTraversalOrder-class.html") 小部件的 **order** 参数来确定顺序。 顺序可以是 [FocusOrder](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FFocusOrder-class.html "https://api.flutter.dev/flutter/widgets/FocusOrder-class.html") 的任何子类来提供。  
[NumericFocusOrder](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FNumericFocusOrder-class.html "https://api.flutter.dev/flutter/widgets/NumericFocusOrder-class.html")  
[LexicalFocusOrder](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FLexicalFocusOrder-class.html "https://api.flutter.dev/flutter/widgets/LexicalFocusOrder-class.html")  
[ReadingOrderTraversalPolicy](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FReadingOrderTraversalPolicy-class.html "https://api.flutter.dev/flutter/widgets/ReadingOrderTraversalPolicy-class.html")  
[FocusTraversalOrde](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FFocusTraversalOrder-class.html "https://api.flutter.dev/flutter/widgets/FocusTraversalOrder-class.html")  
[FocusOrder](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FFocusOrder-class.html "https://api.flutter.dev/flutter/widgets/FocusOrder-class.html")  
[NumericFocusOrder](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FNumericFocusOrder-class.html "https://api.flutter.dev/flutter/widgets/NumericFocusOrder-class.html")  
[LexicalFocusOrder](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FLexicalFocusOrder-class.html "https://api.flutter.dev/flutter/widgets/LexicalFocusOrder-class.html")

下面给出了一个使用 **FocusTraversalOrder** 和 **NumericFocusOrder** 的示例。

```
class OrderedButtonRow extends StatelessWidget {
  const OrderedButtonRow({Key? key}) : super(key: key);
  @override
  Widget build(BuildContext context) {
    return FocusTraversalGroup(
      policy: OrderedTraversalPolicy(),
      child: Row(
        children: <Widget>[
          const Spacer(),
          FocusTraversalOrder(
            order: NumericFocusOrder(2.0),
            child: TextButton(
              child: const Text('ONE'),
              onPressed: () {},
            ),
          ),
          const Spacer(),
          FocusTraversalOrder(
            order: NumericFocusOrder(1.0),
            child: TextButton(
              child: const Text('TWO'),
              onPressed: () {},
            ),
          ),
          const Spacer(),
          FocusTraversalOrder(
            order: NumericFocusOrder(3.0),
            child: TextButton(
              child: const Text('THREE'),
              onPressed: () {},
            ),
          ),
          const Spacer(),
        ],
      ),
    );
  }
}


```

焦点遍历策略
------

**FocusTraversalPolicy** 是在给定请求和当前焦点节点的情况下确定下一个小部件的对象。 请求（成员函数）类似于 **findFirstFocus**、**findLastFocus**、**next**、**previous** 和 **inDirection**。  
**FocusTraversalPolicy** 是具体策略的抽象基类，例如 **ReadingOrderTraveralPolicy**、**OrderedTraveralPolicy** 和 [DirectionalFocusTraversalPolicyMixin](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FDirectionalFocusTraversalPolicyMixin-class.html "https://api.flutter.dev/flutter/widgets/DirectionalFocusTraversalPolicyMixin-class.html") 类。  
为了使用 **FocusTraversalPolicy**，你可以给一个 **FocusTraversalGroup**，它确定策略将在其中有效的小部件子树。  
类的成员函数很少被直接调用：它们是供焦点系统使用的。

焦点管理器
=====

[**FocusManager**](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FFocusManager-class.html "https://api.flutter.dev/flutter/widgets/FocusManager-class.html") 维护当前系统的主要焦点。它只有少数对用户有用的焦点系统的 API, **FocusManager.instance.primaryFocus**, 它包含当前聚焦的焦点节点，也可以从全局 **primaryFocus** 字段访问。

其他有用的属性是 **FocusManager.instance.highlightMode** 和 **FocusManager.instance.highlightStrategy**。 这些在由需要 “触摸” 模式和“传统”（鼠标和键盘）模式之间切换以突出显示焦点的小部件使用。  
当用户使用触摸导航时，焦点突出显示通常是隐藏的，当他们切换到鼠标或键盘时，需要再次显示焦点突出显示，以便他们知道焦点是什么。  
亮点**策略**告诉焦点管理器如何解释设备使用模式的变化：它可以根据最近的输入事件在两者之间自动切换，也可以锁定在触摸或传统模式下。  
Flutter 中提供的小部件已经知道如何使用这些信息，因此只有在从头开始编写自己的控件时才需要它。您可以使用 **addHighlightModeListener** 回调来监听高亮模式的变化。
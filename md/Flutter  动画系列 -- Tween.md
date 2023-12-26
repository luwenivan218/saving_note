> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7311602994582421542)

[Tween](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fanimation%2FTween-class.html "https://api.flutter.dev/flutter/animation/Tween-class.html")
==========================================================================================================================================================================

**Tween** 是一个在两个值之间进行插值的类, 俗称补间动画。这些值可以是不同的数据类型，例如双精度型、颜色型或矩形型等。当具有系统的 **Tween** 类与子类不直接支持的独特数据类型或复杂动画时，我们也可以自定义 Tween 类来实现。

**注意：**  `Tween` 是 “in- Between” 的缩写，用于指代关键帧之间存在的帧。这些框架有助于创造两种最终状态之间过渡的错觉。

`Tween`是一个无状态对象，它根据指定的起点、终点和循环`duration`生成一组新值 。

下面给出的是定义 的语法`Tween`。

```
 Tween doubleTween = Tween<double>(begin: 0, end: 10); 
 value = doubleTween.transform(0.5);


```

`doubleTween` 是一个 对象，表示从 `0`到 `10`的 `Tween` 值 。 调用`transform()` 方法并将值设置为 `0.5`。它决定了一个动画周期的时间段。 `Tween` 动画主要与 `AnimationController` 一起使用，有助于向前或向后播放动画。 `AnimationController` 取一个 `duration` 值，该值确定完成一个动画周期的时间。

[SizeTween](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fanimation%2FSizeTween-class.html "https://api.flutter.dev/flutter/animation/SizeTween-class.html")
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

基于 Size 的过渡变化。

```
 AnimationController  controller = AnimationController(duration: const Duration(seconds: 2), vsync: this,)..repeat(reverse: true);
 Tween sizeTween = new SizeTween(begin: Size(0.0,0.0),end: Size(200.0,200.0));     
 Animation<Size> sizeAnimation = sizeTween.animate(_controller); 
 Container(width: sizeAnimation.value, height: sizeAnimation.value, color: Colors.red,);



```

[ColorTween](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fanimation%2FColorTween-class.html "https://api.flutter.dev/flutter/animation/ColorTween-class.html")
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

基于 Color 的颜色渐变。

```
  AnimationController   controller = AnimationController(duration: const Duration(seconds: 2), vsync: this,)..repeat(reverse: true);
  Tween colorTween = ColorTween(begin: Colors.red, end: Colors.purple,)
  Animation<double>  colorAnimation = colorTween.animate(controller);
  Container(width: 200, height: 200, color: colorAnimation.value,);


```

[RectTween](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fanimation%2FRectTween-class.html "https://api.flutter.dev/flutter/animation/RectTween-class.html")
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

基于坐标的渐变, 表示起始坐标和结束坐标以及对象的位置。

```
  AnimationController controller = AnimationController(duration: const Duration(seconds: 5), vsync: this,)..repeat();
  Tween reactTween = RectTween(begin: Rect.fromLTWH(0, 0, 100, 100), end: Rect.fromLTWH(450, 200, 100, 100),);
  Animation<Rect>  rectAnimation = .animate(controller);
  Positioned.fromRect(rect: rectAnimation.value, child: Container(color: Colors.red,),);


```

注意事项：使用的时候需要使用需要用 AnimatedBuilder 作为父组件

```
 AnimatedBuilder(animation: controller, builder: (context, child) {
    return xxxxx;
 },),


```

....

自定义 Tween
---------

要创建自定义 **Tween**，您需要扩展 **Tween** 类并覆盖 **lerp**（线性插值）函数。**lerp** 函数定义了动画持续时间内开始值和结束值之间发生插值的方式。这是一个简短的指南：

*   定义数据类型：确定要插值的值的类型。这可能是自定义类或更复杂的结构。
*   扩展 Tween 类：创建一个扩展 Tween 的新类，其中 T 是您的数据类型。
*   覆盖 Lerp 函数：实现 lerp 函数以在开始值和结束值之间进行插值。该函数接收一个双精度 t，表示当前的线性插值因子，一般在 0.0 到 1.0 之间。
*   使用补间：将自定义补间应用于**动画控制器**，以随着时间的推移对属性进行动画处理。

例如：慢慢地使蓝色方块出现在屏幕上

```
import 'package:flutter/material.dart';  
  
class DoubleTween extends Tween<double> {  
DoubleTween({double begin = 0.0, double end = 1.0}) : super(begin: begin, end: end);  
@override  
double lerp(double t) {  
   return begin! + (end! - begin!) * t;  
}}  
  
class CustomTweenAnimationWidget extends StatefulWidget {  
@override  
_CustomTweenAnimationWidgetState createState() => _CustomTweenAnimationWidgetState();  
}  
  
class _CustomTweenAnimationWidgetState extends State<CustomTweenAnimationWidget> with SingleTickerProviderStateMixin {  
late AnimationController _controller;  
late Animation<double> _animation;  
  
@override  
void initState() {  
super.initState();  
_controller = AnimationController(vsync: this, duration: const Duration(seconds: 2));  
_animation = DoubleTween(begin: 0.0, end: 1.0).animate(_controller)  
..addListener(() {  
setState(() {});  
});  
_controller.forward();  
}  
  
@override  
void dispose() {  
_controller.dispose();  
super.dispose();  
}  
  
@override  
Widget build(BuildContext context) {  
return Opacity(  
opacity: _animation.value,  
child: Container(  
width: 200,  
height: 200,  
color: Colors.blue,  
),  );  }  }


```

[TweenAnimationBuilder](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FTweenAnimationBuilder-class.html "https://api.flutter.dev/flutter/widgets/TweenAnimationBuilder-class.html")
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

```
const TweenAnimationBuilder({
    Key? key,
    required this.tween,
    required Duration duration,
    Curve curve = Curves.linear,
    required this.builder,
    VoidCallback? onEnd,
    this.child,
  })


```

TweenAnimationBuilder 参数：

*   **Tween** ：如果补间是 double 类型，那么它会从 ****Tween.begin**** 到 ****Tween.end 进行动画处理****
*   **Duration**：Duration 是动画运行的设定时间
*   **Builder**： Builder 帮助我们构建需要使用动画构建的内容。

TweenAnimationBuilder 与 [ImplicitlyAnimatedWidget](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FImplicitlyAnimatedWidget-class.html "https://api.flutter.dev/flutter/widgets/ImplicitlyAnimatedWidget-class.html") 和 [AnimatedWidget](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FAnimatedWidget-class.html "https://api.flutter.dev/flutter/widgets/AnimatedWidget-class.html") 的关系
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

ImplicitlyAnimatedWidget 有许多[子类](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FImplicitlyAnimatedWidget-class.html "https://api.flutter.dev/flutter/widgets/ImplicitlyAnimatedWidget-class.html")，它们提供常规小部件的动画版本。这些子类（如 [AnimatedOpacity](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FAnimatedOpacity-class.html "https://api.flutter.dev/flutter/widgets/AnimatedOpacity-class.html")、 [AnimatedContainer](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FAnimatedContainer-class.html "https://api.flutter.dev/flutter/widgets/AnimatedContainer-class.html")、[AnimatedSize](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FAnimatedSize-class.html "https://api.flutter.dev/flutter/widgets/AnimatedSize-class.html") 等）可以平滑地对其属性进行动画更改，并且它们比这个通用构建器更易于使用。[TweenAnimationBuilder](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FTweenAnimationBuilder-class.html "https://api.flutter.dev/flutter/widgets/TweenAnimationBuilder-class.html")（它本身也是 [ImplicitlyAnimatedWidget](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FImplicitlyAnimatedWidget-class.html "https://api.flutter.dev/flutter/widgets/ImplicitlyAnimatedWidget-class.html") 的子类）也可以方便地将任何小部件属性动画化为给定的目标值，即使框架（或第三方小部件库）未附带该小部件的动画版本。

[ImplicitlyAnimatedWidget](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FImplicitlyAnimatedWidget-class.html "https://api.flutter.dev/flutter/widgets/ImplicitlyAnimatedWidget-class.html")（包括这个 [TweenAnimationBuilder](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FTweenAnimationBuilder-class.html "https://api.flutter.dev/flutter/widgets/TweenAnimationBuilder-class.html")）内部还都管理一个 [AnimationController](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fanimation%2FAnimationController-class.html "https://api.flutter.dev/flutter/animation/AnimationController-class.html") 来驱动动画。如果您想要对动画进行更多控制，而不仅仅是设置目标值、 [持续时间](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FImplicitlyAnimatedWidget%2Fduration.html "https://api.flutter.dev/flutter/widgets/ImplicitlyAnimatedWidget/duration.html")和[曲线](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FImplicitlyAnimatedWidget%2Fcurve.html "https://api.flutter.dev/flutter/widgets/ImplicitlyAnimatedWidget/curve.html")，更多的请查看 [AnimatedWidget](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FAnimatedWidget-class.html "https://api.flutter.dev/flutter/widgets/AnimatedWidget-class.html") 。[AnimatedWidget](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FAnimatedWidget-class.html "https://api.flutter.dev/flutter/widgets/AnimatedWidget-class.html") 的一个示例是 [AnimatedBuilder](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FAnimatedBuilder-class.html "https://api.flutter.dev/flutter/widgets/AnimatedBuilder-class.html")，它的使用方式与 [TweenAnimationBuilder](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fwidgets%2FTweenAnimationBuilder-class.html "https://api.flutter.dev/flutter/widgets/TweenAnimationBuilder-class.html") 类似，但与后者不同的是，它由自己创建并管理的 [AnimationController](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fanimation%2FAnimationController-class.html "https://api.flutter.dev/flutter/animation/AnimationController-class.html") 提供支持。
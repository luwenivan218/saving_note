> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7311972542805540915)

模拟动画（如重力、摩擦力和弹性）
================

此动画在动画中模拟现实世界的行为，使它们感觉更加自然和真实。Flutter 提供了基于物理的动画系统，允许您创建反映现实世界物理属性（例如重力、摩擦力和弹性）的动画。

要在 Flutter 中使用基于物理的动画，您通常使用 **Simulation** 类，该类具有几个模仿现实世界物理的子类，例如 **GravitySimulation**、**FrictionSimulation** 和 **SpringSimulation**。这是一个基本方法：

*   选择正确的物理模拟：根据您想要实现的效果，选择反映所需行为的物理模拟类。例如，弹跳球将使用 SpringSimulation  **（** 也可以合并 **GravitySimulation**）。
*   配置模拟：使用适当的参数实例化模拟，例如起始位置、速度、加速度或张力和摩擦力。
*   创建物理驱动的动画：使用 **PhysicsSimulation** 和 **AnimationController** 来驱动动画。
*   连接到小部件：使用 **AnimatedBuilder 或类似的小部件**将动画附加到要设置动画的小部件的属性。

[GravitySimulation](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fphysics%2FGravitySimulation-class.html "https://api.flutter.dev/flutter/physics/GravitySimulation-class.html")
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

模拟重力的类，以加速度、起点、终点和起始速度初始化。在此示例中，我们提供 100 个单位的加速度，目标为 0 到 500，初始速度为零。

GravitySimulation 4 个参数：

*   **dacceleration(加速度)**：随着时间的推移连续应用的加速度。
*   **开始位置** ：相对于原点的初始位置。
*   **结束位置** ：距原点的距离大小（在任一方向上），认为模拟已 “完成”，该距离必须为正。
*   **初始速度** ：初始速度。

```
simulation = GravitySimulation(  
100.0, // acceleration  
0.0, // starting point  
500.0, // end point  
0.0, // starting velocity  
);


```

[FrictionSimulation](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fphysics%2FFrictionSimulation-class.html "https://api.flutter.dev/flutter/physics/FrictionSimulation-class.html")
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

摩擦参数的的滚动模拟

FrictionSimulation 3 个参数：

*   **drag(阻力)**：流体阻力系数范围从 0（无限摩擦）到 1（无摩擦）。
*   **velocity**：初始速度。
*   **position** ：初始位置。

```
  Offset pixelsPerSecond,
  final velocityPixelsPerSecond = pixelsPerSecond.distance;
  // 创建一个摩擦模拟。阻力参数从 1（无限摩擦）到 0（无摩擦）。
  //position参数告诉引擎在某个点应该开始刷新UI；例如，如果
  // 速度为 100 像素/秒，位置为 200，并且不会启动
  // 在第二秒发送更新的摩擦
  final simulation = FrictionSimulation(0.05, 0, velocityPixelsPerSecond);


```

[SpringSimulation](https://link.juejin.cn?target=url "url")
-----------------------------------------------------------

类似弹簧弹力的模拟。

SpringSimulation 3 个参数：

*   **mass(质量)**：mass 弹簧的质量，越小就越轻，弹性越强，越大越笨重，弹性越小
*   **stiffness（硬度）**：硬度越大，弹性越大。
*   **damping（阻尼系数）** ：阻尼系数 越大，弹性越小。

```
final spring = SpringSimulation(  
SpringDescription(  
mass: 0.5,  
stiffness: 50,  
damping: 6,  
),  
0, // starting point  
1, // ending point  
-2 * widget.dropHeight, // velocity  
);  


```

更多的模拟动画请点击下方的网站

[api.flutter.dev/flutter/phy…](https://link.juejin.cn?target=https%3A%2F%2Fapi.flutter.dev%2Fflutter%2Fphysics%2FSimulation-class.html "https://api.flutter.dev/flutter/physics/Simulation-class.html")
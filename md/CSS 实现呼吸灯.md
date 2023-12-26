> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7315314479204581391)

引言
--

在现代前端开发中，为网站添加吸引人的动画效果是提高用户体验的一种常见方式。其中，呼吸灯效果是一种简单而又引人注目的动画，适用于各种应用场景。本文将深入研究如何使用 CSS 来实现呼吸灯效果，包括基本的实现原理、动画参数调整、以及一些实际应用案例。

第一部分：基本的呼吸灯效果
-------------

### 1. 使用关键帧动画

呼吸灯效果的实现依赖于 CSS 的关键帧动画。我们可以使用 `@keyframes` 规则定义一个简单的呼吸灯动画。

```
@keyframes breathe {
  0% {
    opacity: 0.5;
    transform: scale(1);
  }
  50% {
    opacity: 1;
    transform: scale(1.2);
  }
  100% {
    opacity: 0.5;
    transform: scale(1);
  }
}

.breathing-light {
  animation: breathe 3s infinite;
}


```

在这个例子中，我们定义了一个名为 `breathe` 的关键帧动画，包含三个关键帧（0%、50%、100%）。在不同的关键帧，我们分别调整了透明度和缩放属性，从而形成了呼吸灯效果。

### 2. 应用到元素

接下来，我们将这个动画应用到一个元素上，例如一个 `div`。

```
<div class="breathing-light"></div>


```

通过给这个元素添加 `breathing-light` 类，我们就能够观察到呼吸灯效果的实现。可以根据实际需求调整动画的持续时间、缓动函数等参数。

第二部分：调整动画参数
-----------

### 1. 调整动画持续时间

通过调整 `animation` 属性的第一个值，我们可以改变动画的持续时间。例如，将动画持续时间改为 5 秒：

```
.breathing-light {
  animation: breathe 5s infinite;
}


```

### 2. 调整缓动函数

缓动函数影响动画过渡的方式。可以通过 `animation-timing-function` 属性来调整。例如，使用 `ease-in-out` 缓动函数：

```
.breathing-light {
  animation: breathe 3s ease-in-out infinite;
}


```

### 3. 调整动画延迟时间

通过 `animation-delay` 属性，我们可以设置动画的延迟时间。这在创建多个呼吸灯效果不同步的元素时很有用。

```
.breathing-light {
  animation: breathe 3s infinite;
  animation-delay: 1s;
}


```

第三部分：实际应用案例
-----------

### 1. 页面标题的动态效果

在页面的标题上应用呼吸灯效果，使其在页面加载时引起用户的注意。

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta >
  <title>Breathing Light Title</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <h1 class="breathing-light">Welcome to Our Website</h1>
</body>
</html>


```

```
@keyframes breathe {
  0% {
    opacity: 0.5;
    transform: scale(1);
  }
  50% {
    opacity: 1;
    transform: scale(1.2);
  }
  100% {
    opacity: 0.5;
    transform: scale(1);
  }
}

.breathing-light {
  animation: breathe 3s infinite;
}


```

### 2. 图片边框的动感效果

通过为图片添加呼吸灯效果，为静态图片增加一些生动感。

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta >
  <title>Breathing Light Image</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <div class="image-container">
    <img src="example-image.jpg" alt="Example Image" class="breathing-light">
  </div>
</body>
</html>


```

```
@keyframes breathe {
  0% {
    opacity: 0.5;
    transform: scale(1);
  }
  50% {
    opacity: 1;
    transform: scale(1.2);
  }
  100% {
    opacity: 0.5;
    transform: scale(1);
  }
}

.breathing-light {
  animation: breathe 3s infinite;
}

.image-container {
  display: inline-block;
  overflow: hidden;
  border: 5px solid #fff; /* 图片边框 */
}


```

结语
--

通过本文，我们深入探讨了如何使用 CSS 实现呼吸灯效果。从基本原理、动画参数调整到实际应用案例，希望读者能够深刻理解呼吸灯效果的制作过程，并能够在实际项目中灵活运用这一技术，为用户呈现更加生动有趣的页面效果。不仅如此，这也是提升前端开发技能的一种乐趣。
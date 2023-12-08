> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7309297523294830626)

> 本文翻译自 [The CSS property you didn't know you needed 👈](https://link.juejin.cn?target=https%3A%2F%2Fdev.to%2Ffrancescovetere%2Fthe-css-property-you-didnt-know-you-needed-3fk0 "https://dev.to/francescovetere/the-css-property-you-didnt-know-you-needed-3fk0")，作者：Francesco Vetere， 略有删改。

大家好！👋今天我想谈谈一个 CSS 功能，在我看来没有得到太多的关注。但它应该被关注！我说的就是`isolation`属性！

基本上它提供了更多对元素与文档其他部分交互的控制，并且通常是解决 `z-index` 问题的一种优雅的解决方案。

让我们从一个实际的例子开始。我做了一个简单的卡片：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c75b502f4eb4ec0ac72a6762444037f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=800&h=542&s=305328&e=png&b=7b48c6)

它的 HTML 代码非常简单，如下所示：

```
<article class="card">
  <header class="card__header">
    <img class="card__avatar" src="..." alt="..." />
    <span class="card__name">Daniel Clifford</span>
    <span class="card__role">Verified Graduate</span>
  </header>

  <p class="card__summary">I received ...</p>

  <p class="card__description">I was an EMT ...</p>
</article>


```

现在假设我们想在卡片的右上角添加一个引号图像，作为一个小小的装饰元素。大概是这样的：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cf1e7dc87c70435094fe1072a731c5f3~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=800&h=528&s=300580&e=png&b=7c4ac8)

🤔我们如何才能做到这一点呢？

我们可以将引号图像视为伪元素，并使用`position: absolute`将其放置在卡片上！让我们试试看💪

```
.card {
  position: relative;
}

.card::before {
  content: "";
  position: absolute;
  right: 2em;
  width: 150px;
  aspect-ratio: 1;
  background-image: url(https://raw.githubusercontent.com/francescovetere/testimonial-grid-section/main/images/bg-pattern-quotation.svg);
  background-repeat: no-repeat;
  background-size: contain;
}


```

这是我们得到的结果：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c5d49ad63b3f47109f80a0c18b083dea~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=800&h=535&s=290967&e=png&b=7d4bc9)

它看起来不错，但有一个小问题，我们想把这个图像放置在文本后面！

这就是我们的`z-index`发挥作用的地方。稍微设置一个`z-index: -1`就可以解决问题，对吗？嗯... 貌似不行 😕

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/da31f9f9df1f4f91bf7cddda12d353a5~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=800&h=544&s=300858&e=png&b=7a47c6)

😱 它... 消失了

如果你仔细想想，这完全是没问题的。每个元素，默认情况下，得到一个`z-index: 0`。所以，这就像是说页面上的每个元素默认都位于同一层。

当我们在伪元素上声明一个`z-index: -1`时，我们实际上是把它推到了默认层的下面！

🤔那我们如何解决这个问题？

理想情况下，我们希望有一个局部的、有范围的 `z-index` 的伪元素，当我希望我的伪元素有一个`z-index: -1`时，我希望的是 “**在所有内容后面，但不要在卡片本身后面！**“

这个概念实际上是内置在 CSS 中的，它被称为层叠上下文！

层叠上下文基本上是由一个元素定义的，该元素充当其子元素的`z-index`声明的根源。子元素无论其 `z-index` 多低，都不能脱离出其所在的层叠上下文。

默认情况下，一个普通的 HTML 文档将有一个单一的层叠上下文，每个元素都引用它。这就是为什么当我们将`z-index: -1`分配给我们的伪元素时，它会被隐藏到卡片后面，因为只有一个全局层叠上下文！

但是，我们也可以在任意元素上创建新的层叠上下文！

有很多方法可以在元素上创建新的层叠上下文。其中包括：

*   `transform`
*   `filter`
*   `opacity`
*   ...

完整列表请看 mozilla 文档：`developer.mozilla.org/en-US/docs/Web/CSS/CSS_positioned_layout/Understanding_z-index/Stacking_context#description`

所有这些方法都有一个问题，它们也有副作用！😢

它们创建了一个层叠上下文，这是我们想要的，但它们也做了一些其他的事情，也许我们不一定想要！ 比如为什么我必须对一个元素应用一个不透明度来创建一个层叠上下文，如果我一开始就不想应用任何不透明度呢？

🪄幸运的是，有一个简单的解决方案，它被称为`isolation`！

`isolation`是一个特殊的属性，其唯一的目的是在它所应用的元素上创建一个新的层叠上下文，且没有其他副作用。😄

它接受两个值，`auto`和`isolate`。`isolate`是我们需要的。

所以，这个问题的最终解决方案非常简单：

```
.card {
  position: relative;
  isolation: isolate;
}


```

到此文章分享就结束了，下次遇到类似的场景是不是有了新的解决方案呢？

**看完本文如果觉得有用，记得点个赞支持，收藏起来说不定哪天就用上啦～**

专注前端开发，分享前端相关技术干货，公众号：南城大前端（ID: nanchengfe）
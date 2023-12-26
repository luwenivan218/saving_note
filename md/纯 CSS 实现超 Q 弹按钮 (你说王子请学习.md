> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7315219036300935220)

起因
--

日前网上冲浪时看到一个非常 **Q 弹**可爱的按钮效果，并且是使用纯`CSS`实现的，抓紧时间分享出来和大家一起学习一下。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7d13d52b46334e8ca016cd83c674773b~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=504&h=291&s=338135&e=gif&f=237&b=000000)

第一个问题，CSS 如何记录点击的状态
-------------------

平时我们的点击状态都是要依赖于`JS`参与，监听一个`onClick`事件后记录点击的下标然后刷新状态更新样式名，我们这里使用`CSS`中的`input`标签配合`label`以及`:checked`来完成。

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8CAYAAAA6/NlyAAAAJElEQVRoge3BMQEAAADCoPVP7WkJoAAAAAAAAAAAAAAAAAAAbjh8AAFte11jAAAAAElFTkSuQmCC)

可点击查看效果。

第二个问题，CSS 如何判断方向
----------------

再次感叹一下`CSS`的灵活多变，很多看起来不可思议的事件其实只要转变思路也是可以轻松办到的，我们使用 `~`兄弟选择器配合`+`选择相邻兄弟元素可以确定方向是👈还是👉，原理如下图:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b786f57b145843d4bd38cc8ecde289cc~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=5058&h=1383&s=117223&e=png&b=252525)

比如我们第一个`input`为选中状态，则我们`:hover`到第四个`input`时我们的选择器写法

```
:checked ~input ~input ~input:hover


```

这样我们就能确定向👉的方向了，同理向左的情况如图

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5f08730837474651a99037e58b5298f8~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=5577&h=1134&s=89548&e=png&a=1&b=ffffff)

我们第四个`input`为选中状态，则我们`:hover`到第一个`input`时选择器写法为

```
:hover ~input ~input ~input:checked


```

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8CAYAAAA6/NlyAAAAJElEQVRoge3BMQEAAADCoPVP7WkJoAAAAAAAAAAAAAAAAAAAbjh8AAFte11jAAAAAElFTkSuQmCC)

最后一个问题，如何让按钮像果冻一样 Q 弹
---------------------

我们仔细观察效果图其实能看出来所谓的**果冻**一样的效果慢放看起来只是宽高在不断改变而已，至于为什么能看起来如此 **Q 弹**还是要用到我们的`transition-timing-function`

> **transition-timing-function** 是 CSS 属性，用于指定过渡效果的速度曲线。这个属性决定了过渡动画在开始、中间和结束时的速度变化。

分享两个调试`transition-timing-function`的网站

[cubic-bezier](https://link.juejin.cn?target=https%3A%2F%2Fcubic-bezier.com%2F "https://cubic-bezier.com/")

[easings.net](https://link.juejin.cn?target=https%3A%2F%2Feasings.net%2Fzh-cn%23 "https://easings.net/zh-cn#")

我们通过改变`transition-timing-function`让我们的**平移** **缩放** 看起来不再是匀速进行的，效果如下

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8CAYAAAA6/NlyAAAAJElEQVRoge3BMQEAAADCoPVP7WkJoAAAAAAAAAAAAAAAAAAAbjh8AAFte11jAAAAAElFTkSuQmCC)

结语
--

路漫漫其修远兮，吾将上下而求索！

感谢 [bounciness in the CSS](https://link.juejin.cn?target=https%3A%2F%2Fcodepen.io%2Fboom123bam%2Fpen%2FRwvmdmz "https://codepen.io/boom123bam/pen/Rwvmdmz")
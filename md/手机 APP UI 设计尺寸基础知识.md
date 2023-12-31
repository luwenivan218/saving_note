> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/29392903)

对于刚入行的 UI 设计师，往往会遇到一个基础问题，就是设计移动 APP 时，是用什么尺寸或者用哪种屏幕的尺寸是最适当的？有的同学花了很长时间也不知道怎么做，为了解决这个问题，今天设计师” 十萬個為什麼” 为大家从原理开始介绍一下移动端设计尺寸规范。

现象
--

首先说现象，大家都知道移动端设备屏幕尺寸非常多，碎片化严重。尤其是 Android，你会听到很多种分辨率：480×800, 480×854, 540×960, 720×1280, 1080×1920，而且还有传说中的 2K 屏。近年来 iPhone 的碎片化也加剧了：640×960, 640×1136, 750×1334, 1242×2208。

不要被这些尺寸吓倒。实际上大部分的 app [UI 设计](https://link.zhihu.com/?target=http%3A//www.shejidaren.com/tag/ui%25e8%25ae%25be%25e8%25ae%25a1)和移动端网页，在各种尺寸的屏幕上都能正常显示。说明尺寸的问题一定有解决方法，而且有规律可循。

像素密度
----

![](https://pic2.zhimg.com/v2-64e26161c24843adf957408e66605175_r.jpg)

手机 APP UI 设计尺寸基础知识

要知道，屏幕是由很多像素点组成的。之前提到那么多种分辨率，都是手机屏幕的实际像素尺寸。比如 480×800 的屏幕，就是由 800 行、480 列的像素点组成的。每个点发出不同颜色的光，构成我们所看到的画面。而手机屏幕的物理尺寸，和像素尺寸是不成比例的。最典型的例子，iPhone 3gs 的屏幕像素是 320×480，iPhone 4s 的屏幕像素是 640×960。刚好两倍，然而两款手机都是 3.5 英寸的。

所以，我们要引入最重要的一个概念：像素密度，也就是 PPI（pixels per inch）。这项指标是连接数字世界与物理世界的桥梁。

![](https://pic3.zhimg.com/v2-d83372135a4ce66b91fc7d3c234f4352_r.jpg)

Pixels per inch，准确的说是每英寸的长度上排列的像素点数量。1 英寸是一个固定长度，等于 2.54 厘米，大约是食指最末端那根指节的长度。像素密度越高，代表屏幕显示效果越精细。Retina 屏比普通屏清晰很多，就是因为它的像素密度翻了一倍。

倍率与逻辑像素
-------

![](https://pic4.zhimg.com/v2-52f29b7fe5a1a93770369eb75beeefe3_r.jpg)

再用 iPhone 3gs 和 4s 来举例。假设有个邮件列表界面，我们不妨按照 PC 端网页设计的思维来想象。3gs 上大概只能显示 4-5 行，4s 就能显示 9-10 行，而且每行会变得特别宽。但两款手机其实是一样大的。如果照这种方式显示，3gs 上刚刚好的效果，在 4s 上就会小到根本看不清字。

![](https://pic1.zhimg.com/v2-65958ddcac0aa1a50a50cd355d011e30_r.jpg)

在现实中，这两者效果却是一样的。这是因为 Retina 屏幕把 2×2 个像素当 1 个像素使用。比如原本 44 像素高的顶部导航栏，在 Retina 屏上用了 88 个像素的高度来显示。导致界面元素都变成 2 倍大小，反而和 3gs 效果一样了。画质却更清晰。

在以前，iOS 应用的资源图片中，同一张图通常有两个尺寸。你会看到文件名有的带 @2x 字样，有的不带。其中不带 @2x 的用在普通屏上，带 @2x 的用在 Retina 屏上。只要图片准备好，iOS 会自己判断用哪张，Android 道理也一样。

由此可以看出，苹果以普通屏为基准，给 Retina 屏定义了一个 2 倍的倍率（iPhone 6plus 除外，它达到了 3 倍）。实际像素除以倍率，就得到逻辑像素尺寸。只要两个屏幕逻辑像素相同，它们的显示效果就是相同的。

![](https://pic4.zhimg.com/v2-434b863e7c8d480834600ad1d787c7e7_r.jpg)

Android 的解决方法类似，但更复杂一些。因为 Android 屏幕尺寸实在太多，分辨率高低跨度非常大，不像苹果只有那么几款固定设备、固定尺寸。所以 Android 把各种设备的像素密度划成了好几个范围区间，给不同范围的设备定义了不同的倍率，来保证显示效果相近。像素密度概念虽然重要，但用不着我们自己算，iOS 与 Android 都帮我们算好了。

![](https://pic2.zhimg.com/v2-704632954db3e93a650ea48fd9c4cb8d_r.jpg)

如图所示，像素密度在 120 左右的屏幕归为 ldpi，160 左右的归为 mdpi，以此类推。这样，所有的 Android 屏幕都找到了自己的位置，并赋予了相应的倍率：

*   ldpi [0.75 倍]
*   mdpi [1 倍]
*   hdpi [1.5 倍]
*   xhdpi [2 倍]
*   xxhdpi [3 倍]
*   xxxhdpi [4 倍]

各型号 iPhone 的倍率比较简单，我们后面会讲到。那么 Android 手机那么多，具体怎么分？哪些手机是几倍的倍率呢？我们先看一张表，这是友盟 2014 年 10 月到 2015 年 03 月的数据：

![](https://pic1.zhimg.com/v2-38878d9fce43e1e5e311c55d68d8925c_r.jpg)

就目前市场状况而言，各种手机的分辨率可以这样粗略判断。虽然不全面，但至少在 1 年内都还有一定的参考意义：

*   ldpi 如今已绝迹，不用考虑
*   mdpi [320x480]（市场份额不足 5%，新手机不会有这种倍率，屏幕通常都特别小）
*   hdpi [480x800、480x854、540x960]（早年的低端机，屏幕在 3.5 英寸档位；如今的低端机，屏幕在 4.7-5.0 英寸档位）
*   xhdpi [720x1280]（早年的中端机，屏幕在 4.7-5.0 英寸档位；如今的中低端机，屏幕在 5.0-5.5 英寸档位）
*   xxhdpi [1080x1920]（早年的高端机，如今的中高端机，屏幕通常都在 5.0 英寸以上）
*   xxxhdpi [1440x2560]（极少数 2K 屏手机，比如 Google Nexus 6）

自然地，以 1 倍的 mdpi 作为基准。像素密度更高或者更低的设备，只需乘以相应的倍率，就能得到与基准倍率近似的显示效果。

不过需要注意的是，Android 设备的逻辑像素尺寸并不统一。比如两种常见的屏幕 480×800 和 1080×1920，它们分别属于 hdpi 和 xxhdpi。除以各自倍率 1.5 倍和 3 倍，得到逻辑像素为 320×533 和 360×640。很显然，后者更宽更高，能显示更多内容。所以，即使有倍率的存在，各种 Android 设备的显示效果仍然无法做到完全一致。

单位
--

不难发现，真正决定显示效果的，是逻辑像素尺寸。为此，iOS 和 Android 平台都定义了各自的逻辑像素单位。iOS 的尺寸单位为 pt，Android 的尺寸单位为 dp。说实话，两者其实是一回事。

单位之间的换算关系随倍率变化：

*   1 倍：1pt=1dp=1px（mdpi、iPhone 3gs）
*   1.5 倍：1pt=1dp=1.5px（hdpi）
*   2 倍：1pt=1dp=2px（xhdpi、iPhone 4s/5/6）
*   3 倍：1pt=1dp=3px（xxhdpi、iPhone 6）
*   4 倍：1pt=1dp=4px（xxxhdpi）

单位决定了我们的思考方式。在设计和开发过程中，应该尽量使用逻辑像素尺寸来思考界面。设计 Android 应用时，有的设计师喜欢把画布设为 1080×1920，有的喜欢设成 720×1280。给出的界面元素尺寸就不统一了。Android 的最小点击区域尺寸是 48x48dp，这就意味着在 xhdpi 的设备上，按钮尺寸至少是 96x96px。而在 xxhdpi 设备上，则是 144x144px。

无论画布设成多大，我们设计的是基准倍率的界面样式，而且开发人员需要的单位都是逻辑像素。所以为了保证准确高效的沟通，双方都需要以逻辑像素尺寸来描述和理解界面，无论是在标注图还是在日常沟通中。不要再说 “底部标签栏的高度是 96 像素，我是按照 xhdpi 做的” 这样的话了。

Web 怎么办
-------

移动端页面的绝对单位仍然是 px，至少代码里这么写，但它的道理也和 app 一样。由于像素密度是设备本身的固有属性，它会影响到设备中的所有应用，包括浏览器。前端技术可以善加利用设备的像素密度，只需一行代码，浏览器便会使用 app 的显示方式来渲染页面。根据像素密度，按相应倍率缩放。

可以通过这个测试页面 [http://greenzorro.github.io/demo/basic / 响应式断点. html](https://link.zhihu.com/?target=http%3A//greenzorro.github.io/demo/basic/%25E5%2593%258D%25E5%25BA%2594%25E5%25BC%258F%25E6%2596%25AD%25E7%2582%25B9.html) 来看看你的移动设备屏幕宽度，这是逻辑像素宽度。

以 iPhone 5s 为例，屏幕的分辨率是 640×1136，倍率是 2。浏览器会认为屏幕的分辨率是 320×568，仍然是基准倍率的尺寸。所以在制作页面时，只需要按照基准倍率来就行了。无论什么样的屏幕，倍率是多少，都按逻辑像素尺寸来设计和开发页面。只不过在准备资源图的时候，需要准备 2 倍大小的图，通过代码把它缩成 1 倍大小显示，才能保证清晰。

实际应用
----

大家最关心的还是实际运用，画布该怎么设置。我们就 iOS、Android、Web 三个平台来分别梳理一下。不过在这之前，我要为使用 PS 进行设计的朋友介绍一个小技巧。

![](https://pic3.zhimg.com/v2-00d54679bb689a961acae7b30394c02e_r.jpg)

之前我说过，我们要以逻辑像素尺寸来思考界面。体现到设计过程中，就是要把单位设置成逻辑像素。打开 PS 的首选项——单位与标尺界面，把尺寸和文字单位都改成点（Point）。这里的点也就是 pt，无论设计 iOS、Android 还是 Web 应用，单位都用它。当然，各平台单位名称还是要记住的。这里我们用的只是它的原理，不用在意名称。

要调节倍率，则通过图像大小里的 DPI 来控制。这个 DPI，其实就是 PPI，像素密度。有个常识大家都知道，屏幕上的设计 DPI 设成 72，印刷品设计 DPI 设成 300。为什么是这两个数字？

首先说 300，这和人眼的分辨能力有关。由于 1 英寸是固定长度，每 1 英寸有多少个像素点决定了画质清晰程度。之前说过，这就是像素密度，也就是 DPI。DPI 达到 300 以上，其细腻程度就会给人真实感，像真实世界中的物件。相反，DPI 只有 10 的话，在你一个食指指节大小的长度内只有 10 个像素，这明显就是马赛克了。所以印刷品要设成 300，才能保证清晰。

再说 72，这有一定的历史原因。最早的图形设计是在 mac 电脑上进行的，mac 本身的显示器分辨率就是 72。PS 中把图像 DPI 也设成 72，就能保证屏幕上显示的尺寸和打印尺寸相同，便于设计。72 的 PC 显示器分辨率逐渐成为一种默认的行业标准，这套规则就这么沿用下来。

![](https://pic1.zhimg.com/v2-a749db9208f5bbc755173469ff53af18_r.jpg)

现在回到正题，我们怎么通过 DPI 来调节倍率？既然屏幕本身的分辨率是 72，DPI 设成 72 刚好是 1 倍尺寸，那设成 72 的两倍就是倍率为 2 的屏幕了，就这么简单。

下面来看看 3 个平台各自的画布设置：

IPhone
------

iPhone 的屏幕尺寸各不相同，我说的是逻辑像素尺寸，这确实是让人很头疼的事情。如果想用一套设计涵盖所有 iPhone，就要选择逻辑像素折中的机型。

从市场占有率数据来看，目前最多的是 iPhone5/5s 的屏幕。倍率为 2，逻辑像素 320×568。上升势头最猛，未来有望登上第一的是 iPhone 6 的屏幕。倍率为 2，逻辑像素 375×667。

按照这两种尺寸来设计，都是比较主流的做法。可以兼顾短一些的 iPhone 4s，大一点的 6 plus 也不会过于空旷。

不过在切图的时候要注意，由于 iPhone 6 plus 的 3 倍图是由 2 倍图放大而来，所以位图要注意保证清晰。

Android
-------

都说 Android 碎片化严重，但它现在反而比 iOS 好处理。因为如今的 Android 屏幕逻辑像素已经趋于统一了：360×640，就看你设成几倍了。想以 xhdpi 为准，就把 DPI 设成 72×2=144。想以 xxhdpi 为准，就把 DPI 设成 72×3=216。

对于那些比较老的低端机，宽度是 480px 的那批，画面确实会小一些，显示内容会更少。稍微留意一下，重要内容尽量保持在界面中上部分。

当然，这些机型不出一年就会被边缘化，基本淘汰。现在能运转的也是当作功能机在用，软件多了必卡无疑，用户体验无从谈起。不作考虑也是 OK 的。

Web
---

手机端网页就没有统一标准了，比较流行的做法是按照 iPhone 5 的尺寸来设计。倍率 2，逻辑像素 320×568。

这样的做法比较实在，倍率 2 的屏幕无论在 iOS 还是 Android 方面都是主流，而且又是 2 倍屏幕中逻辑像素最小的。所以图片的尺寸可以保持在较小的水平，页面加载速度快。当然，缺点就是在倍率 3 的设备上看，图片不是特别清晰。

如果追求图片质量，愿意牺牲加载速度，那么可以按照最大的屏幕来设计。也就是 iPhone 6 plus 的尺寸，倍率 3，逻辑像素 414×736。

总结
--

移动端的尺寸比 PC 端复杂，关键就在倍率。但也正因为倍率的存在，把大大小小的屏幕拉回到同一水平线，得以保证一套设计适应各种屏幕。站在这条水平线的角度看，会发现它很好理解。

aHR0cDovL3dlaXhpbi5xcS5jb20vci8tWl90dFZIRTJ3eWtyUlhXOThydg== (二维码自动识别)
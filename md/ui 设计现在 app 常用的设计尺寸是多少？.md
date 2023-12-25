> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.zhihu.com](https://www.zhihu.com/question/427334315) ![](https://picx.zhimg.com/v2-4fe292b9488f02dc28d15be5546d2435_l.jpg?source=1def8aca)千锋设计学习营​

很多新人在开始做移动端 UI 设计的时候，确实往往对界面的一些尺寸规范不是十分清楚，很多时候都是凭借自己的感觉和经验去绘制界面，心里并没有一个清晰的概念，导致做出来的页面总是不那么尽如人意。所以我就想着干脆整理一篇详细的吧～

本文主要整理汇总了**一些界面设计（iOS 系统）中常用的一些尺寸规范和方法，如控件间距、适配、标注、切图等，设计师在设计时并不一定要严格遵守，但对这些规范应有所了解，并融会贯通。**

**目录**
------

*   1. 了解 iphone
*   2. 逻辑像素
*   3. PPI
*   4. 界面设计尺寸及栏高度
*   5. 边距和间距
*   6. 布局
*   7. 图片比例
*   8. 设计方式
*   9. 状态栏和导航栏
*   10. 大标题导航栏
*   11. 导航栏图标
*   12. 标签栏
*   13. 工具栏
*   14. 闪屏资源
*   15. 安全距离
*   16. 色彩
*   17. 字体
*   18. 启动图标
*   19. 控件
*   20. 标注

### **一、了解 iphone**

![](https://pica.zhimg.com/v2-6ca0be6dce4bcae622ab87b59dbb11bf_r.jpg?source=1def8aca)

每次苹果发布会 UI 设计师可能是最睡不着觉的人。 每次发布会苹果推出全新 iPhone 时，我们在 iPhone 平台上的 APP 应用程序必须跟随 iPhone 的尺寸、规范等特性调整设计稿。也就是说，几乎每次苹果发布会都是 UI 设计师加班的通知书！作为一个移动端 UI 设计师或者刚要进入 UI 这个行业，您必须对苹果所有生产过和现役的 iPhone 有所了解。

![](https://pic1.zhimg.com/v2-8bbb9cf2eee0b16552578fb961dceb04_r.jpg?source=1def8aca)

初代 iPhone：iPhone（一代）、iPhone3G（二代）、iPhone3GS（三代） iPhone 初代产品在 2007 年 1 月 9 日由史蒂夫 · 乔布斯在苹果发布会上正式发布。初代的 iPhone 产品的共同特点是：玻璃屏、屏幕清晰度普通、3.5 英寸屏。i 为了让大众习惯直接在手机上点图标（在此之前人机互动都是间接输入的：比如实体键盘、鼠标、[触控笔](https://www.zhihu.com/search?q=%E8%A7%A6%E6%8E%A7%E7%AC%94&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2501219763%7D)等），乔布斯发布了革命性的操作系统 iOS，手机的所有图标都是圆角：这样可以避免用户认为会刺到手指。所有图标和界面全部是拟物设计，这样可以更好地让用户理解哪些是可以点击操作的。按钮在手机上呈现的大小都是 7mm 左右，这是因为人类手指点击区域大概是 7mm – 9mm。系统充分地应用了[多点触控](https://www.zhihu.com/search?q=%E5%A4%9A%E7%82%B9%E8%A7%A6%E6%8E%A7&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2501219763%7D)的功能，你不仅仅可以点手机里的按钮，还可以进行长按、滑动、捏等手势操作。这些用户体验和人性化的设计在当时具有划时代的意义。随后，第二代产品 iPhone 3G、第三代产品 iPhone3GS 先后由乔布斯发布。这块 3.5 英寸屏的手机截图出来后的实际分辨率是 480x320px，所以如果你在当时做 UI 设计的话，那么做 APP 界面建图的尺寸就应该是 480x320px。

![](https://picx.zhimg.com/v2-2b8632e1e24caa487e6ad840b8a38a58_r.jpg?source=1def8aca)

iPhone 4 相关产品：iPhone 4（四代）、iPhone 4s（五代） iPhone 4 于 2010 年 6 月 8 日发布。iPhone 4 延续了 iPhone 一代的多点触摸（Multi-touch）屏界面，并首次加入[视网膜屏幕](https://www.zhihu.com/search?q=%E8%A7%86%E7%BD%91%E8%86%9C%E5%B1%8F%E5%B9%95&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2501219763%7D)、前置摄像头、陀螺仪、后置闪光灯，相机像素提高至 500 万。对我们设计师最重要的就是加上了视网膜屏 Retina。Retina 是苹果提出的标准，它的含义就是在应用场景的视距内让人无法看清单个像素。我们都知道如果你贴的够近，一般的屏幕上都会出现一格一格的像素矩阵。屏幕是由这些矩阵组成的。这种屏幕的问题就是稍微近一些我们就能看到那些网格和矩阵。如果我们希望用户得到最好的体验，自然是让用户看不到格子，那怎么办？答案就是：加大屏幕的密度。如果屏幕的密度到达一个指定的水平（当然也要取决于用户的视距，即用户与屏幕通常离多远），那么用户的眼睛就无法分辨出细小的像素颗粒了。这种屏幕就被称为 Retina 屏，也叫视网膜屏。这是用户体验的巨大进步，但是也是 UI 设计师的噩梦。原先的设计稿统统需要放大才可以在 iPhone4 里显示得完美：比如原来我们一个界面的尺寸是 480x320px，现在因为屏幕密度增加了一倍，我们就需要设计 640x960px 才够用。而且 3GS 并没有完全被淘汰，那么如何让一个 APP 适配两个不同尺寸的手机呢？于是每个 APP 内预装了两套切图，一套 480x320px 尺寸，也就是一倍图（@1x）；一套 960x640px 尺寸，也就是二倍图（@2x）。这两套图像资源的命名完全一样，只是二倍图结尾需要加上 @2x 标记它是高清尺寸，比如 home_icon@2x.png。

![](https://pica.zhimg.com/v2-19f52db31d4d2415cc7d99c200077977_r.jpg?source=1def8aca)

iPhone 5：iPhone 5（六代）、iPhone 5s 和 iPhone 5c（七代） iPhone 5 于 2012 年 9 月 13 日正式发布。iPhone5 在设计上带来了很多争议，因为 iPhone5 没有采用乔布斯认为人类最合适的手机尺寸 3.5 英寸屏，而是用了 4 英寸的屏幕。宽度没变而高度加长了。这样做的原因是市场上越大的手机越受欢迎。当时设计师也几近崩溃，因为又要搞适配了。原来 960x640px 的尺寸变为了 1136x640px，但是这个变化其实不大，就是高了点儿。于是 @2x 高清图的设计稿就变成了 640x1136px。因为 iPhone4 的手机看着也就是长了点儿，滑动就好了。除了闪屏这样的界面需要单独做 iPhone4、iPhone5、3GS 尺寸之外，其他界面仍然维持两套设计稿即可。

![](https://pic1.zhimg.com/v2-44886a14ac23654250fd92d657a090a1_r.jpg?source=1def8aca)

iPhone 6/7/8 和 iPhone Plus 从 iPhone6 到 iPhone8 这段时间苹果新手机的物理像素都是 750x1334px。而所有 Plus 手机的物理像素都是 1242x2208px。如果按照逻辑像素来计算，那么 iPhone6/7/8 的逻辑像素就是 375 x 667 pt（就是 750x1334 除以 2）；而 iPhone Plus 的逻辑像素就是 414 x 736 pt（就是 1242x2208 除以 3，因为这个屏幕太大了视距不同所以屏幕密度更高）。历史到这个时候，原来的手机全部被淘汰了。也就是说 iPhone6/7/8 成为了我们的设计标准，它的切图就是 @2x，iPhone Plus（1242x2208）使用 @3x。从此没有 @1x 倍图了，只存在一个假想的概念。

![](https://pic1.zhimg.com/v2-15173c8457ea549ac26bf3d5e278f188_r.jpg?source=1def8aca)

iPhone X：iPhone X（十一代） 苹果采用方案「齐刘海」，把四周处理成圆角的方式。IPhone X 和后续三款全面屏我们设计师需要注意的就是齐刘海。因为需要规避摄像头和麦克风听筒，所以状态栏比其他 iPhone 系列产品要高；而底部 Tab 栏因为最下方有圆角同样比其他 iPhone 系列要高。而且这两个边界是不应该放置任何操作功能的。也就是说只有看的份儿。 iPhone X 的物理像素是 1125 x 2436 px，而逻辑像素是 375 x 812 pt。也就是说如果你使用 Sketch 或者 Adobe XD 等工具设计界面的话，iPhone X 的宽度和 iPhone 6/7/8 是相同的；只是高了一些。那么如果需要出一套 iPhone X 效果图只需要把头和尾巴替换成新版即可。

![](https://pica.zhimg.com/v2-c6ee585d33265a659da20a3093e988cd_r.jpg?source=1def8aca)

iPhone XS Max：iPhone XS、iPhone XS Max、iPhone XR（十二代） 这三款产品的像素分辨率听上去会比较眼晕： iPhone XS Max：1242 x 2688 px iPhone XS：1125 x 2436 px iPhone XR：828 x 1792 px 但是如果我们用点的单位看就会得到： iPhone XS Max：414 x 896 pt （iPhone Plus 分辨率宽度） iPhone XS：375×812 pt （iPhone 6/7/8 分辨率宽度） iPhone XR：414 x 896 pt （iPhone Plus 分辨率宽度） 所以此次的 iPhone 都是比较友好的，如果使用矢量界面工具那么只需要调整设计稿头和尾巴即可，而切图其实和之前没有变化，不管用什么工具设计还是得出两套切图：@2x（750x1334px）、@3x（1242x2208px）即可。

### **二、** **像素**

![](https://picx.zhimg.com/v2-4ad887dfe25fdaa9e0bcda3df9ead0db_r.jpg?source=1def8aca)

逻辑像素（logic point）：逻辑像素的单位是 PT，它是按照内容的尺寸计算的单位。比如 iPhone 4 的逻辑像素是 480x320pt。但是由于每个逻辑的点因为视网膜屏密度增加了一倍，即 1pt=2px，那么其实 iPhone 4 的物理像素是 960x640px。iOS 开发工程师和使用 Sketch 和 AdobeXD [软件设计](https://www.zhihu.com/search?q=%E8%BD%AF%E4%BB%B6%E8%AE%BE%E8%AE%A1&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2501219763%7D)界面的设计师使用的单位都是 PT。 物理像素的单位就是我们常说的 pixel，简写成 PX。它是我们在 Photoshop 和切图中使用的单位，屏幕设计中最小的单位就是 1px 不可再分割。使用 Photoshop 设计移动端界面和网站的设计师使用的单位是 PX。如果您使用 Photoshop 设计界面，那么只需要记住所有 px 单位的数值和支持 Photoshop 的工具，如果使用 Sketch 或 Adobe XD 设计界面，那么只需要记住所有 pt 单位的数值和对应的工具即可。

### **三、 PPI**

PPI（pixels per inch）指的是屏幕分辨率的单位，表示的是每英寸显示的像素密度。屏幕的 PPI 值越高，那么这个屏幕每英寸能容纳的像素颗粒也就越多，那这个产品的画面的细节度也就越丰富。PPI 值大于 300 一般我们就无法用肉眼察觉出屏幕上的「马赛克」格子了。但是如果屏幕很大，那么需要呈现视网膜屏的 PPI 值也需要更大，所以 iPhone Plus 系列的 PPI 值比 iPhone6/7/8 要大。PPI 在我们设计的工作中其实关系不大，但理解它对于帮助我们理解为什么 iPhone4 比 iPhone3GS 实际像素大一倍有帮助。  

![](https://picx.zhimg.com/v2-33b09dd8bee6b499cd2418f42c43e816_r.jpg?source=1def8aca)

### **四、界面设计尺寸及栏高度**

目前主流的 iOS 设备主要有 iPhone SE（4 英寸）、iPhone 6s/7/8（4.7 英寸）、iPhone 6s/7/8 Plus（5.5 英寸）、iPhone X（5.8 英寸），它们都采用了 Retina 视网膜屏幕，其中 iPhone 6s/7/8 Plus 和 iPhone X 采用的是 3 倍率的分辨率，其他都是采用的 2 倍率的分辨率，无论是栏高度还是应用图标，设计师提供给开发人员的切片大小，前者始终是后者的 1.5 倍，并分别以 @3x 和 @2x 在文件名结尾命名，程序再根据不同分辨率自动加载 @3x 或者 @2x 的切片。  

![](https://pic1.zhimg.com/v2-fa981d8232fb72810228003dcb25b9b2_r.jpg?source=1def8aca)![](https://pica.zhimg.com/v2-726c3d326eed01bf3c406dfe6c3cdb3c_r.jpg?source=1def8aca)

通过前面的讲解和图示我们了解到 iPhone 不同设备的物理尺寸，那么他们的像素分辨率又是多少呢？也就是说我们用 Photoshop 做设计新建画布应该设置多大呢？另外，iOS 应用中的栏，包括状态栏、导航栏、标签栏、工具栏等，它们的高度又分别是多少呢？（注意：iOS 严格规定了各个栏的高度，这个是必须遵守的）通过下面的表格和图示来为你解答上面的问题。  

![](https://picx.zhimg.com/v2-162ee15df87125b303985c143226e0b2_r.jpg?source=1def8aca)![](https://picx.zhimg.com/v2-1d3d80150013dc670ee536fd90cf0022_r.jpg?source=1def8aca)

注意：在进行 iphone x 设计的时候我们依然可以采用熟悉的 iphone 7 的设计尺寸作为模板，只是高度增加了 290px，设计尺寸为 750*1624（@2x）。注意状态栏的高度由原来的 40px 变成了 88px，另外底部要预留 68px 的主页指示器的位置。  

![](https://picx.zhimg.com/v2-725018e6e19ca72512af5fb1aa0d1886_r.jpg?source=1def8aca)

### **五、边距和间距**

在移动端页面的设计中，页面中元素的边距和间距的[设计规范](https://www.zhihu.com/search?q=%E8%AE%BE%E8%AE%A1%E8%A7%84%E8%8C%83&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2501219763%7D)是非常重要的，一个页面是否美观、简洁、是否通透和边距间距的设计规范紧密相连，所以说我们有必要对它们进行了解。

**1. 全局边距**

全局边距是指页面内容到屏幕边缘的距离，整个应用的界面都应该以此来进行规范，以达到页面整体视觉效果的统一。全局边距的设置可以更好的引导用户竖向向下阅读。  

![](https://pic1.zhimg.com/v2-58c5b00653e60f658517100273146b5a_r.jpg?source=1def8aca)

在实际应用中应该根据不同的产品气质采用不同的边距，让边距成为界面的一种[设计语言](https://www.zhihu.com/search?q=%E8%AE%BE%E8%AE%A1%E8%AF%AD%E8%A8%80&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2501219763%7D)，常用的全局边距有 32px、30px、24px、20px 等等，当然除了这些还有更大或者更小的边距，但上面说到的这些是最常用的，而且有一个特点就是数值全是偶数。

**2. 卡片间距**

在移动端页面设计中卡片式布局是非常常见的布局方式，至于卡片和卡片之间的距离的设置需要根据界面的风格以及卡片承载信息的多少来界定，通常最小不低于 16px，过小的间距会造成用户的紧张情绪，使用最多的间距是 20px、24px、30px、40px，当然间距也不宜过大，过大的间距会使界面变得松散，间距的颜色设置可以与分割线一致，也可以更浅一些。 以 iOS（750*1334px）为例，设置页面不需要承载太多的信息，因此采用了较大的 70px 作为卡片间距，有利于减轻用户的阅读负担，而通知中心承载了大量的信息，过大的间距会让浏览变得不连贯和界面视觉松散，因此采用了较小的 16px 作为卡片的间距。

![](https://picx.zhimg.com/v2-256e403e61e37ae0fb3f4db6b97b901c_r.jpg?source=1def8aca)

**3、内容间距**

一款 APP 除了各种栏（状态栏、导航栏、标签栏、工具栏）和控件 icon 就是内容了，内容的布局形式多种多样，这里不去探讨内容具体应该如何去布局，我们来说一说内容的间距设置问题。 先来介绍一下[格式塔](https://www.zhihu.com/search?q=%E6%A0%BC%E5%BC%8F%E5%A1%94&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2501219763%7D)原则中的一个重要的原则就是邻近性，格式塔邻近性原则认为：单个元素之间的相对距离会影响我们感知它是否以及如何组织在一起，互相靠近的元素看起来属于一组，而那些距离较远的则自动划分组外，距离近的关系紧密。来看下图，左图中的圆在水平方向比垂直距离近，那么，我们看到了 4 排圆点，而右图则看成 4 列。  

![](https://pic1.zhimg.com/v2-dcf1b01ab51628efb1078941ee93b178_r.jpg?source=1def8aca)

在 UI 设计中内容布局时，一定要重视邻近性原则的运用，比如在下面这款轻芒阅读 APP 的主界面中，每一个应用名称都远离其他图标，与对应的图标距离较近，保持亲密的关系，也让用户的浏览变得更直观，如果应用名称与上下图标距离相同，就分不出它是属于上面还是下面，从而让用户产生错乱的感觉。  

![](https://picx.zhimg.com/v2-bf3df8493ce116efa68e6fd176ebd09d_r.jpg?source=1def8aca)![](https://picx.zhimg.com/v2-d93e0ed09105ab61a710ed53f810085d_r.jpg?source=1def8aca)

### **六、布局**

1、**列表式布局**

以我们最常用的微信和 QQ 为例，其「信息」页面都是采用的列表式布局，在采用这种布局形式的时候要注意列表舒适体验的最小高度是 80px。而自如 app 和唯品会 app 信息列表分别为 110px 和 106px。  

![](https://picx.zhimg.com/v2-53ab442f54dfe61a35106c3ba52b8372_r.jpg?source=1def8aca)

**2、卡片式布局**

其特点在于，每张卡片的内容和形式都可以相互独立，互不干扰，在使用卡片式布局的时候要注意卡片本身一般是白色的，而卡片之间的间距颜色一般是浅灰色，当然不同产品风格颜色可能不一样，有些是浅灰色偏蓝等。 双栏卡片的布局形式，比较常见于以图片信息为主导的 App。例如一些商城的商品陈列页面。这种形式与卡片式类似，但它能在一屏里显示更多的内容，至少 4 张卡片。同时，由于分开左右两栏的显示，用户可以更加方便地对比左右两栏卡片的内容。  

![](https://pic1.zhimg.com/v2-ba277933c65aad645167d6735794a913_r.jpg?source=1def8aca)

### **七、图片比例**

界面图片比例 对于图片的尺寸和比例没有严格的规范，设计师往往凭借经验和感觉设置一个看起来不错的尺寸，但事实上我们是有章可循的。运用科学的手段设置图片的尺寸，可以获得最优的方案，常见的图片尺寸有 16:9、4:3、3:2、1:1 和 1:0.618（黄金比例）等。

### **八、设计方式**

![](https://pica.zhimg.com/v2-f99fb37b7e63c554ba79606d5ae0d17d_r.jpg?source=1def8aca)![](https://pic1.zhimg.com/v2-4bfd0622ae67bde74ae2b5237a722917_r.jpg?source=1def8aca)

在 iPhone6/7/8 存量仍然很大的情况下，我们做设计稿仍然需要以 iPhone6/7/8 为尺寸来建图。从苹果官网下载好 UIKit，上面有我们需要的一切元素。这些元素有 PSD、Sketch 以及 XD 版本，不管用什么设计软件均可找到对应版本。打开之后你会发现苹果已经将我们所需要的规范元素准备好了。如果你需要一些弹窗或者控件，那么就在 UI Elements 里找。如果需要界面的尺寸模板，就在 Design Templates 找。所有文件都有两份，结尾带有 iPhoneX 的是为 iPhone X 系列设计的模板。没有标识的是为 iPhone6/7/8 设计的模板。

**九、状态栏和导航栏**

状态栏（Status Bars）就是 iPhone 最上方用来显示时间、运营商信息、电池电量的那个很窄的区域。

导航栏（Navigation Bars）就是状态栏之下的区域，一般来说导航栏中间是页面标题，左右是放置功能图标的区域。

在 iPhone6/7/8 设计中，状态栏的高度为 20pt（40px）。导航栏的高度是 44pt（88px）。这两个区域在 iOS7 代之后就进行了一体化设计。所以它们加起来的高度是 64pt（128px）。

在 iPhoneX 设计中，状态栏的高度为 44pt（132px）。导航栏的高度也是 44pt（132px）。这两个区域同样要进行一体化设计。所以它们加起来的高度是 84pt（264px）。

这里注意一下，因为 iPhone X 的 PPI 值为 458，所以并不是如 iPhone6/7/8 一样 1pt=2px 换算。  

![](https://picx.zhimg.com/v2-883b6adce9d2cf2be5f70f0e17a3152a_r.jpg?source=1def8aca)

**十、大标题导航烂**
------------

在最新的苹果设计中导航出现了一种新形式：大标题。出现这种形式就是为了减少视觉噪音，让内容更加突出。很明显大标题的设计很像报纸的[版式设计](https://www.zhihu.com/search?q=%E7%89%88%E5%BC%8F%E8%AE%BE%E8%AE%A1&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2501219763%7D)，在第一眼我们就会明白页面的主题。大标题导航栏的高度一般为 116pt（232px）：这包括了 20pt（40px）状态栏的高度，同时也能放得下 34pt（68px）的大标题和辅助信息（如返回等图标）。但是注意一下，大标题并不应该像传统导航一样常驻在页面之上，因为它太占空间了。所以在滑动页面时大标题会变成正常导航栏的 64pt（128px）的高度。当然如果设计稿为 iPhone X 那么数值需要另外换算。  

![](https://picx.zhimg.com/v2-b32a14da73dce489eac01f3806171bb1_r.jpg?source=1def8aca)

### **十一、导航栏图标**

图标作为文字的补充，在移动端中应用非常广泛。在导航栏区域上的功能诸如搜索、添加、更多、返回等均需要用图标来表达。说明：@2x 和 @3x 在逻辑像素单位是一样的，如果您使用如 Sketch、Adobe XD 等矢量工具设计，可以参照逻辑像素数值设计即可。但是如果您用 Photoshop 工具以 iPhone6/7/8 尺寸进行设计，就需按照 @2x 下的 px 单位数值设计。  

![](https://pica.zhimg.com/v2-cede67a8a4e3c9a3229419fd0c6a4626_r.jpg?source=1def8aca)

**十二、标签栏**
----------

Tab 就是点击的意思，Tab 栏（也叫标签栏）指的是 APP 底部的区域，如微信底部常驻有聊天、通讯录、发现、我的四个图标。iOS 规范中 Tab 栏一般有五个、四个、三个图标的形式。也就是把宽度平分为五、四、三份。

iPhone6/7/8 设计中，标签栏的高度为 49pt（98px）。Tab 栏的操作是最常用的，因为手指最方便点击而且这个栏是常驻在页面之上的。所以 Tab 栏的图标至关重要，因为很多用户可能因为看不懂图标而找不到重要功能的入口，通常我们会在 Tab 栏图标之下加上 10pt（20px）的注释文字，这个注释文字一般来说都是非常浅的浅灰色。

![](https://pic1.zhimg.com/v2-979c7fc4a5df7088bb17df46fe86931f_r.jpg?source=1def8aca)

标签栏图标 我们在标签栏上的图标一般来说 30pt（60px）大小左右，苹果给出了四种不同形状标签栏图标的尺寸参考供大家设计时参考。其意义是让不同外形的图标看上去是差不多大的，保证图标的平衡。标签栏图标的选中态应该是一个彩色，来区别于非选中状态。

![](https://picx.zhimg.com/v2-75b8c20553622f52ea153a07f6d8c669_r.jpg?source=1def8aca)![](https://picx.zhimg.com/v2-52809a123cc35daa5a523eb90bcb250f_r.jpg?source=1def8aca)![](https://picx.zhimg.com/v2-6b405d1a3370ee15551a2e6173b98987_r.jpg?source=1def8aca)

### **十三、工具栏**

我们在苹果自带浏览器底部就能看到工具栏。工具栏提供了和当前任务相关的操作和按钮，在滑动时可以收起。工具栏同 Tab 栏一样都是位于底部，但是高度略窄，它的高度是 44pt（88px）。

### **十四、闪屏资源**

由于闪屏是一张完整的静态满屏图片，而不是诸如其他页面一样是由切图和文本拼成的，所以闪屏的适配更简单粗暴：我们需要提供不同尺寸的闪屏效果。闪屏资源就是满尺寸的一张 png，上端不需要状态栏里的信息，程序会在开发完毕时自动在闪屏中补上状态栏里的信息。

![](https://pica.zhimg.com/v2-1af860afe800cf640a454cd6b3c87974_r.jpg?source=1def8aca)

### **十五、安全距离**

作为 iPhone 全面屏系列手机，齐刘海无疑是一个特征。但是全面屏给我们带来了使用上的问题：上下左右是圆角、顶部齐刘海使屏幕凹下一块。所以在带有圆角和齐刘海的红色标注区域不应该放置任何功能，仅可在上端放置状态栏，底部圆角区域留白。我们界面竖屏使用时左右临近手机边缘的区域不建议放任何操作，应留出一定的边距（Margin）。这个边距是多少呢？没有明确严格的规定，但是一般的 APP 会留出 8pt-24pt 不等的边距防止用户在屏幕边缘不好点击。不过内容展现却可以呈现在边距里。如果我们横屏使用手机时，左右同样不好点对吧？横屏同时还有令人闹心的「齐刘海」，所以同样左右需留出一定的边距来。所以我们就得到一个安全距离的矩形，内容可以完整地呈现在这个安全距离内。

![](https://pic1.zhimg.com/v2-553c04ce3ee31c0901c69bb488955245_r.jpg?source=1def8aca)

### **十六、色彩**

其实在 iPhone 上显示的色域要比我们作图时的 RGB 色域要广。所以在 iPhone 上设计怎样的颜色都可以。只要符合产品气质并且在[色彩心理学](https://www.zhihu.com/search?q=%E8%89%B2%E5%BD%A9%E5%BF%83%E7%90%86%E5%AD%A6&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2501219763%7D)理论上思考，用什么颜色是设计师的自由。官方建议的系统色彩如下：  

![](https://picx.zhimg.com/v2-7cab40572a54896f0ef7be7aaa49ff75_r.jpg?source=1def8aca)

### **十七、字体**

iOS 中英文使用的是 San Francisco（SF）字体。（下载地址：[https://developer.apple.com/fonts](https://link.zhihu.com/?target=https%3A//developer.apple.com/fonts)），中文使用的是苹方黑体。安装好以后你会发现中文苹方的字族有不少可供选择的粗细，那么我们设计界面时需要根据信息的逻辑权重分配粗细：标题应该较粗，而说明字体应该较细并且可以设计成灰色。其实字体的设计最重要的考量就是信息层级。苹果认为 APP 的字体信息层级有：大标题（Large Title）、标题一（Title 1）、标题二（Title 2）、标题三（Title 3）、头条（Headline）、正文（Body）、标注（Callout）、副标题（Subhead）、注解（Footnote）、注释一（Caption 1）、注释二（Caption 2）这几种。  

![](https://picx.zhimg.com/v2-74970b00ced5322228c66971d795023a_r.jpg?source=1def8aca)

注意一下，以上 HIG 的建议全部是针对英文 SF 字体而言的，中文字体需要我们灵活运用，以最终呈现效果为基准调整。在设计具体界面时我们一定要以用户的使用情景来考虑，把设计稿导入手机去思考行高与字体大小是否是可读的。10pt（20px）是手机上显示的最小字体，最大的应该是目前的大标题字体了，达到了 34pt（68px）。

### **十八、启动图标**

在设计模板还没有如今这么发达时，设计师需要设计启动图标（1024x1024px）之后按照程序员的要求切出几十个不同尺寸的图标。比如，在手机中 @3x 情况下桌面图标尺寸为 180x180px，在 @2x 情况下为 120x120px；在[应用商店](https://www.zhihu.com/search?q=%E5%BA%94%E7%94%A8%E5%95%86%E5%BA%97&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2501219763%7D)图标需要使用的尺寸是 1024x1024px；这个工作太烦人了，好在现在我们只需要专注在启动[图标设计](https://www.zhihu.com/search?q=%E5%9B%BE%E6%A0%87%E8%AE%BE%E8%AE%A1&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2501219763%7D)本身上了。在苹果给我们的这套资源中，有 Template-AppIcons-iOS 这个文件。打开这个文件，用我们自己设计的启动图标替换掉智能对象里的内容，你会发现所有尺寸的图标都变成了我们的图标。然后我们把背景隐藏，切出这些图标即可。图标设计建议使用 AI 等[矢量软件](https://www.zhihu.com/search?q=%E7%9F%A2%E9%87%8F%E8%BD%AF%E4%BB%B6&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2501219763%7D)，然后使用规范切出图像资源。  

![](https://picx.zhimg.com/v2-8d453a8ff0537666fe4327fa4670caa8_r.jpg?source=1def8aca)![](https://picx.zhimg.com/v2-3a1345fa1f972f4e81ea7d5f5806d354_r.jpg?source=1def8aca)![](https://pic1.zhimg.com/v2-3cfceee1005673f068f09b52ec0cba4a_r.jpg?source=1def8aca)

**十九、控件**

![](https://pic1.zhimg.com/v2-353d70d967483682321e96f33d295e0e_r.jpg?source=1def8aca)![](https://pic1.zhimg.com/v2-76b66ec3c2c587c142d5ebc2ce94ed0d_r.jpg?source=1def8aca)

控件包括：输入框、按钮、滑杆、页卡、开关等，在设计模板中已经全部列出。这里格外说明一下，为了让设计更符合整体产品品牌调性，这些控件都可以做成自定义的设计样式。但是会增加工作量和切图资源，所以一般我们在诸如设置界面这些无需太体现设计感的页面中都使用系统默认控件，而在一些品牌感需要强调的页面或产品（诸如白噪音产品、游戏等）则会使用自定义的样式。如果我们想自己设计控件，那么注意两件事：

第一，点击区域基本符合 44pt（88px）原则，也就是在手机上大小大概是 7mm-9mm，适合手指点击。

第二，要设计操作的不同状态，不要只设计一种状态。

### **二十、标注**

当界面设计定稿之后，设计师需要对界面进行标注给开发工程师在还原界面时进行参考。在一份设计稿中需要标注的内容是文字的字号大小、粗细、颜色、不透明度；界面的背景颜色、不透明度；各个图标、列表、文字之间的间距。  

![](https://pic1.zhimg.com/v2-5fc20b8aeb92ba32925d7ab3b5d43e13_r.jpg?source=1def8aca)

常用的标注工具有蓝湖、 Mark Man 、PX Cook 像素大厨

1. 蓝湖平台自动标注功能 将 Sketch 和 Adobe XD、Photoshop 的设计稿上传至蓝湖后，在蓝湖平台每个页面左侧有一个类似分享的图标，点击会获取一个网址，这个网址就是系统生成的自动标注。它会自动识别设计稿中字体大小和间距等，甚至有代码参考。

2. 使用 Px 像素大厨标注 像素大厨同样提供了自动标注、手动标注两种标注方法。自动标注需要上传设计稿，手动标注需要设计师使用「尺子」来测量距离、「吸管」来吸取色号。在界面上部有单位选择，如果我们给 iOS 开发做标注，那么单位最好选择 PT，与开发环境一致。

**总结**

到这里我们就已经把 ios 设计规范全部讲完啦，希望对大家有帮助。**觉得有用的话，就动动你的小手，关注 + 点赞吧！**

![](https://pic1.zhimg.com/50/v2-dad1af3120cd6a8e5aa0571fcd7a11f5_720w.jpg?source=1def8aca)

![](https://pic1.zhimg.com/v2-7e8a2ed32f33e507ac853204d7453d2d_l.jpg?source=1def8aca)Hell.C

本人现在毕业参加工作不久，在一家小公司干 UI，刚开始真的就感觉很懵，感觉无从下手，就连尺寸也[糊里糊涂](https://www.zhihu.com/search?q=%E7%B3%8A%E9%87%8C%E7%B3%8A%E6%B6%82&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A1550747792%7D)，然后自己摸索，在网上找资料什么的，整理出来了常见的一些[尺寸](https://www.zhihu.com/search?q=%E5%B0%BA%E5%AF%B8&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A1550747792%7D)规范，希望可以帮助到你。

![](https://pic1.zhimg.com/v2-41f2f6df15882d949739399b0fab72e1_r.jpg?source=1def8aca)

![](https://picx.zhimg.com/v2-50deaae0a61fba0f49d91f36b7e6b795_l.jpg?source=1def8aca)Pixso 协同设计​

UI 设计选择哪种尺寸，主要还是取决于最终呈现 App 的手机屏幕，[苹果手机](https://www.zhihu.com/search?q=%E8%8B%B9%E6%9E%9C%E6%89%8B%E6%9C%BA&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2366604290%7D)的产品线比较固定，在确定设计尺寸时比较方便，相比之下，[安卓手机](https://www.zhihu.com/search?q=%E5%AE%89%E5%8D%93%E6%89%8B%E6%9C%BA&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2366604290%7D)的屏幕尺寸就非常多了，在设计时要结合市场上现有分辨率占比数据，优先采用高占比[分辨率](https://www.zhihu.com/search?q=%E5%88%86%E8%BE%A8%E7%8E%87&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2366604290%7D)相对应的设计尺寸。

  
在多数设计工具中，例如**在线设计工具 Pixso**，提供了常见的手机型号及对应的设计尺寸，这些现成的预设，可以完美地解决题主遇到的问题，在刚创建文件时，不需要去网上搜索不同手机的设计尺寸。

![](https://pic1.zhimg.com/v2-b5e5891bfd6e040b543dd85ef14bb764_r.jpg?source=1def8aca)

  
提供设计尺寸的预设，只是在线设计工具 Pixso 的「基操」，作为一款专业的在线 UI 设计工具，Pixso 还有非常多可以**提高 UI 设计效率**的功能：

  
**丰富的设计素材和模板**

  
Pixso 资源社区汇集了多位 UI [设计师](https://www.zhihu.com/search?q=%E8%AE%BE%E8%AE%A1%E5%B8%88&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2366604290%7D)上传的作品和模板，既可以给我们提供设计灵感，也可以将模板中的一些组件复用到自己的作品中，快速提高出图效率。

  
Pixso 资源社区每月都会更新一大批新的**设计素材**和**模板**，做设计的过程如果没有好的思路，记得常来这里看看哦 ↓↓

![](https://picx.zhimg.com/v2-c787f705e8584733b12f96366ba15fa1_r.jpg?source=1def8aca)

###   
**多人实时协作，降低沟通成本**

  
UI 设计处于产品设计和代码实现的中间环节，设计师需要与上下游的[产品经理](https://www.zhihu.com/search?q=%E4%BA%A7%E5%93%81%E7%BB%8F%E7%90%86&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2366604290%7D)、前端开发进行多次的沟通，传统的产品、设计、研发之间的协作需要涉及多个软件，大大增加了跨部门的沟通成本，使用 Pixso 则可以很好地解决这个问题。

  
Pixso 支持多人在线实时协作，我们可以将设计相关的人员一并邀请到设计项目中，沟通当前版本存在的问题或未满足的需求，相关人员可以及时做出响应，有利于项目的持续推进。

![](https://pica.zhimg.com/50/v2-043338248c27ba4213e687c23cc50a7c_720w.gif?source=1def8aca)

###   
**版面自适应变化，兼容多种设备**

  
在 UI 设计过程中，我们可能需要调整画布的大小，来模拟版面在不同屏幕尺寸上的效果，Pixso 内置的「**约束**」功能，可以很好地帮我们达到这个目的。

  
给画板上的元素分别设置好约束后，拖动改变画板的宽度，版面上的元素会自适应发生变化，表现为自适应变更位置和长度，实时[预览版](https://www.zhihu.com/search?q=%E9%A2%84%E8%A7%88%E7%89%88&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2366604290%7D)面在不同屏幕尺寸下的效果，让发布后的界面可以兼容更多的设备。

![](https://picx.zhimg.com/50/v2-214f6619e5607dbce354b2d918ab31ce_720w.gif?source=1def8aca)

###   
**支持添加页面交互**

Pixso 不是一款单纯的 UI 设计工具，只能用来设计界面，它集成了产品、设计、研发沟通协作可能会用到的各种功能，譬如[原型设计](https://www.zhihu.com/search?q=%E5%8E%9F%E5%9E%8B%E8%AE%BE%E8%AE%A1&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2366604290%7D)、页面交互、自动标注等。

  
以页面交互为例，在 Pixso 中做好设计稿之后，可以给页面**添加交互**和**原型播放**，模拟产品最终想要达到的形态，之后进入[开发阶段](https://www.zhihu.com/search?q=%E5%BC%80%E5%8F%91%E9%98%B6%E6%AE%B5&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2366604290%7D)，负责代码实现的工程师也能给出更符合预期的产品。

![](https://pic1.zhimg.com/v2-48f79abd3b9d07b14a7a43f80748094c_r.jpg?source=1def8aca)

  
以上就是 Pixso 中 **5 个可以提升 UI 设计效率的实用功能**，最后再来总结一下：

*   提供设计尺寸预设
*   丰富的设计素材和模板
*   多人实时协作，降低沟通成本
*   版面自适应变化，兼容多种设备
*   支持添加页面交互

不知道你最喜欢其中哪一个功能呢？欢迎留言告诉皮仔哇 (●ˇ∀ˇ●)

  
目前 Pixso 对所有用户免费开放，欢迎各位前来体验。体验 Pixso 的过程中，如果有遇到问题，或者有不清楚的地方，也欢迎来评论区留言哦~

以上，希望对各位有帮助。

我是皮仔

@Pixso 协同设计 ，那我们下次再见咯～

![](https://picx.zhimg.com/v2-2504d708ebc6e5e7c60be6114d64db77_l.jpg?source=1def8aca)石头

![](https://picx.zhimg.com/v2-bb50373cefca87918db4e1d146d38027_r.jpg?source=1def8aca)

来源设计导航

![](https://picx.zhimg.com/v2-6b01ae95e4129ce720d1076058de7efc_l.jpg?source=1def8aca)小标同学

375*667 一倍设计尺寸，以做小的尺寸往大了适配比较容易，不用考虑安卓的尺寸，一般按苹果设备尺寸做设计稿拿去适配。
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.zhihu.com](https://www.zhihu.com/question/458982303) ![](https://pic1.zhimg.com/v2-50deaae0a61fba0f49d91f36b7e6b795_l.jpg?source=1def8aca)Pixso 协同设计​

题主说到的「[苹果 750X1334](https://www.zhihu.com/search?q=%E8%8B%B9%E6%9E%9C750X1334&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2416456905%7D) 和 1125*2436」，分别是 iPhone 8 (8, 7, 6S, 6) 和 iPhone X (X,XS) 的屏幕分辨率，单位为 [px](https://www.zhihu.com/search?q=px&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2416456905%7D)。

在 UI 设计中，app 或者说画板尺寸更常采用的单位是 **dp**，dp 和 px 之间有如下的转换关系：

px = dp * (dpi/160)

> 注：这里的 [dpi](https://www.zhihu.com/search?q=dpi&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2416456905%7D) 是 dots per inch 的缩写，即每英寸[像素点](https://www.zhihu.com/search?q=%E5%83%8F%E7%B4%A0%E7%82%B9&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2416456905%7D)的数量，计算方法为 屏幕分辨率 / 屏幕的物理尺寸。

相比 px，dp 的优点在于，它会根据手机屏幕分辨率的不同，**自适应更改**元素在页面上的像素大小。

举个简单的例子，假设界面上有一个 100dp 的[圆角矩形](https://www.zhihu.com/search?q=%E5%9C%86%E8%A7%92%E7%9F%A9%E5%BD%A2&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2416456905%7D)，它在 320dpi 的手机上的像素大小为 100*320/160 = 200px，在 160dpi 的手机上的像素大小为 100*160/160 = 100px。

基于自适应的特点，**在实际的 UI 设计工作中，更常采用以 [dp](https://www.zhihu.com/search?q=dp&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2416456905%7D) 为单位的画板尺寸**，如下图，使用 dp 为[计量单位](https://www.zhihu.com/search?q=%E8%AE%A1%E9%87%8F%E5%8D%95%E4%BD%8D&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2416456905%7D)的 iPhone 8 和 iPhone X 画板尺寸分别为：

> iPhone 8 (8, 7, 6S, 6) ：414 x 736  
> iPhone X (X,XS) ：375 x 812

![](https://picx.zhimg.com/v2-4f9f22757ff784743329991cd1913427_r.jpg?source=1def8aca)

当然，现在市面上的手机型号那么多，一个个去记 (手机的 dp 尺寸) 显然不太现实。

考虑到这一点，不少 UI 设计软件，例如 **Pixso**，提供了**以 dp 为单位的尺寸预设**。

在 Pixso 中创建的文件，点击顶部的「**画板**」工具，在右侧的设计[选项卡](https://www.zhihu.com/search?q=%E9%80%89%E9%A1%B9%E5%8D%A1&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2416456905%7D)，可以看到 Pixso 内置的各种画板预设，包含各种型号的 iPhone 和 Android 手机预设。

![](https://picx.zhimg.com/v2-06fe7113b6640a4568a38a640bd1e6e7_r.jpg?source=1def8aca)

作为一款集合了**设计、交付、资源**和**管理**众多功能的 UI 设计软件，提供设计尺寸预设，只是 Pixso 的「基操」，Pixso 还内置了不少可以提高 UI 设计效率的功能：

在线使用，无需安装
---------

Pixso 是一款基于浏览器的**在线设计工具**，支持 Windows、macOS 和 Linux 系统。使用之前无需安装软件，打开 Pixso 官网就能使用，既省去了安装时间，也节约了电脑磁盘空间。

独特的矢量网格，绘制更高效
-------------

Pixso 在国内率先突破了矢量网格等新型图形算法，提供了高效绘制矢量形状 (矢量网格) 的操作：

如下图，在[六边形](https://www.zhihu.com/search?q=%E5%85%AD%E8%BE%B9%E5%BD%A2&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2416456905%7D)的基础上，可以从其中任意一个[顶点](https://www.zhihu.com/search?q=%E9%A1%B6%E7%82%B9&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2416456905%7D)出发绘制线段，按同样的方式绘制另外两条线段，连接起来就可以快速得到一个[立方体](https://www.zhihu.com/search?q=%E7%AB%8B%E6%96%B9%E4%BD%93&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2416456905%7D)，相比其他软件，大大简化了绘制的步骤。

![](https://pic1.zhimg.com/50/v2-90a0fb3bbb2bf3373c162b6ccf2ebe51_720w.gif?source=1def8aca)

内置免费可商用[字体](https://www.zhihu.com/search?q=%E5%AD%97%E4%BD%93&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2416456905%7D)
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Pixso 内置了多个**免费可商用的中文字体**，例如 Google 的「**思源黑体**」，[站酷网](https://www.zhihu.com/search?q=%E7%AB%99%E9%85%B7%E7%BD%91&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2416456905%7D)的「**站酷快乐体**」，**江西拙楷体**等，无需手动安装即可使用，应用到商业设计项目中无侵权风险。

![](https://picx.zhimg.com/v2-f5710324610fc4328f86e87c53bf9e5b_r.jpg?source=1def8aca)

如果 Pixso 内置的云端字体库不能满足你的需求，你还可以安装「**字体助手**」控件，从本地导入电脑上安装的字体，给你的设计字体提供更多样化的选择。

![](https://pic1.zhimg.com/v2-665a9c03f44294d4b9fdd4614d8a4bf1_r.jpg?source=1def8aca)

团队组件库，便捷共享团队资源
--------------

传统的 UI 设计软件大多都是基于本地的[客户端](https://www.zhihu.com/search?q=%E5%AE%A2%E6%88%B7%E7%AB%AF&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2416456905%7D)，没有云端同步功能，团队内部共同的[设计文件](https://www.zhihu.com/search?q=%E8%AE%BE%E8%AE%A1%E6%96%87%E4%BB%B6&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2416456905%7D)或素材，往往要通过 U 盘或网盘传输。

而如果是使用 Pixso，可以点击 Pixso 工作台左侧的「**创建团队**」，在 Pixso 中创建一个「**[团队共享空间](https://www.zhihu.com/search?q=%E5%9B%A2%E9%98%9F%E5%85%B1%E4%BA%AB%E7%A9%BA%E9%97%B4&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2416456905%7D)**」，将团队常用的文件或素材上传到共享空间，加入空间的成员就能看到共享的文件。

![](https://pic1.zhimg.com/v2-319056c1e4a842929650e460ab67895e_r.jpg?source=1def8aca)

对于设计过程中经常会用到的组件，可以使用「**团队组件库共享**」功能，将这些高频使用的组件，发布为**团队组件库**。

之后团队成员在不同页面或项目中，就可以调用来自团队组件库的组件，既可以让页面的视觉设计保持统一，还能在组件库的样式发生更新后，保证调用的组件实例也会自动更新。

![](https://pic1.zhimg.com/v2-5a30e7b698d84b350c378b5e0d6c562c_r.jpg?source=1def8aca)

自带标注[切图](https://www.zhihu.com/search?q=%E5%88%87%E5%9B%BE&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2416456905%7D)，变更后实时更新
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Pixso 自带**标注切图**功能，可以在设计稿中直接查看标注信息，无需将文件导出到本地，再上传到 Zeplin 等第三方工具获取标注信息。

打开右侧栏的「**代码**」面板，选中任意元素，就能看到相应的属性和 **CSS 代码**，另外按住 **Alt/Option** 键，也可以快速获取元素距离容器的相对位置。

![](https://picx.zhimg.com/v2-f6eb0ebe50ca6097edb5206f5574ba7c_r.jpg?source=1def8aca)

[前端工程师](https://www.zhihu.com/search?q=%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B%E5%B8%88&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2416456905%7D)如果想要复制相应的代码，将鼠标移动到 CSS 代码上方，顶部就会弹出隐藏的「**[复制](https://www.zhihu.com/search?q=%E5%A4%8D%E5%88%B6&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2416456905%7D)**」按钮。

![](https://picx.zhimg.com/v2-a59ead3ec1050f4db9b2d8c7aa682626_r.jpg?source=1def8aca)

导出切图方面，点击右侧面板的「**设计**」选项卡，在面板底部可以看到「**导出**」的选项，**支持导出不同的倍数**。

如果同时选中多个元素，再点击右侧的「**导出已选项**」按钮，就可以**批量导出多张切图**。

![](https://picx.zhimg.com/v2-1449d514c439b5a9dbda4f2a29ad5a40_r.jpg?source=1def8aca)

此外，基于 Pixso 云端实时更新的特点，后续设计文件发生修改，**对应的标注信息也会随之自动更新**，整个过程是静默完成的，完全无需手动操作。

Pixso 个人版[永久免费](https://www.zhihu.com/search?q=%E6%B0%B8%E4%B9%85%E5%85%8D%E8%B4%B9&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2416456905%7D)
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Pixso 目前对所有个人用户免费开放，现在前往官网注册账号，可以享受到 Pixso 的所有功能：**无限文件数量、无协作者人数限制、[无限云存储空间](https://www.zhihu.com/search?q=%E6%97%A0%E9%99%90%E4%BA%91%E5%AD%98%E5%82%A8%E7%A9%BA%E9%97%B4&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2416456905%7D)**等等，快来尝试一下吧！

以上，希望有帮助。

码字不易，如果对你有帮助，请别忘了赏个**【三连】**或是**【关注】我哦，关注不迷路**！

我是皮仔

@Pixso 协同设计 ，那我们下次再见咯。

![](https://picx.zhimg.com/v2-fd2b6bc11b8cd50532856f0e6c3c8387_l.jpg?source=1def8aca)即时设计​

**如果想找最新的设计尺寸 / 设计规范，我们即时设计的资源社区就有呀！**

  
**[点击获取→大厂规范](https://link.zhihu.com/?target=https%3A//js.design/community%3Fcategory%3Dsearch%26search%3D%25E8%25AE%25BE%25E8%25AE%25A1%25E8%25A7%2584%25E8%258C%2583%26source%3Dzhqa%26plan%3Dqaivs)**

![](https://pic1.zhimg.com/v2-1b0039b793a427130aa383776ef0e52a_r.jpg?source=1def8aca)

**  
[点击获取→Ios 17+iPadOS 17 设计规范](https://link.zhihu.com/?target=https%3A//js.design/community%3Fcategory%3Ddetail%26type%3Dresource%26id%3D649bafe5c9a5eec7132416f9%26source%3Dzhqa%26plan%3Dqaivs)**

  
Apple 官方 iOS 17 设计规范组件库，包括组件、系统界面、文本样式、颜色样式、布局指南等内容。可帮助你快速创建高度逼真的 iOS 和 iPadOS [应用程序](https://www.zhihu.com/search?q=%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A3118675786%7D)。

![](https://picx.zhimg.com/v2-d1d31681fda96c2faebc5c3a972731d0_r.jpg?source=1def8aca)

  
**[点击获取→iOS 16 设计规范](https://link.zhihu.com/?target=https%3A//js.design/community%3Fcategory%3Ddetail%26type%3Dresource%26id%3D632c0171fb0bba36cf5854bb%26source%3Dzhqa%26plan%3Dqaivs)**  

![](https://picx.zhimg.com/v2-3ade9a3742850fd478b75232cf0061a9_r.jpg?source=1def8aca)

  
**  
[点击获取→IOS 15 官方设计规范](https://link.zhihu.com/?target=https%3A//js.design/community%3Fcategory%3Ddetail%26type%3Dresource%26id%3D619c9092b218eddde0ca4079%26source%3Dzhqa%26plan%3Dqaivs)**

![](https://picx.zhimg.com/v2-8680378ebceee1e1f6618e2ddbd0b376_r.jpg?source=1def8aca)

**  
[点击获取→UI 设计常用尺寸规范大全](https://link.zhihu.com/?target=https%3A//js.design/community%3Fcategory%3Ddetail%26type%3Dresource%26id%3D62a2ad1db36899ea3f824199%26source%3Dzhqa%26plan%3Dqaivs)**

这套由「即时设计」团队出品的 UI 设计规范，包含 APP 设计组件规范库，网页设计规范，B 端设计规范，小程序设计规范，车载设计规范，Apple Watch 设计规范，可以供[设计师](https://www.zhihu.com/search?q=%E8%AE%BE%E8%AE%A1%E5%B8%88&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A3118675786%7D)们来参考复用  

![](https://picx.zhimg.com/v2-e2d3c0283f51aa84100fdac71b0f53d4_r.jpg?source=1def8aca)![](https://picx.zhimg.com/v2-37b0daeca7444fa253bac549f0d37dd1_r.jpg?source=1def8aca)

  
**[点击获取→Ant Design 设计规范](https://link.zhihu.com/?target=https%3A//js.design/community%3Fcategory%3Ddetail%26type%3Dresource%26id%3D60d3301e333c5ce7ab0c0adf%26source%3Dzhqa%26plan%3Dqaivs)**

  
Ant Design Mobile 一个基于 Ant Design 设计体系的 Preact / React / React Native 的 UI 组件库，主要用于研发移动端产品。该项资源中提供了 40+ 基础组件、覆盖各类场景，组件特性丰富、满足各种[功能需求](https://www.zhihu.com/search?q=%E5%8A%9F%E8%83%BD%E9%9C%80%E6%B1%82&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A3118675786%7D)。

![](https://pic1.zhimg.com/v2-cce5b220ed738e32ca23a8c5b16a0999_r.jpg?source=1def8aca)

  
**[点击获取→字节 Arco Design 企业级设计系统全套组件库](https://link.zhihu.com/?target=https%3A//js.design/community%3Fcategory%3Ddetail%26type%3Dresource%26id%3D6194e775c530ee24f130e742%26source%3Dzhqa%26plan%3Dqaivs)**  
**[点击获取→饿了么 Element 官方设计规范库](https://link.zhihu.com/?target=https%3A//js.design/community%3Fcategory%3Ddetail%26type%3Dresource%26id%3D61e62e5a019153e9f2501285%26source%3Dzhqa%26plan%3Dqaivs)**  

* * *

  
**除此之外，我们还有更多的 APP 设计、[小程序设计](https://www.zhihu.com/search?q=%E5%B0%8F%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A3118675786%7D)、[样机素材](https://www.zhihu.com/search?q=%E6%A0%B7%E6%9C%BA%E7%B4%A0%E6%9D%90&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A3118675786%7D)、组件库资源，篇幅有限就不在这儿一一分享了，感兴趣的可以去「即时设计 - 资源社区」自行搜索下载使用。**  

[即时设计 - 可实时协作的专业 UI 设计工具](https://link.zhihu.com/?target=https%3A//js.design/community%3Fcategory%3Dexplore%26source%3Dzhqa%26plan%3Dqaivs)

![](https://pic1.zhimg.com/v2-b6ade749f2d9a7e0ebc45195d07add55_l.jpg?source=1def8aca)秋天的风

一般要兼顾 iOS 和安卓两端

iOS 代表尺寸：750x1334，安卓代表尺寸：720x1280（1080x1920 是它的 1.5 倍）；

750 和 720 相差就 30px，相当于 15pt（dp），所以用 750 的宽来做设计稿就行了，至于高度，可以就按 1334 的来。x 的是 1125x2436，它缩放下来是 750x1624，鉴于长屏手机多了，也可以按 1624 的高来。

所有的尺寸规范可以去下面这个网看，虽然是 18 年出的，但也不过时。

[https://www.xueui.cn/design/142395.html](https://link.zhihu.com/?target=https%3A//www.xueui.cn/design/142395.html)

 ![](https://picx.zhimg.com/v2-228ad3dfe9aa24e81148c6d78efe5abf_l.jpg?source=1def8aca) 设计是门艺术

UI 设计的尺寸可以通过大厂设计规范和官方设计规范中获取，即时设计的资源社区提供多个大厂的设计规范和官方规范，可以从设计规范中，具体找到具体的尺寸信息。

![](https://pic1.zhimg.com/v2-151308a686efe0d8b1330c101527009e_r.jpg?source=1def8aca)

### [官方 iOS 15 设计规范](https://link.zhihu.com/?target=https%3A//js.design/community%3Fcategory%3Ddetail%26type%3Dresource%26id%3D619c9092b218eddde0ca4079%26source%3Dzhqa%26plan%3Dys2416456905)

![](https://picx.zhimg.com/v2-ce931b1ab6896d7148e640b8456782d8_r.jpg?source=1def8aca)

由「即时设计」团队整理的 Apple 官方 iOS 15 设计规范，资源中包含浅色主题下对于控件、颜色、文本等具体设计要求，帮助你在搭建 iOS 界面时把控[整体设计](https://www.zhihu.com/search?q=%E6%95%B4%E4%BD%93%E8%AE%BE%E8%AE%A1&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A3125589225%7D)语言和视觉风格。

### [IOS 16 的设计规范](https://link.zhihu.com/?target=https%3A//js.design/community%3Fcategory%3Ddetail%26type%3Dresource%26id%3D632c0171fb0bba36cf5854bb%26source%3Dzhqa%26plan%3Dys2416456905)

![](https://picx.zhimg.com/v2-2e950c0b55e141054e8a40bdbd3afa4b_r.jpg?source=1def8aca)

IOS 16 的最新设计规范

### [UI 设计常用尺寸规范大全](https://link.zhihu.com/?target=https%3A//js.design/community%3Fcategory%3Ddetail%26type%3Dresource%26id%3D62a2ad1db36899ea3f824199%26source%3Dzhqa%26plan%3Dys2416456905)

![](https://picx.zhimg.com/v2-daf4ed7028552b090199c85d8c204168_r.jpg?source=1def8aca)

包含 APP 设计组件规范库，网页设计规范，B 端设计规范，小[程序设计](https://www.zhihu.com/search?q=%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A3125589225%7D)规范，车载设计规范，Apple Watch 设计规范，供设计参考复用。

### [Arco Design Mobile](https://link.zhihu.com/?target=https%3A//js.design/community%3Fcategory%3Ddetail%26type%3Dresource%26id%3D62da60dafbe4bd471c6265e0%26source%3Dzhqa%26plan%3Dys2416456905)

![](https://picx.zhimg.com/v2-195194743e46b916435768e0f6e9d5b5_r.jpg?source=1def8aca)

Arco Design Mobile 提供了 50+ 基础组件。随着 Arco Design Mobile 的开源，我们希望在持续获取意见和建议并进行迭代的同时鼓励社区共建，使组件更易用、功能更强大、使用者更广泛。

### [钉钉移动端组件规范](https://link.zhihu.com/?target=https%3A//js.design/community%3Fcategory%3Ddetail%26type%3Dresource%26id%3D6357ad6c3459581fc086da55%26source%3Dzhqa%26plan%3Dys2416456905)

![](https://pic1.zhimg.com/v2-7a21dca32c52b11f16a4bcff3ccb33f3_r.jpg?source=1def8aca)

### [WeUI 设计规范](https://link.zhihu.com/?target=https%3A//js.design/community%3Fcategory%3Ddetail%26type%3Dresource%26id%3D61d6e29988214c7c8fce7754%26source%3Dzhqa%26plan%3Dys2416456905)

![](https://picx.zhimg.com/v2-2d735984c3661f0c17f058fb3bb83928_r.jpg?source=1def8aca)

WeUI 设计规范是一套同微信原生视觉体验一致的基础样式库，由微信官方设计团队为微信内网页和[微信小程序](https://www.zhihu.com/search?q=%E5%BE%AE%E4%BF%A1%E5%B0%8F%E7%A8%8B%E5%BA%8F&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A3125589225%7D)量身设计，令用户的使用感知更加统一。更方便[设计师](https://www.zhihu.com/search?q=%E8%AE%BE%E8%AE%A1%E5%B8%88&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A3125589225%7D)进行设计，同时也提供方便[开发者](https://www.zhihu.com/search?q=%E5%BC%80%E5%8F%91%E8%80%85&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A3125589225%7D)调用的资源。

更多的相关设计规范可以进去即时设计资源社区进行搜索查找。

[即时设计 - 可实时协作的专业 UI 设计工具](https://link.zhihu.com/?target=https%3A//js.design/community%3Fcategory%3Dsearch%26search%3D%25E8%25AE%25BE%25E8%25AE%25A1%25E8%25A7%2584%25E8%258C%2583%26source%3Dzhqa%26plan%3Dys2416456905)![](https://picx.zhimg.com/v2-abed1a8c04700ba7d72b45195223e0ff_l.jpg?source=1def8aca)知乎用户

都可以的，不能把界面当成固定不变的一个尺寸。一般用主流机型尺寸。

我平时习惯用 1 倍尺寸做图。

另外设计软件里，新建画板都会有尺寸推荐。

![](https://picx.zhimg.com/35df4aebdeebfdfdb0a59c5915b9bceb_l.jpg?source=1def8aca)xiexinmeii

现在移动端的设备尺寸比较新，大众常用的还是到苹果 x 了，方便适配一点

安卓机的尺寸更多了 可以参照新的规范：  
[https://www.sogou.com/link?url=NdaMVEDuTuUbuDBCSldoi64VTi1p9CL-KqL3utA6z9A.](https://link.zhihu.com/?target=https%3A//www.sogou.com/link%3Furl%3DNdaMVEDuTuUbuDBCSldoi64VTi1p9CL-KqL3utA6z9A)

**如果你是对设计比较感兴趣，可以看看这些干货文章，挺有帮助的:**

[天琥教育](https://link.zhihu.com/?target=https%3A//mr.baidu.com/r/hXGAgFWgoM%3Ff%3Dcp%26u%3D93791%25207f572cd9927)

![](https://pic1.zhimg.com/v2-52074832f415a21123777bbeacd20c9e_l.jpg?source=1def8aca)不如吃茶去

x 尺寸是 1125*2436
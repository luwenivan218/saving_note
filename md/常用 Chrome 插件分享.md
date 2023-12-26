> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7315260397370802191)

前言
--

由于 Chrome 扩展商店需要特殊网络，有需求的可以在下面站点里搜索下载，不需要特殊网络，很全很新。

[www.crx4chrome.com/](https://link.juejin.cn?target=https%3A%2F%2Fwww.crx4chrome.com%2F "https://www.crx4chrome.com/")

工作
--

### CSP Evaluator

[chromewebstore.google.com/detail/csp-…](https://link.juejin.cn?target=https%3A%2F%2Fchromewebstore.google.com%2Fdetail%2Fcsp-evaluator%2Ffjohamlofnakbnbfjkohkbdigoodcejf "https://chromewebstore.google.com/detail/csp-evaluator/fjohamlofnakbnbfjkohkbdigoodcejf")

可以用来检测网站的`Content Security Policy (CSP)`，可以更清楚看到每项配置，还提供每项配置的 security 等级和建议。

比如扫描百度首页，如下图，`frame-ancestors`限制了具体哪些网站可以用 iframe 内嵌自己的网站。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bf811eec12fa429192e9b7f2275ad257~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=775&h=600&s=45172&e=png&b=fefefe)

### Window Resizer

[chromewebstore.google.com/detail/wind…](https://link.juejin.cn?target=https%3A%2F%2Fchromewebstore.google.com%2Fdetail%2Fwindow-resizer%2Fkkelicaakdanhinjdeammmilcgefonfh "https://chromewebstore.google.com/detail/window-resizer/kkelicaakdanhinjdeammmilcgefonfh")

可以调整浏览器窗口的大小以模拟各种屏幕分辨率，内置了很多常用分辨率，也可以自定义分辨率。适合在大屏屏幕里测试小分辨率响应式。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/05e94d9001494848a3a6040e1a9a4b68~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=328&h=410&s=26145&e=png&b=f8f5f4)

### CSSViewer

[chromewebstore.google.com/detail/cssv…](https://link.juejin.cn?target=https%3A%2F%2Fchromewebstore.google.com%2Fdetail%2Fcssviewer%2Fggfgijbpiheegefliciemofobhmofgce "https://chromewebstore.google.com/detail/cssviewer/ggfgijbpiheegefliciemofobhmofgce")

一个简单的 CSS 属性查看器，开启后，鼠标浮动到元素上就能显示对应的基本样式，适合调试和测试网页样式。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/49285e98ca544e518354429b40f5958c~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=476&h=574&s=57223&e=png&b=fdfdfd)

### Ajax Interceptor

[chromewebstore.google.com/detail/ajax…](https://link.juejin.cn?target=https%3A%2F%2Fchromewebstore.google.com%2Fdetail%2Fajax-modifier%2Fnhpjggchkhnlbgdfcbgpdpkifemomkpg "https://chromewebstore.google.com/detail/ajax-modifier/nhpjggchkhnlbgdfcbgpdpkifemomkpg")

用来 mock api 的返回结果，适合用来调试 api，比如把任意环境的 api 数据 mock 到自己开发环境中调试，而不用改动代码。

但是此功能在新版`Chrome Overrides`功能已经满足此需求了，功能差不多。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b3c847a4b6dc4a2d9a34c54cdc5951a6~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=615&h=220&s=18320&e=png&b=fffefe)

### Sourcegraph

[chromewebstore.google.com/detail/sour…](https://link.juejin.cn?target=https%3A%2F%2Fchromewebstore.google.com%2Fdetail%2Fsourcegraph%2Fdgjhfomjieaadpoljlnidmbgkdffpack "https://chromewebstore.google.com/detail/sourcegraph/dgjhfomjieaadpoljlnidmbgkdffpack")

提供在线阅读 GitHub、Gitlab 等源码能力，比如代码智能导航、定义跳转、搜索等等，相当于一个在线 IDE，适合在不 clone 情况下在线读源码。虽然现在 GitHub 本身就支持了代码导航和搜索功能，但是也可以当做一个备选方法。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/781925a8936e44efbfd6e590ebdc1293~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=348&h=145&s=5176&e=png&b=f4f7f9)

以 react 库为例，安装扩展后就会在 GitHub 里显示此按钮，打开后跳转下面链接，然后就可以阅读了。

[sourcegraph.com/github.com/…](https://link.juejin.cn?target=https%3A%2F%2Fsourcegraph.com%2Fgithub.com%2Ffacebook%2Freact%40main "https://sourcegraph.com/github.com/facebook/react@main")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8411730ecac940d7814bdb089df1bda8~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1082&h=730&s=95546&e=png&b=fefefe)

### Site Speed by DebugBear

[chromewebstore.google.com/detail/peom…](https://link.juejin.cn?target=https%3A%2F%2Fchromewebstore.google.com%2Fdetail%2Fpeomeonecjebolgekpnddgpgdigmpblc "https://chromewebstore.google.com/detail/peomeonecjebolgekpnddgpgdigmpblc")

官网：[www.debugbear.com/](https://link.juejin.cn?target=https%3A%2F%2Fwww.debugbear.com%2F "https://www.debugbear.com/")

用来测试网页性能指标。开启时可能会占用内存，不用时建议关掉。

但是指标数据信息不多，更多建议用第三方网站来测试，比如：[www.debugbear.com/test/websit…](https://link.juejin.cn?target=https%3A%2F%2Fwww.debugbear.com%2Ftest%2Fwebsite-speed "https://www.debugbear.com/test/website-speed")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/414ac7c4192345c282a193934ad0771d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=700&h=536&s=35143&e=png&b=fdfdfd)

### Web Vitals

[chromewebstore.google.com/detail/web-…](https://link.juejin.cn?target=https%3A%2F%2Fchromewebstore.google.com%2Fdetail%2Fweb-vitals%2Fahfhijdlegdabablpippeagghigmibma "https://chromewebstore.google.com/detail/web-vitals/ahfhijdlegdabablpippeagghigmibma")

功能跟上面的类似，常用来测试页面性能指标。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6d5d5d9eb58d426ea233b2c3e7249e04~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=577&h=600&s=31438&e=png&b=fdfdfd)

### Monica - Your AI Copilot powered by ChatGPT-4

[chromewebstore.google.com/detail/moni…](https://link.juejin.cn?target=https%3A%2F%2Fchromewebstore.google.com%2Fdetail%2Fmonica-your-ai-copilot-po%2Fofpnmcalabcbjgholdjcjblkibolbppb "https://chromewebstore.google.com/detail/monica-your-ai-copilot-po/ofpnmcalabcbjgholdjcjblkibolbppb")

AI 插件，GPT-4 肯定是收费的了，高级功能基本都是收费的，但是也有免费的基本功能。

优点是不需要特殊网络，有免费 AI 聊天功能、读 pdf、翻译页面（没啥用）。

其它
--

### Reopen closed tab Button

[chromewebstore.google.com/detail/reop…](https://link.juejin.cn?target=https%3A%2F%2Fchromewebstore.google.com%2Fdetail%2Freopen-closed-tab-button%2Fjjchodckpgecejjbbdedboikbidieebe "https://chromewebstore.google.com/detail/reopen-closed-tab-button/jjchodckpgecejjbbdedboikbidieebe")

提供一键 reopen 关闭的 tab，很简易，也很常用，特别是在不小心关闭 tab 的时候，我一天得点个至少几十下。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7513ddef99d84743bd83a8cc21fa18d6~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=146&h=61&s=2577&e=png&b=fcfcfc)

### OneTab

[chromewebstore.google.com/detail/onet…](https://link.juejin.cn?target=https%3A%2F%2Fchromewebstore.google.com%2Fdetail%2Fonetab%2Fchphlpgkkbolifaimnlloiipkdnihall "https://chromewebstore.google.com/detail/onetab/chphlpgkkbolifaimnlloiipkdnihall")

可以一键将所有或部分打开的标签页转换成一个列表，并展示在扩展独立页面中，已于管理。在 tab 特别多的时候特别有用，有时候发现一些有趣的网页没时间看，就会一直放在标签页里，不想删还占内存，这时就可以将它们存放在这个扩展里，等有时间可以随时打开。扩展页标签管理页面功能也很多，支持分组、编辑、删除、导出等等，功能很全。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d3c434694a894095914b40ad94ef5899~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=134&h=56&s=2423&e=png&b=fdfdfd)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5cfbcc7d2b9246cd8a6f90f142613b15~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=673&h=573&s=72424&e=png&b=fffefe)

### Infinity New Tab (Pro)

[chromewebstore.google.com/detail/infi…](https://link.juejin.cn?target=https%3A%2F%2Fchromewebstore.google.com%2Fdetail%2Finfinity-new-tab-pro%2Fnnnkddnnlpamobajfibfdgfnbcnkgngh "https://chromewebstore.google.com/detail/infinity-new-tab-pro/nnnkddnnlpamobajfibfdgfnbcnkgngh")

自定制新标签页，类似功能的扩展有很多，我比较喜欢这款。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d486cbb395294277893d0916d2d1d0f8~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1280&h=800&s=720390&e=png&b=c9d0db)

### Grow in 掘金

[chromewebstore.google.com/detail/grow…](https://link.juejin.cn?target=https%3A%2F%2Fchromewebstore.google.com%2Fdetail%2Fgrow-in-%25E6%258E%2598%25E9%2587%2591%2Fkiejcjemfigohhmeielfbifkikkiefeg "https://chromewebstore.google.com/detail/grow-in-%E6%8E%98%E9%87%91/kiejcjemfigohhmeielfbifkikkiefeg")

支持使用插件查看掘友们的社区活跃度和成长趋势，非常易用直观，见证在掘金社区的每一步成长。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0065e6c4a38b496ba7dd9e073d4e0ddc~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=728&h=654&s=70444&e=png&b=ffffff)

### SuperCopy 超级复制

[chromewebstore.google.com/detail/supe…](https://link.juejin.cn?target=https%3A%2F%2Fchromewebstore.google.com%2Fdetail%2Fsupercopy-enable-copy%2Fonepmapfbjohnegdmfhndpefjkppbjkm "https://chromewebstore.google.com/detail/supercopy-enable-copy/onepmapfbjohnegdmfhndpefjkppbjkm")

一键破解禁止右键、破解禁止选择、破解禁止复制、破解禁止粘贴，启用复制，启用右键，启用选择，启用粘贴。建议直接配置白名单，懂得都懂。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3720f4647ea1441cb662233bf389bcdb~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=786&h=214&s=15228&e=png&b=f8f5f4)

### 知乎 + 掘金 + 简书 + CSDN 外链直接访问（不再弹窗询问）

[chromewebstore.google.com/detail/%E7%…](https://link.juejin.cn?target=https%3A%2F%2Fchromewebstore.google.com%2Fdetail%2F%25E7%259F%25A5%25E4%25B9%258E%2B%25E6%258E%2598%25E9%2587%2591%2B%25E7%25AE%2580%25E4%25B9%25A6%2Bcsdn-%25E5%25A4%2596%25E9%2593%25BE%25E7%259B%25B4%25E6%258E%25A5%25E8%25AE%25BF%25E9%2597%25AE%25EF%25BC%2588%25E4%25B8%258D%25E5%2586%258D%25E5%25BC%25B9%25E7%25AA%2597%2Fkgbclalpeifjppmoefloadfelmpohpgm "https://chromewebstore.google.com/detail/%E7%9F%A5%E4%B9%8E+%E6%8E%98%E9%87%91+%E7%AE%80%E4%B9%A6+csdn-%E5%A4%96%E9%93%BE%E7%9B%B4%E6%8E%A5%E8%AE%BF%E9%97%AE%EF%BC%88%E4%B8%8D%E5%86%8D%E5%BC%B9%E7%AA%97/kgbclalpeifjppmoefloadfelmpohpgm")

懂得都懂。不过现在不支持鼠标滚轮中键点击打开，也就是右键里的 Open link in new tab，我总习惯点滚轮中键。

### Hide Images/Videos

[chromewebstore.google.com/detail/hide…](https://link.juejin.cn?target=https%3A%2F%2Fchromewebstore.google.com%2Fdetail%2Fhide-imagesvideos%2Fidmcpoiccfmffdglijpiapgienjemppo "https://chromewebstore.google.com/detail/hide-imagesvideos/idmcpoiccfmffdglijpiapgienjemppo")

一键隐藏页面所有图片和视频，摸鱼专用。缺点就是不支持再次点击显示图片，只能刷新页面。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9a84eb2fa84645f49b24c2bbd922d3c0~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=104&h=51&s=1711&e=png&b=fefefe)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8638b3a2f5fc464cb4bc5be2616d0255~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=419&h=188&s=8245&e=png&b=fefefe)

总结
--

就这些常用的了，下篇会继续介绍我比较常用的 VSCode 插件。
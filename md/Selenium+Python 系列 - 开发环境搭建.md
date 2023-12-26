> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7155872752186949646)

一、写在前面
------

我从未想过自己会写 python 系列的自动化文章，有些同学会问，那你现在为什么又开始写了？

不止一个人找过我，问我可以写一些`Python`自动化的文章吗，答案是肯定的。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/13bbe0c33e8a4b54bc96433a470c9051~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

笔者`Java`党，整`Python`其实也是能整的，哈哈。

那么，以后我将给大家带来接口和 UI 自动化两个方面的分享，还请大家持续关注我！

二、环境搭建
------

### 1、Python 环境搭建

**使用版本：**

*   Mac 系统
*   Python 3.10.8
*   Selenium4.5.0

**python 的安装：**

从`https://www.python.org/`下载安装.

终端输入`python3`，如下图所示：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/852cab0b187c49d390ef9d9d911304c3~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

**安装 Selenium 及驱动**

selenium 类库安装

`pip3 install selenium`

驱动类库安装（告别手动下载驱动包）

`pip install webdriver-manager`

安装完成，如下图所示：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/52a883a7de354d9a8d4787e1d2ecc874~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

这里有一个警告，是`pip3`命令需要进行升级（pip 是一个用于安装及维护 Python 包的命令）

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ebdbae467b0f4129acd2e6abb5a64b4f~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

### 2、第一个脚本

环境基本搞定了，使用`pycharm`创建好工程后，运行如下代码：

```
# -*- coding: utf-8 -*-
"""
@Time ： 2022/10/18 10:21 PM
@Auth ： 软件测试君
@File ：demo.py
@IDE ：PyCharm
@Motto：ABC(Always Be Coding)
"""
from selenium import webdriver
from webdriver_manager.chrome import ChromeDriverManager

driver = webdriver.Chrome(ChromeDriverManager().install())
driver.get("https://www.baidu.com/")
driver.quit()


```

### 3、可能遇到的问题

就像我一样，把代码复制到编译器里运行报错，如下图所示： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/87dad08293254072a4320b4c1c2f3f01~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

### 4、解决办法

终端输入如下： `pip install packaging`

**注意：** 这些 pip 命令也要在 Pycharm 中输入，如下图所示：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/32d50f34e3ac42fb91700c6a19f3557b~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

成功解决问题，这里要吐槽下自己，度娘后发现，居然是缺少类库引起，真的是笨的可以，哭笑不得，哈哈哈！

### 5、运行效果

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f8fa609dd426492493a3b5efbd581e0e~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

三、写在最后
------

到此，整个`web`自动化的开发环境就搭建完毕了，不得不说，真的比`Java`开发环境简单容易多了，虽然然容易，但是我还是喜欢写`Java`！😂
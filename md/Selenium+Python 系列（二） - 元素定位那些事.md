> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7156618036131135501)

一、写在前面
------

今天一实习生小孩问我，说哥你自动化学了多久才会的，咋学的？

自学三个月吧，真的是硬磕呀，当时没人给讲😅！

其实，学什么都一样，**真的就是你想改变的决心有多强罢了。**

二、元素定位
------

这部分内容可以说是重中之重了，也是大部分写`web`自动化的同学，必会入门技能之一了。

### 1、常见八种定位元素方法

我们还是直接来看源代码吧，示例如下：

```
# Licensed to the Software Freedom Conservancy (SFC) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The SFC licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

"""
The By implementation.
"""


class By:
    """
    Set of supported locator strategies.
    """

    ID = "id"
    XPATH = "xpath"
    LINK_TEXT = "link text"
    PARTIAL_LINK_TEXT = "partial link text"
    NAME = "name"
    TAG_NAME = "tag name"
    CLASS_NAME = "class name"
    CSS_SELECTOR = "css selector"


```

### 2、根据 id 定位元素

`driver.find_element(By.ID,"kw")`

### 3、根据 xpath 定位元素

`driver.find_element(By.XPATH, '//*[@id="kw"]')`

### 4、根据 css 定位器定位元素

`driver.find_element(By.CSS_SELECTOR, '#kw')`

### 5、根据 name 属性值定位元素

`driver.find_element(By.NAME, 'wd')`

### 6、根据 class_name 类名定位元素

`driver.find_element(By.CLASS_NAME, 's_ipt')`

### 7、根据链接文本定位元素

`driver.find_element(By.LINK_TEXT, 'hao123')`

### 8、根据部分链接文本定位元素

`driver.find_element(By.PARTIAL_LINK_TEXT, 'hao')`

### 9、根据标签名定位元素

`driver.find_element(By.TAG_NAME, 'input')`

三、find_element 与 find_elements 区别
---------------------------------

*   find_elemnet: 定位到是一个对象，定位不到则报错。
*   find_elemnets: 定位到是一个含元素的列表，定位不到是一个空列表。

四、值得关注的问题
---------

### 1、举个栗子

```
# 这句运行直接报错
driver.find_element_by_id('kw').send_keys('python')
# 这句就正常
driver.find_element(By.ID,"kw").send_keys(u"久曲健 博客园")


```

### 2、为什么报错

来吧，还是直接看源代码学习，如下所示：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/88d92169d696440498580294d7efc095~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

不难看出，最新版本只能通过 find 这种写法去写，已经不支持老版本写法。

五、写在最后
------

相信大家和我一样，基本都**喜欢白嫖别人的教程**，把珍藏多年的教程翻出来学了起来！

看到这，你肯定会说，六哥，**你居然也这样吗**，那是必然的！！

细心点，你会发现，**你收藏的教程或者学习视频都过时了，对，你没看错，它就是过时了，😂！**

虽然元素定位很简单，但是细致很重要，光看不动手实践，又怎么会发现问题呢？

我是六哥，如果觉得写的还不错，请继续关注我，并帮忙转发文章到朋友圈，你的每一次转发，对我都很重要！🙏
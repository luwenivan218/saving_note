> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7158423878665633799)

一、写在前面
------

上篇文章介绍的是关于浏览器的常见操作，接下来，我们将继续分享关于元素的常见操作，建议**收藏、转发！**

二、元素的状态
-------

在操作元素之前，我们需要了解元素的常见状态。

### 1、常见元素状态判断，傻傻分不清

*   `is_displayed()`
*   `is_enabled()`
*   `is_selected()`

### 2、is_displayed()

判断元素是否显示

```
element.is_displayed()


```

**注意：**

判断`button`是否显示，和`is_displayed()`容易混淆的是`is_enabled()`。

区别在于，直接用`element.is_enabled()`方法判断`button`是否显示，返回值为`true`，因为`button`是使用`CSS`方法判断是否有效，这并不是真正的方法，需要判断其`class`中是否有值为`disabled`来判断是否真正处于`disabled`的状态.

### 3、is_enabled()

判断元素是否有效，即是否为灰化状态

```
element.is_enabled()  


```

### 4、is_selected()

一般判断表单元素，如 radio 或 checkbox 是否被选中。

```
element.is_selected() 


```

三、常见元素的操作
---------

这部分主要演示的常见点击操作，例如：文本输入、复选框、单选按钮、选择选项、鼠标点击事件等等。

### 1、元素点击操作

**演示案例：**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/df1f58831ac84e4b899c39d495b67e18~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

**点击（鼠标左键）页面按钮**：`click()`

**示例代码如下：**

```
driver.get("http://localhost:8080/click.html")
button1 = driver.find_element(By.ID, "button1")
is_displayed = button1.is_enabled()
if is_displayed:
    button1.click()


```

### 2、Submit 操作

**演示案例：**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6808b3f4f948432ca70ecf8cb440f143~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

**点击（鼠标左键）页面按钮**：`submit()`

**示例代码如下：**

```
driver.get("http://localhost:8080/submit.html")
login = driver.find_element(By.ID, "login")
is_displayed = login.is_enabled()
if is_displayed:
    login.submit()
    # login.click()


```

**小贴士：**

支持`submit`的肯定支持`click`, 但是支持`click`的，不一定支持`submit`，可能会报错如下：

### 3、输入、清空输入操作

**演示案例：**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d7be16bed6d47ec8c259e7bbf593ead~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

**输入、清空输入操作**：`clear(), send_keys()`

**示例代码如下：**

```
username = driver.find_element(By.CSS_SELECTOR, "input[type='text']")
username.clear()
username.send_keys(u"公众号：软件测试君")
# 输出:公众号：软件测试君
print('输入值：{0}'.format(username.get_attribute("value")))
time.sleep(1)


```

四、鼠标键盘事件操作
----------

### 1、模拟回车操作

模拟打开百度搜索输入博客园，回车操作，示例代码如下：

```
driver.get("https://www.baidu.com/")
driver.find_element(By.ID, "kw").send_keys("久曲健 博客园", Keys.ENTER)


```

### 2、常见鼠标操作

**演示案例：**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a6d9e3eb1c1443a4a1341dcad74fcb4b~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

常见鼠标操作很多，如左键点击、悬浮、移动、双击、右键等等，示例代码如下：

```
driver.get("http://localhost:8080/mouse.html")
# 鼠标左键点击
ActionChains(driver).click(driver.find_element(By.ID, "mouse2")).perform()
time.sleep(1)
driver.switch_to.alert.accept()
time.sleep(1)
# 鼠标悬浮并移动操作
ActionChains(driver).move_to_element(driver.find_element(By.ID, "mouse1")).pause(1).move_to_element(
    driver.find_element(By.ID, "mouse6")).perform()
time.sleep(1)
driver.switch_to.alert.accept()
# 鼠标双击操作
ActionChains(driver).double_click(driver.find_element(By.ID, "mouse3")).perform()
time.sleep(1)
driver.switch_to.alert.accept()
# 鼠标右键
ActionChains(driver).context_click(driver.find_element(By.ID, "mouse5")).perform()


```

### 3、常见的键盘操作

<table><thead><tr><th>键盘操作</th><th>对应代码</th></tr></thead><tbody><tr><td>键盘 F1 到 F12</td><td>send_keys(Keys.F1) 把 F1 改成对应的快捷键</td></tr><tr><td>复制 Ctrl+C</td><td>send_keys(Keys.CONTROL,'c')</td></tr><tr><td>粘贴 Ctrl+V</td><td>send_keys(Keys.CONTROL,'v')</td></tr><tr><td>全选 Ctrl+A</td><td>send_keys(Keys.CONTROL,'a')</td></tr><tr><td>剪切 Ctrl+X</td><td>send_keys(Keys.CONTROL,'x')</td></tr><tr><td>制表键 Tab</td><td>send_keys(Keys.TAB)</td></tr></tbody></table>

五、演示案例源码
--------

**示例代码：**

```
# -*- coding: utf-8 -*-
"""
@Time ： 2022/10/25 21:39
@Auth ： 软件测试君
@File ：element_actions.py
@IDE ：PyCharm
@Motto：ABC(Always Be Coding)

"""
import time

from selenium.webdriver import Keys, ActionChains
from selenium.webdriver.common.by import By
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager

'''
初始化操作
'''
driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()))


def init():
    # 最大化操作
    driver.maximize_window()
    driver.set_script_timeout(60)
    # 智能等待找到元素后立即继续执行，全局生效
    driver.implicitly_wait(60)
    driver.set_page_load_timeout(60)


init()

'''
元素点击操作
'''


def clickDemo():
    # 点击（鼠标左键）页面按钮：click()
    driver.get("http://localhost:8080/click.html")
    button1 = driver.find_element(By.ID, "button1")
    is_displayed = button1.is_enabled()
    if is_displayed:
        button1.click()

    # 关闭弹窗
    driver.switch_to.alert.accept()


### 元素基本操作
clickDemo()
time.sleep(1)

'''
submit操作
'''


def submitDemo():
    # 点击（鼠标左键）页面按钮：submit()
    driver.get("http://localhost:8080/submit.html")
    login = driver.find_element(By.ID, "login")
    is_displayed = login.is_enabled()
    if is_displayed:
        login.submit()
        # login.click()
    # 小贴士：支持submit的肯定支持click,但是支持click的，不一定支持submit，可能会报错如下：


submitDemo()

'''
输入、清空输入操作
'''


def clearInputDemo():
    # 输入、清空输入操作：clear() send_keys()
    username = driver.find_element(By.CSS_SELECTOR, "input[type='text']")
    username.clear()
    username.send_keys(u"公众号：软件测试君")
    # 输出:公众号：软件测试君
    print('输入值：{0}'.format(username.get_attribute("value")))
    time.sleep(1)


clearInputDemo()

'''
模拟打开百度搜索输入博客园，回车操作
'''


def mockEnterDemo():
    # 模拟打开百度搜索输入博客园，回车操作 示例代码
    driver.get("https://www.baidu.com/")
    driver.find_element(By.ID, "kw").send_keys("久曲健 博客园", Keys.ENTER)


### 键盘操作
mockEnterDemo()
def mouseDemo():
    driver.get("http://localhost:8080/mouse.html")
    # 鼠标左键点击
    ActionChains(driver).click(driver.find_element(By.ID, "mouse2")).perform()
    time.sleep(1)
    driver.switch_to.alert.accept()
    time.sleep(1)
    # 鼠标悬浮并移动操作
    ActionChains(driver).move_to_element(driver.find_element(By.ID, "mouse1")).pause(1).move_to_element(
        driver.find_element(By.ID, "mouse6")).perform()
    time.sleep(1)
    driver.switch_to.alert.accept()
    # 鼠标双击操作
    ActionChains(driver).double_click(driver.find_element(By.ID, "mouse3")).perform()
    time.sleep(1)
    driver.switch_to.alert.accept()
    # 鼠标右键
    ActionChains(driver).context_click(driver.find_element(By.ID, "mouse5")).perform()


###  常见键盘事件操作
mouseDemo()

time.sleep(3)
driver.quit()


```

六、最后
----

到此，常见元素操作演示结束，这里只是列举了一些常用的操作，关于其他操作，感兴趣的同学请左键查看源代码 ！

我是六哥，如果觉得文章对您有帮助，建议收藏、转发！

请继续关注我，我的公众号：软件测试君，并帮忙转发文章到朋友圈。

你的每一次转发，我都当做了喜欢！🙏
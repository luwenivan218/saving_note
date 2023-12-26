> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7166981439400411149)

前言
--

突然，想把所有之前未更新的常用`Api`操作、演示写出来，算是对 API 的一种完结吧。

下面按照`Api`模块来做逐一介绍。

### 一、iframe 操作

**iframe 识别：**

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/113a5afbe6924fefa16dbbcc7f04c325~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

**语法：**

`driver.switch_to.frame('方式')`

#### 1、常见处理方法三种

*   index：下标
*   name：id 或 name 属性的值
*   webelement：元素

#### 2、通过下标进入

进入第一个 iframe `driver.switch_to.frame(0)`

#### 3、通过 id 或 name 属性的值进入

通过 id 或 name 属性的值进入指定的 iframe

```
driver.switch_to.frame('iframe')
driver.switch_to.frame('iframeName')


```

#### 4、通过 iframe 元素进入 iframe

通过 iframe 元素进入指定 iframe

```
iframe=driver.find_element(By.ID,"iframe")
driver.switch_to.frame(iframe)


```

完整案例代码如下：

```
from selenium import webdriver
from selenium.webdriver.common.by import By
from webdriver_manager.chrome import ChromeDriverManager

driver = webdriver.Chrome(ChromeDriverManager().install())
driver.get("http://localhost:8080/iframeDemo.html")
# 通过下标进入frame
driver.switch_to.frame(0)
# 通过id或name属性的值进入指定的iframe
driver.switch_to.frame('iframe')
driver.switch_to.frame('iframeName')
# 通iframe元素进入iframe
iframe=driver.find_element(By.ID,"iframe")
driver.switch_to.frame(iframe)
driver.find_element(By.ID,'user').clear()
driver.find_element(By.ID,'user').send_keys("this is a frame test !")
print(driver.find_element(By.ID,'user').get_attribute('value'))


```

### 二、select 下拉框操作

#### 1、select 控件识别

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3fb3ecd206fc4f3a9ff056713e8bb206~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

常见操作有两种：一步到位，二次管控！

#### 2、一步到位

**一步到位：** 直接定位元素点击即可，示例如下：

```
# 一步到位，直接选择典韦
driver.find_element(By.CSS_SELECTOR,"[value='3']").click()
print(driver.find_element(By.CSS_SELECTOR,"[value='3']").text)


```

#### 3、二次管控

**二次管控：** 先定位 select 框，再定位 select 里的选项, 通过 Select 对象进行强转，来调用 select 控件中的 Api 来达到操作的目的。

**常见操作方法：**

*   select_by_index()：通过下标选择对应项
*   select_by_value()：通过 value 选择对应项
*   select_by_visible_text()：通过可见文本选择对应项

示例代码如下：

```
select = Select(driver.find_element(By.ID, "select"))
# 选择第一个选项
select.select_by_index(0)
# 调用first_selected_option就能获取当前下拉框选中值啦
print(select.first_selected_option.text)
sleep(2)
# 选择典韦
select.select_by_value("3")
# 调用first_selected_option就能获取当前下拉框选中值啦
print(select.first_selected_option.text)
sleep(2)
# 选择凯
select.select_by_visible_text("凯")
# 调用first_selected_option就能获取当前下拉框选中值啦
print(select.first_selected_option.text)



```

#### 4、遍历所有选项

示例代码如下：

```
# 打印所有选项的text
for option in select.options:
    print("选项为："+option.text)



```

完整代码示例：

```
from time import sleep

from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.select import Select
from webdriver_manager.chrome import ChromeDriverManager

driver = webdriver.Chrome(ChromeDriverManager().install())
driver.get("http://localhost:8080/SelectDemo.html")
# 一步到位，直接选择典韦
# driver.find_element(By.CSS_SELECTOR,"[value='3']").click()
# print(driver.find_element(By.CSS_SELECTOR,"[value='3']").text)
select = Select(driver.find_element(By.ID, "select"))
# 选择第一个选项
select.select_by_index(0)
# 调用first_selected_option就能获取当前下拉框选中值啦
print(select.first_selected_option.text)
sleep(2)
# 选择典韦
select.select_by_value("3")
# 调用first_selected_option就能获取当前下拉框选中值啦
print(select.first_selected_option.text)
sleep(2)
# 选择凯
select.select_by_visible_text("凯")
# 调用first_selected_option就能获取当前下拉框选中值啦
print(select.first_selected_option.text)

# 打印所有选项的text
for option in select.options:
    print("选项为："+option.text)
sleep(2)


```

关于 Select 模块的其他方法还有很多，其他方法，还请各位各位读者朋友自己去尝试，就不一一列举了！

### 三、交互操作弹出框的处理

#### 1、弹出框分类：

弹出框分为两种，一种基于原生 JavaScript 写出来的弹窗，另一种是自定义封装好的样式的弹出框，即原生 JavaScript 写出来的弹窗，另一种弹窗用 click() 基本就能搞定。 原生 JavaScript 写出来的弹窗又分为三种：

**`alert`**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7873f36a038344c09f586bbfbfde3fda~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

**`confirm`**

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9f7e9f9a5e3a47a0b92254b30036b0e4~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

**`prompt`**

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/938e9d644a024caeb4a776577c3a0c99~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

#### 2、弹窗处理常用方法：

`alert/confirm/prompt`弹出框操作主要方法有：

*   `driver.switch_to.alert：`切换到 alert 弹出框上
*   `alert.text：`获取文本值
*   `accept() ：`点击 "确认"
*   `dismiss()：`点击 "取消" 或者关闭对话框
*   `send_keys() ：`输入文本值 -- 仅限于 prompt, 在 alert 和 confirm 上没有输入框

**`alert弹窗处理`**

示例代码如下：

```
# alert弹窗处理
driver.find_element(By.ID,"alert").click()
alert=driver.switch_to.alert
print(alert.text)
# 确定
alert.accept()
sleep(2)


```

**`confirm弹窗处理`**

示例代码如下：

```
# dialog对话框处理
driver.find_element(By.ID,"dialog").click()
alert=driver.switch_to.alert
print(alert.text)
# 取消操作
alert.dismiss()
sleep(2)


```

**`prompt弹窗处理`**

```
# 弹窗输入框
driver.find_element(By.ID,"welcome").click()
alert=driver.switch_to.alert
print(alert.text)
alert.send_keys("input 框")
alert.accept()
sleep(2)
print(alert.text)


```

### 四、执行 Js 操作

在做 web 自动化时，有些情况 selenium 的 api 无法完成，需要通过第三方手段比如 js 来完成实现，比如去改变某些元素对象的属性或者进行一些特殊的操作，本文将来讲解怎样来调用 JavaScript 完成特殊操作。

#### 1、用法

`driver.execute_script(js语句)`

#### 2、模拟场景

**场景 1**

打开百度首页，并弹窗提示 hellow,world!，关闭弹窗，控制台输出弹窗文本 hellow,world! 示例代码如下：

```
# 执行js语句
driver.execute_script("alert('hellow,world!')")
alert=driver.switch_to.alert
print(alert.text)
# 确定
alert.accept()


```

**场景 2** 示例代码如下：

```
# 将百度按钮改成MyLove
element = driver.find_element(By.ID, "su");
driver.execute_script("document.getElementById('su').setAttribute('value', 'MyLove');", element);


```

效果如下： ![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e4195140032a4ce2b26c3e2fa3d8d9ec~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

#### 3、模拟滚动条操作

在写脚本时，总会遇到一种情况，就是当滚动拉倒最下面了，表单或者下拉框、按钮这些元素未在当前页面展示，而 webdriver 提供的方法都是操作当前页面可见的元素，这时我们使用 JavaScript 操作浏览器的滚动条，滚动后使页面元素可见，就可完成后面的元素操作了。

**核心思路：**

就是使用 js 去控制浏览器滚动条的位置，在使用 selenium 调用 JavaScript 操作 js 完成。

下面举例几种常用滚动条的 js 代码示例如下：

```
//拖动滚动条至底部
document.documentElement.scrollTop=10000
window.scrollTo(0,document.body.scrollHeight)

//拖动滚动条至顶部
document.documentElement.scrollTop=0
arguments[0].scrollIntoView(false);

//左右方向的滚动条可以使用window.scrollTo(左边距，上边距)方法
window.scrollTo(200,1000)


```

**实际案例**

以博客园我的文章列表页为例，来演示滚动条操作，具体代码如下：

```
from time import sleep

from selenium import webdriver
from selenium.webdriver.common.by import By
from webdriver_manager.chrome import ChromeDriverManager

driver = webdriver.Chrome(ChromeDriverManager().install())
driver.get("https://www.cnblogs.com/longronglang/")
driver.maximize_window()

# 获取第一篇文章列表元素
element = driver.find_element(By.CSS_SELECTOR,".forFlow [role='article']:nth-of-type(1) .vertical-middle")
sleep(2)
# 将页面滚动条拖到底部
driver.execute_script("window.scrollTo(0,document.body.scrollHeight)")
# 将滚动条滚动至第三篇文章列表位置
driver.execute_script("arguments[0].scrollIntoView(true)", element)
sleep(2)
# 将滚动条滚动到顶部
driver.execute_script("arguments[0].scrollIntoView(false)", element)
sleep(2)
# 将滚动条滚动到指定位置
driver.execute_script("window.scrollTo(200,1000)")


```

### 五、Cookie 操作

下面我们就使用`cookie`操作，绕过登录验证码

还是以博客园为例，下面本文来介绍下如何绕过下图验证码，进入博客园

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/78a16064f95348f2a40cfd9de6a34cad~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

#### 1、工具准备

*   Fiddler.exe
*   IDEA/Eclipse
*   selenium 的 cookie 操作

**如何操作？**

****看完之后，记得收藏 + 转发。****

#### 2、使用 Fiddler 抓包

一般登陆网站成功后，会生成一个已登录状态的 cookie，那么只需要直接把这个值拿到，用 selenium 进行 addCookie 操作即可。

可以先手动登录一次，然后抓取这个 cookie，这里我们就需要用抓包工具 fiddler 了

先打开博客园登录界面，手动输入账号和密码（不要点登录按钮）

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/79ac5a7a197441908ccdd1f3e6da647d~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

打开 fiddler 抓包工具，此时再点博客园登录按钮

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8884d5df48854806b6a40b3f444a431c~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

登录成功后，再查看 cookie 变化，发现多了两组参数，多的这两组参数就是我们想要的，copy 出来，一会有用

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f78fd416579a46d3b6ffc2818c2148d0~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

#### 3、cookie 操作语法

`driver.add_cookie（）`

add_cookie(cookie_dict) 方法里面参数是 cookie_dict，说明里面参数是字典类型。

源码官方文档介绍：

```
add_cookie(self, cookie_dict)  

Adds a cookie to your current session.  
  
Args:  

- cookie_dict: A dictionary object, with required keys - "name" and "value"; 
optional keys - "path", "domain", "secure", "expiry"  

Usage:  

driver.add_cookie({'name' : 'foo', 'value' : 'bar'})  
driver.add_cookie({'name' : 'foo', 'value' : 'bar', 'path' : '/'})  
driver.add_cookie({'name' : 'foo', 'value' : 'bar', 'path' : '/', 'secure':True})


```

从官方的文档里面可以看出，添加 cookie 时候传入字典类型就可以了，等号左边的是 name，等号左边的是 value。

把前面抓到的两组数据（参数不仅仅只有 name 和 value），写成字典类型：

```
{'name':'.CNBlogsCookie','value'：'2C3AE01E461B2D2F1572D02CB936D77A053089AA2xxxx...'}
{'name':'.Cnblogs.AspNetCore.Cookies','value':'CfDJ8Mmb5OBERd5FqtiQlKZZIG4HKz_Zxxx...'}


```

#### 4、完整示例代码

```
# coding:utf-8
from selenium import webdriver
import time

driver = webdriver.Chrome()
driver.maximize_window()
driver.get("https://www.cnblogs.com/longronglang/")

# 添加cookie
c1 = {u'domain': u'.cnblogs.com',
      u'name': u'.CNBlogsCookie',
      u'value': u'xxxx',
      u'expiry': 15412950521,
      u'path': u'/',
      u'httpOnly': True,
      u'secure': False}

c2 = {u'domain': u'.cnblogs.com',
      u'name': u'.Cnblogs.AspNetCore.Cookies',
      u'value': u'xxxx',
      u'expiry': 15412950521,
      u'path': u'/',
      u'httpOnly': True,
      u'secure': False}
# 添加2个值
driver.add_cookie(c1)  
driver.add_cookie(c2)
time.sleep(3)

# 刷新下页面就见证奇迹了
driver.refresh()
# 再来个登录后操作
driver.find_element_by_link_text(u"博客园").click()
driver.find_element_by_link_text("Refain").click()


```

**效果图**

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9a84c4e0648640ebb2d3ae7e56552dfb~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

****有几点需要注意：****

*   登录时候要勾选下次自动登录按钮。
*   addCookie（）只添加 name 和 value，对于博客园的登录是不成功。
*   本方法并不适合所有的网站，一般像博客园这种记住登录状态的才会适合。
*   学习过程中有遇到疑问的，可以加我微信 1399811201 交流

最后
--

本来想着一口气都写完的，结果时间不允许，今天先更新这么多了，996 的工作情况，请原谅我的” 懒惰 “，您的转发、转发、点赞，我都当作了喜欢🙏！
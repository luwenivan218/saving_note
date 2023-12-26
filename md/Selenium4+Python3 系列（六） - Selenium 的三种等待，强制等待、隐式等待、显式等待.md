> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7160677465546260510)

### 为什么要设置元素等待

直白点说，**怕报错**，哈哈哈！

肯定有人会说，这也有点太直白了吧。

用一句通俗易懂的话就是：**等待元素已被加载完全之后，再去定位该元素，就不会出现定位失败的报错了。**

### 如何避免元素未加载出来而导致定位失败 ？

三种方式，**强制等待、隐式等待、显式等待！**

#### 1、强制等待

就是`sleep()` ，也叫硬等待； 缺点就是：如果等待时间过长，即使元素已被加载出来了，但还是要继续等，这样会导致整个脚本的执行上会浪费很多时间。

示例代码如下：

```
# 强制等待案例
driver.get("http://localhost:8080/wait.html")
driver.find_element(By.ID, "wait").click()
time.sleep(3)
text = driver.find_element(By.ID, "green_box").text
print('text is : '+text)


```

#### 2、隐式等待

`WebDriver` 提供了三种隐性等待方法：

*   `implicitly_wait`

识别对象时的超时时间。过了这个时间如果对象还没找到的话就会抛出`NoSuchElementException` 异常。

*   `set_script_timeout`

异步脚本的超时时间。`WebDriver` 可以异步执行脚本，这个是设置异步执行脚本，脚本返回结果的超时时间。

*   `set_page_load_timeout`

页面加载时的超时时间。因为 `WebDriver` 会等页面加载完毕再进行后面的操作，所以如果页面超过设置时间依然没有加载完成，那么 `WebDriver` 就会抛出异常。

以上三种都是在整个 webDriver 生命周期有效，即全局设置，相当于全局变量！

示例代码如下：

```
def init():
    # 最大化操作
    driver.maximize_window()
    driver.set_script_timeout(60)
    # 智能等待60秒，找到元素后立即继续执行，全局生效
    driver.implicitly_wait(60)
    driver.set_page_load_timeout(60)


init()
# 强制等待案例
driver.get("http://localhost:8080/wait.html")
driver.find_element(By.ID, "wait").click()
# 硬等待
# time.sleep(3)
text = driver.find_element(By.ID, "green_box").text
print('text is : '+text)


```

#### 3、显式等待

就是明确的要等到指定元素（相当于局部变量）的出现或者是某个元素的可点击等条件等到为止，才会继续执行后续操作，等不到，就一直等，如果在规定的时间之内都没找到，就会抛出异常！

显示等待与隐式等待相对，显示等待必须在每个需要等待的元素前面进行声明。

示例代码如下：

```
# -*- coding: utf-8 -*-
"""
@Time ： 2022/10/31 8:12 PM
@Auth ： 软件测试君
@File ：test_wait.py
@IDE ：PyCharm
@Motto：ABC(Always Be Coding)
"""

from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.wait import WebDriverWait
from webdriver_manager.chrome import ChromeDriverManager

'''
初始化操作
'''
driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()))


def init():
    # 最大化操作
    driver.maximize_window()


init()
driver.get("http://localhost:8080/wait.html")
driver.find_element(By.ID, "wait").click()
# 显示等待案例
# 设置元素等待实例，最多等5秒，每0.5秒查看条件是否成立
element = WebDriverWait(driver, 5, 0.5).until(EC.presence_of_element_located((By.ID, "green_box")))
print('text is : ' + element.text)
driver.quit()


```

##### 3.1、显示等待需要用到两个类

`WebDriverWait`和`expected_conditions`两个类。

**WebDriverWait(driver,timeout,poll_frequency=0.5,ignored_exceptions=None) 参数说明：**

`driver：`浏览器驱动

`timeout：`最长超时时间，默认以秒为单位

`poll_frequency：`检测的间隔步长，默认为 0.5s

`ignored_exceptions：`超时后的抛出的异常信息，默认抛出 NoSuchElementExeception 异常。

##### 3.2、until() 和 until_not() 的方法

**until**

*   `WebDriverWait(driver,10).until(method，message="")`
*   调用该方法提供的驱动程序作为参数，直到返回值为 True

`method:` 在等待期间，每隔一段时间（__init__中的 poll_frequency）调用这个传入的方法，直到返回值不是 False

`message:` 如果超时，抛出 TimeoutException，将 message 传入异常

**until_not**

*   WebDriverWait(driver,10).until_not(method，message="")
*   调用该方法提供的驱动程序作为参数，直到返回值为 False

与 until 相反，until 是当某元素出现或什么条件成立则继续执行，until_not 是当某元素消失或什么条件不成立则继续执行，参数也相同。

##### 3.3、expected_conditions 类

各种类，达到某种条件，返回 True 和 False，详细参考下表。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f27010a711a14919b6b90b2000de97fc~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

##### 3.4、显示等待, 自定义等待条件

示例代码如下：

```
# 设置等待
wait = WebDriverWait(driver, 10, 0.5)
# 使用匿名函数
element = wait.until(lambda diver: driver.find_element(By.ID, 'green_box'))
print(element.text)
driver.quit()


```

### 写在最后

其实隐式等待和显示等待在本质上是一致的，只是显示等待多了一个指定元素条件超时时间，在使用场景上，可以使用隐式等待来做一个全局的控制，例如设置全局隐式等待 6 秒；

如果某个控件比较特殊，需要更长的时间加载，比如十几秒或者更长，就可以使用显示等待对其进行单独处理；

**参考文章：**

[blog.csdn.net/qq_36821826…](https://link.juejin.cn?target=https%3A%2F%2Fblog.csdn.net%2Fqq_36821826%2Farticle%2Fdetails%2F115668538 "https://blog.csdn.net/qq_36821826/article/details/115668538")
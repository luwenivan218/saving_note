> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7316300942609530890)

哈喽，大家好，我是老表，学 Python 编程，找老表就对了。

我正在参与掘金 2023 年度人气创作者打榜中，**[点击给我投票，免费送你 GPT3.5 私聊服务](https://activity.juejin.cn/rank/2023/writer/993614243314551?utm_campaign=annual_2023&utm_medium=self_web_share&utm_source=%E8%80%81%E8%A1%A8 "https://activity.juejin.cn/rank/2023/writer/993614243314551?utm_campaign=annual_2023&utm_medium=self_web_share&utm_source=%E8%80%81%E8%A1%A8")**

好文章，先点赞～谢谢你。

今天和大家分享一个超赞的自动化库 --PyAutoGUI，PyAutoGUI 可以让 Python 脚本控制鼠标和键盘，通过代码操作鼠标、键盘自动与其他应用程序交互，该 Python 包支持在 Windows、macOS 和 Linux 上运行。

开始动手动脑
------

本地电脑打开 Powershell/Terminal ，切换到 Python 环境，输入以下指令即可安装 PyAutoGUI：

```
pip install pyautogui


```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/65484e01bb014ad0afd8f3e723fa5e7a~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1134&h=222&s=60233&e=png&b=fefefe)

安装好后就可以直接开始使用了。

首先导入包：

```
import pyautogui


```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2fd6c67b299349209b91402e621322b9~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=489&h=49&s=3889&e=png&b=ffffff)

### 常用基础操作

**1、定位：** 这是最关键的，找到要点击的位置（像素坐标）。

**规定坐标原点是屏幕左上角。** 我们可以使用以下指令查看屏幕大小：

```
# 查看屏幕尺寸，目前只支持在主屏上操作
pyautogui.size()


```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9084347f302747eba0349bbc245cf1f5~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=402&h=87&s=13457&e=png&b=f8f8f8)

查看当前鼠标所在位置：

```
# 查看当前鼠标位置，坐标原点是屏幕左上角
pyautogui.position() 


```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/760fdf3610f3495787c0980860b17c6d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=373&h=88&s=13741&e=png&b=f9f9f9)

**2、移动鼠标：** 找到要点击鼠标位置后，即可开始移动鼠标。 移动鼠标使用`moveTo`函数，可以通过 duration 参数设置移动速度。

```
# 在 num_seconds 秒内将鼠标移动到 (x,y)
x,y = (409, 300)
num_seconds = 1
pyautogui.moveTo(x, y, duration=num_seconds)  


```

还可以使用`moveRel`函数相对路径移动，将当前位置作为坐标轴原点。

```
# 在 num_seconds 秒内将鼠标移动到相对当前的位置 (x,y)
x,y = (409, 300)
num_seconds = 1
pyautogui.moveRel(x, y, duration=num_seconds)  


```

**3、点击：** 鼠标移动到对应位置后，即可点击了，这是最终操作，点点点～

点击流程是先移动鼠标到指定位置，然后进行点击，使用`click`函数，参数说明：

*   x,y 鼠标点击位置
*   clicks 点击次数
*   interval 点击频率，如果是 1 就是每秒点击 1 次，直到完成 clicks 次点击
*   button 支持 left right middle，分别对应鼠标左键、右键、中键

```
x,y = (620, 538)
num_of_clicks = 2
secs_between_clicks = 1
pyautogui.click(x=x, y=y, clicks=num_of_clicks, interval=secs_between_clicks, button='left')


```

### 应用案例 1：关注公众号太多，程序帮你批量取关

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/371ea3fe13564b92afab7708db1c870d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=760&h=416&s=44337&e=png&b=f5f5f5)

766 个，真难顶！每天公众号里都有如潮水般的消息涌出，久而久之就懒得去看了，但其中也有很多关键信息，为提高阅读效率，先批量取关，然后根据需要再关注吧～

#### 分析自动化步骤

先看手动取关步骤：

*   点击联系人 - 公众号（直接手动）
*   点开`要取关公众号`- 点击`查看历史消息` ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/658744709cff45749ce4bbc65f79d974~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1788&h=648&s=102274&e=png&b=fbfbfb)
*   点击`已关注`- 点击`不再关注`，即可 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7c282d982f3e4ea1967092e8a12e8aaf~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=327&h=181&s=33912&e=png&b=f7f7f7)

从上面步骤不难分析出，可以自动化操作的是后两步，总共涉及 4 个需要点击的地方，正常情况这四个地方不会有变化，所以我们只需找到这四个地方坐标，然后开启自动化，按顺序点击即可。

#### 找要点击位置坐标

比较简单的方法是直接将鼠标放到对应位置，然后使用`pyautogui.position()`获取鼠标所在位置。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1258b520e64f49419c94be5fb8beb65e~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=348&h=87&s=12845&e=png&b=f9f9f9) 这样操作的前提是你得有两个屏幕，不然你没地方运行代码查坐标。当然还有一种方法就是使用截屏，去看对应位置的像素坐标。

*   `要取关公众号`坐标：Point(x=388, y=386)
*   `查看历史消息`坐标：Point(x=861, y=342)
*   `已关注`坐标：Point(x=704, y=373)
*   `不再关注`坐标：Point(x=891, y=539)

#### 测试自动化

有了前面学习分析，写出自动化代码不难，一个 for 循环，然后里面点击 点击 点击 点击即可，代码如下。

```
def click_btn(x,y):
    num_of_clicks = 1
    secs_between_clicks = 1
    pyautogui.click(x=x, y=y, clicks=num_of_clicks, interval=secs_between_clicks, button='left')

# 自动化间隔
pyautogui.PAUSE = 1
for i in range(3):
    # 点开`要取关公众号`-点击`查看历史消息`
    click_btn(388,386)
    print("点击了 要取关公众号")
    click_btn(861,342)
    print("点击了 查看历史消息")
    # 点击`已关注`-点击`不再关注`，即可
    click_btn(704,373)
    print("点击了 已关注")
    click_btn(891,539)
    print("点击了 不再关注")
    break


```

实际运行发现了一个问题，就是 要取关公众号 位置是不变的，但其它的 查看历史消息、已关注 是会受公众号简介内容长短而变化的，公众号简介长，按钮位置就会下移。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a60333a0084c4817a25c2c03b891464d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=909&h=424&s=77197&e=png&b=f5f5f5)

如何定位 查看历史消息、已关注 按钮位置成了现在的主要问题！

#### 主要问题：根据文字定位

查了会资料，发现 pyautogui 有一个内置函数就支持通过指定内容来查找对应内容所在位置，这里用到的就是`locateOnScreen`函数，其原理是通过图像识别去匹配需要查找内容在图片中的像素区域位置。以下是其工作原理的简要描述：

1.  **截屏**：首先，`pyautogui` 会获取当前屏幕的截图。
    
2.  **模板匹配**：然后，`pyautogui` 将你提供的参考图像（模板）在截取的屏幕图像上移动，尝试在屏幕上找到一个位置，使得参考图像与屏幕截图的某个区域的匹配度最高。
    
3.  **像素比较**：在模板匹配过程中，算法会对参考图像和屏幕截图的每个像素进行比较，计算它们之间的相似度。相似度通常是通过计算颜色差异来评估的。
    
4.  **确定位置**：如果找到了一个区域，其与参考图像的相似度超过了设定的阈值（有时你可以设置一个`confidence`参数来指定这个阈值），`pyautogui` 便会返回这个区域的坐标和大小。这个坐标是屏幕截图上参考图像左上角的位置。
    
5.  **返回结果**：如果找到了匹配的区域，`pyautogui` 返回一个包含了`left`, `top`, `width`, `height`的元组或矩形对象；如果没有找到匹配区域，它会返回`None`。
    

主要参数解析：

*   image：这是一个字符串或 Pillow 的 Image 对象，指定要在屏幕上查找的图像。如果是字符串，它应该是图像文件的路径。
*   confidence：可选参数，指定匹配的可信度阈值，介于 0 到 1 之间。一个更高的值意味着更精确的匹配，但可能导致没有找到匹配项。默认情况下，这个值是未设置的，但是如果安装了 OpenCV，可以使用这个参数。
*   region：可选参数，一个四元组（left, top, width, height），指定屏幕上一个区域来限制搜索范围。这可以提高搜索速度并减少误匹配。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a974c2d45afd49e59c1fcd001efaf51b~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=667&h=169&s=38895&e=png&b=f4f3f3)

经过多次测试发现 locateOnScreen 匹配出来的坐标 x、y 值都是原位置的 2 倍，所以得到了查找 查看历史消息、已关注 按钮位置的方法，代码如下：

```
text_location = pyautogui.locateOnScreen(image='ckls.jpg', confidence=0.7)
click_btn(text_location.left/2+15, text_location.top/2+4)
print("点击了 查看历史消息")
text_location = pyautogui.locateOnScreen(image='ygz.jpg', confidence=0.7)
click_btn(text_location.left/2+25, text_location.top/2+10)
print("点击了 已关注")


```

其中 ckls.jpg、ygz.jpg 为 查看历史消息、已关注 截图。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a9316fbf7fa54e148daee4d6dcb262b8~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=580&h=92&s=22971&e=png&b=f3f3f3)

#### 再次测试自动化

结合前面分析，将代码稍作修改即可，测试的时候发现新问题，点击`查看历史消息`后，公众号详情页面加载需要时间，如果直接执行点击`已关注`可能会出现错误，所以代码里在点击`查看历史消息`后加了`time.sleep(1.5)`给点缓冲时间，具体代码如下：（大家复现的时候里面的坐标需要改成大家屏幕对应的，位置分析方法前面已经分享过了）

```
import pyautogui
import time
def click_btn(x,y):
    num_of_clicks = 1
    secs_between_clicks = 1
    pyautogui.click(x=x, y=y, clicks=num_of_clicks, interval=secs_between_clicks, button='left')

# 自动化间隔
for i in range(671):
    try:
        print(f"正在取关第{i+1}个公众号号")
        # 点开`要取关公众号`-点击`查看历史消息`
        click_btn(509,497)
        print("点击了 要取关公众号")
        text_location = pyautogui.locateOnScreen('ckls.jpg', confidence=0.7)
        click_btn(text_location.left/2+15, text_location.top/2+4)
        # print("点击了 查看历史消息")
        # 上一步后加载页面需要时间
        time.sleep(1.5)
        text_location = pyautogui.locateOnScreen('ygz.jpg', confidence=0.7)
        click_btn(text_location.left/2+25, text_location.top/2+10)
        # print("点击了 已关注")
        click_btn(949,620)
        print("点击了 不再关注")
    except Exception as e:
        continue


```

运行结果： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d87a29afeb2d495da6a5c1ee287732aa~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=600&h=338&s=8255437&e=gif&f=192&b=d9d2ce)

如果想要更多自动化源码，或者交流 Python 相关问题，可以私信我或者评论区提问。
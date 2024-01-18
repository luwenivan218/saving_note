> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_17718631/article/details/84111428)

### **Grid 布局解决列等分问题**

实际需求：

![](https://img-blog.csdnimg.cn/20181116105532536.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE3NzE4NjMx,size_16,color_FFFFFF,t_70)

通常我们一般会选择用 [flex](https://so.csdn.net/so/search?q=flex&spm=1001.2101.3001.7020) 布局来实现:

```
{
	display: flex;
	flex-direction: row;
	justify-content : space-between;
}

```

**但是这样的方式会导致最后一行不沾满的情况下排列出现问题，像这样：**  
![](https://img-blog.csdnimg.cn/20181116105738606.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE3NzE4NjMx,size_16,color_FFFFFF,t_70)

其实我们是希望其他行等分，最后一行左对齐的效果的，如果一定要用 flex 来做，就必须写占位的 div，让最后一行沾满，让每一行的个数相等，达到等分效果。但是在屏幕自适应，内容大小不确定且不固定一行显示多少个的情况下，这个逻辑就显得相对复杂，所以最后考虑用 [grid](https://so.csdn.net/so/search?q=grid&spm=1001.2101.3001.7020) 布局来实现。

之前有分享过 grid 的一些基础用法，这次的重点也不在基础上，所以直接跳过，直接上进入主题  
[grid 布局基础链接](http://www.css88.com/archives/8506)

首先要普及一下 demo 用到的一些概念

**1、等分 (fr) 单位**

Grid 带来了一个全新的值，称为等分单位，即 fr 。它允许你将容器可用空间分成你想要的多个等分空间。

```
{
	display: grid;
    grid-template-columns: 1fr 1fr 1fr;
}

```

结果：  
![](https://img-blog.csdnimg.cn/2018111614172297.gif)

**2、repeat()**  
可以创建重复的网格轨道。这个适用于创建相等尺寸的网格项目和多个网格项目。repeat() 接受两个参数：第一个参数定义网格轨道应该重复的次数，第二个参数定义每个轨道的尺寸  
所以下面两个写法结果相等

```
{
	display: grid;
    grid-template-columns: 1fr 1fr 1fr;
}

{
	display: grid;
	grid-template-columns: repeat(3, 1fr);
}


```

效果和上个例子一样

**3、auto-fit （自适应）**  
但是我们不希望固定行的次数，希望自动填充次数，充满外面的容器，可以用 auto-fill 或者 auto-fit

```
{
    display: grid;
    grid-template-columns: repeat(auto-fit, 200px);
}

```

结果：  
![](https://img-blog.csdnimg.cn/20181116141808206.gif)

现在这个网格已经可以通过容器的宽度来改变列的数量。但是貌似还不能完全满足需求  
它只是尽可能的把 100px 宽的盒子往一列里面塞，我们永远得不到我们想要的灵活性，因为它们很少会加起来正好等于容器的宽度，所以右边经常会留白。

**4、minmax()**  
可以通过 minmax() 函数来创建网格轨道的最小或最大尺寸。  
minmax() 函数接受两个参数：第一个参数定义网格轨道的最小值，第二个参数定义网格轨道的最大值。可以接受任何长度值，也接受 auto 值。auto 值允许网格轨道基于内容的尺寸拉伸或挤压  
所以在代码上改成：

```
 {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
}

```

![](https://img-blog.csdnimg.cn/20181116141846667.gif)

所以现在列的宽度至少 200px 。但是，如果有更多的可用空间，网格将分配给每个列，因为列的值变成了一个等分单位 1fr ，而不是 200px

**项目实际效果：**

![](https://img-blog.csdnimg.cn/20181116143804832.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE3NzE4NjMx,size_16,color_FFFFFF,t_70)

虽然 css 网格布局能轻松做到平均分配内容，但是实际实用上还是会有部分问题，比如要考虑浏览器的兼容，还有当只有一行数据没有参考行的时候，需要另外写兼容

**用户和浏览器支持**  
全球占 85% 网站流量的浏览器支持  
!([https://img-blog.csdnimg.cn/20181115224240488.png](https://img-blog.csdnimg.cn/20181115224240488.png)![](https://img-blog.csdnimg.cn/20181115230408808.png)  
![](https://img-blog.csdnimg.cn/20181115224933929.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE3NzE4NjMx,size_16,color_FFFFFF,t_70)
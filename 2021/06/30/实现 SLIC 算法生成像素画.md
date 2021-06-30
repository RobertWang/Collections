> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Ui7mRpXs2UqJIutjpIXPsA)

前言—
---

什么是 SLIC 算法—
------------

首先，引入一个概念——超像素。超像素是 2003 年 Xiaofeng Ren 提出和发展起来的图像分割技术，是指具有相似纹理、颜色、亮度等特征的相邻像素构成的有一定视觉意义的不规则像素块 [1]。

通过将图片分割为超像素，可以得到相似的像素簇，相似的像素使用同一个颜色进行填充，得到的像素画会更合理。

超像素点分割的方法包括了提取轮廓、聚类、梯度上升等多种。论文`[1]`提出的`SLIC`超像素点分割算法（简单线性迭代聚类，`simple linear iterative clustering`）就是其中一种，它基于`K-means`聚类算法，根据像素的颜色和距离特征进行聚类来实现良好的分割结果，与若干种超像素点分割算法相比，`SLIC`具有简单灵活、效果好、处理速度快等优势。![](https://mmbiz.qpic.cn/mmbiz_png/VicflqIDTUVUjoe9ntD53p3NoSUe5lbq7g5G6DzqlU2JxNwbHVEWmndCqueKXEPIWcwTH9a7UhDvPQbRiaER7WCA/640?wx_fmt=png)

如何实现 SLIC 算法—
-------------

`SLIC`的基本流程如下：

1.  图像预处理。
    
    将图像从`RGB`颜色空间转换到`CIE-Lab`颜色空间，`Lab`颜色空间更符合人类对颜色的视觉感知。这个空间里的距离能反映人感觉到的颜色差别，相关计算更为准确。
    
    `Lab`颜色空间同样具有三个通道，分别是`l`，`a`，`b`，其中`l`代表亮度，数值范围为`[0,100]`，`a`表示从绿色到红色的分量，数值范围为`[-128,127]`，`b`表示蓝色到黄色的分量，数值范围为`[-128,127]`。
    
    `RGB`和`LAB`之间没有直接的转换公式，需要将`RGB`转为`XYZ`颜色空间再转为`LAB`，代码见文末完整代码。
    
2.  初始化聚类中心。
    
    根据参数确定超像素的数目，也就是需要划分为多少个区域。假设图片有`N`个像素点，预计分割为`K`个超像素，每个超像素大小为`N/K`，相邻中心距离为`S=Sqr(N/K)`，得到`K`个聚类坐标。
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/VicflqIDTUVUjoe9ntD53p3NoSUe5lbq7sTn2XAibp2ugLHLRSDXuLIJzTfPho9PVKnTGPJQicN16YKYTL3icZyANg/640?wx_fmt=png)
    
3.  优化初始聚类中心。在聚类中心的`3*3`邻域内选择梯度最小的像素点作为新的聚类中心。
    
    把图像看成二维离散函数，梯度也就是这个函数的求导，当相邻像素值有变化就会存在梯度，而在边缘上的像素点的梯度最大。将聚类中心挪到梯度最小的地方可以避免其落到边缘轮廓上，影响聚类效果。
    
    离散梯度的梯度计算这里不做详细推导了，由于其中包含了若干平方与开方，计算量较大，一般会简化为用绝对值来近似平方和平方根的操作。简化后的计算坐标为`(i,j)`的像素点的梯度公式为：
    
    ![](https://mmbiz.qpic.cn/mmbiz_jpg/VicflqIDTUVUjoe9ntD53p3NoSUe5lbq760KfH5TDNIlVnpKYjYlctxA7QVksOgk3VUsc40XfJT66raR4uoOIxg/640?wx_fmt=jpeg)
    
    其中`（i+1,j）`与`（i,j+1）`为像素右侧点与像素下方点的坐标。`l(a,b)`为`（a，b）`坐标上像素的亮度通道值`l`。
    
4.  计算像素点与聚类中心的距离。
    
    在聚类中心距离 S 的区域内  `2S*2S`的邻域内计算像素点与每个聚类中心的距离。
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/VicflqIDTUVUjoe9ntD53p3NoSUe5lbq7ztJpzPibXC0qWLj1QOtViboHgTexPSR04PxCddVvZicQDJWqmKfusCDDw/640?wx_fmt=png)
    
    这里的距离使用的是欧式距离，总距离`D`由`dc`颜色距离与`ds`空间距离两部分组成。公式如下：
    
    ![](https://mmbiz.qpic.cn/mmbiz_jpg/VicflqIDTUVUjoe9ntD53p3NoSUe5lbq7wAicar15avbS7QmWRZz2XOoYQ6yDaL4oggb7MMwRibuznibreJ9CmBWLg/640?wx_fmt=jpeg)
    
    如果直接将`l`，`a`，`b`，`x`，`y`拼接成一个矢量计算距离，当超像素的大小变化时，`x`，`y`的值可以取到非常大 ，比如如果一张图`1000*1000`，空间距离可以达到`1000*Sqr(2)`，而颜色距离最大仅`10*Sqr(2)`，导致最终计算得到的距离值中，空间距离`ds`权重占比过大。
    
    所以需要进行归一化，除以最大值即超像素点的初始宽度`S`，将值映射到`[0,1]`。
    
    而颜色空间距离也会给到一个固定的值`m`来调节颜色距离与空间距离的影响权重，`m`取值范围为`[1,40]`。
    
    距离公式即变成了
    
    ![](https://mmbiz.qpic.cn/mmbiz_jpg/VicflqIDTUVUjoe9ntD53p3NoSUe5lbq7raoIibwspr9iah9lVG3trYFG85Q7f4wStrx56QicTV4ia6Bk69k6E68WRA/640?wx_fmt=jpeg)
    
      
    
    当`m`越大，颜色空间除以`m`后的值越小，即空间距离的权重越大，生成的像素会更为形状规则，当`m`越小，颜色距离权重更大，超像素会在边缘更为紧凑，而形状大小较为不规则。
    
5.  像素点分类。
    
    标记每个像素点的类别为距离其最小的聚类中心的类别。
    
6.  重新计算聚类中心。
    
    计算属于同一个聚类的所有像素点的平均向量值，重新得到聚类中心 。
    
7.  迭代`4～6`的过程。
    
    直到旧聚类中心与新聚类中心的距离小于一定阈值或者达到一定迭代次数，一般来说，当迭代次数到达`10`，算法能够达到收敛。
    
8.  聚类优化。
    
    迭代到最后，可能会出现与聚类中心不属于同一连通域的孤立像素点，可以使用到连通算法将其分配到最近的聚类标签。
    
    论文中并未给出具体的实现算法。而本文的应用场景是生成像素画，会对像素进行下取样，并不会细化到每个像素，由此，本文不做聚类优化处理。
    
    ![](https://mmbiz.qpic.cn/mmbiz_jpg/VicflqIDTUVUjoe9ntD53p3NoSUe5lbq7fOicByDCzIQ4ricaprlq7Gp52RroGmbuccq5M5vV16PqYBtJiciaficNia9Q/640?wx_fmt=jpeg)
    

小小总结一下，SLIC 算法流程大体与`K-means`是一致的，不断迭代计算距离最小的聚类簇，不同的是只对聚类中心的`S`距离内像素点进行计算，减少了不少的计算量。

生成像素画—
------

基于`SLIC`算法，我们已经可以把一张图划分为`N`个超像素点。每个超像素中像素都是相近的。也就是说，每个像素都被归类为一个超像素，有一个聚类中心。那么将像素的颜色赋值为其聚类中心的颜色即得到我们想要的效果。

设定一定步长`stride`，使用`Canvas`，每隔`stride`个像素，将像素赋值为其聚类中心的颜色，即得到最终的像素化结果。

![](https://mmbiz.qpic.cn/mmbiz_jpg/VicflqIDTUVUjoe9ntD53p3NoSUe5lbq7hS736n7iayubDsJn3nNgSbbFHdtFRVcVfAtXrQaGwuyjFAgQ9ID7HEw/640?wx_fmt=jpeg)

而每个人对于像素画的主观感受是不一致的，为了让用户有更多的选择，得到自己满意的结果。可以暴露更多的人工干预参数，比如取消聚类优化的终止条件，改为由用户来设置迭代次数，以及最终取像素值的步长。人工设定的参数包括了

*   超像素点大小`blocksize`；`blocksize`越小，超像素点分割越细腻。
    
*   迭代次数`iters`；`iters`越大，分割结果更精准，计算时间越长。
    
*   颜色空间权重`weight`；`weight`越大，颜色对于分割结果的影响越大。
    
*   取像素点步长`stride`；`stride`越小，生成的像素图越接近超像素点，也就越细腻。
    

实现用户交互界面—
---------

作为一个工具，自然需要用户交互界面，前端界面基于`HTML/Javascript/CSS`搭建，使用`Canvas API`绘制图像内容，而用户交互面板选择的是`dat.gui` [3] 库。`dat.gui`是一个轻量级的图像化界面库，非常适用于参数的修改，常用作可视化 Demo 的演示。支持的参数类型包括了`Number`、`String`、`Boolean`、自定义函数等。可以为不同的属性绑定相应的响应事件，当属性值改变时自动触发事件。

为生成像素化工具添加以下属性与事件：

*   当`iters、stride、blockSize、weight`（颜色空间权重 m）参数变化时重新进行`SLIC`算法的计算，并重新绘制计算结果；
    
*   添加`Upload image` 与`Export image`按钮，支持用户上传图片与下载像素化后的图片；
    

在绘制图像的`Canvas`画布层上叠加一层`Canvas`画布，对算法的结果进行可视化，添加以下功能

*   `grid`开关控制是否绘制像素网格；
    
*   `Centers`开关控制是否显示聚类中心；
    
*   `Contours`开关控制是否显示聚类边缘轮廓；
    

![](https://mmbiz.qpic.cn/mmbiz_png/VicflqIDTUVUjoe9ntD53p3NoSUe5lbq7vyiasZ6nMgFqFRIKh5TAtt4gibVaenrEJbc7nQFFqicMTY5Fj0O5kthlw/640?wx_fmt=png)

其中聚类中心点`Centers`的绘制直接使用`ctx.fillRect` 传入中心点坐标即可。

超像素轮廓`Contours`的绘制则需要先计算得到轮廓点。

可以对每个像素点与周围的`8`个像素点进行比较，如果聚类中心不同的像素点个数大于`2`，则代表着这个像素点周围有两个以上不同类别的点，则这个点为轮廓。效果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/VicflqIDTUVUjoe9ntD53p3NoSUe5lbq7CwJle0M1kHMPwpn0UtibMU0BUAibekaDnFnpfzgiatwvG1Vr8LbTgQiamw/640?wx_fmt=png)

最后，就得到一个简单的生成像素画工具了。

![](https://mmbiz.qpic.cn/mmbiz_png/VicflqIDTUVUjoe9ntD53p3NoSUe5lbq7v4f12hMppCAwRubxDzlDFFaeWrWpElt6RiaiaAicLxccopdDckvjEOjRg/640?wx_fmt=png)

体验地址 [1]

完整版代码地址（JS 版）[2]

参考文献—
-----

[1]

体验地址: _https://xs7.github.io/Pixelate/index.html_

[2]

完整版代码地址（JS 版）: _https://github.com/xs7/Pixelate_

[3]

_Achanta R, Shaji A, Smith K, Lucchi A, Fua P, Su ̈sstrunk S. SLIC superpixels. Technical Report. IVRG CVLAB; 2010._

[4]

_Gerstner T ,  Decarlo D ,  Alexa M , et al. Pixelated image abstraction with integrated user constraints[J]. Computers & graphics, 2013._

[5]

_https://github.com/dataarts/dat.gui_
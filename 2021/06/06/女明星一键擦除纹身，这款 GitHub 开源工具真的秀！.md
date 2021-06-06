> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg2MjUyNjM3Nw==&mid=2247486549&idx=2&sn=7ac59f5cf9626467fefdaf86ffe7eefc&chksm=ce07c293f9704b8538586bc6ee18a45f2b41b0361f31c1ab45ee2c5587026880f4a44327ba25&mpshare=1&scene=1&srcid=0604vSN8OQ1AGYRKctPnwxgI&sharer_sharetime=1622819346974&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

公众号关注 “GitHub 宝典”

设为 “星标”，每天带你逛 GitHub！

来自机器之心

> 深度学习去纹身的应用，看起来有不小的应用潜力。

有些时候，我们需要把一些人身上的纹身覆盖掉，以避免引人效仿。有的时候人们只是单纯地好奇，想知道一些大明星如果没有纹身会是什么样子。来自印度的机器学习研究者 Vijish Madhavan 最近开源的一个机器学习工具 SkinDeep 满足了我们的需求。

网友们也使用这一工具去处理了一些画了重度纹身的人物照片，效果还不错。

该项目的作者 Vijish Madhavan 在看完加拿大歌手贾斯汀・比伯的 MV《Anyone》后，萌生了做这个项目的计划。贾斯汀・比伯在化妆师的帮助下花了好几个小时的时间才把他的一身纹身覆盖掉。

MV 视频的效果非常完美，因为制作视频输出是非常困难的，因此项目作者选择图像来处理。该项目的起点是深度学习能否胜任这项工作，与 photoshop 相比又如何？

项目地址：https://github.com/vijishmadhavan/SkinDeep

有人会问，为什么不把纹身直接 PS 掉？Photoshop 可以产生非常好的效果，但问题是使用 Photoshop 需要专业知识，如果用 PS 处理纹身的话，你可能需要花费几个小时的时间去修饰整个图像。

我们先来看一下效果如何？阿伦・艾弗森（美国篮球运动员）的纹身就是用这个模型去掉的。

![](https://mmbiz.qpic.cn/mmbiz_gif/KmXPKA19gW9qk1GjSLCoYgpbkAZyF9DZjNT7Q4FicfJFfJ4azMFwMZaecjhHjahgicIa4fkNO0yKl3gpg9sEcqAA/640?wx_fmt=gif)

下图中第一行为输入图像，第二行为输出图像，输出结果明显感觉到纹身被去除了。

![](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gW9qk1GjSLCoYgpbkAZyF9DZHfQV0zibmFu7hz4dxlw6r4uQ2FKWSnksksAYksibuw3KXlNMeJdaJ3fg/640?wx_fmt=png)

脸部有大量密集纹身的图像，还有其他装饰，AI 的纹身去除效果也非常好：

![](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gW9qk1GjSLCoYgpbkAZyF9DZdbK1IgeTLdxSg91Bc4SEfO7eWZiaMqUScGdCGXZBfp4BicvYVP23AURA/640?wx_fmt=png)

与专业图像处理软件 photoshop 相比，效果也不错：

![](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gW9qk1GjSLCoYgpbkAZyF9DZjL6CJTRjH5frxBvOgjWIQo1ibgYEa4Zc3bWgicyhbUWicDBIMFQwrXotw/640?wx_fmt=png)

 看起来 SkinDeep 的效果还不错，但如果纹身是彩色的，还会有一些残留的痕迹。

**项目介绍**

根据作者介绍，完成这个项目需要大量的图像对，因为没有合适的数据集，很多时候训练内容采用合成数据来完成，具体来说：

*   首先将 APDrawing 数据集图像对与一些背景去掉纹身设计的图像叠加在一起，使用 Python OpenCV 实现；
    
*   绘制数据集有线条艺术对，可以模拟纹身线条，这将有助于模型学习和删除这些线条；
    
*   APDrawing 数据集只有头像，对于全身图像，项目作者采用了以前的项目 ArtLine，并将输出与输入图像叠加在一起；
    

![](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gW9qk1GjSLCoYgpbkAZyF9DZKM5Z5aqoMaYFd2E2ibbBQ55JUbx3qgU01iafttHw8Ulia6soNZmWMm3Vg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gW9qk1GjSLCoYgpbkAZyF9DZkeaZ088XOtIG8bibbJ9X2rRHl50pDgiaEZkVP32suQrpFTg07leqqZbg/640?wx_fmt=png)

*   ImageDraw.Draw 与森林绿色（forest green colour）色码一起使用，并随机放置在身体图像上，类似于 fast.ai 中的 Crappify ；
    
*   Photoshop 也被用来在需要弯曲和角度改变的对象上放置纹身。
    

![](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gW9qk1GjSLCoYgpbkAZyF9DZHiapyz5ScqKrk5KPneqGA9a1lQcrmHSce1on2Aq56YH4OoJswzoygnA/640?wx_fmt=png)

这一项目是由 Fast.AI 库构建的，你需要安装 fastai 1.0.61 版（及其依赖库），以及 PyTorch 1.6.0，不支持更高的版本。

尝试这一项目的最快方法就是在 Colab 上：

*   https://colab.research.google.com/github/vijishmadhavan/SkinDeep/blob/master/SkinDeep.ipynb
    

它的输出限制为 500 像素。

**限制**

去纹身的机器学习模型虽然看起来并不复杂，但在现实世界千奇百怪的情况下，有时仍然会出现一些「贴图错误」的情况。该项目的构建者表示，由于缺乏数据集支持，所以用于训练的数据集容量有限。另外，如果有人纹了彩色纹身，恐怕人工智能目前还是认不出来的。

![](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gW9qk1GjSLCoYgpbkAZyF9DZqqjdRDNicJCGb85pq8ibkOprSztsuSmcibaxV1EJ4U9lbRzCgKtc8wdMw/640?wx_fmt=png)

如果这个效果被做成网站，或者成为美颜 app 的一个滤镜，那就太好了。最后，SkinDeep 能不能反过来给人加纹身呢？「试穿」的效果或许会火起来。

公众号
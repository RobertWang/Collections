> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/fWw-Hsm6Cnyv3wdzzJPEpQ)

1 简介
====

费老师我在几年前写过的一篇文章（`https://www.cnblogs.com/feffery/p/13392024.html`）中，介绍过`tqdm`这个在当下`Python`圈子中已然非常流行的进度条库，可以帮助我们为任何具有循环迭代过程的代码逻辑添加进度条，从而帮助我们感知代码运行的过程。

而随着`tqdm`这几年来的发展迭代，更多更好用的功能加入其中，今天的文章中我就给大家总结了6条非常值得学习的`tqdm`特性。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/g64sbb6FfmcbDqM9YQZcKlAGG3GSH1Mpf1g2vdLlwcs8aTOwoZMGRSDcSR4lkf5JktY1XPibtcico0HpclkFtOpA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

2 tqdm中实用的6个特性
==============

2.1 autonotebook自动切换进度条风格
-------------------------

用过`tqdm`的朋友们大都知道它可以在常规的终端以及`jupyter`风格的各种编辑器中使用，且在后者中会以更美观的形式进行渲染，而以往我们通常需要在常规的终端里使用`from tqdm import tqdm`，在`jupyter`风格的编辑器中使用`from tqdm.notebook import tqdm`来分别导入。

而`tqdm`最近几个版本中引入了实验性质的新特性，使得我们只需要统一通过`from tqdm.autonotebook import tqdm`导入`tqdm`，就可以自适应检测不同的运行环境从而自动控制显示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/g64sbb6FfmcbDqM9YQZcKlAGG3GSH1MpRHhNOLbTXtEDb9YObXY1XcLicrbcLaibXlzB7icO9NSex6EhpibPFHLWibw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

2.2 延迟渲染进度条
-----------

有时候我们希望当循环过程很快就执行完时，可以不打印进度条，毕竟进度条的主要目的是监控长时间运行过程，这时我们就可以给`tqdm()`添加参数`delay`来设置延时的秒数，当循环过程实际运行时长低于`delay`则无需打印多余的迭代过程：

![图片](https://mmbiz.qpic.cn/mmbiz_png/g64sbb6FfmcbDqM9YQZcKlAGG3GSH1Mp2IuRzUTIQzekAqplUbe4VyGicwKRLun2t8yIiat3cc32O76meRcywTOw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

2.3 自定义进度条色彩
------------

通过为`tqdm()`设置参数`colour`，可以传入多种常见色彩格式值，这在`jupyter`类编辑器中效果尤为明显：

![图片](https://mmbiz.qpic.cn/mmbiz_png/g64sbb6FfmcbDqM9YQZcKlAGG3GSH1MpR9q0s93RSBlopnYc6GBZhvDZuKibpHfwLrapXJZfhjV4ZZarhWSo6Ow/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

2.4 自主控制的进度上限
-------------

有些情况下，我们传入`tqdm()`的对象在迭代过程中是无法预先计算得到进度上限轮次的，典型如`pandas`中数据框的`itertuples()`，这种时候我们就可以利用`total`参数自行预设上限：

![图片](https://mmbiz.qpic.cn/mmbiz_png/g64sbb6FfmcbDqM9YQZcKlAGG3GSH1MpC3Puqhr33bMT2PmibicZMltEg7tPtXNY5jS4Wo0OKCIZZhwhRLUDx2Cg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

2.5 针对enumerate、zip和map的替代
--------------------------

`Python`中除了常规的循环过程以外，还有几种内置函数也具有迭代循环的属性，而`tqdm`为了方便我们对这些非典型的循环过程添加进度条，也单独开发了`tenumerate`、`tzip`以及`tmap`这三个API，用于替代`enumerate`、`zip`和`map`：

![图片](https://mmbiz.qpic.cn/mmbiz_png/g64sbb6FfmcbDqM9YQZcKlAGG3GSH1Mpo7H6mL5KGWNVyicQatKfxM9YeHTwmepuVBytGgcJ8hAyx2MxlI3urGQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

2.6 设置进度条“用完即逝”
---------------

当我们希望为多层循环过程添加进度条监视时，常规的为每一层都直接使用`tqdm()`，会导致打印出过多的进度条，反而不利于我们观察进度过程。

而通过使用`tqdm.auto`中的`trange()`，我们可以通过设置参数`leave=False`，来让我们对应的进度条加载到头就自动消失掉，譬如下面动图中所展示的例子：

![图片](https://mmbiz.qpic.cn/mmbiz_gif/g64sbb6FfmcbDqM9YQZcKlAGG3GSH1MpKzLY4z92dDxfhHxp5OCaO2JicCfg4gIEy4AYKcmCE4SDRibAmwmhsvpA/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)

* * *

以上就是本文的全部内容，欢迎在评论区与我进行讨论~
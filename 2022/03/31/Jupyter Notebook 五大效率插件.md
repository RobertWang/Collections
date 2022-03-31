> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652577871&idx=3&sn=d794e9a37b69236141ca1fab95ae7e75&chksm=84650805b3128113b867fe703654dea350b6f7b4e5e20e91554d71f19cef5b291bb330c0e6a9&scene=21#wechat_redirect)

极简版教程
-----

在命令提示符中运行以下命令：

```
pip install jupyter_contrib_nbextensions && jupyter contrib nbextension install


```

启动 Jupyter Notebook，并导航至新的 Nbextensions 选项卡：![](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gW8icT5ibSYYR2p8wF5IdiaovvOdZLEWnPicHobHn9RpxHIPjcQz2x7ib95JbiaSHNr5fp467HD37h7uyj0w/640?wx_fmt=jpeg)

**Jupyter Notebook 扩展选项**

选择你想要的扩展功能，享受它带来的优势。

（如果你没看到扩展选项，打开 `notebook`，单击「edit」，然后点「nbextensions config」）

在 `notebook` 的工具栏里可以看到扩展插件：

![](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gW8icT5ibSYYR2p8wF5IdiaovvOHLibUIF1Uy0hlkNic8gjXccQdAhgeZtXKJMZAKFC3rt1bUYufMiaDdnJQ/640?wx_fmt=png)

**较长版教程**

如果你觉得上面的教程太简单，可以看看下面的复杂版。我还列出了我最喜欢的五个扩展插件。

**什么是 notebook 扩展插件？**

`Jupyter Notebook` 扩展插件是扩展 notebook 环境基本功能的简单插件。它们用 JavaScript 语言编写，会自动套用代码格式或者在单元格完成后发送浏览器通知。扩展插件目前仅支持 `Jupyter Notebook`（不支持 Jupyter Lab）。

为什么要使用扩展插件？Jupyter Notebook 是一个很好用的工具，可用于教学、学习、原型设计、探索和尝试新方法（甚至可用于 Netflix 的生产过程中）。但是，原版 notebook 功能有限，有时令人挫败。虽然 Jupyter Notebook 扩展插件没有完全解决这个问题，但它们确实能让你的工作变得更轻松。

**该用哪些扩展插件？**

以下是我最常用的五种 Jupyter Notebook 扩展插件：

1、Table of Contents：更容易导航
-------------------------

如果你在一个 Jupyter Notebook 中同时开启了十几个单元格，那你想跟踪所有单元格就会有些困难。Table of Contents 通过添加 TOC 链接解决了这个问题，通过 TOC 链接你可以定位到页面中的任何位置。

![](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gW8icT5ibSYYR2p8wF5IdiaovvOcLDx6hsLpbZV171kMWQmbITZTf74oEaCh1NzGwBJypctQ8fjHfajqQ/640?wx_fmt=jpeg)

**Table of Contents 界面**

你还可以使用该扩展插件在 `notebook` 的顶部添加一个链接目录。这样会显示你选择了哪一个目录以及哪一个正在运行：

![](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gW8icT5ibSYYR2p8wF5IdiaovvOHWId1JWjn46tL8ibrICQvMdgHuLufl0JeEDDOXwyzOpNicebRkd7WxQA/640?wx_fmt=jpeg)

_Notebook 中的 Table of Contents 界面_

2、Autopep8：轻轻一击就能获得简洁代码
-----------------------

我们都应该编写符合 pep8 标准的代码，但有时你会陷入分析，难以坚持这种标准。所以当你写完代码后，只要单击这个选项，就可以让代码变得简洁漂亮。

![](https://mmbiz.qpic.cn/mmbiz_gif/KmXPKA19gW8icT5ibSYYR2p8wF5IdiaovvOTYJxriaLRxiawBRbibicbVJicEpsUmtX5E3974SofREd6C6x8VibA6wicTdAw/640?wx_fmt=gif)

_使用 Autopep8 的效果_

这个插件可以称得上是最好的插件了，仅需点击一下，就能完成一项耗时又乏味的工作，让你专注于思考。

3、variable inspector：跟踪你的工作空间
-----------------------------

variable inspector 会显示你在 notebook 中创建的所有变量的名称，以及它们的类型、大小、形状和值。

![](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gW8icT5ibSYYR2p8wF5IdiaovvOQwt7QyIslBMAZuibMbNGAJiaiaEx20e6R3t56QZic1nMapO1P7ibHrNcrnQ/640?wx_fmt=jpeg)

**variable inspector**

这个工具对于从 RStudio 迁移过来的数据科学家来说是无价之宝。如果你不想继续打印 df.shape 或无法重新调用 x 的 type，这个工具对你来说也同样重要。

4、ExecuteTime：显示单元格的耗时
----------------------

我经常不知道某个单元格需要运行多久或者最后一次运行一个打开好几天的 notebook 是什么时候。ExecuteTime 完美解决这个问题，它会显示单元格的运行完成时间和所耗时长。

![](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gW8icT5ibSYYR2p8wF5IdiaovvOQcPgoZYhic8UTEEUiaULHgJh0ibdhek0RXlvVS0UVSXnpEsVMaCas1R5g/640?wx_fmt=jpeg)

**ExecuteTime 的输出**

的确有更好的计时方法，如 %%timeit，但 ExecuteTime 易于实现，且可以覆盖 notebook 中的所有单元格。

5、隐藏代码输入：隐藏过程，展示结果
------------------

虽然有些人喜欢看到某项艰苦工作的具体分析，但有些人却只想看到结果。隐藏所有输入的插件让你能够立即隐藏 notebook 中的所有代码，只保留结果。

![](https://mmbiz.qpic.cn/mmbiz_gif/KmXPKA19gW8icT5ibSYYR2p8wF5IdiaovvOTt6r5OAwicvFb5ZpMqXicrWEOw2G40fBaZrTibiauehUASvANf1VLSIUug/640?wx_fmt=gif)

**隐藏所有代码**

下一次如果有人说他们只想看结果，你只要单击一下就可以了。（不过你还是得经常检查代码。）

以上是我最常用的五种扩展插件，你还可以尝试其他扩展插件。先安装扩展插件，然后打开 notebook 探索吧！

结论
--

安装 `Jupyter Notebook` 扩展插件，花点时间弄清楚哪些有用，然后提高自己的工作效率。虽然这些功能不至于改变你的人生，但它们带来的益处也是值得的。而且累积起来为你节约了很多宝贵的开发时间。

如果你正在写生产代码，你可能会更愿意花些时间学习 IDE（我喜欢 `VS Code`），但 `Jupyter Notebook` 仍然是数据科学工作流程中不可或缺的一部分。如果你要使用这个环境，就最好把它用到极致，以从中获得最大的收益。

> 选自 towardsdatascience
> 
> 作者：William Koehrsen
> 
> 编译：机器之心（_nearhuman2014_）
> 
> 原文：_towardsdatascience.com/jupyter-notebook-extensions-517fa69d2231_

- EOF -

推荐阅读  点击标题可跳转

1、[七个 Python 必备的 GUI 库，这次一定要学会！](http://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652577852&idx=2&sn=d7e29796661f66a8ee54e692011f1554&chksm=84650876b312816020566f40db4a9b7bc81b8a2b62b129a9741a8f5821dec85e706799a32c70&scene=21#wechat_redirect)

2、[PyPy 为什么能让 Python 比 C 还快？一文了解内在机制](http://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652577821&idx=1&sn=14dbf11a2bdf97d65cdab8f4e946a586&chksm=84650857b3128141747a5a0541a80de4a4ac2d13c38b36e9fc5b7aa88b8b675bd061f15b6700&scene=21#wechat_redirect)

3、[使用 MitmProxy 玩爬虫的，这篇文章别错过了](http://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652577777&idx=3&sn=ff6338b1b7f4dd3709c03abb4810f7d1&chksm=846537bbb312bead1b873ba316110bdde1da03cadab80f082f2d91e16ef7ba9b1e0b1390339b&scene=21#wechat_redirect)
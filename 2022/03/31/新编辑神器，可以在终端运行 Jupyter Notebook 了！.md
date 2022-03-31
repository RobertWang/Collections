> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652577974&idx=2&sn=8d36db244e81b6c56da37301c7eb5626&chksm=846508fcb31281ead63692dfd5ed0769385919de8277c5371fb8a3aa211ff71da484f4b50dc6&scene=21#wechat_redirect)

在我初学`Python`的时候，一直惯用着安装式的编辑器软件，比如`PyCharm`和`Spyder`。并且，一直以为编辑器都是这种形式的，有的区别只是体验和功能上的差异。

web 和终端对比

更神奇的是，它也支持代码交互和`markdown`的富文本。虽然代码在哪敲都是敲，并没有改变本质，但真没想到基于 web 的`Jupyter Notebook`有一天还可以在命令行中运行，和大家分享一下，说不定哪天能用上。

这个工具就是 **`nbterm`**，下面来介绍下。

**GitHub 链接：**https://github.com/davidbrochart/nbterm

nbterm 的使用姿势
------------

### 1. 安装

支持各种软件管理包的下载。

```
$ pip install nbterm


```

或者`conda`

```
$ mamba install nbterm -c conda-forge


```

除此外，还需要一个内核，比如适用于`Python`的`ipykernel`或`xeus-python`，适用于 C++ 的`xeus-cling`。

### 2. 启动 notebook

```
$ cd ~/nbterm #你的nbterm存储路径
$ nbterm my_notebook.ipynb


```

然后使用终端来敲代码：

![](https://mmbiz.qpic.cn/mmbiz_gif/YicUhk5aAGtA1y0syEZic7Oe5N2LUD0icLs3wN0icPSKyDheK6R6NSduGfa2UzEDyR6e3Ky6LsRMCibHLf1Nm5kW03Q/640?wx_fmt=gif)

### 3. nbterm 基本命令

输入`help`可以看到`nbterm`命令的其它命令选项。

```
$ nbterm --help
Usage: nbterm [OPTIONS] [NOTEBOOK_PATH]

Arguments:
  [NOTEBOOK_PATH]  Path to the notebook.  [default: ]

Options:
  --no-kernel                     Don't launch a kernel.
  --run                           Run the notebook.
  --save-path TEXT                Path to save the notebook.
  --version                       Show the version and exit.
  --help                          Show this message and exit.


```

比如，在批处理模式中运行`notebook`所有单元。

```
$ nbterm --run my_notebook.ipynb


```

如果未使用`--save-path`指定新名称，则会自动生成名为`my_notebook_run.ipynb`的新文档。

嵌入式用法
-----

除了上面那样操作以外，也可把`nbterm`当作库嵌入到自己的程序中，所有协作者都可以进行编辑。比如你可以重新排列单元格，然后一起运行：

```
import asyncio
from nbterm import Notebook
nb = Notebook("my_notebook.ipynb")
nb.cut_cell(3)
nb.paste_cell(1)
asyncio.run(nb.run_all())
nb.save()


```

一个轻量级 Jupyter 的尝试
-----------------

这个工具的创作者叫 David Brochart，是一位任职于 quantstack 的软件开发员，致力于`Jupyter`生态库的开发，比如`nbclient`、`jupyter-client`、`ipykernel`、`ipywidgets`等。

他本人提到，`nbterm`不会重用 Jupyter 的基本组件，如`jupyter-client`和`nbformat`，而是想要尝试不涉及向后兼容限制的新项目，或者说测试一下开发一个轻量 notebook 客户端的难度，所以现在的`nbterm`还是一个相当精简的代码库。

显然，`nbterm`对于`notebook`而言还是有一些功能需要完善的，比如终端虽然只限制于显示字符。不过 ASCII 码可以使这个问题迎刃而解。

大佬已经使用`ASCII`后端对`matplotlib`图形库尝试了绘制：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/NOM5HN2icXzyqAQEnB4icK7zYQyaMLAzgicJVg5CCtH0PuDpfWaHACdwjROAic0iaM8I8LiackrMCOJ7v67V2zmxXr0Q/640?wx_fmt=png)

但这个绘制目前还只能在`MacOS`上使用。

除此之外，该项目也提出了要添加一些类似`ipywidgets`的交互功能，以及更多简单的滑块、按钮、菜单等 GUI 部件等。

这个工具虽然目前来看可用性不强，但也确实是一个启发。就像我当时觉得 web 敲代码很奇怪一样，随着逐渐熟悉也就习惯了，**只要它香我在哪敲都行，命令行里敲还能顺便装一下**![](https://mmbiz.qpic.cn/sz_mmbiz_png/NOM5HN2icXzyqAQEnB4icK7zYQyaMLAzgiclicTOr2P1WvjHUoByCGsYgyXYvew4Yy9DbZN8eiaUodfxwxxib0xFUFjA/640?wx_fmt=png)

参考链接： 

[1] https://github.com/davidbrochart/nbterm 

[2] https://blog.jupyter.org/nbterm-jupyter-notebooks-in-the-terminal-6a2b55d08b70

- EOF -

推荐阅读  点击标题可跳转

1、[Jupyter Notebook 五大效率插件](http://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652577871&idx=3&sn=d794e9a37b69236141ca1fab95ae7e75&chksm=84650805b3128113b867fe703654dea350b6f7b4e5e20e91554d71f19cef5b291bb330c0e6a9&scene=21#wechat_redirect)

2、[又一个 Jupyter 神器，操作 Excel 自动生成 Python 代码！](http://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652577636&idx=1&sn=83adc28a0290b92fe8be5873f3fe4907&chksm=8465372eb312be389ae83ae699ee94a3c4289031cdc01d1bf3e89c5ccb634de84dd3ab790c2f&scene=21#wechat_redirect)

3、[大幅提高生产力：你需要了解的 10 大 Jupyter Lab 插件](http://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652576614&idx=2&sn=3778e12191bd6bd427b7042098b6cbed&chksm=8465332cb312ba3a86571d097646a5223b45dad5a63297af80ae3eebef9f5bfb0288dc94da1b&scene=21#wechat_redirect)
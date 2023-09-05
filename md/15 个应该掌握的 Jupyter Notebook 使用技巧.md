> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/256039090)

Jupyter Notebook 是一个基于浏览器的交互式编程环境（REPL, read eval print loop），它主要构建在 IPython 等开源库上，允许我们在浏览器上运行交互式 python 代码。并且有许多有趣的插件和神奇的命令，大大增强了 python 的编程体验。

1. 计算单元的执行时间
------------

我们可以在一个 jupyter notebook 单元的开头使用 %%time 命令来计算执行该单元的时间。

![](https://pic2.zhimg.com/v2-df2ae1c166f26107e72241cc6e0978fd_b.jpg)

2. 进度条
------

可以使用 python 外部库创建进度条，它可以实时更新代码运行的进度。它让用户知道正在运行的代码脚本的状态。你可以在这里获得相关的库 [Github 库](https://link.zhihu.com/?target=https%3A//github.com/tqdm/tqdm)。使用进度条设置，具体操作如下：

首先，安装 tqdm 库：  
在 cmd 命令窗口输入 pip3 install tqdm 或者直接在 jupyter notebook 单元内输入! pip install tqdm。

然后，通过以下命令生成进度条：  

![](https://pic3.zhimg.com/v2-6ffe7da10068bca56f119030b5f24f16_b.gif)

3. 代码格式自动补全
-----------

有时 jupyter notebook 单元格中的代码段格式不好，通过 nb_black 库，可以自动调整代码段的正确格式，让代码具有更好的可读性。

安装 nb_black 库：

pip3 install nb_black

在 jupyter notebook 中使用：

%load_ext nb_black

![](https://pic2.zhimg.com/v2-edfe33f654142ba8019295155ebe54a1_b.jpg)

  
格式混乱的代码段

![](https://pic2.zhimg.com/v2-65ccb6cb30d08fc77d966059e52045a1_b.jpg)

  
自动调整后的代码段

4. 下载并安装 Python 库
-----------------

Jupyter notebook 可以通过在单元格内输入! pip install *** 代码，自动下载并安装指定的 python 库。  
以 pandas 库为例，具体代码如下：  

![](https://pic2.zhimg.com/v2-a066f7eb56faf68e16e17f266b445f49_b.jpg)

5. 函数说明文档
---------

通过 shift+tab 快捷键，可以在 jupyter notebook 内直接打开函数的说明文档。

具体使用方式如下：

*   输入使用的函数名
*   按下快捷键 shift+tab
*   点击弹出窗口中的 ^ 按钮可以在当前窗口中显示说明文档
*   点击 + 可以控制文本向下滑动
*   点击 x 可以关闭说明文档窗口

![](https://pic1.zhimg.com/v2-33f1ab710f2ad0340489ccc5743a65b8_b.jpg)

  
pandas 中 read_csv 函数的说明文档

6. 代码自动补全
---------

Jupyter notebook 可以显示任何函数名或变量的补全建议。若要查看补全建议，可以按键盘上的 Tab 键，建议将出现在一个自上而下显示的菜单中。单击关键字或在所选关键字上单击 enter 键以确认补全的代码。

![](https://pic4.zhimg.com/v2-f834f4be18a5a621663efa1099e9a79f_b.jpg)

  
pandas 中函数的补全建议

7. 调整输出结果的显示窗口
--------------

Jupyter notebook 可以在代码单元格的下方显示输出。当用户的输出过多时，可以选择调整显示窗口的尺寸，将该显示窗口调整为一个滚动窗口。并且在显示窗口左边双击，可以折叠该窗口。

![](https://pic2.zhimg.com/v2-d969f5043a8fce8fd7b7f5e96dc63ebd_b.jpg)

  
调整显示窗口

8. 单元运行快捷键
----------

通过以下快捷键可以提高编程效率：

*   shift+enter 运行当前单元，并且高亮显示下一单元，如果没有下一单元就新建一个单元。
*   alt+enter 运行当前单元，并且插入一个新单元并高亮显示。

9. Markdown 笔记
--------------

Jupyter notebook 的单元格不仅可以运行代码段，还可以设置单元格为 Markdown 方式用来编写文本。

转换方式如下：

*   点击目标单元格
*   选中 “Markdown” 选项

![](https://pic2.zhimg.com/v2-3920058ae4a37ee076121c5102d831e1_b.jpg)

  
单元格由代码模式转换至 Markdown 模式

![](https://pic3.zhimg.com/v2-dd0e6676acbcb9757f60c21cbc05f932_b.jpg)

  
Markdown 模式运行效果

10. 运行不同的编程语言
-------------

Jupyter notebook 还可以用来编译和运行来自不同语言的代码。只需要在单元格开头处输入 %%**** 命令，就可以运行 **** 对应的语言代码：

*   %%bash
*   %%HTML
*   %%python2
*   %%python3
*   %%ruby
*   %%perl

![](https://pic3.zhimg.com/v2-387a0e3336d83e6ec071cc06ef66d7be_b.jpg)

  
在 jupyter notebook 单元格内运行 HTML 代码

11. 多行同时编辑
----------

Jupyter Notebook 支持同时使用多个光标编辑代码。通过 alt 键选择要编辑的代码段后，可以同时使用多个光标编辑代码。

![](https://pic4.zhimg.com/v2-6591b305b1bf2820655124c3497f9bcf_b.gif)

12. 创建演示文档
----------

Jupyter notebook 可用于创建 PowerPoint 样式的演示文稿。在这里，笔记本的每个单元格或单元格组都可以视为幻灯片。

*   首先，安装 RISE 库（conda install -c damianavila82 rise）
*   安装后，RISE 相关按钮将会添加进工具栏（view->cell->toolbar->slideshow）
*   选中需要展示的单元格，可将其设置为一个幻灯片
*   选择完毕后，点击 RISE Sliedeshow 按钮完成演示文档的创建

![](https://pic4.zhimg.com/v2-1da170a94e15183730ae9bb268ece133_b.gif)

13. 共享 Jupyter notebook
-----------------------

程序代码写完后，Jupyter notebook 提供了多种形式以便于用户进行分享：

*   以 HTML, PDF, ipynb, py 等文件格式进行分享  
    

![](https://pic4.zhimg.com/v2-9203b4a3edc0439d961a3395f02843ef_b.jpg)

*   使用 JupyterHub，它可以创建一个多用户共享 Hub，该 Hub 生成、管理和代理用户 Jupyter 笔记本服务器。
*   直接上传到网络当中

14. 数据展示
--------

Jupyter notebook 可以通过众多的 python 库和 R 语言相关库，生成不同的图表。常用的库有：

*   Matplotlib
*   Seaborn
*   bokeh
*   [http://plot.ly](https://link.zhihu.com/?target=http%3A//plot.ly)

![](https://pic3.zhimg.com/v2-e0183bf6d4d7c92576d6b862a016203e_b.jpg)

  
各种图表样式

15. 快捷键方式
---------

使用快捷方式可以节省程序员大量的时间并优化编程体验。Jupyter notebook 有很多内置的键盘快捷键，可以在 “help” 菜单栏下找到：“help”>“Help>Keyboard Shortcuts”。

Jupyter notebook 还提供了编辑键盘快捷键的功能，以方便程序员进行个性化设置。

![](https://pic3.zhimg.com/v2-c118c9ba980d3fb87c728d2b5a013c9a_b.jpg)

  
快捷键面板（命令模式）

快捷键面板（编辑模式）

作者：Satyam Kumar

deephub 翻译组: Oliver Lee
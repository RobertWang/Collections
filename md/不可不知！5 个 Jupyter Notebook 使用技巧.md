> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU5Nzg5ODQ3NQ==&mid=2247516825&idx=2&sn=51dabe624353eeafaa51ce6f830b2158&scene=21#wechat_redirect)

大家好，我是小五🧐

大家在学习 Python 并熟练掌握后，往往会选择更专业的 IDE（PyCharm 等）。不过熟悉我的小伙伴会发现，小五经常使用的还是`Jupyter Notebook`。

这是因为我觉得它是一款集 Python 编程和**写作**于一体的效率工具！

本文将介绍几个有趣有用的 Jupyter Notebook 使用技巧~

快速打开 Jupyter
------------

第一个技巧，如何快速打开 Jupyter Notebook？

这个真的太简单了，可以选择双击应用图标，不过这时候打开的是默认路径。

![](https://mmbiz.qpic.cn/mmbiz_jpg/tXYict40xfLhuhjvEoTJCWqPoYKxDtAUjcG6e3caDx1NL7RrVyjO3xDRhehLMFfPIbXtqGLwZWvJHNKy6pALdhA/640?wx_fmt=jpeg)

还可以选择从命令行打开，输入：jupyter notebook 回车即可。

![](https://mmbiz.qpic.cn/mmbiz_png/tXYict40xfLhuhjvEoTJCWqPoYKxDtAUjKc7fSPOcPYecVXibZpGPGQCND51MibEhxSUB06Z72JoKiaD7xogZpRgibw/640?wx_fmt=png)

想指定路径的话，就先 cd 命令进入目标路径，再执行上面一步，即可在指定文件夹快速打开 Jupyter Notebook 了。

不过还是略麻烦，幸好我整理了更加简单的办法，点击下方蓝字查看技巧。

[如何在 指定文件夹 快速打开 jupyter notebook](https://mp.weixin.qq.com/s?__biz=MzU5Nzg5ODQ3NQ==&mid=2247511126&idx=2&sn=684a62cf38ddb5c34d415555474cb299&chksm=fe4e89d2c93900c4c052eaf1212e3d492c7f9d8b1cb99cce7869d055016c96820c61cc2c6bd2&scene=21#wechat_redirect)

[将偷懒进行到底，直接双击打开. ipynb 文件](https://mp.weixin.qq.com/s?__biz=MzU5Nzg5ODQ3NQ==&mid=2247512262&idx=2&sn=6356ae043bd0e269e6f8ae37b44a08b6&chksm=fe4e9542c9391c541844b19f15f04cdd2d36552a566ee4d1357e889ff708f24cc3d3b0486f84&scene=21#wechat_redirect)

快捷键
---

我们都知道快捷键的重要性，熟练掌握快捷方式可以大大提升我们的工作效率。

*   按下`Esc`键，进入命令模式
    
*   按下`H`键，进入编辑模式
    

Jupyter 笔记本有两种不同的键盘输入模式。**编辑模式**允许您将代码或文本输入到一个单元格中，并通过一个**绿色**的单元格来表示。

**命令模式**将键盘与笔记本级命令绑定在一起，并通过一个灰色的单元格边界显示，该边框为**蓝色**的左边框。

![](https://mmbiz.qpic.cn/mmbiz_png/tXYict40xfLhuhjvEoTJCWqPoYKxDtAUj1KSyPc022OVicSZKGa1P7uxBPfgBwXpiaYrvDZB3OysYsY94ibqyOsaKQ/640?wx_fmt=png)

如何查看所有的快捷键呢？

同时按`Esc + H`键即可调出下图中的快捷键大全。

![](https://mmbiz.qpic.cn/mmbiz_png/tXYict40xfLhuhjvEoTJCWqPoYKxDtAUj9kbJ2Nib7vTAWxd40nDMv8GyN7RXCnbkp80ibutG6RAvN9icAdQEYVoQQ/640?wx_fmt=png)

### 常用快捷键

这里演示一下最最最常用的快捷键

当 cell 处于命令模式（即单元格左框为蓝色）时，可以使用以下快捷键。

*   按`a`键，在当前单元格前面生成一个新单元格
    
*   按`b`键，在当前单元格后面生成一个新单元格
    
*   按`x`键，删除当前单元格
    
*   按`z`键，恢复刚刚删除的单元格
    

Markdown
--------

前文有提到，小五认为 Jupyter Notebook 是一款集 Python 编程和**写作**于一体的效率工具！最大的原因就是它完美兼容了 Markdown，这样就可以非常舒服的添加各种文字注释、思维导图等。

在 cell 处于编辑模式（即单元格左框为绿色）时，默认是代码格式。

![](https://mmbiz.qpic.cn/mmbiz_png/tXYict40xfLhuhjvEoTJCWqPoYKxDtAUjtibXPTNtgNzhKKbLC5niac4n1KJqpUFVgrTwrKPQVcra3iazhTys5w5MQ/640?wx_fmt=png)

如果想使用 Markdown 的话，可以手动在菜单栏选择本单元格的格式为 Markdown。

不过不推荐这种办法，可以直接使用快捷键。

*   按`y`键，切换为代码格式
    
*   按`m`键，切换为文本（Markdown）格式
    

![](https://mmbiz.qpic.cn/mmbiz_png/tXYict40xfLhuhjvEoTJCWqPoYKxDtAUjZgt2oicT1ibeib5ibmj2jA2CIOjgzNfrKicw4823UA480NyWH5ZLoNOrUoA/640?wx_fmt=png)

同时编辑多行
------

复制过来的代码或者文本会有需要调整的部分，我之前更喜欢用`Sublime Text`去处理这类问题。

其实 Jupyter Notebook 也可以实现同时编辑多行，先按下`alt`键，再用鼠标左键进行选择即可。

![](https://mmbiz.qpic.cn/mmbiz_gif/tXYict40xfLhuhjvEoTJCWqPoYKxDtAUjVXUkWWE2DVqXVT5Sg2DiaBOUzezLGGOEiccIJj4C446aRqWsLoz4PeOw/640?wx_fmt=gif)

省时省力。

输出多个变量
------

在 Jupyter Notebook 运行 python 代码，默认只输出最后一个变量的结果。

![](https://mmbiz.qpic.cn/mmbiz_png/tXYict40xfLhuhjvEoTJCWqPoYKxDtAUjIPIeVQeg5rmGFdW5Kga5M7176ic8miaXBb4iaiaXOILOXsKqtUibpFn35tA/640?wx_fmt=png)

如果写很多个 print 打印输出呢，又不太优雅。后来我看到了一个办法：设置 `InteractiveShell.astnodeinteractivity` 参数为 `all`即可。

```
from IPython.core.interactiveshell import InteractiveShell 
InteractiveShell.ast_node_interactivity = 'all'
```

![](https://mmbiz.qpic.cn/mmbiz_png/tXYict40xfLhuhjvEoTJCWqPoYKxDtAUj8zBK7aplKvKibHe1TKmybQoiaLX4fhvdPMk9weIKHgSEoNeib3hPOhs4Q/640?wx_fmt=png)

这样就让所有的变量或者声明都能显示出来了。

好了，今天只介绍了 5 个`Jupyter Notebook`的使用小技巧。更多的比如魔法命令，各种好用的插件，我们下篇继续安利~

觉得有技巧帮助到你提升效率的话，别忘了给本文点个赞哦👍
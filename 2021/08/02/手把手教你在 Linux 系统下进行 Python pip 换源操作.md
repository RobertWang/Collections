> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1627906445&ver=3228&signature=DjychfE2TTJbXSF8NvKPFXr2leuEx7uO-fDQ8znDxmXk66Hb3*FP0Ryrldw1PYQ6uk8X0s2hEta-Ciw6yvTt*zwHD20ppE2Z5akbSL7r-Im1r-TRKbcDgJe3Z2GvCOwl&new=1)

    pip 版本为 9.0.1。  

![](https://mmbiz.qpic.cn/mmbiz_png/icjSAZybsq555HlvFgkB9bzK9cM6ibpU9KlhHb7z1xO19TQP6RjUx5dXym2ynTWJl11ibNOlZ1zKY1BcYa6pFoiahQ/640?wx_fmt=png)

    其实这个版本很低了，我们需要更新一下。

    使用升级命令：

```
sudo Python3 -m pip install --upgrade pip

```

![](https://mmbiz.qpic.cn/mmbiz_png/icjSAZybsq555HlvFgkB9bzK9cM6ibpU9K0TvF6awWxf4Korqp4OeicgW5aialdELbCkEJfhDb7hNCB5OusXT9CaDw/640?wx_fmt=png)  

    成功更新成最新版。

    之后执行命令：

```
pip -V

```

![](https://mmbiz.qpic.cn/mmbiz_png/icjSAZybsq555HlvFgkB9bzK9cM6ibpU9KskUQlksTFnNgnMACUfH3g3DySOibwtyhO6rOLViapuEyRyyxy7YY2b7Q/640?wx_fmt=png)

    其实应该执行的是 pip3 -V 但是不知道为什么这里需要执行 pip -V，似乎是更新完之后就直接成 pip 了，不再是 pip3 了。

    pip 默认源地址，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/icjSAZybsq555HlvFgkB9bzK9cM6ibpU9K1EOlpOEwiadicy1foHU4C7wibdYShXPvkdicMMPqkfE3F8e1lDJl0r9SdQ/640?wx_fmt=png)

    可以看得出来，默认都是官网源，并且很慢。

**/3 国内源列表 /**

    首先来看国内的源列表。老规矩，先列出源列表，如下所示。

**/4 换源流程 /**

    换源命令：

```
pip config set global.index-url 源链接

```

    如果在上述过程中，你已经更新了 pip 版本，并且 pip 的版本 > 10，你就可以使用此命令换源，不需要进行复杂的新建文件什么的操作。

    例如清华源：

```
pip config set global.index-url https：//pypi.tuna.tsinghua.edu.cn/simple/

```

![](https://mmbiz.qpic.cn/mmbiz_png/icjSAZybsq555HlvFgkB9bzK9cM6ibpU9KAXDib0ZIgW0w4K1flKtQ0Q2485vOPvbTaglKicyGIWlBQQMkZLVBKxXg/640?wx_fmt=png)

    writing 代表写入成功，一条命令搞定，自己也省的折腾了，多好。

**/5 效果展示 /**

换源前：

    下载速度是 16kb/s，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/icjSAZybsq555HlvFgkB9bzK9cM6ibpU9K1EOlpOEwiadicy1foHU4C7wibdYShXPvkdicMMPqkfE3F8e1lDJl0r9SdQ/640?wx_fmt=png)

换源后：

    下载速度是起步都是 500+k/s，最高都到 4.4m/s，是不是很 Nice！

![](https://mmbiz.qpic.cn/mmbiz_png/icjSAZybsq555HlvFgkB9bzK9cM6ibpU9KFfykITI6Gd2or7HqdQZfhlPZTu5axEGsy5AVZKwWsegUO3bicXfiaTUQ/640?wx_fmt=png)

    果然是，没有对比就没有伤害。  

    黄色警告的意思是正在使用一个旧的 pip 安装，然后它切到了新的 pip。

    如果没理解错的话，在上述执行过命令后

```
sudo Python3 -m pip install --upgrade pip

```

 pip3 命令已经成为 pip 了，以后只需要使用 pip 就可以了。

 打开 Python 控制台，导入 requests 包，并没有报错，说明 pip 安装的是成功的。

![](https://mmbiz.qpic.cn/mmbiz_png/icjSAZybsq555HlvFgkB9bzK9cM6ibpU9KjHPjJwXcr4tqibR9HpWnFLzA9lzzvnibfQWTmEOyKOJMuEm2g4cQic04g/640?wx_fmt=png)

**/6 小结 /**

    本文主要内容是针对 Linux 系统下进行 Python 的 pip 换源操作，换源之后下载库的速度较换源前要快很多，方法简单且行之有效，欢迎大家积极尝试。

    如果中间提示有黄色等内容，不要着急，可能是文件夹什么的没有创建，它会自动创建。

     如果在操作期间有任何问题记得留言哈，小编看到的话会一一解决，谢谢各位小伙伴的支持。

**********---**--************---****---**********---**--************---************************************ End **********---**--************---**--**---**********---**--************-************************************
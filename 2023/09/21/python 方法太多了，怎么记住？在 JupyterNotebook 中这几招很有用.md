> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/luzrugQy0khnQHoMqdxL7w)

> 公众号回复 \ x26quot; 工具 \ x26quot;，获取 pandas 方法查询工具 Python 数据人你竟然不会用 JupyterNotebo

> Python 数据人你竟然不会用 JupyterNotebook！

* * *

很多初学者学习一些 python 库的时候，最难受的事情莫过于记不住大量的函数或方法名字，实在说，我自己也是时常忘记。为什么？因为实在是太多方法了。

比如最常用的 pandas，就用上千个的方法，谁会这么无聊去记住他们。

今天我就来把我日常查阅库方法的技巧列出来。最后一招你肯定没有看过！

* * *

如果你是在用 IPython 或相关环境，比如 jupyter notebook 等，此时你可以利用问号`?`查询方法。注意是英文的问号：

![](https://mmbiz.qpic.cn/mmbiz_png/0h1HziavCGLInZRQBNQPT334H8iajEot2icfODR7rQ1lNDmHJOCUfq7G1WFqW4nJ38hj1bibDDscIBLtxWEurRP6lw/640?wx_fmt=png)

但是，我可能根本不记得完整的方法名字。  

不怕，他还支持模糊查询：

![](https://mmbiz.qpic.cn/mmbiz_png/0h1HziavCGLInZRQBNQPT334H8iajEot2icqwu93pAJWvgicXDVvBVpI9EkZVGytlKeUaBngYfeDJWQPacby9P4ic7w/640?wx_fmt=png)

*   星号是通配符，表示任意内容
    
*   最后的问号表示查询的意思
    

此时我们就知道， DataFrame 中 sort_ 开始的方法名就有这 2 个。

* * *

此时，我们知道方法名字，要查阅方法的使用文档，可以用双问号：

![](https://mmbiz.qpic.cn/mmbiz_png/0h1HziavCGLInZRQBNQPT334H8iajEot2icAahm6MibibibvNiadk8PBJYjfkKrRWjmPJ550jfGHX0Qomf70auprNIamw/640?wx_fmt=png)

*   注意，此时不能使用星号，而且必需输入完整的方法名字
    

这是我以前经常使用的技巧。

但是，用久了你就会发现，这实在太麻烦，每次我都要先用单问号 + 通配符查询，然后再用双问号得到文档。

并且，每次都需要在 jupyter 上执行一下。

每次要换另一个类型的方法，就要重新修改查询代码。

有没有更加方便的方式？

* * *

最近我在学习 fastapi 和一些前端的东西。于是就把这个查询 pandas 方法 作为需求，制作了一个小工具。他可以方便我们查询方法和文档。

直接看动图感受下：

*   不需要使用通配符，自动模糊查询
    
*   不需要敲打多余按键，输入结束一定时间 (几百毫秒内)，就自动查询
    
*   一次查阅多种类型的方法，简单勾选就可以指定某种类型
    
*   点击 "复制" 按钮，即可复制方法名字到剪切板，不再需要 ctrl + c ，ctrl + v 了
    

当然，现在这工具也有缺点，就是只能查询 pandas 的常用类型的方法。后面有空再改进一下，让大家可以自己配置支持其他的库吧。

> 当 python 和 js 两门动态语言结合，可以做出非常灵活的工具

别忘记点个赞，转发，支持一下 ^^^

今天先到这里
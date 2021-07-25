> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU5Nzg5ODQ3NQ==&mid=2247517158&idx=2&sn=48c38803cd034c23a316ef126d855a7e&chksm=fe4ea662c9392f7427fdd0ea27b790ee1425f6e16a1fe239968158aaa1f4e037ceb65d78882a&mpshare=1&scene=1&srcid=0724K0JzmnQARwQdl7vnRo0f&sharer_sharetime=1627115806169&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

大家好，我是小五🧐

上周一共分享了 10 个有趣的 Jupyter Notebook 使用技巧👇

[5 个 Jupyter Notebook 使用技巧（第一弹）](https://mp.weixin.qq.com/s?__biz=MzU5Nzg5ODQ3NQ==&mid=2247516825&idx=2&sn=51dabe624353eeafaa51ce6f830b2158&scene=21#wechat_redirect)

[5 个 Jupyter Notebook 使用技巧（第二弹）](https://mp.weixin.qq.com/s?__biz=MzU5Nzg5ODQ3NQ==&mid=2247516876&idx=2&sn=6d6885912e9579f4df5ec3bc1f884fff&scene=21#wechat_redirect)

花几分钟掌握部分技巧，就可以提升自己的工作效率，节约时间岂不乐乎？

今天是第三篇，本文继续介绍有趣有用的**魔法命令**。

魔法命令
----

何为魔法命令？

官方给出的定义是：IPython 有一组预先定义好的所谓的魔法函数（Magic Functions），你可以通过命令行的语法形式来访问它们。这些指令独立于 Python 语法，可以完成一些特殊的功能。

魔法命令共分为两类：

*   **行魔法命令 (line magic)** : 前缀为 "%"，且全部指令（包含主要参数）不可以换行。
    
*   **单元格法术 (cell magic)**: 前缀为 "%%"，整个单元格都是魔法命令，单元格第一行必须是 "%%"
    

在单元格 Cell 中输入`%lsmagic`即可查看所有的魔法命令，如果临时忘了某个函数名怎么拼写，可以用此查询。如果连 %lsmagic 也忘了的话，问搜索引擎吧（摊手）。。。

![](https://mmbiz.qpic.cn/mmbiz_png/tXYict40xfLhabrCZtDHLnCA8GXyeXdtCUEgJ5xqbCaqKFXcl18dpySMdp5qL3ZEoos0OrPjhpSDrbscj02uc2A/640?wx_fmt=png)

关于查询魔法命令，还有两个函数比较常用：

*   `%quickref`：输出所有魔法命令的简单版帮助文档
    
*   `%Magics_Name?`：输出某个魔法命令详细帮助文档
    

下面我将先罗列一下常见的魔法函数，并对其中五个进行详细演示讲解。

![](https://mmbiz.qpic.cn/mmbiz_png/tXYict40xfLhabrCZtDHLnCA8GXyeXdtCicfw1kkyibkaLlWt2zvxhu4zxpqPB3s2QnvEzkHfAUjfGYVQnB8jzk4g/640?wx_fmt=png)

注：由于 IPython 的内置 magic 函数，那么在 Pycharm 中是不会支持的。

计算运行时间
------

有时候我们需要进行代码优化，就需要计算对比一下函数或过程运行时间，以此来衡量**代码的效率**。

在 Jupyter Notebook 中有几个魔法函数可以实现，功能效果各有差异。

![](https://mmbiz.qpic.cn/mmbiz_png/tXYict40xfLhabrCZtDHLnCA8GXyeXdtC2Qxx7G5qdSFibz6oVNkhur7NS6AniamiaoeibcOMwLM5sEK8rYB25TDy0w/640?wx_fmt=png)

这样我们就可以快速得到代码的运行时间，以此来对比代码的优化效果。

![](https://mmbiz.qpic.cn/mmbiz_png/tXYict40xfLhabrCZtDHLnCA8GXyeXdtCDty9ehwjicheFd6FWIcUsMWicNTFnWOz7kyoYZUXuws4icG3YFWG4S89Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/tXYict40xfLhabrCZtDHLnCA8GXyeXdtCCyRzCcvd9RicdU0HbgID7V1KOe4nK61n7VGD4rviaAI4Aqm8ehJtJh7A/640?wx_fmt=png)

查看当前变量
------

在 Jupyter Notebook 一行一行编写代码时，会发现自己定义的变量越来越多，到后面就不好想起来都在哪些单元格定义了哪些变量了。

其实我们可以使用魔法命令——`%who_ls`查看已经定义了哪些变量？

![](https://mmbiz.qpic.cn/mmbiz_png/tXYict40xfLhabrCZtDHLnCA8GXyeXdtCQUicu6QHh0wRzPKX02MCcjwJd1ImFdiaILeCvXwtib03eibKKj25g91KEA/640?wx_fmt=png)

包括导入的模块别名也在其中。

其中还可以**指定变量类型查看**，比如我们只想查看目前已经定义了哪些列表：

```
%who_ls list
```

![](https://mmbiz.qpic.cn/mmbiz_png/tXYict40xfLhabrCZtDHLnCA8GXyeXdtCUxC4X2PnzibMlYOghahXKtfNLMQVkwdvwiblgicpGWymiae3DGrG3Nw0Uw/640?wx_fmt=png)

保存单元格内容到文件
----------

我们可以在 Jupyter Notebook 只保存部分内容为. py 文件。这里需要使用魔法命令`%%writefile`，它的作用是将单元格中的内容保存到外部的文件中。

在下方的演示中，将代码

```
message = "Hello World"
print(message)
```

保存为`test.py`文件

![](https://mmbiz.qpic.cn/mmbiz_png/tXYict40xfLhabrCZtDHLnCA8GXyeXdtC2jKQcnYhqnOU2clEVqAAN3kxCkGvL7ibccZ6DxRkn1NDOlicys1IrrdA/640?wx_fmt=png)

打开当前工作目录，可以发现已生成`test.py`文件

![](https://mmbiz.qpic.cn/mmbiz_png/tXYict40xfLhabrCZtDHLnCA8GXyeXdtCeiaeHmsodEdNYwUg9eR2qUAXwmlf7ibA16gEEia5QNfrPMGzxDuh85zvA/640?wx_fmt=png)

这时候我们想验证一下文件内容是否一致，有一个魔法命令`%pycat`，能够以弹出框的形式显示外部文件的内容。这样我们就不必离开 Jupyter Notebook 去查看`.py`文件了。

![](https://mmbiz.qpic.cn/mmbiz_png/tXYict40xfLhabrCZtDHLnCA8GXyeXdtCMYPepcbCFKeY8w92gqrkClCFKqKFhzabb954CGVJxv8OYO2PTxfrBA/640?wx_fmt=png)

设置环境变量
------

在机器学习 / 深度学习里，我们经常遇到使用环境变量的情况 [1]。

在 Jupyter Notebook 中，也是可以通过 % 魔术命令进行设置修改环境变量的。

例如`%env`或`%set_env`，用法也很简单`%env MY_VAR=MY_VALUE`或`%env MY_VAR MY_VALUE`。

另外，单独使用`%env`可以打印出当前的环境变量。

```
%env THIS_IS_ENV_EXAMPLE "TEST2021"
```

其中`THIS_IS_ENV_EXAMPLE`是环境变量，它的值是字串 `TEST2021`

![](https://mmbiz.qpic.cn/mmbiz_png/tXYict40xfLhabrCZtDHLnCA8GXyeXdtCqWiaS76CStgLojgo2qeqiaZkHZTRGD1fYZmj99UXQFV6PILLVPISXicibA/640?wx_fmt=png)

代码分享
----

举个例子，我们正在 Jupyter Notebook 运行代码，遇到了一些问题想把代码分享出去。这时候要截图，还是另存为代码文件再发送文件呢？

其实还有另外一种选择——以**链接**的形式分享代码。

```
%pastebin
```

该魔法命令可以将代码上传到 Pastebin 并返回一个链接。其中`Pastebin`是一个线上内容托管服务，我们可以在上面存储纯文本，如源代码片段，所形成的链接也可以分享给他人 [2]。

我们还可以指定分享的 cell（单元格），比如：

```
%pastebin 1-4
```

这样就生成了一个隐私的链接 url 供我们分享代码。

![](https://mmbiz.qpic.cn/mmbiz_png/tXYict40xfLhabrCZtDHLnCA8GXyeXdtCUxdqJxn6JVQGtlPpl85gC9kwMaMqlRxRIN8aiaj8uJiaaEEYSws4ia9ZA/640?wx_fmt=png)

网页打开链接，即可得到下图的界面。需要注意，生成的链接有效期只有 7 天。

![](https://mmbiz.qpic.cn/mmbiz_png/tXYict40xfLhabrCZtDHLnCA8GXyeXdtCmt0svBTGBNibdNv8KYqUHGek9RXs6naEnguK38GULREzIueib6CGa2Vw/640?wx_fmt=png)

> 哔哔叨：这种分享方式还是需要联网的。有一些小伙伴在公司使用的是不能联网的机器，想请教别人分享代码一直是个难题。最近小五发现有人开发了通过二维码传递代码本文的方法，很有趣，有时间好好了解一下。

好了，今天介绍了`Jupyter Notebook`中的魔法命令，利用它可以轻松实现一些纯 Python 要很麻烦才能实现的功能。

限于篇幅，只介绍了 5 个有趣的魔法命令，`%pdb`代码调试、运行 py 文件等都没有细讲，大家想详细了解的话可以对照前文的帮助文档查看。下篇将会继续介绍一波非常实用的**插件**，敬请期待。

觉得有技巧帮助到你提升效率 / 乐趣的话，别忘了给本文点个赞哦👍

### 参考资料

[1]

迷途小书童的 Note: https://mp.weixin.qq.com/s/sB9MKerqKnwY8aZScOROCg

[2]

jupyter notebook python 很慢: https://blog.csdn.net/weixin_39562089/article/details/110911409
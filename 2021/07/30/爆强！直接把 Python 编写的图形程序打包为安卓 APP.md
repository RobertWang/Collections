> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652579150&idx=1&sn=248b79ab70bfaa24e45bbacf6fb14c70&chksm=84650d04b31284122b5eef5427bf035349c43ca2633e56b017a3d11982fdd63443dffeac9412&mpshare=1&scene=1&srcid=07272C6yzMznNBsSPocfXXd8&sharer_sharetime=1627392830838&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

不过**这些框架都是只能创建桌面图形界面程序**，比如 Windows、Linux 和 macOS。

据了解，**Kivy 和 BeeWare 都宣称「一次编写，处处部署」，基于这些个框架编写的图形界面程序，都能够打包为全平台的应用程序，比如：Windows、Linux、macOS、Android、IOS。**

今天，咱们就尝试使用一下 BeeWare 这个框架，编写一个图形界面程序，然后打包为一个安卓 APP。

开始吧！

BeeWare 是一个基于 Python 构建的跨平台应用开发框架，其宣传「Write once. Deploy everywhere.」

**能够让 Python 编写的图形程序在 iOS, Android, Windows, MacOS, Linux, Web, 和 tvOS 上运行，看起来是很强大的。**

安装
--

根据 BeeWare 的文档说明，在 Windows 上使用，我们首先需要安装 Git 和 WiX Toolset，根据给出的网址，下载安装即可。

然后，我们使用 pip 工具安装 BeeWare：

```
pip install briefcase
```

创建应用
----

BeeWare 安装完成之后，我们就可以通过`briefcase`命令在命令行终端进行 BeeWare 应用的管理，比如新建、运行、构建、打包等等。

我们先使用命令`briefcase new`创建一个应用。

命令输入之后，会让我们输入「应用的正式名称」、「应用程序名称」、「域名」、「项目名称」等等信息，在这里出于演示，我们统统使用默认值。

![](https://mmbiz.qpic.cn/mmbiz_png/AnkmC6oCTPD65TnvDic3Iv6v4REic0qVgdjUSzUdficsr9Q08xz5VXgLHrO2pV77m0exQXxTMjLbfrt88K4Tfx1Dg/640?wx_fmt=png)

输入完成之后，BeeWare 会开始创建应用，创建完成之后，会有如下提示：

![](https://mmbiz.qpic.cn/mmbiz_png/AnkmC6oCTPD65TnvDic3Iv6v4REic0qVgdE46P4xyPiaUoEmql5Ld0PQn2tH7gapPfxkKpia4ts9x2h9OeJsdnYI3A/640?wx_fmt=png)

同时目录下多出了一个与应用程序名称同名的目录：

![](https://mmbiz.qpic.cn/mmbiz_png/AnkmC6oCTPD65TnvDic3Iv6v4REic0qVgdOFWXBRgU1avwIhEWvXJ1bBrxjVFFZrVxMtiafylmGzhibjuytgEX3oKw/640?wx_fmt=png)

我们的程序的主要代码都将在 app.py 里面编写，默认 app.py 文件内已经有一个 demo 代码，我们可以直接运行项目：

```
briefcase dev
```

在命令行输入上述命令，会生成一个如下图所示的窗口：

![](https://mmbiz.qpic.cn/mmbiz_png/AnkmC6oCTPD65TnvDic3Iv6v4REic0qVgdWkpRlgufRhqMAW7tSUtehcwQtiawHic1J4iauicicSY3S6ypmBabuFAo72Q/640?wx_fmt=png)

打包为 Windows 程序
--------------

出于演示，在这里不对 BeeWare 的图形界面控件进行过多的演示，直接使用了它的 demo。

如果我们需要将编写好的图形程序打包成 Windows 桌面程序，那么可以执行下面的步骤：

首先，创建应用的脚手架：

```
briefcase create
```

运行命令，将会生成一些预配置文件，然后下载依赖的包。

完成之后，项目目录下会生成一个 Windows 的目录，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AnkmC6oCTPD65TnvDic3Iv6v4REic0qVgd7hFXQUAJw9icUvuQd8Igq5oibib8abqL2f3xIxMszFLLbGKRtiaw5iat4UQ/640?wx_fmt=png)

然后，构建应用：

```
briefcase build
```

接着，运行构建的应用：

```
briefcase run
```

最后，打包应用：

```
briefcase package
```

打包完成后，./Windows 目录下会生成一个 .msi 的二进制安装文件：

![](https://mmbiz.qpic.cn/mmbiz_png/AnkmC6oCTPD65TnvDic3Iv6v4REic0qVgdumvaWtbqHtSibwTXy8NpDR77OiaHjXzCic0n09Q4563EMqofuEia84sZtA/640?wx_fmt=png)

我们双击运行它，会出现常见的 Windows 程序的安装界面：

![](https://mmbiz.qpic.cn/mmbiz_png/AnkmC6oCTPD65TnvDic3Iv6v4REic0qVgdyDWJBEQwhIZk6Ns60uytr1jiby3jPgSM6m3437IV5RT95UM1g9MUiaPA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/AnkmC6oCTPD65TnvDic3Iv6v4REic0qVgd6HuRgFFOFQ96SFwgd4NI3bVriaV76TSia9Qo6QquaMacuhGrBrBNRWDw/640?wx_fmt=png)

安装完成之后，可以在 Windows 的应用程序列表中看到它：

![](https://mmbiz.qpic.cn/mmbiz_png/AnkmC6oCTPD65TnvDic3Iv6v4REic0qVgd3v8I8Qr8WLVYf40XBXnVBEgAUXjJgO4anc6IphgfFHnaEicoRJ7iadKA/640?wx_fmt=png)

点击它，就会打开我们之前用命令运行的程序界面；

![](https://mmbiz.qpic.cn/mmbiz_png/AnkmC6oCTPD65TnvDic3Iv6v4REic0qVgdS3qBTVymO09hmHOsrCKhuLNhoYdLkrbNAUEVxnI8EhwHMxWFAYDhicA/640?wx_fmt=png)

打包为安卓 APP
---------

如果我们要将应用打包为安卓 APP，过程也是类似的。

首先，创建应用的安卓脚手架：

```
briefcase create android
```

接着，构建安卓应用：

```
briefcase build android
```

![](https://mmbiz.qpic.cn/mmbiz_png/AnkmC6oCTPD65TnvDic3Iv6v4REic0qVgd0ia2K2ZyYlIwAFwuYTkneicdTRv6M7Heibh0ciatcuZQ8rEa7wRpuzibwQg/640?wx_fmt=png)

然后，我们运行一下构建好的安卓应用：

```
briefcase run android
```

在这里会让我们选择设备，可以选择 BeeWare 提供的安卓虚拟机或者是在电脑上连接自己的手机，在这里，我们选择安卓虚拟机：

![](https://mmbiz.qpic.cn/mmbiz_png/AnkmC6oCTPD65TnvDic3Iv6v4REic0qVgdzlwtoBqx9iatOlW39GBIN3ibZuicpZ0G1MWffUK63T4MnhotH5V8U4OAQ/640?wx_fmt=png)

最后，打包安卓应用：

```
briefcase package android
```

![](https://mmbiz.qpic.cn/mmbiz_png/AnkmC6oCTPD65TnvDic3Iv6v4REic0qVgd1JLeoSoicQ3Eb0XvDWF7FnHZB3PibJ6Cg5MtSgn8ia0RibEqkkFwPMgLJw/640?wx_fmt=png)

打包完成之后，我们可以在 .\android\gradle\Hello World\app\build\outputs 找到打包好的文件：

![](https://mmbiz.qpic.cn/mmbiz_png/AnkmC6oCTPD65TnvDic3Iv6v4REic0qVgdsLibUXXDkMSVPjahbBpmJd2ia8UuFVstUOtnOmpQIYJDdMQHnGMZrQDg/640?wx_fmt=png)

BeeWare 提供了两种打包好的文件，一种是用于上架 Google Play 的. aab 格式文件，

![](https://mmbiz.qpic.cn/mmbiz_png/AnkmC6oCTPD65TnvDic3Iv6v4REic0qVgd8bePyylBQr2npSicFJS7vxrcgiaWzricot99TqefHzfWygCZ4vDfEIFkA/640?wx_fmt=png)

一种是用于调试的 .apk 文件:

![](https://mmbiz.qpic.cn/mmbiz_png/AnkmC6oCTPD65TnvDic3Iv6v4REic0qVgdGFhXqpxyalxdtSM3ly8UHSYeDceMI6cYEPUiaQvPTyZVzs6Z3bFJ21g/640?wx_fmt=png)

apk 文件咱们的手机可以直接安装，所以就用 QQ 把它传到手机上：

![](https://mmbiz.qpic.cn/mmbiz_png/AnkmC6oCTPD65TnvDic3Iv6v4REic0qVgdicgbZbhMXxMmMh3SU0jEtCI8kZJvGzKaNQIPqCSJMd8TYYT66cLmRKQ/640?wx_fmt=png)

QQ 内可以识别安装：

![](https://mmbiz.qpic.cn/mmbiz_jpg/AnkmC6oCTPD65TnvDic3Iv6v4REic0qVgdx9CjTrqRpnPXvtEzp6YqcubZ7mK3C01ib8DZ1YtUDLNS1WtKBvgoyUw/640?wx_fmt=jpeg)

安装它：

![](https://mmbiz.qpic.cn/mmbiz_jpg/AnkmC6oCTPD65TnvDic3Iv6v4REic0qVgdLyAwBqicjd4OOgYD45dyZljUibp35mQXt8NG5SkGs35JINzRRZgPK3KQ/640?wx_fmt=jpeg)

安装完成：

![](https://mmbiz.qpic.cn/mmbiz_jpg/AnkmC6oCTPD65TnvDic3Iv6v4REic0qVgdmXwMI6ZldibAM9duYK3Erib2a6Z2JXH39Lia2NpZWZNJYjIYYx9l3nmXA/640?wx_fmt=jpeg)

打开应用：

![](https://mmbiz.qpic.cn/mmbiz_jpg/AnkmC6oCTPD65TnvDic3Iv6v4REic0qVgdiaCE03ibpA44ziaCfvN8gDib3pIwttYESQplkKANn6o3pL5KhC1stqXAww/640?wx_fmt=jpeg)

显示程序内容：

![](https://mmbiz.qpic.cn/mmbiz_jpg/AnkmC6oCTPD65TnvDic3Iv6v4REic0qVgdmYVA4ozIVxHpmYSmsFHzoicbY2dTPotYXwxia0iacfSYwetV1qUicaVgtA/640?wx_fmt=jpeg)

这样，我们就把 Python 编写的图形程序直接打包为了安卓 APP。

IOS 的打包流程也是类似，大家可以参考官网文档尝试一下。

有问题欢迎留言交流讨论~

推荐阅读  点击标题可跳转

1、[作为一只爬虫，如何科学有效地处理短信验证码？](http://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652579011&idx=1&sn=472e67473ecba8df2a9d6519b4f6eb95&chksm=84650c89b312859f3c8312581d1e9377ccdb35d05f7ae545e97ed02c885b785b6db71a5ec641&scene=21#wechat_redirect)

2、[有了这款神器，轻松用 Python 写个 APP](http://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652578957&idx=1&sn=c9184ae7f4b3c1ef10593bf95e0a9418&chksm=84650cc7b31285d146817c6195c29e9886b413387ee21854d14796fcd83d1502a9c2a39bd5e2&scene=21#wechat_redirect)

3、[装上后这 14 个插件后，PyCharm 能飞起](http://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652578916&idx=1&sn=34db92b570aa53a2e1af50a50022dd93&chksm=84650c2eb31285387e08e767bad102899070c81864842929a7baab5a4919c3dd8b9aa19dc114&scene=21#wechat_redirect)

推荐关注「Python 开发者」，提升 Python 技能

公众号
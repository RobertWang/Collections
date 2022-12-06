> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzkzNzIzNzczMA==&mid=2247490777&idx=1&sn=7bae711a9365b1f45b3854e2e55c0c95&chksm=c293d154f5e4584214cf6ffe7c213385b967a89e87fcfbb4d852de97172b16408d58eb394c5a&mpshare=1&scene=1&srcid=1026N5sPDoQGciJCZ4O8pYPj&sharer_sharetime=1666786564089&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

> 不背电脑，也能随时随地解决问题...

程序员平常虽说放假，但也都是 24 小时随时待命，电脑常年不离身，过年放假也一样，走亲访友，都带在身边，一旦有任何风吹草动，以便随时顶上；在一天走亲戚的时候，突然要紧急处理点事情；而今年过年，天气异常的冷，好巧不巧的是，带在身边电脑 SSD 不工作了（当时以为坏了），导致开不了机！由于没电脑，但又必须得处理，没办法只能通过手机，使用网页版的 **Tabby**（https://app.tabby.sh/）远程服务器，处理问题，虽然说操作起来有些不太方便，处理起来耗时长了些，但还是完美的解决了问题。

今天再次推荐一下这款神器...

Tabby
-----

Tabby 是一名老外在 Github 开源的终端连接的工具，至今已经累积 36K+ star。

![](https://mmbiz.qpic.cn/mmbiz_png/GjuWRiaNxhnRt8X2JiajKXIv1xmCIcmIcf9Ysp43FCErGhZbwx53rEVU2ajbOs1C2JbP5fYAMoMxJOOQTichq48lQ/640?wx_fmt=png)

Tabby 的功能特性大概有：

1.  支持多平台，Windows、MacOS（Intel 芯片 / M1 芯片）、Linux 都有对应的安装包的；
    
2.  自带 SFTP 功能，能够与 Linux 系统传输文件；
    
3.  炫酷的终端页面，简单易用，以及各种插件支持等
    

安装包
---

![](https://mmbiz.qpic.cn/mmbiz_png/GjuWRiaNxhnRt8X2JiajKXIv1xmCIcmIcfzct6Fpv2NIROhC5kk1CHKBY20bpiaq1MzbyB2RBpQtrB5ic9xp18I9Ug/640?wx_fmt=png)

github 地址：https://github.com/Eugeny/tabby/releases

找到适合自己电脑的安装版本

安装之后是这个页面

![](https://mmbiz.qpic.cn/mmbiz_png/GjuWRiaNxhnRt8X2JiajKXIv1xmCIcmIcfHFrevmGUSkBVfEwLAmzkw8KOGmKpPJCgJMGktJ2hoRibXksZWnqOU9A/640?wx_fmt=png)

SSH 连接
------

**一开始我以为点击「New terminal」是弹出填写连接服务器的信息。**

结果不是，它默认是新建一个针对本地电脑的终端窗口，比如如果你的电脑是 windows 系统就会新建一个 cmd 控制窗口，如果是 macOS 系统就会新建一个 terminal。

所以，要想新建一个连接服务器的终端，要点击「Settings」，进入到配置页面。

![](https://mmbiz.qpic.cn/mmbiz_png/GjuWRiaNxhnRt8X2JiajKXIv1xmCIcmIcfISqpXbIBqrTEA9W1gvaRwrpNbV4VqbbGG5Ic0lJR3glMhXjdcNcuOg/640?wx_fmt=png)

进入到设置页面后，选择 profiles&connections 这个选项，然后点击「New profile」新建一个终端配置

![](https://mmbiz.qpic.cn/mmbiz_png/GjuWRiaNxhnRt8X2JiajKXIv1xmCIcmIcfUcrEj0PoPINXaUPtmibzw3hvQhltQUuribLYLJic0otrQyia6ooMwgNKuQ/640?wx_fmt=png)

然后选择 ssh connection。

![](https://mmbiz.qpic.cn/mmbiz_png/GjuWRiaNxhnRt8X2JiajKXIv1xmCIcmIcfyl5JKVpnyDBw7jwa87Th4KslicCHyW9E169eXrV7vfKYftubcMKkGng/640?wx_fmt=png)

随后就会弹出配置 ssh 连接的信息，填上终端名称、IP 地址、端口号、账号密码就可以了。

![](https://mmbiz.qpic.cn/mmbiz_png/GjuWRiaNxhnRt8X2JiajKXIv1xmCIcmIcfIIk003ic7zUMA2EKNeoBONZI78RHvVNoErcbzWbjfHlE4TWNTDR2gkQ/640?wx_fmt=png)

保存完后，就会出现刚新增的终端配置，然后点击运行的图标就可以了。

![](https://mmbiz.qpic.cn/mmbiz_png/GjuWRiaNxhnRt8X2JiajKXIv1xmCIcmIcfb3a1mYD8Wej4ZFTcwDrIbicAJThMETehbgMyxTseKB4VWTFen3ibR0mQ/640?wx_fmt=png)

也可以通过图中的小方块， 选择连接的服务器。

![](https://mmbiz.qpic.cn/mmbiz_png/GjuWRiaNxhnRt8X2JiajKXIv1xmCIcmIcfssd1xsIouatK5jKw2sZxEjicPmJrzb2PTyhomNbeDgVNxAcSiagkmPuA/640?wx_fmt=png)

选择后，就会进入到终端页面了，也就可以对服务器进行操作了。

![](https://mmbiz.qpic.cn/mmbiz_png/GjuWRiaNxhnRt8X2JiajKXIv1xmCIcmIcfribvllPh8Hyn2FDSye4pRibfmrSwBCCbfAfNvibibJ4n4SqLvFQSyVlFfw/640?wx_fmt=png)

SFTP 传输工具
---------

前面也介绍过，这款终端工具是自带 SFTP 功能的。要使用的话，直接点击下图中的 SFTP 图标就行。

![](https://mmbiz.qpic.cn/mmbiz_png/GjuWRiaNxhnRt8X2JiajKXIv1xmCIcmIcfxgbVdyiakp94gbT59aictric1UK4oh0RaaP4lUlBZs7lVxCsjMCBUg3KQ/640?wx_fmt=png)

然后就会弹出服务器上的目录

![](https://mmbiz.qpic.cn/mmbiz_png/GjuWRiaNxhnRt8X2JiajKXIv1xmCIcmIcfiaXRuaicNBtjNwk3Y6I5nWDZAak8J6ht03DtJm9JL1RY6VW3OkHPImgw/640?wx_fmt=png)

如果你想把服务器上的文件传输到本地电脑，你只需要找到服务器的文件，然后点击，就会弹出保存文件的提示。

![](https://mmbiz.qpic.cn/mmbiz_png/GjuWRiaNxhnRt8X2JiajKXIv1xmCIcmIcfF4ayvCQ7WH04gt8BPZ7LiagtI9kJ4OaFsRhib5qLa2zvmAic9mmu2c0zQ/640?wx_fmt=png)

如果你想把本地电脑的文件放到服务器上，只需要把文件拖拽到对应的目录就行。或者点击右上角上传文件

![](https://mmbiz.qpic.cn/mmbiz_png/GjuWRiaNxhnRt8X2JiajKXIv1xmCIcmIcfB5OduSFaWayLiatNsiaic0kSF0dOVXNWGiaP5vhLXicmp3BaNbaAJxylrcA/640?wx_fmt=png)

设置
--

Tabby 提供很多终端页面风格，都挺好看的。

![](https://mmbiz.qpic.cn/mmbiz_png/GjuWRiaNxhnRt8X2JiajKXIv1xmCIcmIcf1hY3jw0jBicxSFjso6lm7z6p9giaoGvZprVJoe0OQSyIskFYKModXAZw/640?wx_fmt=png)

还有字体的大小设置等

![](https://mmbiz.qpic.cn/mmbiz_png/GjuWRiaNxhnRt8X2JiajKXIv1xmCIcmIcfbVDciaF5g7VcJia5TgcnibGjBweEicUqlBibUCDnj8DxzN91x2a9RBYXhUQ/640?wx_fmt=png)

以及常用的快捷键

![](https://mmbiz.qpic.cn/mmbiz_png/GjuWRiaNxhnRt8X2JiajKXIv1xmCIcmIcfUsM98DTBsqy5zrzVVe7licB4Nt9fGzgvrVRZVvmiazKFzpjO8LZ2UqXg/640?wx_fmt=png)

网页版
---

Tabby 网页版的入口：https://app.tabby.sh/

使用方式和 PC 工具没有大的差异，因为有网页工具，只要有网络的地方，就能随时随地处理问题。

详细细节，可参考 github 官方仓库：https://github.com/Eugeny/tabby
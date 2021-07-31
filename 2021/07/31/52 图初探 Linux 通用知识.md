> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666555692&idx=2&sn=42ffc8ae8e13e6b3bbd322732d042813&chksm=80dcad87b7ab2491bc472fa5db1451769fa4ced812e05dd4f9fa3ead79996680ea4506bf36f6&scene=21#wechat_redirect)

> 这篇文章是 Linux 的超级基础且经常用到的内容，不多说，直接肝！

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEZ5JwMADzAZBJDr96jQYKHT49ZutA737ZKNCuvxKkVwHhL3ibJfJkFicw/640?wx_fmt=png)

Linux 软件安装

* * *

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEr7M5ibOnH7SAzLbprMicCsz1fX8xESoYzVLow8456J4pwoSRfueHl25g/640?wx_fmt=png)

Linux 排查问题套路

* * *

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEdOibI4fAffOmeZ7bRbC999Ybe16OXVia4K5p8qxCXbNFBMcdrYd5qA8A/640?wx_fmt=png)

Linux 命令详解

一 Linux 通用知识
============

> 说到操作系统，如果读大学的时候是计算机专业，那肯定就会上这门课，我猜测当时的你们想法是这样的

*   上大学使用的都是 Windows 系统，界面友好，上手快，习惯性的点点点操作
    
*   大部分的课程在 windows 中操作，比如 C++ 用的 Vistual Studio，学数据库的 SQL Server
    
*   大学中的操作系统更加偏向理论研究，至于到底是怎么运作的可能懵懵懂懂
    

知道上了研究生到了实验室，我发现实验室的怎么都是对着一个窗口操作，瞬间觉得以前的计算机知识白学了，于是开启了 Linux 之路。

其实大部分的系统，团购，打车，快递都部署在服务端，其中都包含 Linux，什么云计算，虚拟化，大数据等也是基于 Linux，那为啥在大学里都是 windows？

![](https://mmbiz.qpic.cn/mmbiz_gif/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEzKgsrvWezIH7rg4U9KynIYafCJRWiaCzibcXMwzzTmUibYZ00dyicQsgkA/640?wx_fmt=gif)

咦  

> 为什么说了解 Linux 的生态，会让你学到更多的新技术?

我们要知道很多的大牛通过 Linux 来开发各种如那件，数据库 Mysql，kafka，Spark 等技术都会默认提供 Linux 的安装运维手册，所以尽快的进入 Linux 的世界对于个人的进步和职业发展都是非常有好处的

每当我们买了手机，买了电脑，上手就可以用，这是因为预装了操作系统。所以呀，那有什么岁月静好，知识有人帮我们负重前行了，操作系统就是这样一个角色。

> 那么操作系统帮助我们做了哪些事儿呢？

*   跑几个问题，桌面上的图标是什么，为啥子敲一下键盘就出来了画面
    
*   电脑咋个知道我们鼠标点击的那个位置
    
*   为什么我一回车，这些字符就飞出去了
    

这几个任何一个操作，基本上都覆盖了操作系统的所有功能，那我来认识熟悉而默认的操作系统

1 vmvare
--------

> 虚拟机是什么？

虚拟机通过软件来模拟具有完整硬件系统功能的，运行在完全隔离的完整计算机系统。每个虚拟计算机可以独立运行并安装各种软件和应用

*   首先从官方下载并解压虚拟机安装包，然后双击运行
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEq13WzGic61pDSLQ1SVLBENGZiaburZy8ficyPjOOpf5OxpsC4PsF32NPw/640?wx_fmt=png)

双击 VMVARE

*   下一步
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEMlX7GvP0h56Koc5ZwubibDJlTicQT3VEJC4xtc9JHibicPeC8TFN1zMmbQ/640?wx_fmt=png)

接受许可进行下一步

*   选择安装位置，最好不要出现中文
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MENcRqaywXjOsKEtdosibBlTOiaibgpkcAm0fg9o09IcYicpZicbOhTS44MlQ/640?wx_fmt=png)

自定义路径

*   设置用户体验选项，都可以选择
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6ME3kp9pib7RaDFTEtuSzMs7hiaKmHKd1WciceEfEVfUicWOy6QVYk2zd3ejg/640?wx_fmt=png)

设置用户体验

*   在桌面和开始菜单程序文件夹创建快捷方式。
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MERlsOM6icicAicGC3Gq49WibyENRhRibNJhUX2SyCL8G3r1JYDgyULDMicAxw/640?wx_fmt=png)

创建快捷方式

*   百度一个许可证 ZG1WH-ATY96-H80QP-X7PEX-Y30V4
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6ME6zNAHD0tZFasMWSggL38IcPNaWJst17xQMh3wSgRNn0AnfG1lHEBBg/640?wx_fmt=png)

输入许可证密钥

*   打开 vmvare
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MERsef2sOCiasCqPWPIDCJYibF2SVTkTia4J2E8icYBPHPY3kdA1x95UTNKQ/640?wx_fmt=png)

打开 vmvare

*   点击新建虚拟机向导 选择文件 - 新建虚拟机打开
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEH8UTLR0sIwtqssrL2hDJbEbHYq2RHU3Oia2F2oUicqnbL0rajsRwMBhg/640?wx_fmt=png)

新建虚拟机

*   选择自定义 下一步
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6ME1SbClmiaiag4CA0bPcnUxbofEohzCbSBOATrqO26NomWVoicFibrkj8dgg/640?wx_fmt=png)

选择自定义

*   下一步
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEZhibuQ8Jxfa6oaU9uAQoYdnVNmS3A2S9CMQvpVdwsuE5qL9GvwziaJIQ/640?wx_fmt=png)

选择下一步

*   安装客户机操作系统，选择稍后安装操作系统
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEbbBOCwicI6N8Q8Bm4wYUyLSNPH7Y2iaycwU2nt25a68YhQ4mggb91VpA/640?wx_fmt=png)

选择稍后安装操作系统

*   命名虚拟机 更改虚拟机名称并选择安装得位置
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEBMGke1wU7ibNn0jn1eZOibLOhdD4Gic2hcUN4dKebnJYWVttSaQ2eokHw/640?wx_fmt=png)

命名虚拟机

*   更改主机配置进行处理的分配
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEboB9liaANwiaUVeAwpEbQibhkwoeVQKHC98dJsLMia4RiaRiaxY51egazKtA/640?wx_fmt=png)

处理器核心数分配

*   虚拟内存分配：注意内存分配不能大于主机内存
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6ME3BkoWUkpxSQhhrRiavAX29OQ7YaLiaoqicy3vB40qWfXNiaXiaAZXHjiclmg/640?wx_fmt=png)
    
    虚拟内存分配
    
*   设置虚拟机网络得类型，这里选择 NAT
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEzrvgibLm6TFFZT3qXoenbWzsqicD7VsBxAd9MFJPo2icc3R8ibhHA8eNrw/640?wx_fmt=png)

网络类型暂设为 NAT

*   IO 控制器选择，选择 LSILogic
    
*   磁盘类型选择 SCSI 即可
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6ME3KG4ibKMbJCadZmYkkDV6YYUXXoJcv3gLsSM3CY425f1PSuywKuicKYQ/640?wx_fmt=png)

*   创建磁盘选择创建新虚拟磁盘
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEAKw66kib3rgCfeld8iabI67k5U2AibEb1GCMHUlqrTup1dA4QIGCXF62Q/640?wx_fmt=png)

创建新虚拟磁盘

*   指定磁盘文件
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEiaQSKEq4WKgrhP8X7xd8bPLqIvK40NNPFAbZW3vYS1wWBodkstSmxjw/640?wx_fmt=png)

指定磁盘文件

*   修改路径
    
*   选择自定义硬件
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEjkicVhPoleqNJqrFgibS5CGd3bCODTIzhEbaiaIpG0qsXGicbnu7g5W7QA/640?wx_fmt=png)

选择自定义硬件

*   选择 centos 得 ISO 镜像文件，先选择 CDDVN---ISO 镜像文件 --- 浏览找到镜像、
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEdXUWC2mlmtT0GhcBa6pqfDiau2r22iaiavrX8dfARZRfefABd9PZ8ay0A/640?wx_fmt=png)

导入镜像

*   点击完成
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEAGiaEPlFEByrvuIUHD1YUkicCmxEE09caAKqMxj7pm5QPTvVkZ9WffdA/640?wx_fmt=png)

完成

*   开启虚拟机 选择配置好的虚拟机
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEFBAiaxa9gNuCYfT6qHCIhZP0JT0cb1TjkU2cjKwU0HFWp3Rcd05u46w/640?wx_fmt=png)

开启虚拟机

*   鼠标移动到虚拟机内部，上下键选择 install centos7 然后回车
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MELqtAZduHu1wSsmNGBQNqClmsfyr7NsN2XABDTnTzr2u5eEob6zjOTg/640?wx_fmt=png)

install centos7

*   选择软件选择最小安装，选择语言
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEWjVtK0icbbjUB6030SWsKpbfibRGmRz4TRD2ngYj9AnNA17dCHG6ShDQ/640?wx_fmt=png)
    
    选择最小化安装
    
*   软件安装
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEV7icFIBGFicap8AX4W35V7kD08sgTia9ve5T4pkLQRNiarJDyJXYh3Ygnw/640?wx_fmt=png)

软件安装

*   选择计算节点
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEJk4XEEnMNL9c0Wud0JPZZMDdGHhyxwxV7sibZPFiaFVWRhhEglwmfO0w/640?wx_fmt=png)

选择计算节点

*   开始安装
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEDdgx7TZcO1HEQSWcruw44jsNCqrOLa1eFqxnToEicJiboGI1Gw1SxXVQ/640?wx_fmt=png)

开始安装

*   设置 root 密码，点击完成配置
    
*   ![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEwbe3j36vlApubKveuy9qVtHhicwTzSQM5cTp9MlmHACYvopnuzPHiatg/640?wx_fmt=png)
    
    设置 root 密码
    

2 进行网络配置
--------

> 现在我们的 centos 还是个空壳子，如果我们需要访问外网，则需要进一步配置一波

*   打开配置文件
    

```
#vi /etc/sysconfig/network-scripts/ifcfg-eth0
```

*   更改相应的配置
    

```
DEVICE=eth0 #设备名称，可根据ifcofnig命令查看到。
BOOTPROTO=dhcp #连接方式，dhcp会自动分配地址，此时不需要在下面设置ip和网关
HWADDR=00:0C:29:AD:66:9F #硬件地址，可根据ifcofnig命令查看到。
ONBOOT=yes #yes表示启动就执行该配置，需要改为yes
```

*   service restart network 完事 ping www.baidu.com
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEc96nsicjrOJJsHHicicr6f8JK63tEyau3qSFMzDW8hKT5rYoG0DdicTic4A/640?wx_fmt=png)

网络检测

3 安装 xshell
-----------

> 我们已经完成了安装 vmvare 并导入了 centos，那么我们如何去玩儿这个看似很牛皮的玩意？直接上手？不习惯吧，那我们用个远程工具连连

Xshell 是一个强大的安全终端模拟软件，Xshell 可以在 Windows 界面下用来访问远端不同系统下的服务器，从而比较好的达到远程控制终端的目的。除此之外，其还有丰富的外观配色方案以及样式选择。

*   下载 xshell(别去下了，贼慢麻烦)
    
*   链接测试 (因为使用的 ssh，那么确保 centos 中 22 端口已经打开了)
    
*   文件 ----- 属性进行 XHSELL 相关的配置，比如配色，字体大小等
    

4 基本命令的使用
---------

> 命令太多，必须要全部记忆，但是要学会如何查每个命令的参数。我画了个思维导图可以当作小字典查看，下面列出可能我们使用频率会更高的命令

<table><thead><tr data-darkmode-bgcolor-16277440621604="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(255,255,255)" data-style="font-size: inherit; color: inherit; line-height: inherit; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th width="228" data-darkmode-bgcolor-16277440621604="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="padding: 0.5em 1em; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">执行命令</th><th width="266" data-darkmode-bgcolor-16277440621604="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="padding: 0.5em 1em; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">含义</th></tr></thead><tbody><tr data-darkmode-bgcolor-16277440621604="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(255,255,255)" data-style="font-size: inherit; color: inherit; line-height: inherit; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td width="228" data-darkmode-bgcolor-16277440621604="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(255,255,255)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">cd ~</td><td width="266" data-darkmode-bgcolor-16277440621604="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(255,255,255)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">切换到登录用户的主目录即 / home / 用户名</td></tr><tr data-darkmode-bgcolor-16277440621604="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(248, 248, 248)" data-style="font-size: inherit; color: inherit; line-height: inherit; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td width="228" data-darkmode-bgcolor-16277440621604="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(248, 248, 248)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">cd /</td><td width="266" data-darkmode-bgcolor-16277440621604="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(248, 248, 248)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">进入根目录</td></tr><tr data-darkmode-bgcolor-16277440621604="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(255,255,255)" data-style="font-size: inherit; color: inherit; line-height: inherit; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td width="228" data-darkmode-bgcolor-16277440621604="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(255,255,255)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">cd /home/lj</td><td width="266" data-darkmode-bgcolor-16277440621604="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(255,255,255)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">将 / home/LJ 作为当前的目录</td></tr><tr data-darkmode-bgcolor-16277440621604="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(248, 248, 248)" data-style="font-size: inherit; color: inherit; line-height: inherit; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td width="228" data-darkmode-bgcolor-16277440621604="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(248, 248, 248)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">cd ..</td><td width="266" data-darkmode-bgcolor-16277440621604="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(248, 248, 248)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">返回到上一层目录</td></tr><tr data-darkmode-bgcolor-16277440621604="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(255,255,255)" data-style="font-size: inherit; color: inherit; line-height: inherit; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td width="228" data-darkmode-bgcolor-16277440621604="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(255,255,255)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">cd -</td><td width="266" data-darkmode-bgcolor-16277440621604="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(255,255,255)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">回到上次所在的目录</td></tr><tr data-darkmode-bgcolor-16277440621604="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(248, 248, 248)" data-style="font-size: inherit; color: inherit; line-height: inherit; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td width="228" data-darkmode-bgcolor-16277440621604="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(248, 248, 248)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">cd ../../</td><td width="266" data-darkmode-bgcolor-16277440621604="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(248, 248, 248)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">去上上层目录</td></tr><tr data-darkmode-bgcolor-16277440621604="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(255,255,255)" data-style="font-size: inherit; color: inherit; line-height: inherit; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td width="228" data-darkmode-bgcolor-16277440621604="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(255,255,255)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">ls</td><td width="266" data-darkmode-bgcolor-16277440621604="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(255,255,255)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">查看当前目录</td></tr><tr data-darkmode-bgcolor-16277440621604="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(248, 248, 248)" data-style="font-size: inherit; color: inherit; line-height: inherit; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td width="228" data-darkmode-bgcolor-16277440621604="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(248, 248, 248)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">ls -la</td><td width="266" data-darkmode-bgcolor-16277440621604="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(248, 248, 248)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">查看当前目录的文件信息 包含了隐藏文件</td></tr><tr data-darkmode-bgcolor-16277440621604="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(255,255,255)" data-style="font-size: inherit; color: inherit; line-height: inherit; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td width="228" data-darkmode-bgcolor-16277440621604="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(255,255,255)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">pwd</td><td width="266" data-darkmode-bgcolor-16277440621604="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(255,255,255)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">查看当前目录的绝对路径</td></tr><tr data-darkmode-bgcolor-16277440621604="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(248, 248, 248)" data-style="font-size: inherit; color: inherit; line-height: inherit; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td width="258" data-darkmode-bgcolor-16277440621604="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(248, 248, 248)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">cp / 目录 / 1.txt / 目录 /</td><td width="266" data-darkmode-bgcolor-16277440621604="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(248, 248, 248)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">复制</td></tr><tr data-darkmode-bgcolor-16277440621604="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(255,255,255)" data-style="font-size: inherit; color: inherit; line-height: inherit; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td width="228" data-darkmode-bgcolor-16277440621604="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(255,255,255)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">rm</td><td width="266" data-darkmode-bgcolor-16277440621604="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(255,255,255)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">删除</td></tr><tr data-darkmode-bgcolor-16277440621604="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(248, 248, 248)" data-style="font-size: inherit; color: inherit; line-height: inherit; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td width="228" data-darkmode-bgcolor-16277440621604="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(248, 248, 248)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">q！</td><td width="266" data-darkmode-bgcolor-16277440621604="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(248, 248, 248)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">不保存文件退出</td></tr><tr data-darkmode-bgcolor-16277440621604="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(255,255,255)" data-style="font-size: inherit; color: inherit; line-height: inherit; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td width="228" data-darkmode-bgcolor-16277440621604="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(255,255,255)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">wq！</td><td width="266" data-darkmode-bgcolor-16277440621604="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(255,255,255)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">保存退出</td></tr><tr data-darkmode-bgcolor-16277440621604="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(248, 248, 248)" data-style="font-size: inherit; color: inherit; line-height: inherit; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td width="228" data-darkmode-bgcolor-16277440621604="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(248, 248, 248)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">hostname</td><td width="266" data-darkmode-bgcolor-16277440621604="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(248, 248, 248)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">查看当前主机名</td></tr><tr data-darkmode-bgcolor-16277440621604="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(255,255,255)" data-style="font-size: inherit; color: inherit; line-height: inherit; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td width="228" data-darkmode-bgcolor-16277440621604="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(255,255,255)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">ifconfig</td><td width="266" data-darkmode-bgcolor-16277440621604="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(255,255,255)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">查看网卡相关信息</td></tr><tr data-darkmode-bgcolor-16277440621604="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(248, 248, 248)" data-style="font-size: inherit; color: inherit; line-height: inherit; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td width="228" data-darkmode-bgcolor-16277440621604="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(248, 248, 248)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">firewall-cmd --state</td><td width="266" data-darkmode-bgcolor-16277440621604="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277440621604="#fff|rgb(248, 248, 248)" data-style="padding: 0.5em 1em; border-color: rgb(204, 204, 204); color: inherit; line-height: inherit; font-size: 1em; text-align: left;">centos7 查看卡其关闭防火墙状态</td></tr></tbody></table>

5 用户管理
------

> 刚才说了可以创建自己的用户，那么怎么创建自己的用户呢?

添加用户

```
useradd -d /home/lanj -m lanj
```

更改密码

```
passwd lanj
```

系统有很多的用户，怎么进行用户的切换？

```
su -lanj
```

```
su -root
```

> 如果需要

用户之间的切换使用 su 命令实现。root 用户可以无需输入密码切换到 lj 用户，如果普通用户 lj 切换到 root 用户则需要输入密码，我们看看

su -lj

su -root

> 如何切换路径，绝对路径和相对路径

6 软件的安装方法
---------

> 在 Linux 安装相关的工具分为三种方式，分别为源码安装，RPM 包安装以及 YUM 安装方式

**源码安装方式**

> 开源软件都会提供源码下载的方式，对于源代码安装方式的好处即可以定制软件功能，安装需要的模块，不需要的模块可以屏蔽，方便管理，卸载等。

对于源码安装的步骤如下

*   下载解压源码
    

> 一般下载下来源码以后都会存在一个 Readme 文件，首先应该仔细阅读这个文件，可能有很多需要修复的以前人家遇见的问题都会在上面做记录，以免入坑不回头

*   分析平台环境
    
*   编译安装软件
    

这里会使用 make 工具，make 工具就会通过 makefile 文件来实现。makefile 文件是一种按照某种语法来编写且定义了各个文件的依赖关系。

在 Linux 中，习惯使用 Makefile 替代 makefile，当用户执行 configure 后，就会在当前目录生成这个 makefile 文件，然后用户输入 make 就开始运行。我们看看 Makefile 是怎么个有样子

```
edit : main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o       /*注释:如果后面这些.o文件比edit可执行文件新,那么才会去执行下面这句命令*/
    cc -o edit main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o

main.o : main.c defs.h
    cc -c main.c
kbd.o : kbd.c defs.h command.h
    cc -c kbd.c
command.o : command.c defs.h command.h
    cc -c command.c
display.o : display.c defs.h buffer.h
    cc -c display.c
insert.o : insert.c defs.h buffer.h
    cc -c insert.c
search.o : search.c defs.h buffer.h
    cc -c search.c
files.o : files.c defs.h buffer.h command.h
    cc -c files.c
utils.o : utils.c defs.h
    cc -c utils.c
clean :
    rm edit main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o
```

> make 和 make install 的关系

当我们输入 make 命令过后即进入了编译阶段，编译时间根据软件的程序规模大小以及硬件配置有关，当输入 make install 就会开始安装软件，我们可以指定安装目录也可以不指定，系统将给你默认指定目录为 / user/local，这样安装完毕。

**RPM 安装方式**

> RPM 是 Red Hat 公司开发出来的 Linux 下的软件包管理工具。这些以. rpm 结尾的包包含了已经编译好的二进制可执行文件，一句话即将源代码进行编译，安装，然后封装为 RPM 包

优点即安装简单，方便，因为已经编译完成，安装只是用来验证和解压过程，缺点也比较明显，过于依赖于操作系统，要求 RPM 包的安装环境必须和 RPM 封装时的环境保持一致，

> RPM 包是怎么个样子？

```
server-2.1.0-22.I386.rpm
```

其中：server 为如那件的名称

2.1.0：软件的版本号

22：软件更新发行的次数

i386: 适合硬件发行的次数

.rpm:rpm 软件包的标识

**YUM 安装方式**

*   查看是否存在 yum
    

```
rpm -qa | grep yum
```

*   没有则安装
    

```
rpm -ivh yum-*.noarch.rpm
```

*   自定义 yum 的配置。我们可以通过打开 / etc/yum.repos.d/Centos-Base.repo 进行源的配置
    

> YUM 有哪些特点呢

*   安装方便
    
*   可以同时配置多个源
    
*   配置文件简单明了
    

> 推荐个不错的 yum 源

*   EPEL
    

是一个针对红帽企业版 Linux 及衍生发行版的一个高质量附加软件包项目。网址：http://fedoraproject.org/wiki/EPEL/zh-cn

*   RPMForge
    

> 这是一个第三方软件仓库，被 centos 社区认为是一个最安全最稳定的一个软件仓库

6 shell
-------

> 大部分情况都是 Linux 操作系统，那么熟悉命令的用法以外，熟悉使用 shell 脚本能介绍不少时间

**shell 是什么**

> “平时经常在 Linux 操作系统中使用各种命令，比如查看当前的目录文件，我们会使用 "ls" 或者 "ls -l"，这些字符串参数实际上会被 "某段程序" 执行并启动它。这个负责将用户输入的字符串转换为需要执行程序的东西叫做 "shell"。即帮用户更方便使用操作系统接口的 “壳”。同样的壳还有当我输入 Maven + 相关参数的时候是不是就会去执行相应的功能，我们驶入 sql 语句的时候，数据库引擎是不是也会各种调用，一样的道理

**尝试编写第一个 shell**

> vim 创建打开一个文件，扩展名为. sh。如下所示

```
#!/bin/bash #告诉系统使用什么解析器
echo "Hello xiaolan !" # echo进行输出
```

*   执行方法 1
    

```
 chmod +x ./hello.sh ./hello.sh
```

*   执行方法 2
    

```
 /bin/sh hello.sh
```

**变量**

> 变量名和等号之间不能有空格

定义变量注意事项

*   命名首个字符不能是数字，只能使用英文字母、数字和下划线
    
*   不能使用标点符号
    
*   不能使用 bash 中关键字
    

**变量使用**

> 使用变量 (使用变量的过程中，最好加上花括号)，只需要在变量前面加上美元符号即可

```
#!/bin/bash
James="小皇帝"
echo $James
```

**只读变量**

> 使用 readonly 将变量定义为只读，只读意味着不能改变

```
#!/bin/bash
James="小皇帝"
readonly James
James="登哥"
```

删除变量

> 使用 unset 删除变量 变量删除以后不能再次使用，且不能删除只读变量

```
#!/bin/bash
James="小皇帝"
unset James
echo $James #不会有任何输出
```

变量类型

*   局部变量
    

> 仅当前 shell 可用

*   环境变量
    

> 所有程序都能访问环境变量

*   shell 变量
    

> 通过一部分环境变量和 shell 变量保证 shell 的正常运行

字符串

> 使用字符串的过程中，既可以用双引号也可以用单引号，也可以不用

*   单引号
    

> 单引号内容原样输出，不能包含变量，且不能出现单独单引号

*   双引号
    

> 可以出现转义字符

```
#!/bin/bash
James="小皇帝"
str="\"$ James\"! oh my gad \n "
echo -e $str 
```

**获取字符串长度**

> 使用#

```
string="qwert"
echo $(#string)

# 提取子字符串
echo $(string:1:3)
#查找字符串
echo 
```

**数组**

> 支持以为数组

定义数组

> 数组元素使用 “空格” 隔开

```
array=(value1,value2,value3)
```

读取数组

```
value1=${array[0]}
```

使用 @输出数组所有元素

```
echo ${array[@]}
```

获取数组中所有元素以及数组长度

```
#! /bin/bash
# author：xiaolan
array[0]=a
array[1]=b
array[3]=c

echo “数组的元素为：${array[*]}”
echo “数组的元素为：${array[@]}”
echo “数组的个数为：${#array[*]}”
echo “数组的个数为：${#array[@]}”
```

执行

```
./array.sh
```

结果

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEdIPJ8dbSEI8NFwhVicbh7bbeIiafn2q3aVicUEksLBTS3VVm0EicJgcERA/640?wx_fmt=png)

result

**注释**

**单行注释**

> 使用 #开头的行为注释，会被解释器忽略

多行注释

**shell 传递参数**

> **在执行** shell 的时候，命令行指定参数，如下所示

```
#!/bin/bash
James="小皇帝"
echo "执行的文件名为:$0"
echo "第一个参数为:$1"
echo "第二个参数为:$2"
```

**执行**

> ./param.sh 1 2

**结果**

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEQicD6uCdoVLpQ6nj6D7Wj7evxGoSgeDqIHNxqbictfaUPibib825TJVrbA/640?wx_fmt=png)

result

**几个特殊字符**

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEJCG0yvshIBBcroz7u9iccz46AcrJgWs4T86ElB3Qs2ibYMNO5ib8XE42Q/640?wx_fmt=png)

result

案例 (partionnal.sh)

```
#!/bin/bash
# author:xiaolan

echo "-- \$* 演示 ---"
for i in "$*"; do
    echo $i
done

echo "-- \$@ 演示 ---"
for i in "$@"; do
    echo $i
done
```

执行

```
./demo2.sh 1 2 3
```

结果

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEA0icZsXIvicaKc598iaZeK0UibhHzSBz7R3mErzU3mjEWKJuHBsmpOKLAA/640?wx_fmt=png)

img

相同点：都是会引用所有参数

不同点：在使用双引号的时候。如果脚本运行时两个参数为 a,b，则 "*" 等价于 "ab", 而 "@" 等价于 "a","b"

```
#!/bin/bash
# author:xiaolan

echo "-- \$* 演示 ---"
for i in "$*"; do
    echo $i
done

echo "-- \$@ 演示 ---"
for i in "$@"; do
    echo $i
done
```

8 printf

> 使用 printf 格式化字符串，同时可以指定字符串宽度和对齐方式，格式如下

```
printf format-string [arguments...]

#!/bin/bash
# author:xiaolan

printf "%-8s %-8s %-4s\n" 姓名 科目 分数  
printf "%-8s %-8s %-4f\n" 小明 数学 97
printf "%-8s %-8s %-4f\n" 小话 语文 89
printf "%-8s %-8s %-4f\n" 王三 英语 93
```

**结果**

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEtXRr1va60BAmDNoNCVfDialic0vupIekFn0EbpRX4oXD35s4ldy5EYsw/640?wx_fmt=png)

img

**9 test**

> shell 中的 test 用于检查某个条件是否成立

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6ME4m2ice8IQQf2BibcMzdZfK4hDqlqOqMgYe5aHzdf8Dykq4vb3ff5WNSQ/640?wx_fmt=png)

result  

案例

```
#!/bin/bash
# author:xiaolan
num1=55
num2=55
if test $[num1] -eq $[num2]
then
    echo '两个数相等！'
else
    echo '两个数不相等！'
fi
```

结果

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEFNWpRfPw885fCXrZkLwJqUjNHlW2ZFNNQV4dmfoNfibcpCVuicXibF4nw/640?wx_fmt=png)

result

字符串比较

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEnIyO5qqF4BFibVW494nb1mibXrhvpwlIIqGrbzLo3azj4ic0MoDUibOgjA/640?wx_fmt=png)

字符串比较

```
#!/bin/bash
# author:xiaolan
num1="xiaolan"
num2="xiaolna"
if test $num1 = $num2
then
    echo '两个字符串相等!'
else
    echo '两个字符串不相等!'
fi
```

结果

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEX0JjAmaQ8Awl8WQb0YB5sGHNk6mKk73pLHRN9r9l63SvoU5FmQ82fg/640?wx_fmt=png)

result

**流程**

if 语句语法格式

```
if condition
then
    exec1 
    exec2
    ...
    execN 
fi
```

如果简化为一行

```
if [$(ps -ef | grep -c "httpd") -gt 1];then echo "true";fi
```

if else-if else

```
if condition1
then
    exec1
elif condition2 
then 
    exec2
else
    execn
fi
```

案例 判断两数值是否相等

```
#!/bin/bash
# author:xiaolan
a=2
b=3
if [ $a == $b ]
then
   echo "a 等于 b"
elif [ $a -gt $b ]
then
   echo "a 大于 b"
elif [ $a -lt $b ]
then
   echo "a 小于 b"
else
   echo "没有符合的条件"
fi
```

for 循环

```
for loop in 1 2 3 4 5
do
    echo "The value is: $loop"
done
```

while 语句

> “ 通常用于从输入文件不断读取数据

```
while condition
do
    exec
done

#!/bin/bash
# author:xiaolan
int=1
while(( $int<=6 ))
do
    echo $int
    let "int++"# 用于执行一个或者多个
done
```

无限循环

```
while true
do
    exec
done
```

case 语句

> 多选择语句。取值后面为单词 in，每一个模式以 ")" 结束。匹配发现取值符合某一模式后，其间所有命令开始执行直至 ";;"。

```
#!/bin/bash
# author:xiaolan
echo '输入 1 到 3 之间的数字:'
echo '你输入的数字为:'
read aNum
case $aNum in
    1)  echo '你选择了 1'
    ;;
    2)  echo '你选择了 2'
    ;;
    3)  echo '你选择了 3'
    ;;
    *)  echo '你没有输入 1 到 3 之间的数字'
    ;;
esac
```

输入不同的内容，会有不同的结果，例如：

```
输入 1 到 4 之间的数字:
你输入的数字为:
你选择了 3
```

跳出循环

break

> break 命令允许跳出所有循环

```
#!/bin/bash
# author:xiaolan
while :
do
    echo -n "输入 1 到 3 之间的数字:"
    read aNum
    case $aNum in
        1|2|3) echo "你输入的数字为 $aNum!"
        ;;
        *) echo "你输入的数字不是 1 到 3 之间的! 游戏结束"
            break
        ;;
    esac
done
```

continue

> 跳出当次循环

```
#!/bin/bash
while :
do
    echo -n "输入 1 到 3 之间的数字: "
    read aNum
    case $aNum in
        1|2|3|4|5) echo "你输入的数字为 $aNum!"
        ;;
        *) echo "你输入的数字不是 1 到 3 之间的!"
            continue
            echo "游戏结束"
        ;;
    esac
done
```

**10 shell 函数**

> 用户定义函数，然后在 shell 脚本中随便调用，格式如下

```
[ function ] funname [()]

{

    action;

    [return int;]

}
```

例子

```
#!/bin/bash
# author:xiaolan

Fun1(){
    echo "这是我的第一个 shell 函数!"
}
echo "-----函数开始执行-----"
Fun1
echo "-----函数执行完毕-----"
```

带 return 语句

```
#!/bin/bash
# author:xiaolan

FunReturn(){
    echo "这个函数会对输入的两个数字进行相加运算..."
    echo "输入第一个数字: "
    read aNum
    echo "输入第二个数字: "
    read anotherNum
    echo "两个数字分别为 $aNum 和 $anotherNum !"
    return $(($aNum+$anotherNum))
}
FunReturn
echo "输入的两个数字之和为 $? !"
```

函数参数

```
#!/bin/bash
# author:xiaolan

funParam(){
    echo "第一个参数为 $1 !"
    echo "第二个参数为 $2 !"
    echo "参数总数有 $# 个!"
    echo "作为一个字符串输出所有参数 $* !"
}
funParam 1 2 3 4
```

**shell 重定向**

输出重定向

> command1 > file # 如果 file 中存在内容将被清空覆盖。如果追加使用 command1 >>file

```
ls -l > dir.txt
```

cat dir.txt

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6METlQKXhGiaQGzog6z1knB9JJpYs9OmJe8nVwo5KnOxSAnnRgU8vchicJw/640?wx_fmt=png)

img

/dev/null 文件

> 写入到它的内容都会被丢弃，会起到 "禁止输出" 的效果，如果希望屏蔽 stdout 和 stderr  “command> /dev/null 2>&1

注意：Linux 命令行都会打开三个文件

*   标准输入文件: stdin 文件描述符为 0
    
*   标准输出文件: stdout 文件描述符为 1
    
*   标准错误文件: stderr 文件描述符 2
    

**12 运算符**

下表列出了常用的算术运算符，假定变量 a 为 2，变量 b 为 3：

算术运算符

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEttdro7Lp57R9axp80utYO5hIYFRar7rx92GJvCqNE7YzKUictiaibKiaQw/640?wx_fmt=png)

算术运算符

关系运算符

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEpxbUaic6ibheW16n0qD8Nn2p7Oib6ticLITZ0PLpkh9KXRX1m0icbb9AIwA/640?wx_fmt=png)

关系运算符

布尔运算符

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEK1Biblp5z9TGjqTPPVt00NzAD8ahn4PLNYNmjoNCG1dtiaEicotUNibB5g/640?wx_fmt=png)

布尔运算符

逻辑运算符

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEfM0MdeQHlSPJLFcLTCX8lMCoBpQq0PqhcF9T1k90sfaXAGM7ou8diag/640?wx_fmt=png)逻辑运算符

字符串运算符

  

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEG71icXZ0C0GS5EQc5b2WZsfgmIYF4ngLaSVlb1JvZ0mpibohdvNxS52w/640?wx_fmt=png)

**12 shell 实战**  

*   请将当前目录中 demo.txt 第二行第三列数据输出到 demo2.txt 中
    

```
 cat demo.txt|awk ’NR==2{print $3}’ >demo2.txt 
```

*   日志如下统计访问 ip 最多的前 10 个
    

```
awk ’{print $1}’ *.log | sort | uniq -c | sort -nr | head -n 
```

uniq - 删除排序文件中的重复行 sort 对于文本进行排序 -l 按照当前环境排序. -m 合并已经排序好的文件, 不排序. -n 按照字符串的数值顺序比较, 暗含 - b -r 颠倒比较的结果.

*   查看占用内存最大的进程的 PID 和 VSZ
    

```
ps -aux|sort -k5nr|awk ’BEGIN{print ”PID VSZ”}{print ![2,(https://www.zhihu.com/equation?tex=2%2C)2,5}’|awk ’NR<3′ 
```

*   如何检查文件系统中是否存在某个文件
    

```
if [-f /var/log/messages]
then
echo "File exts"
fi
```

*   每个脚本开始的 #!/bin/sh 或 #!/bin/bash 表示什么意思 ?
    

> #!/bin/bash 表示脚本使用 /bin/bash。对于 python 脚本，就是 #!/usr/bin/python

*   & 和 && 区别
    

> ““&” 脚本在后台运行时使用它。“&&” 当前一个脚本成功完成才执行后面的命令

*   脚本文件中，如何将其重定向标准输出和标准错误流到 log.txt 文件 ?
    

```
./a.sh >log.txt 2>&1
```

*   如何计算本地用户的数目
    

```
wc -l /etc/passwd | cut -d
```

*   shell 中进行字符串比较和数字比较
    

```
[ $A == $B ] – 用于字符串比较
[ $A -eq $B ] – 用于数字比较
```

*   去掉字符串空格
    

> echo $string | tr -d " "

*   统计内存使用
    

```
#! /bin/bash
# author:xiaolan
sum=0
for mem in `ps aux |awk '{print $6}' |grep -v 'RSS' `
do
    sum=$[$sum+$mem]
done
echo "The total memory is $sum""k"
```

结果

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MErtzzPOZhLBeFnQDplRLuCicNNyLH50CyIhzuEwrbEqy2oibgjibFXGkyA/640?wx_fmt=png)

result

*   批量更改文件名
    

> 批量修改 123 目录下 txt 为 txt.temp。将 temp 打包为 test.tar.gz

```
#!/bin/bash
##查找txt文件
find /123 -type f -name "*.txt" > /tmp/txt.list
##批量修改文件名
for f in `cat /tmp/txt.list`
do
    mv $f $f.temp
done
##创建一个目录，为了避免目录已经存在，所以要加一个复杂的后缀名
d=`date +%y%m%d%H%M%S`
mkdir /tmp/123_$d
##把.temp文件拷贝到/tmp/123_$d
for f in `cat /tmp/txt.list`
do
    cp $f.temp /tmp/123_$d
done
##打包压缩
cd /tmp/
tar czf 123.tar.gz 123_$d/
```

7 awk 文本处理工具
------------

> awk 是一个处理文本文件的应用程序，几乎所有的 Linux 系统都自带了这个程序

依次处理每一行，并读取里面的每一个字段。对于处理生产环境的日志有着非常高校的作用

**基本用法**

```
# 格式
awk 做什么 文件吗
awk 'print $0' lan.txt
```

上面 lan.txt 是 awk 需要处理的文本文件。前面单引号里面有一个大括号，单引号里面就是每一行的处理动作。其中 print 为打印命令，

**上菜**

```
echo 'my name is lanlan' | awk '{print $0}'
```

上面代码中，print 0 位当前行，所以执行结果就是把每一行原样打印出来∗∗上菜∗∗¨G56G 上面代码中，print0 即将标准输入 my name is lanlan ,c 重新打印一遍

awk 根据空格和制表符，将每一行分成若干段，依次为 2

```
echo 'my name is lanlan'| awk '{print $3}'
```

为了方便，我们直接使用 / etc/passwd 文件进行操作，

```
awk -F ':' '{ print $1 }' demo.txt
```

**3 变量**

上面我们说了，可以使用符号 “3 代表第一个字段，第二个字段，第三个字段 ¨G57G 为了方便，我们直接使用 /etc/passwd 文件进行操作，¨G58G∗∗3 变量∗∗上面我们说了，可以使用符号 “+” 数字的方式表示第几个字段，其实还有一些变量可以直接表示相应的字段。比如 “$NFb” 表示最后一个字段

```
echo 'my name is lanlan'| awk '{print $NF}'
awk -F ':' '{print NR ") " $1}' demo.txtshe
```

这里出现了双引号，表示原样输出

其他常用的内置变量

*   FILENAME：当前文件名
    
*   FS：字段分隔符，默认是空格和制表符。
    
*   RS：行分隔符，用于分割每一行，默认是换行符。
    
*   OFS：输出字段的分隔符，用于打印时分隔字段，默认为空格。
    
*   ORS：输出记录的分隔符，用于打印时分隔记录，默认为换行符。
    
*   OFMT：数字输出的格式，默认为％.6g。
    

**4 函数**

既然算是一门语言，函数当然少不了，下面看一波常用的函数

函数 toupper() 用于将字符转为大写

```
awk -F ':' '{ print toupper($1) }' demo.txt
```

可以发现第一个字段输出的时候变成了大写

*   tolower()：字符转为小写。
    
*   length()：返回字符串长度。
    
*   substr()：返回子字符串。
    
*   sin()：正弦。
    
*   cos()：余弦。
    
*   sqrt()：平方根。
    
*   rand()：随机数。
    

5 条件 **

通过使用相应的条件，过滤出自己想要的内容

```
awk '条件 动作' 文件名
```

上菜

```
$ awk -F ':' '/usr/ {print $1}' demo.txt
root
daemon
bin
sys
```

这里 / usr / 表示只输出包含 usr 的行

这个例子输出奇数行

```
# 输出奇数行
$ awk -F ':' 'NR % 2 == 1 {print $1}' demo.txt
root
bin
sync

# 输出第三行以后的行
$ awk -F ':' 'NR >3 {print $1}' demo.txt
sys
sync
```

下面的例子输出第一个字段等于指定值的行。

```
$ awk -F ':' '$1 == "root" {print $1}' demo.txt
root

$ awk -F ':' '$1 == "root" || $1 == "bin" {print $1}' demo.txt
root
bin
```

**5 if 语句**

通过 if 语句编写比较复杂的内容

```
$ awk -F ':' '{if ($1 > "m") print $1}' demo.txt
root
sys
sync
```

上面代码输出第一个字段的第一个字符大于`m`的行。

if 结构还可以指定 else 部分。

8 进程管理与定时任务和后台执行
----------------

> crond 是什么？

crond 是一个可以在指定时间执行一个 shell 脚本或者一系列的 Linux 命令。和 Windows 下的计划任务类似。当安装完操作系统后，默认会安装这个服务工具，并且会自动启动 crond 进程。

在 Linux 中任务的调度分为**两类**

*   系统任务的调度
    

> 系统会周期性的执行一些工作，比如说写缓存的数据到硬盘，清理日志等

*   用户任务的调度
    

> 用户定期也会执行一些任务，比如用户数据的备份，定时的邮件提醒等，这些都是通过 crondtab 来设置

**那么 crontab 到底怎么用么**

首先看看 crontab 的使用格式：

```
crontab -u user file
```

**常见的选项**

*   -u user: 很明显是需要表明是那个用户的 crontab 服务，别瞎搞
    
*   file:file 是命名文件的名字，表示将 file 作为 crontab 的任务列表文件并载入到 crontab 中
    
*   -e:e 为 edit，表示标记某个用户的 crontab 文件内容
    
*   -l: 显示用户的 crontab 文件、
    

crontab 的含义

创建的 crontab 文件，每一行代表一项任务，每个字段都有对应的设置规则，一共分为 6 个字段，分别为：

```
minute hour day month week command
```

*   minute: 区间 0-59
    
*   hour: 区间 0-23
    
*   day: 区间 0-31
    
*   month: 区间 1-12
    
*   week: 区间 0-7 周日可以是 0/7
    
*   command
    

这里的 command 代表的是需要执行的而命令，通常为脚本文件，

除了上面几个字段，还需要注意几个特殊字段

*   *: 代表所欲呕可能的值
    
*   ，: 通过，来表示区间范围的值
    
*   _: 整数之间的中杠表示一个证书范围
    
*   正斜线：表示时间的间隔频率，比如 0-23/2 表示每两个小时执行一次  
    开始放几个例子 **
    

```
crontab -e
0 5 * * * /root/bin/backup.sh
```

这代表的是每天早上 5 点运行 backup.sh

> 每个工作日 11:59pm 进行备份作业

```
59 11 * * 1-5 /root/bin/backup.sh
```

> 每五分钟运行一个命令

```
*/5 * * * * /root/bin/check-status.sh
```

> crontab 有哪些选项

crontab -e: 修 改 crontab 文件，如果文件不存在会自动创建

crontab -l：显示 crontab 文件

crontab -r：删除 crontab 文件

crontab -ir: 删除 crontab 文件前提醒用户

9 后台运行
------

**用途**：不挂断的运行命令

**语法**：nohup Command [Arg …] [&]

*   无论是否将 nohup 命令的输出**重定向**到终端，输出都将附加到当前目录的 nohup.out 文件中。
    
*   如果当前目录的 "nohup.out" 文件不可写，输出重定向到 "home/nohup.out"
    
*   如果没有文件能创建或打开以用于追加，那么 Command 参数指定的命令不可调用。
    

**退出状态**：该命令返回下列出口值：　

*   126 可以查找但不能调用 Command 参数指定的命令。　
    
*   127 nohup 命令发生错误或不能查找由 Command 参数指定的命令。　
    
*   否则，nohup 命令的退出状态是 Command 参数指定命令的退出状态。
    

**使用 &**  
用途：在后台运行, 一般两个一起用

```
nohup command &
```

  

- EOF -

1、[华为开发者被批评在 Linux 内核刷 KPI](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666555423&idx=1&sn=0ed317fcbd6ef03172f2d6b4649083c6&chksm=80dcacb4b7ab25a238315331ebdfa84422b1e075744ca4c7113103282badde78260938715290&scene=21#wechat_redirect)

2、[Linux 终端复用神器 Tmux 使用详解，看完我飘了～](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666555307&idx=2&sn=0f473d4b3368ea63731a41fbeef572d5&chksm=80dca300b7ab2a161e709839ab3ac5930bbdabc3f886d3d3c5c4d0dadca97525e9a5ff785eaf&scene=21#wechat_redirect)

3、[万字整理，肝翻 Linux 内存管理所有知识点](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666555189&idx=1&sn=258e86295e102cfd83509b22b1da320d&chksm=80dca39eb7ab2a8848e3b9072c2ce24781d613eee6518a637b04fb304c789d59f38540fac755&scene=21#wechat_redirect)
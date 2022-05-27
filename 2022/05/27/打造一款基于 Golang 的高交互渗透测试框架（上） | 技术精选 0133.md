> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAwMzYxNzc1OA%3D%3D&mid=2247499601&idx=1&sn=11e154a5c0bba4a826bf5e633105ba2a&scene=45#wechat_redirect)

  

本文约 1600 字，阅读约需 6 分钟。

“为什么要自己写？”

“自己写的工具有大公司写的成熟工具好用吗？”

这是当初有这想法的时候，大佬们问的最多的问题。其实对我个人而言，只是想学习一下 Go 语言，同时为开源社区做做贡献而已。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56pWiaI9aSleMcoDiafiborNDkaXVyFGNzNCER9TeKlBPjxKoPYYFVzagGrbtOMVj4PfFPM3zv5a2CKA/640?wx_fmt=png)

然而，在日常渗透测试时，时常发现有些工具用着不很顺手——参数太多记不住、想用的功能还要收费、明明存在问题的地方用工具测试居然报错、不支持跨平台......

这些不方便的地方又增强了自己写一个工具的想法。然而，在随后边学边写的过程中，发现自己写了个 BUG 出来。

下面给大家介绍一下这个 “BUG” 是如何打造的。

**1**

  

**搭建高交互 CLI**

  

---

由于对 MSF 一直有种莫名的崇拜，想要做一款类似 MSF 的工具，于是选取了一个高交互式 CLI（命令行框架），采用 c-bata/go-prompt 这个库，主要包含 Executor 和 Completer。
-----------------------------------------------------------------------------------------------------------

Executor 相当于执行器，用于设置 CLI 需执行的指令；Completer 相当于解释器，用于给指令需要的参数设置自动补全。  

---------------------------------------------------------------------

基本代码结构如下：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56pWiaI9aSleMcoDiafiborNDkiaFa7CENBJJuJSdnXkYicWdx64YAhrOBJc7g28BGLJ8yglCtoRNS7U3g/640?wx_fmt=png)

首先，生成一个自定义的 CLI。由于要动态修改 CLI 前缀，需设置 OptionLivePrefix 属性：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56pWiaI9aSleMcoDiafiborNDkibCL7iaekHZlfH5nDHnfXo1wzU0mTqZibFicxj5WOXIHHCyNNcJJicVFGvQ/640?wx_fmt=png)

进入 Executor 进行设置。设置支持的指令（load、poc、help、exit、quit）：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56pWiaI9aSleMcoDiafiborNDkB5wnicCuss3mKqdOPrqhkuXWmlibLJvWQan6E7eKz1CdxjHBWmUCrbRg/640?wx_fmt=png)

接下来，给这些指令设置自动补全，有些命令使用的参数不同，需要设置多个函数进行对应：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56pWiaI9aSleMcoDiafiborNDkylYuy0jPc8l27ibJP9wSaqtQYkSANPoelQWdzgmyHBw1M2aoWicOQiaPA/640?wx_fmt=png)

自动检测传入的参数有几个，随即进行设置：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56pWiaI9aSleMcoDiafiborNDkkYbUNE7GS9H4ODxIGAB5OCP3365L8poK9MoNQByPB3XJ2Lrqa5rXPg/640?wx_fmt=png)

**2**

  

**基础扫描模块**

代码结构：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56pWiaI9aSleMcoDiafiborNDkf5AC6xdILOvDDEhE2VMEsxHdWNPkVjTN2f2oVqoL0BfS5DUUIHx34w/640?wx_fmt=png)

### **端口扫描**

定义端口扫描的 CLI：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56pWiaI9aSleMcoDiafiborNDkoZib02G3hhgXUTfzDED2vSu7VF230Dy582mLxsw2H84ApbHAnbW6XyQ/640?wx_fmt=png)

设置扫描用的参数，针对端口扫描，优化了 set 参数。考虑到大范围扫描，支持读取文件操作：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56pWiaI9aSleMcoDiafiborNDkzSurLtz3o4QicOKffTmquCmicv9RDqiagTbslibiaoGASzOWfhnGrWgptPg/640?wx_fmt=png)

获取设置的 IP，随后进行全端口扫描，之后对扫描出的端口进行服务识别：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56pWiaI9aSleMcoDiafiborNDkGVzic5EXRvzB1LicdbxfFrPozmBhwNicKcxka6jf8FVQmkqqoOlkhPiapQ/640?wx_fmt=png)

Go 语言天然的高并发性是当初选择它的原因，但问题也随之而来——在进行多线程扫描时很容易产生死锁。使用了 sync.Mutex 互斥锁之后，发现耗时很高，还不如单线程快。

好不容易运行结束，又由于扫描进程同时在 CLI 进程中，导致使用互斥锁后，不知为何将 CLI 进程退出了。

在处理多线程时纠结了好久。后来又想到一个办法——使用 channel 对 goroutine 进行控制。这样可以保证在同一时刻，仅有一个 goroutine 能向一个通道发送元素值，同时也仅有一个 gorountine 能从它那里接收元素值。

多线程全端口扫描：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56pWiaI9aSleMcoDiafiborNDkk5tj3at9WKOd9RlXaIkmoTfe1ODRQmDR56vOdkxjjfVJjTNCYlhYeQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56pWiaI9aSleMcoDiafiborNDkY8iaq3HyFPeXicene7gwiaDDpUBarIMIjYkibA5PFtS8mqQ1cKmxG16jhw/640?wx_fmt=png)

对端口进行服务和应用版本检测，利用 banner 进行识别，内置指纹探针采用 nmap-service-probes：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56pWiaI9aSleMcoDiafiborNDkTMnfiam5oVA7gWxfP9sxCrzNtHjOjwglCsiclgst8jjXFVpJ8qH9kYmw/640?wx_fmt=png)

完成后的效果：

![](https://mmbiz.qpic.cn/mmbiz_gif/WTOrX1w0s56pWiaI9aSleMcoDiafiborNDkAyDibolWEEEqVbxwEkv1bc8WANFia3WbHmSq2qUJjvgvicY9XUOkuFMvw/640?wx_fmt=gif)

看着代码顺利的运行，我满心欢喜，沉浸在激动的心情中。然而此时大佬泼来一盆冷水。

原来大佬对自己的服务器进行了测试，发现扫描的结果不准——明明开了 3389，愣是扫不到。

经过排查发现，工具默认的是对全端口进行扫描，测试时，为了达到最大性能，设置了 70000 的线程。

大佬测试时只要开始运行，本地网络就呈现断开的情形——原来是自己把自己的带宽消耗光了。这难道就是 “发起狠来连我自己都害怕”。

随后将线程调低，使用结果还是蛮准的：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56pWiaI9aSleMcoDiafiborNDked6I8EO5MIE5HGfrXZlwbWs63jVyBibQdqF9lncMgq7Xa0CqI2P8Khw/640?wx_fmt=png)

### **子域名扫描和敏感路径扫描**

对于子域名和敏感路径扫描功能，借鉴了 gobuster 的实现方式（向 gobuster 作者致敬）：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56pWiaI9aSleMcoDiafiborNDk8LI9OMRGLibkKYIu2W1dgiahzeZjibLxT67FVzRHGB08K3NVApFsMyogQ/640?wx_fmt=png)

首先定义一个执行器：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56pWiaI9aSleMcoDiafiborNDkpkNsOnjqb2eBQiabnB9kMadIe61RxIuibWCMod0pM85GpwOXmmDaj7Cg/640?wx_fmt=png)

利用 init 函数定义初始化的参数，RunE 参数后面是要执行的主要函数：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56pWiaI9aSleMcoDiafiborNDk34Vl6oc6COsAgnoib9yfbzmen8Q1g2Z7eEsc41rHgT6Q14UbbCZjICQ/640?wx_fmt=png)

敏感路径扫描模块如下图。实现代码与子域名模块类似，目前针对敏感路径和子域名，只实现了通过字典进行爆破的方式：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56pWiaI9aSleMcoDiafiborNDkicco3PaicgZSsWr793YpVhyVVafw2e6U5zBkXNQ2NNibQ6u6iapKM08iafQ/640?wx_fmt=png)

还可以针对性地设置一些 cookie、header 等 http 访问参数：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56pWiaI9aSleMcoDiafiborNDk9HOM4SVog7JC1grsJGMPUjqibzZ4DYib7qGsyEpdk9ESYFbxIrSetXiaA/640?wx_fmt=png)

完成之后的效果：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56pWiaI9aSleMcoDiafiborNDkRfVK4ygHECDyMcDs1gc7FmpficCx10lDhA8x4eia1gzzD2oJhNKibicReQ/640?wx_fmt=png)

**3**

  

**下期预告**

在下周的更新中，将带来此工具的弱口令爆破模块以及 POC 模块，当然，还有**项目的开源地址**。

下周同一时间，我们不见不散。
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/wEFr3jAiLdmEa6bFxAwqKg)

> vscode 阅读 Linux 源码的完美体验，性能无上限！

![](https://mmbiz.qpic.cn/mmbiz_png/4UtmXsuLoNfrjwr6vCeRTaL3kQLG1CZcCos67QOeONfVkuHCEKOjM7VfdRj61ibISlvIDiaUlRW0WjmMfBxt24uw/640?wx_fmt=png)  

Linux 内核代码用什么编辑器？

分享过怎么学习 Linux 内核代码的思路，当时顺便提了一点，用 vscode 看内核代码。有同学对此提出了疑问：

1.  vscode 看 Linux 代码不卡吗？
    
2.  vscode 符号跳转怎么老有问题？
    
3.  windows 开发 Linux 项目好麻烦，总是要手动同步代码？
    

其实，上面提出的三点疑问，合理配置 vscode 是可以完美解决的。今天就以 Linux 内核源码为例，分享一下超大项目源码的源码开发阅读的姿势。

思考遇到的几个问题

**我们经常遇到两个问题：**

1.  本地电脑存在瓶颈，单机性能有限，毕竟资金紧缺？
    
2.  一般电脑安装的都是 windows 或者 mac 图形支撑好的系统，而开发的项目又必须是 Linux 上编译运行？
    

如果本机直接用 vscode ，势必会遇到上面的问题。比如 Linux 代码量巨大，本机性能是 hold 不住，编辑器建立内存的索引非常庞大，既要吃内存又要吃 CPU 。对于编译，那就更麻烦了，涉及到手动的同步。

**我们经常怎么解决它？**

1.  纵向优化电脑，给自己电脑加个内存条，换个 cpu ，这个思路是可以的，但是它永远存在性能瓶颈。这还真不是钱的问题；
    
2.  对于平台依赖，以前的实践是 windows 下编辑，然后用 scp 或者其他同步工具，把代码同步到另外一台 Linux 下去编译，很麻烦；
    

**怎么才能彻底解决它？**

奇伢的最佳实践：vscode 远程开发，利用多机性能；

怎么做才能解决单机的瓶颈和平台依赖？

**解决单机瓶颈的思路非常简单，那就是分布式、多机部署。**

我的使用姿势是本机打开一个 vscode ，但是代码放在远程主机，远程主机是一台性能强劲的 Linux 服务器，速度杠杠的。

这样的好处就是把资源消耗的压力分摊到远程主机，本机的消耗非常少，再大的项目也非常稳定。

第二个好处就是平台的无缝切换，即使你使用的是 windows 机器，也能非常丝滑的进行 Linux 项目的开发，代码编辑，项目编译无感切换。

远程主机是一台 32 核 128 G SSD 盘 的服务器，跑个 Linux 源代码的解析那还不是绰绰有余。

**有童鞋问，这服务器哪里来？**

答：你不是在上班嘛。公司的。

并且这种开发方式能够让你的电脑不用再焦虑，把纵向优化的思路转变成横向的扩展之后，你的瓶颈将不复存在。你的本机只做一个界面即可。

**效果展示：**

![](https://mmbiz.qpic.cn/mmbiz_png/4UtmXsuLoNfrjwr6vCeRTaL3kQLG1CZcWG4SXic5Wy5IOzVotRiatGWEo15kZDQJgBGDicTv5foAdh2A4UsquQfEw/640?wx_fmt=png)

怎么配置 vscode 远程开发？

讲了那么多，我们接下来看看怎么实践。怎么配置这个呢？下面以 Linux 源码为例，手把手教你配置。  

### 准备 Linux 主机

这是我们的远程主机，性能怎么好怎么来。当然，如果只是为了跨平台，其实可以是虚拟机。

### 远程主机安装 global 工具

注意，是在远程主机上安装哦，为了更好，更快的解析我们的符号表。下面是 ubuntu 的命令，其他的 Linux 系统可以查一下，比如 centos 是 yum 安装。

```
apt install global


```

安装完之后，确认两个二进制文件，`global`， `gtags`， 一般在 /usr/bin/ 目录下。有这两个文件，就说明 OK 。

### Linux 源码下载

这个自行去 Github 或者其他镜像网站上下载即可。Github 地址：https://github.com/torvalds/linux.git

下载之后，放到远程的目录即可：

```
root@ubuntu20:/mnt/opensource/linux-3.10# pwd
/mnt/opensource/linux-3.10


```

vscode 安装 Remote-SSH 插件
-----------------------

现在有主机，有代码了，怎么才能让 vscode 具备连接远程主机的能力呢？

**vscode 远程连接主机主要是依赖于微软提供的插件 Remote-SSH。**

![](https://mmbiz.qpic.cn/mmbiz_png/4UtmXsuLoNfrjwr6vCeRTaL3kQLG1CZcYqjRZqrnbIPyiasHKkOcuJtFutY0cP1jpVmyeyUuMNpjoxEOiaLiakJUw/640?wx_fmt=png)

安装完插件之后，vscode 就具备了连接远程主机的能力。在**左下角**有个符号 `><` ，**点击它**就能选择连接哪个远程主机。连接上了之后，会重新开一个窗口，左下角也会显示。

![](https://mmbiz.qpic.cn/mmbiz_png/4UtmXsuLoNfrjwr6vCeRTaL3kQLG1CZczQcITn3lW8brVQQav8ujicZuq2XKkSEbfkOcdTlsPuzCPHxnicPYic5vw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/4UtmXsuLoNfrjwr6vCeRTaL3kQLG1CZcxTRBYNR1ibvaFfTbPEZgELDDb01EATFdlhPCrWuiaRTib6WOolkYAXIwA/640?wx_fmt=png)

这里顺便提一下，微软一共发布了三款远程连接的插件：

1.  Remote - Containers
    
2.  Remote - SSH
    
3.  Remote - WSL
    

名字上也很容易区分，就是支持连接容器，SSH 主机，WSL 子系统。

### 安装 **C/C++ GNU Global 插件**

好，现在万事俱备了。下一步就是把符号解析的事情准备好，你就能享受快速的源代码阅读体验了。

首先我们连接上远程主机，注意：这个时候会重新打开一个窗口，而不是在本地安装。

![](https://mmbiz.qpic.cn/mmbiz_png/4UtmXsuLoNfrjwr6vCeRTaL3kQLG1CZc9hrbBdwic7DNTGQfy3viaCBiaphoMf8oPE5jZ8hDVG19MdEicRPmcnzVHQ/640?wx_fmt=png)

当然，C/C++ 插件也最好安装一下。

**划重点：一定要先连接上主机，然后再安装 GNU Global 插件哦。一定是在远程主机上安装。**

### vscode 配置 global 路径

vscode 的配置（ssh）里，输入以下配置：

![](https://mmbiz.qpic.cn/mmbiz_png/4UtmXsuLoNfrjwr6vCeRTaL3kQLG1CZcka7xQTJBhjoe696LZqWR6sl9seyYsDJWvZrPFyvR0PFdy2IxibzMEXA/640?wx_fmt=png)

在 vscode 的 settings.json 配置里，指定 global 的相关路径。

```
"gnuGlobal.globalExecutable": "/usr/bin/global",
"gnuGlobal.gtagsExecutable": "/usr/bin/gtags",
// 指明生成的符号表存放在哪个位置
"gnuGlobal.objDirPrefix": "/mnt/.global"


```

**注意："gnuGlobal.objDirPrefix" 的路径必须要手动创建好，如果不存在，会导致后续 Rebuild 的失败。**

### 测试是否成功执行远程主机上的 Gtags

好啦，现在插件安装完成了。测试一下安装配置的是否正确。shift + command + P 把命令面板掉出来，执行 Global: Show GNU Global Version 命令，看是否能成功显示版本。在右下角显示版本号，那么就说明一切就绪：

```
global (GNU GLOBAL) 6.6.4


```

### 最后一击，生成符号表

shift + command + P 把命令面板调出来，执行 Global: Rebuild Gtags Database 命令。等待右下角的通知，如果显示：

```
Build tag files successfully


```

那么就说明符号表解析完成了。符号表生成成功会在 "gnuGlobal.objDirPrefix" 的路径里生成三个文件：

```
root@ubuntu20:/mnt/opensource/linux-3.10# ll -lh /mnt/.global/mnt/opensource/linux-3.10/
total 395M
drwxr-xr-x 2 root root 4.0K Feb 14 19:06 ./
drwxr-xr-x 3 root root 4.0K Feb 14 18:33 ../
-rw-r--r-- 1 root root 7.6M Feb 14 19:06 GPATH
-rw-r--r-- 1 root root 278M Feb 14 19:06 GRTAGS
-rw-r--r-- 1 root root 109M Feb 14 19:06 GTAGS


```

好了，上面的搞完，就可以愉快的使用 vscode 看源码了，速度非常快。

![](https://mmbiz.qpic.cn/mmbiz_png/4UtmXsuLoNfrjwr6vCeRTaL3kQLG1CZc3DINUH6AgVKDddgKJvpFh80ZrBIv0wZ55gY8zEGb1Kuo78Gw01csWg/640?wx_fmt=png)

好啦，现在你可以尽情享受 Linux 代码的阅读开发了，既能享受图形界面的便捷，又能无缝的进行 Linux 的开发。而且不用在受限于本机电脑的资源瓶颈，具有无限的扩展空间。  

总结

1.  单机总是存在瓶颈，**纵向优化它总有极限**，并且价格不菲；
    
2.  既要图形界面的便捷？又要无缝切换 Linux 开发模式？**远程开发是个不错的体验；**
    
3.  vscode 使用插件来实现远程开发，本机电脑作为一个界面，符号解析放在远程主机，真正做到**横向扩展，**理论上**性能无上限**；
    
4.  Linux 源代码的解析放到远程主机，vscode 远程连接，源代码的阅读流畅丝滑，**开发体验完美**；
    

后记

好的开发工具让我每天上班都带劲！**点赞、在看** 是最大的支持。

- EOF -

推荐阅读  点击标题可跳转

1、[为什么企业里用 VS Code 的这么多？](http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651160818&idx=1&sn=9b8dd6a450b7e853b0f9090de3a145ad&chksm=806451adb713d8bb292b28185505215130d59a5cf70310d9045428f4a46578c9f3d0ee990c37&scene=21#wechat_redirect)

2、[用 VS Code 居然能找对象了，不看脸的那种](http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651160607&idx=2&sn=8b8905ad9c61e620e8861cc7ec298af7&chksm=80645140b713d856116ba2f6f929b90b1a8689a35943caba7c00e725d27afd4b1df0b76c5a28&scene=21#wechat_redirect)

3、[天天讲路由，那 Linux 路由到底咋实现的！？](http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651170375&idx=1&sn=6bfe3e018304ee8deb15aa78adf8157f&chksm=80647718b713fe0e5f79b825bbcf955efe72957cf7e4c3c5d615e45fbf5706fb5b15b5b579be&scene=21#wechat_redirect)
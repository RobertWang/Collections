> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIzMzMzOTI3Nw==&mid=2247502805&idx=1&sn=e28f8a7727ddbfff81940c73db251825&chksm=e885ab37dff222213215e3bc40961cf9f63d987f1bc2041d07c2b6120895531a20bce7aa7f6f&mpshare=1&scene=1&srcid=0730MYHdfKNpzSebeI0dKsca&sharer_sharetime=1627609947079&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

大家好，我是明哥哥~

前两天，我发现有个前同事写的 Shell 脚本经常在出问题，考虑这个脚本一直挺不稳定的，维护起来也挺头疼，原因是 Shell 脚本写稍微复杂一点的逻辑，代码就变得十分臃肿，对比 Python 真的太差劲了。

这个 Shell 脚本中有一个功能是检查机器上的 rpm 包与中心端的包版本进行对比，在本地用 Shell 取 rpm 信息很方便，但要取 rpm 包版本，其实是很难的。

原因是 rpm 包的版本格式分非常多种，根本无法使用简单的字符串分割来取得具体的版本号，更不用说版本对比。

在重写这个功能的时候，我在找到了能直接获取 Linux 机器 rpm 包的 Python 接口库，这个库要使用 yum 进行安装

装上之后，就可以直接导入使用。由于不是通过 pip 安装的，因此 rpm-python 是安装在 `/usr/lib64/python2.7/site-packages/` 目录下的。

```
>>> import rpm
>>> rpm.__path__
['/usr/lib64/python2.7/site-packages/rpm']
```

# 1. 问题来了
---------

接口库找到了，也安装上了，可问题是。。。

**该怎么用？？？**

你以为百度一下就知道了？

百度出来的，关于这个库的介绍几乎没有。找到的几个也不知所云，完全 get 不到逻辑。

![](https://mmbiz.qpic.cn/mmbiz_png/QB6G4ZoE185AuRII3ncniaicCRnyLoE5oMGBicRvgcHcu5rr48jgPCxJHp8QEQ7VmYIAIxbLnsAK1ysJGrOMFu9fA/640?wx_fmt=png)

于是我尝试着去该库的 pypi 和 github 上，希望找到一些 demo 啥的，先入个门。

看来是我想多了，要啥没啥，一片空白。。

![](https://mmbiz.qpic.cn/mmbiz_png/QB6G4ZoE185AuRII3ncniaicCRnyLoE5oMSLkxLuChgNCVTib2XZN4icSFFwymibMntjoa5ohPDV16St9EFQ4voaDZQ/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_png/QB6G4ZoE185AuRII3ncniaicCRnyLoE5oMuf3iahPEsCd014p2lqQLUnzictYibHzoTYk3BtJRUSOfkBZm9fuWdGgicg/640?wx_fmt=png)

# 2. 神奇的网站
----------

好在 Google 上还是有点用的，它把一个神奇的网站推送到了我的面前，这个网站，就是今天我要为你介绍的主角。

网站的主界面如下，站如其名啊，就是通过 **代码示例** 来让你学习各种库的使用。

![](https://mmbiz.qpic.cn/mmbiz_png/QB6G4ZoE185AuRII3ncniaicCRnyLoE5oMq4Uuw2yC1R0L7YCiaDBpLCFt85V9jkNW19hZuKkv1wcPfYLHibgtWNzA/640?wx_fmt=png)

整个网站非常的简洁，只有一个搜索框，在这个搜索框里输入你想要学习的 python 库，就会立马为你找到该库的用法示例，并且会查到当前有多少的开源项目在使用它。

![](https://mmbiz.qpic.cn/mmbiz_png/QB6G4ZoE185AuRII3ncniaicCRnyLoE5oMrEG36sfQ0icvYiaMSBiaIkReabnvqt44iaoIs0lSH1q08WvX8PPBfRwRNA/640?wx_fmt=png)

很明显上面的第二个包，才是我们需要的东西，点进去后，你会发现一个全新的世界。

在你面前的是一个又一个的完整的代码示例，这些示例系统、全面，非常适合初次学习阶段的理解。

需要强调的是，这些示例全部摘自开源项目，然后按照每个示例上方的链接转到原始项目或源文件。

若你觉得有些示例的代码写得不错，对你也有帮助的，可以给它点个赞。。

![](https://mmbiz.qpic.cn/mmbiz_png/QB6G4ZoE185AuRII3ncniaicCRnyLoE5oMz5mlqGdrCdnY5qUib4xdAJugPWCRBty0yphFSpvLtV8licnmSxqMOGGA/640?wx_fmt=png)

就以 `rpm` 库为例，来感受一下。

**获取已安装的所有所有 rpm 包**

![](https://mmbiz.qpic.cn/mmbiz_png/QB6G4ZoE185AuRII3ncniaicCRnyLoE5oM8fs0ewGfg59qxia8ia8T7zCBqc5HsSK5vRxE1aicQ0zDaarmb7R0p35Bw/640?wx_fmt=png)

**检查一个库是否已经安装过？**

![](https://mmbiz.qpic.cn/mmbiz_png/QB6G4ZoE185AuRII3ncniaicCRnyLoE5oMATATyBiaBhIhHlibqVeoA8WibtGWrYz8YalFicfzjKHOXXiaiaVcTjlicg4rg/640?wx_fmt=png)

**如何根据关键词搜索指定的包并精准获取其版本**

![](https://mmbiz.qpic.cn/mmbiz_png/QB6G4ZoE185AuRII3ncniaicCRnyLoE5oM49A5IxHWTwTmHKTdiaOWITgpQt4WYiaEySqNqerUPUC5Lchrdc4nianMQ/640?wx_fmt=png)

**如何获取离线 rpm 包的信息**

![](https://mmbiz.qpic.cn/mmbiz_png/QB6G4ZoE185AuRII3ncniaicCRnyLoE5oMTEwgaHZN0tSO9tRdG8wIIBfJmBKT3HyUJWRlNlibIhoVLvZyIXp3biaA/640?wx_fmt=png)

还有挺多示例的，这里就不一一列举了。

参照着上面给出的真实案例，我也整理出我属于我自己的 rpm 包的用法，比全网 90% 的文章都来得清晰易懂

rpm 包无非就分两种：

1.  未安装的 rpm 包
    
2.  已安装的 rpm 包
    

想要获取这两种包，方式是不一样的。

###  获取未安装的 rpm 包信息

```
>>> import rpm
>>> ts = rpm.TransactionSet()
>>> rpmhdr = ts.hdrFromFdno("/root/librbd1-devel-10.2.10-0.el7.centos.x86_64.rpm")
>>> rpmhdr["NAME"]
'librbd1-devel'
>>> rpmhdr["VERSION"]
'10.2.10'
>>> rpmhdr["RELEASE"]
'0.el7.centos'
>>> rpmhdr["ARCH"]
'x86_64'
```

###  获取已安装的 rpm 包信息

```
>>> import rpm
>>> ts = rpm.TransactionSet()
>>> query = ts.dbMatch("name", "librbd1")
>>> query.count()
>>> pkg_info = next(query)
>>> pkg_info["NAME"]
'librbd1'
>>> pkg_info["VERSION"]
'12.2.9.1.1'
>>> pkg_info["RELEASE"]
'0.el7.centos'
>>> pkg_info["ARCH"]
'x86_64'
```

但喝水不忘挖井人，以上都是从这个网站中提炼出来的。

本篇文章的主角并不是 `rpm` 这个库的用法，而是上面这个网站。

**与 Python 官方网站提供的标准库示例不一样（赶紧切点题，不然有人说我标题党了）**，这个网站 ，**不仅涵盖了 Python 的内置库，只要你能说得上名的 Python 库**（当然你自己测试上传到 pypi 的那种库肯定不能算是吧）**应该都在这个网站上找到你对应的代码示例**。

![](https://mmbiz.qpic.cn/mmbiz_png/QB6G4ZoE185AuRII3ncniaicCRnyLoE5oMmic2I9pn9hOG4WRH6SQ2AZia3rY9qB36ickeLV1ibyRYeF0sO6k9Vvfiaww/640?wx_fmt=png)
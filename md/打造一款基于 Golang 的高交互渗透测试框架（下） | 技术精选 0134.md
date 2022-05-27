> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAwMzYxNzc1OA==&mid=2247499692&idx=1&sn=707adc365519d25821ee975b01bfeabb&chksm=9b3ad91dac4d500ba5b92e96887205437ac317e84e0b37034df6e02b0f747e9abca978ffe4e2&scene=132#wechat_redirect)

  

本文约 1900 字，阅读约需 7 分钟。

“为什么要自己写？”

  

“自己写的工具有大公司写的成熟工具好用吗？”

  

这是当初有这想法的时候，大佬们问得最多的问题。其实对我个人而言，只是想学习一下 Go 语言，同时为开源社区做做贡献而已。

  

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s56pWiaI9aSleMcoDiafiborNDkaXVyFGNzNCER9TeKlBPjxKoPYYFVzagGrbtOMVj4PfFPM3zv5a2CKA/640?wx_fmt=png)

  

然而，在日常渗透测试时，时常发现有些工具用着不很顺手——参数太多记不住、想用的功能还要收费、明明存在问题的地方用工具测试居然报错、不支持跨平台......

  

这些不方便的地方又增强了自己写一个工具的想法。然而，在随后边学边写的过程中，发现自己写了个 BUG 出来。

  

  

上一期，我们讲到了这个 BUG 的 CLI 和基础扫描模块。这一期，我们继续讲一下它的弱口令爆破模块以及 POC 模块，当然，还有**项目的开源地址**。

  



-------

**1**

  

**弱口令爆破模块**

  

代码结构：

  

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54Kt7S4QsQsJY16WyPdtdOxBAxfXAI8KeAwQfXTON4wvWjW5X3Yl5BAza1fLAnU0zX9XYTrbInmsg/640?wx_fmt=png)

  

目前支持 "RDP","JAVADEBUG","REDIS","FTP", "SNMP", "POSTGRESQL","SSH","MONGO","SMB","MSSQL","MYSQL","ELASTICSEARCH" 这些协议的弱口令，实现方式大体相同，以 FTP 服务进行举例。

  

首先会从数据库中获取将要扫描的 IP：

  

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54Kt7S4QsQsJY16WyPdtdOxXt78HnR3zQQ1me3EHCZvBvEs8V7cX2VfYJQmlOnVaiauicprSQZ03Wibw/640?wx_fmt=png)

  

读取用户名字典（user.txt）和密码字典（pass.txt），随后进行爆破：

  

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54Kt7S4QsQsJY16WyPdtdOxB1NDic675qPib2EadkianLESbZJ3DCLMZicatsNgicnhdXphME8hQo2s2JA/640?wx_fmt=png)

  

一开始是直接执行，扫描完成后输出结果，后来同事看到我敲完命令后没有动静，以为是卡死了，搞得很尴尬。从那以后，这一点变成了需求——要对用户有反馈。

  

那就增加个进度条吧：

  

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54Kt7S4QsQsJY16WyPdtdOxf8MmH64KnAHGibWbcF2HY18k851q7WPG0NOgz4FMbib77cCibUQ23ib15w/640?wx_fmt=png)

  

**2**

  

**POC 模块**

  

**代码结构：**

  

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54Kt7S4QsQsJY16WyPdtdOxI00YNFbgdrBFmvwZ2JrAjxnT9IOb6RY7cAfa8WibgIRiaPfwR12gmVKA/640?wx_fmt=png)

  

**框架对比：**

  

搜索网上开源 POC 框架时，选取了三个可以支持 Go 插件的框架，xray、goby 和 kunpeng，通过对比发现插件的类型主要有三种：Yaml、Go 和 Json 类型：

  

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54Kt7S4QsQsJY16WyPdtdOxOEx7ENl9caoru7UaDWTXJPyoAcn5BKpwpvRuAmo5evsVryYAeZ1ZsQ/640?wx_fmt=png)

  

对比各种插件能否动态加载，发现不管是 Yaml、Go 还是 Json 写的插件，都有动态加载的方式实现：

  

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54Kt7S4QsQsJY16WyPdtdOxUb65ibfjCQ9UkXfcT273NnFYktTV5c6hxTr7F2PG0bWLE1NsBtFlGEw/640?wx_fmt=png)

### 

  

### **Go 插件**

  

首先看一下 Go 插件的动态加载，通过搜索网上的资料，Golang 在 1.8 版本之后提供了一个 GoPlugins 的插件机制，可以动态的加载 so 文件，实现插件化。

  

本以为这样就可以解决 Go 插件动态加载了，然而自己还是太年轻，看到大家的友情提示，才发现这个东西不支持 Windows。

  

  

网上还有一种思路，是实现一个 Go 解释器。因为 Golang 不支持编译后动态加载插件的原因是因为它是编译性语言，对 Go 文件需要编译后才能运行。内置一个动态解析运行 Go 脚本的 Go 解释器，就能够在不使用其他语言的情况下，实现动态加载 Go 插件。

  

然而理想很美好，现实很残酷，由于精力有限，没有继续深入研究。

  

目前，Go 插件的处理是借鉴 kunpeng 的加载方式，将 Go 插件在编译时利用 interface 机制动态注册：

  

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54Kt7S4QsQsJY16WyPdtdOxiarKDFEibrfpibURetKqosmHQYzk1ric11njzQQTNp3ee7ba83sys8ibHicA/640?wx_fmt=png)

  

Go 插件的编写符合 kunpeng 的编码规则，先自定义一个结构体：

  

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54Kt7S4QsQsJY16WyPdtdOxfNH93iaBwznJppUonCqh6pnJsSJ1sPN9a0Nmib09TTpVsGXA0zYG44tg/640?wx_fmt=png)

  

利用 init 函数进行插件注册，在编译时自动加载：

  

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54Kt7S4QsQsJY16WyPdtdOxic0PDW5Wp60YJLR9EWmWfRzmCw3JPwZMuWuCghtZBr8ubQQRXBLpyDw/640?wx_fmt=png)

  

设置插件基础信息，并将插件信息存入数据库：

  

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54Kt7S4QsQsJY16WyPdtdOxlyMsoTXtribCjpqO3goiczspkX6bzwx6r3kDFrygcaQ8400MeOtEN07g/640?wx_fmt=png)

  

定义返回结果的函数：

  

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54Kt7S4QsQsJY16WyPdtdOxQjLz8NRhXcVBuaCf3CV3zFmMGOzEcOM4kGiciaqOEgaMJFYEx5ba7GJw/640?wx_fmt=png)

  

利用 Check 函数执行 POC 代码：

  

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54Kt7S4QsQsJY16WyPdtdOx60olNYTyiby5m9RYzAuHnpX0TeTyZRwjN3BhP6pVRLiauOc5NqaL4Qzg/640?wx_fmt=png)

### 

  

### **Yaml 插件**

  

对于 Yaml 插件的处理，采用类似 Xray 的处理方式，也利用 cel 表达式进行解析，首先将 Yaml 反序列化到结构体：

  

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54Kt7S4QsQsJY16WyPdtdOxOFgxZZ3xuTZ2kprfsh7hVtlXIqhw1f7ygTID47w6sGyiaz6Zbe7bwfg/640?wx_fmt=png)

  

为了有更好的兼容性，将 Yaml 插件的语法表达为类似 Xray 的格式：

  

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54Kt7S4QsQsJY16WyPdtdOxGtGgAY3b9qhVXCeB52C2zB9nn1Iu4g8XAUYlyn4tQUuj78L3ngTsdQ/640?wx_fmt=png)

  

加载 Yaml 插件：

  

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54Kt7S4QsQsJY16WyPdtdOxYQMnicdBFUNRjTpFibC9DhlgL8XFj7iarwEmOtKKmX5wB0UuozribctGZA/640?wx_fmt=png)

  

set 参数保存全局变量，通过 cel 表达式解析后，定义一个 map 来对这些变量进行保存：

  

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54Kt7S4QsQsJY16WyPdtdOxtTcWA9hNTeSN7YjhWiaqFNA1agpApFZot9KWAjy2h9FXnra1g9gDvgw/640?wx_fmt=png)

  

在 expression 字段中有一个 response 结构体，但是在 cel 中是不能直接使用 golang 的 struct 的，需要用 proto 来做一个转换：

  

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54Kt7S4QsQsJY16WyPdtdOxtYbVnT4u2lHyWcq9gHB8w3Ryib9hx1Zz2zjfu5zeU9QlZnrm9oKRgJA/640?wx_fmt=png)

  

通过 protoc-I . --go_out=. http.proto 生成 Go 文件：

  

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54Kt7S4QsQsJY16WyPdtdOxk7NFVREOM4I4VtLM8Bl44via5FiaGXUpialU6YQciaHYlGgib62CvkSXsbA/640?wx_fmt=png)

  

检查 rule 中设置的参数：

  

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54Kt7S4QsQsJY16WyPdtdOxbZMWrFCibDIB2ErgkMiaQJ2J9AYBCQJ9DCQuQB7bb7KbRApnH0Vpy4Dw/640?wx_fmt=png)

  

对 url 进行访问，执行 POC：

  

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54Kt7S4QsQsJY16WyPdtdOxawjrX7a4ZoaXuVBTVdqdPprUsK4562pVbACfsmqQagSDQDy2ymZwuw/640?wx_fmt=png)

  

如果设置了 search 规则对返回的结果进行正则匹配：

  

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54Kt7S4QsQsJY16WyPdtdOxpTTOWKInF4AlZFWZ7F85pzCkHNgtq5Gx8yUDHJgEI4DxFQXkSRic3VA/640?wx_fmt=png)

  

对执行 POC 后的结果进行判断，是否存在风险：

  

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s54Kt7S4QsQsJY16WyPdtdOxUbiaJ3ugicekDsrnnvPnJuxInXxjZEIVPPvz7tN1VvdKTYZ1TqcbicChg/640?wx_fmt=png)

  

完成后的效果：

  

以 CVE-2022-22965 为例进行演示：

  

![](https://mmbiz.qpic.cn/mmbiz_gif/WTOrX1w0s54Kt7S4QsQsJY16WyPdtdOxsRicVEria8DNra58SDpRdXicBvwd5lKNs3Lu9o40SrbouiboOkPTHX857Q/640?wx_fmt=gif)

  

**3**

  

**项目地址**

  

最后说一下给项目起的名字。给项目起名是个问题，还专门查了一下山海经，发现上古神兽、神器都被各大厂商取了一个遍，实在没想到什么了，索性暂定为 taiji，希望它有无限的可能，也欢迎各位大佬交流使用。  

  

  

**项目地址：**

  

https://github.com/sulab999/Taichi

  

参考资料：

  

```
https://github.com/c-bata/go-prompt
https://github.com/hrvngit/gobuster
https://github.com/jjf012/gopoc
https://github.com/opensec-cn/kunpeng
```

**4**

  

**免责声明  
**

  

该工具仅用于安全自查检测。由于传播、利用此工具所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，作者不为此承担任何责任。

  

作者拥有对此工具的修改和解释权。未经网络安全部门及相关部门允许，请勿善自使用本工具进行任何攻击活动，请勿以任何方式将其用于商业目的。
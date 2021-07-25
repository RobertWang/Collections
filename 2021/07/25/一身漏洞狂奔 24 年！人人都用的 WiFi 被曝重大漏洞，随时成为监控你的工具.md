> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5OTY0NTY1Mg==&mid=2650185018&idx=1&sn=b3ce1630fe868efdf4531661f342881f&chksm=bf3a6581884dec97bfe05b981eac66f281b4af764c598e56e95c465f5e70f4c71d18536d997a&scene=21#wechat_redirect)

##### 转自：量子位（ID：QbitAI）

而且，漏洞波及所有连接 WiFi 的设备。

WiFi 本来已经是和阳光空气一样普遍的东西，已经是当代人生存必要条件之一

但这次被发现的漏洞，却在 WiFi 最底层的协议中，“潜伏” 了 24 年。

漏洞有多严重？
-------

最先发现漏洞的比利时荷语鲁汶大学 Mathy Vanhoef 教授，实况演示了这些漏洞会造成怎样的严重后果。

首先是通过 WiFi 截取关键的账号和密码。

利用漏洞，黑客锁定目标 WiFi，然后 “克隆” 一个特征完全相同的网络。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtCNnmj7eyn1coXlBNnwtAacBdXzrBT3Sg4P8UJ22EAdlZ0cHFnfONA6M7eG822FTBEqyq5Ik76qEw/640?wx_fmt=png)

然后，给受害者发一封链接 WiFi 的认证邮件或短信，其中包含一张 “人畜无害” 的图片，受害者在加载时，会自动收到一个 TCP 包。

而这个 TCP 包，会在原有的 WiFi 协议框架里注入新的帧，受害者下一次打开 WiFi 连接时，就会自动连上假 WiFi。

接着，黑客只需要使用 Wireshark 这种抓包工具，就能截取使用者在网络上收发的信息。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtCNnmj7eyn1coXlBNnwtAacwicD75icPs8EdAozLsarxS5VABZvohiatqt8nutiaV6lp6ibrlB3ucMoCBA/640?wx_fmt=png)

基本上，你在网络上输入账号密码这类操作，相当于 “实况直播” 给了黑客。

当然，这种手段最适合机场、酒店这种公共场合 WiFi。

但是，攻击者也可以多花一些功夫，伪装成网络运营商给家庭 WiFi 用户发邮件。

![](https://mmbiz.qpic.cn/mmbiz_jpg/YicUhk5aAGtCNnmj7eyn1coXlBNnwtAacdeInticMDr8fEXlJFN1Vz2jR1vBwMicOUsw2pBRJXtXDndMNPRMNh1zQ/640?wx_fmt=jpeg)

第二种威胁，是攻击者直接利用 WiFi 远程获取设备使用权限，比如电脑、智能音响，监控摄像等等。

演示中，Vanhoef 以一个可以连接 WiFi 进行远程控制的智能台灯为例。

首先，他先通过使用同一 WiFi 的苹果 Mac 电脑追踪到目标 IP 地址，由于 WiFi 协议中的漏洞，甚至不用知道 WiFi 密码，就能远程操控设备：

‍![](https://mmbiz.qpic.cn/mmbiz_gif/YicUhk5aAGtCNnmj7eyn1coXlBNnwtAacbCFjCv4RJszFPawEzQpOO4Ha1OibpmEg6FPj8y6PgkXuEZG52nWGicrg/640?wx_fmt=gif)

试想一下，如果黑客操控的是家中的智能家居、或智能音箱这类带有摄像录音功能的设备，会有多么可怕的后果。

最后，利用这些漏洞，攻击者还可以实现非常复杂的黑客操作。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtCNnmj7eyn1coXlBNnwtAacm2B0QklcyR0A9yfhjk0ZRU4BLzFoIeKgp8975eWFcpE2MWOicyIyMbQ/640?wx_fmt=png)

因为这些漏洞存在于协议底层，意味着即使不接入公共网络，仅在局域网的设备也面临风险。

比如演示中的目标是一台隔绝于外网的 Win7 系统电脑。

攻击者只要同样接入这个局域网，就能利用漏洞直接击穿路由防火墙，把程序植入目标电脑。

接下来，在受害者电脑上的一举一动，都被实时直播：

![](https://mmbiz.qpic.cn/mmbiz_gif/YicUhk5aAGtCNnmj7eyn1coXlBNnwtAac77UibKsNs5EBaAYtMlFMTPlD0gLBQYWGGB5K9SRxSvfCoBS5o4tzXFw/640?wx_fmt=gif)

而且，攻击者还能远程夺取控制权，或者悄悄植入程序。

###### ‍

![](https://mmbiz.qpic.cn/mmbiz_gif/YicUhk5aAGtCNnmj7eyn1coXlBNnwtAacxxJ9E6IP1pL6cNibVRbX8QSia6ZeAqTicDKHxU1v0Hwgq6NYZiaN1ppo4Q/640?wx_fmt=gif)

###### **△**演示中远程打开了系统的计算器  

如此危险的漏洞，到底是怎么出现的呢？

“潜伏”24 年的漏洞
-----------

这次发现的漏洞，涉及基本所有的 WiFi 安全协议，包括最新的 WPA3 规范。

甚至 WiFi 的原始安全协议，即 WEP，也在其中。

这意味着，这几个设计缺陷自 1997 年发布以来一直 “潜伏” 在 WiFi 中。

‍![](https://mmbiz.qpic.cn/mmbiz_jpg/YicUhk5aAGtCNnmj7eyn1coXlBNnwtAacHkEv79cFpemDTwx3YpIp4v4Yl7Uw1rcv4MJSozFgp8RD5oqBXWh2Xg/640?wx_fmt=jpeg)‍

但幸运的是，这些缺陷并不是那么容易被利用，因为这样做需要用户互动，或者在不常见的网络设置时才可能。

所以这也是为什么这些漏洞能潜伏 24 年之久。

所以，在实践中，最大的隐患来自 WiFi 产品中设计缺陷。

#### 纯文本注入漏洞

黑客可以轻松地将帧注入到受保护的 Wi-Fi 网络中。

攻击者通常会精心构建一个框架来注入未加密的 WiFi。

针对路由器，也可以被滥用来绕过防火墙。

而在实际中，某些 Wi-Fi 设备允许接受任何未加密的帧，即使连接的是受保护的 WiFi 网络。

这意味着攻击者不需要做任何特别的事情。此外，许多 Windows 上的 WiFi 加密在被分割成几个（明文）片段时，也会错误地接受明文帧。

#### 帧聚合漏洞

帧聚合功能本来是将小帧合并成一个较大的聚合帧来提高网络的速度和吞吐量。

为了实现这一功能，每个帧都包含一个标志，表示加密传输的数据是否包含一个单一的或聚合的帧。

‍![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtCNnmj7eyn1coXlBNnwtAacchTPX3Ab6NHAxhYLDqrURGPpNTydygFibKia9OibrSmagWUlDbayVJvLA/640?wx_fmt=png)‍

‍但这个 “聚合 “标志没有经过验证，可以被对手修改，这意味着受害者可以被欺骗，以一种非预期的方式处理加密传输的数据。

#### 帧碎片功能漏洞

第二个缺陷是 Wi-Fi 的帧碎片功能。

该功能通过将大的帧分割成较小的片段来增加连接的可靠性。‍

‍当这样做时，属于同一帧的每个片段都使用相同的密钥进行加密。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtCNnmj7eyn1coXlBNnwtAacia1GKuqXlfcFwg6umzricUGtEZUmkGVAuAAzEUYbbILUBXTdkaaLwHdg/640?wx_fmt=png)

然而，接收者并不需要检查这一点，他们会重新组合使用不同密钥解密的片段。在极少数情况下，这可以被滥用来渗出数据。

#### 路由器端验证信息缺失

一些路由器会将握手帧转发给终端，即使来源还没有任何认证。

这个漏洞允许对手进行聚合攻击，并注入任意的帧，而无需用户互动。

此外，还有另一个极其常见的漏洞，接收端也从不检查收到的所有的片段是否属于同一个框架，这意味着对手可以通过混合两个不同框架的手段来伪造信息。

最后，市面上还有一些设备将碎片帧作为全帧处理，这样的缺陷可以被滥用来注入数据包。

怎么办？
----

WiFi 底层协议带着一身漏洞狂奔了 24 年，如今无数设备都在使用。

再想从底层协议开始改，要付出的成本和工作量不可想象。

Mathy Vanhoef 专门开发出了测试工具，可以检验设备是否存在前面所说的漏洞。

还在 Github 上贴出了所有漏洞标识符。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtCNnmj7eyn1coXlBNnwtAacYGBHweptxzod2LbqGmuCrL4Cf8EOeqYJy53Eoag0WfYvk9rjUKpgSA/640?wx_fmt=png)

既然不能改 WiFi 协议，唯一的方法就是升级设备了。

目前升级程序还在制作中，不久就会放出来。

**漏洞列表：**

https://github.com/vanhoefm/fragattacks/blob/master/SUMMARY.md

**漏洞测试工具：**

https://github.com/vanhoefm/fragattacks
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/JiHFBaVSwfyFrl0yChdaFA)

来源丨经授权转自 码农翻身（ID：coderising）

作者丨码农翻身刘欣

**文末包邮送书~**

1997 年，刚刚成立网易的丁磊注意到互联网上出现了一个神奇的服务：Hotmail。

![](https://mmbiz.qpic.cn/mmbiz_jpg/KyXfCrME6UJ3NMy8KXiajBm67U8Mlhs1K8GFYNqh9bA4mfd2zwRHG1MoWs5JJ3qtuic0IubMSYCYIsricVZERQm8w/640?wx_fmt=jpeg)

Hotmail 和之前的电子邮箱不同，一是免费，二是可以用浏览器访问。

丁磊嗅到了未来巨大的商机，就准备借 10 万美元买下它。

但是 Hotmail 公司根本不想卖，开了一个丁磊根本买不起的天价。

于是丁磊决定自己开发一套电子邮箱系统，他选定 FreeBSD 做服务器的操作系统。

7 个月后，电子邮箱系统开发完成，第一套系统以 100 多万卖给了广州电信，并且免费赠送了一个域名：163.net。

163.net 一炮走红，每天都有 2000 多用户注册，很快就达到了 30 万用户。

随后首都在线，金陵在线，商都信息港，国中网等陆续开通，通过销售电子邮箱系统，到 1998 年年底，网易有了 400 万的利润。

丁磊赚到了人生的第一桶金。

作为程序员，在感慨丁磊商业眼光的同时，可能会注意到丁磊选择了一个 “奇怪” 的操作系统 FreeBSD？为什么不用 Linux 呢?

![](https://mmbiz.qpic.cn/mmbiz_png/KyXfCrME6UJ3NMy8KXiajBm67U8Mlhs1KoExslyL4MjjlBjPIia2MGOXVWsJQ5tmhAcLxyn6nMCgWGMALriar5mUw/640?wx_fmt=png)

因为在那个时候，Linux 还没有形成气候，还没有在服务器端的商业领域证明自己的价值。

当时 PC 端的操作系统霸主肯定是 Windows，在服务器端，Unix 则当仁不让。

和 Windows 不同的是，Unix 在各个 IT 巨头的支持下，有很多版本。

Sun : Solaris

IBM : AIX

HP : HP-UX

SGI :IRIX

![](https://mmbiz.qpic.cn/mmbiz_jpg/KyXfCrME6UJ3NMy8KXiajBm67U8Mlhs1Ke8cqibyX1xA8zziaiaMQibWiaQ841pVUicEZzickCKweuwsKusgwmRicZEMiaBQ/640?wx_fmt=jpeg)

这些 Unix 功能强大，和巨头们的硬件深度绑定，在电信，银行，证券等领域占据核心地位。

当然，这样一套系统价格也非常感人。

FreeBSD 则不同，它是一个免费的 Unix 操作系统，提供了完整的 TCP/IP 的网络功能，可以稳定运行 www、email、ftp、NFS、Firewall、BBS 等服务。

简单来说，FreeBSD 可以把廉价的 PC 变成先进的、强大的网络服务器。

丁磊在《PC 不只是便宜的工作站》中写道，当时 Hotmail 的 2000 多万用户，就跑在 500 多台 FreeBSD 服务器上。Yahoo 的 50 台服务器用的也都是 FreeBSD。

可见 FreeBSD 在 90 年代末是互联网服务的中流砥柱。

可是，这么强大的，流行的 FreeBSD 为什么败给了 Linux 呢？

2

Ken Thomson 给加州大学伯克利分校带去了 Unix 的火种，而天才的 Bill Joy 接力开发出了 BSD。

（详情参见《[那些神一样的程序员](http://mp.weixin.qq.com/s?__biz=MzAxOTc0NzExNg==&mid=2665518509&idx=1&sn=3beadb181a2246c31b27e57c467834e4&chksm=80d66deeb7a1e4f8f7b147f361f603600b50abdc1f9784eb85e17da93b56236dd4bdbb3bcdca&scene=21#wechat_redirect)》）

![](https://mmbiz.qpic.cn/mmbiz_png/KyXfCrME6UJ3NMy8KXiajBm67U8Mlhs1K2hN4HO0V6nxllbsQCbXoqSLUFaG5HsuH5ObmuV0ZC2bLhj6pls0X6Q/640?wx_fmt=png)

1992 年，AT&T 挥动专利大棒，起诉加州大学，伯克利计算机研究小组没有办法，只好完全抛弃 AT&T Unix 的源码，从头重写。

Lynne Jolitz 和 William Jolitz 这对儿夫妻敏锐地意识到，x86 架构将来会超越 RISC，成为世界的主宰，于是他们把 BSD 移植到了 Intel 80386 的微处理器上，开创了著名的 386BSD。

![](https://mmbiz.qpic.cn/mmbiz_png/KyXfCrME6UIhfQ5xBqrFgicxFMpGrlWoyGfgn0P1EQQQBA0x94UTFtNgjHXsKZnuCAgwwPgRSFPKJA51K6arsibg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/KyXfCrME6UJ3NMy8KXiajBm67U8Mlhs1K1ZcUmJgWURhHoS2TjZnTssqBYP9xJXuE73ltXoeiaBPxzRkqlVH37Jg/640?wx_fmt=png)

Linux 之父 Linus Torvalds 说：“**如果在我创造 Linux 之前 386BSD 已经可用，那么 Linux 可能不会出生。**”

的确，如果 386BSD 就此发展起来，也就没 Linux 什么事了。

Jolitz 夫妇和 Linus 一样，都非常注重操作系统的代码质量，但是他们试图通过自己完成大部分工作来控制质量，这种有点儿精英主义的做法不可避免地使得开发速度很慢，发布周期更慢。

其他的一些贡献者感受到了潜在的冷落，慢慢地分歧产生了，386BSD 开始分裂，最终形成三大分支：FreeBSD，NetBSD 和 OpenBSD。

这其中 FreeBSD 发展得最好，影响力最广。

FreeBSD1.0 1993 年 11 月发布，Linux 1.0 1994 年 3 月发布，两者是前后脚发展起来。

但是 FreeBSD 和 Linux 的社区文化截然不同。

在 Linux 社区，每个人都可以尝试各种 “奇怪的” 或者 “实验性” 的功能，看看那些有真正的价值，这吸引了很多开发人员。

BSD 社区相对保守，更倾向于把现有的技术弄好，而不是尝试革命性的新技术。所以 FreeBSD 稳定、强大，受到了网络管理员，系统管理员的喜爱。

Linux 像一个程序员的游乐场，程序员们在这里乐此不疲，随着时间的推移，Linux 上的软件包越来越多，数量远超 FreeBSD，生态越来越完善。

在产品决策上，FreeBSD 奉行民主制，如果出现争议，则有每两年选举一次的一个小组来解决，集体领导本来是不错的，但就怕达不成共识，不决策，那产品开发势必要延误了。

一个非常典型的案例是，2000 年的时候，FreeBSD 就在讨论放弃古老的 CVS，改用新的版本管理系统。有些人建议用 BitKeeper，有些人建议用 Mercurial，Git，讨论了 8 年，FreeBSD 团队迟迟做不了决定，2008 年，Peter Wemm 强行推进使用 Subversion，这才结束了争论。

相比而言，Linus 这个独裁者就霸道得多，先是用 BitKeeper，后来没法用了，就立刻自己开发 Git，迅速解决问题，效率极高。

后来，Linux 社区出现了 RedHat 这样的厂商，专门做 Linux 发行版的技术支持，再加上 IT 巨头如 IBM、Dell、HP 等直接支持在服务器上运行 Linux，彻底解除了 Linux 在商业领域应用的封印，无数的中小公司敢用 Linux 了！

就这样，胜利的天平慢慢倒向了 Linux。

不仅仅是 FreeBSD，就连强大的微软，专注打压 Linux 20 年，最后也进入了 Linux 的怀抱。

不过，FreeBSD 并没有消亡，许多 IT 公司（例如 IBM、Nokia、Juniper Networks 和 NetApp）都在使用 FreeBSD 来构建他们的产品。PlayStation 和 Nintendo Switch 操作系统也基于 FreeBSD，Netflix、WhatsApp、和 FlightAware 也在大量使用 FreeBSD，对外提供网络服务。

特别值得一提的是，FreeBSD 是 Darwin 不可或缺的一部分，而 Darwin 是 macOS、iOS、iPadOS、watchOS 和 tvOS 的基础。

![](https://mmbiz.qpic.cn/mmbiz_png/KyXfCrME6UJ3NMy8KXiajBm67U8Mlhs1KHiaxI7D9TK1icyZgfvgYZpdV4jM5mjJLgXIkaib9bH2pHPyrJwsl6jRKg/640?wx_fmt=png)

所以，如果你在使用苹果的产品，可以拿起它看一下，FreeBSD 就在其中呢。

本文作者刘欣，著有畅销书《码农翻身》，《半小时漫画计算机》，前 IBM 架构师，领导过多个企业应用架构设计和开发工作；洞察技术本质，擅长用故事去讲解复杂技术。
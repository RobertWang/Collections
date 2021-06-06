> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg3MDYwMzkwMw==&mid=2247484246&idx=1&sn=5a3d3eb97d54f2e88644ad0db2f147d3&chksm=ce8a0e46f9fd8750cbaf1e898094ba789f9d4ab225ef47a06424088458dcc6c9e3915675cf83&mpshare=1&scene=1&srcid=06066S0xwn7DXFRgeILMoRzC&sharer_sharetime=1622991247752&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

来自量子位

大家好，我是你们的 Lab 哥

你知道 GitHub 安全吗？

GitHub 忽然 “开源” 了自己代码的一部分，还将它放在了 GitHub 上。

事件起因是这样的：

TypeScript 的开发者 Resynth 忽然 Po 了篇文章，表示代码托管服务 GitHub 的全部源代码被泄露。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtAea0Xew8lg5hjYWYsIM6B85CIeMUiaFBTVOLQDG7mYv93hz7OtLvKybH1poljf0FVKd1UJgGb1BqA/640?wx_fmt=png)

他表示，在向官方 GitHub DMCA 提交的可疑文件中，一个身份不明的人利用 GitHub 应用程序中的一个漏洞，冒充 GitHub 的 CEO 纳特・弗里德曼（Nat Friedman）上传了机密源代码。

事情一出，在 HN 上激起了网友的热烈讨论，也再次引发了关于 GitHub 安全问题的思考。

网友 lrvick 表示，包括他在内的许多安全人员，早就对 GitHub 上很多相关漏洞进行了公开演示。但除非 “搞出个病毒”，微软根本就不承认这些漏洞的存在。

而且，他早就说过，GitHub 提交签名的部分存在严重的设计缺陷，然而如今这件事发生，他们才引起重视。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtAea0Xew8lg5hjYWYsIM6B8pvbxTpa9osBbVqz9OlWKWOF4MqfMehwlfQGbT9NqjCEclzSK5Xmtcg/640?wx_fmt=png)

所以，这位陌生用户是怎么做到的？

如何伪造成 CEO 本人泄露代码？
-----------------

GitHub 的源代码管理器 Git，并不能有效地防止用户假冒。

Git 的提交方式更接近于电子邮件，这也就意味着，用户可以随意起用户名和填写邮箱，所以做点小手脚也没关系。

—— 除非提交的信息上有 GitHub CEO 弗里德曼的 GPG 签名，否则 Git 在提交信息时，根本不会确认这是不是 CEO 本人的提交。（这次有问题的代码提交，就没有 CEO 本人的签名信息）

> GPG（GNU Privacy Guard）是一个密钥软件，用于加密、签名通信的内容，也可作为管理非对称密码学的密钥。

除非 GPG 签名与邮箱地址相关联，它并不会对提交对象的真伪进行确认。

也就是说，当你提出一个提交请求到 Git 本地仓库时，你就会得到一个代表提交请求的哈希值，可以通过它直接跳转到你的分支。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtAea0Xew8lg5hjYWYsIM6B8icpw3T8QX5Udk6DsyfK9cUHJPu9DQPrjl05dZYbqC815Nnt29ak5l9w/640?wx_fmt=png)

GitHub 类似于一个 Web 应用程序，负责提供浏览器到 Git 底层架构的请求交互，它会将所有的分支保存到一个底层仓库里，哪怕它不以通常的形式出现在在 URL 架构中。

于是，一位陌生的用户提交的文件 “光明正大地” 进入了 GitHub 的 DCMA 存储库，还伪造成了 CEO 弗里德曼的样子。

![](https://mmbiz.qpic.cn/mmbiz_jpg/a7wPU9Eqe9uBHYNtj8ykPFzK2rYkuamGsLegNmQB4wm93lBGiaABoy5yUKPf8PXK1nn6yllLfDe7FHcHDo7XyQA/640?wx_fmt=jpeg)

为了做到这一点，这位陌生用户先是复制了一份 DCMA 存储库、搞个分支出来，便于提交**要泄露的 GitHub 源代码**；

然后，陌生用户伪造了弗里德曼的用户名和邮箱，将它提交了。于是，在 DCMA 存储库里，名为弗里德曼的用户，自己提交了一份 GitHub 源代码。

CEO 回应后，网友却炸了
-------------

对此，GitHub CEO 弗里德曼做出了回应，表示 GitHub 前段时间不小心混淆了一部分源代码给客户，但这不会影响 GitHub 的安全。

他甚至还吟了首勃朗宁的诗：一切都很好，情况也很正常，云雀展翅飞翔，蜗牛在荆棘上爬动，世上一切顺当！

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtAea0Xew8lg5hjYWYsIM6B8IfOS39DOm96WiaGqDuvQTIEw3J2IIsCsTmZNBok31BbiaMtclT7ibmnbQ/640?wx_fmt=png)

但显然，网友们并不在意这段源码是不是 CEO 本人泄露的，相反，这件事情再一次激起了他们针对 “GitHub 开源” 这件事本身的怒火。

> 网友 exabrial：您（指 CEO）认为这是正常情况？你们是不是想通过伪造 / 无效的 DCMA，删掉其他的什么项目？
> 
> CEO 弗里德曼：这边建议您阅读 DCMA 工作原理呢。
> 
> 网友 dannyw：如果 GitHub 真的提倡开源，它就不会是现在这样。据我所知，微软是 RIAA 的成员哦。

网友 dannyw 之所以提到 RIAA（美国唱片业协会），是因为 GitHub 前段时间应 RIAA 的要求，直接删除了 GitHub 上开源的油管视频下载器 **Youtube-dl**。

一石激起千层浪，原本 GitHub 最初删掉的相关项目就 18 个，现在一搜，竟然冒出了 4000 多个。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtAea0Xew8lg5hjYWYsIM6B8V3l6sjR6PBejZ5EMQbyEyKiaZwU4Jbs0P9pDucvS6dFia3XIN5ZZxudA/640?wx_fmt=png)

有开发者称，这次的 “伪造事件” 估计与 Youtube-dl 项目被删有关，也可能是伪造者对微软并不开放 GitHub 源代码的控诉。

关于 GitHub 开源，还得从微软收购 GitHub 后的一系列举动说起。

微软和它的 “开源”
----------

自 2018 年微软收购 GitHub 后，一直声称自己 “致力于开源”。

Resynth 表示：“我们已经从大量商业广告里看到了（微软对开源的热爱），微软打的这些广告，的确让它处在开源开发的最前沿。”

但与微软提倡的 “开源” 理念相对，它直接封禁了好几次社区开源的代码。

闹到最近，就是这次伪造事件导火索的 “Youtube-dl 被封禁事件”。

有开发者表示，想要让 GitHub 开放自己的源码，如今在微软这看来，是绝对不可能的。

Resynth 也表示，由于有闭源软件的存在、以及 Git 的扩张，让 GitHub 看起来更像是一个试图 “包含开源项目” 的平台，**而非开源本身**。

例如，今年 6 月，GitHub 曾经出现过宕机两小时的情况，这期间，成千上万个开源项目无法被访问和使用。

对于这次 GitHub 泄露源码的事件，你怎么看？

**传送门**
-------

_已经走丢的 GitHub 源码网址：  
https://web.archive.org/web/2/https://github.com/github/dmca/tree/565ece486c7c1652754d7b6d2b5ed9cb4097f9d5_

**参考链接**
--------

_https://arstechnica.com/information-technology/2020/11/githubs-source-code-was-leaked-on-github-last-night-sort-of/  
https://www.zdnet.com/article/github-denies-getting-hacked/  
https://resynth1943.net/articles/github-source-code-leak/  
https://news.ycombinator.com/item?id=24994746  
https://www.theverge.com/2020/6/29/21306674/github-down-errors-outage-june-2020_
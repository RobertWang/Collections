> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxOTcxNTIwNQ==&mid=2457958340&idx=2&sn=a86f7422e27858fab440903001eb4376&chksm=8cb7122ebbc09b3839269705f147979f92bf0abe9de0e5f0b5107d14e4c37225ba1beb2e2ec3&mpshare=1&scene=1&srcid=062201qqDRFQNo5w51XjuymW&sharer_sharetime=1624349937505&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

来自量子位

一个 C 文件，就写出一个 Web 服务器。

最近这个软件，在圈里很火。Hackernews 上热度高达 **700+**。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtAeT6r5icUAfEP8qibTt92Ry56kMVSVzHaFEBEol3SnBkgBBJKibQzCKo7E87TQAz3o7vRvJK4qIUPiaA/640?wx_fmt=png)

有网友直接问：他 GitHub 账号是哪个？

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtAeT6r5icUAfEP8qibTt92Ry5gicia7FJFjOQa4tddgATcm3oOibSHFhhsEI4uuiaGgcriaXibXwuVjYCDSRA/640?wx_fmt=png)

但也有网友质疑说，这个源文件得有几万行代码吧。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtAeT6r5icUAfEP8qibTt92Ry5j0XYJZ5yn5xXg4O9KrGXMTrNJTpBQX9CicslzyaLv92Q1hAYtrhic17g/640?wx_fmt=png)

No，No，No！

只有 **2592 行**，而且**完全开源**！于是就有旁友打算回去抄代码。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtAeT6r5icUAfEP8qibTt92Ry5B0ppF9WJq2ASXNpuGrLlg2MbibRguM2TmMqtBKvgJRaibHOic7mHYJCQw/640?wx_fmt=png)

当然，到处还流淌着各种对大佬的仰慕，在这就不一一列举了。（手动狗头）

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtAeT6r5icUAfEP8qibTt92Ry5P9ib3HTXgpzfrx7zlGDMialf49SmHMc5UuszA9cH3tnBc0ceOFZiaWSGA/640?wx_fmt=png)

究竟是谁打造了这一 “精品”，背后到底是何方神圣？

**Richard Hipp**，一个已经 60 岁的技术大牛。

你没有听过他的名字，但你当前使用的手机，一定有几十甚至上千个他开发的数据库 ——**SQLite**。比如，微信的聊天记录就存在那里面。

![](https://mmbiz.qpic.cn/mmbiz_jpg/YicUhk5aAGtAeT6r5icUAfEP8qibTt92Ry5VNlCv9B6dROZcXJsRcljhZS5r4AWhGSFoGp9z0eRiaMpUsAnMsUvdYg/640?wx_fmt=jpeg)

可以说，它是世界上装机最多的数据库，没有之一。

以至于最新的 Web 服务器一出，就有网友高呼：大神写个淘宝吧。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtAeT6r5icUAfEP8qibTt92Ry5BwgUC6uCOcQGao3eJO1MtDGoOPo3k2xlTvQCsum160WOcN7lhjiaLpw/640?wx_fmt=png)

打造世界上使用最广泛的数据库
--------------

说到 Richard Hipp，就不得不提他的成名作：SQLite。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtAeT6r5icUAfEP8qibTt92Ry5PZMC6mX9NGTka3I5uAJInEFOuF3tDzmBicm2Rvw677OwhY69q3l0pSA/640?wx_fmt=png)

SQLite 是一款轻型的数据库，最初版本的大小只有 900KB 左右。

它最大的特点就是**嵌入式**，支持 Windows/Linux/Unix 等等主流的操作系统，同时能够跟很多程序语言相结合，比如 Tcl、C#、PHP、Java 等，还有 ODBC 接口。

所以，SQLite 可以应用在非常多产品中，除了手机 APP、电脑浏览器，甚至连电视机顶盒中也有它的身影。

并且，与同类数据库 Mysql、PostgreSQL 相比，它的运行速度也更快。

**如此强大的数据库，Richard Hipp 是怎么设计出来的呢？**

这还要追溯到 20 多年前，他接下国防公司通用动力的一个项目说起。

当时，他要解决如何在导弹的小型计算机上安装数据库的问题。

美国海军所使用的 Informix 数据库体积太大、无法安装，而且它是一个单独运行的进程，即使想方设法安装成功，运行的效率也不高，甚至还要耗费大量人力来操控。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtAeT6r5icUAfEP8qibTt92Ry57tELCoVlicGdcXbqJj4dlp85Yy2bJJTB0eERTQwFuRYKOW3HyydmRug/640?wx_fmt=png)

由此，Richard 想：“为什么不能直接从磁盘上读取数据呢？”

这样既能提高运行效率，也能减小数据库大小。

于是，他用 C 语言写了一个小文件，它可以被嵌入到任何程序中，并且大小只有 **900KB** 左右！

所以，你就能 Get 为啥这个新服务器，只有一个 C 文件了吧。（一直都很 Richard 风格）

假设要启动导弹上一个 GPS 程序，这个文件只需在其内部创建一个小数据库，就能来管理相应的数据。

第一版 SQLite 就这样诞生了。

之后，Richard 对 SQLite 进行过多次更新。

2001 年刚发布第二版后不久，摩托罗拉就给他打来合作的电话，希望把 SQLite 应用在他们的手机上。

2005 年，Richard 直接开源了 SQLite，并于同年获得 Google O’Reilly 开源奖。

而他最新发布的网络服务器 **Althttpd**，其实从 2004 年就开始运行 SQLite 官网了。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtAeT6r5icUAfEP8qibTt92Ry52giadA1b1IUADXziacicTPib4Cap03KCdVHDVlUAy1oXsoumK5v8KEu3zQ/640?wx_fmt=png)

官网上介绍，Althttpd 的处理能力还可以。

截至到 2018 年，Althttpd 在 SQLite 官网每天处理约 50 万个 HTTP 请求（每秒 5、6 个），每天能够提供 50GB 的内容（约 4.6 兆比特 / 秒）。

网友：真・轮子哥
--------

他学习编程的缘起，还要从中学说起。

大概是在 70 年代左右，当时 Richard 在读 9 年级。一次偶然的经历，让他看到电传打字机，背后都连着一个大型计算机。

大概是这个样子。

![](https://mmbiz.qpic.cn/mmbiz_jpg/YicUhk5aAGtAeT6r5icUAfEP8qibTt92Ry5uQmS7jOh0iaoLkfkFCnDlI1WLOUcLhccJeA5elDAPrvnOfGfLT4akFA/640?wx_fmt=jpeg)

他被震撼到了，于是下定决心说：必须要学会编程。

执行力超强的他，立马就去学校图书馆，将所有关于计算机的书都借出来。

实际上，只有三本。那天晚上，他就将三本书看完了，还开启了学习用 BASIC 编程的旅途。

随后不久，Apple II 出来了。

![](https://mmbiz.qpic.cn/mmbiz_jpg/YicUhk5aAGtAeT6r5icUAfEP8qibTt92Ry5m4oM5vI8ibaL4ol64pEMGVFcKxEndfoqWcg9YS832c2TXp7cy5nsM5g/640?wx_fmt=jpeg)

不太富裕的他，只买了一个主板，然后自己搭建键盘、电源，并将它们全部焊接起来。

这当中还发生了一个小插曲。

第一个主板不能用，他就电话给苹果。联合创始人史蒂夫・沃兹尼亚克就给他寄了另一个主板。

组装成功之后，他就试图在 4K 的内存中编写程序，这里面还包括视频内存。

当时还因为没有显示器，他就调制射频，将它挂在电视机的天线上。

虽然分辨率感人，整个屏幕只有 40 个字符宽，24 行高。

直到现在，他仍然表示：

> It was the most amazing thing in the world.

这样的创造因子，是从他父亲那里继承而来。

Richard 这样形容他的父亲：“他是那种最原始的制造者，比如内燃机什么的”。

而 Richard 则将同样的想法 —— 从零开始创造事物，放在了抽象的东西上。

接触到计算机之后，Richard 喜欢上了编程，原因很简单：不需要用任何具象的材料，就能构建一个完全不同的世界。

事实上，他也一直在付诸实践。

SQLite 之后，他接着写了分布式版本控制系统 **Fossil**、Bug 追踪系统 **CVSTrac**，以及解析器生成器 Lemon。

每次都是因为遇到了一个问题，然后就自己去编写。

因此就有网友戏称：真・轮子哥。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtAeT6r5icUAfEP8qibTt92Ry5XFHPpuIW5ysxqI9EIReNyaym3vbF1l7yhjjSZCM9sRG4NHlIR1MUMw/640?wx_fmt=png)

不过也有网友为他解释：自己写的工具确实更顺手。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtAeT6r5icUAfEP8qibTt92Ry5FIjSE19kzvE76tXicsx5jq0NvY47xrbL0zSVX0XQJkmse4k3AuYwP9Q/640?wx_fmt=png)

但还有比编写 SQLite 更难的事情
-------------------

不过，对于这位大佬来说，还有比编程更难的事情。

那就是让他的妻子 **Ginger Wyrick** 嫁给他 ![](https://mmbiz.qpic.cn/mmbiz_png/uDRkMWLia28iaTQ3w4bKEp3dibSYY3wZtIfceqIVkByJBp4Uzd3sg8dbdSalsTdfUzh5TWABvQoUZqb83XHntLsNw/640?wx_fmt=png)。

![](https://mmbiz.qpic.cn/mmbiz_jpg/YicUhk5aAGtAeT6r5icUAfEP8qibTt92Ry5icP2Lhltm6dJsLfut7oXn1SO1bpJSdxwYXhYRNxZLuvlFOrJNpibPCicw/640?wx_fmt=jpeg)

甚至在结婚之后，公司也改名了 ，**Hipp, Wyrick & Company**。

并将所有股份转让给她。

Richard 在接受采访时调侃，有时候一度不得不从她那购买一半的股票。

这波狗粮我先干为敬，你们随意 ![](https://mmbiz.qpic.cn/mmbiz_png/uDRkMWLia28iaTQ3w4bKEp3dibSYY3wZtIf9UCTWiaPNadVuNoQnUjtMGm7VcLub1V4szYrdLiauuhBTPN4gKXu7t5g/640?wx_fmt=png)。

参考链接：  
[1]https://sqlite.org/althttpd/doc/trunk/althttpd.md  
[2]https://changelog.com/podcast/201  
[3]https://hackernoon.com/the-story-of-dwayne-richard-hipp-and-the-development-of-sqlite-in-1999-yc4v356q

公众号
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/i-YNqJ7A3u4Bg2KecjlUBg)

来源丨经授权转自 码农翻身（ID：coderising）

作者丨码农翻身刘欣

  
世界上使用最流行的软件是什么？

Windows ? Android?  Office ?

都不对！

答案是 SQLite ！一个嵌入式数据库。

![](https://mmbiz.qpic.cn/mmbiz_png/KyXfCrME6UJ8XeZ3XpiaAQAC1pvbTWqGveaHvQqSbNQMZCDjgmqoj7ciakeAINaCScvnpZG0uvzPzgElZnshNKcQ/640?wx_fmt=png)

你可能没听说过它，但是它就在你身边的：

每一台智能手机中（Android 和 iOS），Mac 电脑，Windows 10 电脑。

每一个主要的浏览器中（Chrome, Firefox, Safari）

大部分的机顶盒当中

每个 PHP 和 Python 安装目录中

很多流行的桌面应用 (微信、QQ、 DropBox、 Skype、 iMessage、WhatsApp、 Adobe Acrobat Reader....)

......

不信的话可以在电脑中搜索一下 “*.db”，看看能发现多少个。

SQLite 的发明人是 Richard Hipp。

![](https://mmbiz.qpic.cn/mmbiz_png/KyXfCrME6UJ8XeZ3XpiaAQAC1pvbTWqGvgJkIhh8K6g4gCLbS2q2VBhnYm5OLdcia7fCRW2spcYPSKRubu0c7qsw/640?wx_fmt=png)

Richard 不但写了 SQLite，他还写了另外一个版本控制软件 **Fossil**。

![](https://mmbiz.qpic.cn/mmbiz_png/KyXfCrME6UJ8XeZ3XpiaAQAC1pvbTWqGv8ialgSKqZ7MSyyfHxQB90qiaGxzTD6MQl31k5ibb6Giak8eazcvQC64jFQ/640?wx_fmt=png)

有趣的是 SQLite 项目使用 Fossil 来做版本控制。

而 Fossil 又使用 SQLite 来存储内容。

有点儿鸡生蛋，蛋生鸡的感觉。

我们不仅要问：现在 Git 已经是源码管理系统中最流行的软件，SQLite 为什么不使用 Git，而要另起炉灶呢？

Richard 写了一篇文章《Why SQLite Does Not Use Git》，解释了其中的原因，几个要点如下：

**1. Git 的思维模型过于复杂**

Git 的复杂性分散了人们对于正在开发软件的注意力，Git 用户需要牢记一下所有内容

（1）The working directory

（2）The "index" or staging area

（3）The local head

（4）The local copy of the remote head

（5）The actual remote head

Git 提供了很多命令和选项在所有这些位置之间进行文件移动和比较。

相比而言，Fossil 只需要考虑他们的工作目录和正在处理的 check-in，干扰减少了 60%，每个开发人员的大脑周期是有限的，Fossil 需要的大脑周期更少，从而可以释放智力资源来专注正在开发的软件。

正如一个使用过 Git 和 Fossil 的用户在 HackerNews 上缩写的：

Fossil 让我安心，因为我拥有一切...... 通过一个命令同步到服务器...... 我从来没有通过 Git 获得过这种安心。 

**2. Git 没有提供良好的态势感知能力**

当 Richard 想看看 SQLite 最近发生了什么情况时，他可以使用 Fossil 的 Timeline 功能，在一个屏幕上看到所有更改的摘要，只需几下点击，就可以看到细节信息，甚至用手机也可以，非常方便。

GitHub 和 GitLab 没有提供类似的功能，最接近的是 “Network graph”，但是它渲染起来很慢（除非事先有缓存），并且不提供那么多的细节，移动设备上效果更不好。

GitHub 的 commit 视图不错，有详细信息，速度快，可是每次只能提供显示一个分支，无法轻松知道所有最近的更改。 

很多 Git 用户会使用第三方的 Git 图形查看器，它们需要单独安装和管理，并且很多是特定平台的（例如仅适用于 Mac 的 GitUp），想用这些图形查看器，首先还得同步本地存储库，很麻烦。 

**3. Git 不跟踪历史分支名称**

Git 保留了 commit 序列完整的 DAG，但 branch tag 是本地信息，它不会同步，不会保留，这使得查看历史分支变得非常乏味。

Richard 用一个分支的例子对比了 Git 和 Fossil，Fossil 可以清楚地显示 Branch 开始的位置，什么时候合并回主干，GitHub 则不行，除非使用第三方的工具。

![](https://mmbiz.qpic.cn/mmbiz_png/KyXfCrME6UJ8XeZ3XpiaAQAC1pvbTWqGvKeZoX2iciaC93pzUI9PIibWUYr0SE3A0vhXtCDXpFb8mrVicHR8hibr8R0A/640?wx_fmt=png)

**4. Git 需要更多的管理支持**

Git 是个复杂的软件，建立 Git 服务器并不容易，所以大多数开发人员使用第三方服务如 GitHub 和 GitLab，从而引入额外的依赖项。

相比之下，Fossil 是个独立的二进制软件，包含 GitHub,GitLab 的核心功能，建立一个服务器非常高效，只需几分钟时间就拥有一个带有 wiki、错误跟踪和论坛的社区服务器，为用户提供打包下载，登录管理等功能。

Fossil 对硬件要求很低，可以在 5 美元 / 月的 VPS 或 Raspberry Pi 上正常运行。

**5.Git 提供了糟糕的用户体验**

下面这个 xkcd 的漫画虽然夸张，但是却切中要害。

![](https://mmbiz.qpic.cn/mmbiz_png/KyXfCrME6UJ8XeZ3XpiaAQAC1pvbTWqGvKribYjacc32UItPQcA3tIoOOGrS3WlhKL5bSXQOGCYuI6icAHutCbwzA/640?wx_fmt=png)

说实话，很少人质疑 Git 提供的用户界面不理想，很多底层的实现都展示在了和用户交互的接口中，交互接口设计很糟糕，有个网站甚至专门生成假的 Git 帮助手册：https://git-man-page-generator.lokaltog.net/#ZWR1Y2F0ZSQkaGVhZA==

Richard 的吐槽挺犀利的，但我能感同身受的只有第一点和最后一点：模型复杂，用户体验差。 

我刚开始接触 Git 时也有很强的抵触情绪：项目组就这么几个人，为什么要用分布式的系统？搞什么本地仓库，远程仓库，还得记住各种各样烦人的命令...... 

集中式管理 SVN 它不香吗？ 

用得多了，发现有两个好处：

(1) 在本地有个副本，可以自由地修改，并且能提交到本地的代码仓库中，先把版本管理起来，这是很爽的一件事情。等到合适的时候再 push，什么事情都不耽误。

(2) Git 的分支实在是强，创建分支不像 SVN 那样得复制目录，很轻量级，新特性开发都可以用分支来搞。

当然，代价就是记住，用熟那些复杂的命令。

Richard 是个挺有意思的人，他很喜欢造自己的小工具，喜欢自给自足。

除了 SQLite 和 Fossil 之外，他还开发了一个 Web 服务器 althttpd，这是个小巧，简单，安全，低资源占用的 Web 服务器，现在 sqlite.org 网站就架在它之上，每天处理 50 万个 Http 请求，传输 200G 的数据。

Richard 还开发过一个叫 CVSTrac 的 Bug 跟踪系统，也是使用 SQLite 来存储相关数据。 

所以，Richard 对 Git 的吐槽有为自己产品宣传的成分（至少这篇文章在 HackerNews 中引发了三次大讨论，赚足了眼球，吸尽了流量），但也真的是自己使用觉得觉得不爽的地方。

SQLite 选择了 Fossil，那是因为对 Richard 来说，Fossil 足够了，并且在某些功能上更好，更能满足自己的需求。 

就像他使用自家的 althttpd，而不是 Apache 一样。

但是对于更多的程序员来说，Git 和 GitHub 的生态系统更有效。

这个世界应该是百花齐放的。

本文作者刘欣，著有畅销书《码农翻身》，《半小时漫画计算机》，前 IBM 架构师，领导过多个企业应用架构设计和开发工作；洞察技术本质，擅长用故事去讲解复杂技术。
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIwNDgzMzI3Mg==&mid=2247492939&idx=1&sn=9fecb10d90b7d96d70a5b188d50cdd20&chksm=97388ddca04f04caf0ae54e0df6503b71e7227055c903522f786576a27b147c8479298b0c13b&scene=178&cur_album_id=1571213952619954180#rd)

你，我是小金！今天给大家推荐一个我最近正在学习的 Go 开源项目，一个使用 Go 语言实现的简易版 Redis —— Godis。

目前，Godis 这个项目在 Github 上收获了 2.3k star。

![](https://mmbiz.qpic.cn/mmbiz_png/BcyAypujBVY5qOxsV9wu12HStAUYKPGfpt10lnsib9FRttHHkxiamATjLeG7HOV6iaWQINe4TauMAE1ab17fd71yw/640?wx_fmt=png)

麻雀虽小，五脏俱全！目前这个项目已经支持下面这些功能：

*   支持 string, list, hash, set, sorted set, bitmap 数据结构
    
*   自动过期功能 (TTL)
    
*   发布订阅
    
*   地理位置
    
*   AOF 持久化及 AOF 重写
    
*   加载和导出 RDB 文件
    
*   Multi 命令开启的事务具有`原子性`和`隔离性`. 若在执行过程中遇到错误, godis 会回滚已执行的命令
    
*   内置集群模式. 集群对客户端是透明的, 您可以像使用单机版 redis 一样使用 godis 集群
    

*   `MSET`, `MSETNX`, `DEL`, `Rename`, `RenameNX` 命令在集群模式下原子性执行, 允许 key 在集群的不同节点上
    
*   Multi 命令开启的事务在集群模式下支持在同一个 slot 内执行
    

*   并行引擎, 无需担心您的操作会阻塞整个服务器.
    

如果你正在学习 Redis 的话，这个项目有助于你理解 Redis 的底层运行原理。

如果你正在学习 Go 语言的话，这个项目不仅可以让你对 Go 语言有进一步的认识，还能让你知道如何使用 Go 语言开发高并发中间件。

项目地址：https://github.com/HDT3213/godis 。

项目运行
----

你可以在 Release 页下载 Darwin(MacOS) 和 Linux 版可执行文件，然后使用下面的命令启动：

```
./godis-darwin
./godis-linux


```

![](https://mmbiz.qpic.cn/mmbiz_png/BcyAypujBVY5qOxsV9wu12HStAUYKPGf7ia7rzWmWOaOPCvWXAzasibmpDloIhbHOpB0ImkIL1gu0MaWGLoGF7ag/640?wx_fmt=png)

然后，我们可以使用 redis-cli 或者其它 redis 客户端连接 Godis 服务器 (默认监听 6399 端口)：

```
redis-cli -p 6399


```

![](https://mmbiz.qpic.cn/mmbiz_png/BcyAypujBVY5qOxsV9wu12HStAUYKPGfnReWjxwapX3tzxVWVSfcHfxUmyR1DBCz4hsrOhibuqVW4uW6mTSNjuQ/640?wx_fmt=png)

这个项目的源代码还是比较多的。不过，如果你想要学习的话，也没有关系。

![](https://mmbiz.qpic.cn/mmbiz_png/BcyAypujBVY5qOxsV9wu12HStAUYKPGfOOll0L3HXHic4ZFusKZUEPymJNCga8Sx0Xk3jfB6oM1lclTBbwAlQdg/640?wx_fmt=png)

源码学习
----

我建议你看看作者提供了一系列非常详细的指导教程，一共有 10 多篇文章。

![](https://mmbiz.qpic.cn/mmbiz_png/BcyAypujBVY5qOxsV9wu12HStAUYKPGfryffddJmAYFeClUFnWtrrrA9GxlCVISpGXsDTIpybYmfm1zrBIYxlg/640?wx_fmt=png)

从使用 Go 编写 TCP 服务器，到使用 Go 实现内存数据库再到使用 GeoHash 实现搜索附近的人，再到 RDB 文件实现，都有介绍到。

文章地址：https://www.cnblogs.com/Finley/category/1598973.html 。

推荐
--

用心发掘优质开源项目，欢迎关注，欢迎点赞分享！

历史优质开源项目推荐地址：[Github 掘金计划](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzIwNDgzMzI3Mg==&action=getalbum&album_id=1571213952619954180#wechat_redirect)。

*   [计算机基础](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1635325633234780161&__biz=MzIwNDgzMzI3Mg==#wechat_redirect)：精选计算机基础（操作系统、计算机网络、算法、数据结构）相关的开源项目。
    
*   [神器工具](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzIwNDgzMzI3Mg==&action=getalbum&album_id=1692140336665378820#wechat_redirect) : 一些好用的插件、软件、网站。
    
*   [程序人生](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzIwNDgzMzI3Mg==&action=getalbum&album_id=2084343476975878144#wechat_redirect)：编程经历、英语学习、延寿指南。
    
*   [项目实战](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1632590550748938241&__biz=MzIwNDgzMzI3Mg==#wechat_redirect) ：精选实战类型的开源项目。
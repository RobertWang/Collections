> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIwNDgzMzI3Mg==&mid=2247492829&idx=1&sn=d33f8fef32a84883fc3abeca558cc6db&chksm=97388c4aa04f055ca1fe21d8459d09c5de976e4cc3027f85c20422be331abee3f8bb4b85367a&scene=178&cur_album_id=1571213952619954180#rd)

你好，我是小金。

今天在逛开源社区的时候发现了一个挺有意思的基于 Go 语言开发的论坛系统——bbs-go，推荐一下，希望这个项目对你有帮助。

bbs-go 实现了一个论坛系统必备的几乎所有的功能。

![](https://mmbiz.qpic.cn/mmbiz_png/BcyAypujBVZNsUoHMOrkGelqeQ3xprMibLcPyRS4tWrefHnFycXwQib7ZWkh8jtEAvODmqSMJ1GG5I1Cqk1pjXyQ/640?wx_fmt=png)bbs-go 功能简介

如果你想要学习 Go 语言的话，这个项目就是一个很不错的练手项目，难度适中且涉及到的 Go 知识点比较常见实用。

如果你想自己搭建一个论坛系统的话，也可以对这个项目做二次开发。不过，需要注意的是，这个项目的商用授权需要花钱。

目前，bbs-go 这个项目在 Github 上收获了 2.1k star，被码云官方评为 GVP 项目。

![](https://mmbiz.qpic.cn/mmbiz_png/BcyAypujBVZNsUoHMOrkGelqeQ3xprMib6LymGMjt3cF5GiarJicicf1JnuzpD2Etkmb7MxvQ5wFc2xwoIj3NGt9UA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/BcyAypujBVZNsUoHMOrkGelqeQ3xprMibTz27u6XicajALUxZphymgUfXDKM1Sqd9ZhXficByBGGHlEIFWltFetVQ/640?wx_fmt=png)

技术栈
---

bbs-go 为前后端分离架构：

*   前端 ：用户界面使用 Nuxt.js 进行渲染，后台界面基于 Element UI
    
*   后端：基于 Go 语言开发对应的 API
    

后端具体的技术选型如下：

*   iris (https://github.com/kataras/iris) Go 语言 mvc 框架
    
*   gorm (http://gorm.io/) 最好用的 Go 语言数据库 orm 框架
    
*   resty (https://github.com/go-resty/resty) Go 语言好用的 http-client
    
*   cron (https://github.com/robfig/cron) 定时任务框架
    
*   goquery (https://github.com/PuerkitoBio/goquery) html dom 元素解析
    

效果概览
----

**首页**

![](https://mmbiz.qpic.cn/mmbiz_png/BcyAypujBVZNsUoHMOrkGelqeQ3xprMibNSr835XzbclJv87NudYe85qWxvsdb6t01DC2kicXFicWtySJlk0ZIrIA/640?wx_fmt=png)

手机端也做了适应，显示效果还不错。

![](https://mmbiz.qpic.cn/mmbiz_png/BcyAypujBVZNsUoHMOrkGelqeQ3xprMibcxMbbg1BUf83umoNBkkegjzwdg27cF3iawZboUDwicIh55NXBKeA8ACg/640?wx_fmt=png)

**发表帖子**

![](https://mmbiz.qpic.cn/mmbiz_png/BcyAypujBVZNsUoHMOrkGelqeQ3xprMibhR7FicokticegoLrNos9ibUUzUefibUhiaAao1CMNnb7kHZpXC9QBAu4eUg/640?wx_fmt=png)

**发表动态**

![](https://mmbiz.qpic.cn/mmbiz_png/BcyAypujBVZNsUoHMOrkGelqeQ3xprMibwqXrj3h1jK0PHQdOOX8OYLffw2HibxZvkStvau0UjezFCoT9r8OD4vg/640?wx_fmt=png)

**个人中心**

![](https://mmbiz.qpic.cn/mmbiz_png/BcyAypujBVZNsUoHMOrkGelqeQ3xprMib1Lbf6meAvMX0bnNMLvQyUu3m4icvJD3LYmWVrc9kfAtDg8nLTsGEdhg/640?wx_fmt=png)

用心发掘优质开源项目，欢迎关注，欢迎点赞分享！

历史优质开源项目推荐地址：[Github 掘金计划](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzIwNDgzMzI3Mg==&action=getalbum&album_id=1571213952619954180#wechat_redirect)。

*   [计算机基础](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1635325633234780161&__biz=MzIwNDgzMzI3Mg==#wechat_redirect)：精选计算机基础（操作系统、计算机网络、算法、数据结构）相关的开源项目。
    
*   [神器工具](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzIwNDgzMzI3Mg==&action=getalbum&album_id=1692140336665378820#wechat_redirect) : 一些好用的插件、软件、网站。
    
*   [程序人生](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzIwNDgzMzI3Mg==&action=getalbum&album_id=2084343476975878144#wechat_redirect)：编程经历、英语学习、延寿指南。
    
*   [项目实战](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1632590550748938241&__biz=MzIwNDgzMzI3Mg==#wechat_redirect) ：精选实战类型的开源项目。
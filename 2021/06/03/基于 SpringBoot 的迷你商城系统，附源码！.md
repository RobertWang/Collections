> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIyNTY4NjU0OQ==&mid=2247506503&idx=2&sn=6998f75e5f6388d670303e03cc559687&chksm=e8797f3ddf0ef62b55e7637bac12b84d6b5d2b6e980a3044b262d189cd76ba3fe2cdca2f3a37&mpshare=1&scene=1&srcid=0601nw34Cv3lMbbCigBC8jnj&sharer_sharetime=1622554169791&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

公众号

上次已经给大家分享了一个基于 ssm 的商城系统。  

小伙伴后台私信我：能不能分享一个基于 springboot 的项目源码啊。

迷你天猫商城
======

### 介绍

所有页面均兼容 IE10 及以上现代浏览器。

### 部署方式

1.  项目使用 IntelliJ IDEA 开发，请使用 IntelliJ IDEA 的版本控制检出功能，输入 “https://gitee.com/project_team/Tmall_demo.git” 拉取项目即可。
    
2.  项目数据库为 MySQL 5.7 版本，请在**码云附件**或**问题交流群文件**上下载 SQL 文件并导入到数据库中。
    
3.  使用 IDEA 打开项目后，在 maven 面板刷新项目，下载依赖包。
    
4.  配置数据库连接并启动 SpringBootApplication 即可。
    

### 项目默认运行地址

*   前台地址：http://localhost:8080/tmall
    
*   后台地址：http://localhost:8080/tmall/admin
    

### 注意事项：

1.  后台管理界面的订单图表没有数据为正常现象，该图表显示的为近 7 天的交易额。
    
2.  该项目同时兼容 eclipse，但如有自行扩展代码的意愿，建议使用 IDEA。
    
3.  该项目是几个学生在校合作完成的一个练习项目，目的是让编程初学者和应届毕业生可以参考一下用较少的代码实现一个完整 MVC 模式，Spring Boot 体系的电商项目，相关领域大神们可以给我们建议，让我们做得更好。
    

### 项目界面

*   ##### 后台界面 (部分)
    

![](https://mmbiz.qpic.cn/mmbiz_png/OwiaX7M4K6dEiavfD2x9CGB73k91DOkib3lLPv12TTWYzibLKaeIASKPhptzOycGplGHOf4zXiamG0K09ib2eLy7UJtA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/OwiaX7M4K6dEiavfD2x9CGB73k91DOkib3l8fuZM3lZwbzsiaEXWSeGROficjWViaHzQuylhCicRQq858YIOQenYo83sQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/OwiaX7M4K6dEiavfD2x9CGB73k91DOkib3lwTlCDicicibSX3CQylru4EZb4KINU5yU9G6wSp22PNibraOgYRrspaJXNA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/OwiaX7M4K6dEiavfD2x9CGB73k91DOkib3librn7I8AGEWSCqjaFQX6f6zqC1yAXyMQ0QIuxjLGHicGWy4QdWNibpZZg/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/OwiaX7M4K6dEiavfD2x9CGB73k91DOkib3lMLK8Dib0GreGJGcpNK5KRNHcsia3PWSxxspP5KarxtHU8BibTJBj02u3Q/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/OwiaX7M4K6dEiavfD2x9CGB73k91DOkib3lA7cAoXUCnicA1IwhyYXB0gyKNsxHmdzNmKzUYAlOqg3qTzbVyFUEPpw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/OwiaX7M4K6dEiavfD2x9CGB73k91DOkib3lZAc0Rjwfu5wvpPkhzhJLBfWAI2ricb9djticElNpLDkdCT3TjqSRf4Bw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/OwiaX7M4K6dEiavfD2x9CGB73k91DOkib3lOur0kMibRbhglwvCy0oGlxdJy1r0kkjuHq2tARxu16tRhBSBWLmDmiaw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/OwiaX7M4K6dEiavfD2x9CGB73k91DOkib3lBW28QWXaWyAdv2qT9tAjtBjmicM5xqu5FhVCqRWrxTt9qlzmjAxdibibA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/OwiaX7M4K6dEiavfD2x9CGB73k91DOkib3lxJH6zGaLic22dJrHOKG8etwMxsqcQ8QpmY3qaMibsq9fXichgpbdxDicgw/640?wx_fmt=png)

*   ##### 前台界面 (部分)---
    

![](https://mmbiz.qpic.cn/mmbiz_png/OwiaX7M4K6dEiavfD2x9CGB73k91DOkib3lIUibtTLDtWxB00mjrGDlcpsWc5n4jfoxtEibxUWPZsbDUTGWwtwgL4Sg/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/OwiaX7M4K6dEiavfD2x9CGB73k91DOkib3lJtKgnbLVD3icyTVTXN4sVKgpgbCy9q43WrvR94191CtheQQGhBibDqbw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/OwiaX7M4K6dEiavfD2x9CGB73k91DOkib3lGN6h3fMO83Wx51sRtlnHd77lHgdayDPKqNK1wEjIkDruOZwCicbl9Fg/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/OwiaX7M4K6dEiavfD2x9CGB73k91DOkib3lun1f1kQYQXibyTq2a1EINTdHLLicduKshKOPSNu7XVmogobrDzF1dicaw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/OwiaX7M4K6dEiavfD2x9CGB73k91DOkib3lfcxq5EwZ25SI8sGyXiaaeloJtRfqhMI1GnndVYmQF32Ocf0zqba8YBg/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/OwiaX7M4K6dEiavfD2x9CGB73k91DOkib3lVvmT1rxucFD0nG72VY9GfliapibQDPtXeePNiclicdAnmA6bb500TEHBwg/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/OwiaX7M4K6dEiavfD2x9CGB73k91DOkib3lK1m3SXorib8uv1LBYXW3yVzg9tlE0N0YGTSExaJsYzusPFF64JibtiaNg/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/OwiaX7M4K6dEiavfD2x9CGB73k91DOkib3lMbiaDiaogTtIMdruMrHn4YHDpppXniavo9rwzPibibkkXdRLiaq1ysClicCnQ/640?wx_fmt=png)
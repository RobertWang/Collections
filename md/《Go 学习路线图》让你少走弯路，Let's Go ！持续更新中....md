> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7119123646471208968)*   作者：[宇宙之一粟](https://juejin.cn/user/3526889034751639 "https://juejin.cn/user/3526889034751639") 10 篇
*   作者：[Masters](https://juejin.cn/user/1239904847411406 "https://juejin.cn/user/1239904847411406") 4 篇
*   作者：[二牛 QAQ](https://juejin.cn/user/2344623087289965 "https://juejin.cn/user/2344623087289965") 2 篇
*   作者：[吴德宝 AllenWu](https://juejin.cn/user/1187128287436808 "https://juejin.cn/user/1187128287436808") 2 篇
*   作者：[任沫](https://juejin.cn/user/184373685261719 "https://juejin.cn/user/184373685261719") 1 篇
*   作者：[TodoCoder](https://juejin.cn/user/2472125987040093 "https://juejin.cn/user/2472125987040093") 1 篇
*   作者：[ReganYue](https://juejin.cn/user/3008695929418318 "https://juejin.cn/user/3008695929418318") 1 篇
*   作者：[jy 白了个白](https://juejin.cn/user/3421335915080093 "https://juejin.cn/user/3421335915080093") 1 篇

感谢大家投稿！

概览
==

首先分享了我的**学习经验**：讲一讲 Go 语言为什么值得学习？以及我是如何高效学习 Go 语言的。

然后就是刻意练习了，需要大家和我一样，坚持每天手撸代码，多敲多想：

通过对 **Go 基础篇**的学习，可以从 Go 小白升级成为能用 Go 撸代码的 gopher。

通过对 **Go 进阶篇**的学习，可以从 Go 初级程序员升级为 Go 中级工程师。

通过 **Go PHP JAVA 类比篇**的学习，可以更好的理解 Go 的优势，更好的理解 Go 的设计思想。

**框架篇** 不仅对比了目前主流的 Go 框架，还重点讲解了 GoFrame 框架相关的知识点。

**GoFrame** 是类似 PHP-Laravel, Java-SpringBoot 的 Go 企业级开发框架，非常值得大家学习。

最后，会通过几篇**应用实践**的文章收尾，带大家体验一下用 Go 开发企业项目的快乐。

> 说明：下面的文章没有标注作者信息的是我的文章；标明作者的都是优秀创作者投稿，经过筛选的优质文章。

为什么学 Go?
========

[# Go 在云原生时代的优势](https://juejin.cn/post/7110226868418117639 "https://juejin.cn/post/7110226868418117639") 作者：[# 宇宙之一粟](https://juejin.cn/user/3526889034751639 "https://juejin.cn/user/3526889034751639")

学习经验分享
======

[# 回顾一下我的 Go 学习之旅 | Go 主题月](https://juejin.cn/post/6949109361331568670 "https://juejin.cn/post/6949109361331568670")

[# [译] 如何真正学习 Go 语言](https://juejin.cn/post/7061777327641853960 "https://juejin.cn/post/7061777327641853960") 作者：[# 宇宙之一粟](https://juejin.cn/user/3526889034751639 "https://juejin.cn/user/3526889034751639")

[# 写 Go 最近踩的坑 | 日志、内聚和复用、gjson、调整心态](https://juejin.cn/post/7103887534299545613 "https://juejin.cn/post/7103887534299545613")

基础篇
===

数据类型
----

[#【Go 基础】编译、变量、常量、基本数据类型、字符串](https://juejin.cn/post/6942795881049489438/ "https://juejin.cn/post/6942795881049489438/")

[# Go const 和 iota 使用实战](https://juejin.cn/post/7068456474133364743/ "https://juejin.cn/post/7068456474133364743/")

[# Go 基础数据类型使用实战：int float bool](https://juejin.cn/post/7068457967133130782 "https://juejin.cn/post/7068457967133130782")

切片
--

[# Go slice 切片详解和实战](https://juejin.cn/post/7071960543283642404 "https://juejin.cn/post/7071960543283642404")

[# Go slice 切片详解和实战 (2) make append copy](https://juejin.cn/post/7068573594879524894 "https://juejin.cn/post/7068573594879524894")

[# 深入理解 slice 非常硬核！](https://juejin.cn/post/7122495050067476510 "https://juejin.cn/post/7122495050067476510") 作者：[# 二牛 QAQ](https://juejin.cn/user/2344623087289965 "https://juejin.cn/user/2344623087289965")

数组
--

[# Go 数组详解和实战](https://juejin.cn/post/7072165701036802055 "https://juejin.cn/post/7072165701036802055")

[# Go map 详解和实战](https://juejin.cn/post/7072905649708859429 "https://juejin.cn/post/7072905649708859429")

rune
----

[# Go rune 详解和实战](https://juejin.cn/post/7068726981159829541 "https://juejin.cn/post/7068726981159829541")

指针
--

[# Go pointer & switch fallthrough 详解和实战](https://juejin.cn/post/7072502044170387492 "https://juejin.cn/post/7072502044170387492")

流程控制
----

[# go if 判断和 for 循环实战 goto 使用的那些坑](https://juejin.cn/post/7069549500649439245 "https://juejin.cn/post/7069549500649439245")

函数
--

[# Go 函数详解 func 匿名函数 闭包](https://juejin.cn/post/7073279289101123614 "https://juejin.cn/post/7073279289101123614")

ORM
---

[# Go 语言中操作 MySQL 数据库](https://juejin.cn/post/7103531271803912199 "https://juejin.cn/post/7103531271803912199") 作者：[# 宇宙之一粟](https://juejin.cn/user/3526889034751639 "https://juejin.cn/user/3526889034751639")

部署
--

[# 如何优雅的通过 Shell 脚本一键部署 GO 项目到服务器](https://juejin.cn/post/6943843305750970399 "https://juejin.cn/post/6943843305750970399")

扩展包
---

[# Go 时间包 jsontime 深入浅出 如何优雅的对时间进行格式化 ｜Go 主题月](https://juejin.cn/post/6943897665960689678 "https://juejin.cn/post/6943897665960689678")

[# Go 语言 json 包的使用技巧](https://juejin.cn/post/6945023713930641445 "https://juejin.cn/post/6945023713930641445")

[# Go 入门很简单：如何在 Go 中使用日志包](https://juejin.cn/post/7088342353638850567 "https://juejin.cn/post/7088342353638850567") 作者：[# 宇宙之一粟](https://juejin.cn/user/3526889034751639 "https://juejin.cn/user/3526889034751639")

重要概念
----

[# Go 开发 web 必懂的概念和底层原理，通过对比的方式让大家更好的理解 | Go 主题月](https://juejin.cn/post/6950954283068031012 "https://juejin.cn/post/6950954283068031012")

进阶篇
===

协程
--

[# 什么时候用 Goroutine？什么时候用 Channel？](https://juejin.cn/post/6943952470993272845 "https://juejin.cn/post/6943952470993272845")

[# Goroutine 就是协程：进程 线程 协程 各自的概念以及三者的对比分析](https://juejin.cn/post/6950952506176471071 "https://juejin.cn/post/6950952506176471071")

RPC
---

[# Go RPC 入门指南 1：RPC 的使用边界在哪里？如何实现跨语言调用？](https://juejin.cn/post/6946452659159171102 "https://juejin.cn/post/6946452659159171102")

反射
--

[# Golang 的反射 reflect 深入理解和示例](https://juejin.cn/post/6844903559335526407 "https://juejin.cn/post/6844903559335526407") 作者：[吴德宝 AllenWu](https://juejin.cn/user/1187128287436808 "https://juejin.cn/user/1187128287436808")

[# Go 语言中的反射](https://juejin.cn/post/7097534989029343246 "https://juejin.cn/post/7097534989029343246") 作者：[任沫](https://juejin.cn/user/184373685261719 "https://juejin.cn/user/184373685261719")

interface
---------

[# Golang interface 接口深入理解](https://juejin.cn/post/6844903555141222407 "https://juejin.cn/post/6844903555141222407") 作者：[吴德宝 AllenWu](https://juejin.cn/user/1187128287436808 "https://juejin.cn/user/1187128287436808")

错误处理
----

[# Go 函数并发情况的错误处理](https://juejin.cn/post/7114970981872959525 "https://juejin.cn/post/7114970981872959525") 作者：[Masters](https://juejin.cn/user/1239904847411406 "https://juejin.cn/user/1239904847411406")

并发安全
----

[# Go 源码解读 - sync.Map 的实现](https://juejin.cn/post/7068192854761275429 "https://juejin.cn/post/7068192854761275429") 作者：[Masters](https://juejin.cn/user/1239904847411406 "https://juejin.cn/user/1239904847411406")

部署
--

[# Go 打包 部署 优雅的把 Go 项目部署到 Linux 服务器](https://juejin.cn/post/6954309251892248612 "https://juejin.cn/post/6954309251892248612")

规范 & 技巧
-------

[# Go 语言中比较优雅的写法 | 硬核！](https://juejin.cn/post/7082536852590166029 "https://juejin.cn/post/7082536852590166029")

[# 爆肝分享两千字 Go 编程规范](https://juejin.cn/post/7086094606856618014 "https://juejin.cn/post/7086094606856618014")

[# Go 开发技巧和踩坑分享 | 代码结构 调试技巧 配置文件 元数据](https://juejin.cn/post/7102605823003590692 "https://juejin.cn/post/7102605823003590692")

Go 对比 PHP/JAVA/C
================

[# Java VS Go 还在纠结怎么选吗，(资深后端 4000 字带你深度对比)](https://juejin.cn/post/7118866795418615822 "https://juejin.cn/post/7118866795418615822") 作者：[TodoCoder](https://juejin.cn/user/2472125987040093 "https://juejin.cn/user/2472125987040093")

[# 为什么我觉得 GoFrame 的 garray 比 PHP 的 array 还好用？](https://juejin.cn/post/7105012753210802213 "https://juejin.cn/post/7105012753210802213")

[# GoFrame gset 使用入门 | 对比 PHP、Java、Redis](https://juejin.cn/post/7105231214390280200 "https://juejin.cn/post/7105231214390280200")

[# 如何在 Go 代码中运行 C 语言代码](https://juejin.cn/post/7077843585088897037 "https://juejin.cn/post/7077843585088897037") 作者：[# 宇宙之一粟](https://juejin.cn/user/3526889034751639 "https://juejin.cn/user/3526889034751639")

好用的扩展包
======

[# GO 语言框架中如何快速集成日志模块](https://juejin.cn/post/7119390985863299085 "https://juejin.cn/post/7119390985863299085") 作者：[Masters](https://juejin.cn/user/1239904847411406 "https://juejin.cn/user/1239904847411406")

[# Go Web 编程入门：Go pongo2 模板引擎](https://juejin.cn/post/7102001354654089223 "https://juejin.cn/post/7102001354654089223") 作者：[# 宇宙之一粟](https://juejin.cn/user/3526889034751639 "https://juejin.cn/user/3526889034751639")

[# 使用 Gorilla Mux 和 CockroachDB 编写可维护 RESTful API](https://juejin.cn/post/7120256019225116679 "https://juejin.cn/post/7120256019225116679") 作者：[# 宇宙之一粟](https://juejin.cn/user/3526889034751639 "https://juejin.cn/user/3526889034751639")

设计模式
====

[# golang 设计模式 - 单例模式](https://juejin.cn/post/7124720007447052302 "https://juejin.cn/post/7124720007447052302") 作者：[# 二牛 QAQ](https://juejin.cn/user/2344623087289965 "https://juejin.cn/user/2344623087289965")

框架篇
===

学哪个框架？
------

[# Go 主流框架对比：Gin Echo Beego Iris](https://juejin.cn/post/7067347764899741709 "https://juejin.cn/post/7067347764899741709")

[# 非常适合 PHP/JAVA 同学使用的 GO 框架：GoFrame](https://juejin.cn/post/7075098594151235597 "https://juejin.cn/post/7075098594151235597")

[# 12 个值得一看的 Go 开源项目 / 框架](https://juejin.cn/post/7119348879820554247 "https://juejin.cn/post/7119348879820554247") 作者：[ReganYue](https://juejin.cn/user/3008695929418318 "https://juejin.cn/user/3008695929418318")

Gin 框架 & 中间件
------------

[# Go gin 框架封装中间件之 1：用户角色权限管理中间件](https://juejin.cn/post/6943147832937447431 "https://juejin.cn/post/6943147832937447431")

[# Go gin 框架封装中间件之 2：操作日志中间件](https://juejin.cn/post/6943503384729583652 "https://juejin.cn/post/6943503384729583652")

GORM
----

[# Go GORM 是时候升级新版本了 2.0 新特性介绍（1）](https://juejin.cn/post/6945404499850854408 "https://juejin.cn/post/6945404499850854408")

[# Go GORM 是时候升级新版本了 2.0 新特性介绍（2）| Go 主题月](https://juejin.cn/post/6946012224573931528 "https://juejin.cn/post/6946012224573931528")

Echo
----

[# 回声嘹亮 之 Go 的 Echo 框架指南 —— 上手初体验](https://juejin.cn/post/7068555737756073997 "https://juejin.cn/post/7068555737756073997") 作者：[# 宇宙之一粟](https://juejin.cn/user/3526889034751639 "https://juejin.cn/user/3526889034751639")

Beego
-----

[# go-web 框架 - beego 的使用](https://juejin.cn/post/7121536082151211022 "https://juejin.cn/post/7121536082151211022") 作者：[# jy 白了个白](https://juejin.cn/user/3421335915080093 "https://juejin.cn/user/3421335915080093")

GoFrame
-------

### 数据结构

[# 为什么我觉得 GoFrame 的 garray 比 PHP 的 array 还好用？](https://juejin.cn/post/7105012753210802213 "https://juejin.cn/post/7105012753210802213")

[# GoFrame garray 使用实践](https://juejin.cn/post/7090901734247104548 "https://juejin.cn/post/7090901734247104548")

[# GoFrame gset 使用入门 | 对比 PHP、Java、Redis](https://juejin.cn/post/7105231214390280200 "https://juejin.cn/post/7105231214390280200")

[# GoFrame gset 使用实践 | 交差并补集](https://juejin.cn/post/7105572330612457486 "https://juejin.cn/post/7105572330612457486")

[# GoFrame gset 使用技巧总结 | 出栈、子集判断、序列化、遍历修改](https://juejin.cn/post/7106013158279479327 "https://juejin.cn/post/7106013158279479327")

[# GoFrame glist 基础使用和自定义遍历](https://juejin.cn/post/7101515355062796296 "https://juejin.cn/post/7101515355062796296")

[# GoFrame gmap 详解 hashmap、listmap、treemap 使用技巧](https://juejin.cn/post/7101797623484383246 "https://juejin.cn/post/7101797623484383246")

[# GoFrame gtree 使用入门 | 养成读源码的好习惯](https://juejin.cn/post/7106458930057855013 "https://juejin.cn/post/7106458930057855013")

### 类型转换

[# GoFrame 代码优化：使用 gconv 类型转换 避免重复定义 map](https://juejin.cn/post/7081078067682082823 "https://juejin.cn/post/7081078067682082823")

### 通用变量

[# GoFrame 通用类型变量 gvar | 对比 interface{}](https://juejin.cn/post/7106712908326764552 "https://juejin.cn/post/7106712908326764552")

### 数据校验

[# GoFrame 数据校验之校验对象 | 校验结构体](https://juejin.cn/post/7110222819631464485 "https://juejin.cn/post/7110222819631464485")

[# GoFrame 数据校验之校验结果 | Error 接口对象](https://juejin.cn/post/7110952333193773064 "https://juejin.cn/post/7110952333193773064")

[# GoFrame 如何实现顺序性校验](https://juejin.cn/post/7113360526410776583 "https://juejin.cn/post/7113360526410776583")

### 错误处理

[# GoFrame 错误处理的常用方法 & 错误码的使用](https://juejin.cn/post/7112428421392629773 "https://juejin.cn/post/7112428421392629773")

### 上下文

[# GoFrame 如何优雅的共享变量 | Context 的使用](https://juejin.cn/post/7113118741776793636 "https://juejin.cn/post/7113118741776793636")

[# Go 并发编程基础：什么是上下文](https://juejin.cn/post/7123200814402764831 "https://juejin.cn/post/7123200814402764831") 作者：[# 宇宙之一粟](https://juejin.cn/user/3526889034751639 "https://juejin.cn/user/3526889034751639")

### ORM

[# GoFrame ORM 使用实践分享](https://juejin.cn/post/7082278651681013773 "https://juejin.cn/post/7082278651681013773")

[# GoFrame ORM 原生方法 开箱体验 （上）](https://juejin.cn/post/7089980894525521957 "https://juejin.cn/post/7089980894525521957")

[# GoFrame ORM 原生方法 开箱体验 （下）](https://juejin.cn/post/7090358951501365278 "https://juejin.cn/post/7090358951501365278")

[# GoFrame 必知必会之 Scan：类型转换](https://juejin.cn/post/7084569454956249101 "https://juejin.cn/post/7084569454956249101")

### 缓存管理

[# GoFrame 如何优雅的缓存查询结果](https://juejin.cn/post/7109465768445607967 "https://juejin.cn/post/7109465768445607967")

[# GoFrame gcache 使用实践 | 缓存控制 淘汰策略](https://juejin.cn/post/7107986667293638663 "https://juejin.cn/post/7107986667293638663")

[# GoFrame gredis 配置管理 | 配置文件、配置方法的对比](https://juejin.cn/post/7108272085452980261 "https://juejin.cn/post/7108272085452980261")

[# GoFrame gredis 硬核解析 | DoVar、Conn 连接对象、自动序列化](https://juejin.cn/post/7108698563328081928 "https://juejin.cn/post/7108698563328081928")

[# GoFrame gredis 如何优雅的取值和类型转换](https://juejin.cn/post/7109092852101021704 "https://juejin.cn/post/7109092852101021704")

### 协程管理

[# GoFrame gpool 对象复用池 | 对比 sync.pool](https://juejin.cn/post/7102979667925139463 "https://juejin.cn/post/7102979667925139463")

[# goFrame 的 gqueue 详解 | 对比 channel](https://juejin.cn/post/7103502362429358116 "https://juejin.cn/post/7103502362429358116")

[# grpool goroutine 池详解 | 协程管理](https://juejin.cn/post/7104661248213516319 "https://juejin.cn/post/7104661248213516319")

### 避坑指南

[# GoFrame 避坑指南和实践干货](https://juejin.cn/post/7081959981456556068 "https://juejin.cn/post/7081959981456556068")

[# GoFrame 避坑指南和实践干货（2）](https://juejin.cn/post/7085730973836378126 "https://juejin.cn/post/7085730973836378126")

### 性能测试

[# GoFrame grpool 性能测试 | 对比原生 goroutine](https://juejin.cn/post/7110510747498594341 "https://juejin.cn/post/7110510747498594341")

### 调试 & 单元测试

[# Go 本地测试 如何解耦 任务拆解 & 沟通](https://juejin.cn/post/7111325551549218823 "https://juejin.cn/post/7111325551549218823")

[# Go Web 编程入门： 一探优秀测试库 GoConvey](https://juejin.cn/post/7115009937847091214 "https://juejin.cn/post/7115009937847091214") 作者：[# 宇宙之一粟](https://juejin.cn/user/3526889034751639 "https://juejin.cn/user/3526889034751639")

应用实践
====

[# gtoken 替换 jwt 实现 sso 登录 | 带你读源码](https://juejin.cn/post/7099449898529095717 "https://juejin.cn/post/7099449898529095717")

[# gtoken 替换 jwt 实现 sso 登录 | 排雷避坑](https://juejin.cn/post/7102389025050361864 "https://juejin.cn/post/7102389025050361864")

[# Go 对接三方 API 实践](https://juejin.cn/post/7083479769878102030 "https://juejin.cn/post/7083479769878102030")

[# Go 一分钟对接 ElasticSearch 实践](https://juejin.cn/post/7084235852921962503 "https://juejin.cn/post/7084235852921962503")

[# 瞄一眼 clickhouse(附 go demo)](https://juejin.cn/post/7068192828790145060 "https://juejin.cn/post/7068192828790145060") 作者：[Masters](https://juejin.cn/user/1239904847411406 "https://juejin.cn/user/1239904847411406")

Git
===

[# Git 使用实战：多人协同开发，紧急修复线上 bug 的 Git 操作指南。](https://juejin.cn/post/7018771333173477383 "https://juejin.cn/post/7018771333173477383")

[# Git 重命名远程分支 | 操作不规范，亲人两行泪。](https://juejin.cn/post/7104258964732575775 "https://juejin.cn/post/7104258964732575775")

刷题
==

如果你是学生党，没有机会接触商业项目，不用难过。刷力扣是个非常好的选择！

[用 Go 语言刷力扣专栏](https://juejin.cn/column/7070334316957401125 "https://juejin.cn/column/7070334316957401125")

为了方便大家刷 Go 语言的知识点，特意整理了面试题相关的文章：

[# 【狂刷面试题】GO 常见面试题汇总](https://juejin.cn/post/7131717990558466062 "https://juejin.cn/post/7131717990558466062")

最后
==

欢迎大家和我一起学习 Go 语言，有任何和 Go 语言学习相关的问题都可以给我留言。

我会在看到的第一时间回复大家。

**为了让掘友们能够更全面更系统的学习 Go 语言**。

搞一个小活动：`欢迎掘友们把你们优质的Go语言文章在这篇文章底部进行留言，我会把符合条件的文章增加到这篇《Go学习路线图》中。`

**截止到 7 月 31 日，投稿成功数量最多的掘友，送掘金活动同款投影仪一个。**

[技术专题 18 期 - 聊聊 Go 语言框架](https://juejin.cn/post/7117898969866305566 "https://juejin.cn/post/7117898969866305566")
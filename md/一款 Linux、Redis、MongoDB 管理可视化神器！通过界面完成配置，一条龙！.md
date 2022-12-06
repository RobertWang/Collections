> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxNDMwMTMwMw==&mid=2247542412&idx=1&sn=769ae57bb8b0e215cb5636ac5460201e&chksm=9b970794ace08e8253e05e246b79e783b60489f69b83a8147eddbf239b6b0f1fed28e55b68d3&mpshare=1&scene=1&srcid=1030vYc20AYpQTKaktWDFp1E&sharer_sharetime=1667061034268&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

**一、开源项目简介**

基于 DDD 分层实现的 web 版 linux(终端 文件 脚本 进程)、数据库（mysql postgres）、redis(单机 集群)、mongo 统一管理操作平台

**二、开源协议**

使用 Apache-2.0 开源协议

**三、界面展示**

系统核心功能截图

记录操作记录

**机器操作**

状态查看

![](https://mmbiz.qpic.cn/mmbiz_jpg/utD23ZXCiaX2ZBqAMr2Z3hJarFAsy7QnLbTAvqC7cEZBVO32I3Gk2erfYraicMR8u0dAIVBGYicDsUUdwIicnjUDMQ/640?wx_fmt=jpeg)

ssh 终端

![](https://mmbiz.qpic.cn/mmbiz_jpg/utD23ZXCiaX2ZBqAMr2Z3hJarFAsy7QnLnOxHBmuDI8HRnnjXNyxsXHibljcqIhU4DoFmHficH19YJgvnk6NA2mtg/640?wx_fmt=jpeg)

文件操作

![](https://mmbiz.qpic.cn/mmbiz_jpg/utD23ZXCiaX2ZBqAMr2Z3hJarFAsy7QnLGyjbatVLdePZ6ibTQRw4s2oK6FJvIicFWq8CdSougJoMXRhNKuoGicahQ/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/utD23ZXCiaX2ZBqAMr2Z3hJarFAsy7QnLOwxEpWk7XqKZXOKa03Kiatv2FdHc6qZ1RqnsJF1QkKmumB3DyWBgjDA/640?wx_fmt=jpeg)

**数据库操作**

sql 编辑器

![](https://mmbiz.qpic.cn/mmbiz_jpg/utD23ZXCiaX2ZBqAMr2Z3hJarFAsy7QnLYcXkoLRZQy4YwugcnkibpNiaTHz3hJy1BteheQDOk7OJeO3o2I8sZqow/640?wx_fmt=jpeg)

在线增删改查数据

![](https://mmbiz.qpic.cn/mmbiz_jpg/utD23ZXCiaX2ZBqAMr2Z3hJarFAsy7QnLRzCl7jzwzLicTOxmwggY9ebvfux7XOC38xJT5VorRibOe1AIqZ5uGlNg/640?wx_fmt=jpeg)

Redis 操作

![](https://mmbiz.qpic.cn/mmbiz_jpg/utD23ZXCiaX2ZBqAMr2Z3hJarFAsy7QnLWebplicyaS9Lq1CN4hHTdwmIJhMHXhreeQDp2X7XVmbe8h47xWjSBkQ/640?wx_fmt=jpeg)

Mongo 操作

![](https://mmbiz.qpic.cn/mmbiz_jpg/utD23ZXCiaX2ZBqAMr2Z3hJarFAsy7QnLZhicfkcJII26eJWdZ7apN5QH6CY1a6TsbjcEhgVWic4RpxLK9j9iaR0aQ/640?wx_fmt=jpeg)

**系统管理**

账号管理

![](https://mmbiz.qpic.cn/mmbiz_png/utD23ZXCiaX2ZBqAMr2Z3hJarFAsy7QnLEPhSYSSeeKZk4mTFEKSAAQrPbJ6dMJ7UmCVwdm6MxzQm6zhj5XL7iaQ/640?wx_fmt=png)

角色管理

![](https://mmbiz.qpic.cn/mmbiz_png/utD23ZXCiaX2ZBqAMr2Z3hJarFAsy7QnLD2RkgSlibLZmKqlSPG3GKLEibiaAJDseq6LxCK4OZibBfpmZibl1cVYSskA/640?wx_fmt=png)

资源管理

![](https://mmbiz.qpic.cn/mmbiz_png/utD23ZXCiaX2ZBqAMr2Z3hJarFAsy7QnLWFibblhLGjnFxM5nOWPwj5DGuibMExokiayKRrWzvEh7icpWwkmNB99rHg/640?wx_fmt=png)

**四、功能概述**

**功能介绍**

*   linux：ssh 终端，文件查看（可根据常见后缀名高亮显示关键词等）、修改、上传、下载、删除等，脚本管理执行，进程操作，运行状态查看等（可当做堡垒机使用）。
    
*   dbms(目前支持 mysql、postgres)： 可视化数据增删改查，sql 语句提示，表信息、索引信息、建表语句查看，建表等（类似 mini 版 navicat）。
    
*   redis(单机、集群)： 增删改查 redis 数据，redis 基本信息查看，如版本，内存，cpu 等使用情况、集群信息节点查看。
    
*   mongo： 增删改查 mongo 文档数据，数据库、集合状态查看，新建删除集合等。
    
*   支持 ssh tunnel 访问： linux 机器、数据库、redis、mongo 都支持 ssh 隧道访问操作。
    
*   系统管理： 同时拥有完善的账号、角色、资源权限控制等，也可基于该项目进行二次开发作为系统后台系统。
    

**为什么开发这个系统 ?**

方便公司统一管理且更加安全高效地维护管理以及操作相关资源信息，开发测试人员可无需查阅文档或咨询前辈索要 ip 账号密码等资源信息。

解决日常开发人员需要安装各种相应客户端的烦恼（可满足前端，测试等人员 100% 不安装各类客户端如: xshell，navicat，redis desktop 等即可完成对应的资源数据操作。后端开发人员 80% 的操作也可以不依赖以上各类客户端）。

**特点**

简单地基于 DDD(领域驱动设计) 分层架构实现。

对前后端进行了大部分通用功能的封装，使用起来更加简洁，功能逻辑清晰，能快速上手学习开发。

项目使用的 Go 语言开发，使用更小的内存及资源运行更高效的应用，二进制文件部署，方便快捷。

日志记录一些重要操作步骤的出入参及操作人信息等。

**五、技术选型**

发语言与主要框架

前端：typescript、vue3、element-plus

后端：golang、gin、gorm

**六、源码地址**

https://github.com/may-fly/mayfly-go/archive/refs/heads/master.zip
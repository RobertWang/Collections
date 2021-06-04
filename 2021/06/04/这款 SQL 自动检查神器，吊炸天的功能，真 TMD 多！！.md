> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI4NTM1NDgwNw==&mid=2247491836&idx=1&sn=3e90572c45448536242161d2c16e1631&chksm=ebefdea4dc9857b23b96035711ac2196bc6f513ec2fbde207133fb4d705e2c764ae5f86e53c5&mpshare=1&scene=1&srcid=05072Pgi8ujMP0Ofxayj7iUs&sharer_sharetime=1620325353059&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

**大家好，我是磊哥。**
=============

Yearning MySQL 是一个 SQL 语句审核平台。提供查询审计，SQL 审核等多种功能，支持 MySQL ，可以在一定程度上解决运维与开发之间的那一环，功能丰富，代码开源，安装部署容易！

![](https://mmbiz.qpic.cn/mmbiz_jpg/GpcH5Yqqj0nn5mqAnzQph3hVAp2JibeCs9tgkMiblb6W1mVibwQHuEAU0IufM16DX4dpZLx8jczUGczpEQ8VgeCfw/640?wx_fmt=jpeg)

开源地址
====

> [https://gitee.com/cookieYe/Yearning](https://mp.weixin.qq.com/s?__biz=MzU1MzUyMjYzNg==&mid=2247484724&idx=2&sn=b95de726f836b96a1699b00f7af76939&scene=21#wechat_redirect)
> 
> [Yearning  安装地址](https://mp.weixin.qq.com/s?__biz=MzU1MzUyMjYzNg==&mid=2247484724&idx=2&sn=b95de726f836b96a1699b00f7af76939&scene=21#wechat_redirect)
> 
> [https://guide.yearning.io/install.html](https://mp.weixin.qq.com/s?__biz=MzU1MzUyMjYzNg==&mid=2247484724&idx=2&sn=b95de726f836b96a1699b00f7af76939&scene=21#wechat_redirect)

功能介绍
====

[**3、**历史审核记录](https://mp.weixin.qq.com/s?__biz=MzU1MzUyMjYzNg==&mid=2247484724&idx=2&sn=b95de726f836b96a1699b00f7af76939&scene=21#wechat_redirect)

[**5、**推送 E-mail 工单推送钉钉 webhook 机器人工单推送](https://mp.weixin.qq.com/s?__biz=MzU1MzUyMjYzNg==&mid=2247484724&idx=2&sn=b95de726f836b96a1699b00f7af76939&scene=21#wechat_redirect)

[**6、**其他 LDAP 登陆用户权限及管理拼图式细粒度权限划分 (共 12 项独立权限, 可随意组合)](https://mp.weixin.qq.com/s?__biz=MzU1MzUyMjYzNg==&mid=2247484724&idx=2&sn=b95de726f836b96a1699b00f7af76939&scene=21#wechat_redirect)

模块介绍
====

**Dashboard**

[dashboard 主要展示 Yearning 各项数据包括用户数 / 数据源数 / 工单数 / 查询数以及其他图表，个人信息栏内用户可以修改密码 / 邮箱 / 真实姓名，同时可以查看该用户权限以及申请权限](https://mp.weixin.qq.com/s?__biz=MzU1MzUyMjYzNg==&mid=2247484724&idx=2&sn=b95de726f836b96a1699b00f7af76939&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_jpg/WwPkUCFX4x5wdOr2xut4q4A90P1mvAvaaZIqOGpAEUjLp45jxV577IzlbNHOf4WlBF82KluocLXTwdUMyW1U0Q/640?wx_fmt=jpeg)

**我的工单**

展示用户提交的工单信息.，对于执行失败 / 驳回的工单点击详细信息后可以重新修改 sql 并提交

对于执行成功的工单可以查看回滚语句并且快速提交 SQL

![](https://mmbiz.qpic.cn/mmbiz_png/GpcH5Yqqj0nn5mqAnzQph3hVAp2JibeCsQyTHQGNiazaBBJsNpBwP6bhXBTVtzbZYj8X3WBxh7NwKVTDtoSjy0WQ/640?wx_fmt=png)

**工单 DLL**

DDL 相关 SQL 提交审核，查看表结构 / 索引，SQL 语法高亮 / 自动补全

![](https://mmbiz.qpic.cn/mmbiz_jpg/WwPkUCFX4x5wdOr2xut4q4A90P1mvAvaZnaJygvS91wC0GaXRV9ZaCAt6O6YwAIZxd9VgrVXcCer37zkv7uFtg/640?wx_fmt=jpeg)

**DML 审核**

DML 相关 SQL 提交审核，SQL 语法高亮 / 自动补全

![](https://mmbiz.qpic.cn/mmbiz_jpg/WwPkUCFX4x5wdOr2xut4q4A90P1mvAvaFB66rOdrfUv3IusSGTVjTArQMK6tpflmRXYYJmOxlia5m7dJrJjtYSw/640?wx_fmt=jpeg)

**查询**

查询 / 导出数据 SQL 语法高亮 / 自动补全 快速 DML 语句提交

![](https://mmbiz.qpic.cn/mmbiz_jpg/WwPkUCFX4x5wdOr2xut4q4A90P1mvAvaSXaQXrvUbhQeNtvKQwClcuYXhyc0Dpf5Ly6HOujS7EtTw5icj2SnnIw/640?wx_fmt=jpeg)

**工单审核**

DDL/DML 管理员审核并执行

![](https://mmbiz.qpic.cn/mmbiz_jpg/WwPkUCFX4x5wdOr2xut4q4A90P1mvAvax2vPkBwicrwLwl7DJ5sxpX9vFPv8A6cS8H9oNPG6ofhoJiaXWBy4ICicw/640?wx_fmt=jpeg)

**查询审核**

用户查询审核

![](https://mmbiz.qpic.cn/mmbiz_jpg/WwPkUCFX4x5wdOr2xut4q4A90P1mvAvaxaKOnbZJicoOgKUHjRoTdIN6lExZv5ZcyPlMOiciawtiacc8y1kBRP54QA/640?wx_fmt=jpeg)

**权限审核**

用户权限审核

![](https://mmbiz.qpic.cn/mmbiz_jpg/WwPkUCFX4x5wdOr2xut4q4A90P1mvAvaibbv7CgN1NbicviaFUhBD26EVEnEiaCYTiciaiasZsX0k11ICFuiaEjGceOyxw/640?wx_fmt=jpeg)

**用户管理**

创建 / 修改 / 删除用户

![](https://mmbiz.qpic.cn/mmbiz_jpg/WwPkUCFX4x5wdOr2xut4q4A90P1mvAvaSSp5bIQIZia3OrXibwiaZgRCy4onGOHibvnueSsQY5qaAic1CwDMJCxKIhg/640?wx_fmt=jpeg)

**数据库管理**

添加 / 编辑 / 删除 数据源

![](https://mmbiz.qpic.cn/mmbiz_jpg/WwPkUCFX4x5wdOr2xut4q4A90P1mvAvaEkEZmSBslMgMl6dBMq55Lnmfl7Xxib50vFkcOvAdzDALGicPKNG6ONOg/640?wx_fmt=jpeg)

**用户权限**

用户权限修改 / 清空

![](https://mmbiz.qpic.cn/mmbiz_jpg/WwPkUCFX4x5wdOr2xut4q4A90P1mvAvaeiaYUrrI84amy0Lqdzc2aQMdBKkHm2SA9EQb6FbG2OxUEp2bLyUaGsg/640?wx_fmt=jpeg)

**基础设置和进阶设置**

设置消息推送相关信息 包括钉钉机器人 / email，设置 LDAP 相关信息，全局配置信息，全局配置开关

![](https://mmbiz.qpic.cn/mmbiz_jpg/WwPkUCFX4x5wdOr2xut4q4A90P1mvAvaEHuVJpQhcn081fYEV9ssXTtsCTlgs2ibhr7GLdO3Ph1sMibYr1GAKd3g/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/WwPkUCFX4x5wdOr2xut4q4A90P1mvAvaSQg6ibzZjCibIINzdgFzhwtEh6zYcVZecvRtNzdibrEJno2295Z7ckL6w/640?wx_fmt=jpeg)

**审核规则**

设置 SQL 检测规则

![](https://mmbiz.qpic.cn/mmbiz_jpg/WwPkUCFX4x5wdOr2xut4q4A90P1mvAvamdcuCIicWA3jQIueTybav1j4Jae9wNftXVuxSKlzbfYZLwiaEGLIwkTg/640?wx_fmt=jpeg)

审核流程
====

[Yearning 采用二级 / 多级的审核模式, 可根据实际需求变更相关使用流程，执行人角色必须在开启多级审核之后才可指定 (开启请前往设置页面)，如果需要将多级审核改为二级审核, 请先确保所有多级审核的工单都已确认执行。否则未执行工单将无法找回。当多级审核关闭后系统并不会自动将角色为执行人的用户重置角色，请自行重置相应用户角色](https://mp.weixin.qq.com/s?__biz=MzU1MzUyMjYzNg==&mid=2247484724&idx=2&sn=b95de726f836b96a1699b00f7af76939&scene=21#wechat_redirect)

二级审核流程：

[**3、**执行记录将会记录在该管理员用户下](https://mp.weixin.qq.com/s?__biz=MzU1MzUyMjYzNg==&mid=2247484724&idx=2&sn=b95de726f836b96a1699b00f7af76939&scene=21#wechat_redirect)

[**1、**使用人根据自己拥有的权限向对应的工单提交单元 (DDL,DML) 提交工单，](https://mp.weixin.qq.com/s?__biz=MzU1MzUyMjYzNg==&mid=2247484724&idx=2&sn=b95de726f836b96a1699b00f7af76939&scene=21#wechat_redirect)

[**2、**管理员收到消息后在审核工单页面审核该工单请求并同意 / 驳回 对应工单并选择对应执行人（执行人必须是角色为执行人的用户）](https://mp.weixin.qq.com/s?__biz=MzU1MzUyMjYzNg==&mid=2247484724&idx=2&sn=b95de726f836b96a1699b00f7af76939&scene=21#wechat_redirect)

[**3、**执行人收到工单后 执行 / 驳回该工单](https://mp.weixin.qq.com/s?__biz=MzU1MzUyMjYzNg==&mid=2247484724&idx=2&sn=b95de726f836b96a1699b00f7af76939&scene=21#wechat_redirect)

[**4、**执行记录将会记录在该执行人用户下](https://mp.weixin.qq.com/s?__biz=MzU1MzUyMjYzNg==&mid=2247484724&idx=2&sn=b95de726f836b96a1699b00f7af76939&scene=21#wechat_redirect)

安装（这部分可以直接接到码云或者官网查看）
=====================

[Yearning 不依赖于任何第三方 SQL 审核工具作为审核引擎, 内部已自己实现审核 / 回滚相关逻辑。仅依赖 Mysql 数据库。mysql 版本必须 5.7 及以上版本，请事先自行安装完毕且创建 Yearning 库, 字符集应为 UTF-8/UTF8mb4 (仅 Yearning 所需 mysql 版本)Yearning 日志仅输出 error 级别, 没有日志即可认为无运行错误！Yearning 基于 1080p 分辨率开发仅支持 1080p 及以上显示器访问（可到官网下载二进制文件）](https://mp.weixin.qq.com/s?__biz=MzU1MzUyMjYzNg==&mid=2247484724&idx=2&sn=b95de726f836b96a1699b00f7af76939&scene=21#wechat_redirect)

**填写配置文件**

```
cat conf.toml
[Mysql]
Db = "Yearning"
Host = "127.0.0.1"
Port = "3306"
Password = "xxxx"
User = "root"

[General]   #数据库加解密key，只可更改一次。
SecretKey = "dbcjqheupqjsuwsm"

```

**初始化数据库**

./Yearning -m

**启动服务**

默认启动

```
./Yearning -s

```

参数启动

```
./Yearning -s -b "172.27.80.35" -p "8000"

```

打开浏览器 http://172.27.80.35:8000

默认密码：admin/Yearning_admin

**开源地址：**

总结
==

**近期技术热文**

[**第 3 版：互联网大厂面试题**  
](https://mp.weixin.qq.com/s?__biz=MzA3MTUzOTcxOQ==&mid=2452981393&idx=2&sn=9c0005ed9e5119eef7183eb25db0edee&scene=21#wechat_redirect)

[包括 Java 集合、JVM、多线程、并发编程、设计模式、算法调优、Spring 全家桶、Java、MyBatis、ZooKeeper、Dubbo、Elasticsearch、Memcached、MongoDB、Redis、MySQL、RabbitMQ、Kafka、Linux、Netty、Tomcat、Python、HTML、CSS、Vue、React、JavaScript、Android 大数据、阿里巴巴等大厂面试题等、等技术栈！  
](https://mp.weixin.qq.com/s?__biz=MzA3MTUzOTcxOQ==&mid=2452981393&idx=2&sn=9c0005ed9e5119eef7183eb25db0edee&scene=21#wechat_redirect)

[**阅读原文：** **高清** **7701 页大厂面试题  PDF**](https://mp.weixin.qq.com/s?__biz=MzA3MTUzOTcxOQ==&mid=2452981393&idx=2&sn=9c0005ed9e5119eef7183eb25db0edee&scene=21#wechat_redirect)
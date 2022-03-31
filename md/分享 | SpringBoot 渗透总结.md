> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/aMv1lHaSwTlSLNt125U7iw)

> **文章来源 ：****雁行安全团队**

**一、前言**
--------

**二、攻击思路**
----------

### 1.  总体分析

### 2.  版本

![](https://mmbiz.qpic.cn/mmbiz_png/ofBa42GG7SiaJsj5tdmOlia3xW50whFic5XkorSWadicH6cGMqwz20czBEaMicsvsIRicykIvuiac3Cwg4xfwvC4HLjEw/640?wx_fmt=png)

springboot 大版本可以分为 1.x 和 2.x，通过暴露的监控端点可以区分其版本。1.x 版本在监控端点未授权的情况下，默认是监控端点是全部打开的，而在 2.x 版本，可能是官方为了安全着想，默认仅开启了几个无伤大雅的端点。  

下图为 Springboot 1.x 启动时开启的监控端点

![](https://mmbiz.qpic.cn/mmbiz_png/ofBa42GG7SiaJsj5tdmOlia3xW50whFic5XNXXEHRxeaOT3wLnm8H59NrvZTNyjSRmbECHru2ib3j5XiaEMtZOtM0qw/640?wx_fmt=png)

下图为 Springboot 2.x 启动时开启的监控端点  

![](https://mmbiz.qpic.cn/mmbiz_png/ofBa42GG7SiaJsj5tdmOlia3xW50whFic5X4Yl5lpYTdZ6icQ5kSt2OEaqBkloKtf0Wzylt7U70Q0qSZicOmnmpKKmQ/640?wx_fmt=png)

只有当 Endpoint. Shutdown. enabled 属性设置为 true 时才会暴露出其他敏感端点  

![](https://mmbiz.qpic.cn/mmbiz_png/ofBa42GG7SiaJsj5tdmOlia3xW50whFic5XQvekdiaGpkWJNyNpevXFDJaY3rvevvBcFib8lxeIH8MEic7sJ7pbIZ5Hg/640?wx_fmt=png)

### 3.  env 端点攻击  

env 端点是在渗透时候比较重要的一个端点，一些历史 rce 漏洞基本都需要依赖该端点 post 数据给服务器，并且改端点还会暴露系统一些比较敏感的信息。

#### 3.1 获取脱敏敏感信息

该端点对敏感信息会进行脱敏处理，对于获取脱敏敏感信息，主要分为远程请求 vps 和 heapdump 内存中查找。我建议从内存中查找，这样可以避免 vps 地址暴露。其次敏感信息不局限于数据库密码等，还有可能存在邮箱账号，企业微信 apikey，微信小程序 apikey 等，这些都是在项目中确切获取到过的。

如下为某企业微信 apikey，微信有公开的 api 文档，通过这些可以获取目标大量人员信息，甚至是加入到目标的企业微信中。

![](https://mmbiz.qpic.cn/mmbiz_png/ofBa42GG7SiaJsj5tdmOlia3xW50whFic5XC62BXhwJnlHGqH4Ln5DrHAtxDE1vxk8ibtwSjAKToSyjj1OYBNIYlmA/640?wx_fmt=png)

如下为某小程序微信 apikey，使用官方 api 查看该小程序存在大量用户，拥有该小程序的某些控制权限危害还是比较大的。

![](https://mmbiz.qpic.cn/mmbiz_png/ofBa42GG7SiaJsj5tdmOlia3xW50whFic5XFLdgiadg3dB9KXyr7FXobia6RNaLHtsicUXvYfy8JzZNuqXGAGtYSG6OA/640?wx_fmt=png)

对于获取到的数据库连接信息都是内网的，不要认为没有用处，密码和端口还是有一定作用，对于非随机的密码，可以在目标所属 ip 段进行碰撞或者组合密码规则获取数据库权限也是有可能的，在《记一次 hw 打点》文章中也确实获取了外网 redis 的权限。微信 apikey、邮箱账号等权限都是重要信息，可以更加贴近目标，对于接下来的攻击做铺垫。

#### 3.2 env 端点下的 rce

历史 rce 在 github 项目中已经提及七七八八，各位师傅可以下载项目中的靶场环境进行复现，需要注意的一点就是 1.x 和 2.x 中提交数据时的 Content-Type 分别为 application/x-www-form-urlencoded 和 application/json，否则会提交数据失败。

### 4.  httptrace 端点

httptrace 端点可以获取当前 web 访问的请求信息，可能找到未销毁的管理员 cookie 信息，在这里建议判断到一个 web 为 springboot 开发时候不要扫描目录，别问我是怎么知道的（/(ㄒ o ㄒ)/~~，因为这个端点的记录是有上限的，有一次扫描目录后发现所有的记录都是我扫描的记录，可能把有些有用的东西给覆盖了）。

### 5.  gateway 端点

gateway 端点的利用主要是 ssrf，现有文章可能比较少，这边做一个复现。当 gateway 端点存在未授权时，直接访问 gateway 是一个 404 的状态。

![](https://mmbiz.qpic.cn/mmbiz_png/ofBa42GG7SiaJsj5tdmOlia3xW50whFic5XL8icE1m3gwxo64y1MIHLeddSNa9oXJOgVQBBzMwCtK5m9GJyhYibef1w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ofBa42GG7SiaJsj5tdmOlia3xW50whFic5XxUAGx3SbYY0Uq6XBIb847ib90AM6xicDjEYGfU5JtQR9PWNWnfVibTP6A/640?wx_fmt=png)

访问 actuator/gateway/routes 路由，可以看到系统的所有口接口信息  

![](https://mmbiz.qpic.cn/mmbiz_png/ofBa42GG7SiaJsj5tdmOlia3xW50whFic5XMHYCZ7TlgFFGLH2WYibzueygAeVZGqOYaIy9Wz7T9MiaulUZJjyfdR3g/640?wx_fmt=png)

我们可以为该接口添加路由，比如添加个 index 路由，将路由地址设置为百度，状态回显 201 则路由创建成功。

![](https://mmbiz.qpic.cn/mmbiz_png/ofBa42GG7SiaJsj5tdmOlia3xW50whFic5XpicWaDF1kOjGyWHWYp89OluCM6MVIYKDUe4iaMyP22Af2It6UibIr8HjQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ofBa42GG7SiaJsj5tdmOlia3xW50whFic5XsiasUpVjhKMdAyYCboN4pYMMdicvDdc86udNZkA7k1GSdG7gjIu5KbaQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ofBa42GG7SiaJsj5tdmOlia3xW50whFic5Xk8Mpq3oxt8RVW1Jvo2HGTcQYDTUPicAeKzY0LZFbB5ibS1GKZlV7QplA/640?wx_fmt=png)

访问 index 路由  

![](https://mmbiz.qpic.cn/mmbiz_png/ofBa42GG7SiaJsj5tdmOlia3xW50whFic5XSGhEvUicENVXwGBVUsfNMGbl9gGsZhHQkJdvOMnYlNG9CsV6WiaGcE3Q/640?wx_fmt=png)

当然我们也可以删除掉路由，delete /actuator/gateway/routes/index 接口  

![](https://mmbiz.qpic.cn/mmbiz_png/ofBa42GG7SiaJsj5tdmOlia3xW50whFic5Xxjibp52UElw3icWb2icWN2nSGnUjc5k3CcfaF2iaQnTLRB9HwDbSooCxTw/640?wx_fmt=png)

再次进行刷新配置  

![](https://mmbiz.qpic.cn/mmbiz_png/ofBa42GG7SiaJsj5tdmOlia3xW50whFic5XR0voAXldFjuG4y3odRianAIIRjWEcYBicZicuP71pibARMALib8lshTCw8A/640?wx_fmt=png)

查看路由，发现 index 路由已经不在了  

![](https://mmbiz.qpic.cn/mmbiz_png/ofBa42GG7SiaJsj5tdmOlia3xW50whFic5XX6h9UklJwYiayDLBXkmRqUxmpwuicex4Wibib2IibvdfwTBrOrPswvA4SYA/640?wx_fmt=png)

对于 gateway 端点的利用，很多师傅可能脑中会有很多自己的攻击思路，比如钓鱼，如果一个目标域名的站点存在漏洞，那么该钓鱼的可信程度会大大增加。ssrf 基于伪协议的利用暂未实现成功。  

**三、总结**
========

关于 springboot 的总结暂时就是这些，仅是个人的浅显经验。
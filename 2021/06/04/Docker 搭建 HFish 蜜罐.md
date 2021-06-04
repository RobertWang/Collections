> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650459870&idx=3&sn=a7afbd78ea54c5d22b803b098ab82616&chksm=83bbab3ab4cc222ced8ac9bfad83521b1153f9ca3e759865aae510df2d256ba1745b15646e8a&scene=21#wechat_redirect)

  

> **文章来****源：华盟论坛**

HFish 是一款基于 Golang 开发的跨平台多功能主动诱导型开源蜜罐框架系统，为了企业安全防护做出了精心的打造，全程记录黑客攻击手段，实现防护自主化。  
1、多功能 不仅仅支持 HTTP(S) 蜜罐，还支持 SSH、SFTP、Redis、Mysql、FTP、Telnet、暗网 等

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXL9Cd0SCUiaicGwOXMLW3DgMn6Xw4kUkHC1oNqttFTxnG04eJfibQ08lL2oNgZoDudYQBHO4clOkZzRrg/640?wx_fmt=png)

  
docker pull imdevops/hfish

  
主节点管理端部署 docker run -d -it -p 8080:8080 -p 8989:8989 -p 9000:9000 -p 9001:9001 -p 6379:6379 7da65a1950f0 

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXL9Cd0SCUiaicGwOXMLW3DgMn6LGDObKPs3FkKCCrPFdianAFreLvvBsMYfBZFmPOGe7Jo2POWicqiabI9w/640?wx_fmt=png)

  
客户端子节点部署

docker run -d -it  -p 7879:7879 -p 6379:6379 -p 8080:8080 -p 8989:8989 -p 9000:9000 -p 11211:11211 -e CLUSTER_IP=192.168.123.49:6379 -e NODE_NAME=clinet 7da65a1950f0

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXL9Cd0SCUiaicGwOXMLW3DgMn6D0jH4GgZap28xXDGHCX0V4J0DZqYgu5DQRsF9AASzTRg7nUibZpRlrA/640?wx_fmt=png)

以下端口根据实际需要决定是否打开，并注意端口冲突。

*   21 为 FTP 端口
    
*   22 为 SSH 端口
    
*   23 为 Telnet 端口
    
*   3306 为 Mysql 端口
    
*   6379 为 Redis 端口
    
*   8080 为 暗网 端口
    
*   8989 为 插件 端口
    
*   9000 为 Web 端口
    
*   9001 为 系统管理后台 端口
    
*   11211 为 Memcache 端口
    
*   7879 为集群通信 RPC 端口
    
*   环境变量 CLUSTER_IP 为集群主节点 IP 地址
    
*   环境变量 NODE_NAME 为客户端节点名称，集群唯一，不可重名  
    

  
HFish 已经搭建好了，先访问下 web 好像是 Wordpress 管理后台  

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXL9Cd0SCUiaicGwOXMLW3DgMn6Y2w6drftqaSyy2icY7PN2iaQhAMPzyoYuX5LwKTP03wXI5TI7KibKkqow/640?wx_fmt=png)

然后再看看 HFish 后台，端口是 90001

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXL9Cd0SCUiaicGwOXMLW3DgMn6WhQlENibuAZibNu72ibbt1HticcwB5KA04fab933ibuRVUEBicxF3D0aQYFQ/640?wx_fmt=png)

输入密码登入，默认账密都是 admin，映入眼帘的是一个地球有点意思，可以切换显示状态  

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXL9Cd0SCUiaicGwOXMLW3DgMn64ZqdBTaoLk4QmhTPsHUBWgR3KNsFgO6CiakpA2kjRp82m5NtsxhYibUg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXL9Cd0SCUiaicGwOXMLW3DgMn6AfhlzTSGojZNzYicKZaibuZARVs88wfGfJIkUAwD53U0jKyI2BLicju9A/640?wx_fmt=png)

访问一下之前搭建的蜜罐看看是否能监测到。  

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXL9Cd0SCUiaicGwOXMLW3DgMn6Ox8qFJxnyohUfnJkiachvcIKdZaxxTjicibCcjXECw8kvhUra0KQKaKNQ/640?wx_fmt=png)

  
在访问一下管理端 HFish 的 web，也是可以监控的还能显示出登入的账号和密码。

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXL9Cd0SCUiaicGwOXMLW3DgMn6HEmp0pOqeg9kedJcSn2rTR5jOWHaKUXEkrwHMLlVEjskM9VDekIBlA/640?wx_fmt=png)

注意事项

  
1、邮箱 SMTP 配置后需要开启方可使用  
2、API 接口 info 字段，&& 为换行符  
3、启动 WEB 蜜罐，请先启动 API 模块  
4、WEB 插件 需在 WEB 目录下 编写  
5、WEB 插件 下面必须存在两个目录  
6、集群 心跳为 60 秒, 断开显示会延迟 60 秒

推荐文章 ++++

* [蜜罐正式开源 - 简单易用 - 支持 16 种协议](http://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650451948&idx=2&sn=66af6da07a1302beaa8c1bc7c4dcabd1&chksm=83bbca08b4cc431e653aedfb0df50205faa11b05ef92c70f200f710e96323c9682099b0e7f97&scene=21#wechat_redirect)

* [蜜罐学习之 ssh](http://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650448788&idx=3&sn=6885aaf63f37f630908b5ff5e5c32e2d&chksm=83bbc670b4cc4f66ffb1306ebdc0352918d90f1e710b8ed82180dc048785e7eff85cdbd5ac32&scene=21#wechat_redirect)
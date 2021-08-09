> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.linuxe.cn](http://www.linuxe.cn/post-543.html)

> 由于大多数网站的前端都有 CDN 或者负载均衡，这样会导致 Nginx 在获取客户端 IP 的时候看到的是 CDN 的 IP，而非客户端真实 IP。

由于大多数网站的前端都有 CDN 或者负载均衡，这样会导致 Nginx 在获取客户端 IP 的时候看到的是 CDN 的 IP，而非客户端真实 IP。为了解决这个问题需要使用 Nginx 的 realip 模块或者 proxy_set_header 模块，该模块可以从一个指定的请求头中去获取客户端 IP 信息。

**一、Nginx realip 模块的使用**

1、在编译安装 nginx 的时候加上 --with-http_realip_module 即可开启该模块。

2、配置 realip 模块

```
set_real_ip_from CDN_IP;  #设置可信任的IP地址，之后会使用real_ip_header从这些IP中获取请求头信息
real_ip_header  X-Forwarded-For;  #从X-Forwarded-For头中获取IP地址，多个IP存在时最左边的是客户端IP

```

3、防止 IP 地址伪造

由于 X-Forwarded-For 已经是通用请求头，恶意攻击可以伪造该请求头进行攻击，这样无法得知攻击者的真实 IP。如果 Nginx 前面还有 CDN 的话，可以和 CDN 商协定一个自定义请求头，将真实 IP 写进日中

[![](http://www.linuxe.cn/content/uploadfile/201912/thum-a8df1576479326.png)](http://www.linuxe.cn/content/uploadfile/201912/a8df1576479326.png)

二、**Nginx **proxy_set_header** 模块的使用**

配置示例如下：

```
location /{  
    proxy_pass http://localhost:8080/web/;  
    #以下三个proxy_set_header配置项是重点  
    proxy_set_header Host $host;  
    proxy_set_header X-Real-IP $remote_addr;  
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  

```

版权声明

```
本站所有文章均为原创，转载请注明出处！小站维护不易，如果对您有所帮助，希望能点击一下站内广告，谢谢！

```
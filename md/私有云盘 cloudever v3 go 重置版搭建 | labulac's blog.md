> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.labulac.top](https://blog.labulac.top/802e51e1.html)

> 德国免费的小鸡，虽然配置差了点，但是那免费的 “无限” 流量着实有点羡慕，闲着也是闲着，不如拿来做个网盘，配置个 aria2 做下载站 cloudreve v3 go 重置版特性☁️ 支持本机、从机、七牛、阿里云 ......

> 德国免费的小鸡，虽然配置差了点，但是那免费的 “无限” 流量着实有点羡慕，闲着也是闲着，不如拿来做个网盘，配置个 aria2 做下载站

### [](#cloudreve-v3-go重置版特性 "cloudreve v3 go重置版特性")cloudreve v3 go 重置版特性

*   ☁️ 支持本机、从机、七牛、阿里云 OSS、腾讯云 COS、又拍云、OneDrive (包括世纪互联版) 作为存储端
    
*   📤 上传 / 下载 支持客户端直传，支持下载限速
    
*   💾 可对接 Aria2 离线下载
    
*   📚 在线 压缩 / 解压缩、多文件打包下载
    
*   💻 覆盖全部存储策略的 WebDAV 协议支持
    
*   ⚡ 拖拽上传、目录上传、流式上传处理
    
*   🗃️ 文件拖拽管理
    
*   👩‍👧‍👦 多用户、用户组
    
*   🔗 创建文件、目录的分享链接，可设定自动过期
    
*   👁️‍🗨️ 视频、图像、音频、文本、Office 文档在线预览
    
*   🎨 自定义配色、黑暗模式、PWA 应用、全站单页应用
    
*   🚀 All-In-One 打包，开箱即用
    
*   🌈 … …
    

### [](#官方支持的网站和文档 "官方支持的网站和文档")官方支持的网站和文档

*   官网：[https://cloudreve.org/](https://cloudreve.org/)
*   github：[https://github.com/cloudreve/Cloudreve](https://github.com/cloudreve/Cloudreve)
*   下载：[https://github.com/cloudreve/Cloudreve/releases](https://github.com/cloudreve/Cloudreve/releases)
*   安装文档：[https://docs.cloudreve.org/getting-started/install](https://docs.cloudreve.org/getting-started/install)
*   演示：[https://demo.cloudreve.org](https://demo.cloudreve.org/)

### [](#宝塔面板部署 "宝塔面板部署")宝塔面板部署

#### [](#1-面板安全放行5212端口 "1.面板安全放行5212端口")1. 面板安全放行 5212 端口

[![](https://cdn.jsdelivr.net/gh/labulac/pic/picgo/20201012221400.png)](https://cdn.jsdelivr.net/gh/labulac/pic/picgo/20201012221400.png "image-20201012221400577")

[image-20201012221400577](https://cdn.jsdelivr.net/gh/labulac/pic/picgo/20201012221400.png "image-20201012221400577")

#### [](#2-打开ssh，依次执行下面的语句 "2.打开ssh，依次执行下面的语句")2. 打开 ssh，依次执行下面的语句

```
mkdir /www/wwwroot/cloudreve
cd /www/wwwroot/cloudreve
wget https://github.com/cloudreve/Cloudreve/releases/download/3.0.0/cloudreve_3.1.1_linux_amd64.tar.gz
tar -zxvf cloudreve_3.1.1_linux_amd64.tar.gz
chmod +x ./cloudreve
```

#### [](#3-配置永久守护程序 "3.配置永久守护程序")3. 配置永久守护程序

宝塔面板的软件商店下载堡塔应用管理器并点击设置打开

[![](https://cdn.jsdelivr.net/gh/labulac/pic/picgo/20201012222058.png)](https://cdn.jsdelivr.net/gh/labulac/pic/picgo/20201012222058.png "image-20201012222058185")

[image-20201012222058185](https://cdn.jsdelivr.net/gh/labulac/pic/picgo/20201012222058.png "image-20201012222058185")

配置如下参数

[![](https://cdn.jsdelivr.net/gh/labulac/pic/picgo/20201012223746.png)](https://cdn.jsdelivr.net/gh/labulac/pic/picgo/20201012223746.png "image-20201012223515140")

[image-20201012223515140](https://cdn.jsdelivr.net/gh/labulac/pic/picgo/20201012223746.png "image-20201012223515140")

点击启动，查看默认的用户名与密码

[![](https://cdn.jsdelivr.net/gh/labulac/pic/picgo/20201012223442.png)](https://cdn.jsdelivr.net/gh/labulac/pic/picgo/20201012223442.png "image-20201012223442595")

[image-20201012223442595](https://cdn.jsdelivr.net/gh/labulac/pic/picgo/20201012223442.png "image-20201012223442595")

#### [](#4-配置反向代理，使用域名访问 "4.配置反向代理，使用域名访问")4. 配置反向代理，使用域名访问

新建一个网站，配置反向代理如下：

[![](https://cdn.jsdelivr.net/gh/labulac/pic/picgo/20201012223739.png)](https://cdn.jsdelivr.net/gh/labulac/pic/picgo/20201012223739.png "image-20201012223739898")

[image-20201012223739898](https://cdn.jsdelivr.net/gh/labulac/pic/picgo/20201012223739.png "image-20201012223739898")

开启 ssl，使用 https 访问：

[![](https://cdn.jsdelivr.net/gh/labulac/pic/picgo/20201012224022.png)](https://cdn.jsdelivr.net/gh/labulac/pic/picgo/20201012224022.png "image-20201012224022734")

[image-20201012224022734](https://cdn.jsdelivr.net/gh/labulac/pic/picgo/20201012224022.png "image-20201012224022734")

#### [](#5-个人配置文件分享，使用了mysql与redis "5.个人配置文件分享，使用了mysql与redis")5. 个人配置文件分享，使用了 mysql 与 redis

```
[System]
Mode = master
Listen = :5212
SessionSecret = 29vARCK2gS6hE2Poa5tBTke9VXUwGj5Nk2qPm71fEY2dIq3VFN9j
HashIDSalt = KX0V9dq2TJMy7y3JDHBo16jLbdabTOq6o7usm7jw3kf8YiV

[Database]
Type = mysql
User = 111
Password = 123456
Host = 127.0.0.1
Name = 1111
TablePrefix = cd

[Redis]
Server = 127.0.0.1:6379
Password =
DB = 0
```

### [](#最后 "最后")最后

搭建很方便，使用也不错，配合 5T 的 onedrive 和 aria2 离线下载，自动上传，简直香喷喷啊！
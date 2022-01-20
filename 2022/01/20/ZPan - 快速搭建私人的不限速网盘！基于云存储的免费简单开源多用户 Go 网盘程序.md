> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.iplaysoft.com](https://www.iplaysoft.com/zpan.html)

自建网盘程序在异次元上就曾推荐过同步型的 [NextCloud](https://www.iplaysoft.com/nextcloud.html)、[SeaFile](https://www.iplaysoft.com/seafile.html)、[ownCloud](https://www.iplaysoft.com/owncloud.html) 和相对简单型的 [Cloudreve](https://www.iplaysoft.com/cloudreve.html)、[Z-File](https://www.iplaysoft.com/z-file.html)、[EDL](https://www.iplaysoft.com/evoluted-directory-listing.html)、[KOD](https://www.iplaysoft.com/kodexplorer.html) 等。而今天介绍的「**ZPan**」则是又一款追求简约轻量的基于云存储的[网盘](https://www.iplaysoft.com/tag/网盘)程序，并且它致力于打造成 “不限速的网盘系统”。

ZPan 网盘完全[开源](https://www.iplaysoft.com/tag/开源)免费，使用 Go 语言开发，它最大的特色是实现了用户「直连云存储」(比如[阿里云 OSS](https://www.iplaysoft.com/go/aliyunoss) / [腾讯云 COS](https://www.iplaysoft.com/go/qcloudcos) 等)，不受[服务器](https://www.iplaysoft.com/tag/服务器)本身的带宽和流量限制，实现 “**不限速**”且 “不耗服务器流量” 的文件上传和下载。

传统网盘最大问题是存储空间、上传下载速度都会受到服务器带宽和[硬盘](https://www.iplaysoft.com/tag/硬盘)大小的限制，你想要快就只能购买高价高带宽的机器。而 **ZPan** 则不同，它完全 “基于云存储服务” 实现底层文件存储，即便服务器只有 1M 带宽，也能实现几乎满速的文件上传下载，并且也不占用服务器本地的存储空间。

### 功能简洁实用

ZPan 的界面清新简约直观，支持多语言，可以**支持文件及文件夹分享** (可以设置提取码、有效期，允许不登录访问)；支持文档预览，以及音频和[视频](https://www.iplaysoft.com/tag/视频)在线播放。在功能上它跟 [Cloudreve](https://www.iplaysoft.com/cloudreve.html) 和 [Z-File](https://www.iplaysoft.com/z-file.html) 比较相似，不过 ZPan 更好的地方就是「可以支持多用户」，同时也能支持用户的存储空间限制。

![](https://img.iplaysoft.com/wp-content/uploads/2020/zpan/zpan_screenshot.png!0x0.webp)

#### 基于云存储的开源网盘程序：

本质上，ZPan 并不会将文件保存在你的[服务器](https://www.iplaysoft.com/tag/服务器)，而是通过挂载各种云存储 / 对象存储服务，把文件保存到后端的云存储中。同时，ZPan 提供给用户一个可视化的**网盘文件管理**界面，浏览文件列表时实际上是在你服务器进行的，而上传下载文件则都是直连到后端云存储服务去的。

所以当你上传下载文件时，并不会受到服务器本身的带宽速度限制，也不会耗费服务器流量，速度超快，使用体验极佳。

### 支持对接各大主流云存储平台

**ZPan** 支持所有兼容 S3 协议的云存储 / 对象存储平台，比如[阿里云的 OSS](https://www.iplaysoft.com/go/aliyunoss)、[腾讯云的 COS](https://www.iplaysoft.com/go/qcloudcos)、[七牛云 KODO](https://www.iplaysoft.com/go/qiniu)、[华为云 OBS](https://www.iplaysoft.com/go/huaweicloud)、[UCloud](https://www.iplaysoft.com/go/ucloud)、[亚马逊](https://www.iplaysoft.com/go/amazon) AWS S3 等等。这些都是商业用途的云服务，可靠性极高！几乎完全不必担心丢失数据，而且传输速度也是极快！

而且像[腾讯云](https://www.iplaysoft.com/go/qcloud)和[阿里云](https://www.iplaysoft.com/go/aliyun)等都有一定的「免费额度」，个人使用一般都够，即便超额后付费的价格也很低廉 (以阿里云为例，OSS 空间是 0.12 元 / GB，上传免费，下载 0.25~0.5 元 / GB，还有包年包月套餐可选)。而大多数人的网盘主要就是存一些[文档](https://www.iplaysoft.com/tag/文档)，体积不大流量也不会用得特别狠，所以实际费用支出是很少的。而它们的速度、稳定性和可靠性却无可挑剔，所以总体来说性价比很高，用来搭建个人网盘或朋友、[团队](https://www.iplaysoft.com/tag/团队)内使用是绝佳的选择。

### 与其他同类网盘程序对比：

*   [NextCloud](https://www.iplaysoft.com/nextcloud.html)：功能强大但相对复杂，支持同步，其挂载云存储是通过服务器中转实现的，无法解决上传下载速度受限于服务器带宽的问题。
*   [Cloudreve](https://www.iplaysoft.com/cloudreve.html)：两者功能整体比较相似，Cloudreve 功能更丰富些，而 ZPan 更克制更精简。作者表示之前用的就是 Cloudreve，后来根据自己的需求和喜好才开发出 ZPan，两者之间青菜萝卜，大家可以试试再做决定。
*   [Z-File](https://www.iplaysoft.com/z-file.html)：主打 “在线文件目录” 的程序，同样支持各种对象存储和本地存储，但其定位是个人存放常用工具下载或做公共文件库，不会向多账户方向开发，而 ZPan 则支持多用户。

### 安装教程文档：

如果你有一定的 [Linux](https://www.iplaysoft.com/os/linux-platform) 操作经验 (初学者可以参考 [Linux 就该这么学](https://www.iplaysoft.com/linux-probe-book.html)、[鸟哥的 Linux 私房菜](https://www.iplaysoft.com/linux-vbird.html)等教程)，那么部署一个 ZPan 还算是比较简单的。它提供了一个安装脚本，也可以通过 Docker 来安装使用，具体可以参考[官网文档](https://zpan.space/#/zh-cn/?utm_source=iplaysoft.com&hmsr=iplaysoft.com)。

#### Linux 安装命令：

```
# 运行一键安装脚本
curl -sSf https://dl.saltbo.cn/install.sh | sh -s zpan

# 启动服务
systemctl start zpan

# 查看服务状态
systemctl status zpan

# 设置开机启动
systemctl enable zpan
```

#### Docker：

```
docker run -p 80:8222 -v /etc/zpan:/zpan -it saltbo/zpan:latest
```

#### 配置文件范例 (/etc/zpan/zpan.yml)

```
#详细配置文档可参考： https://zpan.space/#/zh-cn/config

debug: false
invitation: false # 邀请注册是否开启，开启后只允许邀请注册，默认关闭
storage: 104857600 # 给每个用户分配的初始空间，单位：字节

database:
driver: mysql
dsn: root:admin@tcp(127.0.0.1:3306)/zpan?charset=utf8&parseTime=True&loc=Local
#数据库支持 MySQL, PostgreSQL, SQlite, SQL Server 四种数据库驱动
#默认情况下不修改这里，会使用 SQlite 作为数据库

provider:
name: oss
bucket: saltbo-zpan-test
endpoint: https://oss-cn-zhangjiakou.aliyuncs.com
customHost: http://dl-test.saltbo.cn
accessKey: LTAIxxxxxxxxxxxxxxx7YoV
accessSecret: PFGVwxxxxxxxxxxxxxxxxRd09u

#配置发信邮箱即可开启账号注册的邮箱验证
#email:
# host: smtpdm.aliyun.com:25
# sender: no-reply@saltbo.fun
# username: Zpan
# password: mGxxxxxxxxh9
```

你需要先到自己喜欢的云存储提供商，比如[阿里云 OSS](https://www.iplaysoft.com/go/aliyunoss)、[腾讯云 COS](https://www.iplaysoft.com/go/qcloudcos) 等去注册开通一个 “Bucket”，然后按照格式将 bucket 名称、endpoint、accessKey、accessSecret 等信息填入配置文件即可。具体这里就不做保姆级的[教程](https://www.iplaysoft.com/tag/教程)了，大家摸索摸索吧。

### 写在后面：

作为一款基于云存储的网盘程序，[ZPan](https://www.iplaysoft.com/zpan.html) 非常适合服务器配置不高、空间 / 带宽不多的朋友使用。实际上，ZPan 更像是一个简单易用的云存储服务 “在线文件管理器”。

这样的好处显而易见，上传下载文件时能直连到云存储去，实现不限速不耗服务器流量，同时文件不占用服务器空间，且可靠性极高，不必担心丢失文件！所以如果你需要**搭建属于自己的私有网盘**，并且不介意将文件存到 OSS、COS 等云存储上去，同时还需要多用户支持的话，那么[开源](https://www.iplaysoft.com/tag/开源)免费的 ZPan 值得你一试。
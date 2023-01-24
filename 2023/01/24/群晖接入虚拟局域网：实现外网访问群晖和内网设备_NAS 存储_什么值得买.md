> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [post.smzdm.com](https://post.smzdm.com/p/all2qprg/?send_by=3547620770&invite_code=zdm4ptnapkinv&zdm_ss=iOS_3547620770_&from=singlemessage)

群晖接入虚拟局域网：实现外网访问群晖和内网设备
=======================

2022-07-01 15:45:43 83 点赞 800 收藏 46 评论

[![](https://qnam.smzdm.com/202207/01/62be9c901e4c44524.png_e1080.jpg)](https://post.smzdm.com/p/all2qprg/pic_2/)

本篇文章记录下，如何在外网访问[群晖](https://pinpai.smzdm.com/2315/)

[](https://pinpai.smzdm.com/2315/)

，并利用群晖来访问内网其他设备（以目前比较流行的虚拟局域网软件有

`ZeroTier`

和

`Tailscale`

为例）。

写在前面
----

异地访问 NAS，常见的还是`公网IP`+`DDNS`或`FRP`方案，需要暴露较多的端口在公网上，因此考虑使用`虚拟局域网`组网。  
个人使用感受：  
`ZeroTier`稳定易用，但异网打洞成功率较低，且由于中继[服务器](https://www.smzdm.com/fenlei/fuwuqi/)在外国，中继后对速度较慢，要求高的可以尝试自建 Moon / 根服务器或采用双 ZeroTier 网络 (参考扩展链接)。  
`Tailscale`打洞成功率更高，支持`MagicDNS`（[https://tailscale.com/kb/1081/magicdns/](https://tailscale.com/kb/1081/magicdns/)），使用也很方便。  
笔者由于目前使用环境是电信与[联通](https://pinpai.smzdm.com/96204/)

[](https://pinpai.smzdm.com/96204/)

组网，ZeroTier 无法成功打洞，且暂不想折腾自建 Moon，所以选用 Tailscale。

准备工作
----

*   由于 DSM 7 不允许第三方应用直接使用 root 权限运行，因此使用 Docker 安装。
    
*   本文在容器上执行的所有操作都通过 Docker CLI 完成。
    
*   本文均为在`DSM7`下操作，理论上可以兼容`DSM6`（未测试）。
    
    ### 创建持久性 TUN
    
    使用 SSH 连接到 NAS
    
    > ssh user@local-ip -p 22
    

以下设置步骤必须以 root 用户身份运行

> sudo -i

写入脚本，使得设备启动时调用`/dev/net/tun`

> echo -e ‘#!/bin/sh -e ninsmod /lib/modules/tun.ko’ > /usr/local/etc/rc.d/tun.sh

给这段脚本添加权限（其实应该先 vi 这个空的脚本，然后添加权限，最后在写入上面的脚本内容，不然会提示你 readonly）

> chmod a+x /usr/local/etc/rc.d/tun.sh

运行脚本一次以创建 TUN

> /usr/local/etc/rc.d/tun.sh

检查 TUN 的运行状态

> ls /dev/net/tun  
> /dev/net/tun

### 在 NAS 上安装 Docker

打开`套件中心`-> 搜索`Docker`-> 安装

ZeroTier 的安装和配置
---------------

### 设置容器

创建目录以存储 ZeroTier 的身份和配置

> mkdir /volume1/docker/zerotier-one

创建 ZeroTier 容器，命名为`zt`(Repo: [zerotier/zerotier-synology](https://hub.docker.com/r/zerotier/zerotier-synology))

> docker run -d  
> —name zt  
> —restart=always  
> —device=/dev/net/tun  
> —net=host  
> —cap-add=NET_ADMIN  
> —cap-add=SYS_ADMIN  
> -v /volume1/docker/zerotier-one:/var/lib/zerotier-one zerotier/zerotier-synology:latest

### 授权入网

1.  查看节点 ID 和状态（c715e6680c 为节点 ID）
    
    > docker exec -it zt zerotier-cli status  
    > 200 info c715e6680c 1.6.5 ONLINE
    
2.  加入您的网络 （替换成自己的网络 ID）
    
    > docker exec -it zt zerotier-cli join e5cd7a9e1cae134f
    
3.  在管理后台（[https://my.zerotier.com/](https://my.zerotier.com/)），对 NAS 进行授权和分配 ID。
    
    > ZeroTier 管理后台的其他设置和详细用法可以参考官方文档或者网上其他教程
    

### 常用命令

查看节点状态

> docker exec -it zt zerotier-cli status

查看网络状态

> docker exec -it zt zerotier-cli listnetworks

显示正在运行的容器（可选）

> docker ps

输入容器（可选）

> docker exec -it zt bash

Tailscale 的安装和配置
----------------

### 设置容器

创建目录以存储 Tailscale 的身份和配置

> mkdir /volume1/docker/tailscale

创建 Tailscale 容器，命名为`ts`(Repo: [tailscale/tailscale](https://hub.docker.com/r/tailscale/tailscale))

> docker run -d  
> —name ts  
> —restart=always  
> —device=/dev/net/tun  
> —net=host  
> —cap-add=NET_ADMIN  
> —cap-add=SYS_ADMIN  
> -v /volume1/docker/tailscale:/var/lib/tailscale tailscale/tailscale:stable

### 授权入网

1.  执行以下代码，获取授权链接
    
    > docker exec ts tailscale up
    
2.  复制并在浏览器打开授权链接，然后登录（鉴于国内环境，建议选择 Microsoft）并授权
    
3.  可以进入管理后台查看节点状态（[https://login.tailscale.com/admin/machines](https://login.tailscale.com/admin/machines)）
    
    ### 可选操作
    

*   禁用授权自动过期（Tailscale 的授权默认 6 个月过期）
    

操作路径：后台 ->`Machines`-> 点击节点右侧『•••』->`Disable key expiry`

*   启用 MagicDNS（可使用用机器名作为域名访问设备
    

操作路径：后台 ->`DNS`->`Add nameserver`然后`Enable MagicDNS`

在外网访问群晖所在内网的其他设备
----------------

### 设置群晖的内网转发

1.  使用`SSH`连接到 NAS 并切换到`ROOT`身份
    
2.  启用 IP 转发（永久修改）
    
    > echo ‘net.ipv4.ip_forward = 1’ | tee -a /etc/sysctl.conf  
    > echo ‘net.ipv6.conf.all.forwarding = 1’ | tee -a /etc/sysctl.conf  
    > sysctl -p /etc/sysctl.conf
    

第三行可能会执行失败，可以重启系统或者执行以下代码临时开启 IP 转发：

> echo 1 > /proc/sys/net/ipv4/ip_forward

1.  设置 NAT 转发
    

*   方式一（推荐
    
    > iptables -t nat -A POSTROUTING -o ! lo -j MASQUERADE
    
*   方式二
    

eth0 为[网卡](https://www.smzdm.com/fenlei/wangka/)名，如果是多网口的机器，请使用 ifconfig 查找自己机器的网卡名

> iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

1.  添加用户自定义脚本，使得启动时自动设置 NAT 转发
    

操作路径：  
`控制面板`->`服务`->`任务计划`-> 新增 -> 触发的任务 -> 用户自定义的脚本  
常规 -> 一般设置 -> 事件 -> 开机  
任务设置 -> 用户自定义的脚本，输入以下代码并确定  
为保证开机启动已加载所有网络模块，延迟 1 分钟再添加 NAT，否则会遇到模块不存在错误

> sleep 1m  
> iptables -t nat -A POSTROUTING -o ! lo -j MASQUERADE

### 设置 ZeroTier

1.  登录管理后台（[https://my.zerotier.com/](https://my.zerotier.com/)），进入 NetWorks 管理页面
    
2.  在`NetWorks`->`Advanced`->`Managed Routes`添加子网路由规则即可
    
    > 10.10.2.0/24 via 10.10.10.10
    

根据自己实际设置，第一个为群晖所在内网的 IP 段、第二个为 ZeroTier 分配给群晖的 IP

### 设置 Tailscale

1.  设置子网路由（10.10.2.0/24 为当前节点的内网段，自行替换）
    
    > docker exec ts tailscale up —advertise-routes=10.10.2.0/24 —ac[cep](https://pinpai.smzdm.com/18179/)
    > 
    > [](https://pinpai.smzdm.com/18179/)t-routes
    
2.  登录管理后台（[https://login.tailscale.com/admin/machines](https://login.tailscale.com/admin/machines)）并启用该节点的子网：
    

操作路径：后台 ->`Machines`-> 点击节点右侧『•••』->`Edit route settings`-> 打开对应子网前面的开关

验证
--

在外网设备上配置好对应的虚拟局域网客户端，然后就可以直接 Ping 群晖以及内网 IP 了

后记
--

本文外网访问内网设备的重点在于`设置TUN`和`设置内网转发`部分，虽然是以群晖为例，但各位也可以根据下方的官方文档稍加变通，应用于其他设备。  
`Tailscale`与`ZeroTier`还有其他强大的功能，大家可以自行探索。目前免费版额度均可满足个人使用，有额外需求的可以考虑它们的付费套餐。

参考链接
----

1.  Tailscale 官方安装文档 [https://tailscale.com/kb/installation/](https://tailscale.com/kb/installation/)
    
2.  群晖 DSM7 安装 Docker 版 Zerotier 教程 (官方) [Install ZeroTier For Synology NAS](https://docs.zerotier.com/devices/synology/)
    
3.  群晖 DSM7 安装 Docker 版 Zerotier 教程 (译文) [https://zhuanlan.zhihu.com/p/479171790](https://zhuanlan.zhihu.com/p/479171790)
    
4.  外网使用内网 IP 访问全内网设备 [https://post.smzdm.com/p/adwgkopd/](https://post.smzdm.com/p/adwgkopd/)
    

扩展阅读
----

1.  **详解 + 原理**：基于 Zerotier 的虚拟局域网 (内网穿透方案) [https://zhuanlan.zhihu.com/p/383471270](https://zhuanlan.zhihu.com/p/383471270)
    
2.  **中继 + 优化**：基于 Zerotier 的虚拟局域网 (VPS 中继优化) [https://zhuanlan.zhihu.com/p/431770438](https://zhuanlan.zhihu.com/p/431770438)
    
3.  ZeroTier 配置 Moon 节点 [https://post.smzdm.com/p/allv5k6o/](https://post.smzdm.com/p/allv5k6o/)
    
4.  放弃 moon 节点，直接搭建 Zerotier 根服务器 [https://post.smzdm.com/p/apxkx2m7/](https://post.smzdm.com/p/apxkx2m7/)
    

作者声明本文无利益相关，欢迎值友理性交流，和谐讨论～
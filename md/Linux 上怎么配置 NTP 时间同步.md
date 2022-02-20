> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NTY1MjY0MQ==&mid=2650840065&idx=4&sn=c7df754daae60034c0108ffe7fa6e10e&chksm=bd01094f8a768059bf75e3549102021464bae9192ee1110c69cbdcf6b6a029ba0907ae48c90c&mpshare=1&scene=1&srcid=0220EUrrDvtmFPha9yqRNW7Y&sharer_sharetime=1645350711721&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

在 Linux 中，保持准确的日期和时间至关重要，因为许多服务（如 cron 作业和脚本）依赖于准确的时间才能得到预期的结果

用于同步日期和时间的 `ntpd` 服务，在新的 Linux 发行版 ( `centos8、Ubuntu 20.04、Fedora 30` ) 中已经废弃了，取而代之的是 chrony

当然，选择 chrony 是有原因的，相比 ntpd , 它有如下的优点：

```
1、时间同步的速度比 ntpd 更快

2、chrony 很好的处理了同步延迟以及网络延迟

3、即使出现网络降级，chrony 仍然能正常工作

4、本地机器可以作为时间服务器，其他机器从这台服务器上同步时间
```

### 安装

在新的 Linux 发行版（`centos8、Ubuntu 20.04、Fedora 30`）及以后的版本中，系统默认已经安装了 chrony，在这之前的版本是没有安装的，可以使用下面的命令进行安装

```
yum install chrony
```

安装完成后，chrony 服务默认会自动加到 systemctl 中管理，下面列出了一些常用的命令

```
#查询 chronyd 服务状态
systemctl status chronyd     

#启动 chronyd 服务
systemctl start chronyd   

#关闭 chronyd 服务
systemctl stop chronyd   

#重启 chronyd 服务
systemctl restart chronyd   

#设置 chronyd 服务开机自启
systemctl enable chronyd
systemctl daemon-reload
```

另外，启动 chronyd 服务的时候如果出现下面截图中的错误，需要安装或者升级 `libsepol、policycoreutils-python`

![](https://mmbiz.qpic.cn/mmbiz_png/a2ibvjLwErmklKzkM0eScMVIWe7z9nibOLVWPLib4dQN3MsGjy87qNicpqg1H8yOX53vMYxCZd1qeqq8gMsRpKfJoA/640?wx_fmt=png)

具体的问题说明详见 `https://bugzilla.redhat.com/show_bug.cgi?id=1592775`

```
# 安装 libsepol 和 policycoreutils-python
yum install libsepol policycoreutils-python
```

### chrony 的组成

chrony 是由 守护进程 `chronyd` 以及 命令行工具 `chronyc` 组成的，具体如下图

![](https://mmbiz.qpic.cn/mmbiz_png/a2ibvjLwErmklKzkM0eScMVIWe7z9nibOLticHuXhHJicM1Sn5wSiazBVC2aNTYo9CKCAxqZriagic2zzK5RpXEKB63Vg/640?wx_fmt=png)

`chronyd` 在后台静默运行并通过 123 端口与时间服务器定时同步时间，默认的配置文件是 `/etc/chrony.conf`

`chronyc` 通过 323 端口与 `chronyd` 交互，可监控 `chronyd` 的性能并在运行时更改各种操作参数

`chronyc` 通过下面的方式访问 `chronyd`

```
1、通过 IPv4 或 IPv6 访问

2、通过 Unix 域 socket, 但只能访问到本地的 chronyd，而且需要 root 用户或者 chrony 用户才能访问
```

默认情况下，`chronyc` 先通过 Unix 域 socket 访问 `chronyd`，默认的 socket 文件是 `/var/run/chrony/chronyd.sock`, 如果失败（常见的原因是使用非 root 用户运行 chronyc ），将尝试通过 `127.0.0.1` 访问 `chronyd`

### chrony 配置

守护进程 `chronyd` 的默认配置文件是 `/etc/chrony.conf`，其中可配置项很多，这里介绍一些常用的

<table data-source-line="93" width="NaN"><thead><tr data-darkmode-bgcolor-16453799440763="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16453799440763="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; background-color: rgb(255, 255, 255); border-top: 1px solid rgb(198, 203, 209);"><th data-darkmode-bgcolor-16453799440763="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16453799440763="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-top-width: 1px; border-color: rgb(223, 226, 229); word-break: break-all;">配置项</th><th data-darkmode-bgcolor-16453799440763="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16453799440763="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-top-width: 1px; border-color: rgb(223, 226, 229); word-break: break-all;">说明</th></tr></thead><tbody><tr data-darkmode-bgcolor-16453799440763="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16453799440763="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; background-color: rgb(255, 255, 255); border-top: 1px solid rgb(198, 203, 209);"><td data-darkmode-bgcolor-16453799440763="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16453799440763="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">server</td><td data-darkmode-bgcolor-16453799440763="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16453799440763="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">客户端需找哪个服务器询问当前时间</td></tr><tr data-darkmode-bgcolor-16453799440763="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16453799440763="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; background-color: rgb(246, 248, 250); border-top: 1px solid rgb(198, 203, 209);"><td data-darkmode-bgcolor-16453799440763="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16453799440763="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">pool</td><td data-darkmode-bgcolor-16453799440763="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16453799440763="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">同 server 配置项</td></tr><tr data-darkmode-bgcolor-16453799440763="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16453799440763="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; background-color: rgb(255, 255, 255); border-top: 1px solid rgb(198, 203, 209);"><td data-darkmode-bgcolor-16453799440763="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16453799440763="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">driftfile</td><td data-darkmode-bgcolor-16453799440763="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16453799440763="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">本地时钟和服务器时钟的预估漂移保存到哪个文件中</td></tr><tr data-darkmode-bgcolor-16453799440763="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16453799440763="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; background-color: rgb(246, 248, 250); border-top: 1px solid rgb(198, 203, 209);"><td data-darkmode-bgcolor-16453799440763="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16453799440763="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">makestep</td><td data-darkmode-bgcolor-16453799440763="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16453799440763="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">纠正客户端时间的步进参数</td></tr><tr data-darkmode-bgcolor-16453799440763="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16453799440763="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; background-color: rgb(255, 255, 255); border-top: 1px solid rgb(198, 203, 209);"><td data-darkmode-bgcolor-16453799440763="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16453799440763="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">rtcsync</td><td data-darkmode-bgcolor-16453799440763="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16453799440763="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">是否允许内核同步实时时钟</td></tr><tr data-darkmode-bgcolor-16453799440763="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16453799440763="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; background-color: rgb(246, 248, 250); border-top: 1px solid rgb(198, 203, 209);"><td data-darkmode-bgcolor-16453799440763="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16453799440763="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">allow</td><td data-darkmode-bgcolor-16453799440763="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16453799440763="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">允许客户端通过内网地址同步时钟</td></tr><tr data-darkmode-bgcolor-16453799440763="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16453799440763="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; background-color: rgb(255, 255, 255); border-top: 1px solid rgb(198, 203, 209);"><td data-darkmode-bgcolor-16453799440763="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16453799440763="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">logdir</td><td data-darkmode-bgcolor-16453799440763="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16453799440763="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">日志目录</td></tr></tbody></table>

### 配置时间同步

守护进程 `chronyd` 既可作为客户端 与 服务器同步时间，又可作为一个服务器，接受其他客户端同步时间的请求

下面以配置局域网时间服务器为例来说明 `chronyd` 的客户端和服务器的配置，相关的参数如下：

```
客户端IP: 192.168.70.22

服务器IP: 192.168.70.21
```

下面是具体的配置步骤

##### 1. 安装客户端服务器

分别在客户端机器和服务器机器安装 chrony，安装方法前面有讲述，这里不赘述了

##### 2. 编辑 `/etc/chrony.conf`

安装好 chrony 之后，编辑 `/etc/chrony.conf` 配置文

*   客户端配置
    

```
# 同步时间的服务器 IP 或 域名
server 192.168.70.21 iburst

# 系统时钟的预估漂移保存到指定的文件中，是为了在下次启动时能稳定的同步
driftfile /var/lib/chrony/drift

# 如果系统时钟由于某种原因与启动后的服务器时间相差甚远，允许 chronyd 
# 通过步进而不是回转来快速纠正它
makestep 1 3

# 为了使客户端实时时钟接近服务器的时钟，以便下次时钟启动时更接近真实的时间
# 增加了一种 rtcsync 模式，该模式下，系统时间会定期的拷贝到实时时钟里
rtcsync

# 日志
logdir /var/log/chrony
```

*   服务器配置
    

安装 chrony 之后，默认的配置是客户端的启动配置的，要想作为一个时间服务器来运行的话， 需要在配置中增加 `allow` 配置项，它表示允许客户端通过该地址和服务器同步时间

另外，时间服务器的时间也需要从网络上其他的时间服务器进行同步，这里直接用默认的即可，具体的配置如下

```
# 同步时间的服务器 IP 或 域名
pool 0.centos.pool.ntp.org iburst
pool 1.centos.pool.ntp.org iburst
pool 2.centos.pool.ntp.org iburst
pool 3.centos.pool.ntp.org iburst

# 为了在下次启动时稳定的同步，系统时钟的预估漂移需要保存到指定的文件中
driftfile /var/lib/chrony/drift

# 如果系统时钟由于某种原因与启动后的服务器时间相差甚远，允许 chronyd 
# 通过步进而不是回转来快速纠正它，这个过程将花费很长时间
makestep 1 3

# 为了使客户端实时时钟接近服务器的时钟，以便下次时钟启动时更接近真实的时间
# 增加了一种 rtcsync 模式，该模式下，系统时间会定期的拷贝到实时时钟里
rtcsync

# 允许客户端通过该地址和服务器同步时间，其实这里配置的就是时间服务器的地址
allow 192.168.70.21
```

`pool` 配置项

客户端需要去时间服务器获取时间，配置文件中 `server` 和 `pool` 配置项表示时间服务器的地址，支持域名或者 IP

建议配置多个时间服务器的地址，优先选择同步良好，网络稳定且靠近客户端的地址

`pool` 与 `allow` 的区别

`pool`指的是进行时间同步的服务器 IP 地址或域名，作为服务器来说，其机器时间也需要从其他时间服务器同步，此时，服务器是作为一个客户端从网络服务器上获取时间

而 `allow` 字段表示的是作为服务器，允许客户端从该地址获取时间，此地址其实就是服务器的内网地址

##### 3. 处理防火墙

时间服务器如果有开启防火墙的话，需要开启 UDP 协议 的 123 端口，以允许客户端向服务器发送获取时间的请求

```
[root@cghost22 ~]# firewall-cmd --permanent --add-port=123/udp
success
[root@cghost22 ~]# firewall-cmd --reload
success
```

如果想要关闭防火墙的 123 端口，在服务器机器上执行下面的命令

```
[root@cghost22 ~]# firewall-cmd --permanent --remove-port=123/udp
success
[root@cghost22 ~]# firewall-cmd --reload
success
```

##### 4. 重启 chrony

配置好客户端和服务器之后，为使配置生效，需要重启 chronyd 服务

```
systemctl restart chronyd
```

##### 5. 查询信息

在服务器端输入 `chronyc clients` 命令查看同步的客户端信息

![](https://mmbiz.qpic.cn/mmbiz_png/a2ibvjLwErmklKzkM0eScMVIWe7z9nibOLwJkUprCUHoQB9ObR9wP1s0AWzPvgTYvicrV29fvaRC8kC3ZwOYWlWIw/640?wx_fmt=png)

在客户端输入 `chronyc sources` 命令查查看时间服务器的信息 ![](https://mmbiz.qpic.cn/mmbiz_png/a2ibvjLwErmklKzkM0eScMVIWe7z9nibOLJhIDV9HLGPYMBf2p737OHYeRuse6RXCmj3hGFYeQK11YHEAFwX45Kw/640?wx_fmt=png)

### 问题

在配置时间服务器的过程中，可能遇到各种问题，下面记录了一些常见的问题以及解决思路

##### 客户端无法进行时间同步

第一个想到的应该是 服务器之上存在防火墙，这时客户端向服务器请求时间不会有任何的响应，在客户端机器输入 `chronyc sources` 命令可以看到 `Reach` 字段的值为 0 ，表示客户端无法收到服务器的响应了，具体请看下图

![](https://mmbiz.qpic.cn/mmbiz_png/a2ibvjLwErmklKzkM0eScMVIWe7z9nibOLD1qFBdxjxRTIcsr4Tic6gSdEg5IV02zFAkmO8rTcgedCHu6SI2icNlmw/640?wx_fmt=png)

如果排除了防火墙的问题的话，可能需要使用一些工具（`tcpdump`、`wireshark`）检查下客户端是否有收到服务器的数据包

##### 客户端和服务器时间差得太多，如何快速修正

通常 `chrony` 会逐步的修正和服务器之间的时间差，根据需要加快或者减慢时钟，如果客户端和服务器时间差得太多了的话，这个过程会持续很长时间

这种情况可以使用 `chronyc makestep` 命令快速修复客户端时间，`makestep` 后面不带任何参数时，表示 `chronyd` 取消正在修正时间的动作，将当前客户端时间直接修改成服务器的时间

不过，需要注意的是，这种方法直接越过了一段系统时间，有可能会对应用程序造成严重的问题，所以，推荐按照 `chronyd` 逐步修正的方式来同步时间

### 小结

本文介绍了 Linux 中，时间同步的配置方法，提供常见的问题的解决思路或者方案，更多关于 `chrony` 的介绍请参考下方的网站

```
https://chrony.tuxfamily.org/documentation.html
```
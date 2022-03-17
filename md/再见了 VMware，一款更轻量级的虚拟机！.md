> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666560380&idx=1&sn=a14c185dbd2ec52b68d858bf39a6e485&chksm=80dcbfd7b7ab36c18fbade28438a5f219b20c10fe5288e31d17f165545440e9fadb6b1e67546&scene=21#wechat_redirect)

```
最近，打算在 Mac 电脑上安装一个虚拟机，用来学习 K8S。
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/emOFDDdibbjKo1EHq9hsje5pQ5GJPYyHPgWhwd7XoM9RG6tD9hPEa6b5lZMtcJ22mgntfxbzyIKWsZXKaXSdCTg/640?wx_fmt=jpeg)

无法白嫖

同时，**Multipass** 提供了一个命令行界面来启动和管理 **Linux** 实例。下载一个全新的镜像需要几秒钟的时间，并且在几分钟内就可以启动并运行 **VM**。（直呼好家伙，大大降低的我们安装虚拟机的成本了）

> Multipass 官网：https://multipass.run/

![](https://mmbiz.qpic.cn/mmbiz_png/emOFDDdibbjKo1EHq9hsje5pQ5GJPYyHPBpbJ4SM5Uo08rnOtf5RoUMmIia8KVZVGfFFo0pfialhXCZrsSib8r8xKg/640?wx_fmt=png)

Multipass 官网

开始使用
----

在使用 **Multipass** 之前 ，首先需要安装 **Multipass** 工具，可以打开官网进行下载

```
https://multipass.run/
```

选择对应的操作系统，因为我的是 **Mac** 电脑，所以选择 MacOS，点击 **Download** 开始下载

![](https://mmbiz.qpic.cn/mmbiz_png/emOFDDdibbjKo1EHq9hsje5pQ5GJPYyHPMH2AQnWj0uemtGFjpMp7ay1TNx8UTyBWLhHe3XTTmYKuwJwDYDIKQg/640?wx_fmt=png)

选择对应版本安装

如果电脑装了 **brew** 可以直接命令行下载安装:

```
brew cask install multipass
```

下载安装成功后, 执行以下命令可以查看当前软件版本:

```
multipass --version
```

安装成功后，点击运行，即可打开 **Multipass** 客户端，这里能够快速的通过 **Open Shell** 创建和启动一个虚拟机

![](https://mmbiz.qpic.cn/mmbiz_png/emOFDDdibbjKo1EHq9hsje5pQ5GJPYyHPiaYc7c74JmxibEMh1TtsLXZhYxQtOXCbj7kFDVpedfmxCtNWWfcMqwXQ/640?wx_fmt=png)

Multipass 客户端

下面，我们来介绍如何通过命令的方式，创建一台虚拟机

创建 Ubuntu 虚拟机
-------------

首先，通过以下指令查看可供下载的 **Ubuntu** 镜像

```
multipass find
```

运行成功后，可以看到下面的这些镜像列表，包含各种版本的

```
Image                       Aliases           Version          Description
core                        core16            20200818         Ubuntu Core 16
core18                                        20200812         Ubuntu Core 18
18.04                       bionic            20211109         Ubuntu 18.04 LTS
20.04                       focal,lts         20211118         Ubuntu 20.04 LTS
21.04                       hirsute           20211119         Ubuntu 21.04
21.10                       impish            20211103         Ubuntu 21.10
appliance:adguard-home                        20200812         Ubuntu AdGuard Home Appliance
appliance:mosquitto                           20200812         Ubuntu Mosquitto Appliance
appliance:nextcloud                           20200812         Ubuntu Nextcloud Appliance
appliance:openhab                             20200812         Ubuntu openHAB Home Appliance
appliance:plexmediaserver                     20200812         Ubuntu Plex Media Server Appliance
anbox-cloud-appliance                         latest           Anbox Cloud Appliance
minikube                                      latest           minikube is local Kubernetes
```

下载最新版的 **Ubuntu** 镜像并运行，初次创建时需要下载 **Ubuntu** 镜像，网络畅通的情况下，稍等片刻即可。

```
multipass launch -n vm01 -c 1 -m 1G -d 10G
```

自定义配置创建可以参考如下方式:

```
-n, --name: 名称
-c, --cpus: cpu核心数, 默认: 1
-m, --mem: 内存大小, 默认: 1G
-d, --disk: 硬盘大小, 默认: 5G
```

操作虚拟机
-----

### 查看虚拟机列表

虚拟机创建完成后，可以使用 **multipass list** 命令进行查看虚拟机列表

```
% multipass list
Name                    State             IPv4             Image
vm01                    Running           192.168.64.2     Ubuntu 20.04 LTS
```

可以看到目前正在运行一台 **Ubuntu 20.04** 版本的虚拟机，并且对应的 **IP** 地址为 **192.168.64.2**

### 外部操作虚拟机

通过 **multipass exec** 命令在实例内执行给定的命令。第一个参数是运行命令的实例，也就是 **vm01**

我们通过 **exec** 命令，就可以在外部操作刚刚创建的虚拟机，例如查看内部所处的目录，执行 **pwd** 命令

```
# multipass exec vm01 pwd
/home/ubuntu
```

### 查看虚拟机信息

通过 **multipass info** 命令，即可查看当前运行的虚拟机信息

```
# multipass info vm01
Name:           vm01
State:          Running
IPv4:           192.168.64.2
Release:        Ubuntu 20.04.3 LTS
Image hash:     91740d72ffff (Ubuntu 20.04 LTS)
Load:           0.00 0.00 0.00
Disk usage:     1.3G out of 9.5G
Memory usage:   170.9M out of 1.9G
Mounts:         --
```

### 进入虚拟机

通过 **multipass shell** 命令，即可进入到虚拟机内部

```
multipass shell vm01
```

运行后的结果如下所示，可以看到目前虚拟机的一些系统配置信息，以及内存和磁盘的使用情况

![](https://mmbiz.qpic.cn/mmbiz_png/emOFDDdibbjKo1EHq9hsje5pQ5GJPYyHPcXEUr2NHibyvK5aUIXbNbyF0HUh9ibcCNiasibKjJgk4ce5Z6UGychcXdg/640?wx_fmt=png)

进入虚拟机内部

同时会运行一个新的 **Ubuntu** 系统，此时便进入到了 **Ubuntu** 环境中, 在里面可以执行相关的 **linux** 指令

当然如果不想进入系统内部，也可以通过上述提到的 **multipass exce** 命令，来操作 **Ubuntu** 系统

首先执行下面命令，给系统设置一个 **root** 密码，设置好密码后，使用 **su root** 切换到 **root** 用户

```
# 设置密码
sudo passwd
# 切换 root
su root
```

**ubuntu** 是使用 **apt-get** 来进行包管理的，首先更新一下 **apt-get**，然后安装 **nginx**

```
# 更新apt
apt-get update
# 安装 nginx
apt-get install nginx
```

```
root@vm01:/lost+found# cd /etc/nginx/
root@vm01:/etc/nginx# ls
conf.d        fastcgi_params  koi-win     modules-available  nginx.conf    scgi_params      sites-enabled  uwsgi_params
fastcgi.conf  koi-utf         mime.types  modules-enabled    proxy_params  sites-available  snippets       win-utf
```

![](https://mmbiz.qpic.cn/mmbiz_png/emOFDDdibbjKo1EHq9hsje5pQ5GJPYyHPP9gg2fE9JOyfR8Td5FiciaOO3SLmvSArZcJAxJF8DNpgTyF2mUpOl1rw/640?wx_fmt=png)

nginx 安装成功

### 挂载数据卷

**multipass** 还提供和 **Docker** 一样的挂载数据卷的功能，能够与外部宿主机的文件保持同步。

```
# 挂载格式
multipass mount 宿主机目录  实例名:虚拟机目录
```

下面，我在用户的目录下，创建一个 **hello** 目录

```
# 创建hello目录
mkdir hello
# 挂载
multipass mount /Users/moxi/hello  vm01:/hello
```

挂载完成后，我们回到实例 vm01 中，可以看出多了一个 **hello** 文件夹，说明已经成功挂载～

![](https://mmbiz.qpic.cn/mmbiz_png/emOFDDdibbjKo1EHq9hsje5pQ5GJPYyHP1fIvF57PS4FRTBIPtibwASZHs2xQq7alRmoyUzQkicmic0DSKDYC6DHZg/640?wx_fmt=png)

挂载成功

因此，以后在 **vm01** 的 **hello 文件夹** 中创建的文件，都会在的宿主机的 **hello 文件夹**同步显示

### 卸载数据卷

如果以后不需要用到挂载了，可以使用 **unmount** 命令卸载

```
#卸载数据卷
multipass umount 容器名
```

### 传输文件

除了使用上述的 **mount** 挂载卷的方式实现文件的交互，同时还可以通过 transfer 命令，将宿主机的文件，发送到虚拟机内部

```
multipass transfer 主机文件 容器名:容器目录
```

例如，将 **hello.txt** 发送到

```
multipass transfer hello.txt vm01:/home/ubuntu/
```

### 删除和释放实例

使用下面的命令，可以开启、停止、删除和释放实例

```
# 启动实例
multipass start vm01
# 停止实例
multipass stop vm01
# 删除实例（删除后，还会存在）
multipass delete vm01
# 释放实例（彻底删除）
multipass purge vm01
```

### 容器配置自动化

为了保持开发环境和线上环境一致性 同时节省部署时间 **multipass** 给我们提供了 **--cloud-init** 选项进行容器启动初始化配置:

```
multipass launch --name ubuntu --cloud-init config.yaml
```

上面 **config.yaml** 则是容器的初始化配置文件，例如，我们想在初始化容器的时候，自动下载安装 **Node.js**，内容如下：

```
#cloud-config
runcmd:
  - curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
  - sudo apt-get install -y nodejs
```

`runcmd` 可以指定容器 **首次启动** 时运行的命令

> 凡是用户自定义的 cloud-init 的配置文件, 必须以 #cloud-config 开头，这是 cloud-init 识别它的方式。

**yaml** 配置文件可以参考下面的文章

```
https://cloudinit.readthedocs.io/en/latest/topics/examples.html?highlight=lock-passwd#including-users-and-groups
```

更多关于 **multipass** 的高阶的技巧，欢迎访问 **multipass** 官方文档

```
https://multipass.run/docs/
```

好了，本期关于 **Multipass** 的基础学习就到这里了，感兴趣的小伙伴环境下载自己动手学习呢～

总结
--

使用了一阵子后，**Multipass** 在使用起来很简洁直观，在整体操作上和 **docker** 类似，如果想在自己电脑上快速搭建一个 **Linux** 系统用于学习的话，选择使用 **Multipass** 还是非常方便的。

唯一不足的地方就是，**Multipass** 是由 **Ubuntu** 背后的 **Canonical** 公司开发的，因此 **Multipass** 所使用的镜像也都是 **Ubuntu** 镜像（这也可以理解，毕竟支持自家的产品），不过对于习惯了使用 **CentOS** 的用户来说，还是用些许不太习惯。不过瑕不掩瑜，**Multipass** 在我看来还是一款非常棒的软件～

> Multipass 在 GitHub 的链接：  
> https://github.com/canonical/multipass

- EOF -
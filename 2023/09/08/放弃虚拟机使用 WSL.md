> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/sCzXKQDCmhNyPV3Mupf_mQ)

> 前言我的开发环境一直是 Windows，像 MySQL 和 Redis 这种常用的组件，我一般都是在 Windows 中起

前言
--

我的开发环境一直是 Windows，像 MySQL 和 Redis 这种常用的组件，我一般都是在 Windows 中起一个虚拟机，在虚拟机里安装，然后将虚拟机端口映射在宿主机上，开发调试的时候就可以使用 127.0.0.1 这种地址进行调试

在 Windows10 之后提供了适用于 Linux 的 Windows 子系统 ---WSL（Windows Subsystem for Linux），相当于可以直接在 Windows 上使用 Linux 系统

安装
--

确认使用的 WSL 版本，最好使用版本 2，使用 `wsl --set-default-version 2` 进行修改

打开 powershell 直接运行 `wsl --install` 即可安装一个 Ubuntu 子系统，使用 `wsl --install <Distribution Name>` 安装其他发行版本的 Linux，具体默认支持哪些发行版本可以执行 `wsl --list --online` 查看

```
wsl --list --online
以下是可安装的有效分发的列表。
使用 'wsl.exe --install <Distro>' 安装。
NAME                                   FRIENDLY NAME
Ubuntu                                 Ubuntu
Debian                                 Debian GNU/Linux
kali-linux                             Kali Linux Rolling
Ubuntu-18.04                           Ubuntu 18.04 LTS
Ubuntu-20.04                           Ubuntu 20.04 LTS
Ubuntu-22.04                           Ubuntu 22.04 LTS
OracleLinux_7_9                        Oracle Linux 7.9
OracleLinux_8_7                        Oracle Linux 8.7
OracleLinux_9_1                        Oracle Linux 9.1
openSUSE-Leap-15.5                     openSUSE Leap 15.5
SUSE-Linux-Enterprise-Server-15-SP4    SUSE Linux Enterprise Server 15 SP4
SUSE-Linux-Enterprise-15-SP5           SUSE Linux Enterprise 15 SP5
openSUSE-Tumbleweed                    openSUSE Tumbleweed

```

下面再介绍下如何安装 ArchLinux

1.  到 Github 上下载最新的 Arch.zip，下载地址：https://github.com/yuk7/ArchWSL/releases
    
2.  下载好之后，解压其中的文件到你需要存放 ArchLinux 的路径，例如 D:\vm\archlinux
    
3.  执行目录下的 Arch.exe 文件，安装程序会自动将 ArchLinux 安装到同目录下面，并配置好 wsl
    

安装完成之后，打开终端，应该可以看到刚装好的 ArchLinux 系统

```
wsl --list
适用于 Linux 的 Windows 子系统分发:
Arch (默认)

```

如果有多个子系统，可以使用 `wsl --set-default Arch` 命令设置 Arch 为默认子系统

配置使用 Arch
---------

现在可以使用命令 `wsl -d Arch` 进入系统了，我一般从终端进入：鼠标右键 --》选择在终端中打开，终端列表中会多了一个 Arch 系统

![](https://mmbiz.qpic.cn/sz_mmbiz_png/HjSED1jVeWF0nxYBHiaeJMbVS3fkMr7WF5IOOdQRjoAzqOUqCaibzMGEMkNiaMYPUyZbY0ibqo1VlR3FkbOqiaias1yA/640?wx_fmt=png)

也可以使用 Win+r 快捷键，输入 `wt` 打开

#### 配置 pacman

配置 pacman 镜像源，改为国内的，`vim /etc/pacman.d/mirrorlist` 添加如下内容

```
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch

```

输入下面命令，配置 pacman key：

```
pacman-key --init
pacman-key --populate
pacman -Sy archlinux-keyring

```

更新系统：

```
pacman -Syu

```

配置 archlinuxcn 镜像源，`vim /etc/pacman.conf` 添加如下内容

```
[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch

```

安装 archlinuxcn 的 key

```
pacman -Sy archlinuxcn-keyring

```

安装 podman
---------

像 MySQL 和 Redis 这种组件如果只是开发使用的话，建议使用容器启动，简单方便，可以试试 docker 的替代品 podman，上手很简单，只需要把之前的 docker 命令中的 docker 换成 podman 即可，比如 `docker pull` 改成 `podman pull`

安装：`pacman -S podman`

验证：`podman version`

安装 MySQL

```
podman run -d --name mysql -v /data/mysql_data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -p 3306:3306 docker.io/library/mysql

```

> 注意：镜像地址不能直接写 mysql，需要写完整地址

同理安装 Redis

```
# 获取配置文件
wget http://download.redis.io/redis-stable/redis.conf -O /etc/redis.conf
# 必改项
bind 0.0.0.0
protected-mode no
daemonize no
# 启动
podman run -d --name redis -p 6379:6379 -v /etc/redis.conf:/etc/redis.conf docker.io/library/redis redis-server /etc/redis.conf

```

查看状态

```
$ podman ps
CONTAINER ID IMAGE                           COMMAND               CREATED       STATUS     PORTS                   NAMES
43daf4496a39 docker.io/library/mysql:latest mysqld                46 hours ago Up 2 hours  0.0.0.0:3306->3306/tcp mysql
3bce4e999b39 docker.io/library/redis:latest redis-server /usr...  29 hours ago Up 2 hours  0.0.0.0:6379->6379/tcp redis

```

现在就可以在 Windows 上通过 127.0.0.1 访问 MySQL 和 Redis 了

配置开机自启
------

每次开机，子系统中的服务是不会自动运行的，每次都需要手动启动服务肯定很麻烦

在 wsl 系统内创建启动脚本文件 `/etc/startup.sh` ，添加如下内容

```
podman start mysql redis

```

在 Windows 上运行 Win+r，输入 `shell:startup` ，在弹出的文件夹内，新建一个文件，名字随意，后缀使用. vbs，比如 arch.vbs，输入如下内容

```
Set ws = CreateObject("Wscript.Shell")
ws.run "wsl -d Arch -u root sh /etc/startup.sh", vbhide

```

其他问题
----

在刚安装好 MySQL 和 Redis 后，通过 127.0.0.1 可以正常访问 MySQL，但是无法访问 Redis，使用 WSL 系统的 IP 可以正常访问，排查了很久都没有解决，想着后续使用 WSL 系统的 IP 连接也还行，但发现每次机器重启 WSL 系统的 IP 都会改变，总不能每次重启机器后都改下 IP 吧，最后试着使用 `wsl --update` 升级了下 WSL，没问题了.........

如果遇到了同样的问题，建议升级 wsl 试试，如果还不行，可以使用其他大佬做的一个工具：https://github.com/shayne/go-wsl2-host ，这个我验证过了，确实可行

参考
--

https://learn.microsoft.com/zh-cn/windows/wsl/install

https://zhuanlan.zhihu.com/p/613454594
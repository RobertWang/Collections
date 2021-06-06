> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU3NTgyODQ1Nw==&mid=2247485564&idx=2&sn=38620597bda9f3dfdd7208c4f94d7ff2&chksm=fd1c70faca6bf9ec7fab69f1fdd5453f63bd7040940b33b1a730c1d12a42675424b70f906e69&scene=21#wechat_redirect)

一个系统管理员可能会同时管理着多台服务器，这些服务器也许会放在不同的地方。要亲自一台一台的去访问来管理它们显然不是最好的方法，通过远程控制的方法应该是最有效的。

Linux 系统的远程管理工具大概有几种：telnet，ssh，vnc 等，其中 ssh 是最常用的管理方法，采用密文的传输方式，简单安全。

`Secure Shell`缩写是`SSH`， 由 IETF 的网络工作小组（`Network Working Group`）所制定，`SSH`是一项创建在应用层和传输层基础上的安全协议，为计算机的`shell`提供安全的传输和使用环境。

下面我们来介绍 SSH 的 7 大用法。

#### 1. 基本用法

最简单的用法就是不带参数，仅输入 ssh 再加上主机地址，比如：

```
ssh 192.168.0.116
```

这种形式登陆主机，会默认使用当前用户进行登录。第一次连接的时候，SSH 会确认目标主机的真实性，如果没有问题的话，输入 yes 即可。

![](https://mmbiz.qpic.cn/mmbiz_png/IsrmVA0RIYNlCBlwISrIOS51frylhn2KH2akOnfSZIem96Md3ibibic1h24hiauI4LdRfsZSibsobm1Vh30DprjtONQ/640?wx_fmt=png)

如果我们想要以指定用户名来登录主机，有两种方法：  

**a. 使用 `-l` 选项**

```
ssh -l alvin 192.168.0.116
```

**b. 使用 user@hostname 格式**

```
ssh alvin@192.168.0.116
```

这两种方法，其中第二种尤为常用。

#### 2. 指定端口登录

SSH 默认使用的端口号是 22。大多现代的 Linux 系统 22 端口都是开放的。如果你运行 ssh 程序而没有指定端口号，它直接就是通过 22 端口发送请求的。

如果我们不想通过 22 端口登录，那么我们可以使用 `-p` 选项来指定端口。

```
ssh 192.168.0.116 -p 1234
```

引申话题：如何修改端口号？

只需修改 `/etc/ssh/ssh_config` ，修改如下一行：

```
Port 22
```

#### 3. 对所有数据请求压缩

使用 `-C` 选项，所有通过 SSH 发送或接收的数据将会被压缩，并且任然是加密的。

```
ssh -C 192.168.0.116
```

但是，这个选项在网速不是很快的时候比较有用，而当网速较快的时候，使用压缩反而会降低效率，所以要视情况使用。

#### 4. 打开调试模式

因为某些原因，我们想要追踪调试我们建立的 SSH 连接情况。SSH 提供的 `-v` 选项参数正是为此而设的。其可以看到在哪个环节出了问题。

```
[Alvin.Alvin-computer] ➤ ssh -v pi@192.168.0.116
OpenSSH_7.1p2, OpenSSL 1.0.1g 7 Apr 2014
debug1: Reading configuration data /etc/ssh_config
debug1: Connecting to 192.168.0.116 [192.168.0.116] port 22.
debug1: Connection established.
debug1: key_load_public: No such file or directory
debug1: Enabling compatibility mode for protocol 2.0
debug1: Local version string SSH-2.0-OpenSSH_7.1
debug1: Remote protocol version 2.0, remote software version OpenSSH_7.4p1 Raspbian-10+deb9u4
debug1: match: OpenSSH_7.4p1 Raspbian-10+deb9u4 pat OpenSSH* compat 0x04000000
debug1: Authenticating to 192.168.0.116:22 as 'pi'
debug1: SSH2_MSG_KEXINIT sent
debug1: SSH2_MSG_KEXINIT received
```

#### 5. 绑定源地址

如果你的客户端有多于两个以上的 IP 地址，你就不可能分得清楚在使用哪一个 IP 连接到 SSH 服务器。为了解决这种情况，我们可以使用 `-b` 选项来指定一个 IP 地址。这个 IP 将会被使用做建立连接的源地址。

```
[Alvin.Alvin-computer] ➤ ssh -b 192.168.0.105 pi@192.168.0.116
Linux raspberrypi 4.14.71-v7+ #1145 SMP Fri Sep 21 15:38:35 BST 2018 armv7l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Feb 24 08:52:29 2019 from 192.168.0.105
```

#### 6. 远程执行命令

如果我们想在目标主机执行一条命令，我们通常的做法是，先登录到目标主机，执行命令，再退出来。这样做当然是可以，但是比较麻烦。

如果我们仅仅是想远程执行一条命令，可以直接在后面跟上命令就好，如下：

```
[Alvin.Alvin-computer] ➤ ssh pi@192.168.0.116 ls -l
Desktop
Documents
Downloads
MagPi
Music
```

#### 7. 挂载远程文件系统

另外一个很赞的基于 SSH 的工具叫 `sshfs`。 sshfs 可以让你在本地直接挂载远程主机的文件系统。它的使用格式如下：

```
sshfs -o idmap=user user@hostname:/home/user ~/Remote
```

比如：

```
sshfs -o idmap=user pi@192.168.0.116:/home/pi ~/Pi
```

这个命令可以将远程主机 pi 用户的主目录挂载到本地主目录下的 Pi 文件夹。
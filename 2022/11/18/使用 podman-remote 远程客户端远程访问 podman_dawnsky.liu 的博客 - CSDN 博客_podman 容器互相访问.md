> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_43902588/article/details/124796454)

《[OpenShift / RHEL / DevSecOps 汇总目录](https://blog.csdn.net/weixin_43902588/article/details/105060359)》

### 文章目录

*   [podman 远程客户](#podman__4)
*   [环境说明](#_10)
*   [节点 1 配置 podman 服务](#_1__podman__17)
*   [节点 2 远程访问 podman](#_2__podman_29)
*   *   [直接访问远程 podman](#_podman_30)
    *   [使用 Connection 访问](#_Connection__42)
    *   [使用 Connection 免密访问](#_Connection__67)
*   [参考](#_96)

podman 远程客户
===========

podman 是替换 [docker](https://so.csdn.net/so/search?q=docker&spm=1001.2101.3001.7020) 的容器环境，它不但可以对本地容器进行操作，还可以使用远程客户端对运行了 podman 服务的远程环境上的容器进行操作。

要从远程访问 podman，需要在被访问一侧运行 podman.[socket](https://so.csdn.net/so/search?q=socket&spm=1001.2101.3001.7020) 服务。客户端通过 SSH 通道访问远程的 podman 服务。  
podman-remote 是专门的客户端程序，它只能用来访问远程 podman 环境。而 podman 程序可以通过 --remote 参数对远程进行访问。  
![](https://img-blog.csdnimg.cn/739c4d4fd78447a48c604557e712ad51.png)

环境说明
====

环境说明：需要 2 个 Linux 节点。用以下命令在节点 1 安装 podman 命令，节点 2 安装 podman-remote。

```
$ yum install podman
$ yum install podman-remote

```

节点 1 配置 podman 服务
=================

说明：dawnsky 是节点 1 的用户，并且能进行 sudo 操作。节点 1 的主机名是 node-1。

```
$ ssh dawnsky@localhost
$ systemctl --user enable podman.socket
Created symlink /home/dawnsky/.config/systemd/user/sockets.target.wants/podman.socket → /usr/lib/systemd/user/podman.socket.
$ systemctl --user start podman.socket
$ sudo loginctl enable-linger $USER
[sudo] dawnsky 的密码：

```

节点 2 远程访问 podman
================

直接访问远程 podman
-------------

1.  可以用 --url 参数访问节点 1 的远程 podman 服务。

```
$ NODE_1_IP=<YOUR-IP>
 
$ podman-remote --url ssh://dawnsky@${NODE_1_IP}/run/user/1000/podman/podman.sock info | grep hostname
Login password:
  hostname: node-1

```

入如果出现以下错误，需要使用 [ssh](https://so.csdn.net/so/search?q=ssh&spm=1001.2101.3001.7020) @ 登录节点 2。  
ERRO[0000] XDG_RUNTIME_DIR directory “/run/user/0” is not owned by the current user

使用 Connection 访问
----------------

1.  可以创建 connection 记录远程访问方式。

```
$ podman system connection add remote-1 ssh://dawnsky@${NODE_1_IP}/run/user/1000/podman/podman.sock
$ podman system connection list
Name        URI                                                             Identity    Default
remote-1    ssh://dawnsky@192.168.1.48:22/run/user/1000/podman/podman.sock              true

```

2.  使用 --connection 访问节点 1 的 podman 服务。如果是 Default 的 connection，可以不用写。由于以上的 connection 没有配置 Identity，因此远程访问还需要提供节点 1 的 dawnsky 密码。

```
$ podman-remote info | grep hostname
Login password:
  hostname: node-1
 
$ podman-remote --connection remote-1 info | grep hostname
Login password:
  hostname: node-1

```

3.  另外还可在节点 2 上用 podman --remote 也会使用远程方式访问。

```
$ podman --remote --url ssh://dawnsky@${NODE_1_IP}/run/user/1000/podman/podman.sock info | grep hostname
Login password:
  hostname: node-1

```

使用 Connection 免密访问
------------------

1.  执行命令生成公钥私钥对，所有都回车即可。

```
$ ssh-keygen -t ed25519

```

2.  将公钥文件的内容复制到节点 1 目标用户下的 ~/.ssh/authorized_keys 文件里。执行完命令可自行确认。

```
$ sh-copy-id -i .ssh/id_ed25519.pub dawnsky@${NODE_1_IP}
dawnsky@192.168.1.48's password:
 
Number of key(s) added: 1
 
Now try logging into the machine, with:   "ssh 'dawnsky@192.168.1.48'"
and check to make sure that only the key(s) you wanted were added.

```

3.  执行命令创建一个新的 connection 配置，其中配置有访问证书。

```
$ podman system connection add --identity .ssh/id_ed25519 remote-no-password ssh://dawnsky@192.168.1.48/run/user/1000/podman/podman.sock
$ podman system connection list
Name                URI                                                             Identity         Default
remote-1            ssh://dawnsky@192.168.1.48:22/run/user/1000/podman/podman.sock                   true
remote-no-password  ssh://dawnsky@192.168.1.48:22/run/user/1000/podman/podman.sock  .ssh/id_ed25519  false

```

4.  用新建的 connection 访问远程的 podman。

```
$ podman-remote --connection remote-no-password info | grep hostname
  hostname: centos-48

```

参考
==

https://www.redhat.com/sysadmin/podman-clients-macos-windows  
https://docs.podman.io/en/latest/markdown/podman-remote.1.html  
https://github.com/containers/podman/blob/main/docs/tutorials/remote_client.md  
https://github.com/containers/podman/blob/main/docs/source/markdown/podman-system-service.1.md
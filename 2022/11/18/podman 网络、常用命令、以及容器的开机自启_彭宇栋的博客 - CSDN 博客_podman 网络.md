> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_49530779/article/details/121956453)

### 目录

*   [1. podman 网络](#1_podman_1)
*   *   [1.1 rootfull 和 rootless 容器网络之间的差异](#11_rootfullrootless_2)
    *   [1.2 防火墙](#12__4)
    *   [1.3 容器间通信示例：](#13__9)
    *   [1.4 查看防火墙规则](#14__84)
*   [2. podman 常用命令](#2_podman_254)
*   [3. 容器的开机自启](#3__430)
*   *   [3.1 root 用户](#31_root_431)
    *   [3.2 普通用户](#32__465)

1. podman 网络
============

1.1 rootfull 和 rootless 容器网络之间的差异
---------------------------------

podman 容器联网的指导因素之一将是容器是否由 root 用户运行。这是因为非特权用户无法在主机上创建网络接口。因此，对于 rootfull 容器，默认网络模式是使用容器网络接口 (CNI) 插件，特别是[桥接](https://so.csdn.net/so/search?q=%E6%A1%A5%E6%8E%A5&spm=1001.2101.3001.7020)插件。对于 rootless，默认的网络模式是 slir4netns。由于权限有限，slirnetns 缺少 CNI 组网的一些功能；例如，slirp4netns 无法为容器提供可路由的 IP 地址。cni 是容器网络接口

1.2 防火墙
-------

防火墙的作用不会影响网络的设置和配置，但会影响这些网络上的流量。最明显的是容器主机的入站网络流量，这些流量通常通过端口映射传递到容器上。根据防火墙的实现，我们观察到防火墙端口由于运行带有端口映射的容器（例如）而自动打开。如果容器流量似乎无法正常工作，请检查防火墙并允许容器正在使用的端口号上的流量。一个常见的问题是重新加载防火墙会删除 cni iptables 规则，从而导致 rootful 容器的网络连接丢失。podman v3 提供了 podman network reload 命令来恢复它而无需重新启动容器。  
**基本网络设置**  
大多数使用 Podman 运行的容器和 Pod 都遵循几个简单的场景。默认情况下，rootfull Podman 将创建一个桥接网络。这是 Podman 最直接和首选的网络设置。桥接网络在内部桥接网络上为容器创建一个接口，然后通过网络地址转换（NAT）连接到互联网。我们还看到用户也希望 macvlan 用于联网。这 macvlan 插件将整个网络接口从主机转发到容器中，允许它访问主机所连接的网络。最后，无根容器的默认网络配置是 slirp4netns。slirp4netns 网络模式功能有限，但可以在没有 root 权限的用户上运行。它创建了一个从主机到容器的隧道来转发流量。

1.3 容器间通信示例：
------------

```
//启动一个test容器
[root@localhost ~]# podman run -it --name test docker.io/library/busybox:latest /bin/sh
Trying to pull docker.io/library/busybox:latest...
Getting image source signatures
Copying blob 3cb635b06aa2 done  
Copying config ffe9d497c3 done  
Writing manifest to image destination
Storing signatures
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0@if4: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue 
    link/ether ca:92:b7:65:48:f5 brd ff:ff:ff:ff:ff:ff
    inet 10.88.0.2/16 brd 10.88.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::c892:b7ff:fe65:48f5/64 scope link 
       valid_lft forever preferred_lft forever
/ # 


```

```
//启动一个test1容器
[root@localhost ~]# podman run -it --name test1 docker.io/library/busybox:latest /bin/sh
/ # ping -c2 10.88.0.2
PING 10.88.0.2 (10.88.0.2): 56 data bytes
64 bytes from 10.88.0.2: seq=0 ttl=64 time=0.060 ms
64 bytes from 10.88.0.2: seq=1 ttl=64 time=0.043 ms

--- 10.88.0.2 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.043/0.051/0.060 ms
/ # 


```

```
//每当启动一个容器就会在宿主机上启动一个veth类型的网卡，当容器停止运行就会关闭
[root@localhost ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:83:47:c1 brd ff:ff:ff:ff:ff:ff
    inet 192.168.8.120/24 brd 192.168.8.255 scope global noprefixroute ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe83:47c1/64 scope link 
       valid_lft forever preferred_lft forever
3: cni-podman0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether c2:67:e8:f8:57:cc brd ff:ff:ff:ff:ff:ff
    inet 10.88.0.1/16 brd 10.88.255.255 scope global cni-podman0
       valid_lft forever preferred_lft forever
    inet6 fe80::c067:e8ff:fef8:57cc/64 scope link 
       valid_lft forever preferred_lft forever
4: vethdd05fdfb@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master cni-podman0 state UP group default 
    link/ether 76:62:fa:93:7c:74 brd ff:ff:ff:ff:ff:ff link-netns cni-670fb712-6556-79c2-1e72-373183ec0e09
    inet6 fe80::7462:faff:fe93:7c74/64 scope link 
       valid_lft forever preferred_lft forever
5: vetha6c5d015@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master cni-podman0 state UP group default 
    link/ether 9a:18:07:37:98:72 brd ff:ff:ff:ff:ff:ff link-netns cni-d382232a-92ec-813c-eeca-b1e3a8ced5e9
    inet6 fe80::9818:7ff:fe37:9872/64 scope link 
       valid_lft forever preferred_lft forever
[root@localhost ~]# 


```

1.4 查看防火墙规则
-----------

```
[root@localhost ~]# iptables -t nat -nvL
Chain PREROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain POSTROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
[root@localhost ~]# 


```

```
//运行一个容器进行测试，当运行一个容器之后就会自动给容器添加一个规则，并放行其端口号
[root@localhost ~]# podman run -d -p 80:80 --name web --rm docker.io/library/httpd
Trying to pull docker.io/library/httpd:latest...
Getting image source signatures
Copying blob 94c13707a187 done  
Copying blob e5ae68f74026 done  
Copying blob bc36ee1127ec done  
Copying blob 0e5b7b813c8c done  
Copying blob a343142ddd8a done  
Copying config 8362f26158 done  
Writing manifest to image destination
Storing signatures
a720319928f7c47f5dda9200fa936a6966501f3ec1870cdc32d6ff84396bc723
[root@localhost ~]# iptables -t nat -nvL
Chain PREROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 CNI-HOSTPORT-DNAT  all  --  *      *       0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain POSTROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 CNI-HOSTPORT-MASQ  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* CNI portfwd requiring masquerade */
    0     0 CNI-cfb28a04db7056bb23430c39  all  --  *      *       10.88.0.4            0.0.0.0/0            /* name: "podman" id: "a720319928f7c47f5dda9200fa936a6966501f3ec1870cdc32d6ff84396bc723" */

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 CNI-HOSTPORT-DNAT  all  --  *      *       0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL

Chain CNI-cfb28a04db7056bb23430c39 (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 ACCEPT     all  --  *      *       0.0.0.0/0            10.88.0.0/16         /* name: "podman" id: "a720319928f7c47f5dda9200fa936a6966501f3ec1870cdc32d6ff84396bc723" */
    0     0 MASQUERADE  all  --  *      *       0.0.0.0/0           !224.0.0.0/4          /* name: "podman" id: "a720319928f7c47f5dda9200fa936a6966501f3ec1870cdc32d6ff84396bc723" */

Chain CNI-HOSTPORT-SETMARK (2 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 MARK       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* CNI portfwd masquerade mark */ MARK or 0x2000

Chain CNI-HOSTPORT-MASQ (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 MASQUERADE  all  --  *      *       0.0.0.0/0            0.0.0.0/0            mark match 0x2000/0x2000

Chain CNI-HOSTPORT-DNAT (2 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 CNI-DN-cfb28a04db7056bb23430  tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* dnat name: "podman" id: "a720319928f7c47f5dda9200fa936a6966501f3ec1870cdc32d6ff84396bc723" */ multiport dports 80

Chain CNI-DN-cfb28a04db7056bb23430 (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 CNI-HOSTPORT-SETMARK  tcp  --  *      *       10.88.0.0/16         0.0.0.0/0            tcp dpt:80
    0     0 CNI-HOSTPORT-SETMARK  tcp  --  *      *       127.0.0.1            0.0.0.0/0            tcp dpt:80
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:80 to:10.88.0.4:80
[root@localhost ~]# podman inspect -l | grep -i address
            "IPAddress": "10.88.0.4",
            "GlobalIPv6Address": "",
            "MacAddress": "a2:3d:36:17:02:05",
            "LinkLocalIPv6Address": "",
                    "IPAddress": "10.88.0.4",
                    "GlobalIPv6Address": "",
                    "MacAddress": "a2:3d:36:17:02:05",
[root@localhost ~]# 

//访问测试
[root@localhost ~]# curl 10.88.0.4
<html><body><h1>It works!</h1></body></html>
[root@localhost ~]# 


```

```
//使用重启容器恢复防火墙规则
[root@localhost ~]# iptables -t nat -F  //清空防火墙规则
[root@localhost ~]# iptables -t nat -nvL   //已经没有80端口
Chain PREROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain POSTROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain CNI-cfb28a04db7056bb23430c39 (0 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain CNI-HOSTPORT-SETMARK (0 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain CNI-HOSTPORT-MASQ (0 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain CNI-HOSTPORT-DNAT (0 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain CNI-DN-cfb28a04db7056bb23430 (0 references)
 pkts bytes target     prot opt in     out     source               destination         
[root@localhost ~]# 


```

```
//重启容器
[root@localhost ~]# podman restart -l
a720319928f7c47f5dda9200fa936a6966501f3ec1870cdc32d6ff84396bc723
[root@localhost ~]# iptables -t nat -nvL
Chain PREROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 CNI-HOSTPORT-DNAT  all  --  *      *       0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain POSTROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 CNI-HOSTPORT-MASQ  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* CNI portfwd requiring masquerade */
    0     0 CNI-cfb28a04db7056bb23430c39  all  --  *      *       10.88.0.5            0.0.0.0/0            /* name: "podman" id: "a720319928f7c47f5dda9200fa936a6966501f3ec1870cdc32d6ff84396bc723" */

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 CNI-HOSTPORT-DNAT  all  --  *      *       0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL

Chain CNI-HOSTPORT-SETMARK (2 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 MARK       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* CNI portfwd masquerade mark */ MARK or 0x2000

Chain CNI-HOSTPORT-MASQ (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 MASQUERADE  all  --  *      *       0.0.0.0/0            0.0.0.0/0            mark match 0x2000/0x2000

Chain CNI-HOSTPORT-DNAT (2 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 CNI-DN-cfb28a04db7056bb23430  tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* dnat name: "podman" id: "a720319928f7c47f5dda9200fa936a6966501f3ec1870cdc32d6ff84396bc723" */ multiport dports 80

Chain CNI-cfb28a04db7056bb23430c39 (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 ACCEPT     all  --  *      *       0.0.0.0/0            10.88.0.0/16         /* name: "podman" id: "a720319928f7c47f5dda9200fa936a6966501f3ec1870cdc32d6ff84396bc723" */
    0     0 MASQUERADE  all  --  *      *       0.0.0.0/0           !224.0.0.0/4          /* name: "podman" id: "a720319928f7c47f5dda9200fa936a6966501f3ec1870cdc32d6ff84396bc723" */

Chain CNI-DN-cfb28a04db7056bb23430 (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 CNI-HOSTPORT-SETMARK  tcp  --  *      *       10.88.0.0/16         0.0.0.0/0            tcp dpt:80
    0     0 CNI-HOSTPORT-SETMARK  tcp  --  *      *       127.0.0.1            0.0.0.0/0            tcp dpt:80
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:80 to:10.88.0.5:80
[root@localhost ~]# 


```

![](https://img-blog.csdnimg.cn/836f4b4ce18e4222bc9b7a794c751134.png)

2. podman 常用命令
==============

podman 官网：[https://docs.podman.io/en/latest/markdown/podman.1.html](https://docs.podman.io/en/latest/markdown/podman.1.html)

<table><thead><tr><th align="left">常用命令</th><th align="left">作用</th></tr></thead><tbody><tr><td align="left">podman attach</td><td align="left">附加到正在运行的容器。</td></tr><tr><td align="left">podman auto update</td><td align="left">根据其自动更新策略自动更新容器</td></tr><tr><td align="left">podman build</td><td align="left">使用 Containerfile 构建容器映像。</td></tr><tr><td align="left">podman commit</td><td align="left">根据更改的容器创建新图像。</td></tr><tr><td align="left">podman completion</td><td align="left">生成 shell 完成脚本</td></tr><tr><td align="left">podman container</td><td align="left">管理容器。</td></tr><tr><td align="left">podman cp</td><td align="left">在容器和本地文件系统之间复制文件 / 文件夹。</td></tr><tr><td align="left">podman create</td><td align="left">创建一个新容器。</td></tr><tr><td align="left">podman diff</td><td align="left">检查容器或映像文件系统上的更改。</td></tr><tr><td align="left">podman events</td><td align="left">监控 Podman 事件</td></tr><tr><td align="left">podman exec</td><td align="left">在正在运行的容器中执行命令。</td></tr><tr><td align="left">podman export</td><td align="left">将容器的文件系统内容导出为 tar 存档。</td></tr><tr><td align="left">podman generate</td><td align="left">基于容器、Pod 或卷生成结构化数据。</td></tr><tr><td align="left">podman healthcheck</td><td align="left">管理容器的健康检查</td></tr><tr><td align="left">podman history</td><td align="left">显示镜像的历史记录。</td></tr><tr><td align="left">podman image</td><td align="left">管理镜像。</td></tr><tr><td align="left">podman images</td><td align="left">列出本地存储中的镜像。</td></tr><tr><td align="left">podman import</td><td align="left">导入 tarball 并将其另存为文件系统映像。</td></tr><tr><td align="left">podman info</td><td align="left">显示 Podman 相关的系统信息.</td></tr><tr><td align="left">podman init</td><td align="left">初始化一个或多个容器</td></tr><tr><td align="left">podman inspect</td><td align="left">显示容器、镜像、卷、网络或 pod 的配置。</td></tr><tr><td align="left">podman kill</td><td align="left">杀死一个或多个容器中的主进程。</td></tr><tr><td align="left">podman load</td><td align="left">将 tar 存档中的图像加载到容器存储中。</td></tr><tr><td align="left">podman login</td><td align="left">登录到容器注册表。</td></tr><tr><td align="left">podman logout</td><td align="left">注销容器注册表。</td></tr><tr><td align="left">podman logs</td><td align="left">显示一个或多个容器的日志。</td></tr><tr><td align="left">podman machine</td><td align="left">管理 Podman 的虚拟机</td></tr><tr><td align="left">podman manifest</td><td align="left">创建和操作清单列表和图像索引。</td></tr><tr><td align="left">podman mount</td><td align="left">挂载一个工作容器的根文件系统。</td></tr><tr><td align="left">podman network</td><td align="left">管理 Podman CNI 网络。</td></tr><tr><td align="left">podman pause</td><td align="left">暂停一个或多个容器。</td></tr><tr><td align="left">podman play</td><td align="left">根据结构化输入文件播放容器、Pod 或卷。</td></tr><tr><td align="left">podman pod</td><td align="left">容器组的管理工具，称为 pod。</td></tr><tr><td align="left">podman port</td><td align="left">列出容器的端口映射。</td></tr><tr><td align="left">podman ps</td><td align="left">从注册表中拉取镜像。</td></tr><tr><td align="left">podman push</td><td align="left">将镜像、清单列表或镜像索引从本地存储推送到其他地方。</td></tr><tr><td align="left">podman rename</td><td align="left">重命名现有容器。</td></tr><tr><td align="left">podman restart</td><td align="left">重启一个或多个容器。</td></tr><tr><td align="left">podman rm</td><td align="left">移除一个或多个容器。</td></tr><tr><td align="left">podman rmi</td><td align="left">删除一个或多个本地存储的镜像。</td></tr><tr><td align="left">podman run</td><td align="left">在新容器中运行命令。</td></tr><tr><td align="left">podman save</td><td align="left">将镜像保存到存档。</td></tr><tr><td align="left">podman search</td><td align="left">在注册表中搜索图像。</td></tr><tr><td align="left">podman secret</td><td align="left">管理 podman 机密。</td></tr><tr><td align="left">podman start</td><td align="left">启动一个或多个容器。</td></tr><tr><td align="left">podman stats</td><td align="left">显示一个或多个容器的资源使用统计的实时流。</td></tr><tr><td align="left">podman stop</td><td align="left">停止一个或多个正在运行的容器。</td></tr><tr><td align="left">podman system</td><td align="left">管理 podman。</td></tr><tr><td align="left">podman tag</td><td align="left">向本地镜像添加附加名称。</td></tr><tr><td align="left">podman top</td><td align="left">显示容器的运行进程。</td></tr><tr><td align="left">podman unmount</td><td align="left">卸载工作容器的根文件系统。</td></tr><tr><td align="left">podman unpause</td><td align="left">取消暂停一个或多个容器。</td></tr><tr><td align="left">podman unshare</td><td align="left">在修改后的用户命名空间内运行命令。</td></tr><tr><td align="left">podman untag</td><td align="left">从本地存储的图像中删除一个或多个名称。</td></tr><tr><td align="left">podman version</td><td align="left">显示 Podman 版本信息。</td></tr><tr><td align="left">podman volume</td><td align="left">简单的卷管理工具。</td></tr><tr><td align="left">podman wait</td><td align="left">等待一个或多个容器停止并打印其退出代码。</td></tr></tbody></table>

**常用命令示例：**

```
//拉取一个nginx镜像
[root@localhost ~]# podman pull docker.io/library/nginx:latest
Trying to pull docker.io/library/nginx:latest...
Getting image source signatures
Copying blob e5ae68f74026 skipped: already exists  
Copying blob 21e0df283cd6 done  
Copying blob 77700c52c969 done  
Copying blob ed835de16acd done  
Copying blob 881ff011f1c9 done  
Copying blob 44be98c0fab6 done  
Copying config f652ca386e done  
Writing manifest to image destination
Storing signatures
f652ca386ed135a4cbe356333e08ef0816f81b2ac8d0619af01e2b256837ed3e

//查看镜像
[root@localhost ~]# podman images
REPOSITORY                 TAG         IMAGE ID      CREATED       SIZE
docker.io/library/httpd    latest      8362f2615893  10 hours ago  148 MB
docker.io/library/busybox  latest      ffe9d497c324  7 days ago    1.46 MB
docker.io/library/nginx    latest      f652ca386ed1  12 days ago   146 MB

//运行一个容器
[root@localhost ~]# podman run -itd --name web01 nginx:latest
dc55487665dee3c036fdcaf98996eec9808fda1666784c6103fe11ef17250eec

//查看容器
[root@localhost ~]# podman ps 
CONTAINER ID  IMAGE                           COMMAND               CREATED         STATUS            PORTS               NAMES
a720319928f7  docker.io/library/httpd:latest  httpd-foreground      16 minutes ago  Up 9 minutes ago  0.0.0.0:80->80/tcp  web
dc55487665de  docker.io/library/nginx:latest  nginx -g daemon o...  8 seconds ago   Up 8 seconds ago                      web01
[root@localhost ~]# 

//进入一个容器
[root@localhost ~]# podman exec -it web01 /bin/bash
root@dc55487665de:/# exit
exit
[root@localhost ~]# 

//"写在容器运行时要执行的命令" 容器id + 将容器重新命名，最好命名为你docker仓库的名字  
// -p是暂停容器
[root@localhost ~]# podman commit -p  -c 

//要传的文件或目录写绝对路径 容器的id或名字:要传到容器的那个目录里面去
[root@localhost ~]# podman cp 

//查看镜像的历史信息
[root@localhost ~]# podman history nginx:latest
ID            CREATED      CREATED BY                                     SIZE        COMMENT
f652ca386ed1  12 days ago  /bin/sh -c #(nop)  CMD ["nginx" "-g" "daem...  0 B         
<missing>     12 days ago  /bin/sh -c #(nop)  STOPSIGNAL SIGQUIT          0 B         
<missing>     12 days ago  /bin/sh -c #(nop)  EXPOSE 80                   0 B         
<missing>     12 days ago  /bin/sh -c #(nop)  ENTRYPOINT ["/docker-en...  0 B         
<missing>     12 days ago  /bin/sh -c #(nop) COPY file:09a214a3e07c91...  7.17 kB     
<missing>     12 days ago  /bin/sh -c #(nop) COPY file:0fd5fca330dcd6...  3.58 kB     
<missing>     12 days ago  /bin/sh -c #(nop) COPY file:0b866ff3fc1ef5...  4.1 kB      
<missing>     12 days ago  /bin/sh -c #(nop) COPY file:65504f71f5855c...  3.07 kB     
<missing>     12 days ago  /bin/sh -c set -x     && addgroup --system...  62 MB       
<missing>     12 days ago  /bin/sh -c #(nop)  ENV PKG_RELEASE=1~bullseye  0 B         
<missing>     12 days ago  /bin/sh -c #(nop)  ENV NJS_VERSION=0.7.0       0 B         
<missing>     12 days ago  /bin/sh -c #(nop)  ENV NGINX_VERSION=1.21.4    0 B         
<missing>     12 days ago  /bin/sh -c #(nop)  LABEL maintainer=NGINX ...  0 B         
<missing>     13 days ago  /bin/sh -c #(nop)  CMD ["bash"]                0 B         
<missing>     13 days ago  /bin/sh -c #(nop) ADD file:ece5ff85ca549f0...  83.9 MB     
[root@localhost ~]# 


```

```
//删除容器 -f强制删除
[root@localhost ~]# podman ps
CONTAINER ID  IMAGE                           COMMAND               CREATED         STATUS             PORTS               NAMES
a720319928f7  docker.io/library/httpd:latest  httpd-foreground      20 minutes ago  Up 13 minutes ago  0.0.0.0:80->80/tcp  web
dc55487665de  docker.io/library/nginx:latest  nginx -g daemon o...  3 minutes ago   Up 3 minutes ago                       web01
[root@localhost ~]# podman rm -f web web01 
dc55487665dee3c036fdcaf98996eec9808fda1666784c6103fe11ef17250eec
a720319928f7c47f5dda9200fa936a6966501f3ec1870cdc32d6ff84396bc723
[root@localhost ~]# docker ps -a
-bash: docker: 未找到命令
[root@localhost ~]# podman ps -a
CONTAINER ID  IMAGE                             COMMAND     CREATED         STATUS                       PORTS       NAMES
694bdbece78d  docker.io/library/busybox:latest  /bin/sh     28 minutes ago  Exited (0) 23 minutes ago                test
8f6be5b77189  docker.io/library/busybox:latest  /bin/sh     27 minutes ago  Exited (127) 23 minutes ago              test1
[root@localhost ~]# podman rm -f test test1
694bdbece78d0686164f18a37e8183a470434052950365f0f10ae7f91c29e739
8f6be5b77189be990cc1ca9e0b171fed482eaf5e1da529a16a23d1a24771ff8c
[root@localhost ~]# podman ps -a
CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES
[root@localhost ~]# 


```

```
//删除镜像
[root@localhost ~]# podman images -aq
8362f2615893
ffe9d497c324
f652ca386ed1
[root@localhost ~]# podman rmi $(podman images -aq)
Untagged: docker.io/library/httpd:latest
Untagged: docker.io/library/busybox:latest
Untagged: docker.io/library/nginx:latest
Deleted: 8362f2615893c70073d4b6576c884eade7f6ce81482780ac989d535b621750f9
Deleted: ffe9d497c32414b1c5cdad8178a85602ee72453082da2463f1dede592ac7d5af
Deleted: f652ca386ed135a4cbe356333e08ef0816f81b2ac8d0619af01e2b256837ed3e
[root@localhost ~]# podman images
REPOSITORY  TAG         IMAGE ID    CREATED     SIZE
[root@localhost ~]# 


```

3. 容器的开机自启
==========

3.1 root 用户
-----------

```
//创建一个nginx容器
[root@localhost ~]# podman create --name nginx nginx
dfa08c036fc1ca6c8a20ec69edf76c00f11c9b00d1304876d2ffa1e2eb5c299f

//生成开机自启文件
[root@localhost ~]# podman generate systemd --files --name nginx
/root/container-nginx.service
[root@localhost ~]# mv container-nginx.service  /usr/lib/systemd/system/
mv：是否覆盖'/usr/lib/systemd/system/container-nginx.service'？ yes
[root@localhost ~]# systemctl enable --now container-nginx
Created symlink /etc/systemd/system/multi-user.target.wants/container-nginx.service → /usr/lib/systemd/system/container-nginx.service.
Created symlink /etc/systemd/system/default.target.wants/container-nginx.service → /usr/lib/systemd/system/container-nginx.service.
[root@localhost ~]# systemctl status container-nginx
● container-nginx.service - Podman container-nginx.service
   Loaded: loaded (/usr/lib/systemd/system/container-nginx.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2021-12-16 09:18:43 CST; 7s ago
     Docs: man:podman-generate-systemd(1)
  Process: 7792 ExecStart=/usr/bin/podman start nginx (code=exited, status=0/SUCCESS)
 Main PID: 7885 (conmon)
    Tasks: 3 (limit: 23485)
   Memory: 6.1M
   CGroup: /system.slice/container-nginx.service
           ├─7826 /usr/bin/fuse-overlayfs -o metacopy=on,lowerdir=/var/lib/containers/storage/overlay/l/UE>
           └─7885 /usr/bin/conmon --api-version 1 -c dfa08c036fc1ca6c8a20ec69edf76c00f11c9b00d1304876d2ffa>

12月 16 09:18:43 localhost.localdomain systemd[1]: Starting Podman container-nginx.service...
12月 16 09:18:43 localhost.localdomain podman[7792]: nginx
12月 16 09:18:43 localhost.localdomain systemd[1]: Started Podman container-nginx.service.



```

3.2 普通用户
--------

```
[root@localhost ~]# yum -y install crun
[root@localhost ~]# vim /usr/share/containers/containers.conf
[root@localhost ~]# sed -n '433,434p' /usr/share/containers/containers.conf
runtime = "crun"   取消#
#runtime = "runc"  注释掉
[root@localhost ~]# vim /etc/containers/storage.conf
[root@localhost ~]# sed -n '77p' /etc/containers/storage.conf
mount_program = "/usr/bin/fuse-overlayfs"   #取消注释

//创建用户
[root@localhost ~]# useradd iu
[root@localhost ~]# echo "1"|passwd --stdin iu
更改用户 iu 的密码 。
passwd：所有的身份验证令牌已经成功更新。
[root@localhost ~]# ssh iu@192.168.8.120
iu@192.168.8.120's password: 1

//必须在家目录下创建此目录。不能跟改名字
[iu@localhost ~]$ mkdir -p ~/.config/systemd/user
[iu@localhost ~]$ cd ~/.config/systemd/user

//创建一个容器
[iu@localhost user]$ podman run -d --name test nginx
✔ docker.io/library/nginx:latest
Trying to pull docker.io/library/nginx:latest...
Getting image source signatures
Copying blob 881ff011f1c9 done  
Copying blob ed835de16acd done  
Copying blob 21e0df283cd6 done  
Copying blob e5ae68f74026 done  
Copying blob 44be98c0fab6 done  
Copying blob 77700c52c969 done  
Copying config f652ca386e done  
Writing manifest to image destination
Storing signatures
2d97bbed668d48df206957fcbd64dca89a722b58211476305941c69ffdf3b2c0
[iu@localhost user]$ podman generate systemd --name test --files --new
/home/iu/.config/systemd/user/container-test.service

//停止容器
[iu@localhost user]$ podman stop test
test
[iu@localhost user]$ podman ps
CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES

//如果不是ssh登陆或重新进入linux系统的需重新加载系统服务
[iu@localhost user]$ systemctl --user daemon-reload
[iu@localhost user]$ systemctl --user enable --now container-test.service 
Created symlink /home/iu/.config/systemd/user/multi-user.target.wants/container-test.service → /home/iu/.config/systemd/user/container-test.service.
Created symlink /home/iu/.config/systemd/user/default.target.wants/container-test.service → /home/iu/.config/systemd/user/container-test.service.
[iu@localhost user]$ podman ps
CONTAINER ID  IMAGE                           COMMAND               CREATED        STATUS             PORTS       NAMES
2d8b2a5d59fd  docker.io/library/nginx:latest  nginx -g daemon o...  9 seconds ago  Up 10 seconds ago              test
[iu@localhost user]$ systemctl --user status container-test.service 
● container-test.service - Podman container-test.service
   Loaded: loaded (/home/iu/.config/systemd/user/container-test.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2021-12-16 09:06:56 CST; 26s ago
     Docs: man:podman-generate-systemd(1)
  Process: 6961 ExecStartPre=/bin/rm -f /run/user/1001/container-test.service.ctr-id (code=exited, status=>
 Main PID: 7006 (conmon)
   CGroup: /user.slice/user-1001.slice/user@1001.service/container-test.service
           ├─7001 /usr/bin/fuse-overlayfs -o ,lowerdir=/home/iu/.local/share/containers/storage/overlay/l/>
           ├─7003 /usr/bin/slirp4netns --disable-host-loopback --mtu=65520 --enable-sandbox --enable-secco>
           ├─7006 /usr/bin/conmon --api-version 1 -c 2d8b2a5d59fdf1f5bee2b31550ae27bfa414a793a5b96d39a13f3>
           ├─7009 nginx: master process nginx -g daemon off;
           ├─7035 nginx: worker process
           ├─7036 nginx: worker process
           ├─7037 nginx: worker process
           └─7038 nginx: worker process


```
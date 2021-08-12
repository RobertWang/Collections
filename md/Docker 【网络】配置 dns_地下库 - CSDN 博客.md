> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/xixihahalelehehe/article/details/114979382)

### 文章目录

*   *   [1. docker 配置 DNS 方法](#1_dockerDNS_3)
    *   [2. 默认 DNS 配置](#2_DNS_10)
    *   [3. 启动时配置 dns 参数](#3_dns_22)
    *   [4. daemon.json 配置 DNS 格式](#4_daemonjsonDNS_56)

1. docker 配置 DNS 方法
-------------------

docker 容器配置 dns 解析地址，我知道的有以下几种办法（优先级从高到低）：

1.  启动的时候加–dns=IP_ADDRESS；
2.  守护进程启动参数中添加 DOCKER_OPTS="–dns 8.8.8.8" ；
3.  在 / etc/docker/deamon.json 中添加 dns 信息（与守护进程参数会冲突不能同时添加。）；
4.  使用宿主机的 / etc/resolv.conf 文件；

2. 默认 DNS 配置
------------

怎样为 Docker 提供的每一个容器进行主机名和 DNS 配置，而不必建立自定义镜像并将主机名写 到里面？它的诀窍是覆盖三个至关重要的在 / etc 下的容器内的虚拟文件，那几个文件可以写入 新的信息。你可以在容器内部运行 mount 看到这个：

```
$ mount
...
/dev/disk/by-uuid/1fec...ebdf on /etc/hostname type ext4 ...
/dev/disk/by-uuid/1fec...ebdf on /etc/hosts type ext4 ...
/dev/disk/by-uuid/1fec...ebdf on /etc/resolv.conf type ext4 ...
...

```

3. 启动时配置 dns 参数
---------------

<table><thead><tr><th>Options</th><th>Description</th></tr></thead><tbody><tr><td>-h HOSTNAME or --hostname=HOSTNAME</td><td>在该容器启动时，将 HOSTNAME 设置到容器内的 / etc/hosts, /etc/hostname, /bin/bash 提示中。</td></tr><tr><td>–link=CONTAINER_NAME or ID:ALIAS</td><td>在该容器启动时，将 ALIAS 和 CONTAINER_NAME/ID 对应的容器 IP 添加到 / etc/hosts. 如果 CONTAINER_NAME/ID 有多个 IP 地址 ？</td></tr><tr><td>–dns=IP_ADDRESS…</td><td>在该容器启动时，将 nameserver IP_ADDRESS 添加到容器内的 / etc/resolv.conf 中。可以配置多个。</td></tr><tr><td>–dns-search=DOMAIN…</td><td>在该容器启动时，将 DOMAIN 添加到容器内 / etc/resolv.conf 的 dns search 列表中。可以配置多个。</td></tr><tr><td>–dns-opt=OPTION…</td><td>在该容器启动时，将 OPTION 添加到容器内 / etc/resolv.conf 中的 options 选项中，可以配置多个</td></tr></tbody></table>

如果 docker run 时不含`--dns=IP_ADDRESS`…, `--dns-search=DOMAIN`…, or `--dns-opt=OPTION`… 参数，docker daemon 会将 copy 本主机的`/etc/resolv.conf`，然后对该 copy 进行处理（将那些 / etc/resolv.conf 中 ping 不通的 nameserver 项给抛弃）, 处理完成后留下的部分就作为该容器内部的 / etc/resolv.conf。因此，如果你想利用宿主机中的 / etc/resolv.conf 配置的 nameserver 进行域名解析，那么你需要宿主机中该 dns service 配置一个宿主机内容器能 ping 通的 IP。  
如果宿主机的 / etc/resolv.conf 内容发生改变，docker daemon 有一个对应的 file change notifier 会 watch 到这一变化，然后根据容器状态采取对应的措施：

*   如果容器状态为 stopped，则立刻根据宿主机的 / etc/resolv.conf 内容更新容器内的 / etc/resolv.conf.
*   如果容器状态为 running，则容器内的 / etc/resolv.conf 将不会改变，直到该容器状态变为 stopped.
*   如果容器启动后修改过容器内的 / etc/resolv.conf，则不会对该容器进行处理，否则可能会丢失已经完成的修改，无论该容器为什么状态。
*   如果容器启动时，用了–dns, --dns-search, or --dns-opt 选项，其启动时已经修改了宿主机的 / etc/resolv.conf 过滤后的内容，因此 docker daemon 永远不会更新这种容器的 / etc/resolv.conf。

> 注意: docker daemon 监控宿主机 / etc/resolv.conf 的这个 file change notifier 的实现是依赖 linux 内核的 inotify 特性，而 inotfy 特性不兼容 overlay fs，因此使用 overlay fs driver 的 docker deamon 将无法使用该 / etc/resolv.conf 自动更新的功能。、

```
 $ sudo docker run --hostname 'myhost' -it centos
 [root@myhost /]# cat /etc/hosts
 172.17.0.7    myhost

 $  sudo docker run -it --dns=192.168.5.1  centos
 [root@6a38049c9052 /]# cat /etc/resolv.conf
 nameserver 192.168.5.1

 $  sudo docker run -it --dns-search=www.domain.com  centos
 [root@ae0e9e99596f /]# cat /etc/resolv.conf
 nameserver 192.168.4.1
 search www.mydomain.com

```

4. daemon.json 配置 DNS 格式
------------------------

```
root@node-7:~# cat /etc/docker/daemon.json
{
  "data-root": "/data/docker",
  "dns": ["172.18.0.52", "172.18.0.70", "183.XX.XX.XX"],
  "dns-search": ["fiibeacon.local"],
  "hosts": ["unix:///var/run/docker.sock", "tcp://172.18.0.141:2375"],
  "storage-driver": "overlay2"
}

```

参考连接：  
[docker 高级网络配置](http://www.dockerinfo.net/%E9%AB%98%E7%BA%A7%E7%BD%91%E7%BB%9C%E9%85%8D%E7%BD%AE)  
[docker container DNS 配置介绍和源码分析](https://cloud.tencent.com/developer/article/1096388)
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/whatday/article/details/104094119)

1. 缺少 ps 命令

apt-get install -y procps

yum install -y procps

2. 缺少 pstree 命令

apt-get install -y psmisc

yum install -y psmisc

3. 出现找不到包的问题：

```
root@7ff7c2b8677e:/# apt-get install -y procps
Reading package lists... Done
Building dependency tree
Reading state information... Done
E: Unable to locate package procps
```

更新一下就可以了：

```
apt-get update

```

再安装：

```
root@7ff7c2b8677e:/# apt-get install -y procps
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following additional packages will be installed:
  libprocps6 psmisc
The following NEW packages will be installed:
  libprocps6 procps psmisc
0 upgraded, 3 newly installed, 0 to remove and 86 not upgraded.
Need to get 431 kB of archives.
After this operation, 1434 kB of additional disk space will be used.
Get:1 http://deb.debian.org/debian stretch/main amd64 libprocps6 amd64 2:3.3.12-3+deb9u1 [58.5 kB]
Get:2 http://deb.debian.org/debian stretch/main amd64 procps amd64 2:3.3.12-3+deb9u1 [250 kB]
Get:3 http://deb.debian.org/debian stretch/main amd64 psmisc amd64 22.21-2.1+b2 [123 kB]
Fetched 431 kB in 0s (456 kB/s)
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package libprocps6:amd64.
(Reading database ... 17157 files and directories currently installed.)
Preparing to unpack .../libprocps6_2%3a3.3.12-3+deb9u1_amd64.deb ...
Unpacking libprocps6:amd64 (2:3.3.12-3+deb9u1) ...
Selecting previously unselected package procps.
Preparing to unpack .../procps_2%3a3.3.12-3+deb9u1_amd64.deb ...
Unpacking procps (2:3.3.12-3+deb9u1) ...
Selecting previously unselected package psmisc.
Preparing to unpack .../psmisc_22.21-2.1+b2_amd64.deb ...
Unpacking psmisc (22.21-2.1+b2) ...
Setting up psmisc (22.21-2.1+b2) ...
Setting up libprocps6:amd64 (2:3.3.12-3+deb9u1) ...
Setting up procps (2:3.3.12-3+deb9u1) ...
update-alternatives: using /usr/bin/w.procps to provide /usr/bin/w (w) in auto mode
Processing triggers for libc-bin (2.24-11+deb9u3) ...
root@7ff7c2b8677e:/#
```

就可以成功了
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qazwsx_19931126/article/details/108172610)

1.  删除某软件, 及其安装时自动安装的所有包

```
sudo apt-get autoremove docker docker-ce docker-engine  docker.io  containerd runc

```

2.  删除 docker 其他没有没有卸载

```
dpkg -l | grep docker
dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P # 删除无用的相关的配置文件

```

3.  卸载没有删除的 docker 相关插件 (结合自己电脑的实际情况)

```
sudo apt-get autoremove docker-ce-*

```

4.  删除 docker 的相关配置 & 目录

```
 sudo rm -rf /etc/systemd/system/docker.service.d
 sudo rm -rf /var/lib/docker

```

5.  确定 docker 卸载完毕

```
docker -version

```

[原文地址：侵权必删](https://www.cnblogs.com/shmily3929/p/12085163.html)
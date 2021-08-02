> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [iimx.net](https://iimx.net/2019/04/02/docker%E5%9F%BA%E4%BA%8Ealpine%E5%AE%9A%E5%88%B6%E9%95%9C%E5%83%8F/)

> docker 基于 alpine 定制镜像

发表于 2019-04-02   |   分类于 [笔记](https://iimx.net/categories/%E7%AC%94%E8%AE%B0/)

docker 基于 alpine 定制镜像  

拉取 alpine 镜像
------------

```
docker pull alpine
```

创建并进入容器
-------

```
docker run -it alpine /bin/sh  # 创建进入容器
exit  # 退出

docker ps -a  # 查看刚刚创建的容器id
docker start <container id>  # 启动容器
docker exec -it <container id> sh  # 进入容器
```

修改容器
----

```
# 修改源
sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories
# 修改时区
apk add -U tzdata
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
apk del tzdata
```

创建镜像
----

```
docker commit -a "镜像作者" -m "提交文字" <container id> <新镜像名称>:<tag>
```

导出导入镜像
------

```
docker save -o 导出名称.tar <image name>:<tag>  # 导出镜像
docker load < xxx.tar  # 导入镜像
```
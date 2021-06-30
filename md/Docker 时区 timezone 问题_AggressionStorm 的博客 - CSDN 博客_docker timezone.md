> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/AggressionStorm/article/details/100041070)

### 文章目录

*   *   *   [Linux 时间类型](#Linux_1)
        *   [docker 时间、时区问题](#docker_5)
        *   *   [docker-compose 启动时的设置：](#dockercompose_8)
            *   [dockerfile 进行镜像设置生成](#dockerfile_20)
            *   [容器启动时直接设置](#_29)

### Linux 时间类型

*   在 Unix 类的机器下的`/usr/share/zoneinfo/`文件内为所有代码调用的`ZONEINFO`的数据位置，想查看设置哪个时区时，直接去里边看名字即可。  
    ![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9ub3RlLnlvdWRhby5jb20veXdzL3B1YmxpYy9yZXNvdXJjZS82OTM2N2FhNjA0ZWYzZjZmYmQ3ZmUxNmFjNjJlODM3MC94bWxub3RlL0JEQjdCNUU4M0YyMjQzQ0NBRDc2NzVERDFBMkFGN0NGLzQ1ODA?x-oss-process=image/format,png)

### docker 时间、时区问题

> docker 容器内默认为 utc 时间

#### docker-compose 启动时的设置：

设置容器内为宿主机时间：

```
volumes:
  - /etc/localtime:/etc/localtime:ro
  - /etc/timezone:/etc/timezone:ro   # 这个只在Linux上有
  # “ro”的意思是只读(read-only)模式，可以保证其挂载卷不被 Docker 容器内部文件系统配置所覆盖

# 通过环境变量设置时区    
environment:
  - TZ=Asia/Shanghai   # 设置容器时区为CST

```

#### dockerfile 进行镜像设置生成

```
RUN echo "Asia/Shanghai" > /etc/timezone
RUN dpkg-reconfigure -f noninteractive tzdata

# 已上是 Ubuntu 修改时区的命令。
# Docker 默认使用 Ubuntu系统。
# 如果你的自定义镜像使用的是其他发行版，那么这里的命令也要改变

```

#### 容器启动时直接设置

```
docker run -v /etc/localtime:/etc/localtime <IMAGE:TAG>

```
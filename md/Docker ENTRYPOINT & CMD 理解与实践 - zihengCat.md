> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zihengcat.github.io](https://zihengcat.github.io/2019/06/12/understanding-and-practicing-docker-entrypoint-cmd/)

> ziheng's Blog

Docker 指令`CMD`与`ENTRYPOINT`用于**配置容器启动时的运行命令**，通常会写在`Dockerfile`中。本文讲解`CMD`与`ENTRYPOINT`这两条指令的异同点，并通过一个实例指导你如何在`Dockerfile`中使用它们。

`ENTRYPOINT`指令的语义为 **” 容器入口点”**，该指令用于设置容器首次启动时需要执行的命令（Command）及其参数（Parameters）。

任何传递至`docker run <image> ...`的命令行参数都会被附加到`ENTRYPOINT`命令后，并覆盖原有`CMD`指令。

Docker 规定，在`Dockerfile`中使用全大写字母的`ENTRYPOINT`指令，`ENTRYPOINT`有两种形式： _exec 形式_ 与 _shell 命令形式_ 。

*   ENTRYPOINT [“executable”, “param1”, “param2”]
    
*   ENTRYPOINT command param1 param2
    

Docker `ENTRYPOINT` 指令的最佳实践是：准备一枚`entrypoint.sh`启动脚本，将程序运行的具体逻辑放到启动脚本之中。

例如，Postgres 官方镜像`Dockerfile`中，就使用了一枚启动脚本作为容器入口点`ENTRYPOINT`。

```
...
COPY ./docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["postgres"]
...
```

> 注：Postgres Official Image `Dockerfile`

Docker `ENTRYPOINT` 指令也是可以被重载的（虽然不常用），我们可以使用`docker run --entrypoint`重载`ENTRYPOINT`指令。

Docker `CMD` 指令也可以设置容器启动时需要执行的命令（Command）及其参数（Parameters），但`CMD`指令的主要目的提供可运行容器的默认命令，并为`ENTRYPOINT`指令提供默认参数。`CMD`指令将会在`ENTRYPOINT`后执行。

`CMD`指令有三种形式，如果省略执行命令，`CMD`将为`ENTRYPOINT`提供默认参数。

*   CMD [“executable”, “param1”, “param2”]
    
*   CMD [“param1”, “param2”]
    
*   CMD command param1 param2
    

使用`docker run <image> ...`启动容器，传递至`docker run <image> ...`的命令行参数会覆盖掉原有`CMD`指令。

Docker `CMD` 指令的最佳实践为：为`ENTRYPOINT`指令提供默认参数。

> 注：在`Dockerfile`中，只允许一条`CMD`指令出现，如果出现了多条，只有最后一条`CMD`指令生效。

接下来，我们手写一枚`Redis`镜像`Dockerfile`，来理解与实践`CMD`与`ENTRYPOINT`。

```
# -------------------------
#  Dockerfile - Redis 5.0.5
# -------------------------
# ================================
# Using Base OS -> CentOS 7 Latest
# ================================
FROM       docker.io/centos:latest
# =============================================
# LABEL following the standard set of labels by
# The Open Containers Initiative (OCI)
# =============================================
LABEL      org.opencontainers.image.title="DockerImage - Redis" \
           org.opencontainers.image.authors="zihengCat" \
           org.opencontainers.image.version="1.0.0" \
           org.opencontainers.image.licenses="MIT" \
# ==============================================================
RUN        PKG_BASE_DEPS="epel-release \
                          make \
                          gcc \
                          gcc-c++" && \
           REDIS_PKG_PATH="/root/redis-5.0.5.tar.gz" && \
           REDIS_PKG_ROOT="/root/redis-5.0.5/" && \
           REDIS_PKG_INSTALLED="/opt/redis_5.0.5/" && \
           REDIS_PKG_URL="http://download.redis.io/releases/redis-5.0.5.tar.gz" && \
           cp "/usr/share/zoneinfo/Asia/Shanghai" "/etc/localtime" && \
           echo "Asia/Shanghai" > "/etc/timezone" && \
           yum -y install "epel-release" && \
           yum -y install ${PKG_BASE_DEPS} && \
           curl -o ${REDIS_PKG_PATH} ${REDIS_PKG_URL} && \
           cd "/root/" && \
           tar -xvz -f ${REDIS_PKG_PATH} && \
           cd ${REDIS_PKG_ROOT} && \
           make V=1 && \
           make PREFIX=${REDIS_PKG_INSTALLED} install && \
           rm -rf ${REDIS_PKG_ROOT} ${REDIS_PKG_PATH} && \
           yum -y autoremove ${PKG_BASE_DEPS} && \
           yum clean all
# ======================================
WORKDIR    /root/
# ======================================
# Using a Docker `entrypoint.sh` and CMD
# ======================================
COPY ./entrypoint.sh /root/
ENTRYPOINT ["sh", "/root/entrypoint.sh"]
CMD ["redis-cli"]
# ======================================
# EOF
```

> 代码清单：`Redis`镜像`Dockerfile`

这里，我们使用一枚启动脚本`entrypoint.sh`作为容器入口点`ENTRYPOINT`，该脚本接受命令行参数并执行相应的命令。使用`CMD`指令为启动脚本提供默认参数`redis-cli`。

```
#!/bin/bash
# -------------------------------
set -e
# -------------------------------
REDIS_FLAG=false
REDIS_PATH='/opt/redis_5.0.5/bin'
REDIS_COMMANDS="redis-cli \
                redis-server"
# -------------------------------
for redis_cmd in ${REDIS_COMMANDS}
do
    if [[ ${1} == ${redis_cmd} ]]
    then
        REDIS_FLAG=true
    fi
done
# -------------------------------
if [[ ${REDIS_FLAG} == true ]]
then
    ${REDIS_PATH}/${@}
else
    echo '[ERROR]: No such Redis command...'
    exit 1
fi
# -------------------------------
# EOF
```

> 代码清单：`Redis`镜像`entrypoint.sh`

使用`docker build`构建起`Redis`镜像后，可在`docker run`后附加容器需执行的`Redis`命令及其参数。

```
#!/bin/bash
# ------------------------------
REDIS_COMMAND="redis-server"
REDIS_OPTIONS="--loglevel verbose \
               --bind 0.0.0.0 \
               --port 6379 \
               --requirepass foobar123 \
               --maxclients 10000 \
               --timeout 0 \
               --tcp-keepalive 300 \
               --daemonize no \
               --supervised no \
               --pidfile none \
               --databases 32 \
               --always-show-logo yes \
               --appendonly no \
               --appendfsync no"
# ------------------------------
sudo docker run -d -i -t \
                --env LC_ALL=en_US.UTF-8 \
                --name redis_test \
                -p 0.0.0.0:6379:6379 \
                ziheng/redis:latest \
                ${REDIS_COMMAND} ${REDIS_OPTIONS}
# ------------------------------
# EOF
```

> 代码清单：`Redis`镜像`docker run`启动命令

Docker `CMD` 与 `ENTRYPOINT` 指令都可以定义容器启动时执行的命令，但它们有所不同。

*   `Dockerfile`中只应出现一条`CMD`或`ENTRYPOINT`指令
    
*   `ENTRYPOINT`定义容器启动时的运行命令
    
*   `CMD`为`ENTRYPOINT`提供默认参数，或提供一枚单次执行的`Ad-Hoc`命令
    
*   `CMD`可以很容易地被容器运行时指定的命令行参数重载
    

*   Docker’s Dockerfile Reference: [https://docs.docker.com/engine/reference/builder/](https://docs.docker.com/engine/reference/builder/)

* * *
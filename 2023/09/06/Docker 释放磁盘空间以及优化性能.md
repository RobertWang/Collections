> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/gbbdy0f8IsDl04U6TS6MFA)

在 Docker 的日常使用中，我们或许偶尔遇到下面这些情况：

```
$ docker-compose ps
[27142] INTERNAL ERROR: cannot create temporary directory!


$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        489M  132K  488M   1% /dev
tmpfs           497M     0  497M   0% /dev/shm
/dev/xvda1      7.8G  7.7G     0 100% /


```

这时候，我们大概明白了，大概是 Docker 把磁盘占满了。这时就需要我们去做一些清理了，这篇博客主要总结如下一些有效的 Docker 清理操作。

> 在了解这些之前，我相信你已经对下面的一些词汇已经有一定的了解。

*   image: 一个只读模版，可以用来创建 container。如，一个包含 ubuntu 系统的镜像。
    
*   container: 从镜像创建的运行实例。可以用 docker 命令去控制这些 container。
    
*   volume: docker 数据持久化。
    
*   dangling: 未使用的 image。
    
*   network: 连接 docker 容器服务。
    

使用 `docker system df`即可查看：

```
$ docker system df
TYPE              TOTAL    ACTIVE      SIZE               RECLAIMABLE
Images            13       7           6.4 GB              7.2 MB (88%)
Containers        8        0           42.85 MB            42.85 MB (100%)
Local Volumes     5        5           3.541 GB            0 B (0%)


```

如上，镜像占了 `6.4GB`, 容器占了 `42.85 MB`, 数据卷占了 `3.541 GB`。了解基本占用后，我们就可以用下面介绍的命令进行针对性的清理了。

要清理 docker, 就要知道 docker 数据在哪，具体有哪些 docker 进程。

docker 提供了一些快捷的命令去清除未使用的容器，网络和镜像：

```
$ docker system prune
WARNING! This will remove:
        - all stopped containers
        - all networks not used by at least one container
        - all dangling images
        - all dangling build cache
Are you sure you want to continue? [y/N]


```

> 默认是没有清除数据卷的功能，由于数据比较重要，防止意外删除一些数据。可以通过 `--volumes`指定。

同时，我们还可以将 `—all`清除未使用的 images。使用 `--force`免确认。

```
$ docker system prune --all --volumes
WARNING! This will remove:
        - all stopped containers
        - all networks not used by at least one container
        - all volumes not used by at least one container
        - all images without at least one container associated to them
        - all build cache
Are you sure you want to continue? [y/N]


```

当然，我们也可以单独清除。

*   `docker container prune` 清除停止的容器；
    
*   `docker volume prune` 清除未使用的数据卷；
    
*   `docker image prune` 清除未使用的镜像；
    

上面的命令不会影响运行中的容器以及关联的镜像，数据卷和网络。如果你需要全部清理就需要将所有容器都停止下来。

使用 `docker container stop [CONTAINERS...]`能停止正在运行的容器。同时我们可以通过下面命令获取正在运行的容器 ID。

```
$ docker container ls -aq


```

*   `ls` 列出所有容器；
    
*   `--all / -a`列出所有容器（包含未运行的）；
    
*   `--quiet / -q` 只显示容器 ID；
    

于是我们可以使用下面命令停止所有容器：

```
$ docker container stop $(docker container ls -a -q)


```

结合清除的命令，完整的清除所有的容器命令如下：

```
$ docker container stop $(docker container ls -a -q) && docker system prune -a -f --volumes


```

同理，我们可以想到：

*   清除容器 `docker container rm $(docker container ls -a -q)` / `docker rm $(docker ps -a -q)`;
    
*   清除镜像 `docker image rm $(docker images ls -a -q)`;
    
*   清除数据卷 `docker volume rm $(docker volume ls -q)`;
    
*   清除网络 `docker network rm $(docker network ls -q)`;
    

很多时候，我们发现我们都是被日志文件撑爆的，解决问题的源头就是限制容器日志大小，方法有三：

*   修改 `daemon.json`配置；
    

```
{
    "log-opts": {
        "max-size" : "521m"
    }
}


```

*   修改 `docker-compose`；
    

```
ubuntu:
  image: ubuntu
  restart: always
  logging:
    driver: "json-file"
    options:
      max-size: "1g"


```

*   通过参数；
    

```
$ docker run -d --log-opt max-size=1g ubuntu


```

*   清除指定日志文件；如果你要删除指定容器的日志，只有几步即可。
    

```
$ docker inspect 2ed640d8fcd1  --format '{{.LogPath}}'
/mnt/data/docker/containers/2ed640d8fcd1bd464a23be78513d23be1807c8ad6a95116da5cb9118a6b2380a/2ed640d8fcd1bd464a23be78513d23be1807c8ad6a95116da5cb9118a6b2380a-json.log


```

知道了日志地址，你就可以删除或清空该日志了，不过注意权限哦～

*   杀死所有正在运行的容器；
    

```
$ docker kill $(docker ps -a -q)


```

*   删除所有已经停止的容器；
    

```
$ docker rm $(docker ps -a -q)


```

*   删除未打标签的镜像；
    

```
$ docker rmi $(docker images -q -f dangling=true)


```

*   批量删除指定镜像 / 容器等；我们可以通过 `--format`指出 docker 命令的输出形式，通过 `grep`去筛选，然后删除。如：
    

```
$ docker rmi $(docker images --format '{{.Repository}}' | grep 'hub.docker.com')

$ docker kill $(docker ps -a --format '{{.Images}}' | grep 'ubuntu')


```

当使用 Docker 进行日常开发和部署时，我们经常会遇到需要进行清理操作的情况。在前面的部分，我们介绍了一些有效的 Docker 清理操作，包括查看磁盘使用情况、找到 Docker 数据与进程、清除未使用的镜像、容器、数据卷和网络，以及重置 Docker 和限制容器日志大小等。现在，让我们继续探讨一些其他常见的 Docker 清理命令和一些建议。

清除未使用的构建缓存
----------

在使用 Docker 构建镜像时，会生成一些中间构建映像和缓存。这些构建缓存可能会占据大量磁盘空间，尤其是在多次构建镜像后。为了释放磁盘空间，可以使用 `docker builder prune`命令清除未使用的构建缓存。

```
$ docker builder prune


```

这将删除所有未使用的中间构建映像和缓存，帮助你释放宝贵的磁盘空间。

清除悬空的卷和网络
---------

有时，Docker 中可能会存在一些悬空的卷和网络，即它们不再与任何容器关联，但仍然占据磁盘空间或系统资源。为了清理这些未使用的卷和网络，可以使用 `docker volume prune`和 `docker network prune`命令。

```
$ docker volume prune
$ docker network prune


```

这些命令会删除所有未与任何容器关联的数据卷和网络，确保你的系统资源得到充分利用。

清除未打标签的镜像
---------

在使用 Docker 构建和使用镜像的过程中，可能会产生一些未打标签的镜像。这些镜像没有与任何标签关联，因此很难进行管理和识别。为了清理这些未打标签的镜像，可以使用 `docker image prune`命令。

```
$ docker image prune


```

这将删除所有未打标签且没有与任何容器关联的镜像，使你的镜像列表更加整洁。

清除停止的容器
-------

当我们停止容器后，这些容器将保留在系统中，占据磁盘空间。如果这些容器不再需要，可以通过 `docker container prune`命令清除停止的容器。

```
$ docker container prune


```

这将删除所有已停止的容器，释放磁盘空间和系统资源。

自动清理策略
------

除了手动执行清理命令外，还可以通过配置 Docker 自动执行清理操作。在 Docker 的配置文件 `daemon.json`中，可以设置一些参数来定义自动清理策略。

例如，可以设置镜像和容器的过期时间，使其自动清理。

```
{
  "image-prune-filters": {
    "until": "24h"
  },
  "container-prune-filters": {
    "until": "24h"
  }
}


```

上述配置将清理所有超过 24 小时未使用的镜像和容器。

此外，还可以设置日志的最大大小和日志文件的保留时间，以限制日志文件的增长。

```
{
  "log-opts": {
    "max-size": "100m",
    "max-file": "10"
  }
}


```

上述配置将限制每个日志文件的最大大小为 100MB，并保留最多 10 个日志文件。

通过自动清理策略，可以定期清理不再需要的镜像、容器和日志，保持系统的整洁和良好的性能。

Docker 清理是日常维护工作中不可或缺的一部分。通过清除未使用的镜像、容器、网络、数据卷和构建缓存，以及设置自动清理策略，可以释放磁盘空间，优化系统性能，并保持 Docker 环境的可靠性和稳定性。

在进行任何清理操作之前，请确保你理解清理命令的含义，并仔细考虑清除的资源是否真的不再需要。不正确地清理可能导致数据丢失或其他意外情况。
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.51cto.com](https://blog.51cto.com/windchasereric/4796157)

> dockerfile 最佳实践，dockerfile 最佳实践

使用官方镜像作为基础镜像
------------

*   背景

```
比如我们需要部署一个node应用，我们该如何选择何种镜像作为基础镜像？
第一种：选择通用的系统基础镜像，之后通过安装应用及工具来辅助构建应用
第二种：使用官方的基础镜像

1.
2.
3.

```

*   说明

```
两种方案均可以实现我们的目的，但第一种方案可能出现问题：
1. 从头开始进行基础应用镜像的构建，工作量会更加复杂
2. 编写对应的dockerfile需要更为严谨，避免引入不必要的文件或程序，导致构建后的镜像文件变大
3. 避免引入不必要的安全问题

1.
2.
3.
4.

```

*   建议

使用固定版本的镜像
---------

*   背景

```
在编写dockerfile时，引用的基础镜像时，不建议不指定固定标签或使用latest标签

1.

```

*   说明

```
若引用的基础镜像无指定版本，则默认会使用latest标签。
基础镜像里的应用可能会随着版本的迭代而发生变化，而一般会将稳定的，最新的版本都标识为latest标签 (简单理解就是 latest标签的基础镜像会随时发生变化)
这样会造成以下问题：
1. 不同时间构建的镜像，可能基础环境不一样，排查问题更难
2. 不同版本，可能会导致我们自己的应用出现兼容性问题

1.
2.
3.
4.
5.

```

*   建议

```
建议使用固定的镜像版本，例如：golang:1.17.3

1.

```

使用轻量级 alpine 的发行版
-----------------

*   背景

```
每个基础镜像可能存在不同的版本号，以及不同的发行版，改如何选择

1.

```

*   说明

```
基于发行版的系统，因为完整的系统里，会包含大量与应用无关的工具及环境，这样最终构建出来的镜像文件会比较大，而且因为引用了完整系统，由于系统自身可能存在漏洞问题，这样反而会增加镜像容器内应用程序的安全性
总结下，使用发行版的完整系统，而且其可能出现下列的问题：
1. 增加构建后的镜像文件大小
2. 降低镜像推送及拉取的速度，占用带宽
3. 可能会引入不必要的安全问题

而精简的发行版(alpine)：其仅会捆绑必要的系统工具和库，提供容器里应用程序启动的一切环境及组件，因此其更为轻量。
alpine的优点：
1. 构建出来的镜像文件相对会比较小
2. 减少引入不必要的安全问题


1.
2.
3.
4.
5.
6.
7.
8.
9.
10.
11.

```

*   建议

```
如果你的应用程序不需要特定的系统使用程序或库，推荐使用更为精简的，更小的docker镜像来做为基础镜像
例如： FROM node:17.0.1   --切换-->  FROM node:17.0.1-alpine

1.
2.

```

优化镜像层缓存
-------

*   背景

```
先理解 镜像层 及 镜像层缓存的说明：
镜像层： 在docker中，其依赖一系列的等技术，包括文件系统，写时复制，联合挂载的技术，导致在dockerfile中的每一行命令或指定，都会在基础镜像上创建一个层，而这个称之为镜像层
镜像层缓存： 当我们执行build时，每个镜像层都会被docker缓存下来，而当需要重建时，若dockerfile中的指令没有发生变更时，docker在构建时，会直接利用缓存层，这样可提高构建速度。

如何更好的使用缓存，来增加构建速度？

1.
2.
3.
4.
5.

```

*   说明

```
可通过命令 docker history IMAGE:TAG 查看生成的镜像(如下图示例) ，可看出除了<missing>其他的均是此镜像在基础镜像基础上新增的镜像层

1.

```

![](https://s8.51cto.com/images/blog/202112/13203029_61b73ce574e8822162.png)

```
尝试使用下面示例的 "Dockerfile"，执行docker build，用于模拟首次构建 v0.0.1版本
修改main.go的示例代码，并在此执行构建为 v0.0.2，留意下图左侧的缓存使用情况，发现每次修改 main.go 里的文件时，每次构建 COPY main.go main.go 以下的缓存均发生失效，尤其是需要重新下载依赖包

尝试将Dockerfile中的 9 和 10 行互换，在完成上面的演示，发现在 “RUN go mod download” 仍在使用缓存，而只有copy后续的工作没使用到缓存。 

可从示例中，若某一层的命令或内容发生了变更，意味着后续的所有的镜像层缓存都不可被复用(就是在构建时不再使用缓存)，后续的都镜像层都会被重新创建

1.
2.
3.
4.
5.
6.

```

```
package main
import "github.com/gin-gonic/gin"
func main() {
    r := gin.Default()
    r.GET("/ping", func(c *gin.Context) {
        c.JSON(200, gin.H{
            "message": "pong4",
        })
    })
    r.Run("0.0.0.0:8080") 
}

1.
2.
3.
4.
5.
6.
7.
8.
9.
10.
11.

```

```
FROM golang:1.14-alpine as build

ENV GO111MODULE=on \
    GOPROXY="https://goproxy.io"

WORKDIR /go/src/docker_go
ADD go.mod go.sum ./
COPY main.go main.go
RUN go mod download

RUN GOOS=linux CGO_ENABLED=0 GOARCH=amd64 go build -ldflags="-s -w" -installsuffix cgo -o docker_go .
EXPOSE 8080
ENTRYPOINT  ["./docker_go"]

1.
2.
3.
4.
5.
6.
7.
8.
9.
10.
11.
12.
13.
14.

```

![](https://s5.51cto.com/images/blog/202112/13203029_61b73ce557d4950835.png)

*   建议

```
应该将dockerfile涉及到的文件或内容，从最少变更步骤到最频繁变更步骤进行排序，并根据频繁程度由上往下进行编写(不频繁的放在最上面)， 通过这样的方式优化，来提高镜像的构建速度。

1.

```

降低镜像层数
------

*   说明

```
上面提及到的"镜像层"，可以了解，在dockerfile中的每条指定都会创建一个"镜像层"，而有些指定是同类型的操作，无此必要需要分开来写，导致生成的镜像文件存在多个"镜像层"

1.

```

*   建议

```
将同类型的操作放在一个指令，可通过 && 和 \ 来结合，这样既可减少生成镜像层，也更加美观和易于维护

1.

```

*   示例

```
ENV GO111MODULE=on
ENV GOPROXY="https://goproxy.io"


ENV GO111MODULE=on \
    GOPROXY="https://goproxy.io"


RUN yum makecache
RUN yum -y install nginx mysql


RUN yum makecache && \
    yum -y install nginx mysql

1.
2.
3.
4.
5.
6.
7.
8.
9.
10.
11.
12.
13.
14.
15.
16.

```

使用 .dockerignore 文件
-------------------

*   背景

```
在应用程序中，有一些文件或目录，认为在构建过程中并不会被使用到，或者一些敏感的文件，期望可以将其排除掉，而无需加载至镜像中，用于降低生成镜像的大小

1.

```

*   说明

```
在根目录下，可使用 .dockerfileignore 文件，用于排除掉无需加入进行的的目录或文件，其编写的方式和 .gitignore 相同，可过滤 目录，文件，或通过正则来过滤匹配到的文件

1.

```

*   建议

```
使用 .dockerfileignore 排除文件或目录

1.

```

*   示例

```
$ vim .dockerfileignore

.git
.cache

*.md

private.key

1.
2.
3.
4.
5.
6.
7.
8.

```

分阶段构建
-----

*   背景

```
在项目中构建应用程序过程中，会用到很多构建工具，库， 以及需要下载构建时所需的依赖包，而这些仅仅是在构建时才会使用到，而编译好后的应用，就不会在使用这些依赖及构建工具，而将这些资源保存至最终的镜像中，会导致镜像大小的增加

1.

```

*   说明

```
有些程序，在构建完成后，会生成二进制文件或静态文件，例如：
对于类似golang应用，其执行go build 后，将会生成一个可执行的二进制文件。
对于类似vue应用，通过执行下载依赖 npm install, 其会在目录node_modules下载相关的依赖包，最终执行打包命令 npm run build:prod，其打包后会在dist目录下生成相关的静态文件。
无论是golang还是vue应用，执行打包构建后，其在构建时所生成/下载的依赖文件都不会再次使用。 

因此我们清理掉这些不重要的组件，Dockerfile支持使用多阶段构建的方式, 使用不同的基础镜像，选择性的从一个阶段里的数据或文件，复制到新阶段的镜像文件中。其格式 COPY --from (详细参考示例说明)

1.
2.
3.
4.
5.
6.

```

*   建议

```
分阶段构建：在构建过程中，使用多个临时镜像来完成，但最终只保存最新的镜像作为最终的镜像文件
这样有助于将构建工具和依赖项，与运行时所需的内容进行分开

1.
2.

```

*   示例

```
FROM golang:1.14-alpine as build

WORKDIR /go/src/docker_go

ENV GO111MODULE=on \
    GOPROXY="https://goproxy.io"

COPY . .

RUN go mod init && \
    GOOS=linux CGO_ENABLED=0 GOARCH=amd64 go build -ldflags="-s -w" -installsuffix cgo -o docker_go .

EXPOSE 8081
ENTRYPOINT  ["./docker_go"]


FROM scratch as prod 
COPY --from=build /go/src/docker_go/docker_go /

CMD ["/docker_go"]

1.
2.
3.
4.
5.
6.
7.
8.
9.
10.
11.
12.
13.
14.
15.
16.
17.
18.
19.
20.
21.

```

容器中使用非 root 用户启动应用
------------------

*   背景

```
部署在容器里的应用程序，若无明确指定用户，默认将会使用root的身份来运行，但并不建议使用root身份来运行

1.

```

*   说明

```
这里会分两种情况，如果没有启动容器时无开启特权模式(--privileged)运行，则使用root身份运行并不会影响。
但若使用了特权模式运行，则容器里的root身份，有可能可以获取到宿主机的root权限，而这样就会导致安全问题。若容器里应用被黑客攻击，黑客很容易通过root的身份将权限提升至整个宿主机的权限，这样就非常危险

无论是否使用特权模式启动容器，都不建议root身份启动容器里的应用。除非一些特殊的操作，确实需要用到root，那需要评估好安全性问题。

1.
2.
3.
4.

```

*   建议

```
在容器里，尽量使用非root身份启动进程。
我们可在容器创建一个普通用户，或一些镜像会自带普通用户，用普通用户来启动容器

1.
2.

```

*   参考示例

```
FROM node:17.0.1-alpine
...
RUN chown -R node:node /app
USER node 
CMD ["node", "index.js"]

1.
2.
3.
4.
5.

```

参考文档： ​ [​https://www.zhihu.com/question/25580965​](https://www.zhihu.com/question/25580965)​

扫描镜像是否存在漏洞
----------

*   背景

*   说明

```
dockerhub 2020年年底推出了对镜像扫描的功能，其使用了 Snyk 服务来对镜像进行扫描，从而发现镜像是否存在漏洞。

由于使用 snyk 服务商，因此在首次执行扫描时，将会看到下列提示：
Docker Scan relies upon access to Snyk, a third party provider, do you consent to proceed using Snyk? (y/N)

使用方式： 直接通通过 docker scan 命令即可完成对镜像的扫描


由于docker scan 是比较新的功能，因此当使用 docker scan命令时，提示无此命令参数，你可考虑两种方案：
1. 升级docker 程序至最新版本
2. 使用 docker/scan-cli-plugin 插件完成扫描功能

1.
2.
3.
4.
5.
6.
7.
8.
9.
10.
11.
12.
13.

```

*   建议

```
可使用 docker scan 进行镜像的漏洞扫描，我们也可将其添加至cicd流水线进行自动扫描工作。


1.
2.

```

*   操作

```
1. 使用docker-scan 登录 dockerhub (不登录会提示： failed to get DockerScanID: bad status code "400 Bad Request")
$ docker-scan scan --login 
2. 执行镜像扫描
$  docker scan -f Dockerfile --exclude-base docker-scan:e2e

Testing docker-scan:e2e
...
✗ Medium severity vulnerability found in libidn2/libidn2-0
  Description: Improper Input Validation
  Info: https://snyk.io/vuln/SNYK-DEBIAN10-LIBIDN2-474100
  Introduced through: iputils/iputils-ping@3:20180629-2+deb10u1, wget@1.20.1-1.1, curl@7.64.0-4+deb10u1, git@1:2.20.1-2+deb10u3
  From: iputils/iputils-ping@3:20180629-2+deb10u1 > libidn2/libidn2-0@2.0.5-1+deb10u1
  From: wget@1.20.1-1.1 > libidn2/libidn2-0@2.0.5-1+deb10u1
  From: curl@7.64.0-4+deb10u1 > curl/libcurl4@7.64.0-4+deb10u1 > libidn2/libidn2-0@2.0.5-1+deb10u1
  and 3 more...
  Introduced in your Dockerfile by 'RUN apk add -U --no-cache wget tar'

Organization:      docker-desktop-test
Package manager:   deb
Target file:       Dockerfile
Project name:      docker-image|99138c65ebc7
Docker image:      99138c65ebc7
Base image:        golang:1.14.6
Licenses:          enabled

Tested 200 dependencies for known issues, found 16 issues.

1.
2.
3.
4.
5.
6.
7.
8.
9.
10.
11.
12.
13.
14.
15.
16.
17.
18.
19.
20.
21.
22.
23.
24.
25.
26.
27.
28.
29.

```

参考文档：

​ [​Vulnerability scanning for Docker local images | Docker Documentation​](https://docs.docker.com/engine/scan/)​

​ [​docker/scan-cli-plugin: Docker Scan is a Command Line Interface to run vulnerability detection on your Dockerfiles and Docker images (github.com)​](https://github.com/docker/scan-cli-plugin)​

举报文章

请选择举报类型

内容侵权 涉嫌营销 内容抄袭 违法信息 其他

上传截图

格式支持 JPEG/PNG/JPG，图片不超过 1.9M

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

已经收到您得举报信息，我们会尽快审核

相关文章
----
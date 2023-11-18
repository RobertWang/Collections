> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [post.smzdm.com](https://post.smzdm.com/p/apvdxk47/?send_by=3547620770&invite_code=zdm4ptnapkinv&zdm_ss=iOS_3547620770_&from=singlemessage)

> 一、什么是 DockerDocker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中, 然后发布到任何流行的 Linu

AlpineLinux 篇二：安装 Docker 以及 Docker Compose 环境
=============================================

2023-10-29 15:06:11 6 点赞 54 收藏 1 评论

### 一、什么是 Docker

Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中, 然后发布到任何流行的 Linux 或 Windows 操作系统的机器上, 也可以实现虚拟化, 容器是完全使用沙箱机制, 相互之间不会有任何接口。

### 二、Docker 的优势

1、更高效的利用系统资源；

2、更快速的启动时间；

3、一致的运行环境；

4、持续支付和部署；

5、更轻松的迁移；

6、更轻松的维护和拓展。

### 三、Docker 的安装

因上文我们已经在 Oracle VM VirtualBox 虚拟机内安装了 Alpine Linux 所以 Docker 的安装我们就用这个系统安装。（只添加清华源的可以直接用，如果添加的是第一个源的建议有条件的可以爬一下，不然下载安装有点慢）

**1、首先添加源**

输入命令

> vi /etc/apk/repositories

回车后我们会看到这样的一个画面。然后 i 进入编辑页面将清华源和官网的社区源添加进去，添加完成后 Esc 冒号 wq 保存文件。

[http://dl-cdn.alpinelinux.org/alpine/latest-stable/community](http://dl-cdn.alpinelinux.org/alpine/latest-stable/community "http://dl-cdn.alpinelinux.org/alpine/latest-stable/community")

[https://mirrors.tuna.tsinghua.edu.cn/alpine/latest-stable/community](https://mirrors.tuna.tsinghua.edu.cn/alpine/latest-stable/community "https://mirrors.tuna.tsinghua.edu.cn/alpine/latest-stable/community")

[![](https://qnam.smzdm.com/202310/29/653df8aae6a743616.png_e1080.jpg)](https://post.smzdm.com/p/apvdxk47/pic_2/) 添加源

**2、更新最新镜像源列表 输入**

> apk update

[![](https://am.zdmimg.com/202310/29/653df984e2f2d91.png_e1080.jpg)](https://post.smzdm.com/p/apvdxk47/pic_3/) 更新最新镜像源列表

**3、安装 Docker**

输入命令

[![](https://qnam.smzdm.com/202310/29/653dfa3cdb9245134.png_e1080.jpg)](https://post.smzdm.com/p/apvdxk47/pic_4/) 安装 docker

启动 Dokcer 输入命令

> service docker start

开机自启动 docker 输入命令

> rc-update add docker boot

[![](https://qnam.smzdm.com/202310/29/653dfaff77ac57699.png_e1080.jpg)](https://post.smzdm.com/p/apvdxk47/pic_5/) 启动 Dokcer 和开机自启动 docker

查看 docker 版本输入命令

> docker -v

[![](https://qnam.smzdm.com/202310/29/653dfb31e025a9119.png_e1080.jpg)](https://post.smzdm.com/p/apvdxk47/pic_6/) 查看 docker 版本

如果出现上图表示我们的 docker 已经安装完成了。

### 四、**Docker Compose 介绍**

Docker Compose 是一个用来定义和运行复杂应用的 Docker 工具

一个使用 Docker 容器的应用，通常由多个容器组成。使用 Docker Compose 不再需要使用 shell 脚本来启动容器

Compose 通过一个配置文件来管理多个 Docker 容器，在配置文件中，所有的容器通过 services 来定义，然后使用 docker-compose 脚本来启动，停止和重启应用，和应用中的服务以及所有依赖服务的容器，非常适合组合使用多个容器进行开发的场景

**为什么需要 docker-compose**

①提供工具用于定义和运行多个 docker 容器应用；

　　②使用 yaml 文件来配置应用服务 (docker-compse.yml)；

　　③可以通过一个简单的命令 docker-compse up 可以按照依赖关系启动所有服务；

　　④可以通过一个简单的命令 docker-compose down 停止所有服务；

　　⑤当一个服务需要的时候，可以很简单地扩容；

### 五、**Docker Compose 的环境部署**

**1、安装 py-pip 工具**

> apk add py-pip

[![](https://qnam.smzdm.com/202310/29/653dfc7ad04a87740.png_e1080.jpg)](https://post.smzdm.com/p/apvdxk47/pic_7/) 安装 py-pip 工具

**2、安装特定版本的 PyYAML，我试过了默认安装的是 5.4.1 版本，貌似有问题安装不了，但是 5.3.1 版本可以。注意：**PyYAML 名称的大小写。

> pip install PyYAML==5.3.1

[![](https://qnam.smzdm.com/202310/29/653dfcf467a2b7498.png_e1080.jpg)](https://post.smzdm.com/p/apvdxk47/pic_8/) 安装特定版本的 PyYAML

**3、安装 docker-compose**

**输入命令**

> pip install docker-compose

[![](https://am.zdmimg.com/202310/29/653dfd91ad3d37084.png_e1080.jpg)](https://post.smzdm.com/p/apvdxk47/pic_9/)[![](https://am.zdmimg.com/202310/29/653dfd961b5f78299.png_e1080.jpg)](https://post.smzdm.com/p/apvdxk47/pic_10/)

安装完成后查看 docker-compose 版本

> docker-compose --version

[![](https://am.zdmimg.com/202310/29/653dfdfe5a1207028.png_e1080.jpg)](https://post.smzdm.com/p/apvdxk47/pic_11/)

如果出现了上图的提示那表示你的 Docker-compose 已经安装完成。至此我们在 Alpine Linux 系统上已经部署完成了 Docker 以及 Docker Compose 环境。
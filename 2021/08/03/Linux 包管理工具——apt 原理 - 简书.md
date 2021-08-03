> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/c6936efe5b58)

apt 是 Debian 系列的 Linux 操作系统的包管理工具，最近的项目中使用了 stretch Linux，它也是使用 apt 来进行包管理的。

#### apt 工作原理

apt 采用集中式的软件仓库机制，将各式各样的软件分门别类的放在软件仓库之中，从而进行有效的组织和管理。然后，将软件仓库放置在许多镜像服务器中，并保持基本一致。这样一来，所有的用户都能获取最新的软件安装包。对用户而言，这写镜像就是软件源。  
由于用户所处网络的不同，无法随意的访问各个镜像站点。为了能让用户有选择的访问镜像站点，使用了软件源配置文件 / etc/apt/sources.list 列出最合适访问的镜像站点的地址。

![](http://upload-images.jianshu.io/upload_images/2779961-9e7e2292bf7a4a1f.png) /etc/apt 目录内容

![](http://upload-images.jianshu.io/upload_images/2779961-54ccfd908b4153f8.png) /etc/apt/sources.list.d

上图中是我用 docker 镜像运行的容器中的 sources.list.d 文件的内容，由于我在 Dockerfile 中添加了 key，所以 moosefs.list 会存在于 sources.list.d 中。

#### apt-get update

在执行了 apt-get update 命令后，apt 会自动联网寻找 source.list.d 文件中的 list 对应的 Package/Sources/Release 列表文件，如果存在则下载，存放在 / var/lib/apt/lists 目录中。  
然后看一下容器中的 / var/lib/apt/lists 目录。然后 apt-get install 相应的包。

![](http://upload-images.jianshu.io/upload_images/2779961-3ba5a5befdd0ea56.png) image.png

软件源只是告诉了 Linux 系统可以访问的镜像站点地址，但是镜像站点上具体有什么软件并不清楚，如果每安装一个软件包就在镜像站点上搜索一遍效率是很低的。因此，就有必要为这些软件资源列一个清单，这个清单就是索引文件，用来让系统查找包的。

#### apt-get install

apt-get install 是下载命令，下载的软件都会存到 / var/cache/apt/archives 下。  
apt 还会检查 Linux 系统的包依赖关系，简化了用户安装和卸载包的过程。  
要下载一个软件包时，大概需要 4 步：  
1. 扫描本地存放的软件包更新列表，找到最新版本的软件包。  
2. 进行软件包依赖关系检查，找到支持该软件的所有软件包。  
3. 从镜像站点中下载相关软件包 (包含所依赖的软件包)，并存放在 / var/cache/apt/archive  
4. 解压软件包，并自动完成应用程序的安装和配置。

#### apt-get update

正如前面说的，要想使用 apt-get 下载安装软件，需要去 / etc/apt/source/list 中的镜像源地址中去下载，那么我们仅仅是知道去哪里下载，镜像源地址中有什么软件，我们并不清楚，所以需要使用 apt-get update 来刷新软件的索引，从而确定我们要的软件在镜像站点中是否存在。  
apt-get update 会扫描每个镜像站点，并为该站点所具有的软件包资源建立索引文件，存放在本地的 / var/lib/apt/list 中。在使用 apt-get 命令执行安装或者更新操作时都会依赖这些索引文件，所以在每次更新或者安装前应该使用 apt-get update 命令来刷新索引，从而获取最新的软件资源。

#### apt-get upgrade

将系统中所有的软件包一次性升级到最新版本。

提示：如果你和我一样，创建了一个 docker 镜像文件，建议不要使用 apt-get upgrade 命令，因为镜像其实就是系统的一个 “快照”，这个镜像刚好满足了我们程序的需求，如果使用了 apt-get upgrade 命令后将会使得镜像变得很大，而且每次构建镜像时也会耗费更多时间。一个优秀的镜像的原则是，在满足程序需求的同时，体积越小越好。

另外，在下载完成后可以删除 / var/lib/apt/lists / 中索引文件，从而减小镜像的体积。
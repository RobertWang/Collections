> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [liumiaocn.blog.csdn.net](https://liumiaocn.blog.csdn.net/article/details/89702529)

> 在一个使用旧版的 Oracle 的 JDK 的 Alpine 版本的镜像时出现了问题，这篇文章作为后续的整理，以此为契机，简单介绍一下 Alpine 版本中的 musl libc 和 gnu libc 的设定。

在一个使用旧版的 Oracle 的 JDK 的 Alpine 版本的镜像时出现了问题，这篇文章作为后续的整理，以此为契机，简单介绍一下 Alpine 版本中的 musl libc 和 gnu libc 的设定。

*   运行 alpine 容器  
    准备一个 Alpine 镜像，这里使用 3.9 的版本，并将 Alpine 容器运行起来，容器名为 alpine

```
liumiaocn:multibytes liumiao$ docker run --name alpine --rm -it alpine:3.9 sh
/ 

```

*   准备可执行文件  
    这里准备旧版的 JDK，取 7u79，并将其拷贝至 alpine 容器的 / tmp 目录下

```
liumiaocn:Desktop liumiao$ docker cp jdk-7u79-linux-x64.tar.gz alpine:/tmp
liumiaocn:Desktop liumiao$

```

*   解压 JDK 至 / usr/local/share/java

```
/ 
/tmp 
jdk-7u79-linux-x64.tar.gz
/tmp 
/tmp 
tar xzf jdk-7u79-linux-x64.tar.gz -C /usr/local/share/java
/tmp 
/usr/local/share/java 
jdk1.7.0_79
/usr/local/share/java 

```

JDK 只需要设定可执行目录至 PATH 搜索范围中即可，设定之后发现使用 which 可以找到 java 可执行文件，但是执行 java -version 却提示 java not found, 所以现象的提示显得非常诡异。

```
~ 
~ 
/usr/local/share/java/jdk1.7.0_79/bin/java
~ 
~ 
sh: java: not found
~ 

```

*   ldd 错误  
    动态连接库错误，ldd 提示信息如下所示

```
~ 
/usr/local/share/java/jdk1.7.0_79/bin/java
~ 
	/lib64/ld-linux-x86-64.so.2 (0x7f5c1184c000)
	libpthread.so.0 => /lib64/ld-linux-x86-64.so.2 (0x7f5c1184c000)
	libjli.so => /usr/local/share/java/jdk1.7.0_79/bin/../lib/amd64/jli/libjli.so (0x7f5c11635000)
	libdl.so.2 => /lib64/ld-linux-x86-64.so.2 (0x7f5c1184c000)
	libc.so.6 => /lib64/ld-linux-x86-64.so.2 (0x7f5c1184c000)
Error relocating /usr/local/share/java/jdk1.7.0_79/bin/../lib/amd64/jli/libjli.so: __rawmemchr: symbol not found
~ 

```

ldd 错误看起来好像是 jli 的的 relocating 提示错误信息，而实际上由于 Alpine 镜像使用的根本不是 gnu libc 而是 musl libc，所以 / lib64/ld-linux-x86-64.so.2 是不存在的，而实际上 / lib64 都是不存在的。

```
~ 
ls: /lib64: No such file or directory
~ 

```

gnu libc 和 musl libc 号称兼容（部分兼容），基于缺什么补什么的原则，做个软链补上即可（stack overflow 也有人有同样方法进行过验证）。由于 Oracle 的 Java 并不是完整的源码提供，所以 Alpine 也无法拿到源码去全编译来解决这个问题，大概这就是在 Alpine 镜像中更多的是 OpenJDK 的原因。

```
~ 
~ 
~ 

```

这个问题一旦得到对应，现象就不再诡异，再次执行 java 命令，中规中矩地提示了缺少的动态链接库

```
~ 
Error relocating /usr/local/share/java/jdk1.7.0_79/bin/../lib/amd64/jli/libjli.so: __rawmemchr: symbol not found
~ 

```

顺这个问题在 Alpine 的 issue 中很快找到了一个几年前的 issue，写的很详细，jirutka 小哥还提到了 Alpine 的 3S（Small/Simple/Secure）哲学，并坚持认为不应该在 Alpine 里面添加对于 gnu libc 的支持, 以下为轻松时间，可以顺便学点，学不到也完全不要紧，我们主要是坐第一排看热闹。

他们的现象跟本文的现象完全一样不同的是 Oracle 的 JDK8，不同的是非常快得就有人帮着定位出了根本原因，而我查了几个小时  
![](https://img-blog.csdnimg.cn/2019043006263560.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9saXVtaWFvY24uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)  
jirutka 小哥说，如果使用 Oracle 版本的 JDK，或者类似需求，你应该去找支持 glibc 的 linux 发行版  
![](https://img-blog.csdnimg.cn/20190430063301787.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9saXVtaWFvY24uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)  
再次甩给 Oracle，该他们管，Alpine 的已经有 MUSL 了，谁用 Glibc 谁自己负责，另外代码不全部公开 Alpine 也无能为力。而且还认为，使用了 glibc 的 Alpine 就不再是 Alpine！是的，会变成奥特曼的  
![](https://img-blog.csdnimg.cn/20190430063510562.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9saXVtaWFvY24uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)  
有人提出两个 libc 能不能一起来用，小哥坚持认为两个 libc 不是好的方法  
![](https://img-blog.csdnimg.cn/20190430063655998.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9saXVtaWFvY24uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)  
专门有人去跟 Oracle 确认了一下，结论是没有好的办法，不过可以花钱来寻求专业帮助  
![](https://img-blog.csdnimg.cn/20190430065447828.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9saXVtaWFvY24uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)  
另外一位小哥总结了一下：要么用指定的所谓的标准版本，像使用 Alpine 这种非标准方式就只好花钱雇人了  
![](https://img-blog.csdnimg.cn/20190430065602293.png)  
而这些终于在这个 issue 中给得到了解决，由于没有热闹可看，请读者自行阅读

*   [https://github.com/ibmdb/node-ibm_db/issues/217](https://github.com/ibmdb/node-ibm_db/issues/217)

使用的相关内容在这里：

*   [https://github.com/sgerrand/alpine-pkg-glibc](https://github.com/sgerrand/alpine-pkg-glibc)

简单来说，解决的方法就是在 Alpine 里面安装 glibc，让 Alpine 不再是 Alpine

*   详细看这个 issue  
    觉得还想看的话，自己去看这个 issue  
    [https://github.com/gliderlabs/docker-alpine/issues/11](https://github.com/gliderlabs/docker-alpine/issues/11)

看完热闹，现在花 1 分钟快速解决一下遗留问题。重新回到问题现场。按照如下三步骤进行安装

*   步骤 1: 下载 key

```
~ 
~ 
0
~ 
/etc/apk/keys/sgerrand.rsa.pub
~ 

```

*   步骤 2: 下载 apk 安装文件

```
~ 
Connecting to github.com (13.229.188.59:443)
Connecting to github-production-release-asset-2e65be.s3.amazonaws.com (52.216.176.203:443)
glibc-2.29-r0.apk    100% |****************************************************************************************| 2006k  0:00:00 ETA
~ 
glibc-2.29-r0.apk
~ 

```

*   步骤 3: 安装

```
~ 
(1/1) Installing glibc (2.29-r0)
OK: 9 MiB in 15 packages
~ 

```

虽然 ldd 不能正常使用，但是实际上已经能够执行了，这是因为设定了 RPATH。因为没有 readelf 命令，这里就不再展示了。

```
~ 
	/lib64/ld-linux-x86-64.so.2 (0x7f83ae582000)
	libpthread.so.0 => /lib64/ld-linux-x86-64.so.2 (0x7f83ae582000)
	libjli.so => /usr/local/share/java/jdk1.7.0_79/bin/../lib/amd64/jli/libjli.so (0x7f83ae36b000)
	libdl.so.2 => /lib64/ld-linux-x86-64.so.2 (0x7f83ae582000)
	libc.so.6 => /lib64/ld-linux-x86-64.so.2 (0x7f83ae582000)
Error relocating /usr/local/share/java/jdk1.7.0_79/bin/../lib/amd64/jli/libjli.so: __rawmemchr: symbol not found
~ 

```

可以看到已经可以正常使用了, ldd 对使用者来说，基本就是透明的。先凑合用吧。

```
~ 
java version "1.7.0_79"
Java(TM) SE Runtime Environment (build 1.7.0_79-b15)
Java HotSpot(TM) 64-Bit Server VM (build 24.79-b02, mixed mode)
~ 

```

其实小哥说的对，这种拧着做的方法确实不值得推荐，但毕竟生活有很多无奈，且行且拧吧。

[https://stackoverflow.com/questions/34729748/installed-go-binary-not-found-in-path-on-alpine-linux-docker](https://stackoverflow.com/questions/34729748/installed-go-binary-not-found-in-path-on-alpine-linux-docker)  
[https://github.com/docker-library/openjdk/issues/77](https://github.com/docker-library/openjdk/issues/77)  
[https://github.com/gliderlabs/docker-alpine/issues/11](https://github.com/gliderlabs/docker-alpine/issues/11)  
[https://github.com/sgerrand/alpine-pkg-glibc/releases](https://github.com/sgerrand/alpine-pkg-glibc/releases)
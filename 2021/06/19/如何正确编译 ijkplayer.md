> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIyNTY4NjU0OQ==&mid=2247507075&idx=5&sn=7fe2a2908166bc2c692dca65bd709b30&chksm=e87979f9df0ef0ef6df8ddc8bd459e8c4fb37184962d70c83935b983e3fe38bef51a2b8e0343&mpshare=1&scene=1&srcid=0619Wh2JPczmu4kX0poVUwNg&sharer_sharetime=1624113308987&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> PS：无论做什么事，实践最重要，亲身实践积累的经验，不是随便阅读一篇技术文章所能比拟的。

最近项目中可能需要视频播放，而且后期还可能要支持播放 rtsp 协议的视频，权衡了一下还是通过编译 B 站开源的 ijkplayer 吧，ijkplayer 是一个基于 ffmpeg 的轻量级的可在 Android 和 Ios 上使用的跨平台播放器，可以通过编译来实现更多格式的支持，可以说只要是 ffmpeg 支持的格式 ijkplayer 就支持。

刚开始使用 Cygwin 进行编译，但总是在生成 so 文件的时候出错，当然中间还有很多要踩的坑，于是决定使用 Ubuntu 环境编译 ijkplayer，在 Ubuntu 环境下编译时基本没有什么问题，编译过程如下：

1.  准备
    
2.  配置环境变量
    
3.  安装必须组件
    
4.  正式编译
    
5.  运行 ijkplayer
    

#### 准备

安装 VMware 虚拟机并安装 Ubuntu 系统，安装完 VMware 之后创建虚拟机，选择典型安装模式，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/5kgRlkZhZcUQX0vyWibJcE1drAQ6jJVEtmEKUUaQoVBYFiccba449xpqF8jCFxTaWg2hYTJqm2gcYpAYicJ8QmicBg/640?wx_fmt=png)

第一步

然后点击下一步，选择已经下载的系统镜像，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/5kgRlkZhZcUQX0vyWibJcE1drAQ6jJVEtnYNQt8rfTDmodgOHejm2IsiaDIoepXyneavKHh7VpQDEaxicicqSzMd7A/640?wx_fmt=png)

第二步

正确选择后会显示出镜像信息，如我选用的是 Ubuntu 64 位 18.04 ，然后继续下一步，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/5kgRlkZhZcUQX0vyWibJcE1drAQ6jJVEtpp0weFM0JUnrrSgfXZJ5rbnTmq3FSboGyibVKyjedLvxlPZnE2uGKww/640?wx_fmt=png)

第三步

填写用户名、密码等信息，点击下一步，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/5kgRlkZhZcUQX0vyWibJcE1drAQ6jJVEtsugYgVm2tiaZLNpfkibXQbWlyafCoGQukaLJrloIA8kUgdVQfaCvMlmw/640?wx_fmt=png)

第四步

填写虚拟机名称以及虚拟机要安装的位置，点击下一步：

![](https://mmbiz.qpic.cn/mmbiz_png/5kgRlkZhZcUQX0vyWibJcE1drAQ6jJVEtDO55NqicEjibhpflQnjzXva2ZFtYNCUN68b579q57Tt5g9vyK3S65yPw/640?wx_fmt=png)

第五步

设置虚拟机磁盘大小，为了不降低磁盘性能选择将磁盘存储为单个文件，然后点击下一步，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/5kgRlkZhZcUQX0vyWibJcE1drAQ6jJVEtEqPqhnibSkZic8fxh9g80plicYwjVNOEhtETf2ibByZJMk0oskoJNRuCoQ/640?wx_fmt=png)

第六步

Ubuntu 虚拟机到此创建完毕，点击完成，等待 Ubuntu 安装完成，输入设置的密码即可进入 Ubuntu 系统，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/5kgRlkZhZcUQX0vyWibJcE1drAQ6jJVEtNr6DvXLPcRYYMNib85dsOhg5xeZlckEs1NgejRfa15Nj6YzjGFdfCzw/640?wx_fmt=png)

安装完成

此外还需下载好 Linux 版本的的 Android SDK 和 NDK，这里选择的分别是 android-sdk_r24.4.1-linux.tgz 和 android-ndk-r10e-linux-x86_64.zip，下载后可以使用如下命令解压文件：

切记不要将 NDK 目录放在虚拟机的共享目录下，为保证编译顺利进行应将 NDK 目录放在 Ubuntu 的系统目录，也就是 /home / 用户名 下面的目录。

#### 配置环境变量

在 Ubuntu 下的 /home / 用户名 / ，按 Ctrl+h 查看 .bashrc 文件并配置 SDK 和 NDK 环境变量，参考如下：

```
1NDK=/home/jzman/android/android-ndk-r10e
 2export NDK
 3ADB=/home/jzman/android/android-sdk-linux/platform-tools
 4export ADB
 5# ANDROID_NDK和ANDROID_SDK路径
 6ANDROID_NDK=/home/jzman/android/android-ndk-r10e
 7export ANDROID_NDK
 8ANDROID_SDK=/home/jzman/android/android-sdk-linux
 9export ANDROID_SDK 
10# 加入到PATH路径
11PATH=${PATH}:${NDK}:${ADB}:${ANDROID_NDK}:${ANDROID_SDK}
```

配置完成后保存并关闭 .bashrc，打开 Terminal 输入 ndk-build -v 查看 ndk 是否配置成功，运行日志如下则配置成功：

```
1jzman@ubuntu:~$ ndk-build -v
2GNU Make 3.81
3Copyright (C) 2006  Free Software Foundation, Inc.
4This is free software; see the source for copying conditions.
5There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A
6PARTICULAR PURPOSE.
8This program built for x86_64-pc-linux-gnu
```

#### 安装必须组件

依次输入如下命令更新和安装 git、yasm 和 make ，

```
1sudo apt-get update
2sudo apt install git
3sudo apt install yasm
4sudo apt install make
```

使用 git --version 和 make -v 查看 git 和 make 工具是否安装成功，成功则显示对应版本号，参考如下：

```
1jzman@ubuntu:~$ git --version
 2git version 2.17.1
 3jzman@ubuntu:~$ make -v
 4GNU Make 4.1
 5Built for x86_64-pc-linux-gnu
 6Copyright (C) 1988-2014 Free Software Foundation, Inc.
 7License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
 8This is free software: you are free to change and redistribute it.
 9There is NO WARRANTY, to the extent permitted by law.
10jzman@ubuntu:~$ 
```

#### 正式编译

```
1//clone ijkplayer源码
 2git clone https://github.com/Bilibili/ijkplayer.git ijkplayer
 3cd ijkplayer
 4git checkout -B latest k0.8.8
 5//使用更轻量的module-lite.sh
 6cd ijkplayer/config
 7rm module.sh
 8ln -s module-lite module.sh
 9//下载ffmpeg源码
10cd ijkplayer
11./init-android
12//编译ffmpeg
13./compile-ffmpeg.sh clean
14./compile-ffmpeg.sh all
15//编译ijkplayer,生成so文件
16cd ijkplayer/android
17./compile-ijk.sh all
```

如果要支持 https，在编译时执行如下命令：

```
1cd ijkplayer
2./init-android-openssl.sh（支持https）
3cd ijkplayer/android/contrib
4./compile-openssl.sh clean
5./compile-openssl.sh all
```

编译成功之后会在 ijkplayer/android 下面生成对应的 Android 工程，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/5kgRlkZhZcUQX0vyWibJcE1drAQ6jJVEtaib9caRSXc8IvFwjevXkM8HibCLKfjoJnFlmmFlGkiarvuGrGujEibXUew/640?wx_fmt=png)

编译完成

查看各个 abi 库中，如 ijkplayer/android/ijkplayer/ijkplayer-arm64/src/main/libs 下面是否生成对应的 so 文件，以 arm64 为例，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/5kgRlkZhZcUQX0vyWibJcE1drAQ6jJVEtPHs2zdDwKasVHVib7cOL4Wa2ZUFSJz3WVTDMZcAMolS6Ookv6jR9nmA/640?wx_fmt=png)

生成 so 文件成功

#### 运行 ijkplayer

使用 Android Studio 打开编译生成的 Android 工程，运行截图如下：

![](https://mmbiz.qpic.cn/mmbiz_png/5kgRlkZhZcUQX0vyWibJcE1drAQ6jJVEtaXvhPkQTzRKN0nDDDkxDK3ZwZIuJ0Vah9US8gicQnXcF9j3Hh3PVXcg/640?wx_fmt=png)
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/water-wjf/p/8342896.html) 本文算是自己的一个笔记吧。

 

介绍：

　　Unix 于 1969 年诞生于贝尔实验室的计算机科学家 Ken Thompson 的头脑中，Thompson 和 Ritchie 为支持游戏开发而在 PDP-7 上编制的实用程序成了 Unix 的核心——虽然直到 1970 年才产生 Unix 这个名字，1978 年，第一个 Unix 公司（the Santa Cruz Operation，SCO）成立，同年售出第一个商用 C 编译器（Whitesmiths）

 

 

AF_INET 域与 AF_UNIX 域 socket 通信原理对比 [http://blog.csdn.net/sandware/article/details/40923491](http://blog.csdn.net/sandware/article/details/40923491)：

　　1.  AF_INET 域 socket 通信过程

　　2.  AF_UNIX 域 socket 通信过程

 

Linux 系统与 Mac 系统启动区别：

　　Mac OS X 的启动方式不像其他 Unix 系统。MacOSX 没有 /etc/init.d 目录。他寻找启动项通过 launchd 程序。你可以在 in this ADC article 了解更多的内容。

 

　　OSX 内核叫 XNU，是 “X is Not Unix” 的缩写。OSX 是一种类 unix，和 FreeBSD 也是不一样的，是 FreeBSD 的内核捏合了另外两种特性，已经是新的内核了。支持 GNU 标准，所以 GNU\Linux 上 80% 的代码可以直接在 OSX 上编译运行。XNU 是开源的。

 

OSX 系统组成：[http://www.linuxidc.com/Linux/2014-12/110296.htm](http://www.linuxidc.com/Linux/2014-12/110296.htm)

 

[Linux IPC 总结（全）](http://www.cnblogs.com/wangkangluo1/archive/2012/05/14/2498786.html)

[Linux 进程间通信的几种方式总结 --linux 内核剖析（七）](http://blog.csdn.net/gatieme/article/details/50908749)

 

 

 

192.168.3.190 root/alpine

 

strace 在 linux 下用来跟踪某个进程的系统调用

在 solaris 下，对应的是 dtrace

在 mac 下，对应的命令是：dtruss

 

pstack 命令可显示每个进程的栈跟踪。pstack 命令必须由相应进程的属主或 root 运行。可以使用 pstack 来确定进程挂起的位置。此命令允许使用的唯一选项是要检查的进程的 PID。

命令软件包下载地址：[https://packages.debian.org/sid/pstack](https://packages.debian.org/sid/pstack)

 

pstree
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.sina.com.cn](http://blog.sina.com.cn/s/blog_62b2318d0101e40x.html)

osx 下与 strace 功能相近的程序是 dtruss（好像 AIX 下类似功能的命令是 truss，记不清了~~），可以打印程序的系统调用，使用方法和功能与 strace 基本相同。

与 ldd 相同的是 otool -L 命令，使用方法为：

sh-3.2# otool -L /usr/lib/libc++.dylib  /usr/lib/libc++.dylib: /usr/lib/libc++.1.dylib (compatibility version 1.0.0, current version 65.1.0) /usr/lib/libc++abi.dylib (compatibility version 1.0.0, current version 24.2.0) /usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 169.3.0) sh-3.2#
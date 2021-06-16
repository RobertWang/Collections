> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/han-sun/p/12627850.html)

> 在启动 springboot 项目的时候，会停顿好长时间才开始打印日志。
> 
> 对于强迫症这是受不了的。

　　查看第一行的日志打印。

```
InetAddress.getLocalHost().getHostName() took 5004 milliseconds to respond. Please verify your network configuration (macOS machines may need to add entries to /etc/hosts).
```

> 这行日志可以看出它去解析 hostname 的时间就花了 5 秒多。
> 
> 这里说明了它去解析了 hosts 文件。   macOS machines may need to add entries to /etc/hosts

> 1、查看本机的 hostname 是什么

```
~ hostname
han.sun-mac-pro.local
```

> 2、查看 hosts 文件里面配置的是什么
> 
> ```
> broadcasthost
> ```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
~ vim  /private/etc/hosts


127.0.0.1                localhost
255.255.255.255     　　　broadcasthost
::1                      localhost
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

> 3、复制 hostname

```
127.0.0.1              localhost      你的机器名.local
255.255.255.255   　　　broadcasthost
::1                    localhost      你的机器名.local
```

> 4、:wq 或者: x 可能修改不了文件

```
:w !sudo tee %
```

>  5、再去启动 springboot 项目就变得飞快了
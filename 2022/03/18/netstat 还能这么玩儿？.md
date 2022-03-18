> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651170586&idx=1&sn=0d66f4f930796182ce99d3367a093ed1&chksm=80647645b713ff530b1f26836091446ff76e915740f5b9cd7aeb3fa08b947a923a6832bf1594&mpshare=1&scene=1&srcid=0314zbJYf5X3GV7CwvOYbTRD&sharer_sharetime=1647242654193&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

netstat 这个命令在 Linux 、Windows 和 MacOS 操作系统下都兼容，不同的是，netstat 在 UNIX 下显示详细信息的命令是 _man netstat_ ，而在 Linux 和 Windows 下面是 _netstat --help_。

Linux 下的 netstat 命令
-------------------

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaQTk2mOuiaKs337B82O5Axw1Tb6w0NmZ9qYCH2YpshnPGzY0xtmedC4aak2XprdElHWjEJ6TyPVtlg/640?wx_fmt=jpeg)

```
netstat -- show network status
```

**列出网络状态**

但是这网络状态都有啥呢？带着疑问，我在 Linux 下执行了一下。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaQTk2mOuiaKs337B82O5Axw1lIF2nsev8IcGp4vVCib7ibHFR5ulTw5lRGz4dJjHsLLVV3972ZoV3tkA/640?wx_fmt=jpeg)

打印出来是一个六元组，六元组每一列的内容分别是

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaQTk2mOuiaKs337B82O5Axw17WANsUXSo7weckXgbDdrwGVibthjD0wG24OkUVIM7OXhh0gSRvQ5Tmg/640?wx_fmt=jpeg)

**仔细看了一下这个六元组，这好像表明 netstat 这个命令是用于监控传入和传出的网络连接和状态的一个命令行工具啊**。

从整体上来看，netstst 的输出结果可以分为两部分，一部分是 _Active Internet connections_，称为**活跃 TCP 连接**，其中的 Recv-Q 和 Send-Q 指的是客户端发送队列和客户端接收队列。这两个队列的值一般都是 0 ，如果不是 0 的话表示有消息堆积还没有发出去 / 取出，这种情况一般很少见到。

另外一部分是 _Active UNIX domain sockets_, 称为 **活跃的 Unix 域套接字**，这部分中的 socket 和网络 socket 套接字一样，不同的是，这块只能用来本地通信，性能要比网络 socket 高。Active UNIX domain sockets 也是一个六元组，分别表示

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaQTk2mOuiaKs337B82O5Axw1kz7SUMOYB9N2rxGojzgYF94pEkQCKlvtb7dQ8X4fuuks5RmKWRzoIg/640?wx_fmt=jpeg)

netstat 参数释义
------------

下面我们来解释一下 netstat --help 列出来的一些参数，我们从最常见的一些参数开始入手，这样大家看起来也能形成阶段性记忆，不至于失去重点。

### netstat -a

_-a_ 这个参数默认会监控所有的 socket 连接。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaQTk2mOuiaKs337B82O5Axw1YMwRSicje6oIpqkLDlbK31DZ5YJUe9GnaLjOyYmzwXNplVCB0ibhnPMA/640?wx_fmt=jpeg)

包括已经监听的、已经建立连接的、客户端发送的等待服务器的和未被监听都会被列出来。

**netstat -at/-t**

_netstat -at_ 和 _netstat -t_ 这俩后缀都是用来监听与 TCP 协议有关的端口，不同的是 netstat -at 会监听所有 State（状态）下的端口，而 netstat -t 仅仅会监听 _ESTABLISHED_ 状态的端口。

netstat -at

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaQTk2mOuiaKs337B82O5Axw10VgjnibKq4OCAZ97u4Qb3bFNomrmQ8eiaTzJTG1OY2O5EXHyH0hv8F6w/640?wx_fmt=jpeg)

netstat -t

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaQTk2mOuiaKs337B82O5Axw1SCDU6MJ1ic7LJxs7aK6RGhAQrmoUuHsUXlFIgOGCY0GhWb1cTKuIIFw/640?wx_fmt=jpeg)

**netstat -au/-u**

同样的，_netstat -au_ 和 _netstat -u_ 都会监控与 UDP 有关的端口，不同的是 netstat -au 会监听所有 State（状态）下的端口，而 netstat -u 仅仅会监听 _ESTABLISHED_ 状态的端口。

netstat -au

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaQTk2mOuiaKs337B82O5Axw1eiaP3fFiaMU3vTVibnaTBrNhkyaODU3806Vb5JahFasWhGV2Hh5HMJ3bA/640?wx_fmt=jpeg)

netstat -u

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaQTk2mOuiaKs337B82O5Axw1ZntrDKu4BBicnn4DyQibaqSJe4O9icVYv7Tfh22KhFwSnQNibrRia7NQtrg/640?wx_fmt=jpeg)

我这里测试是没有监控已经建立连接状态下的 UDP 协议。

**netstat -ap**

这条命令用于列出程序运行的端口，常用的命令是

```
netstat -ap|grep '程序名'
```

比如我们要找 http 程序，就是 _Netstat -ap|grep http_

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaQTk2mOuiaKs337B82O5Axw1q8ia7YldJZyeKZPsNJTTofpYJUQFHPaapHed087H3Toj5cfzicPuy1og/640?wx_fmt=jpeg)

还可以直接列出端口号

```
netstat -ap|grep 8080
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaQTk2mOuiaKs337B82O5Axw1SnImThH0X23btyoh6V8GnlzRJ5mkWQL3GOsxjOIdRL0Q7E6CcIajicA/640?wx_fmt=jpeg)

> 不过需要注意下，并不是所有的程序都能被找到，没有权限的不会显示，使用 root 权限可以查询所有信息。

### netstat -l

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaQTk2mOuiaKs337B82O5Axw1yBjlc2r6J1KwYtVB3m7BbBTwqPIXjQjM0prI2icXkL22KibcibFPQ1LPw/640?wx_fmt=jpeg)![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaQTk2mOuiaKs337B82O5Axw1M2GVASh8p1VP4uTmcjoMvOtqwQib2RjT7R3P9YQDydYZQdMVNRhzONA/640?wx_fmt=jpeg)![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaQTk2mOuiaKs337B82O5Axw1DonxODsHhn7S2wjTS0HOaMbX7Zn8hkmdf5bKRn1PodaBjsfvGPKmibA/640?wx_fmt=jpeg)

_netstat -lx_ 只用于列出所有监听 UNIX 端口。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaQTk2mOuiaKs337B82O5Axw10XniccbtehMHUJMfpDpfkzAlSR9lcWq7ewHUnbYPVtkCzM0qx2VyfSA/640?wx_fmt=jpeg)

### netstat -s

_netstat -s_ 用于列出所有端口的统计信息。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaQTk2mOuiaKs337B82O5Axw1Maia1LvQJI4jaBb75rFJptn0gyL3reE884IH82vibst4nrvKQWNSAOow/640?wx_fmt=jpeg)

_netstat -st_ 用于列出 TCP 端口的统计信息。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaQTk2mOuiaKs337B82O5Axw1iaJNJLQcEz3hf7ich8IdTEwSGdKkDSw5QPL7OGlc4LY78sm5R7OYkz8g/640?wx_fmt=jpeg)

_netstat -su_ 用于列出 UDP 端口的统计信息。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaQTk2mOuiaKs337B82O5Axw1bf23gToI5lX2O8pQ6td9DPdOtnLaoYlU5LcB5KoIejsb0iafEOqGWWw/640?wx_fmt=jpeg)

### netstat -p

netstat -p 可以与其他参数一起使用，例如 _netstat -pt_ 就可以列出服务名称和 PID 号。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaQTk2mOuiaKs337B82O5Axw13K9jiaxJbUG5Bt5zsjPdb6S3YCsvSgj3ALSRexCsPXr6iaqyRHDgzHTQ/640?wx_fmt=jpeg)

### netstat -c

使用 _netstat -c_ 将每隔一秒列出网络信息。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaQTk2mOuiaKs337B82O5Axw1EFRJAvsKZibVib2ib3wWUQYwANTD5BPVQYEOyeGXCJfI2S3PUr03mvFlQ/640?wx_fmt=jpeg)

### netstat -r

_netstat -r_ 用于列出路由核心信息。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaQTk2mOuiaKs337B82O5Axw1cKD2uG4jF7cSk8syicOQRtLTJyGu0eJmRxEVZ5DlhXUNNrBnzj9QezA/640?wx_fmt=jpeg)

### netstat --verbose

这条命令会列出系统支持的**地址族 (Address Family)**。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaQTk2mOuiaKs337B82O5Axw1hx5FfbuhOicHyElp0QR7LI7EbstPCYMfpjQQgicwLpspInUJEyafyn9Q/640?wx_fmt=jpeg)

> Address Family 简单来说就是底层是使用的哪种通信协议来递交数据的，如 AF_INET 用的是 TCP/IPv4；AF_INET6 使用的是 TCP/IPv6；而 AF_LOCAL 或者 AF_UNIX 则指的是本地通信（即本次通信是在当前主机上的进程间的通信），一般用绝对路径的形式来指明。

### netstat -i

_netstat -i_ 用来列出网络接口数据包，包括传输和接收具有 MTU（最大传输单元）的数据包。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaQTk2mOuiaKs337B82O5Axw1q1A67ibmoorAVLUgfrBeEhMrTUqLyd4NJf9CuibMLUsJbNRQJtUxVypA/640?wx_fmt=jpeg)

另外，**netstat -ie** 还用于列出内核接口表，和 _ifconfig_ 命令很相似

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaQTk2mOuiaKs337B82O5Axw1h8upZu3gGGWq9taI3zpPsWVM4UOabssiaMPR6IWe7q64u7LvqOPnicJw/640?wx_fmt=jpeg)

关于这个问题
------

所以，回到文章刚开始的那个疑问，netstat -tnpl 是干什么用的，其实这就是几个参数的组合

*   -t ：仅列出与 tcp 有关的信息
    
*   -n：以数字形式列出
    
*   -p：列出正在使用 socket PID 和 程序名称
    
*   -l：列出正在监听的服务器 socket
    

我们执行一下这个命令。

![](https://mmbiz.qpic.cn/mmbiz_jpg/A3ibcic1Xe0iaQTk2mOuiaKs337B82O5Axw1xImjGviaK0GwbrXWtzEPWHIp2daYuEXsaQnNGfK0rLIhzcPvbqbjVyA/640?wx_fmt=jpeg)

另外，在 Linux 中，已经推荐使用 _ss_ 来替代 netstat ，使用 _ip route_ 来替代 netstat -r ，使用 _ip -s link_ 来替代 netstat -i ，使用 _ip addr_ 来替代 netstat -g 了。

- EOF -

推荐阅读  点击标题可跳转

1、[说说 C++20 的格式化库](http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651170374&idx=1&sn=3d5ca26287120982d6fb6ee61f02a88f&chksm=80647719b713fe0f8dadc82254d3f7a2cbac04a3b11eb06f50df3b4a9f656d123c33e60ebdef&scene=21#wechat_redirect)

2、[理解可变参数模板](http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651170377&idx=1&sn=c8824e4a39ec44314b7c203569fd8a8b&chksm=80647716b713fe00c6b95b71cb4da10ebf3f28786df00a51d6655b177567febb95da7a603d91&scene=21#wechat_redirect)

3、[天天讲路由，那 Linux 路由到底咋实现的！？](http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651170375&idx=1&sn=6bfe3e018304ee8deb15aa78adf8157f&chksm=80647718b713fe0e5f79b825bbcf955efe72957cf7e4c3c5d615e45fbf5706fb5b15b5b579be&scene=21#wechat_redirect)
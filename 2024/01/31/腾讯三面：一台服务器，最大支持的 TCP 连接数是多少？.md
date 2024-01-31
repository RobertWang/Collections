> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5OTA1MDUyMA==&mid=2655486837&idx=1&sn=c893f05d7cab89ddaa1884d0f90ba298&chksm=bd724f028a05c614044b1782f0d654112819cb0bc5163da4ac30b81b2ff39ee9f7e81ec46086&mpshare=1&scene=1&srcid=0130Oa7kInV38Apg2tl7IONd&sharer_shareinfo=145ce60c241be5886aab4b4551358f72&sharer_shareinfo_first=145ce60c241be5886aab4b4551358f72#rd)

最近有读者面试腾讯的时候，被问到 2 个很有意思的问题：  

*   一个**服务端进程**最大能支持多少条 TCP 连接？
    
*   一台**服务器**最大能支持多少条 TCP 连接？
    

很多同学第一反应就是端口的限制，端口号最多是 65536 个，那就最多只能支持 65536 条 TCP 连接。

实际上这是不对的！

今天都带大家分析一波这两个问题。

一个服务端进程最多能支持多少条 TCP 连接？
-----------------------

首先我们要知道 TCP 连接本质上在内核里就是一个 socket 对象。

```
struct socket {  
    ....
    //INET域专用的一个socket表示, 提供了INET域专有的一些属性，比如 IP地址，端口等
    struct sock             *sk;  
    //TCP连接的状态：SYN_SENT、SYN_RECV、ESTABLISHED.....
    short                   type;  
    ....
};  

struct inet_sock {  
...
  __u32    daddr;   //IPv4的目标地址。  
  __u16    dport;   //目标端口。   
  __u32    saddr;   //源地址。  
  __u16    sport;   //源端口。  
...
};  


```

这个 socket 对象也就是一个数据结构，里面包含了 TCP 四元组的信息：源 IP、源端口、目标 IP、目标端口。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/J0g14CUwaZe4BCfbZtWBjDk24mYQOqY3SOT9W4ej1cezsgs69HDicrnbeRlQx8PUGxay9ElmwWxI2NIZm2NGHIA/640?wx_fmt=png&from=appmsg)TCP 四元组

所以， 只要确认了【源 IP、源端口、目标 IP、目标端口】这四个信息，就能在内核中找到这个 socket 对象，也就能确定一条 TCP 连接。

一个服务端进程通常是监听 1 个端口号（当然也可能监听多个端口号，这里不考虑），比如我的图解网站的 nginx 服务，就监听了 443 端口。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/J0g14CUwaZe4BCfbZtWBjDk24mYQOqY3jFHq9VTx7be11dAibIcs2v0CZNTLlucBsF1txyAdpYuFjtblgn1R8Zw/640?wx_fmt=png&from=appmsg)

你们看图解网站的时候，实际上就是通过 nginx 服务把网页数据发送给你们的。

然后，服务端进程除了会固定监听某个一个端口之外，也通常会绑定 `0.0.0.0` IP 地址。

这个 IP 地址是特殊的， `0.0.0.0` 指的是本机上的所有 IPV4 地址，如果一个主机有两个 IP 地址，`192.168.1.1` 和 `10.1.2.1`，并且该主机上的一个服务监听的地址是`0.0.0.0`，那么通过两个 IP 地址都能够访问该服务。

所以一个服务端进程，意味着他的 IP 地址和端口号是固定的（`0.0.0.0:443`）。

也就是当客户端与服务端建立一条 TCP 连接的时候，这个 TCP 连接的四元组信息中服务端的 IP 地址和端口号是固定的，能产生变化的就是客户端的 IP 地址和端口号了。

因此，一个服务端进程最大能支持的 TCP 连接个数的计算公式如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/J0g14CUwaZe4BCfbZtWBjDk24mYQOqY38kAESnOMXPoGscWa9NW0bpQ2fE8Gsf5tPU0A8x4lsnPLu3D8BF6SMQ/640?wx_fmt=png&from=appmsg)

对 IPv4，客户端的 IP 数最多为 `2` 的 `32` 次方，客户端的端口数最多为 `2` 的 `16` 次方。

那么一个服务端进程理想情况下，最大的 TCP 连接数约为 `2` 的 `48` 次方（`2^32 (ip数) * 2^16 (端口数`），这数值是非常夸张的了，约等于两百多万亿！

当然，服务端进程最大能支持的 TCP 连接数远不能达到理论上限，还会受到**文件描述符、内存大小资源的限制**，毕竟 socket 在 Linux 的视角其实就是文件资源，而且一个 socket 对象也会占用一定的内存资源。

因此，会受以下因素影响：

*   文件描述符限制，每个 TCP 连接都是一个文件，如果文件描述符被占满了，会发生 Too many open files。Linux 对可打开的文件描述符的数量分别作了三个方面的限制：
    

*   **系统级**：当前系统可打开的最大数量，通过 `cat /proc/sys/fs/file-max` 查看；
    
*   **用户级**：指定用户可打开的最大数量，通过 `cat /etc/security/limits.conf` 查看；
    
*   **进程级**：单个进程可打开的最大数量，通过 `cat /proc/sys/fs/nr_open` 查看；
    

*   **内存限制**，每个 TCP 连接都要占用一定内存，操作系统的内存是有限的，如果内存资源被占满后，会发生 OOM。
    

一台服务器最大最多能支持多少条 TCP 连接？
-----------------------

前面分析是一个服务端进程理的情况，理论上能最大支持约为 `2` 的 `48` 次方（`2^32 (ip数) * 2^16 (端口数`），约等于两百多万亿！

那到了一台服务器的视角就会有一点不一样。

一台服务器是可以有多个服务端进程的，每个服务端进程监听不同的端口，比如：ssh 的 22，Redis 的 6339，当然所有 65535 个端口你都可以用来监听一遍。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/J0g14CUwaZe4BCfbZtWBjDk24mYQOqY3kx6PmetuWy6k6qRRdZQiaJJ6NU0geGy3q94hupROqXk4PQJ2oq1j3yw/640?wx_fmt=png&from=appmsg)

当然所有 65535 个端口你都可以用来监听一遍，这样理论上线就到了 2 的 32 次方（ip 数）×2 的 16 次方（port 数）×2 的 16 次方（服务器 port 数）个，感兴趣你可以算一下，这个基本相当于无穷个了。

不过理想和实际总是会有差距的！

因为 Linux 每维护一条 TCP 连接都要花费资源，处理连接请求，保活，数据的收发时需要消耗一些 CPU，维持 TCP 连接主要消耗内存。

我们题目的问题是考虑最大多少个连接，所以我们先不考虑数据的收发，那么 TCP 在静止的状态下，就不怎么消耗 CPU 了，主要消耗内存，而 Linux 上内存是有限的。

首先，我们要知道一条处于 ESTABLISH 状态的 TCP 连接具体占用多大内存？

一个 TCP 对象占用的大小，等于它所包含的一些数据结构占用大小的总和，也是就把上面这些数据结构的大小累加起来，就是一个 TCP 连接占用的大小了。

这里直接给大家一个结论，**一条处于 ESTABLISH 状态的 TCP 连接占用的大小是 3.44 KB**（0.81K+2.19K+0.19K+0.25K）。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/J0g14CUwaZe4BCfbZtWBjDk24mYQOqY39W8dd4fwiaFpKtv6ZxWSyEdEZQ1IRG8ff8NicQEy0gdbADSa01U80OKA/640?wx_fmt=png&from=appmsg)TCP 对象内存开销总结

也就是，每一条静止状态的 TCP 连接大约需要吃 3.44K 的内存。

那么 **8 GB 物理内存的服务器，最大能支持的 TCP 连接数 = 8GB/3.44KB=2,438,956（约 240 万）**！

当然， 实际过程中的 TCP 连接，肯定不是静止状态的，还会进行发送数据和接收数据了，那么这些过程还是会额外消耗更多的内存资源的，并发很难达到百万级别。

总结
--

> 一个服务端进程最多能支持多少条 TCP 连接？

如果在不考虑服务器的内存和文件句柄资源的情况下，理论上一个服务端进程最多能支持约为 `2` 的 `48` 次方（`2^32 (ip数) * 2^16 (端口数`），约等于两百多万亿！

但是在实际中是支持不了这个数值的，每个 TCP 连接都是一个文件，会占用文件句柄资源，也会占用一定的内存空间。

> 一台服务器最大最多能支持多少条 TCP 连接？

一台服务器是可以有多个服务端进程的，每个服务端进程监听不同的端口，当然所有 65535 个端口你都可以用来监听一遍。

当然所有 65535 个端口你都可以用来监听一遍，这样理论上线就到了 2 的 32 次方（ip 数）×2 的 16 次方（port 数）×2 的 16 次方（服务器 port 数）个，这个基本相当于无穷个了。

但是 Linux 每维护一条 TCP 连接都要花费内存资源的，每一条静止状态（不发送数据和不接收数据）的 TCP 连接大约需要吃 3.44K 的内存，那么 8 GB 物理内存的服务器，最大能支持的 TCP 连接数 = 8GB/3.44KB=2,438,956（约 240 万）。

实际过程中的 TCP 连接，还会进行发送数据和接收数据了，那么这些过程还是会额外消耗更多的内存资源的，并发很难达到百万级别。

完！你学废了吗？理想和现实的差距！

- EOF -
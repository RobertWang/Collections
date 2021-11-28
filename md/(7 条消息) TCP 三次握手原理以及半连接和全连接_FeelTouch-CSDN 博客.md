> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/feeltouch/article/details/83155607)

### 问题描述

JAVA 的 client 和 server，使用 socket 通信。server 使用 NIO。 

1.  间歇性的出现 client 向 server 建立连接三次握手已经完成，但 server 的 selector 没有响应到这连接。 
2.  出问题的时间点，会同时有很多连接出现这个问题。 
3.  selector 没有销毁重建，一直用的都是一个。 
4.  程序刚启动的时候必会出现一些，之后会间歇性出现。

### 分析问题

正常 TCP 建连接三次握手过程：   
![](https://img-blog.csdn.net/20170801144656963?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcHRvbmxpeA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) 

*   第一步：client 发送 syn 到 server 发起握手； 
*   第二步：server 收到 syn 后回复 syn+ack 给 client； 
*   第三步：client 收到 syn+ack 后，回复 server 一个 ack 表示收到了 server 的 syn+ack（此时 client 的 56911 端口的连接已经是 established） 

从问题的描述来看，有点像 TCP 建连接的时候全连接队列（accept 队列）满了，尤其是症状 2、4. 为了证明是这个原因，马上通过 ss -s 去看队列的溢出统计数据：   
667399 times the listen queue of a socket overflowed

反复看了几次之后发现这个 overflowed 一直在增加，那么可以明确的是 server 上全连接队列一定溢出了   
接着查看溢出后，OS 怎么处理：

```
cat /proc/sys/net/ipv4/tcp_abort_on_overflow
```

tcp_abort_on_overflow 为 0 表示如果三次握手第三步的时候全连接队列满了那么 server 扔掉 client 发过来的 ack（在 server 端认为连接还没建立起来）   
为了证明客户端应用代码的异常跟全连接队列满有关系，我先把 tcp_abort_on_overflow 修改成 1，1 表示第三步的时候如果全连接队列满了，server 发送一个 reset 包给 client，表示废掉这个握手过程和这个连接（本来在 server 端这个连接就还没建立起来）。   
接着测试然后在客户端异常中可以看到很多 connection reset by peer 的错误，到此证明客户端错误是这个原因导致的。   
于是开发同学翻看 java 源代码发现 socket 默认的 backlog（这个值控制全连接队列的大小，后面再详述）是 50，于是改大重新跑，经过 12 个小时以上的压测，这个错误一次都没出现过，同时 overflowed 也不再增加了。   
到此问题解决，简单来说 TCP 三次握手后有个 accept 队列，进到这个队列才能从 Listen 变成 accept，默认 backlog 值是 50，很容易就满了。满了之后握手第三步的时候 server 就忽略了 client 发过来的 ack 包（隔一段时间 server 重发握手第二步的 syn+ack 包给 client），如果这个连接一直排不上队就异常了。

### 深入理解 TCP 握手过程中建连接的流程和队列

![](https://img-blog.csdn.net/20170801144813240?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcHRvbmxpeA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  
（图片来源：[http://www.cnxct.com/something-about-phpfpm-s-backlog/](http://www.cnxct.com/something-about-phpfpm-s-backlog/)）   
如上图所示，这里有两个队列：**syns queue(半连接队列）；accept queue（全连接队列）**   
三次握手中，在第一步 server 收到 client 的 syn 后，把相关信息放到半连接队列中，同时回复 syn+ack 给 client（第二步）；   
比如 syn floods 攻击就是针对半连接队列的，攻击方不停地建连接，但是建连接的时候只做第一步，第二步中攻击方收到 server 的 syn+ack 后故意扔掉什么也不做，导致 server 上这个队列满其它正常请求无法进来。

第三步的时候 server 收到 client 的 ack，如果这时全连接队列没满，那么从半连接队列拿出相关信息放入到全连接队列中，否则按 tcp_abort_on_overflow 指示的执行。   
这时如果全连接队列满了并且 tcp_abort_on_overflow 是 0 的话，server 过一段时间再次发送 syn+ack 给 client（也就是重新走握手的第二步），如果 client 超时等待比较短，就很容易异常了。   
在我们的 os 中 retry 第二步的默认次数是 2（centos 默认是 5 次）：   
net.ipv4.tcp_synack_retries = 2

### 如果 TCP 连接队列溢出，有哪些指标可以看呢？

上述解决过程有点绕，那么下次再出现类似问题有什么更快更明确的手段来确认这个问题呢？

**netstat -s**   
[root@server ~]# netstat -s | egrep “listen|LISTEN”   
667399 times the listen queue of a socket overflowed   
667399 SYNs to LISTEN sockets ignored

比如上面看到的 667399 times ，表示全连接队列溢出的次数，隔几秒钟执行下，如果这个数字一直在增加的话肯定全连接队列偶尔满了。

**ss 命令**   
[root@server ~]# ss -lnt   
Recv-Q Send-Q Local Address:Port Peer Address:Port   
0 50 _:3306_ :*

上面看到的第二列 Send-Q 表示第三列的 listen 端口上的全连接队列最大为 50，第一列 Recv-Q 为全连接队列当前使用了多少   
全连接队列的大小取决于：min(backlog, somaxconn) . backlog 是在 socket 创建的时候传入的，somaxconn 是一个 os 级别的系统参数   
半连接队列的大小取决于：max(64, /proc/sys/net/ipv4/tcp_max_syn_backlog)。 不同版本的 os 会有些差异

### 实践验证下上面的理解

把 java 中 backlog 改成 10（越小越容易溢出），继续跑压力，这个时候 client 又开始报异常了，然后在 server 上通过 ss 命令观察到：   
Fri May 5 13:50:23 CST 2017   
Recv-Q Send-QLocal Address:Port Peer Address:Port   
11 10 _:3306_ :*

按照前面的理解，这个时候我们能看到 3306 这个端口上的服务全连接队列最大是 10，但是现在有 11 个在队列中和等待进队列的，肯定有一个连接进不去队列要 overflow 掉

### 容器中的 Accept 队列参数

Tomcat 默认短连接，backlog（Tomcat 里面的术语是 Accept count）Ali-tomcat 默认是 200, Apache Tomcat 默认 100.   
ss -lnt   
Recv-Q Send-Q Local Address:Port Peer Address:Port   
0 100 _:8080_ :*

Nginx 默认是 511   
$sudo ss -lnt   
State Recv-Q Send-Q Local Address:PortPeer Address:Port   
LISTEN 0 511 _:8085_ :*   
LISTEN 0 511 _:8085_ :*

因为 Nginx 是多进程模式，也就是多个进程都监听同一个端口以尽量避免上下文切换来提升性能

### 进一步思考

如果 client 走完第三步在 client 看来连接已经建立好了，但是 server 上的对应连接实际没有准备好，这个时候如果 client 发数据给 server，server 会怎么处理呢？（有同学说会 reset，还是实践看看）   
先来看一个例子：   
![](https://img-blog.csdn.net/20170801144948288?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcHRvbmxpeA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  
（图片来自：[http://blog.chinaunix.net/uid-20662820-id-4154399.html](http://blog.chinaunix.net/uid-20662820-id-4154399.html)）   
如上图，150166 号包是三次握手中的第三步 client 发送 ack 给 server，然后 150167 号包中 client 发送了一个长度为 816 的包给 server，因为在这个时候 client 认为连接建立成功，但是 server 上这个连接实际没有 ready，所以 server 没有回复，一段时间后 client 认为丢包了然后重传这 816 个字节的包，一直到超时，client 主动发 fin 包断开该连接。   
这个问题也叫 client fooling，可以看这里：[https://github.com/torvalds/linux/commit/5ea8ea2cb7f1d0db15762c9b0bb9e7330425a071](https://github.com/torvalds/linux/commit/5ea8ea2cb7f1d0db15762c9b0bb9e7330425a071) （感谢浅奕的提示)   
从上面的实际抓包来看不是 reset，而是 server 忽略这些包，然后 client 重传，一定次数后 client 认为异常，然后断开连接。

### 过程中发现的一个奇怪问题

[root@server ~]# date; netstat -s | egrep “listen|LISTEN”   
Fri May 5 15:39:58 CST 2017   
1641685 times the listen queue of a socket overflowed   
1641685 SYNs to LISTEN sockets ignored

[root@server ~]# date; netstat -s | egrep “listen|LISTEN”   
Fri May 5 15:39:59 CST 2017   
1641906 times the listen queue of a socket overflowed   
1641906 SYNs to LISTEN sockets ignored

如上所示：   
overflowed 和 ignored 居然总是一样多，并且都是同步增加，overflowed 表示全连接队列溢出次数，socket ignored 表示半连接队列溢出次数，没这么巧吧。   
翻看内核源代码（[http://elixir.free-electrons.com/linux/v3.18/source/net/ipv4/tcp_ipv4.c](http://elixir.free-electrons.com/linux/v3.18/source/net/ipv4/tcp_ipv4.c)）：   
![](https://img-blog.csdn.net/20170801145033715?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcHRvbmxpeA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)   
可以看到 overflow 的时候一定会 drop++（socket ignored），也就是 drop 一定大于等于 overflow。   
同时我也查看了另外几台 server 的这两个值来证明 drop 一定大于等于 overflow：   
server1   
150 SYNs to LISTEN sockets dropped

server2   
193 SYNs to LISTEN sockets dropped

server3   
16329 times the listen queue of a socket overflowed   
16422 SYNs to LISTEN sockets dropped

server4   
20 times the listen queue of a socket overflowed   
51 SYNs to LISTEN sockets dropped

server5   
984932 times the listen queue of a socket overflowed   
988003 SYNs to LISTEN sockets dropped

### 那么全连接队列满了会影响半连接队列吗？

来看三次握手第一步的源代码（[http://elixir.free-electrons.com/linux/v2.6.33/source/net/ipv4/tcp_ipv4.c#L1249](http://elixir.free-electrons.com/linux/v2.6.33/source/net/ipv4/tcp_ipv4.c#L1249)）：   
![](https://img-blog.csdn.net/20170801145113666?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcHRvbmxpeA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)   
TCP 三次握手第一步的时候如果全连接队列满了会影响第一步 drop 半连接的发生。大概流程的如下：

```
tcp_v4_do_rcv->tcp_rcv_state_process->tcp_v4_conn_request
//如果accept backlog队列已满，且未超时的request socket的数量大于1，则丢弃当前请求  
  if(sk_acceptq_is_full(sk) && inet_csk_reqsk_queue_yong(sk)>1)
      goto drop;
```

### 总结

全连接队列、半连接队列溢出这种问题很容易被忽视，但是又很关键，特别是对于一些短连接应用（比如 Nginx、PHP，当然他们也是支持长连接的）更容易爆发。 一旦溢出，从 cpu、线程状态看起来都比较正常，但是压力上不去，在 client 看来 rt 也比较高（rt = 网络 + 排队 + 真正服务时间），但是从 server 日志记录的真正服务时间来看 rt 又很短。   
希望通过本文能够帮大家理解 TCP 连接过程中的半连接队列和全连接队列的概念、原理和作用，更关键的是有哪些指标可以明确看到这些问题。   
另外每个具体问题都是最好学习的机会，光看书理解肯定是不够深刻的，请珍惜每个具体问题，碰到后能够把来龙去脉弄清楚。

> **参考文章**
> 
> http://veithen.github.io/2014/01/01/how-tcp-backlog-works-in-linux.html
> 
> http://www.cnblogs.com/zengkefu/p/5606696.html
> 
> http://www.cnxct.com/something-about-phpfpm-s-backlog/
> 
> http://jaseywang.me/2014/07/20/tcp-queue-%E7%9A%84%E4%B8%80%E4%BA%9B%E9%97%AE%E9%A2%98/
> 
> http://jin-yang.github.io/blog/network-synack-queue.html#
> 
> http://blog.chinaunix.net/uid-20662820-id-4154399.html
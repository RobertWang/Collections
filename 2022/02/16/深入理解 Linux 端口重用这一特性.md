> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU2MTkwMTE4Nw==&mid=2247497842&idx=2&sn=8aa700d0f53b24e9c17d0745188e8b76&chksm=fc73039ecb048a8867053c1f913f72439bdff26393d51b3a1e0de3952eb35384e48c00871996&mpshare=1&scene=1&srcid=0216hrQPnqYnJEgEalVX8fhx&sharer_sharetime=1644999088154&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

大家好，我是小方！

我相信一定会有一部分同学会答说是不能的。因为很多人都遇到过 “Address already in use” 这个错误。而这个错误产生的原因就是端口已经被占用。

但其实在 Linux 3.9 以上的内核版本里，是允许多个进程绑定到同一个端口号上。这就是我们今天要说的 REUSEPORT 新特性。

本文中我们将阐述 REUSEPORT 是为了解决什么问题而产生的。如果有多个进程复用同一个端口，当用户请求到达时内核是如何选一个进程进行响应的。学习完本文，你将深刻掌握这一提升服务器端性能的利器！

一、 REUSEPORT 要解决的问题
-------------------

我觉得理解一个技术点的很重要的前提是要弄明白这个问题产生的背景，弄明白了背景再理解起技术点来就会容易许多。

关于 REUSEPORT 特性产生的背景其实在 linux 的 commit 中提供的足够详细了（参见：https://github.com/torvalds/linux/commit/da5e36308d9f7151845018369148201a5d28b46d）。

我就在这个 commit 中的信息的基础上给大家展开了说一说。

大家有过服务器端开发的经验的同学都知道，一般一个服务是固定监听某一个端口的。比如 Nginx 服务一般固定监听 80 或 8080，Mysql 服务固定监听 3306 等等。

在网民数量还不够多，终端设备也还没有爆炸的年代里，一直是在使用的是端口不可重复被监听的模式。但是到了 2010 年之后，Web 互联网已经发展到了一个高潮，移动端的设备也开始迎来了大发展。这个时候端口不能重用的性能瓶颈就暴露出来了。

应对海量流量的主要措施就是应用多进程模型。在端口不可被重复 bind 和 listen 的年代里，提供海量服务的多进程 Server 提供一般是采用如下两种进程模型来工作。

第一种是专门搞一个或多个进程服务 accept 新连接，接收请求，然后将请求转给其它的 worker 进程来处理。

![](https://mmbiz.qpic.cn/mmbiz_png/BBjAFF4hcwptKEcFR2Pab27jkOS3ZPonnia2jR0liaaqlicU1w9sUibngp9B4A6tSdfpuxtibQWHE1OgVWibhkDCDUcw/640?wx_fmt=png)

这种多进程模型有两个问题，首先第一个是 dispatcher 进程并不处理任务，需要转交给 worker 进程来处理和响应。这会导致一次额外的进程上下文切换的开销。第二个问题是如果流量特别大的时候 dispatcher 进程很容易成为制约整个服务 qps 提升的瓶颈。  

还有另一种多进程模型是多个进程复用一个 listen 状态的 socket，多个进程同时从一个 socket 中 accept 请求来处理。Nginx 就采用的是这种模型。

![](https://mmbiz.qpic.cn/mmbiz_png/BBjAFF4hcwptKEcFR2Pab27jkOS3ZPon6yiaj94lmxQ2icoN8wB3ja0d91EaWdwO1XicC9yo0CjQoDvnrZ9icJQiafA/640?wx_fmt=png)

这种进程模型解决了第一个模型的问题。但是又带来了新的问题。当 socket 收到一条连接的时候，不能把所有的 worker 进程都招呼起来。需要用锁来保证唯一性，这样就会有锁竞争的问题。  

二、REUSEPORT 的诞生
---------------

为了更高效地让多个用户态的进程接收和响应客户端的请求。Linux 在 2013 年的 3.9 版本中提供了 REUSEPORT 新特性。

![](https://mmbiz.qpic.cn/mmbiz_png/BBjAFF4hcwptKEcFR2Pab27jkOS3ZPon01H4ibYnuvquxUz3rKWg5ETufws7aFGdQJm5icp7tfcTzhyCbEEUsjpQ/640?wx_fmt=png)

> 内核详细 Commit 代码参见 https://github.com/torvalds/linux/commit/da5e36308d9f7151845018369148201a5d28b46d 和 https://github.com/torvalds/linux/commit/055dc21a1d1d219608cd4baac7d0683fb2cbbe8a

该特性允许同一机器上的多个进程同时创建**不同的 socket 来 bind 和 listen 在相同的端口上**。然后在**内核层面实现多个用户进程的负载均衡**。

我们来看下内核是如何支持 reuseport 这个特性的。

### 2.1 SO_REUSEPORT 设置

想给自己的服务开启 REUSEPORT 很简单，就是给自己 server 里 listen 用的 socket 上加这么一句。（这里以 c 为 demo，其它语言可能会有差异，但基本上差不多）

```
setsockopt(fd, SOL_SOCKET, SO_REUSEPORT, ...);
```

这行代码在内核中对应的处理步骤就是把内核 socket 的 sk_reuseport 字段设置为相应的值，开启的话是 1。

```
//file: net/core/sock.c
int sock_setsockopt(struct socket *sock, int level, int optname,
      char __user *optval, unsigned int optlen)
{
 ...
 switch (optname) {
  ...
  case SO_REUSEPORT:
   sk->sk_reuseport = valbool;
  ...
 }
}
```

### 2.2 bind 时的处理

内核在 inet_bind 时会调用到 inet_csk_get_port 函数。我们来看看 bind 时对 reuseport 的处理过程。来看源码：

```
//file: net/ipv4/inet_connection_sock.c
int inet_csk_get_port(struct sock *sk, unsigned short snum)
{
 ...
 //在绑定表（bhash）中查找，
 head = &hashinfo->bhash[inet_bhashfn(net, snum,
   hashinfo->bhash_size)];
 inet_bind_bucket_for_each(tb, &head->chain)
  //找到了，在一个命名空间下而且端口号一致，表示该端口已经绑定
  if (net_eq(ib_net(tb), net) && tb->port == snum)
   goto tb_found;
 ...
}
```

内核通过拉链哈希表的方式来管理所有的 bind 的 socket。其中 inet_bhashfn 是计算哈希值的函数。

![](https://mmbiz.qpic.cn/mmbiz_png/BBjAFF4hcwptKEcFR2Pab27jkOS3ZPonPo66l7mO5UAic4Doico3sIseczN9UYDO7z7gD4PVvvrrmC482yjviaF6Q/640?wx_fmt=png)

当计算找到哈希槽位以后，通过 inet_bind_bucket_for_each 来遍历所有的 bind 状态的 socket，目的是为了判断是否冲突。  

`net_eq(ib_net(tb), net)` 这个条件表示网络命名空间匹配，tb->port == snum 表示端口号匹配。这两个条件加起来，就是说在同一个命名空间下，该端口已经被绑定过了。我们接着看 tb_found 里会干啥。

```
//file: net/ipv4/inet_connection_sock.c
int inet_csk_get_port(struct sock *sk, unsigned short snum)
{
 ...
 if (((tb->fastreuse > 0 &&
       sk->sk_reuse && sk->sk_state != TCP_LISTEN) ||
      (tb->fastreuseport > 0 &&
       sk->sk_reuseport && uid_eq(tb->fastuid, uid))) &&
     smallest_size == -1) {
  goto success;
 } else {
  //绑定冲突
  ......
 }
```

我们看 tb->fastreuseport > 0 和 sk->sk_reuseport 这两个条件。

这两个条件的意思是已经 bind 的 socket 和正在 bind 的 socket 都开启了 SO_REUSEPORT 特性。符合条件的话，将会跳转到 success 进行绑定成功的处理。**也就是说，这个端口可以重复绑定使用！**

uid_eq(tb->fastuid, uid) 这个条件目的是安全性，必须要求相同的用户进程下的 socket 才可以复用端口。**避免跨用户启动相同端口来窃取另外用户服务上的流量。**

### 2.3 accept 响应新连接

当有多个进程都 bind 和 listen 了同一个端口的时候。有客户端连接请求到来的时候就涉及到选择哪个 socket（进程）进行处理的问题。我们再简单看一下，响应连接时的处理过程。

内核仍然是通过 hash + 拉链的方式来保存所有的 listen 状态的 socket。

![](https://mmbiz.qpic.cn/mmbiz_png/BBjAFF4hcwptKEcFR2Pab27jkOS3ZPonOJ0KyEAcPJiaVdxDMm6aatJS6wtt11TXN5rNm0MjZicnhs5Kic0eicBglg/640?wx_fmt=png)

查找 listen 状态的 socket 的时候需要查找该哈希表。我们进入响应握手请求的时候进入的一个关键函数 __inet_lookup_listener 来看。  

```
//file: net/ipv4/inet_hashtables.c
struct sock *__inet_lookup_listener(struct net *net,
        struct inet_hashinfo *hashinfo,
        const __be32 saddr, __be16 sport,
        const __be32 daddr, const unsigned short hnum,
        const int dif)
{
 //所有 listen socket 都在这个 listening_hash 中
 struct inet_listen_hashbucket *ilb = &hashinfo->listening_hash[hash];

begin:
 result = NULL;
 hiscore = 0;
 sk_nulls_for_each_rcu(sk, node, &ilb->head) {
  score = compute_score(sk, net, hnum, daddr, dif);
  if (score > hiscore) {
   result = sk;
   hiscore = score;
   reuseport = sk->sk_reuseport;
   if (reuseport) {
    phash = inet_ehashfn(net, daddr, hnum,
           saddr, sport);
    matches = 1;
   }
  } else if (score == hiscore && reuseport) {
   matches++;
   if (((u64)phash * matches) >> 32 == 0)
    result = sk;
   phash = next_pseudo_random32(phash);
  }
 }
 ...
 return result;
}
```

其中 sk_nulls_for_each_rcu 是在遍历所有 hash 值相同的 listen 状态的 socket。注意看 compute_score 这个函数，这里是计算匹配分。当有多个 socket 都命中的时候，匹配分高的优先命中。我们来看一下这个函数里的一个细节。

```
//file: net/ipv4/inet_hashtables.c
static inline int compute_score(struct sock *sk, ...)
{
 int score = -1;
 struct inet_sock *inet = inet_sk(sk);

 if (net_eq(sock_net(sk), net) && inet->inet_num == hnum &&
   !ipv6_only_sock(sk)) {
  //如果服务绑定的是 0.0.0.0，那么 rcv_saddr 为假
  __be32 rcv_saddr = inet->inet_rcv_saddr;
  score = sk->sk_family == PF_INET ? 2 : 1;
  if (rcv_saddr) {
   if (rcv_saddr != daddr)
    return -1;
   score += 4;
  }
  ... 
 }
 return score;
}
```

那么匹配分解决的是什么问题呢？为了描述的更清楚，我们假设某台服务器有两个 ip 地址，分别是 10.0.0.2 和 10.0.0.3。我们启动了如下三个服务器进程。

```
A 进程：./test-server 10.0.0.2 6000
B 进程：./test-server 0.0.0.0 6000
C 进程：./test-server 127.0.0.1 6000
```

那么你的客户端如果指定是连接 10.0.0.2:6000，那么 A 进程会优先执行。因为当匹配到 A 进程的 socket 的时候，需要看一下握手包中的目的 ip 和这个地址是否匹配，确实匹配那得分就是 4 分，最高分。

如果你指定连接的是 10.0.0.3，那么 A 进程就无法被匹配到。这个时候 B 进程监听时指定的是 0.0.0.0（rcv_saddr 为 false），则不需要进行目的地址的比对，得分为 2。由于没有更高分，所以这次命中的是 B 进程。

C 进程只有你在本机访问，且指定 ip 使用 127.0.0.1 才能命中，得分也是为 4 分。外部服务器或者是在本机使用其它 ip 都无法访问的到。

如果当多个 socket 的匹配分一致，通过调用 next_pseudo_random32 进行随机的选择。**在内核态做了负载均衡的事情，选定一个具体的 socket，避免了多个进程在同一个 socket 上的锁竞争。**

三、动手实践
------

动手试试能体会更深刻。为此我动手写了个简单的开启 SO_REUSEPORT 特性的 server 的代码。核心就是给 server 的 socket

详细源码参见：https://github.com/yanfeizhang/coder-kung-fu/blob/main/tests/network/test08/server.c。

### 3.1 相同 port 多服务启动

编译后，分别在多个控制台下运行一下试试，看是否能够启动起来。

```
$./test-server 0.0.0.0 6000
Start server on 0.0.0.0:6000 successed, pid is 23179
$./test-server 0.0.0.0 6000
Start server on 0.0.0.0:6000 successed, pid is 23177
$./test-server 0.0.0.0 6000
Start server on 0.0.0.0:6000 successed, pid is 23185
......
```

没错，全部起来了！这个 6000 的端口被多个 server 进程重复使用了。

### 3.2 内核负载均衡验证

由于上述几个监听了相同端口的进程都使用的是 0.0.0.0，那么在计算 score 的时候，他们的得分就都是 2 分。那么就由内核以随机的方式进行负载均衡了。

我们再启动一个客户端，随意发起几个连接请求，统计一下各个 server 进程收到的连接数。如下图可见，该服务器上收到的连接的确是平均散列在各个进程里了。

```
Server 0.0.0.0 6000 (23179) accept success:15
Server 0.0.0.0 6000 (23177) accept success:25
Server 0.0.0.0 6000 (23185) accept success:20
Server 0.0.0.0 6000 (23181) accept success:19
Server 0.0.0.0 6000 (23183) accept success:21
```

### 3.3 匹配优先级验证

动用一台两个 ip 地址 的服务器，还假设你的 ip 分别是 10.0.0.2 和 10.0.0.3。启动了如下三个服务器进程。

```
A 进程：./test-server 10.0.0.2 6000
B 进程：./test-server 0.0.0.0 6000
```

另外一台客户端通过使用 telnet 命令即可测试。

```
$ telnet 10.0.0.2 6000 发现是命中 A 进程。
$ telnet 10.0.0.3 6000 发现是命中 B 进程。
```

### 3.4 跨用户安全性验证

先以 A 用户启动一个服务

```
$./test-server 0.0.0.0 6000
Start server on 0.0.0.0:6000 successed, pid is 30914
```

再切换到另外一个用户下，比如 root。

```
# ./test-server 0.0.0.0 6000
Server 30481 Error : Bind Failed!
```

这时候发现 bind 不会通过，服务启动失败！

四、总结
----

在 Linux 3.9 以前的版本中，一个端口只能被一个 socket 绑定。在多进程的场景下，无论是使用一个进程来在这个 socket 上 accept，还是说用多个 worker 来 accept 同一个 socket，在高并发的场景下性能都显得有那么一些低下。

在 2013 年发布的 3.9 中添加了 reuseport 的特性。该特定允许多个进程分别用不同的 socket 绑定到同一个端口。当有流量到达的时候，在内核态以随机的方式进行负载均衡。避免了锁的开销。

Linux 的这一特性是非常有用的，可惜还有大量的工程师不理解它的原理，也更是没有把它用起来，实在可惜！

如果你们业务用的是 Linux 上的多进程 server，赶快去检查下有没有开启 reuseport。如果没有开启你想办法给它加上，再出个性能数据对比，上半年的绩效就有了😎

如果你使用的是 1.9.1 以上版本的 nginx，只需要一行简单的配置就可以体验这个特性。

```
server {
  listen 80 reuseport;
  ...
}
```
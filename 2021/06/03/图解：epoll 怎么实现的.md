> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666554993&idx=2&sn=29ac11b328dad7ac755aca99107e5692&chksm=80dca2dab7ab2bcc44326bae22266deb8b7c7d9dd735a675452e5e291f2434718f6d6083beec&mpshare=1&scene=1&srcid=0603iOMUcQ1n0x6dXsiI40Yt&sharer_sharetime=1622695979563&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

`epoll` 可以说是编写高性能服务端程序必不可少的技术，在介绍 `epoll` 之前，我们先来了解一下 `多路复用I/O` 吧。

多路复用 I/O
--------

`多路复用I/O`：是指内核负责监听多个 I/O 流，当任何一个 I/O 流处于就绪状态（可读或可写）时都会通知进程，以便可以处理该 I/O 流上的数据。如 图 1 所示：

与传统的阻塞型 I/O 相比，多路复用 I/O 的优点是可以同时监听多个 I/O 流，并且会把就绪的 I/O 流告知进程。

epoll 原理
--------

在 Linux 系统中，有多种多路复用 I/O 的实现，比如 select 和 poll 等。而 epoll 也是多路复用 I/O 一种实现，与 select 和 poll 相比，epoll 在性能上有较大的提升。

### 红黑树

epoll 内部使用红黑树来保存所有监听的 socket，红黑树是一种平衡二叉树，添加和查找元素的时间复杂度为 O(log n)，其结构如 图 2 所示：

![](https://mmbiz.qpic.cn/mmbiz_png/ciab8jTiab9J66fibnLrV4ibJoMLH6HhEyVaFaufJdiaAuv0cDFN8VhicUaPFibgl5FB8JAbT7vF34FU5elTg0ENibLJRg/640?wx_fmt=png)

epoll 通过 socket 句柄来作为 key，把 socket 保存在红黑树中。如 图 2 所示，每个节点中的数字代表着 socket 句柄。  

把监听的 socket 保存在红黑树中的目的是，为了在修改监听 socket 的读写事件时，能够通过 socket 句柄快速找到对应的 socket 对象。

### 就绪队列

另外，epoll 还维护着一个就绪队列，当 epoll 监听的 socket 状态发生改变（变为可读或可写）时，就会把就绪的 socket 添加到就绪队列中。如 图 3 所示：

![](https://mmbiz.qpic.cn/mmbiz_png/ciab8jTiab9J66fibnLrV4ibJoMLH6HhEyVaqv4NvGthdXHNf4Oia3GxSFhcLIaRMcR8zIVTH7sZZicvUXQ9znicYwlGw/640?wx_fmt=png)

当 socket 从网络中获取到数据后，会发生通知给 epoll，epoll 会将当前 socket 添加到就绪队列中，并且唤醒等待中的进程（也就是调用 `epoll_wait` 的进程）。

当 socket 状态发生变化时，会调用 `ep_poll_callback` 函数来通知 epoll，我们来看看这个函数的处理过程：

```
static int ep_poll_callback(wait_queue_t *wait, unsigned mode, 
                            int sync, void *key)
{
    ...
    struct epitem *epi = ep_item_from_wait(wait);
    struct eventpoll *ep = epi->ep;
    ...
    // 1) 把 socket 添加到就绪队列中
    list_add_tail(&epi->rdllink, &ep->rdllist);
is_linked:
    // 2) 唤醒调用 epoll_wait() 而被阻塞的进程
    if (waitqueue_active(&ep->wq))
        wake_up_locked(&ep->wq);
    ...
    return 1;
}

```

`ep_poll_callback` 函数的意图很清晰，主要完成两个工作：

*   把就绪的 socket 添加到就绪队列中。
    
*   唤醒调用 `epoll_wait` 函数而被阻塞的进程。
    

当进程被唤醒后，就会从就绪队列中，把就绪的 socket 复制到用户提供的数组中。如 图 4 所示：

![](https://mmbiz.qpic.cn/mmbiz_png/ciab8jTiab9J66fibnLrV4ibJoMLH6HhEyVaSJvSuYI5uIGF1mxNDzYsd2eBzxCgwMUumoZBsOkMDuokAnw0CYm8zg/640?wx_fmt=png)

如 图 4 所示，在调用 `epoll_wait` 时需要提供一个 `events` 数组来存储就绪的 socket。当 `epoll_wait` 返回后，用户就可以从`events` 数组中获取到就绪的 socket，并可对其进行读写操作。

总结
--

本文主要通过图解的方式大概介绍了 `epoll` 的原理，但很多实现的细节只能通过阅读源码来了解。

- EOF -

推荐阅读  点击标题可跳转

1、[Docker 不香吗，为啥还要 K8s ？](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666554921&idx=1&sn=da23617224fe0dcf819cc5dad8818d6d&chksm=80dca282b7ab2b94b48a05def3275f395c653f528718037b5d3bc762001112f7f33697137888&scene=21#wechat_redirect)

2、[面试官：进程和线程，我只问这 19 个问题](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666554921&idx=2&sn=60b69d5b1cfae4e1ebd338ba34aa2e7f&chksm=80dca282b7ab2b948f9ae61013e25e3494b61aa7a97400c56eb11be53c6f94d0062e2e3dbf4f&scene=21#wechat_redirect)

3、[Python 里最具代表性的符号，竟如此强大](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666554929&idx=2&sn=9766563a434e3084395784758960a4c5&chksm=80dca29ab7ab2b8c7f35afe0222f47947d77e555af308699bc87888f3ab88f4528e708d2cc0b&scene=21#wechat_redirect)

看完本文有收获？请分享给更多人  

推荐关注「Linux 爱好者」，提升 Linux 技能

公众号

点赞和在看就是最大的支持❤️
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666555756&idx=2&sn=05ccfd821112de68bea292ab498e43cd&chksm=80dcadc7b7ab24d114667c0fb823021062fee20003978ae5dc9497e8e456c402360d0254939b&scene=21#wechat_redirect)

前言
--

IO 是计算机体系中重要的一部分 。不同的 IO 设备有着不同的特点：数据率不一样、传送单位不一样，数据表示不一样，等等。所以，很难实现一种统一的输入输出方法。  

IO 有两种操作，**同步 IO 和异步 IO**。同步 IO 指的是，必须等待 IO 操作完成后，控制权才返回给用户进程。异步 IO 是，无须等待 IO 操作完成，就将控制权返回给用户进程。

![](https://mmbiz.qpic.cn/mmbiz_gif/bZqlQqGqRvU069YaHcYP6ChjJa23oDZqL9dc5ABzpC5XcgulUDQ9ywxhY8wENOdlRUUJOOiaOedjzCl0aiawGWCA/640?wx_fmt=gif)

上面就是一个典型的阻塞 IO，对方还没有准备好回啥，发送只能等着，知道对方想好回复语，再进行回复。下面学习一下常见的 4 种 IO 模型。  

阻塞 IO 模型  

-----------

在 Linux ，默认情况下所有的 socket 都是阻塞的，一个典型的读操作流程如图所示。

![](https://mmbiz.qpic.cn/mmbiz_gif/bZqlQqGqRvU069YaHcYP6ChjJa23oDZq9mMM2GQQZmrb4Jn0h8YDRKxGSRBkM74gOjwCGbcRcibcHrBBhPZfMbA/640?wx_fmt=gif)

阻塞和非阻塞的概念描述的是**用户线程调用内核 IO 操作**的方式：阻塞是指 IO 操作需要彻底完成后才返回到用户空间；而非阻塞是指 IO 操作被调用后立即返回给用户一个状态值，不需要等到 IO 操作彻底完成。

当应用进程调用了 recvfrom 这个系统调用后，系统内核就开始了 IO 的第一个阶段 ：准备数据。  
  
对于网络 IO 来说，很多时候数据在一开始还没到达时，系统内核就要等待足够的数据到来。而在用户进程这边，整个进程会被阻塞。  
  
当系统内核一直等到数据准备好了，它就会将数据从系统内核中拷贝到用户内存中，然后系统内核返回结果，用户进程才解除阻塞的状态，重新运行起来。所以，阻塞 IO 模型的特点就是 **IO 执行的两个阶段都被阻塞**了。

大部分的 socke 接口都是阻塞型的。所谓阻塞型接口是指系统调用时却不返回调用结果，并让当前线程一直处于阻塞状态，只有当该系统调用获得结果或者超时出错时才返回结果。  
  
实际上，除非特别指定，几乎所有的 IO 接口都阻塞型的。这给网络编程带来了一个很大的问题，如在调用 send 的同时，线程处于阻塞状态，则在此期间，线程将无法执行任何运算或响应任何网络请求。

非阻塞 IO 模型
---------

在 Linux 下，可以通过设置 socket IO 变为非阻塞状态。当一个非阻塞的 socket 执行 read 操作时，流程如图：

![](https://mmbiz.qpic.cn/mmbiz_gif/bZqlQqGqRvU069YaHcYP6ChjJa23oDZqUOSADm2o97paCtLwyl6Uxm1ur9AENCffMqBZj1PywNfqIzh9MLltJQ/640?wx_fmt=gif)

当用户进程发出 read 操作时，如果内核中的数据还没有准备好，那么它**并不会 block 用户进程**，而是立刻返回一个错误。  
  
从用户进程角度讲，它发起 read 操作后，并不需要等待，而是马上就得到了一个结果 当用户进程判断结果是一个错误时，它就知道数据还没有准备好，于是它可以再次发送 read 操作。  
  
一旦内核中的数据准备好了，并且又再次收到了用户进程的系统调用，那么它马上就将数据复制到了用户内存中，然后返回正确的返回值。

所以，在非阻塞式 IO 中，用户进程其实需要**不断地主动询问 kernel 数据是否准备好。**非阻塞的接口相比于阻塞型接口的显著差异在于被调用之后立即返回，使用如下的函数可以将某句柄归设为非阻塞状态：fcntl(fd , F_SETFL , O_NONBLOCK);

在非阻塞状态下，recv 接口在被调用后立即返回，返回值代表了不同的含义，如下所述。

*   recv 返回值大于 0，表示接收数据完毕，返回值即是接收到的字节数。
    
*   recv 返回 0，表示连接已经正常断开。
    
*   recv 返回 -1 ，且 errno 等于 EAGAIN ，表示 recv 操作还没执行完成。
    
*   recv 返回 -1，且 errno 不等于 EAGAIN ，表示 recv 操作遇到系统错误 errno。
    

可以看到服务器线程可以通过循环调用 recv 接口，可以在单个线程内实现对所有连接的数据接收。但是上述模型绝不被推荐，因为循环调用 recv 将**大幅度占用 CPU 使用率**。  
  
此外，在这个方案 recv 更多的是起到检测 “操作是否完成” 的作用，实际操作系统提供了更为高效的检测 “操作是否完成” 作用的接口，例如 select 多路复用模式，可以次检测多个连接是存活跃。  
  

多路 IO 复用模型
----------

多路 IO 复用，有时也称为事件驱动 IO。它的基本原理就是有个函数会不断地轮询所负责的所有 socket ，当某个 socket 有数据到达了，就通知用户进程。IO 复用模型的流程如图：

![](https://mmbiz.qpic.cn/mmbiz_gif/bZqlQqGqRvU069YaHcYP6ChjJa23oDZqdx2dMiclc7cjQRXvsQRu7EqYlMrDedOnq7CgqMCohib2iaibOUe4MaHYVQ/640?wx_fmt=gif)

当用户进程调用了 select ，那么整个进程会被阻塞，而同时，内核会 **"监视" 所有 select 负责的 socket** ，当任何一个 socket 中的数据准备好了， select 就会返回。这个时候用户进程再调用 read 操作，将数据从内核拷贝到用户进程。

这个模型和阻塞 IO 的模型其实并没有太大的不同，事实上还更差一些 因为这里需要使用两个系统调用，而阻塞 IO 只调用了一个系统调用 recvfrom，用 **select 的优势在于它可以同时处理多个连接**。  

如果处理的连接数不是很高的话，使用 select/epoll Web server 定比使用多线程的阻塞 IO Web server 性能更好，可能延迟还更大；select/poll 的优势并不是对于单个连接能处理得更快，而是在于能处理更多的连接。  
  

异步 IO 模型
--------

![](https://mmbiz.qpic.cn/mmbiz_gif/bZqlQqGqRvU069YaHcYP6ChjJa23oDZqdx2dMiclc7cjQRXvsQRu7EqYlMrDedOnq7CgqMCohib2iaibOUe4MaHYVQ/640?wx_fmt=gif)

上面是异步 IO 模型。  

用户进程发起 read 操作之后，立刻就可以开始去做其他的事；而另一方面，从内核的角度，当它收到一个异步的 read 请求操作之后，**首先会立刻返回**，所以不会对用户进程产生任何阻塞。  
  
然后，内核会等待数据准备完成，然后将数据拷贝到用户内存中，当这一切都完成之后，**内核会给用户进程发送一个信号**，返回 read 操作已完成的信息。

调用阻塞 IO 一直阻塞住对应的进程直到操作完成，而非阻塞 IO 在内核还在准备数据的情况下会立刻返回。两者的区别就在于同步 IO 进行 IO 操作时会阻塞进程。  
  
非阻塞 IO 在执行 recvfrom 这个系统调用的时候，如果内核的数据没有准备好，这时候不会阻塞进程。但是当内核中数据准备好时，**recvfrom 会将数据从内核拷贝到用户内存中**，这个时候进程则被阻塞。  
  
而异步 IO 则不 样，当进程发起 IO 操作之后，就直接返回，直到内核发送一个信号，告诉进程 IO 已完成，则在这整个过程中，进程完全没有被阻塞。  
  

絮叨
--

经过上面的学习，你会发现非阻塞 IO 和异步 IO 的区别还是很明显的。  
  
在非阻塞 IO 中，虽然进程大部分时间都不会被阻塞，但是它仍然要求进程去**主动检查**，并且当数据准备完成以后，也需要进程主动地再次调用 recvfrom 来将数据拷贝到用户内存中。  
  
而异步 IO 则完全不同，它就像是用户进程将整个 IO 操作交给了内核完成，然后内核做完后**发信号**通知。

IO 作为计算机的基础知识，后台开发务必要掌握。更多网络编程相关的知识可以去学习 unix 网络编程，祝大家学习愉快！  

- EOF -

1、[C++ 后台开发知识点及学习路线](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666555749&idx=2&sn=80292e2498dbaeea9c23e647a59064fd&chksm=80dcadceb7ab24d8495f7727368f8f9fde222e10f50fa4823701c3703dbeae525f9ae0c4ca66&scene=21#wechat_redirect)

2、[read 文件一个字节实际会发生多大的磁盘 IO？](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666555745&idx=3&sn=fbd861eb4900cdc04d20cf333666829c&chksm=80dcadcab7ab24dce0e50872b4d0a6525fee77ae915c12507d50ec46fd66f946bb5c3769b72d&scene=21#wechat_redirect)

3、[深入理解 Go scheduler 调度器：GPM 源码分析](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666555715&idx=2&sn=5bffdefcad91221ca7c9b5de1e6c7e1a&chksm=80dcade8b7ab24fed9a2638629f99583bb3ba208ea91b93913bd95ac3368cb7e8a368855d083&scene=21#wechat_redirect)
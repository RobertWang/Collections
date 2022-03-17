> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651162977&idx=1&sn=80adb362b2d1adb973bbb9bf15928aad&chksm=8064583eb713d12891e34a392067c736ff53d9eec294fac3ad31a314a536c3600b0587779879&scene=21#wechat_redirect)

**导语 |** 最近使用了基于 Nginx 的 OpenResty 的框架，于是对 Nginx 相关内容进行了学习，现将一些理解撰写成文，和大家探讨。

**一、Nginx 基础架构**

Nginx 启动后以 daemon 形式在后台运行，后台进程包含一个 master 进程和多个 worker 进程。如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95ZygTJHpRA2GHaWdgPjlkJ2vHGFmX2YibdEibWA4om52kgWO0RfubAwRB4ogt04Thicicib9L6TF07QMg/640?wx_fmt=png)

Nginx 是由一个 master 管理进程，多个 worker 进程处理工作的多进程模型。基础架构设计，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95ZygTJHpRA2GHaWdgPjlkJ14aY4mb4rBJTGmbl81cB3LhEthq5qERVUqnGdxn9jz0qtXWNzbacXw/640?wx_fmt=png)

Master 负责管理 worker 进程，worker 进程负责处理网络事件。**整个框架被设计为一种依赖事件驱动、异步、非阻塞的模式**。

如此设计的优点有：

*   可以充分利用多核机器，增强并发处理能力。
    
*   多 worker 间可以实现负载均衡。
    
*   Master 监控并统一管理 worker 行为。在 worker 异常后，可以主动拉起 worker 进程，从而提升了系统的可靠性。并且由 Master 进程控制服务运行中的程序升级、配置项修改等操作，从而增强了整体的动态可扩展与热更的能力。
    

**二、Master 进程**

**1. 核心逻辑**

Master 进程的主逻辑在 ngx_master_process_cycle，核心关注源码：

```
ngx_master_process_cycle(ngx_cycle_t *cycle)
{
    ...
    ngx_start_worker_processes(cycle, ccf->worker_processes,
                                        NGX_PROCESS_RESPAWN);
    ...
    for ( ;; ) {
        if (delay) {...}
        ngx_log_debug0(NGX_LOG_DEBUG_EVENT, cycle->log, 0, "sigsuspend");
        sigsuspend(&set);
        ngx_time_update();
        ngx_log_debug1(NGX_LOG_DEBUG_EVENT, cycle->log, 0,
                             "wake up, sigio %i", sigio);
        if (ngx_reap) {
            ngx_reap = 0;
            ngx_log_debug0(NGX_LOG_DEBUG_EVENT, cycle->log, 0, "reap children");
            live = ngx_reap_children(cycle);
        }
        if (!live && (ngx_terminate || ngx_quit)) {...}
        if (ngx_terminate) {...}
        if (ngx_quit) {...}
        if (ngx_reconfigure) {...}
        if (ngx_restart) {...}
        if (ngx_reopen) {...}
        if (ngx_change_binary) {...}
        if (ngx_noaccept) {
            ngx_noaccept = 0;
            ngx_noaccepting = 1;
            ngx_signal_worker_processes(cycle,
ngx_signal_value(NGX_SHUTDOWN_SIGNAL));
        }
    }
 }
```

由上述代码，可以理解，master 进程主要用来管理 worker 进程，具体包括如下 4 个主要功能：

**1）接受来自外界的信号**。其中 master 循环中的各项标志位就对应着各种信号，如：ngx_quit 代表 QUIT 信号，表示优雅的关闭整个服务。

**2）向各个 worker 进程发送信**。比如 ngx_noaccept 代表 WINCH 信号，表示所有子进程不再接受处理新的连接，由 master 向所有的子进程发送 QUIT 信号量。

**3）监控 worker 进程的运行状态**。比如 ngx_reap 代表 CHILD 信号，表示有子进程意外结束，这时需要监控所有子进程的运行状态，主要由 ngx_reap_children 完成。

**4）当 woker 进程退出后（异常情况下），会自动重新启动新的 woker 进程**。主要也是在 ngx_reap_children。

**2. 热更**

**1）热重载 - 配置热更**

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95ZygTJHpRA2GHaWdgPjlkJT1Cbnyb9FWPLeQRkHnYTachTc1dKJLLeNMcPQpDITCreicVZP8UwIWA/640?wx_fmt=png)

Nginx 热更配置时，可以保持运行中平滑更新配置，具体流程如下：

*   更新 nginx.conf 配置文件，向 master 发送 SIGHUP 信号或执行 nginx -s reload
    
*   Master 进程使用新配置，启动新的 worker 进程
    
*   使用旧配置的 worker 进程，不再接受新的连接请求，并在完成已存在的连接后退出
    

**2）热升级 - 程序热更**

Nginx 热升级过程如下：

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95ZygTJHpRA2GHaWdgPjlkJoibA0IP1OLkriciaSgOdMVpDdj6yticPDL60aic4yviaONp9260htMVNNicXA/640?wx_fmt=png)

*   将旧 Nginx 文件换成新 Nginx 文件（注意备份）
    
*   向 master 进程发送 USR2 信号（平滑升级到新版本的 Nginx 程序）
    
*   master 进程修改 pid 文件号，加后缀. oldbin
    
*   master 进程用新 Nginx 文件启动新 master 进程，此时新老 master/worker 同时存在。
    
*   向老 master 发送 WINCH 信号，关闭旧 worker 进程，观察新 worker 进程工作情况。若升级成功，则向老 master 进程发送 QUIT 信号，关闭老 master 进程；若升级失败，则需要回滚，向老 master 发送 HUP 信号（重读配置文件），向新 master 发送 QUIT 信号，关闭新 master 及 worker。
    

**三、Worker 进程**

**1. 核心逻辑**

Worker 进程的主逻辑在 ngx_worker_process_cycle，核心关注源码：

```
ngx_worker_process_cycle(ngx_cycle_t *cycle, void *data)
{
    ngx_int_t worker = (intptr_t) data;
    ngx_process = NGX_PROCESS_WORKER;
    ngx_worker = worker;
    ngx_worker_process_init(cycle, worker);
    ngx_setproctitle("worker process");
    for ( ;; ) {
        if (ngx_exiting) {...}
        ngx_log_debug0(NGX_LOG_DEBUG_EVENT, cycle->log, 0, "worker cycle");
        ngx_process_events_and_timers(cycle);
        if (ngx_terminate) {...}
        if (ngx_quit) {...}
        if (ngx_reopen) {...}
    }
}
```

由上述代码，可以理解，worker 进程主要在处理网络事件，通过 ngx_process_events_and_timers 方法实现，其中事件主要包括：网络事件、定时器事件。

**2. 事件驱动 - epoll**

Worker 进程在处理网络事件时，依靠 epoll 模型，来管理并发连接，实现了事件驱动、异步、非阻塞等特性。如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95ZygTJHpRA2GHaWdgPjlkJZuH21Bpx9acBSZcWPvqb2KH4cGCibpmrgr73FictFMtmsZRZs8eWBG2A/640?wx_fmt=png)

通常海量并发连接过程中，每一时刻（相对较短的一段时间），往往只需要处理一小部分有事件的连接即活跃连接。基于以上现象，epoll 通过将连接管理与活跃连接管理进行分离，实现了高效、稳定的网络 IO 处理能力。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95ZygTJHpRA2GHaWdgPjlkJSzPUTic0tIqLdrQTh3CUm3dt6iaeHd1VOj5Ot4iacC2eulePsko96bOyQ/640?wx_fmt=png)

其中，epoll 利用红黑树高效的增删查效率来管理连接，利用一个双向链表来维护活跃连接。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95ZygTJHpRA2GHaWdgPjlkJ8lF2gl3ORjmeRdmV9J06ldBtwvPiabstn8icv7HZunGic9icZth0BGeW4w/640?wx_fmt=png)

**3. 惊群**

由于 worker 都是由 master 进程 fork 产生，所以 worker 都会监听相同端口。这样多个子进程在 accept 建立连接时会发生争抢，带来著名的 “惊群” 问题。

Worker 核心处理逻辑 ngx_process_events_and_timers 核心代码如下：

```
void ngx_process_events_and_timers(ngx_cycle_t *cycle){
    //这里面会对监听socket处理
    ...
    if (ngx_accept_disabled > 0) {
            ngx_accept_disabled--;
    } else {
        //获得锁则加入wait集合,
        if (ngx_trylock_accept_mutex(cycle) == NGX_ERROR) {
            return;
        }
        ...
        //设置网络读写事件延迟处理标志，即在释放锁后处理
        if (ngx_accept_mutex_held) {
            flags |= NGX_POST_EVENTS;
        }
    }
    ...
    //这里面epollwait等待网络事件
    //网络连接事件，放入ngx_posted_accept_events队列
    //网络读写事件，放入ngx_posted_events队列
    (void) ngx_process_events(cycle, timer, flags);
    ...
    //先处理网络连接事件，只有获取到锁，这里才会有连接事件
    ngx_event_process_posted(cycle, &ngx_posted_accept_events);
    //释放锁，让其他进程也能够拿到
    if (ngx_accept_mutex_held) {
        ngx_shmtx_unlock(&ngx_accept_mutex);
    }
    //处理网络读写事件
    ngx_event_process_posted(cycle, &ngx_posted_events);
}
```

由上述代码可知，Nginx 解决惊群的方法：

1）将连接事件与读写事件进行分离。连接事件存放为 ngx_posted_accept_events，读写事件存放为 ngx_posted_events。

2）设置 ngx_accept_mutex 锁，只有获得锁的进程，才可以处理连接事件。

**4. 负载均衡**

Worker 间的负载关键在于各自接入了多少连接，其中接入连接抢锁的前置条件是 ngx_accept_disabled > 0，所以 ngx_accept_disabled 就是负载均衡机制实现的关键阈值。

```
ngx_int_t             ngx_accept_disabled;
ngx_accept_disabled = ngx_cycle->connection_n / 8 - ngx_cycle->free_connection_n;
```

因此，在 nginx 启动时，ngx_accept_disabled 的值就是一个负数，其值为连接总数的 7/8。当该进程的连接数达到总连接数的 7/8 时，该进程就不会再处理新的连接了。

同时每次调用'ngx_process_events_and_timers'时，将 ngx_accept_disabled 减 1，直到其值低于阈值时，才试图重新处理新的连接。

因此，nginx 各 worker 子进程间的负载均衡仅在某个 worker 进程处理的连接数达到它最大处理总数的 7/8 时才会触发，其负载均衡并不是在任意条件都满足。如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95ZygTJHpRA2GHaWdgPjlkJibsgML0K1wOXuLzlTyOwx9eo4y9gusjrL4uzXHia0H86zZr4fowHp9Ew/640?wx_fmt=png)

其中'pid'为 1211 的进程为 master 进程，其余为 worker 进程。

**四、思考**

**1. 为什么不采用多线程模型管理连接？**

*   无状态服务，无需共享进程内存
    
*   采用独立的进程，可以让互相之间不会影响。一个进程异常崩溃，其他进程的服务不会中断，提升了架构的可靠性。
    
*   进程之间不共享资源，不需要加锁，所以省掉了锁带来的开销。
    

**2. 为什么不采用多线程处理逻辑业务？**

*   进程数已经等于核心数，再新建线程处理任务，只会抢占现有进程，增加切换代价。
    
*   作为接入层，基本上都是数据转发业务，网络 IO 任务的等待耗时部分，已经被处理为非阻塞 / 全异步 / 事件驱动模式，在没有更多 CPU 的情况下，再利用多线程处理，意义不大。并且如果进程中有阻塞的处理逻辑，应该由各个业务进行解决，比如 openResty 中利用了 Lua 协程，对阻塞业务进行了优化。
    

- EOF -

推荐阅读  点击标题可跳转

1、[万字整理，肝翻 Linux 内存管理所有知识点](http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651162932&idx=2&sn=4f87d39ddaee8cb29f66b820cd761445&chksm=8064586bb713d17d4c2521e79c9493f0340015b9c0efdd08d3eb684c9ce6a0c9e23e5161a8fd&scene=21#wechat_redirect)

2、[如何写一个简单的 node.js C++ 扩展](http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651160210&idx=2&sn=c8392fc54f30b3024c7770e2cd28569b&chksm=8064afcdb71326db93a45b469f1a6bb2d4cd8b59d762549ddade0326b3ebac0e6c29025b92ea&scene=21#wechat_redirect)

3、[25 张图，一万字，拆解 Linux 网络包发送过程](http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651162924&idx=1&sn=7c0c3887c9d55bb483c6874d5f3b6ac8&chksm=80645873b713d165e28b4d1b44d3a1aff8a957e8883692d35205a896df960821f7c78ffc2242&scene=21#wechat_redirect)
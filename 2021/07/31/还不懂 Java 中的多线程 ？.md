> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488085&idx=1&sn=66d41311ea6c4eeb4418a007e5d67b27&chksm=ce0da5d6f97a2cc0e4ea693f403adbd11614eedc9fbb2e3e7e6c51c3b9d03c60e8844ec6eb9d&scene=21#wechat_redirect)

作者 | 纳达丶无忌
----------

原文 | jianshu.com/p/40d4c7aebd66  

  

---

**前言**  

由于此订阅号换了个皮肤，导致部分用户接受文章不及时。可以打开本订阅号，选择置顶（标星）公众号，重磅干货，第一时间送达！

  

**正文**

如果对什么是线程、什么是进程仍存有疑惑，请先 Google 之，因为这两个概念不在本文的范围之内。

用多线程只有一个目的，那就是更好的利用 CPU 的资源，因为所有的多线程代码都可以用单线程来实现。说这个话其实只有一半对，因为反应 “多角色” 的程序代码，最起码每个角色要给他一个线程吧，否则连实际场景都无法模拟，当然也没法说能用单线程来实现：比如最常见的“生产者，消费者模型”。

很多人都对其中的一些概念不够明确，如同步、并发等等，让我们先建立一个数据字典，以免产生误会。

*   多线程：指的是这个程序（一个进程）运行时产生了不止一个线程
    
*   并行与并发：
    
*   并行：多个 CPU 实例或者多台机器同时执行一段处理逻辑，是真正的同时。
    
*   并发：通过 CPU 调度算法，让用户看上去同时执行，实际上从 CPU 操作层面不是真正的同时。并发往往在场景中有公用的资源，那么针对这个公用的资源往往产生瓶颈，我们会用 TPS 或者 QPS 来反应这个系统的处理能力。
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/3xNTnlLnTpVeLDS17u8unARuhqgokib9QutubQFrGjdr7HVMs4h5I9Eb9eIMGzLbGVvrQVejicFYyrnQZjNTLetg/640?wx_fmt=png)

并发与并行

  

*   线程安全：经常用来描绘一段代码。指在并发的情况之下，该代码经过多线程使用，线程的调度顺序不影响任何结果。这个时候使用多线程，我们只需要关注系统的内存，CPU 是不是够用即可。反过来，线程不安全就意味着线程的调度顺序会影响最终结果，如不加事务的转账代码:
    

```
void transferMoney(User from, User to, float amount){
    to.setMoney(to.getBalance() + amount);
    from.setMoney(from.getBalance() - amount);
}
```

*   同步：Java 中的同步指的是通过人为的控制和调度，保证共享资源的多线程访问成为线程安全，来保证结果的准确。如上面的代码简单加入 @synchronized 关键字。在保证结果准确的同时，提高性能，才是优秀的程序。线程安全的优先级高于性能。
    
  

好了，让我们开始吧。我准备分成几部分来总结涉及到多线程的内容：

1. 扎好马步：线程的状态

2. 内功心法：每个对象都有的方法（机制）

3. 太祖长拳：基本线程类

4. 九阴真经：高级多线程控制类

**扎好马步：线程的状态**

先来两张图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/3xNTnlLnTpVeLDS17u8unARuhqgokib9QpLUATOxM4BPqm4tWWKpRX5H2TewjxibC0O1dILZgLc8icgtXtsKLBE0Q/640?wx_fmt=png)

线程状态

![图片](https://mmbiz.qpic.cn/mmbiz_png/3xNTnlLnTpVeLDS17u8unARuhqgokib9Qxh9P8va0bR1sUm77r8hQwBFCQbJkOf6tvZy8rJL3qOxX0jPVu71uTA/640?wx_fmt=png)

线程状态转换

各种状态一目了然，值得一提的是 "Blocked" 和 "Waiting" 这两个状态的区别：  

*   线程在 Running 的过程中可能会遇到阻塞 (Blocked) 情况 
    
    对 Running 状态的线程加同步锁 (Synchronized) 使其进入 (lock blocked pool), 同步锁被释放进入可运行状 (Runnable)。从 jdk 源码注释来看，blocked 指的是对 monitor 的等待（可以参考下文的图）即该线程位于等待区。  
    

*   线程在 Running 的过程中可能会遇到等待（Waiting）情况  
    线程可以主动调用 object.wait 或者 sleep，或者 join（join 内部调用的是 sleep ，所以可看成 sleep 的一种）进入。从 jdk 源码注释来看，Waiting 是等待另一个线程完成某一个操作，如 join 等待另一个完成执行，object.wait() 等待 object.notify() 方法执行。
    

**Waiting 状态**和 **Blocked 状态**有点费解，我个人的理解是：Blocked 其实也是一种 wait ，等待的是 monitor ，但是和 Waiting 状态不一样，举个例子，有三个线程进入了同步块，其中两个调用了 object.wait()，进入了 Waiting 状态，这时第三个调用了 object.notifyAll() ，这时候前两个线程就一个转移到了 Runnable, 一个转移到了 Blocked。

从下文的 monitor 结构图来区别：每个 Monitor 在某个时刻，只能被一个线程拥有，该线程就是  “Active Thread”，而其它线程都是  “Waiting Thread”，分别在两个队列  “ Entry Set” 和  “Wait Set” 里面等候。在 “Entry Set” 中等待的线程状态 Blocked, 从 jstack 的 dump 中来看是  “Waiting for monitor entry”，而在 “Wait Set” 中等待的线程状态是 Waiting，表现在 jstack 的 dump 中是  “in Object.wait()”。

此外，在 runnable 状态的线程是处于被调度的线程，此时的调度顺序是不一定的。Thread 类中的 yield 方法可以让一个 running 状态的线程转入 runnable。

**内功心法：每个对象都有的方法（机制）**

synchronized, wait, notify 是任何对象都具有的同步工具。让我们先来了解他们

![图片](https://mmbiz.qpic.cn/mmbiz_png/3xNTnlLnTpVeLDS17u8unARuhqgokib9QWicAQhu02xpZTBgGhxbXdllTmapTBAy0mkEe5rIPQknXx7hFVCFUInQ/640?wx_fmt=png)

monitor

他们是应用于同步问题的人工线程调度工具。讲其本质，首先就要明确 monitor 的概念，Java 中的每个对象都有一个监视器，来监测并发代码的重入。在非多线程编码时该监视器不发挥作用，反之如果在 synchronized 范围内，监视器发挥作用。

wait/notify 必须存在于 synchronized 块中。并且，这三个关键字针对的是同一个监视器（某对象的监视器）。这意味着 wait 之后，其他线程可以进入同步块执行。

当某代码并不持有监视器的使用权时（如图中 5 的状态，即脱离同步块）去 wait 或 notify，会抛出 java.lang.IllegalMonitorStateException。

也包括在 synchronized 块中去调用另一个对象的 wait/notify，因为不同对象的监视器不同，同样会抛出此异常。

再讲用法：

*   synchronized 单独使用：
    
*   代码块：如下，在多线程环境下，synchronized 块中的方法获取了 lock 实例的 monitor，如果实例相同，那么只有一个线程能执行该块内容
    

*   直接用于方法：相当于上面代码中用 lock 来锁定的效果，实际获取的是 Thread1 类的 monitor。更进一步，如果修饰的是 static 方法，则锁定该类所有实例
    

*   synchronized, wait, notify 结合: 典型场景生产者消费者问题
    

**volatile**

多线程的内存模型：main memory（主存）、working memory（线程栈），在处理数据时，线程会把值从主存 load 到本地栈，完成操作后再 save 回去 (volatile 关键词的作用：每次针对该变量的操作都激发一次 load and save) 。

![图片](https://mmbiz.qpic.cn/mmbiz_png/3xNTnlLnTpVeLDS17u8unARuhqgokib9Qn2UvCtEyhZJIqib926YVqORmxSznPCBFmbK6YLhdA8N25E8dwGR8vSw/640?wx_fmt=png)

volatile

针对多线程使用的变量如果不是 volatile 或者 final 修饰的，很有可能产生不可预知的结果（另一个线程修改了这个值，但是之后在某线程看到的是修改之前的值）。其实道理上讲同一实例的同一属性本身只有一个副本。但是多线程是会缓存值的，本质上，volatile 就是不去缓存，直接取值。在线程安全的情况下加 volatile 会牺牲性能。

**太祖长拳：基本线程类**

基本线程类指的是 Thread 类，Runnable 接口，Callable 接口  

Thread 类实现了 Runnable 接口，启动一个线程的方法：

**Thread 类相关方法**

**关于中断**：它并不像 stop 方法那样会中断一个正在运行的线程。线程会不时地检测中断标识位，以判断线程是否应该被中断（中断标识值是否为 true ）。终端只会影响到 wait 状态、sleep 状态和 join 状态。被打断的线程会抛出 InterruptedException。  
Thread.interrupted() 检查当前线程是否发生中断，返回 boolean

synchronized 在获锁的过程中是不能被中断的。

中断是一个状态！interrupt() 方法只是将这个状态置为 true 而已。所以说正常运行的程序不去检测状态，就不会终止，而 wait 等阻塞方法会去检查并抛出异常。如果在正常运行的程序中添加 while(!Thread.interrupted()) ，则同样可以在中断后离开代码体

**Thread 类最佳实践：**

写的时候最好要设置线程名称  Thread.name，并设置线程组 ThreadGroup，目的是方便管理。在出现问题的时候，打印线程栈 (jstack -pid)  一眼就可以看出是哪个线程出的问题，这个线程是干什么的。

**如何获取线程中的异常**

![图片](https://mmbiz.qpic.cn/mmbiz_png/3xNTnlLnTpVeLDS17u8unARuhqgokib9QvHkibjdicgyFicbxUfO48sxT8BNHa5ic13UKTpSVAdib2a43q88dGGFgRqg/640?wx_fmt=png)

不能用 try,catch 来获取线程中的异常

**Runnable**

与 Thread 类似

**Callable**

future 模式：并发模式的一种，可以有两种形式，即无阻塞和阻塞，分别是 isDone 和 get。其中 Future 对象用来存放该线程的返回值以及状态

**九阴真经：高级多线程控制类**

以上都属于内功心法，接下来是实际项目中常用到的工具了，Java1.5 提供了一个非常高效实用的多线程包: _java.util.concurrent_, 提供了大量高级工具, 可以帮助开发者编写高效、易维护、结构清晰的 Java 多线程程序。

**1.ThreadLocal 类**

用处：保存线程的独立变量。对一个线程类（继承自 Thread )  
当使用 ThreadLocal 维护变量时，ThreadLocal 为每个使用该变量的线程提供独立的变量副本，所以每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本。常用于用户登录控制，如记录 session 信息。

实现：每个 Thread 都持有一个 TreadLocalMap 类型的变量（该类是一个轻量级的 Map，功能与 map 一样，区别是桶里放的是 entry 而不是 entry 的链表。功能还是一个 map 。）以本身为 key，以目标为 value。  
主要方法是 get() 和 set(T a)，set 之后在 map 里维护一个 threadLocal -> a，get 时将 a 返回。ThreadLocal 是一个特殊的容器。

**2. 原子类（AtomicInteger、AtomicBoolean……）**

如果使用 atomic wrapper class 如 atomicInteger，或者使用自己保证原子的操作，则等同于 synchronized

该方法可用于实现乐观锁，考虑文中最初提到的如下场景：a 给 b 付款 10 元，a 扣了 10 元，b 要加 10 元。此时 c 给 b 2 元，但是 b 的加十元代码约为：

```
if(b.value.compareAndSet(old, value)){  
    return ;
}else{
        //try again
        // if that fails, rollback and log
}
```

**AtomicReference**

对于 AtomicReference 来讲，也许对象会出现，属性丢失的情况，即 oldObject == current，但是 oldObject.getPropertyA != current.getPropertyA。  
这时候，AtomicStampedReference 就派上用场了。这也是一个很常用的思路，即加上版本号

**3.Lock 类**

lock: 在 java.util.concurrent 包内。共有三个实现：

*   ReentrantLock
    
*   ReentrantReadWriteLock.ReadLock
    
*   ReentrantReadWriteLock.WriteLock
    

主要目的是和 synchronized 一样， 两者都是为了解决同步问题，处理资源争端而产生的技术。功能类似但有一些区别。

区别如下：

1.lock 更灵活，可以自由定义多把锁的枷锁解锁顺（synchronized 要按照先加的后解顺序）

2. 提供多种加锁方案，lock 阻塞式, trylock 无阻塞式,    lockInterruptily 可打断式， 还有 trylock 的带超时时间版本

3. 本质上和监视器锁（即 synchronized 是一样的）

4. 能力越大，责任越大，必须控制好加锁和解锁，否则会导致灾难。

5. 和 Condition 类的结合。

6. 性能更高，对比如下图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/3xNTnlLnTpVeLDS17u8unARuhqgokib9QgCJibicIrl5PzVgW7MRoUSomX9CaS9UricmCwAIEFCubwtjaahnbBHowA/640?wx_fmt=png)

synchronized 和 Lock 性能对比

**ReentrantLock**

可重入的意义在于持有锁的线程可以继续持有，并且要释放对等的次数后才真正释放该锁。  

使用方法是：

1. 先 new 一个实例

```
static ReentrantLock r=new ReentrantLock();
```

2. 加锁

```
r.lock()或 r.lockInterruptibly();
```

此处也是个不同，后者可被打断。当 a 线程 lock 后，b 线程阻塞，此时如果是 lockInterruptibly，那么在调用 b.interrupt() 之后，b 线程退出阻塞，并放弃对资源的争抢，进入 catch 块。（如果使用后者，必须 throw interruptable exception 或 catch）

3. 释放锁

```
r.unlock()
```

必须做！何为必须做呢，要放在 finally 里面。以防止异常跳出了正常流程，导致灾难。这里补充一个小知识点，finally 是可以信任的：经过测试，哪怕是发生了 OutofMemoryError ，finally 块中的语句执行也能够得到保证。

**ReentrantReadWriteLock**

可重入读写锁（读写锁的一个实现）

两者都有 lock,unlock 方法。写写，写读互斥；读读不互斥。可以实现并发读的高效线程安全代码

**4. 容器类**

这里就讨论比较常用的两个：

*   BlockingQueue
    
*   ConcurrentHashMap
    

**BlockingQueue**

阻塞队列。该类是 java.util.concurrent 包下的重要类，通过对 Queue 的学习可以得知，这个 queue 是单向队列，可以在队列头添加元素和在队尾删除或取出元素。类似于一个管道，特别适用于先进先出策略的一些应用场景。普通的 queue 接口主要实现有 PriorityQueue（优先队列），有兴趣可以研究

BlockingQueue 在队列的基础上添加了多线程协作的功能：

![图片](https://mmbiz.qpic.cn/mmbiz_png/3xNTnlLnTpVeLDS17u8unARuhqgokib9QjFwibwAiaGMTHm4KQCQUZ0Mwg7Ssiae4ib47qtVhXG6VfSPHG8Aq13icJJQ/640?wx_fmt=png)

BlockingQueue

除了传统的 queue 功能（表格左边的两列）之外，还提供了阻塞接口 put 和 take，带超时功能的阻塞接口 offer 和 poll。put 会在队列满的时候阻塞，直到有空间时被唤醒；take 在队　列空的时候阻塞，直到有东西拿的时候才被唤醒。用于生产者 - 消费者模型尤其好用，堪称神器。

常见的阻塞队列有：

*   ArrayListBlockingQueue
    
*   LinkedListBlockingQueue
    
*   DelayQueue
    
*   SynchronousQueue
    

** ConcurrentHashMap**  

高效的线程安全哈希 map。请对比 hashTable , concurrentHashMap, HashMap

**5. 管理类**

管理类的概念比较泛，用于管理线程，本身不是多线程的，但提供了一些机制来利用上述的工具做一些封装。  

了解到的值得一提的管理类：ThreadPoolExecutor 和 JMX 框架下的系统级管理类 ThreadMXBean

**ThreadPoolExecutor  
**

如果不了解这个类，应该了解前面提到的 ExecutorService，开一个自己的线程池非常方便

该类内部是通过 ThreadPoolExecutor 实现的，掌握该类有助于理解线程池的管理，本质上，他们都是 ThreadPoolExecutor 类的各种实现版本。请参见 javadoc：

![图片](https://mmbiz.qpic.cn/mmbiz_png/3xNTnlLnTpVeLDS17u8unARuhqgokib9QvCSwkOCRh73f739JvHKKXn02sHmlWOaDCUlkc5FpHEibPS1d29FBKKg/640?wx_fmt=png)

ThreadPoolExecutor 参数解释

翻译一下：  

**corePoolSize**: 池内线程初始值与最小值，就算是空闲状态，也会保持该数量线程。  

**maximumPoolSize**: 线程最大值，线程的增长始终不会超过该值。  

**keepAliveTime**: 当池内线程数高于 corePoolSize 时，经过多少时间多余的空闲线程才会被回收。回收前处于 wait 状态  

**unit**：  
时间单位，可以使用 TimeUnit 的实例，如 TimeUnit.MILLISECONDS   
**workQueue**: 待入任务（Runnable）的等待场所，该参数主要影响调度策略，如公平与否，是否产生饿死 (starving)  

**threadFactory**: 线程工厂类，有默认实现，如果有自定义的需要则需要自己实现 ThreadFactory 接口并作为参数传入。

请注意：该类十分常用，作者 80% 的多线程问题靠他。

**推****荐****阅****读**

_**1.**_ [依赖冲突了如何解决？](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488050&idx=1&sn=ff7a6dbc138c8f397f09cf1917d573b7&chksm=ce0da5b1f97a2ca79be51c6d8a860824fe845e9c463c45e2e001becb17137625d05cd2f18a5f&scene=21#wechat_redirect)

_**2.**_ [一文读懂分布式 Session 和几种常见的方案](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488017&idx=1&sn=60ee275eca3db93791205e8a77eac73d&chksm=ce0da592f97a2c84172f1d75cd36989ce4db08889a21794c4b31bb0ed27e92b0145483f62025&scene=21#wechat_redirect)

_**3.**_ [GitHub 上的小徽章从何而来?](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488012&idx=1&sn=859ac2f243180119cf852a91c3238dd3&chksm=ce0da58ff97a2c9963f14d167011162b4e677653769a48da7df940a4b0de03176b507193e0e2&scene=21#wechat_redirect)

_**4.**_ [使用 Zeal 打造属于自己的文档](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488006&idx=1&sn=838b0e4892421244414408c157e270ce&chksm=ce0da585f97a2c93079d2283896096ef026643bb1ecd0fa60b4247cffc3aa36f894be2ed02b0&scene=21#wechat_redirect)
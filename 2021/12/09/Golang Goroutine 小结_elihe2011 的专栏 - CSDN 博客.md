> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/elihe2011/article/details/109138396)

### Golang Goroutine 小结

*   [1. CSP 并发模型](#1_CSP__2)
*   [2. 协程、线程、进程](#2__26)
*   [3. 并发的实现原理](#3__54)
*   *   [3.1 用户级线程模型](#31__66)
    *   [3.2 内核级线程模型](#32__80)
    *   [3.3 两级线程模型 (即混合型线程模型)](#33__94)
*   [4. G-P-M 模型](#4_GPM__108)
*   [5. Golang 并发控制模型](#5_Golang__129)
*   [6. gorountine 调度时机](#6_gorountine_137)
*   [7. goroutine 的切换点](#7_goroutine_147)
*   [8. 实例](#8__155)
*   *   [8.1 go 关键字开启新协程](#81_go_156)
    *   [8.2 runtime 包](#82_runtime_172)
*   [9. goroutine 泄露](#9_goroutine__247)

1. CSP 并发模型
===========

Communication Sequential Processes

Do not communicate by sharing memory; instead, share memory by communicating.

**不要通过共享内存来通信，而要通过通信实现内存共享**

Go 的 CSP 并发模型，通过 goroutine 和 channel 来实现：

*   goroutine: 并发的执行单位
    
    > Goroutine 是 Golang 并发的实体，它底层使用协程实现并发，coroutine 是一种**运行在用户态的用户线程**，类似 greenthread，具有如下特点：
    > 
    > *   用户空间，避免了内核态和用户态的切换导致的成本
    > *   可由语言和框架层进行调度
    > *   更小的栈空间允许创建大量的实例
    
*   channel: 并发的通信机制
    
    > channel 被单独创建，可以在进程之间传递，一个实体通过将消息发到 channel 中，然后又监听这个 channel 的实体处理，两个实体之间是匿名的，它实现了实体中间的解藕。
    

2. 协程、线程、进程
===========

*   进程：系统进行资源分配和调度的一个独立单位。**每个进程都有自己独立的内存空间，不同进程通过 IPC 通信**。进程上下文切换开销（栈、寄存器、虚拟内存、文件句柄）比较大，但相对比较稳定安全。
    
*   线程：处理器调度和分配的基本单位。**线程是进程内部的一个执行单元**，每个进程至少有一个主线程，它无需用户去主动创建，由系统自动创建。线程间通信主要通过共享内存、上下文切换较快，资源开销小，但相比进程不够稳定，容易丢数据
    
*   协程：**用户态轻量级线程，它的调度完全由用户控制**。协程拥有自己的寄存器上下文和栈。协程调度切换时，将寄存器上下文和栈保存到其他地方，在切回来的时候，恢复先前保存的寄存器上下文和栈。开销小，切换快。
    

进程、线程、协程的关系和区别：

> *   进程：拥有独立的堆和栈，由操作系统调度
> *   线程：拥有独立的栈，但共享堆。由操作系统调度（标准线程）
> *   协程：和线程一样，拥有独立的栈，共享堆。由程序开发者在代码中显示调度

为什么协程比线程轻量？

> 1.  内存消耗极小：
>     *   goroutine: 4-5KB, 如果栈内存不足，自动扩容
>     *   thread: 1M, 另外还需要一个 "a guard page" 区域用于与其他线程的栈空间进行隔离
> 2.  创建和销毁损耗资源小：
>     *   goroutine: 由 Go runtime 负责管理，创建和销毁的消耗非常小，是用户级别的。
>     *   thread: 是内核级的，创建和销毁都会有巨大的消耗，一般通过线程池来缓解
> 3.  切换快：
>     *   goroutine: 不依赖于系统，由 golang 自己实现的 CSP 并发模型实现：G-P-M。go 协程也叫用户态线程，协程的切换发生在用户态，**约为 200ns**
>     *   thread: 内核对外提供的服务，应用程序可以通过系统调用让内核启动线程。线程在等待 IO 操作时变成 unrunnable 状态触发上下文切换，**1000～1500ns**

3. 并发的实现原理
==========

KSE：Kernel Scheduling Entity, 内核调度实体，即可以被操作系统内核调度器调度的实体对象，它是内核的最小调度单元，也就是**内核级线程**

三种线程模型：

*   用户级线程模型
*   内核级线程模型
*   两级线程模型 (即混合型线程模型)

3.1 用户级线程模型
-----------

![](https://img-blog.csdnimg.cn/img_convert/7bd7546a05077b2447285bb0c5f7b4fb.png)

**多个用户态的线程对应一个内核线程，线程的创建、终止、切换或同步等工作必须自身来完成；**

优点：线程调度在用户层面完成，不存在 CPU 在用户态和内核态之间切换，轻量级，对系统资源消耗少

缺点：做不到真正意义上的并发。如果存在某个线程阻塞调用 (比如 I/O 操作)，其他线程将被阻塞，整个进程被挂起。因为在用户线程模式下，进程内的线程绑定到 CPU 是由用户进程调度实现的，内部线程对 CPU 不可见，即 CPU 调度的是进程，而非线程

Python 协程库 gevent，把阻塞的操作重新封装为完成给阻塞模式，在阻塞点上，主动让出自己，并通知或唤醒其他等待的用户线程。

3.2 内核级线程模型
-----------

![](https://img-blog.csdnimg.cn/img_convert/a332e7728d37d1b6c37cef204d3ea6f2.png)

**直接调用操作系统的内核线程，所有线程的创建、终止、切换、同步等操作，都由内核完成**

优点：简单。直接借助内核的线程和调度器，可以快速实现线程切换，做到真正的并发处理

缺点：直接使用内核区创建、销毁及线程上下文切换和调度，系统资源开销大，影响性能

Java/C++ 线程库

3.3 两级线程模型 (即混合型线程模型)
---------------------

![](https://img-blog.csdnimg.cn/img_convert/28023004abee637c594d479d4cdf6ce1.png)

**一个进程可与多个内核线程 KSE 关联，该进程内的多个线程绑定到了不同的 KSE 上**

进程内的线程并不与 KSE 一一绑定，当某个 KSE 绑定的线程因阻塞操作被内核调度出 CPU 时，其关联的进程中的某个线程又会重新与 KSE 绑定

为什么称为两级？**用户调度实现用户线程到 KSE 的调度，内核调度器实现 KSE 到 CPU 上的调度**

Golang

4. G-P-M 模型
===========

![](https://img-blog.csdnimg.cn/img_convert/9890bd42f8829722711980aa92ab2ce2.png)

G-P-M 模型：

*   G：Goroutine：独立执行单元。相较于每个 OS 线程固定分配 2M 内存的模式，Goroutine 的栈采取动态扩容方式，2k ~ 1G(AMD64, AMD32: 256M)。周期性回收内存，收缩栈空间
    *   每个 Goroutine 对应一个 G 结构体，它存储 Goroutine 的运行堆栈、状态及任务函数，可重用。
    *   G 并非执行体，每个 G 需要绑定到 P 才能被调度执行
*   P：Processor： 逻辑处理器，中介
    *   对 G 来说，P 相当于 CPU，G 只有绑定到 P 才能被调用
    *   对 M 来说，P 提供相关的运行环境 (Context)，如内存分配状态(mcache)，任务队列(G) 等
    *   P 的数量决定系统最大并行的 G 的数量 （CPU 核数 >= P 的数量），用户可通过 GOMAXPROCS 设置数量，但不能超过 256
*   M：Machine
    *   OS 线程抽象，真正执行计算的资源，在绑定有效的 P 后，进入 schedule 循环
    *   schedule 循环的机制大致从 Global 队列、P 的 Local 队列及 wait 队列中获取 G，切换到 G 的执行栈上执行 G 的函数，调用 goexit 做清理工作并回到 M
    *   M 不保留 G 的状态
    *   M 的数量不定，由 Go Runtime 调整，目前默认不超过 10K

5. Golang 并发控制模型
================

*   channel
*   sync.WaitGroup: `Add(), Done(), Wait()`
*   Context: `Done(), Err(), Deadline(), Value()` 较复杂的并发

6. gorountine 调度时机
==================

<table><thead><tr><th align="center">情形</th><th align="left" class="">说明</th></tr></thead><tbody><tr><td align="center">go</td><td align="left">go 创建一个新的 goroutine，Go scheduler 会考虑调度</td></tr><tr><td align="center">GC</td><td align="left">由于进行 GC 的 goroutine 也需要在 M 上运行，因此肯定会发生调度。当然，Go scheduler 还会做很多其他的调度，例如调度不涉及堆访问的 goroutine 来运行。GC 不管栈上的内存，只会回收堆上的内存</td></tr><tr><td align="center">系统调用</td><td align="left">当 goroutine 进行系统调用时，会阻塞 M，所以它会被调度走，同时一个新的 goroutine 会被调度上来</td></tr><tr><td align="center">内存同步访问</td><td align="left">atomic，mutex，channel 操作等会使 goroutine 阻塞，因此会被调度走。等条件满足后（例如其他 goroutine 解锁了）还会被调度上来继续运行</td></tr></tbody></table>

7. goroutine 的切换点
=================

*   I/O, select
*   channel
*   等待锁
*   函数调用 (有时)
*   runtime.Gosched()

8. 实例
=====

8.1 go 关键字开启新协程
---------------

```
func say(s string) {
	for i := 0; i < 5; i ++ {
		time.Sleep(100 * time.Millisecond)
		fmt.Println(s)
	}
}

func main() {
	go say("world")
	say("hello")
}

```

8.2 runtime 包
-------------

`runtime.Gosched()` 让出时间片  
`runtime.Goexit()` 终止协程  
`runtime.GOMAXPROCS(N)` 指定运行 CPU 个数

```
func main() {
	go func() {
		for i := 0; i < 5; i++ {
			fmt.Println("go")
		}
	}()

	for i := 0; i < 2; i++ {
		runtime.Gosched() // 让出时间片
		fmt.Println("hello")
	}
}

```

```
// 打印函数属IO操作，自动切换控制权
func auto() {
	for i := 0; i < 10; i++ {
		go func(i int) {
			for {
				fmt.Printf("Hello from goroutine %d\n", i)
			}
		}(i)
	}

	time.Sleep(time.Millisecond)
}

// 不自动切换控制权
func manual() {
	var a [10]int

	for i := 0; i < 10; i++ {
		go func(i int) { // race condition
			for {
				a[i]++
				runtime.Gosched() // 交出控制权
			}
		}(i)
	}

	time.Sleep(time.Millisecond)
	fmt.Println(a)  // 存在读写抢占
}

// out of range
func outOfRange() {
	var a [10]int

	for i := 0; i < 10; i++ {
		go func() { // race condition
			for {
				a[i]++
				runtime.Gosched() // 交出控制权
			}
		}()
	}

	time.Sleep(time.Millisecond)
	fmt.Println(a)
}

```

```
go run -race goroutine.go   # manual()函数存在抢占，race选项可检查到

```

9. goroutine 泄露
===============

检测 goroutine 泄漏：使用 runtime.Stack() 在测试代码前后计算 goroutine 的数量，代码运行完毕会触发 gc，如果触发 gc 后，发现还有 goroutine 未被回收，那么这个 goroutine 很可能是被泄漏的

打印堆栈：

> *   当前堆栈
>     
>     ```
>     log.Info("stack %s", debug.Stack())
>     
>     ```
>     
> *   全局堆栈
>     
>     ```
>     buf := make([]byte, 1<<16)
>     runtime.Stack(buf, true)
>     log.Info("stack %s", buf)
>     
>     ```
>     

goroutine 泄漏：一个程序不断地产生新的 goroutine，且又不结束它们，会造成泄漏

```
func main() {
	for i := 0; i < 10000; i++ {
		go func() {
			select {}
		}()
	}
}

```
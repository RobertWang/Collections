> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/yNeQ5-OVTDk1Pm_kN-uJAA)

> ❝
> 
> 最近一直在忙着想法的多赚钱，组织各种活动（学习打卡、徒步旅行、收集资料等），没怎么写文章，先发一篇之前的学习笔记。
> 
> ❞

### asyncio 的简单介绍

在介绍之前先来普及两个概念：

**「异步 IO：」**就是发起一个 IO 操作（如：网络请求，文件读写等），这些操作一般是比较耗时的，不用等待它结束，可以继续做其他事情，结束时会发来通知。

**「协程：」**又称为微线程，在一个线程中执行，执行函数时可以随时中断，由程序（用户）自身控制，执行效率极高，与多线程比较，没有切换线程的开销和多线程锁机制。

python 中异步 IO 操作是通过 asyncio 来实现的。

**「官方对 asyncio 的介绍：」**

> ❝
> 
> asyncio is a library to write **「concurrent」** code using the **「async/await」** syntax.
> 
> asyncio is used as a foundation for multiple Python asynchronous frameworks that provide high-performance network and web-servers, database connection libraries, distributed task queues, etc.
> 
> asyncio 是一个使用**「async/await」**语法来编写**「并发」**代码的库。
> 
> asyncio 被用作多个 Python 异步框架的基础，这些框架提供高性能的网络和 Web 服务器、数据库连接库、分布式任务队列等。
> 
> ❞

### asyncio 应用场景

在程序在执行 IO 密集型任务的时候，程序会因为等待 IO 而阻塞。协程遇到 io 操作而阻塞时，立即切换到别的任务，如果操作完成则进行回调返回执行结果。

### asyncio 中几个重要概念

**「event_loop 事件循环」**

> ❝
> 
> 程序开启一个无限循环，把一些函数注册到事件循环上，当满足事件发生的时候，调用相应的协程函数。管理所有的事件，在整个程序运行过程中不断循环执行并追踪事件发生的顺序将它们放在队列中，空闲时调用相应的事件处理者来处理这些事件。
> 
> ❞

**「coroutine」**

> ❝
> 
> 协程：协程对象，指一个使用 async 关键字定义的函数，它的调用不会立即执行函数，而是会返回一个协程对象。协程对象需要注册到事件循环，由事件循环调用。
> 
> ❞

**「future」**

> ❝
> 
> Future 对象表示尚未完成的计算，还未完成的结果。
> 
> **「在 asyncio 中，如何才能得到异步调用的结果呢？先设计一个对象，异步调用执行完的时候，就把结果放在它里面，这种对象称之为未来对象。未来对象有一个 result 方法，可以获取未来对象的内部结果。还有个 set_result 方法，是用于设置 result 的。set_result 设置的是什么，调用 result 得到的就是什么。Future 对象可以看作下面的 Task 对象的容器。」**
> 
> ❞

**「task」**

> ❝
> 
> 任务是 Future 的子类，作用是在运行某个任务的同时可以并发运行多个任务。一个协程对象就是一个原生可以挂起的函数，任务则是对协程进一步封装，其中包含了任务的各种状态。
> 
> ❞

**「async/await 关键字」**

> ❝
> 
> python3.5 用于定义协程的关键字，async 定义一个协程，await 用于挂起阻塞的异步调用接口。
> 
> 使用 async 可以定义协程对象，使用 await 可以针对耗时的操作进行挂起，就像生成器里的 yield 一样，函数让出控制权。
> 
> 协程遇到 await，事件循环将会挂起该协程，执行别的协程，直到其他的协程也挂起或者执行完毕，再进行下一个协程的执行。
> 
> ❞

**「run_until_complete()」**

> ❝
> 
> 阻塞调用，直到协程运行结束才返回。参数是 future，传入协程对象时内部会自动变为 future。
> 
> ❞

**「asyncio.sleep()」**

> ❝
> 
> 模拟 IO 操作，这样的休眠不会阻塞事件循环，前面加上 await 后会把控制权交给主事件循环，在休眠（IO 操作）结束后恢复这个协程。
> 
> 提示：若在协程中需要有延时操作，应该使用 await asyncio.sleep()，而不是使用 time.sleep()，因为使用 time.sleep() 后会释放 GIL，阻塞整个主线程，从而阻塞整个事件循环。
> 
> ❞

### asyncio 基本使用

#### 一个简单的异步的例子

```
#  ====一个简单的异步的例子====
# 我们通过async关键字定义一个协程,当然协程不能直接运行，需要将协程加入到事件循环loop中
async def coroutine_example():
    print(time.localtime())
    await  asyncio.sleep(2)
    print("===========")
    print(time.localtime())

coroutine_example1= coroutine_example()
loop= asyncio.get_event_loop()  # 事件循环：asyncio.get_event_loop：创建一个事件循环
loop.run_until_complete(coroutine_example1)
loop.close()

# time.struct_time(tm_year=2023, tm_mon=4, tm_mday=30, tm_hour=21, tm_min=58, tm_sec=15, tm_wday=6, tm_yday=120, tm_isdst=0)
# ===========
# time.struct_time(tm_year=2023, tm_mon=4, tm_mday=30, tm_hour=21, tm_min=58, tm_sec=17, tm_wday=6, tm_yday=120, tm_isdst=0)


```

```
import asyncio
import time

# 我们通过async关键字定义一个协程,当然协程不能直接运行，需要将协程加入到事件循环loop中
async def do_some_work(x):
    print("waiting:", x)

start = time.time()

coroutine = do_some_work(2)
loop = asyncio.get_event_loop()        # asyncio.get_event_loop：创建一个事件循环
# 通过loop.create_task(coroutine)创建task,同样的可以通过 asyncio.ensure_future(coroutine)创建task
task = loop.create_task(coroutine)     # 创建任务, 不立即执行
loop.run_until_complete(task)         # 使用run_until_complete将协程注册到事件循环，并启动事件循环
print("Time:",time.time() - start)

# waiting: 2
# Time: 0.0009980201721191406

# asyncio.get_event_loop：创建一个事件循环，然后使用run_until_complete将协程注册到事件循环，并启动事件循环
# 协程对象不能直接运行，在注册事件循环的时候，其实是run_until_complete方法将协程包装成为了一个任务（task）对象.
# task对象是Future类的子类，保存了协程运行后的状态，用于未来获取协程的结果


```

看到上面的例子，你也许和我一样会很蒙，不晓得为什么是这样一个流程（官方文档比较苦涩，很难读下去）。没关系，多敲几遍，读大量的文档辅助学习吧。

学习这个 asyncio 的时候，我就一直在想，为什么普遍的教程都是这样讲的，他们是怎么学习一个新的知识的呢。因为刚好用到？因为初期的 asyncio 比较简单，经过演进之后学习曲线平滑？不得而知，我现在能做的就是没事多翻翻相关资料，多跟着敲敲，慢慢的体会。

#### 一个更简单的异步的例子

官网给出的例子，大家可以把代码粘贴进去和前面的比较一下。

```
#  ====一个更简单的异步的例子====

import  asyncio
async  def main():
    print("hello")
    await asyncio.sleep(2)
    print("world")

# asyncio.run(main)    # 错误写法
asyncio.run(main())


```

### 参考资料

<table><thead><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); font-size: 14px; min-width: 85px;">序号</th><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); font-size: 14px; min-width: 85px;" class="">文章链接</th><th width="49" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); font-size: 14px; min-width: 85px;">备注</th></tr></thead><tbody><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">1</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">https://blog.csdn.net/longlong6682/article/details/106501654</td><td width="209.33333333333334" data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px; word-break: break-all;">asyncio 简介与应用场景（基本使用）</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">2</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">https://docs.python.org/3/library/asyncio.html</td><td width="49" data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">官方文档</td></tr></tbody></table>
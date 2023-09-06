> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/WYGZIBlTdbBpc42RCliBeA)

> 多一句没有，少一句不行，用最短的时间，教会你有用的技术。

**「今天分享一点异步的资料。」**

用到的时候再来深入学习，建议根据这里提供的资源，先捡能学的懂的部分看，看完之后建立一个大致的印象，以后遇到项目实践再回过头来翻看就行了。学着学着，我发现好多东西太深了，容易迷失自己真正想要的。现在就算搞明白了，过几天又不懂了，就太浪费时间了，因为真正想搞懂还是结合着实践才行。

### 参考资料 - 1

> 学习的时候，跟着文章做了些笔记。

### Python 的 Sync 与 Async 执行速度的快慢

> https://juejin.cn/post/7026648216829427749

线程的运行都是靠 cpu 来执行的， 而 cpu 是有限的， 同一时刻只能支持固定的几个 worker 运行, 其他线程则得等待被调度, 这样就意味着每个线程都只能工作一个时间分片， 之后就会被调度系统控制进入阻塞或者就绪阶段， 让位给其他线程， 直到下次获取时间分片时才可以继续运行。为了能模拟出同一时刻内， 多个线程同时运行， 且防止其他线程饿死的情况， 线程每次获得的运行时间很短， 线程间的调度切换很频繁， 当启用更多的进程和更多的线程时， 调度就会更加的频繁。

不过调度线程的开销还不算大， 比较大的开销是调度线程而产生的下文切换和竞争条件 (具体可以参考《计算机导论》中进程调度相关的资料， 我这里只是简单说明)， cpu 在执行代码时，它需要把数据加载到 cpu 的缓存中去的再运行， 当 cpu 运行的线程在这个时间分片内执行完成时， 该线程的最新运行数据就会保存起来， 然后 cpu 会去加载准备被调度的线程的数据， 并运行。虽然这部分暂存数据是保存在比内存更快， 比内存更靠近 cpu 的寄存器上， 但是寄存器的访问速度也没有 cpu 缓存的访问速度快， 所以 cpu 在切换运行的线程时， 都会花上一部分时间用来装载数据上还有装载缓存时的竞争问题。

对比线程的调度产生的上下文切换与抢占式， `async`语法实现的协程是非抢占式的， 协程的调度是依赖于一个循环来控制， 这个循环是一个非常常高效的任务管理器和调度器, 由于调度的是一段代码的实现逻辑， 所以 cpu 的执行代码并不用切换， 也就没有上下文切换的开销, 同时， 也不用考虑装载缓存的竞争问题。

### 初识 Python 协程的实现

本文作者还演示了如何使用生成器实现一个协程的代码，蛮难的，多看几遍，跟着多敲敲体会体会。

> https://juejin.cn/post/7028534180023631908

```
def demo():
     a=1
     b=2
     print("aaa",locals())
     yield 1
     print("bbb",locals())
     yield 2
     return "None"



```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/h988a0nsgw6ItIjcM4aEPKXdafNdMhCicrnpYFWW8799k3zds7D62QMDT7Enadtg6vv8HC52CMiaKohSYV2EZ07w/640?wx_fmt=png)

这段代码首先通过函数调用生成一个`demo_gen`的生成器对象, 然后第一次`send`调用时返回值 1， 第二次`send`调用时返回值 2， 第三次`send`调用则抛出`StopIteration`异常， 异常提示为`None`, 同时可以看到第一次打印`aaa`和第二次打印`bbb`时， 他们都能打印到当前的函数局部变量， 可以发现在即使在不同的栈帧中， 他们读取到当前的局部函数内的局部变量是一致的， 这意味着如果使用生成器来模拟协程时， 它还是会一直读取到当前上下文的.

### Python 的可等待对象在 Asyncio 的作用

> https://juejin.cn/post/7085256098885681159

直接`await`Coroutine 对象时，这段程序会一直等待，直到 Coroutine 对象执行完毕再继续往下走，而 Task 对象的不同之处就是在创建的那一刻，就已经把自己注册到事件循环之中等待被安排运行了，然后返回一个 task 对象供开发者等待，由于`asyncio.sleep`是一个纯 IO 类型的调用，所以在这个程序中，两个`asyncio.sleep`Coroutine 被转为 Task 从而实现了并发调用。

与 Coroutine 只有让步和接收结果不同的是 Future 除了让步和接收结果功能外，它还是一个只会被动进行事件调用且带有状态的容器，它在初始化时就是`Pending`状态，这时可以被取消，被设置结果和设置异常。而在被设定对应的操作后，Future 会被转化到一个不可逆的对应状态，并通过`loop.call_sonn`来调用所有注册到本身上的回调函数，同时它带有`__iter__`和`__await__`方法使其可以被`await`和`yield from`调用。

Task 是 Future 的子类，除了继承了 Future 的所有方法，它还多了两个重要的方法`__step`和`__wakeup`，通过这两个方法赋予了 Task 调度能力，这是 Coroutine 和 Future 没有的。

**「== 剩下的大部分是源码解读，太烧脑了，以后再来，能跟着敲的跟着敲一遍。==」**

> https://juejin.cn/column/7026646119589347336

Python Asyncio 调度原理，Python 3.11 Asyncio 新增的两个高级类，Python Asyncio 库之常用函数详解

### 参考资料 - 2

不管三七二十一，看的脑壳疼，先记录于此吧。

<table><thead><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); font-size: 14px; min-width: 85px;">序号</th><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); font-size: 14px; min-width: 85px;" class="">文章链接</th><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); font-size: 14px; min-width: 85px;" class="">备注</th></tr></thead><tbody><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">1</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;" class="">https://www.cnblogs.com/traditional/p/11828780.html</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">asyncio：python3 未来并发编程主流、充满野心的模块</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">2</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">https://www.jianshu.com/p/29ffdbd65679</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">Python 黑魔法 asyncio 之 ---- 在多线程中启用多个事件循环</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">3</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">https://www.cnblogs.com/traditional/p/11828780.html</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">asyncio：python3 未来并发编程主流、充满野心的模块</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">4</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">https://cloud.tencent.com/developer/article/1187407</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">深入理解 Python 异步编程</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">5</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">https://zhuanlan.zhihu.com/p/59621713</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">Python 中协程异步 IO（asyncio）详解</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">7</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">https://www.bilibili.com/video/BV1Rq4y1s7Tg</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;"><strong>「视频教程，且看且珍惜」</strong></td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">8</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">https://www.cnblogs.com/crazymagic/articles/10066612.html</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">python 协程和异步 io- 上面视频教程文字版</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">9</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">https://www.cnblogs.com/crazymagic/articles/10066619.html</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">python asyncio 并发编程 - 上面视频教程文字版</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">10</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">https://www.bilibili.com/video/BV1Ke411W71L</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">2 小时学会 python asyncio - 老男孩的教学视频，讲的蛮细的，可以先看这个哦</td></tr></tbody></table>
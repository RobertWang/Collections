> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/ExMan/p/10426814.html)

> [https://zh.wikipedia.org/zh-cn/%E5%8D%8F%E7%A8%8B](https://zh.wikipedia.org/zh-cn/%E5%8D%8F%E7%A8%8B)

协程可以理解为线程中的微线程，通过手动挂起函数的执行状态，在合适的时机再次激活继续运行，而不需要上下文切换。所以在 python 中使用协程会比线程性能更好。

### Tornado 协程

> [http://blog.csdn.net/wyx819/article/details/45420017](http://blog.csdn.net/wyx819/article/details/45420017)

上面有大牛分析的 Tornado 的线程实现，依赖与 Tornado 的 IOLoop，所以不能单独拿出来使用。有几个需要理解的概念:

1.  Future 对象 用来保存异步获取到的结果，并在 set_reslut 的时候调用 callback 方法，把对应的 callback 方法放到 ioloop 的 callback 列表中等待下一次 ioloop 循环再执行
2.  装饰器 coroutine 在这个装饰器中实现了协程的调度，通过不断的调用 next 函数来不断获取 Future 对象，然后每次拿到 Future 对象在 add_callback 到 ioloop 上，等到 Future 被 set_reslut 后再次 next，直到生成器中抛出 Return 的异常。

具体的实现过程不是很好描述，调度过程比较复杂，还是看看参考文章大牛的解析吧。

### Greenlet

生成器实现的协程调度起来很麻烦，而且不是正在意义上的协程，只是实现的代码执行过程中的挂起，唤醒操作。而 Greenlet 这个 Stackless 的副产品则实现了真正的协程，在使用过程中通过 switch 来中断当前执行的函数，切换到另一个 greenlet，在其它的 geenlet 中调用 switch 会激活之前被挂起的协程。

Greenlet 没有自己的调度过程，所以一般不会直接使用。以下参考文章是 Greenlet get started 的中文翻译。

> [http://www.importcjj.com/greenlet-qing-liang-ji-bing-fa-bian-cheng.html](http://www.importcjj.com/greenlet-qing-liang-ji-bing-fa-bian-cheng.html)

### Eventlet

> [http://blog.csdn.net/gaoxingnengjisuan/article/details/12913275](http://blog.csdn.net/gaoxingnengjisuan/article/details/12913275)  
> [http://blog.csdn.net/gaoxingnengjisuan/article/details/12914831](http://blog.csdn.net/gaoxingnengjisuan/article/details/12914831)

Eventlet 在 Greenlet 的基础上实现了自己的 GreenThread，实际上就是 greenlet 类的扩展封装，而与 Greenlet 的不同是，Eventlet 实现了自己调度器称为 Hub，Hub 类似于 Tornado 的 IOLoop，是单实例的。在 Hub 中有一个 event loop，根据不同的事件来切换到对应的 GreenThread。  
同时 Eventlet 还实现了一系列的补丁来使 Python 标准库中的 socket 等等 module 来支持 GreenThread 的切换。Eventlet 的 Hub 可以被定制来实现自己调度过程。

### Gevent

> [http://xlambda.com/gevent-tutorial/](http://xlambda.com/gevent-tutorial/)  
> [http://www.open-open.com/lib/view/open1409705174822.html](http://www.open-open.com/lib/view/open1409705174822.html)

Gevent 的 2 架马车，libev 与 Greenlet。不同于 Eventlet 的用 python 实现的 hub 调度，Gevent 通过 Cython 调用 libev 来实现一个高效的 event loop 调度循环。同时类似于 Event，Gevent 也有自己的 monkey_patch，在打了补丁后，完全可以使用 python 线程的方式来无感知的使用协程，减少了开发成本。

在 Python 的世界里由于 GIL 的存在，线程一直都不是很好用，所以就有了各种协程的 hack。Gevnet 是当前使用起来最方便的协程了，但是由于依赖于 libev 所以不能在 pypy 上跑，如果需要在 pypy 上使用协程，Eventlet 是最好的选择。
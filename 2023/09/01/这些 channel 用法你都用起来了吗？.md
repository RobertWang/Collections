> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7272674088778858556)*   [GO 通道和 sync 包的分享](https://juejin.cn/post/6973108979777929230 "https://juejin.cn/post/6973108979777929230")
*   [GO 中 channel 实现原理](https://juejin.cn/post/6975280009082568740 "https://juejin.cn/post/6975280009082568740")

**本次，我们主要分享的是关于 nil channel 通道，有缓冲通道，无缓冲通道 的常用方法以及巧妙使用的方式**

**巧用 nil 的 channel 通道**
-----------------------

平日里使用的 channel 通道都是使用无缓冲，或者有缓冲的 channel 通道，或许使用为 nil 的 channel 通道还是比较少，甚至都不知道如何去使用 nil 的 channel 通道

**我们先来看这么一个例子**

1.  创建两个 channel c1, c2 ，数据类型是 struct{} ，用于占位
2.  分别开辟两个子协程，其中子协程 1 在 2 秒之后写入数据给到 c1，另外一个子协程 2 在 1 秒之后写入数据给到 c2
3.  主协程循环等待阻塞读取 c1 , c2 里面的数据，读取后将对应的标识 ok1 / ok2 置为 true
4.  当 ok1 和 ok2 都为 true 的时候，退出循环，结束程序

```
func main() {
    c1, c2 := make(chan struct{}), make(chan struct{})
    go func() {
        time.Sleep(time.Second * 2)
        c1 <- struct{}{}
        //close(c1)
    }()

    go func() {
        time.Sleep(time.Second * 1)
        c2 <- struct{}{}
       // close(c2)
    }()

    var (
        ok1 bool
        ok2 bool
    )
    for {
        select {
        case <-c1:
                ok1 = true
                fmt.Println("1")
        case <-c2:
                ok2 = true
                fmt.Println("2")
        }

        if ok1 && ok2 {
                break
        }
    }

    fmt.Println("program termination ... ")
}


```

运行结果如下：

```
2
1
program termination ...


```

看上去效果一切正常，若此时，我们将上述代码中的 `close(c1)` 和 `close(c2)` 的注释去掉，我们再查看一下结果就回是这样的：

```
...
2
2
2
1
program termination ...


```

出现这样的问题是什么呢？是因为我们 **close channel 通道之后，若还对这个通道写入数据会 panic，若还从这个通道读取数据会立即返回该通道类型的零值，而不会阻塞等待数据**

因此才会有上述情况，那么这个时候，我就可以很好的用好这个 nil 的 channel，咱就可以这样来调整一下关于通道使用的情况

修改为，**从通道中读取数据时，先判断通道是否已经关闭，若关闭则将通道设置为 nil，若未关闭，则打印我们从通道中读取的数据（此处模拟直接打印一个固定的值）**

```
for {
    select {
    case _, ok := <-c1:
        if !ok {
            c1 = nil
        }else{
            fmt.Println("1")
        }
    
    case _, ok := <-c2:
        if !ok {
            c2 = nil
        }else{
            fmt.Println("2")
        }
    }
    
    if c1 == nil && c2 == nil {
            break
    }
}


```

这种时候，我们就知道对于从通道中读取数据，先去判断通道是否关闭，若通道关闭了，那么我们直接显示的给通道设置为 nil

**这里是否会有这么一个疑问？关闭通道，通道变量不应该就变成 nil 了吗？为什么我们还要自己去设置为 nil？**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c228d23410e4de0aa76e2370bb331d7~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp#?w=198&h=198&s=67905&e=png&b=cfbfb3)

实际上这就是我们对于通道的基础知识不扎实了，关闭通道后，通道本身并不会变为 nil。通道变量仍然持有通道的地址，只是通道的状态变为了已关闭

巧用无缓冲 channel 通道
----------------

对于无缓冲的 channel 通道，只有在对其进行接收操作的 goroutine 协程和对其进行发送操作的 goroutine 协程都存在的情况下，通信才能进行，否则单方面的操作会让对应的 goroutine 协程陷入阻塞状态，**因为该 channel 通道没有缓冲**

使用无缓冲的 channel 通道，我们可以用在如下几个方面

1.  信号传递

信号传递我们就可以用在两个协程一对一的传递信号上面，当然我们也可以使用在主协程主动通知所有子协程关闭的全场景下，这就是一对多的传递信号，相关的 demo 可以在这期文章中有展示

[GO 语言的并发模式](https://juejin.cn/post/7268965126127648807 "https://juejin.cn/post/7268965126127648807")

**一对一（一个发一个收）**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9efa68fe4e114aba8ea3315663b2dc7f~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp#?w=984&h=374&s=16304&e=jpg&b=ffffff)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1d6044854c5f49ee850201468481a6aa~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp#?w=934&h=434&s=16669&e=jpg&b=ffffff)

**一对多（一个发多个收，此处可以是 协程 1 close 掉 通道，那么 多个协程默认都能够读取到通道的值是零值，此时多个子协程就可以根据通道的关闭状态来处理后续的逻辑）**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/29de597fcd004bc09c8761869e7ad817~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp#?w=984&h=374&s=16304&e=jpg&b=ffffff)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f361267e6e354dc6b9e2d983129255b3~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp#?w=1094&h=994&s=46906&e=jpg&b=ffffff)

2.  控制同步

GO 语言倡导我们**不要通过共享内存来通信，而应该通过通信来共享内存**，此处 channel 就是这样设计的，当然如果需要有更高的性能，那么我们还是可以使用更加低级的 GO 语言原语 sync 包中的锁机制

可以点击查看往期文章：[sync 锁机制](https://juejin.cn/post/7272523490497986615 "https://juejin.cn/post/7272523490497986615")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e5d38f80799449659f6e7e2aa267c84d~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp#?w=1114&h=1002&s=49803&e=jpg&b=ffffff)

巧用有缓冲 channel 通道
----------------

1.  **用作队列**

用作队列应该是比较好理解的，队列先入先出 FIFO，给 channel 通道设置明确的缓冲区，例如 `ch:=make(chan int, 10)`

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/06a9bbfeec8a4e00b3bcfd6c5f610480~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp#?w=1294&h=594&s=48409&e=jpg&b=ffffff)

多个协程就可以异步的并发处理该队列，由于有缓冲的 channel 通道中有一定的容量，因此，对于协程读取通道中数据时，存在阻塞的情况相对无缓冲的通道来说就会少很多，**相应的在一定程度上就提升了性能**

对于有缓冲的 channel 通道，channel 通道满的时候，写入数据会阻塞，读取数据正常处理， channel 通道空的时候，写入数据正常，读取数据会阻塞

2.  **用作信号量**

有缓冲的 channel 通道还可以用来计数，例如我们有 15 个 job，可是目前只有 3 个 worker，那么同一时间，只会有 3 个 worker 来干活，我们就可以使用通道来查看目前有多少个 worker 在工作，写一个简单的 demo

*   创建 j 和 worker channel 通道，
*   子协程 1 写 15 个任务给到 j 通道中，写完 15 个任务到 j 中便关闭自己的通道（因为后续我们需要使用 for...range 的方式读取通道）
*   使用 sync.WaitGroup 管控开辟的 3 个协程，模拟 3 个 工人去干活
*   能够从写入数据到 worker channel 通道中，则开始干活，干完之后，从 worker channel 通道中读出数据

```
func main() {
    j := make(chan int, 15)
    worker := make(chan int, 3)

    go func() {
        for i := 0; i < 15; i++ {
            j <- i
        }
        close(j)
    }()

    var wg sync.WaitGroup

    for job := range j {
        wg.Add(1)
        go func(job int) {
            defer wg.Done()
            worker <- job
            // 模拟干活
            fmt.Println("正在执行 job : ", job)
            time.Sleep(time.Second * 1)
            <-worker
        }(job)
    }

    wg.Wait()
    fmt.Println("program termination ... ")
}


```

感兴趣的 xdm 的可以复制代码运行一下，可以看到效果是 3 个 job 一起打印，间隔 1 秒后，又是 3 个 job 一起打印的😁

select 和 channel 通道如何结合使用？
--------------------------

1.  心跳

```
func main() {
    h := time.NewTicker(2 * time.Second)
    defer h.Stop()
    for {
        select {
        case <-h.C:
            // 模拟处理心跳
            fmt.Println("hhh")
        }
    }
}


```

2.  使用 default

`select ...default` 这个组合就不必过多赘述了，就是在我们阻塞读取通道数据时，若当前时间没有从任何一个通道中读取到数据，则默认走 default 里面的逻辑

3.  超时机制

超时机制使用的也是非常频繁的，很多时候为了方便，可能我们会使用例如`<- time.After(10 * time.Second)` 的方式，使用这种方式，**GO 语言会维护一个最小堆，当时间到了，通道被唤醒的时候，就会从最小堆顶取出 timer 对象，再执行 timer 中的函数，执行完毕之后，自行就会做删除，自行就会做** **GC**

可是在上述这种方式使用比较多的时候，会给程序带来 GC 的压力，**我们完全可以入如下方式来实现超时机制，显示的去做 GC**

```
func main() {
    c := time.NewTimer(10 * time.Second)
    defer c.Stop()
    for {
        select {
        case <-c.C:
            fmt.Println("program overtime ")
            return
        }
    }
}


```

总结
--

本次演示了关于 nil channel，有缓冲 channel ，无缓冲 channel ， select 如何与 channel 配合使用，上述 demo 完全可以复制下来，xdm 可以自行运行，查看效果

欢迎点赞，关注，收藏
----------

朋友们，你的支持和鼓励，是我坚持分享，提高质量的动力

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9e19850b0c884741a47692ccdfdba272~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp#?w=240&h=240&s=85012&e=webp&f=11&b=fbdbec)

好了，本次就到这里

技术是开放的，我们的心态，更应是开放的。拥抱变化，向阳而生，努力向前行。

我是**阿兵云原生**，欢迎点赞关注收藏，下次见~
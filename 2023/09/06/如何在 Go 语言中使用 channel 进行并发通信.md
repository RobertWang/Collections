> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/rT5RiOjAohHxOoWAPrzS2w)

那么，什么情况下需要关闭 channel 呢？如何正确关闭 channel 呢？在 Golang 中，channel 的关闭确实是一个相对较为复杂的话题，下面我们就来探究一下这个话题。

一、为什么需要关闭 channel？

首先，需要明确一点的是，channel 并不是必须关闭的，也就是说，Golang 会在最后一个指向 channel 的指针被销毁时自动关闭 channel。但不关闭 channel 会导致如下几个问题：

1.  内存泄漏。未关闭的 channel 会一直占用内存，等待数据发送或接收，从而导致内存泄漏。
    
2.  阻塞。未关闭的 channel 会一直等待数据发送或接收，如果持续等待时间过长，可能会导致程序阻塞。
    
3.  无法及时释放资源。在有些场景下，我们需要使用到 channel 来协调多个 goroutine 的操作，如果不及时关闭 channel，可能会导致资源无法及时释放，从而影响程序的运行效率。
    

二、如何正确关闭 channel？

既然我们知道了为什么需要关闭 channel，那么如何正确关闭 channel 呢？其实 Golang 提供了两种方式来关闭 channel：

1.  使用 close() 函数
    

关闭 channel 的最简单也是最常见的方法就是使用 Golang 提供的 close() 函数。该函数的语法格式如下：

```
close(channel)

```

该函数需要传入一个 channel 类型的参数，当传入的 channel 类型的参数被关闭后，就不能再次对它进行发送或接收操作，否则会引发 panic。

2.  使用 for range 循环关闭 channel
    

第二种常用的关闭 channel 的方式是使用 for range 循环，这种方式对于从 channel 中接收值的操作比较常见。对于发送数据到 channel 中的操作，较少使用该方式来关闭 channel。使用 for range 循环关闭 channel 的代码示例如下：

```
for val := range channel {
    fmt.Println(val)
}

// channel被关闭后，上述代码会正常退出循环

```

在 for range 循环中，当 channel 被关闭时，循环会自动结束。值得注意的是，如果我们在 for range 循环中使用 break 或 continue 等语句来跳出循环，也无法避免 channel 的继续接收操作。

在使用 close() 函数关闭 channel 时，有一个比较重要的注意点，就是我们需要确保在向 channel 发送所有值的操作都完成之后关闭该 channel，否则在 channel 没有完全接受之前进行关闭操作可能会导致 panic。下面我们就来看一下如何避免这种情况的发生。

1.  利用缓存的 channel
    

对于一些在发送数据之前需要进行的复杂计算或者校验操作，我们可以使用缓存 channel 的方式来避免 panic 的情况。具体实现方式如下：

```
ch := make(chan bool, 1)
go func() {
    // 进行复杂计算或者校验操作
    // ...
    ch <- true
}()

select {
case <- done:
    // 结束操作
case <- ch:
    // 处理收到的数据
}
close(ch)

```

在上述代码中，我们使用了一个缓冲为 1 的 channel。该 channel 中仅存储了一个布尔类型的值，当我们在创建 goroutine 之后执行完复杂操作之后，向该 channel 中发送 true 值，表示操作完成。然后在 select 语句中等待向该 channel 中发送值或者接收其他 channel 中的值。最后，我们使用 close() 函数关闭了该 channel。

2.  利用 select 语句
    

在使用 select 语句时，我们可以使用 default 分支来处理 channel 关闭之前的场景。代码示例如下：

```
func handleCh(channel chan int) {
    for {
        select {
        case val, ok := <- channel:
            if !ok {
                fmt.Println("channel has closed")
                return
            }
            fmt.Println("recieve val:", val)
        default:
            fmt.Println("no value received")
        }
    }
}

func main() {
    ch := make(chan int)
    for i := 0; i < 5; i++ {
        go func(val int) {
            ch <- val
        }(i)
    }
    close(ch)
    handleCh(ch)
}

// 输出结果：
// recieve val: 0
// recieve val: 1
// recieve val: 2
// recieve val: 3
// recieve val: 4
// channel has closed

```

在上述代码中，我们创建了一个 handleCh() 函数来处理从 channel 中接收到的值。在该函数中，我们使用 select 语句来处理从 channel 中接收到的数据，并在 default 分支中处理 channel 未关闭时的场景。当我们在主函数中关闭 channel 时，handleCh() 函数会正常结束。不过需要注意的是，在使用 default 分支时，一定要将它放到最后，否则会导致程序出错。

四、总结

通过以上的介绍，我们了解了 Golang 中 channel 关闭的原因和方式。一般来说，我们需要手动关闭 channel，以避免出现内存泄漏、阻塞等问题。在关闭 channel 时，我们需要分别使用 close() 函数或者 for range 循环的语句，避免发生 panic 的情况。目前在实际的开发中，我们可以使用缓存的 channel 或者 select 语句等方式来处理 channel 关闭之前的场景，从而达到优化程序效果的目的。

  
以上就是 golang 的 channel 关闭的详细内容，更多请关注公众号其它相关文章！
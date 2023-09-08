> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/DvTKXf4doKbYJieCsQRodA)

> 当谈到并发时，许多编程语言采用共享内存 / 状态模型。然而，Go 通过实现通信顺序进程 (CSP) 来区别自己。

当谈到并发时，许多编程语言采用共享内存 / 状态模型。然而，Go 通过实现通信顺序进程 (CSP) 来区别自己。在 CSP 中，一个程序由并行进程组成，这些进程不共享状态；相反，它们使用通道来通信并同步它们的行为。

因此，对于有兴趣采用 Go 的开发者来说，理解通道的工作原理变得至关重要。在这篇文章中，我将使用 Gophers 运行他们的想象咖啡店的有趣类比来描述通道，因为我坚信人们更容易通过视觉来学习。

场景
--

Partier、Candier 和 Stringer 正在经营一个咖啡馆。由于制作咖啡所需的时间比接受订单要长，Partier 会协助接受顾客的订单，然后将这些订单传递给厨房，Candier 和 Stringer 在那里准备咖啡。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/ETjDRRpUZUhog9uGyY9SgA3EdN3X6aKzGSzEpsO3nnghOticIZcaZA5Vg0dR4TzC2OnmF6QHRnFw4U4O8nlCXIA/640?wx_fmt=jpeg)

无缓冲通道
-----

最初，咖啡店以最简单的方式运作：每当收到一个新订单时，Partier 将订单放入通道，并等待 Candier 或 Stringer 拿走它，然后再接受任何新订单。通过无缓冲通道，使用 `ch := make(chan Order)` 创建，实现了 Partier 和厨房之间的这种沟通。当通道中没有挂起的订单时，即使 Stringer 和 Candier 都准备好接受新订单，他们仍然处于等待状态，等待新订单到来。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/ETjDRRpUZUhog9uGyY9SgA3EdN3X6aKzficOm8NMB7nKdfFcBiaMlaM88NXKic5pXiaruvibXjatqu8tIM7jgyl43Ig/640?wx_fmt=jpeg)

当收到一个新订单时，Partier 将其放入通道，使该订单可以被 Candier 或 Stringer 拿走。但在接受新订单之前，Partier 必须等待其中一个人从通道中取出订单。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/ETjDRRpUZUhog9uGyY9SgA3EdN3X6aKzPpIUITCZKleNsU6205TTduxlZdUOjc2sAMo8BkQzyKmWufCkABy14g/640?wx_fmt=jpeg)

当 Stringer 和 Candier 都准备好接受新订单时，新订单将立即被其中一个拿走。然而，不能保证或预测到底是谁会拿到订单。Stringer 和 Candier 之间的选择是不确定的，这取决于诸如调度和 Go 运行时的内部机制之类的因素。假设 Candier 得到了这个第一个订单。

在 Candier 完成处理第一个订单后，她回到等待状态。如果没有新的订单到来，两个工人，Candier 和 Stringer，都会闲置，直到 Partier 将另一个订单放入通道供他们处理。

当收到一个新订单，且 Stringer 和 Candier 都可以处理它时。即使 Candier 刚刚处理了上一个订单，接收新订单的特定工人仍然是不确定的。在这个场景中，我们假设 Candier 再次被分配了这第二个订单。

新的订单 `order3` 到来，由于 Candier 正在处理 `order2`，她并没有等在 `order := <-ch` 这一行，Stringer 成为了唯一可以接收 `order3` 的可用工人。因此，他会得到它。

在 `order3` 发送给 Stringer 之后，`order4` 立即到达。此时，Stringer 和 Candier 都已经在处理他们各自的订单，没有人可以拿走 `order4`。因为通道没有缓冲，将 `order4` 放入通道会阻塞 Partier，直到 Stringer 或 Candier 有一个变得可以接受 `order4`。我经常看到人们对无缓冲通道（用 `make(chan order)` 或 `make(chan order, 0)` 创建）和缓冲大小为 1 的通道（用 `make(chan order, 1)` 创建）感到困惑。因此，他们错误地期望 `ch <- order4` 立即完成，允许 Partier 接受 `order5`，然后在 `ch <- order5` 上被阻塞。如果你也有这种想法，我在 Go Playground 上创建了一个代码片段，帮助你纠正误解 https://go.dev/play/p/shRNiDDJYB4。

缓冲通道
----

无缓冲通道确实可以工作，但它限制了整体的吞吐量。如果他们仅接受一定数量的订单在后台（厨房）顺序处理会更好。这可以通过**缓冲通道**来实现。现在，即使 Stringer 和 Candier 忙于处理他们的订单，只要通道没有满，Partier 仍然可以在通道中留下新的订单并继续接受其他订单，例如，最多 3 个挂起的订单。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/ETjDRRpUZUhog9uGyY9SgA3EdN3X6aKzZob4WMQqNHhmcyUTTMVcibb6kQk4BF48wvZcoffUolFIocWmsmt9picQ/640?wx_fmt=jpeg)

通过引入缓冲通道，咖啡店增强了处理更多订单的能力。但是，选择适当的缓冲大小以保持合理的客户等待时间是至关重要的。毕竟，没有客户愿意忍受过长的等待时间。有时，拒绝新订单可能比接受它们并且无法及时完成它们更为可取。此外，在短暂的容器化（Docker）应用程序中使用缓冲通道时必须小心，因为预期会有随机重启，在这种情况下，从通道中恢复消息可能是一项具有挑战性的任务，甚至可能是不可能的。

通道与阻塞队列
-------

尽管它们在本质上是不同的，但 Java 中的 Blocking Queue 是用来在线程之间通信的，而 Go 中的 Channel 是用于 Goroutine 的通信，BlockingQueue 和 Channel 的行为在某种程度上是相似的。如果你熟悉 BlockingQueue，那么理解 Channel 肯定会很容易。

常见用途
----

通道是 Go 应用中的一个基础且广泛使用的功能，服务于各种目的。通道的一些常见用途包括：

*   • **Goroutine 通信**：通道使不同的 goroutines 之间能够进行消息交换，使它们可以协作而无需直接共享状态。
    
*   • **工作池**：正如上面的示例中所见，通道经常用于管理工作池，其中多个相同的工作人员从共享通道处理传入的任务。
    
*   • **扇出，扇入**：通道促进了扇出，扇入模式，其中多个 goroutines（扇出）执行工作并将结果发送到一个通道，而另一个 goroutine（扇入）消费这些结果。
    
*   • **超时和截止日期**：与 `select` 语句结合使用，通道可以用于处理超时和截止日期，确保程序可以优雅地处理延迟并避免无限的等待。
    

我将在其他文章中更详细地探讨通道的不同用途。但是，现在，让我们通过实现上述的咖啡店场景来结束这篇入门博客，并亲眼看到通道如何在实践中工作。我们将探索 Partier、Candier 和 Stringer 之间的互动，并观察通道如何促进他们之间的顺畅沟通和协调，使咖啡店能够有效地处理订单和同步。

给我看你的代码！
--------

```
package main

import (
    "fmt"
    "log"
    "math/rand"
    "sync"
    "time"
)

func main() {
    ch := make(chan order, 3)

    wg := &sync.WaitGroup{} // More on WaitGroup another day
    wg.Add(2)

    go func() {
        defer wg.Done()
        worker("Candier", ch)
    }()

    go func() {
        defer wg.Done()
        worker("Stringer", ch)
    }()

    for i := 0; i < 10; i++ {
        waitForOrders()
        o := order(i)
        log.Printf("Partier: I %v, I will pass it to the channel\n", o)
        ch <- o
    }

    log.Println("No more orders, closing the channel to signify workers to stop")
    close(ch)

    log.Println("Wait for workers to gracefully stop")
    wg.Wait()

    log.Println("All done")
}

func waitForOrders() {
    processingTime := time.Duration(rand.Intn(2)) * time.Second
    time.Sleep(processingTime)
}

func worker(name string, ch <-chan order) {
    for o := range ch {
        log.Printf("%s: I got %v, I will process it\n", name, o)
        processOrder(o)
        log.Printf("%s: I completed %v, I'm ready to take a new order\n", name, o)
    }
    log.Printf("%s: I'm done\n", name)
}

func processOrder(_ order) {
    processingTime := time.Duration(2+rand.Intn(2)) * time.Second
    time.Sleep(processingTime)
}

type order int

func (o order) String() string {
    return fmt.Sprintf("order-%02d", o)
}

```

您可以复制这段代码，在您的 IDE 上进行调整并运行它，以更好地理解通道是如何工作的。
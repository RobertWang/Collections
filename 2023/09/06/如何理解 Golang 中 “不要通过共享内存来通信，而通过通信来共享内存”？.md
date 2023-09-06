> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/3MUb9j1xexx_uoXW4HpDvg)

**一、前言**

通信机制是 Golang 实现并发和并行的重要手段之一，也是实现共享内存的关键。

**1.1 共享内存**

最简单的就是同一机器上两个进程之间的通信，进程 1 在共享内存中写入数据，进程 2 从共享内存中读出数据，或者两个进程均可对共享内存进行读写，我写的是最简单的情况：进程 1 写入数据，进程 2 读出数据。

共享内存意味着让多个线程或进程同时访问同一个区域的内存。这样做虽然能够提高程序运行效率，但同时也会导致数据不一致、竞态条件等问题。为了解决这些问题，Golang 采用了 “不要通过共享内存来通信，而应该通过通信来共享内存” 的哲学。

**1.2 比共享内存更合理的解决方案**

人们通过多年试错和迭代，最终 “通过通信来实现进程 / 线程间交互” 的方案脱颖而出，成为了大多数人的共识。而 Golang 通过 channel 来通信实现数据共享。channel 是一种类似于 FIFO 队列的数据结构，支持两种基本操作：发送数据和接收数据。可以将 channel 看作是一条管道，通过管道连接不同的 goroutine，实现数据的传输和共享。channel 的所有操作都是原子的，因此不存在竞态条件。同时，由于 channel 的特性，确保数据的唯一性，从而避免了数据一致性的问题。

**二**、案例说明****

******2.1 atomic 内存共享数据******

```
package main
import (
  "fmt"
  "sync/atomic"
)
func main() {
  var counter int32 = 0
  for i := 0; i < 10; i++ {
    go func() {
      atomic.AddInt32(&counter, 1) // 原子增加计数器的值
    }()
  }
  fmt.Println(atomic.LoadInt32(&counter)) // 输出计数器的值
}

```

在这个示例中，我们定义了一个名为 counter 的 int32 类型变量，并使用 atomic.AddInt32 函数在多个 goroutine 之间累加值。最后我们使用 atomic.LoadInt32 函数读取计数器的值，并输出到控制台上。

需要注意的是，使用 atomic 需要特别小心。我们需要确保操作的原子性和线程安全性，并遵循正确的内存模型和同步协议。如果没有正确地使用 atomic，可能会导致数据竞争和其他一些问题。

********2.2 channel 通信共享数据********

```
package main
import "fmt"
func main() {
  c := make(chan int) // 创建channel
  go func() {         // 启动goroutine1，向channel中发送数据
    fmt.Println("Sending 10 to the channel...")
    c <- 9
  }()
  go func() { // 启动goroutine2，从channel中接收数据
    fmt.Println("Receiving data from the channel...")
    data := <-c
    fmt.Printf("Received: %d \n", data)
  }()
  fmt.Scanln()
}

```

在这个例子中，我们创建了一个 channel，并启动了两个 goroutine。其中，goroutine1 通过 channel 向管道中发送了一个整数 9，而 goroutine2 则从 channel 中读取了这个整数并进行打印输出。可以看到，通过 channel 实现数据的共享和传输非常简单、方便。

****三********、****总结****

传统语言（如 C 系列）中，多个线程同时访问同一块内存，因此需要：

*   用互斥锁的加锁来宣布使用某块内存。
    
*   用解锁来宣布不再使用某块内存。
    

而在 golang 里，各协程并不是各自去访问全局变量（内存块），而是：

*   通过尝试从 channel 收取内存块的引用，来宣布使用某块内存。
    
*   通过将内存块的引用（指针、下标、引用）发送到 channel，来宣布不再使用某块内存。
    

Golang 通过通信实现共享内存是一种非常高效、可靠的方式。使用 channel，可以避免传统共享内存的各种问题，保证程序运行的正确性和稳定性。
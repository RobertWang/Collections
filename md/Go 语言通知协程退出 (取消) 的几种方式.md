> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI2ODIzMDM5MQ==&mid=2650725973&idx=1&sn=0e1edcaba1992b592219adef36721931&chksm=f2f8b5cdc58f3cdb3340b697a241ebce560d67e5ff3bb6f5cc258cfbb02a4027cf5bac811342&mpshare=1&scene=1&srcid=0202mFVFSEEkZmPAMe5CAofS&sharer_shareinfo=102f55c2b4ef45f1744a8518a0ea64d8&sharer_shareinfo_first=102f55c2b4ef45f1744a8518a0ea64d8#rd)

在 Go 语言中，控制 goroutine 的退出或取消很重要，这能使资源得到合理利用，避免潜在的内存泄露。

如下是一些在 Go 中通知协程退出的常见方式：

1.  **使用通道（`Channel`）**：通过发送特定的信号或关闭通道来通知协程退出。这是最简单直接的方法。
    
2.  **使用 `context` 包**：`context` 包提供了一种更标准化的方式来传递取消信号、超时、截止时间等控制信息。
    
3.  ** 使用 `sync.WaitGroup`**：虽然 `WaitGroup` 本身不用于发送取消信号，但它可以用来等待一组协程完成，通常与其他方法（如通道）结合使用来控制协程的退出。
    

  

### 1. 使用通道

  

```
package main

import (
    "fmt"
    "time"
)

func worker(exitChan chan bool) {
    for {
        select {
        case <-exitChan:
            fmt.Println("Worker exiting.")
            return
        default:
            fmt.Println("Working...")
            time.Sleep(1 * time.Second)
        }
    }
}

func main() {
    exitChan := make(chan bool)
    go worker(exitChan)

    time.Sleep(3 * time.Second) // 模拟做了一些工作
    exitChan <- true            // 通知协程退出
    time.Sleep(1 * time.Second) // 给协程时间退出
}


```

输出:

```
Working...
Working...
Working...
Working...
Worker exiting.


```

  

在线代码 [1]

  

### 2. 使用 `context` 包

  

```
package main

import (
    "context"
    "fmt"
    "time"
)

func worker(ctx context.Context) {
    for {
        select {
        case <-ctx.Done():
            fmt.Println("Worker exiting.")
            return
        default:
            fmt.Println("Working...")
            time.Sleep(1 * time.Second)
        }
    }
}

func main() {
    ctx, cancel := context.WithCancel(context.Background())
    go worker(ctx)

    time.Sleep(3 * time.Second)  // 模拟做了一些工作
    cancel()                     // 通知协程退出
    time.Sleep(1 * time.Second)  // 给协程时间退出
}


```

输出:

```
Working...
Working...
Working...
Working...
Worker exiting.


```

  

在线代码 [2]

  

在上面这两个示例中，当主函数完成其工作后，通过通道发送信号或调用 `cancel` 函数来通知协程退出。使用 `context` 包是更推荐的做法，因为其提供了一种更标准化和灵活的方式来管理协程的生命周期。

  

### 3. 使用 `sync.WaitGroup` 控制协程退出

  

`sync.WaitGroup` 主要用于等待一组协程的完成。其不直接提供通知协程退出的机制，但可以与其他方法（如通道）结合使用来控制协程的退出。

#### 示例：

```
package main

import (
    "fmt"
    "sync"
    "time"
)

func worker(id int, wg *sync.WaitGroup, stopCh chan struct{}) {
    defer wg.Done()
    for {
        select {
        case <-stopCh: // 接收退出信号
            fmt.Printf("Worker %d stopping\n", id)
            return
        default:
            fmt.Printf("Worker %d working\n", id)
            time.Sleep(time.Second)
        }
    }
}

func main() {
    var wg sync.WaitGroup
    stopCh := make(chan struct{})

    workerCount := 3
    wg.Add(workerCount)

    for i := 0; i < workerCount; i++ {
        go worker(i, &wg, stopCh)
    }

    time.Sleep(3 * time.Second) // 模拟工作
    close(stopCh)               // 发送退出信号给所有协程

    wg.Wait() // 等待所有协程退出
    fmt.Println("All workers stopped")
}


```

输出:

```
Worker 0 working
Worker 2 working
Worker 1 working
Worker 2 working
Worker 0 working
Worker 1 working
Worker 0 working
Worker 2 working
Worker 1 working
Worker 2 stopping
Worker 0 stopping
Worker 1 stopping
All workers stopped


```

  

在线代码 [3]

  

在上例中，`stopCh` 通道用于通知协程退出。当关闭 `stopCh` 时，所有监听这个通道的协程都会接收到信号，并优雅地停止执行。

但我觉得这个例子并不好, 本质上成了和 **1. 使用通道（Channel）**一样了..

`sync.WaitGroup`的真正作用是卡在`wg.Wait()`处，直到`wg.Done()`被执行 (`wg.Add()`增加的值被减为 0), 才会继续往下执行. 比如往往用于防止 goroutine 还没执行完, 主协程就退出了

  
  

另外, 如果是性能敏感场景, 往往使用原子操作（Atomic）在多个协程之间安全地共享状态 (_原子操作用于安全地读写共享状态，可以用来设置一个标志，协程可以定期检查这个标志来决定是否退出_), 而不使用通道来做协程间的通信

  

更多参考:

golang context 父子任务同步取消信号 [4]

Go 程序员面试笔试宝典 - context 如何被取消 [5]

如何退出协程 goroutine (其他场景)[6]

go 协程取消 [7]

参考资料

[1]

在线代码: https://go.dev/play/p/HrZbNO-jyKf

[2]

在线代码: https://go.dev/play/p/hg_w1bxcmyg

[3]

在线代码: https://go.dev/play/p/9AwV2v9iqdu

[4]

golang context 父子任务同步取消信号: https://blog.csdn.net/whatday/article/details/113771225

[5]

Go 程序员面试笔试宝典 - context 如何被取消: https://golang.design/go-questions/stdlib/context/cancel/

[6]

如何退出协程 goroutine (其他场景): https://geektutu.com/post/hpg-exit-goroutine.html

[7]

go 协程取消: https://www.google.com/search?q=go%E5%8D%8F%E7%A8%8B%E5%8F%96%E6%B6%88
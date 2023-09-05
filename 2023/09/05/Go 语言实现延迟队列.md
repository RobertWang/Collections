> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/N1uVrh3cjN3SIVdI-ztOSQ)

**简单 Go 语言延迟队列思路：**  

```
package main
import (
  "fmt"
  "time"
)
// Message is a structure that holds the contents of the message.
type Message struct {
  ID     int
  Body   string
  Delay  time.Duration
}
// delayMessageQueue holds the queue of messages
type delayMessageQueue struct {
  queue []Message
}
func (d *delayMessageQueue) send(msg Message) {
  go func() {
    timer := time.NewTimer(msg.Delay)
    <-timer.C
    d.queue = append(d.queue, msg)
    fmt.Printf("Message %d sent\n", msg.ID)
  }()
}
func (d *delayMessageQueue) receive() {
  for {
    if len(d.queue) > 0 {
      msg := d.queue[0]
      fmt.Printf("Message %d received \n", msg.ID)
      d.queue = d.queue[1:] // Dequeue
    }
  end
}
func main() {
  dmq := &delayMessageQueue{}
  dmq.send(Message{ID: 1, Body: "Hello, World", Delay: 2 * time.Second})
  dmq.send(Message{ID: 2, Body: "Hello, Go", Delay: 1 * time.Second})
  // Keep the main function alive to let goroutines finish
  time.Sleep(5 * time.Second)
  dmq.receive()
}

```

我们定义了一个消息结构，包含消息 ID、消息内容和延迟。我们也定义了一个延迟消息队列，它有两个方法，一个发送消息，一个接收消息。

发送方法将消息放入一个 goroutine 中，然后用一个定时器等待指定的延迟时间。当定时器到达时，消息会添加到队列中。

接收方法将持续检查队列，一旦发现队列中有消息，就会打印消息内容并将其从队列中移除。通过 time 到达指定的时间然后进行发送。每个 goroutine 都有自己的定时器，这是非常低效的。实际上，我们应该使用一个最小堆维护所有的定时器，并且只有一个 goroutine 在阻塞等待最早的定时器。如果有更早的定时器插入，唤醒那个 goroutine 并阻塞等待新的最早的定时器。

**进阶版本**

其中所有延迟任务都由一个优先队列（最小堆）进行维护，取出最早到达的任务进行处理。我给出一个简版的示例，其他更多的细节如并发控制、错误处理等，您可以在实际开发中完善。

Go 语言标准库`container/heap`提供了堆操作的实现，通过组合使用该包提供的`heap.Push`和`heap.Pop`，可以实现一个优先队列。

```
package main
import (
  "container/heap"
  "fmt"
  "sync"
  "time"
)
// 优先队列内部维护的队列元素
type Item struct {
  value    string
  priority int64 // 延迟任务到期时间(用时间戳表示)
  index    int   // 队列元素在堆中的索引
}
// 优先队列：底层用最小堆实现
type PriorityQueue []*Item
func (pq PriorityQueue) Len() int { return len(pq) }
// 注意这里比较的是任务的优先级，为了让离当前时间最近的任务在堆顶，我们让比较结果颠倒
func (pq PriorityQueue) Less(i, j int) bool {
  return pq[i].priority < pq[j].priority
}
func (pq PriorityQueue) Swap(i, j int) {
  pq[i], pq[j] = pq[j], pq[i]
  pq[i].index = i
  pq[j].index = j
}
// 插入元素
func (pq *PriorityQueue) Push(x interface{}) {
  n := len(*pq)
  item := x.(*Item)
  item.index = n
  *pq = append(*pq, item)
}
// 删除元素
func (pq *PriorityQueue) Pop() interface{} {
  old := *pq
  n := len(old)
  item := old[n-1]
  old[n-1] = nil  // avoid memory leak
  item.index = -1 // for safety
  *pq = old[0 : n-1]
  return item
}
var mutex sync.Mutex // 并发控制互斥锁
// 设定好比cmpTime更晚的任务执行时间并加入堆
func addTask(pq *PriorityQueue, cmpTime int64, diff int64) {
  mutex.Lock() // 加锁
  taskTime := cmpTime + diff*int64(time.Second)
  item := &Item{
    value:    fmt.Sprintf("任务%d", taskTime),
    priority: taskTime,
  }
  heap.Push(pq, item)
  mutex.Unlock() // 解锁
}
// 从堆上取出时间最早的任务执行
func doTask(pq *PriorityQueue) {
  for {
    mutex.Lock()
    if len(*pq) == 0 {
      mutex.Unlock()
      continue // 堆空则不处理
    }
    // 堆非空，取出最早任务项
    item := heap.Pop(pq).(*Item)
    now := time.Now().Unix()
    if item.priority-now > 0 {
      // 未到执行时间，任务重新入堆
      heap.Push(pq, item)
    } else {
      // 执行任务
      fmt.Printf("%s 执行\n", item.value)
    }
    mutex.Unlock()
    // 防止doTask过于紧密，每次循环停顿1秒
    time.Sleep(1 * time.Second)
  }
}
func main() {
  pq := make(PriorityQueue, 0)
  heap.Init(&pq)
  // 启动执行任务goroutine
  go doTask(&pq)
  now := time.Now().Unix()
  // 预设3个延迟任务，延迟时间分别为8s, 2s, 5s
  addTask(&pq, now, 8)
  addTask(&pq, now, 2)
  addTask(&pq, now, 5)
  // 保持主进程
  time.Sleep(15 * time.Second)
}

```

在这个实现中，我们创建一个优先队列（Priority Queue），优先级最高的任务（即最早执行的任务）总是位于堆的顶部，这样我们可以确保总是首先处理最早执行的任务。当新的任务到来或者任务完成时，我们都用`heap.Push`和`heap.Pop`重新调整堆以保证最早执行的任务总是在堆顶。

关于更详细的并发控制以及错误处理，这需要根据实际的业务需求进行对应的修改和处理。
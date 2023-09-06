> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Tx_M9vWYmbIk11pFBi3WMQ)

> go-redis 是一个功能完备、易用性高的 Redis 客户端库, 它覆盖了 Redis 的绝大部分功能, 是 Go 语言连接 Redis 的首选方案

Redis 是一种流行的内存键值数据库, 被广泛用于构建高性能的缓存和消息队列应用。本文将介绍如何通过 go-redis 访问 redis。

go-redis 简介
===========

go-redis 是一个 Go 语言中非常流行的 Redis 客户端库。相比于其他 Go 语言 Redis 客户端, 它具有以下优点:

1.  API 友好, 命令名称和参数与 Redis 原生命令一致, 使用简单方便。
    
2.  支持完整的 Redis 命令集, 覆盖了字符串、哈希、列表、集合、有序集合、HyperLogLog 等数据结构。
    
3.  支持连接池, 可以有效控制连接数, 避免频繁创建连接。
    
4.  支持 Pipeline 和事务, 可以打包多个命令减少网络开销。
    
5.  支持发布订阅 Pub/Sub 和键空间通知等功能。
    
6.  支持哨兵和集群模式, 提供高可用能力。
    
7.  代码维护活跃, 持续有新功能加入。
    
8.  在 Github 上拥有 1.5 万 + 星, 是最流行的 Go Redis 客户端。
    

总之, go-redis 是一个功能完备、易用性高的 Redis 客户端库, 它覆盖了 Redis 的绝大部分功能, 是 Go 语言连接 Redis 的首选方案。  

go-redis 使用 demo
================

```
package main
import (
    "fmt"
    "github.com/go-redis/redis"
)
// 创建redis客户端
func newClient() *redis.Client {
    client := redis.NewClient(&redis.Options{
        Addr:     "localhost:6379", // redis地址
        Password: "",               // 密码
        DB:       0,                // 使用默认数据库
    })
    return client
}
func main() {
    // 创建客户端
    client := newClient()
    defer client.Close() 
    // 设置key
    err := client.Set("name", "john", 0).Err()
    if err != nil {
        panic(err)
    }
    // 获取key
    val, err := client.Get("name").Result()
    if err != nil {
        panic(err)
    }
    fmt.Println("name", val)
}

```

如代码所示, go-redis 提供了非常简洁的 API 来连接 Redis 服务器, 执行命令

go-redis 常用功能
=============

### 发布订阅

```
package main
import (
    "fmt"
    "github.com/go-redis/redis"
)
func subscriber(client *redis.Client) {
    pubsub := client.Subscribe("mychannel")
    defer pubsub.Close()
    // 处理订阅接收到的消息
    for {
        msg, err := pubsub.ReceiveMessage()
        if err != nil {
            return
        }
        fmt.Println(msg.Channel, msg.Payload)
    }
}
func publisher(client *redis.Client) {
    for {
        // 发布消息到频道
        err := client.Publish("mychannel", "hello").Err()
        if err != nil {
            panic(err)
        }
    }
}
func main() {
    client := redis.NewClient(&redis.Options{
        Addr: "localhost:6379",
    })
    go subscriber(client)
    go publisher(client)
    <-make(chan struct{})
}

```

*   通过 client.Subscribe 订阅了一个频道, 然后在循环里接收消息。
    
*   另开一个 goroutine 发布消息。使用 go-redis 可以方便地实现发布订阅模型。
    

### 消息队列

```
package main
import (
    "fmt"
    "math/rand"
    "time"
    "github.com/go-redis/redis"
)
var client *redis.Client 
// 初始化连接
func initClient() {
    client = redis.NewClient(&redis.Options{
        Addr:     "localhost:6379",
        Password: "", 
        DB:       0, 
    })
}
// 生产者 - 发布消息
func producer() {
    for {
        message := rand.Intn(1000)
        err := client.LPush("queue", message).Err()
        if err != nil {
            panic(err)
        }
        fmt.Println("pushed", message)
        time.Sleep(1 * time.Second)
    }
}
// 消费者 - 处理消息
func consumer(id int) {
    for {
        message, err := client.BRPop(0, "queue").Result()
        if err != nil {
            panic(err)
        }
        fmt.Printf("consumer%d popped %s \n", id, message[1])
        time.Sleep(500 * time.Millisecond)
    }
}
func main() {
    // 初始化
    initClient()
    // 生产者goroutine
    go producer()
    // 3个消费者goroutine
    for i := 0; i < 3; i++ {
        go consumer(i)
    }
    // 阻塞主goroutine
    <-make(chan struct{}) 
}

```

*   使用 BRPop 实现阻塞式出队, LPush 入队, 可以构建基于 Redis 的消息队列。
    
*   多个消费者可以共享队列实现负载均衡
    

### pipeline 访问

Redis Pipeline 实现了一种批量发送请求和响应的模式, 它允许客户端在单次网络交互中缓冲并发送多个命令, 然后再次单次接收所有响应。这种方式极大地减少了客户端和服务器之间的网络往返次数, 优化了网络传输开销，显著提升 Redis 的总体吞吐量和命令处理性能。测试结果表明, Pipeline 模式可以使 Redis 命令处理的 TPS 吞吐量提升数倍。同时它也减轻了客户端等待响应的时间, 有效隐藏了网络通信时延。

此外, Pipeline 模式下可轻松传输更大的数据包, 避免小包命令多次网络传输的资源消耗。它还可以减少客户端和服务器端的 CPU 和内存占用。

总之, Redis Pipeline 通过优化网络传输、批量命令执行等手段, 极大地提升了 Redis 的性能, 是非常重要的客户端访问优化方式。它尤其适合用于高负载、低延迟的 Redis 访问场景。

```
func pipelineDemo(client *redis.Client) {
    // 创建pipeline
    pipe := client.Pipeline()
    // 添加多个命令到pipeline
    setCmd := pipe.Set("key1", "value1", 0)
    getCmd := pipe.Get("key1")
    incrCmd := pipe.Incr("counter")
    pipe.Expire("key1", time.Hour)
    // 执行pipeline
    _, err := pipe.Exec()
    if err != nil {
        panic(err)
    }
    fmt.Println("setCmd:", setCmd.Val())
    fmt.Println("getCmd:", getCmd.Val())
    fmt.Println("incrCmd:", incrCmd.Val())
}

```

pipeline 通过 redis.Pipeline() 创建, 使用管道对象添加命令, 最后调用 Exec() 执行。

*   返回的 cmds 包含每个命令的响应, 可以依次处理。
    
*   这样就可以批量的发送多个命令, 优化访问 Redis 服务器。
    

### 访问 Redis 集群

```
func clusterDemo() {
    // 集群节点，可以填一个或者多个
    // go-redis通过部分节点获取整个集群拓扑(所有节点信息)
    client := redis.NewClusterClient(&redis.ClusterOptions{
        Addrs: []string{
            "127.0.0.1:7000",
            "127.0.0.1:7001", 
            "127.0.0.1:7002",
        },
    })
    err := client.Set("key", "value", 0).Err()
    if err != nil {
        panic(err)
    }
    val, err := client.Get("key").Result()
    if err != nil {
        panic(err)
    }
    fmt.Println("key", val)
    err = client.Close()
    if err != nil {
        panic(err)
    }
}

```

*   通过 NewClusterClient 创建客户端, 传入集群节点地址。
    
*   然后就可以像使用单机客户端一样, 直接操作集群了。
    
*   go-redis 会自动将请求路由到正确的节点上。
    
*   因此 go-redis 可以非常容易地访问 Redis 集群
    

### 事务 (transaction)

```
func transactionDemo(client *redis.Client) {
  pipe := client.TxPipeline()
  pipe.Set("key1", "value1", 0)
  pipe.Set("key2", "value2", 0)
  _, err := pipe.Exec()
  if err != nil {
    panic(err)
  }
}

```

*   使用 TxPipeline() 创建一个事务管道。
    
*   然后将命令添加到管道中, 最后调用 Exec() 执行。
    
*   这会将所有命令作为一个事务 (transaction) 发送到 Redis 服务器。
    

需要注意的是，Redis 中的事务 (transaction) 不是一个原子操作。它只是一种将多个命令打包然后顺序执行的机制。与关系型数据库的事务不同, Redis 事务中若某个命令执行失败, 后续的命令将不会继续执行, 但是不会回滚整个事务。
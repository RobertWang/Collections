> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/n199y8JEyjuGFwUEyH2XVg)

Asynq[1] 是一个 Go 实现的分布式任务队列和异步处理库，基于 redis，类似 Ruby 的 sidekiq[2] 和 Python 的 celery[3]。Go 生态类似的还有 machinery[4] 和 goworker

同时提供一个 WebUI asynqmon[5]，可以源码形式安装或使用 Docker image, 还可以和 Prometheus 集成

`docker run --rm --name asynqmon -p 8080:8080 hibiken/asynqmon`, 如果使用的是主机上的 redis，还需加上 `--redis-addr=host.docker.internal:6379`, 否则会报错 [6]

即 `docker run --rm --name asynqmon -p 8080:8080 hibiken/asynqmon --redis-addr=host.docker.internal:6379`

  

```
➜  asynq-demo git:(main) ✗ tree
.
├── client.go
├── const.go
├── go.mod
├── go.sum
└── server.go

0 directories, 5 files


```

其中 const.go:

```
package main

const (
 redisAddr   = "127.0.0.1:6379"
 redisPasswd = ""
)

const (
 TypeExampleTask    = "shuang:asynq-task:example"
)


```

  

client.go:

```
package main

import (
 "encoding/json"
 "fmt"
 "log"
 "time"

 "github.com/hibiken/asynq"
)

type ExampleTaskPayload struct {
 UserID string
 Msg    string
 // 业务需要的其他字段
}

func NewExampleTask(userID string, msg string) (*asynq.Task, error) {
 payload, err := json.Marshal(ExampleTaskPayload{UserID: userID, Msg: msg})
 if err != nil {
  return nil, err
 }
 return asynq.NewTask(TypeExampleTask, payload), nil
}

var client *asynq.Client

func main() {

 client = asynq.NewClient(asynq.RedisClientOpt{Addr: redisAddr, Password: redisPasswd, DB: 0})
 defer client.Close()

 //go startExampleTask()
 startExampleTask()

 //startGithubUpdate() // 定时触发
}

func startExampleTask() {

 fmt.Println("开始执行一次性的任务")
 // 立刻执行
 task1, err := NewExampleTask("10001", "mashangzhixing!")
 if err != nil {
  log.Fatalf("could not create task: %v", err)
 }

 info, err := client.Enqueue(task1)
 if err != nil {
  log.Fatalf("could not enqueue task: %v", err)
 }
 log.Printf("task1 -> enqueued task: id=%s queue=%s", info.ID, info.Queue)

 // 10秒后执行(定时执行)
 task2, err := NewExampleTask("10002", "10s houzhixing")
 if err != nil {
  log.Fatalf("could not create task: %v", err)
 }

 info, err = client.Enqueue(task2, asynq.ProcessIn(10*time.Second))
 if err != nil {
  log.Fatalf("could not enqueue task: %v", err)
 }
 log.Printf("task2 -> enqueued task: id=%s queue=%s", info.ID, info.Queue)

 // 30s后执行(定时执行)
 task3, err := NewExampleTask("10003", "30s houzhixing")
 if err != nil {
  log.Fatalf("could not create task: %v", err)
 }

 theTime := time.Now().Add(30 * time.Second)
 info, err = client.Enqueue(task3, asynq.ProcessAt(theTime))
 if err != nil {
  log.Fatalf("could not enqueue task: %v", err)
 }
 log.Printf("task3 -> enqueued task: id=%s queue=%s", info.ID, info.Queue)
}


```

  

server.go:

```
package main

import (
 "context"
 "encoding/json"
 "fmt"
 "time"

 "github.com/davecgh/go-spew/spew"
 "github.com/hibiken/asynq"
)

var AsynqServer *asynq.Server // 异步任务server

func initTaskServer() error {
 // 初始化异步任务服务端
 AsynqServer = asynq.NewServer(
  asynq.RedisClientOpt{
   Addr:     redisAddr,
   Password: redisPasswd, //与client对应
   DB:       0,
  },
  asynq.Config{
   // Specify how many concurrent workers to use
   Concurrency: 100,
   // Optionally specify multiple queues with different priority.
   Queues: map[string]int{
    "critical": 6,
    "default":  3,
    "low":      1,
   },
   // See the godoc for other configuration options
  },
 )
 return nil
}

func main() {
 initTaskServer()
 mux := asynq.NewServeMux()

 mux.HandleFunc(TypeExampleTask, HandleExampleTask)
 // ...register other handlers...

 if err := AsynqServer.Run(mux); err != nil {
  fmt.Printf("could not run asynq server: %v", err)
 }
}

func HandleExampleTask(ctx context.Context, t *asynq.Task) error {

 res := make(map[string]string)

 spew.Dump("t.Payload() is:", t.Payload())
 err := json.Unmarshal(t.Payload(), &res)
 if err != nil {
  fmt.Printf("rum session, can not parse payload: %s,  err: %v", t.Payload(), err)
  return nil
 }
 //-----------具体处理逻辑------------
 spew.Println("拿到的入参为:", res, "接下来将进行具体处理")
 fmt.Println()
 // 模拟具体的处理
 time.Sleep(5 * time.Second)
 fmt.Println("--------------处理了5s，处理完成-----------------")

 return nil

}


```

  

执行`redis-server`

  

清除 redis 中所有的 key：

  

执行`docker run --rm --name asynqmon -p 8080:8080 hibiken/asynqmon --redis-addr=host.docker.internal:6379`

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Q1ZRKiavkSiaCk0UtEJeJlYNSqqQy4JvTcLDSS6ElYSf4nDHEOeIYhTQQ50g7CgicibOqwykPReEm9ybvmmkSic32nA/640?wx_fmt=png)

执行 `go run client.go const.go` (生产者，产生消息放入队列)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Q1ZRKiavkSiaCk0UtEJeJlYNSqqQy4JvTcOFN8ib3BOLLe1dicPviay7gKFQtRevL9uMNyicUZJCCFoBaxhEmaBl01CA/640?wx_fmt=png)

此时能看到 redis 中多个几个 key

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Q1ZRKiavkSiaCk0UtEJeJlYNSqqQy4JvTcMLV4UibCqP2Gc8SH2HHEbDIJuGZwM0xGlnJiaoGqYLnzSty3HThRHk2A/640?wx_fmt=png)

同时管理后台能看到队列的信息

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Q1ZRKiavkSiaCk0UtEJeJlYNSqqQy4JvTcDZpJE6QdOF03wfIngAbh3x7ZnoFbeNxzuSVJ0S67yAXibp75nlrupMg/640?wx_fmt=png)  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/Q1ZRKiavkSiaCk0UtEJeJlYNSqqQy4JvTcGZcP2iaiaAzCntlrVicpggxdXV0eBxCBpu8JF0Gl1FJYrrDmribAkicguRg/640?wx_fmt=png)

可以看到都被处理了

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Q1ZRKiavkSiaCk0UtEJeJlYNSqqQy4JvTc9z3awxhzHMNWRVNsmagQvicul198fnibGQsyBXWgX71002wnImlhU7ibA/640?wx_fmt=png)

此时 redis 中的 key：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Q1ZRKiavkSiaCk0UtEJeJlYNSqqQy4JvTcYBXoFEgD8JI9ZPicMLibUCEyibmUx6OVKImO5NY4w3XjiclISov9liavkSQ/640?wx_fmt=png)  

此处的业务处理为模拟，实际可能是某个被触发后不需要马上执行的操作

  

* * *

  
  

```
 // 初始化异步任务服务端
 AsynqServer = asynq.NewServer(
  asynq.RedisClientOpt{
   Addr:     redisAddr,
   Password: redisPasswd, //与client对应
   DB:       0,
  },
  asynq.Config{
   // Specify how many concurrent workers to use
   Concurrency: 100,
   // Optionally specify multiple queues with different priority.
   Queues: map[string]int{
    "critical": 6,//关键队列中的任务将被处理 60% 的时间
    "default":  3,//默认队列中的任务将被处理 30% 的时间
    "low":      1,//低队列中的任务将被处理 10% 的时间
   },
   // See the godoc for other configuration options
  },
 )


```

  

### 参考资料

[1]

Asynq: https://github.com/hibiken/asynq

[2]

sidekiq: https://github.com/sidekiq/sidekiq

[3]

celery: https://github.com/celery/celery

[4]

machinery: https://blog.csdn.net/weixin_42681866/article/details/123334654

[5]

asynqmon: https://github.com/hibiken/asynqmon

[6]

报错: https://github.com/hibiken/asynqmon/issues/214

[7]

完整 Demo: https://github.com/cuishuang/asynq-demo

[8]

asynq 队列如何配置队列优先级: https://blog.csdn.net/itopit/article/details/126123626

[9]

go asynq 异步任务 (延迟触发) 简单案例及奇怪的错误: https://my.oschina.net/randolphcyg/blog/5539676
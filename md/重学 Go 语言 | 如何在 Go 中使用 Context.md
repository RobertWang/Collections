> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/oBR-iozVc_50dNzhidmbug)

> 重学 Go 语言 | 如何在 Go 中使用 Context

我们知道在开发 Go 应用程序时，尤其是网络应用程序，需要启动大量的`Goroutine`来处理请求：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/gz1ggv1GsJXgJxd1jQQ8Q8ibA3wReEicicC7EYZ8NhibpQMcURC4Vdtu9WPPTNVqRCxoajb6BpnZlEEf5LwQo1nsIg/640?wx_fmt=png)

不过`Goroutine`被创建之后，除非执行后正常退出或者触发`panic`退出，Go 并没有提供在一个`Goroutine`中关闭另一个`Goroutine`的机制。

有没有一种方法，可以从一个`Goroutine`中通知另一个`Goroutine`退出执行呢？这时候就该`Context`登场了！

什么是 Context？
------------

`Context`，中文叫做`上下文`，Go 语言在`1.7`版本中新增的`context`包中定义了`Context`，`Context`本质是一个接口，这个接口一共定义了四个方法：

```
type Context interface {
 Deadline() (deadline time.Time, ok bool)
 Done() <-chan struct{}
 Err() error
 Value(key any) any
}


```

*   `Dateline()`：获取定时关闭的时间。
    
*   `Done()`：从一个`channel`获取关闭的信号
    
*   `Err()`：获取错误信息。
    
*   `Value()`：根据`key`从`Context`取值
    

为什么要使用 Context？
---------------

为什么要使用`Context`？或者说设计`Context`的目的是什么？

想像一下这样的场景，当用户开启浏览器访问我们的`Web`服务时，我们可能会开启多个`Goroutine`来处理用户的请求，这些`Goroutine`需要读取不同的资源，最终返回给用户，但如果用户在我们的 Web 服务还没处理完成就关闭浏览器，断开连接，而此时 Web 不知道用户已经关闭请求，仍然在处理并返回最终并不会被接收的数据。

`Context`设计的目的就是可以从上游的`Goroutine`发送信息给下游的`Goroutine`，回到处理用户请求的场景，当处理请求的`Goroutine`发现用户断开连接时，通过`Context`发送停止执行的信息，而下游的`Goroutine`得到停止信号时就返回，避免资源的浪费。

概括起来，`Context`的作用主要体现在两个方面：

*   在`Goroutine`之间传递关闭信息，定时关闭，超时关闭，手动关闭。
    
*   在`Goroutine`之间传递数据。
    

Context 的使用
-----------

下面我们来讲讲`Context`的基本使用。

### 创建 Context

任何上下文都是从一个空白的`Context`开始的，创建一个空白的`Context`有两种方式：

*   使用`context.Background()`：
    

```
ctx := context.Backgroud()


```

*   使用`contenxt.TODO()`:
    

```
ctx := context.TODO()


```

当然大部分时候，我们不需要自己创建一个空白的`Context`，比如在处理`HTTP`请求时：

```
http.HandleFunc("/hello", func(w http.ResponseWriter, r *http.Request) {
 ctx := r.Context()
})


```

上面的代码中，可以从`http.Request`中获取`Context`实例，而`http.Request`的实际上也是调用`context.Backgroud()`:

```
//net/http/request.go
func (r *Request) Context() context.Context {
 if r.ctx != nil {
  return r.ctx
 }
 return context.Background()
}


```

### 实例讲解

为了讲解`Context`是如何在`Goroutine`之间传递信号与数据，我们通过下面的案例进行说明：

```
func ReadFile(ctx context.Context, fileName string, result chan<- []byte) {
 file, err := os.Open(fileName)
 if err != nil {
  return
 }
 totalResult := make([]byte, 0)
 for {
  select {
  case <-ctx.Done():
   result <- []byte{}
   return
  default:
  }
  b := make([]byte, 1024)
  _, err := file.Read(b)
  if err == io.EOF {
   totalResult = append(totalResult, b...)
   break
  }
  totalResult = append(totalResult, b...)
 }

 result <- totalResult
}


```

在上面的程序中，我们调用`Context`的`Done()`方法，该方法会返回一个`Channel`，而使用`select`语句则可以让我们在处理业务的同时，监听上游`Goroutine`是否有传递取消执行的信息。

#### 利用 Context 传递停止信号

空白的`Context`并不能发挥什么作用，要达到手动取消执行的目的，需要调用`context`包下的`WithCancel`函数进行封装，封装返回一个新的`context`以及一个取消的句柄`cancel`函数：

```
ctx := context.Background()
ctx, cancel := context.WithCancel(ctx)


```

下面是完整的使用方法：

```
package main

import (
 "context"
 "fmt"
 "io"
 "os"
 "time"
)

func main() {
 ctx := context.Background()
 ctx, cancel := context.WithCancel(ctx)
 result := make(chan []byte)
    
    
 go ReadFile(ctx, "./test.tar", result)

 time.Sleep(100 * time.Millisecond)
 cancel()


 fmt.Println(<-result)
}


```

在上面的程序中，我们创建一个`Context`之后，传给了`ReadFile`函数，并且在暂停 100 毫秒后调用`cancel()`函数，达到手动取消另一个`Goroutine`的目的。

#### 给 Context 设置一个截止时间

除了手动取消，也可以调用`context`包下的`WithDeadline()`函数给`Context`加一个截止时间，这样在某个时间点，`Context`会自动发出取消信号：

```
afterTime := time.Now().Add(30 * time.Millisecond)
ctx, cancel := context.WithDeadline(context.Background(), afterTime)


```

下面是完整的示例：

```
package main

import (
 "context"
 "fmt"
 "io"
 "os"
 "time"
)

func main() {
 afterTime := time.Now().Add(30 * time.Millisecond)
 ctx, cancel := context.WithDeadline(context.Background(), afterTime)
 defer cancel()
 result := make(chan []byte)
 go ReadFile(ctx, "./test.tar", result)
 fmt.Println(<-result)
}


```

#### 给 Context 一个超时限制

调用`context`包下的`WithTimeout()`函数可以为`Context`加一个超时限制，这对于我们编写超时控制程序非常有帮助：

```
ctx, cancel := context.WithTimeout(context.Background(), 10*time.Millisecond)


```

下面是完整的调用程序：

```
package main

import (
 "context"
 "fmt"
 "io"
 "os"
 "time"
)

func main() {
 ctx, cancel := context.WithTimeout(context.Background(), 10*time.Millisecond)
 defer cancel()
 result := make(chan []byte)
 go ReadFile(ctx, "./test.tar", result)
 fmt.Println(<-result)
}


```

#### 使用 Context 传递数据

调用`context`包下的`WithValue()`函数可以生成一个携带数据的`Context`，这个机制方便我们跟踪一个处理流程中的`Goroutine`：

```
 ctx, cancel := context.WithValue(context.Background(), "testKey","testValue")


```

下游的`Goroutine`就可以通过`Context`的`Value()`函数来获取上游传递下来的值了。

```
func MyGoroutine(ctx context.Context){
 ctx.Value("testKey")
}


```

使用 Context 的几点建议
----------------

*   `Context`不要放在结构体中，需要以参数方式传递。
    
*   `Context`作为函数参数时，一般放在第一位，作为函数的第一个参数。
    
*   使用 `context.Background`函数生成根节点的`Context`。
    
*   `Context` 要传值必要的值，不要什么都传。
    
*   `Context` 是多协程安全的，可以在多个协程中使用。
    

小结
--

好了，至此，你应该对`Context`有所了解了吧，总的来说，通过`Context`可以做到：

*   手动控制下游`Goroutine`取消执行。
    
*   定时控制下游`Goroutine`取消执行。
    
*   超时控制下游`Goroutine`取消执行。
    
*   传递数据给下游的`Goroutine`。
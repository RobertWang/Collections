> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5ODc5ODgyMw==&mid=2653587805&idx=1&sn=f6ef4a4d432a8e041fc4db731f9ba1e5&chksm=bd1b1dd58a6c94c32cd8b1771e8a88364c4ffd7b522446c56a4b03cd7c9c70f9ab1099a372cc&scene=21#wechat_redirect)

想必只要是熟悉 Python 的同学对装饰模式一定不会陌生，这类 Python 从语法上原生支持的装饰器，大大提高了装饰模式在 Python 中的应用。尽管 Go 语言中装饰模式没有 Python 中应用的那么广泛，但是它也有其独到的地方。接下来就一起看下装饰模式在 Go 语言中的应用。

  

**简单装饰器**

  

我们通过一个简单的例子来看一下装饰器的简单应用，首先编写一个 hello 函数：  

```
package main

import "fmt"

func hello() {
    fmt.Println("Hello World!")
}

func main() {
    hello()
}
```

完成上面代码后，执行会输出 “Hello World!”。接下来通过以下方式，在打印 “Hello World!” 前后各加一行日志：  

```
package main

import "fmt"

func hello() {
    fmt.Println("before")
    fmt.Println("Hello World!")
    fmt.Println("after")
}

func main() {
    hello()
}
```

代码执行后输出：  

```
before
Hello World!
after
```

当然我们可以选择一个更好的实现方式，即单独编写一个专门用来打印日志的 logger 函数，示例如下：  

```
package main

import "fmt"

func logger(f func()) func() {
    return func() {
        fmt.Println("before")
        f()
        fmt.Println("after")
    }
}

func hello() {
    fmt.Println("Hello World!")
}

func main() {
    hello := logger(hello)
    hello()
}
```

可以看到 logger 函数接收并返回了一个函数，且参数和返回值的函数签名同 hello 一样。然后我们在原来调用 hello() 的位置进行如下修改：  

```
hello := logger(hello)
hello()
```

这样我们通过 logger 函数对 hello 函数的包装，更加优雅的实现了给 hello 函数增加日志的功能。执行后的打印结果仍为：  

```
before
Hello World!
after
```

其实 logger 函数也就是我们在 Python 中经常使用的装饰器，因为 logger 函数不仅可以用于 hello，还可以用于其他任何与 hello 函数有着同样签名的函数。  

当然如果想使用 Python 中装饰器的写法，我们可以这样做：

```
package main

import "fmt"

func logger(f func()) func() {
    return func() {
        fmt.Println("before")
        f()
        fmt.Println("after")
    }
}

// 给 hello 函数打上 logger 装饰器
@logger
func hello() {
    fmt.Println("Hello World!")
}

func main() {
    // hello 函数调用方式不变
    hello()
}
```

但很遗憾，上面的程序无法通过编译。因为 Go 语言目前还没有像 Python 语言一样从语法层面提供对装饰器语法糖的支持。  

  

**装饰器实现中间件**

  

尽管 Go 语言中装饰器的写法不如 Python 语言精简，但它被广泛运用于 Web 开发场景的中间件组件中。比如 Gin Web 框架的如下代码，只要使用过就肯定会觉得熟悉：

```
package main

import "github.com/gin-gonic/gin"

func main() {
    r := gin.New()

    // 使用中间件
    r.Use(gin.Logger(), gin.Recovery())

    r.GET("/ping", func(c *gin.Context) {
        c.JSON(200, gin.H{
            "message": "pong",
        })
    })
    _ = r.Run(":8888")
}
```

如示例中使用 gin.Logger() 增加日志，使用 gin.Recovery() 来处理 panic 异常一样，在 Gin 框架中可以通过 r.Use(middlewares...) 的方式给路由增加非常多的中间件，来方便我们拦截路由处理函数，并在其前后分别做一些处理逻辑。  

而 Gin 框架的中间件正是使用装饰模式来实现的。下面我们借用 Go 语言自带的 http 库进行一个简单模拟。这是一个简单的 Web Server 程序，其监听 8888 端口，当访问 /hello 路由时会进入 handleHello 函数逻辑：

```
package main

import (
    "fmt"
    "net/http"
)

func loggerMiddleware(f http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        fmt.Println("before")
        f(w, r)
        fmt.Println("after")
    }
}

func authMiddleware(f http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        if token := r.Header.Get("token"); token != "fake_token" {
            _, _ = w.Write([]byte("unauthorized\n"))
            return
        }
        f(w, r)
    }
}

func handleHello(w http.ResponseWriter, r *http.Request) {
    fmt.Println("handle hello")
    _, _ = w.Write([]byte("Hello World!\n"))
}

func main() {
    http.HandleFunc("/hello", authMiddleware(loggerMiddleware(handleHello)))
    fmt.Println(http.ListenAndServe(":8888", nil))
}
```

我们分别使用 loggerMiddleware、authMiddleware 函数对 handleHello 进行了包装，使其支持打印访问日志和认证校验功能。如果我们还需要加入其他中间件拦截功能，可以通过这种方式进行无限包装。

启动这个 Server 来验证下装饰器：

![](https://mmbiz.qpic.cn/mmbiz_png/OdIoEOgFgUEzTWPvOLpB170qFNyzNL7wIxjGKUjGNHtdpGO1Be6IdzicrUs1ibMC47fyGxDtwCvvPxUVxjzT4eHg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/OdIoEOgFgUEzTWPvOLpB170qFNyzNL7wwwkKgsDsEbWEnE6YiamYEVvZ8jWxbYqWM8VdZgSibHH1F82zTo8dop3w/640?wx_fmt=png)

对结果进行简单分析可以看到，第一次请求 /hello 接口时，由于没有携带认证 token，收到了 unauthorized 响应。第二次请求时携带了 token，则得到响应 “Hello World!”，并且后台程序打印如下日志：

```
before
handle hello
after
```

这说明中间件执行顺序是先由外向内进入，再由内向外返回。而这种一层一层包装处理逻辑的模型有一个非常形象且贴切的名字，洋葱模型。  

![](https://mmbiz.qpic.cn/mmbiz_png/OdIoEOgFgUEzTWPvOLpB170qFNyzNL7w5x0sU7v0N1iatsbibdeNROaQicoYf1FjB0W8SIyknVYM3dnoyrazeOK3g/640?wx_fmt=png)

但用洋葱模型实现的中间件有一个直观的问题。相比于 Gin 框架的中间件写法，这种一层层包裹函数的写法不如 Gin 框架提供的 r.Use(middlewares...) 写法直观。

Gin 框架源码的中间件和 handler 处理函数实际上被一起聚合到了路由节点的 handlers 属性中。其中 handlers 属性是 HandlerFunc 类型切片。对应到用 http 标准库实现的 Web Server 中，就是满足 func(ResponseWriter, *Request) 类型的 handler 切片。

当路由接口被调用时，Gin 框架就会像流水线一样依次调用执行 handlers 切片中的所有函数，再依次返回。这种思想也有一个形象的名字，就叫作流水线（Pipeline）。

![](https://mmbiz.qpic.cn/mmbiz_png/OdIoEOgFgUEzTWPvOLpB170qFNyzNL7wasfaKia3ynUZU0zX1IFyKoq1HeibicGvEBG2CpxiaOdOaxic2YfAdnibKicnA/640?wx_fmt=png)

接下来我们要做的就是将 handleHello 和两个中间件 loggerMiddleware、authMiddleware 聚合到一起，同样形成一个 Pipeline。

```
package main

import (
    "fmt"
    "net/http"
)

func authMiddleware(f http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        if token := r.Header.Get("token"); token != "fake_token" {
            _, _ = w.Write([]byte("unauthorized\n"))
            return
        }
        f(w, r)
    }
}

func loggerMiddleware(f http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        fmt.Println("before")
        f(w, r)
        fmt.Println("after")
    }
}

type handler func(http.HandlerFunc) http.HandlerFunc

// 聚合 handler 和 middleware
func pipelineHandlers(h http.HandlerFunc, hs ...handler) http.HandlerFunc {
    for i := range hs {
        h = hs[i](h)
    }
    return h
}

func handleHello(w http.ResponseWriter, r *http.Request) {
    fmt.Println("handle hello")
    _, _ = w.Write([]byte("Hello World!\n"))
}

func main() {
    http.HandleFunc("/hello", pipelineHandlers(handleHello, loggerMiddleware, authMiddleware))
    fmt.Println(http.ListenAndServe(":8888", nil))
}
```

我们借用 pipelineHandlers 函数将 handler 和 middleware 聚合到一起，实现了让这个简单的 Web Server 中间件用法跟 Gin 框架用法相似的效果。

再次启动 Server 进行验证：

![](https://mmbiz.qpic.cn/mmbiz_png/OdIoEOgFgUEzTWPvOLpB170qFNyzNL7w37kO2VibIcTTdEiagw74LV5EhNP7uTkMaZT8htuP9QZ2cODnqC4SVnYA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/OdIoEOgFgUEzTWPvOLpB170qFNyzNL7wYABbEoialibKJ1aEGR6AcwUOn6sac3lXG14wq22lMaC6o1X8Sicf0jnsw/640?wx_fmt=png)

改造成功，跟之前使用洋葱模型写法的结果如出一辙。

  

**总结**

  

简单了解了 Go 语言中如何实现装饰模式后，我们通过一个 Web Server 程序中间件，学习了装饰模式在 Go 语言中的应用。  

需要注意的是，尽管 Go 语言实现的装饰器有类型上的限制，不如 Python 装饰器那般通用。就像我们最终实现的 pipelineHandlers 不如 Gin 框架中间件强大，比如不能延迟调用，通过 c.Next() 控制中间件调用流等。但不能因为这样就放弃，因为 GO 语言装饰器依然有它的用武之地。

Go 语言是静态类型语言不像 Python 那般灵活，所以在实现上要多费一点力气。希望通过这个简单的示例，相信对大家深入学习 Gin 框架有所帮助。
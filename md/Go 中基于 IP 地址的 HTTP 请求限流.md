> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/410133019)

如果你在运行 HTTP 服务并且想对 endpoints 进行限速，你可以使用维护良好的工具，例如 **[github.com/didip/tollbooth](https://link.zhihu.com/?target=https%3A//github.com/didip/tollbooth)**。但是如果你在构建一些非常简单的东西，自己实现并不困难。

我们可以使用已经存在的试验性的 Go 包 `x/time/rate`。

在本教程中，我们将创建一个基于用户 IP 地址进行速率限制的简单的中间件。

**「干净的」HTTP 服务**
----------------

让我们从构建一个简单的 HTTP 服务开始，该服务具有非常简单的 endpiont。这可能是个非常「重」的 endpoint，因此我们想在这里添加速率限制。

```
package main

import (
    "log"
    "net/http"
)

func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("/", okHandler)

    if err := http.ListenAndServe(":8888", mux); err != nil {

    // log.Fatalf("unable to start server: %s", err.Error())
    }
}

func okHandler(w http.ResponseWriter, r *http.Request) {
    // Some very expensive database call
    w.Write([]byte("alles gut"))
}


```

在 `main.go` 中，我们启动服务，该服务监听 `:8888` 并拥有一个 endpoint `/`。

**[http://golang.org/x/time/rate](https://link.zhihu.com/?target=http%3A//golang.org/x/time/rate)**
---------------------------------------------------------------------------------------------------

我们将使用 Go 中 `x/time/rate` 包，该包提供了[令牌桶限速算法](https://www.zhihu.com/search?q=%E4%BB%A4%E7%89%8C%E6%A1%B6%E9%99%90%E9%80%9F%E7%AE%97%E6%B3%95&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22article%22%2C%22sourceId%22%3A410133019%7D)。**[rate#Limiter](https://link.zhihu.com/?target=https%3A//godoc.org/golang.org/x/time/rate%23Limiter)** 控制事件发生的频率。它实现了一个大小为 `b` 的「令牌桶」，初始化时是满的，并且以每秒 `r` 个[令牌](https://www.zhihu.com/search?q=%E4%BB%A4%E7%89%8C&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22article%22%2C%22sourceId%22%3A410133019%7D)的速率重新填充。非正式地，在任意足够长的时间间隔中，限速器将速率限制在每秒 r 个令牌，最大突发事件为 b 个。

既然我们想要实现基于 IP 地址的限速器，我们还需要维护一个限速器的 map。

```
package main

import (
    "sync"

    "golang.org/x/time/rate"
)

// IPRateLimiter .
type IPRateLimiter struct {
    ips map[string]*rate.Limiter
    mu  *sync.RWMutex
    r   rate.Limit
    b   int
}

// NewIPRateLimiter .
func NewIPRateLimiter(r rate.Limit, b int) *IPRateLimiter {
    i := &IPRateLimiter{
        ips: make(map[string]*rate.Limiter),
        mu:  &sync.RWMutex{},
        r:   r,
        b:   b,
    }

    return i
}

// AddIP creates a new rate limiter and adds it to the ips map,
// using the IP address as the key
func (i *IPRateLimiter) AddIP(ip string) *rate.Limiter {
    i.mu.Lock()
    defer i.mu.Unlock()

    limiter := rate.NewLimiter(i.r, i.b)

    i.ips[ip] = limiter

    return limiter
}

// GetLimiter returns the rate limiter for the provided IP address if it exists.
// Otherwise calls AddIP to add IP address to the map
func (i *IPRateLimiter) GetLimiter(ip string) *rate.Limiter {
    i.mu.Lock()
    limiter, exists := i.ips[ip]

    if !exists {
        i.mu.Unlock()
        return i.AddIP(ip)
    }

    i.mu.Unlock()

    return limiter
}


```

`NewIPRateLimiter` 创建了一个 IP 限速器的实例，HTTP 服务需要调用该实例的 `GetLimiter` 方法去获取特定 IP 的限速器（从 map 中获取，或者生成一个新的）。

**[中间件](https://www.zhihu.com/search?q=%E4%B8%AD%E9%97%B4%E4%BB%B6&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22article%22%2C%22sourceId%22%3A410133019%7D)**
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

让我们升级我们的 HTTP 服务，将中间件添加到所有的 endpoints 中，因此，如果某 IP 达到了限制速率，服务将会返回 429 Too Many Requests，否则服务将处理该请求。

在 `limitMiddleware` 函数中，每次中间件收到 HTTP 请求，我们都会调用全局限速器的 `Allow()` 方法。如果令牌桶中没有剩余的令牌，`Allow()` 将返回 false，我们返回给用户 429 Too Many Requests 响应。否则，调用 `Allow()` 将消耗桶中的一个令牌，我们将控制权传递给调用链的下一个处理器。

```
package main

import (
    "log"
    "net/http"
)

var limiter = NewIPRateLimiter(1, 5)

func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("/", okHandler)

    if err := http.ListenAndServe(":8888", limitMiddleware(mux)); err != nil {
        log.Fatalf("unable to start server: %s", err.Error())
    }
}

func limitMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        limiter := limiter.GetLimiter(r.RemoteAddr)
        if !limiter.Allow() {
            http.Error(w, http.StatusText(http.StatusTooManyRequests), http.StatusTooManyRequests)
            return
        }

        next.ServeHTTP(w, r)
    })
}

func okHandler(w http.ResponseWriter, r *http.Request) {
    // Some very expensive database call
    w.Write([]byte("alles gut"))
}


```

**构建 & 运行**
-----------

```
go get golang.org/x/time/rate
go build -o server .
./server


```

**测试**
------

有一个非常棒的工具称作 vegeta，我喜欢在 HTTP [负载测试](https://www.zhihu.com/search?q=%E8%B4%9F%E8%BD%BD%E6%B5%8B%E8%AF%95&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22article%22%2C%22sourceId%22%3A410133019%7D)中使用（它也是用 Go 编写的）

```
brew install vegeta


```

我们需要创建一个简单的配置文件，声明我们想要发送的请求。

```
GET http://localhost:8888/

```

然后，以每个时间单元 100 个请求的速率攻击 10 秒。

```
vegeta attack -duration=10s -rate=100 -targets=vegeta.conf | vegeta report


```

结果，你将看到一些请求返回 200，但大多数返回 429。

via: [Go 中基于 IP 地址的 HTTP 请求限流](https://link.zhihu.com/?target=https%3A//studygolang.com/articles/33988)

作者：**[ALEX PLIUTAU](https://link.zhihu.com/?target=https%3A//pliutau.com/)** 译者：**[DoubleLuck](https://link.zhihu.com/?target=https%3A//github.com/DoubleLuck)** 校对：**[unknwon](https://link.zhihu.com/?target=https%3A//github.com/unknwon)**

本文由 **[GCTT](https://link.zhihu.com/?target=https%3A//github.com/studygolang/GCTT)** 原创编译，**[Go 语言中文网](https://link.zhihu.com/?target=https%3A//studygolang.com/articles/33988)** 荣誉推出
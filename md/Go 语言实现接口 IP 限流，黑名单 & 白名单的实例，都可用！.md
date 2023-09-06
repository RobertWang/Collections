> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/50fhXJOhiBP6Z5_pai-_cg)

在 Go 语言中，可以使用令牌桶算法（Token Bucket Algorithm）来实现接口 IP 限流。令牌桶算法基于令牌桶的概念，每个令牌代表一个请求，令牌桶限制了请求的速率。

以下是一个简单的示例代码，演示如何使用**令牌桶算法实现接口 IP 限流**：

```
package main

import (
    "fmt"
    "net"
    "sync"
    "time"
)

type RateLimiter struct {
    bucket map[string]*TokenBucket
    mutex  sync.Mutex
}

type TokenBucket struct {
    rate       float64 // 速率，单位：令牌/秒
    capacity   float64 // 令牌桶容量
    tokens     float64 // 当前令牌数量
    lastUpdate time.Time // 上次更新时间
}

func NewRateLimiter() *RateLimiter {
    return &RateLimiter{
        bucket: make(map[string]*TokenBucket),
    }
}

func (rl *RateLimiter) AllowIP(ip string) bool {
    rl.mutex.Lock()
    defer rl.mutex.Unlock()

    bucket, exists := rl.bucket[ip]
    if !exists {
        // 初始化令牌桶
        bucket = &TokenBucket{
            rate:       10, // 每秒生成10个令牌
            capacity:   10, // 令牌桶容量为10个
            tokens:     10, // 初始时令牌桶为满的状态
            lastUpdate: time.Now(),
        }
        rl.bucket[ip] = bucket
    }

    // 计算时间间隔，并根据速率生成令牌
    now := time.Now()
    elapsed := now.Sub(bucket.lastUpdate).Seconds()
    tokensToAdd := elapsed * bucket.rate

    // 更新令牌桶状态
    if tokensToAdd > 0 {
        bucket.tokens = bucket.tokens + tokensToAdd
        if bucket.tokens > bucket.capacity {
            bucket.tokens = bucket.capacity
        }
        bucket.lastUpdate = now
    }

    // 检查令牌数量是否足够
    if bucket.tokens >= 1 {
        bucket.tokens--
        return true
    }

    return false
}

func main() {
    limiter := NewRateLimiter()

    // 模拟并发请求
    for i := 0; i < 20; i++ {
        go func() {
            ip := GetClientIP() // 获取客户端IP
            if limiter.AllowIP(ip) {
                fmt.Printf("Request from IP %s is allowed\n", ip)
            } else {
                fmt.Printf("Request from IP %s is rate limited\n", ip)
            }
        }()
    }

    // 等待所有请求完成
    time.Sleep(2 * time.Second)
}

func GetClientIP() string {
    conn, _ := net.Dial("udp", "8.8.8.8:80")
    defer conn.Close()
    localAddr := conn.LocalAddr().(*net.UDPAddr)
    return localAddr.IP.String()
}


```

在上述示例中，我们定义了 `RateLimiter` 结构体和 `TokenBucket` 结构体，用于表示 IP 限流的令牌桶和限流器。`RateLimiter` 使用 `sync.Mutex` 来保证并发安全。

通过 `AllowIP` 方法实现 IP 限流逻辑。该方法首先从 `RateLimiter` 中获取对应 IP 的令牌桶，如果令牌桶不存在，则创建一个新的令牌桶。然后根据速率计算时间间隔，并生成相应数量的令牌。最后，检查令牌桶中的令牌数量是否足够，如果足够则返回 `true`，表示允许访问；否则返回 `false`，表示限制访问。

在 `main` 函数中，我们创建了一个 `RateLimiter` 对象，并模拟并发请求。每个请求获取客户端 IP，并通过 `AllowIP` 方法判断是否允许访问。根据令牌桶的速率和容量，前 10 个请求会被允许，后续的请求会被限流。

注意，以上示例代码仅演示了基本的令牌桶算法实现 IP 限流的方法。实际应用中，可能需要考虑更复杂的限流策略，如平滑突发限流、动态调整速率等，以满足具体的需求。

**黑名单 & 白名单的实现**

在 Go 语言中，可以使用 map 数据结构来实现 IP 黑名单和 IP 白名单的功能。以下是一个示例代码，演示如何实现 IP 黑名单和 IP 白名单：

```
package main

import (
    "fmt"
    "net"
)

type IPList struct {
    list map[string]bool
}

func NewIPList() *IPList {
    return &IPList{
        list: make(map[string]bool),
    }
}

func (l *IPList) AddIP(ip string) {
    l.list[ip] = true
}

func (l *IPList) RemoveIP(ip string) {
    delete(l.list, ip)
}

func (l *IPList) ContainsIP(ip string) bool {
    _, exists := l.list[ip]
    return exists
}

func main() {
    // 创建IP黑名单
    blacklist := NewIPList()

    // 添加IP到黑名单
    blacklist.AddIP("127.0.0.1")
    blacklist.AddIP("192.168.0.1")

    // 模拟请求，判断IP是否在黑名单中
    ips := []string{"127.0.0.1", "192.168.0.1", "10.0.0.1"}

    for _, ip := range ips {
        if blacklist.ContainsIP(ip) {
            fmt.Printf("IP %s is in the blacklist\n", ip)
        } else {
            fmt.Printf("IP %s is not in the blacklist\n", ip)
        }
    }
}


```

在上述示例中，我们定义了一个名为 `IPList` 的结构体，用于表示 IP 名单列表。该结构体内部使用 map 来存储 IP，并提供了添加 IP、移除 IP 和判断 IP 是否存在的方法。

在 `main` 函数中，我们创建了一个 `IPList` 对象，并添加一些 IP 到黑名单中。然后，我们模拟请求并判断每个 IP 是否在黑名单中，根据结果输出相应的消息。

如果需要实现 IP 白名单，可以类似地创建一个 `IPList` 结构体，但将逻辑调整为判断 IP 是否在白名单中。然后使用 `AddIP` 方法添加白名单 IP，使用 `ContainsIP` 方法判断 IP 是否在白名单中。

在实际应用中，可以根据需求扩展 `IPList` 结构体的功能，如支持批量添加 IP、从文件中加载名单等。此外，还可以结合网络请求的 IP 获取方法，获取客户端 IP 并进行名单判断。
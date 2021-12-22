> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cyhone.com](https://www.cyhone.com/articles/analysis-of-uber-go-ratelimit/)

> 本文属于 《Golang 源码剖析系列》 uber 在 Github 上开源了一套用于服务限流的 go 语言库 ratelimit, 该组件基于 Leaky Bucket(漏桶) 实现。

uber 在 Github 上开源了一套用于服务限流的 go 语言库 [ratelimit](https://github.com/uber-go/ratelimit/), 该组件基于 Leaky Bucket(漏桶) 实现。

我在之前写过一篇 [《Golang 限流器 time/rate 实现剖析》](https://www.cyhone.com/articles/analisys-of-golang-rate/)，分析了 Golang 标准库中基于 Token Bucket 实现限流组件的 `time/rate` 原理，同时也讲了限流的一些背景。

相比于 TokenBucket 中，只要桶内还有剩余令牌，调用方就可以一直消费的策略。Leaky Bucket 相对来说更加严格，调用方只能严格按照预定的间隔顺序进行消费调用。(虽然 uber-go 对这个限制也做了一些优化，具体可以看下文详解)

还是老规矩，在正式讲其实现之前，我们先看下 ratelimit 的使用方法。

我们直接看下 uber-go 官方库给的例子：

```
rl := ratelimit.New(100) // per second

prev := time.Now()
for i := 0; i < 10; i++ {
  now := rl.Take()
  fmt.Println(i, now.Sub(prev))
  prev = now
}

```

在这个例子中，我们给定限流器每秒可以通过 100 个请求，也就是平均每个请求间隔 10ms。  
因此，最终会每 10ms 打印一行数据。输出结果如下：

```
// Output:
// 0 0
// 1 10ms
// 2 10ms
// 3 10ms
// 4 10ms
// 5 10ms
// 6 10ms
// 7 10ms
// 8 10ms
// 9 10ms

```

要实现以上每秒固定速率的目的，其实还是比较简单的。

在 ratelimit 的 New 函数中，传入的参数是每秒允许请求量 (RPS)。  
我们可以很轻易的换算出每个请求之间的间隔：

```
limiter.perRequest = time.Second / time.Duration(rate)

```

以上 `limiter.perRequest` 指的就是每个请求之间的间隔时间。

如下图，当请求 1 处理结束后, 我们记录下请求 1 的处理完成的时刻, 记为 `limiter.last`。  
稍后请求 2 到来, 如果此刻的时间与 `limiter.last` 相比并没有达到 `perRequest` 的间隔大小，那么 sleep 一段时间即可。

![](https://www.cyhone.com/img/token-bucket/wait-interval.png)

对应 ratelimit 的实现代码如下：

```
sleepFor = t.perRequest - now.Sub(t.last)
if sleepFor > 0 {
	t.clock.Sleep(sleepFor)
	t.last = now.Add(sleepFor)
} else {
	t.last = now
}

```

我们讲到，传统的 Leaky Bucket，每个请求的间隔是固定的，然而，在实际上的互联网应用中，流量经常是突发性的。对于这种情况，uber-go 对 Leaky Bucket 做了一些改良，引入了最大松弛量 (maxSlack) 的概念。

我们先理解下整体背景: 假如我们要求每秒限定 100 个请求，平均每个请求间隔 10ms。但是实际情况下，有些请求间隔比较长，有些请求间隔比较短。如下图所示：

![](https://www.cyhone.com/img/token-bucket/3-requests.png)

请求 1 完成后，15ms 后，请求 2 才到来，可以对请求 2 立即处理。请求 2 完成后，5ms 后，请求 3 到来，这个时候距离上次请求还不足 10ms，因此还需要等待 5ms。

但是，对于这种情况，实际上三个请求一共消耗了 25ms 才完成，并不是预期的 20ms。在 uber-go 实现的 ratelimit 中，可以把之前间隔比较长的请求的时间，匀给后面的使用，保证每秒请求数 (RPS) 即可。

对于以上 case，因为请求 2 相当于多等了 5ms，我们可以把这 5ms 移给请求 3 使用。加上请求 3 本身就是 5ms 之后过来的，一共刚好 10ms，所以请求 3 无需等待，直接可以处理。此时三个请求也恰好一共是 20ms。  
如下图所示：

![](https://www.cyhone.com/img/token-bucket/maxslack.png)

在 ratelimit 的对应实现中很简单，是把每个请求多余出来的等待时间累加起来，以给后面的抵消使用。

```
t.sleepFor += t.perRequest - now.Sub(t.last)
if t.sleepFor > 0 {
  t.clock.Sleep(t.sleepFor)
  t.last = now.Add(t.sleepFor)
  t.sleepFor = 0
} else {
  t.last = now
}

```

注意：这里跟上述代码不同的是，这里是 `+=`。而同时 `t.perRequest - now.Sub(t.last)` 是可能为负值的，负值代表请求间隔时间比预期的长。

当 `t.sleepFor > 0`，代表此前的请求多余出来的时间，无法完全抵消此次的所需量，因此需要 sleep 相应时间, 同时将 `t.sleepFor` 置为 0。

当 `t.sleepFor < 0`，说明此次请求间隔大于预期间隔，将多出来的时间累加到 `t.sleepFor` 即可。

但是，对于某种情况，请求 1 完成后，请求 2 过了很久到达 (好几个小时都有可能)，那么此时对于请求 2 的请求间隔 `now.Sub(t.last)`，会非常大。以至于即使后面大量请求瞬时到达，也无法抵消完这个时间。那这样就失去了限流的意义。

为了防止这种情况，ratelimit 就引入了最大松弛量 (maxSlack) 的概念, 该值为负值，表示允许抵消的最长时间，防止以上情况的出现。

```
if t.sleepFor < t.maxSlack {
  t.sleepFor = t.maxSlack
}

```

ratelimit 中 maxSlack 的值为 `-10 * time.Second / time.Duration(rate)`, 是十个请求的间隔大小。我们也可以理解为 ratelimit 允许的最大瞬时请求为 10。

ratelimit 的 New 函数，除了可以配置每秒请求数 (QPS)， 其实还提供了一套可选配置项 Option。

```
func New(rate int, opts ...Option) Limiter

```

Option 的类型为 `type Option func(l *limiter)`, 也就是说我们可以提供一些这样类型的函数，作为 Option，传给 ratelimit, 定制相关需求。

但实际上，自定义 Option 的用处比较小，因为 `limiter` 结构体本身就是个私有类型，我们并不能拿它做任何事情。

我们只需要了解 ratelimit 目前提供的两个配置项即可：

[](#WithoutSlack "WithoutSlack")`WithoutSlack`
----------------------------------------------

我们上文讲到 ratelimit 中引入了最大松弛量的概念，而且默认的最大松弛量为 10 个请求的间隔时间。

但是确实会有这样需求场景，需要严格的限制请求的固定间隔。那么我们就可以利用 WithoutSlack 来取消松弛量的影响。

```
limiter := ratelimit.New(100, ratelimit.WithoutSlack)

```

[](#WithClock-clock-Clock "WithClock(clock Clock)")`WithClock(clock Clock)`
---------------------------------------------------------------------------

我们上文讲到，ratelimit 的实现时，会计算当前时间与上次请求时间的差值，并 sleep 相应时间。  
在 ratelimit 基于 go 标准库的 time 实现时间相关计算。如果有精度更高或者特殊需求的计时场景，可以用 WithClock 来替换默认时钟。

通过该方法，只要实现了 Clock 的 interface，就可以自定义时钟了。

```
type Clock interface {
	Now() time.Time
	Sleep(time.Duration)
}

```

```
clock &= MyClock{}
limiter := ratelimit.New(100, ratelimit.WithClock(clock))

```

*   [Golang 限流器 time/rate 实现剖析](https://www.cyhone.com/articles/analisys-of-golang-rate/)
*   [Golang 限流器 time/rate 使用介绍](https://www.cyhone.com/articles/usage-of-golang-rate/)
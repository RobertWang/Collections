> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [tonybai.com](https://tonybai.com/2022/11/08/understand-go-context-by-example/)

> 原 weibo 账号处于 jy 状态，临时先用小号

![](https://tonybai.com/wp-content/uploads/understand-go-context-by-example-1.png)

*   **原 weibo 账号处于 jy 状态，临时先用[小号](https://weibo.com/u/6484441286) https://weibo.com/u/6484441286，欢迎大家关注！**
*   “Gopher 部落” 知识星球双十一新人特惠，[领劵加入即享立减 88 元优惠](https://t.zsxq.com/078E1QTjM) – https://t.zsxq.com/078E1QTjM

[本文永久链接](https://tonybai.com/2022/11/08/understand-go-context-by-example) – https://tonybai.com/2022/11/08/understand-go-context-by-example

自从 context 包在 [Go 1.7 版本](https://tonybai.com/2016/06/21/some-changes-in-go-1-7)加入 Go 标准库，它就成为了 Go 标准库中较难理解和易误用的包之一。在我的博客中目前尚未有一篇系统介绍 context 包的文章，很多来自 [Go 专栏](http://gk.link/a/10AVZ)或[《Go 语言精进之路》](https://item.jd.com/13694000.html)的读者都希望我能写一篇介绍 context 包的文章，今天我就来尝试一下 ^_^。

### 1. context 包入标准库历程

2014 年，Go 团队核心成员 Sameer Ajmani 在 Go 官博上发表了一篇文章 [“Go Concurrency Patterns: Context”](https://go.dev/blog/context)，介绍了 Google 内部设计和实现的一个名为 context 的包以及该包在 Google 内部实践后得出的一些应用模式。随后，该包被开源并放在 golang.org/x/net/context 下维护。两年后，也就是 2016 年，golang.org/x/net/context 包正式被挪入 Go 标准库，这就是目前 Go 标准库 context 包的诞生历程。

> 历史经验告诉我们：**但凡 Google 内部认为是好东西的，基本上最后都进入到 Go 语言或标准库当中了**。context 包就是其中之一，后续 [Go 1.9 版本加入的 type alias 语法](https://tonybai.com/2017/07/14/some-changes-in-go-1-9/)也印证了这一点。可以预测：即将于 Go 1.20 版本以实验特性身份加入的 [arena 包](https://github.com/golang/go/issues/51317)离最终正式加入 Go 也只是时间问题了 ^_^！

### 2. context 包解决的是什么问题？

**正确定义问题比解决问题更重要**。在 Sameer Ajmani 的文章中，他在一开篇就对引入 context 包要解决的问题做了明确的阐述：

> 在 Go 服务器中，每个传入的请求都在自己的 goroutine 中处理。请求的处理程序经常启动额外的 goroutine 来访问后端服务，如数据库和 RPC 服务。处理一个请求的一组 goroutine 通常需要访问该请求相关的特定的值，比如最终用户的身份、授权令牌和请求的 deadline 等。当一个请求被取消或处理超时时，所有在该请求上工作的 goroutines 应该迅速退出，以便系统可以回收他们正在使用的任何资源。

从这段描述中，我至少 get 到两点：

*   传值

后端服务程序有这样的需求，即在处理某请求的函数 (Handler Function) 中调用其他函数时，**传递与请求相关的 (request-specific)、请求内容之外的值信息 (以下称之为上下文中的值信息)**，如下图所示：

![](https://tonybai.com/wp-content/uploads/understand-go-context-by-example-2.png)

我们看到：这种函数调用以及传值可以发生在同一 goroutine 的函数之间 (比如上图中的 Handler 函数调用 middleware 函数)、同一进程的多个 goroutine 之间 (如被调用函数创建了新的 goroutine)，甚至是不同进程的 goroutine 之间 (比如 rpc 调用)。

*   控制

同一 goroutine 下因处理外部请求 (request) 而发生函数调用时，如果被调用的函数 (callee) 并没有启动新 goroutine 或进行跨进程的处理(如 rpc 调用)，这时更多的是在函数间传值，即传递上下文中的值信息。

但当被调用的函数 (callee) 启动新 goroutine 或进行跨进程处理时，这通常会是一种**异步调用**。为什么要启动新 goroutine 进行异步调用呢？更多是为了**控制**。如果是同步调用，一旦被调用方出现延迟或故障，这次调用很可能长期阻塞，调用者自身既无法消除这种影响，也不能及时回收掉处理这次请求所申请的各种资源，更无法保证服务接口之间的 SLA。

> 注意：调用者与被调用者之间可以是同步调用，也可以是异步调用，而被调用者则通常启动新的 goroutine 来实现一种 “异步调用”。

那么怎么控制异步调用呢？这回我们在调用者与被调用者之间传递的不再是一种值信息，而是一种 “默契”，即一种控制机制，如下图所示：

![](https://tonybai.com/wp-content/uploads/understand-go-context-by-example-3.png)

当被调用者在调用者的限定时间内完成任务，调用成功，被调用者释放所有资源；当被调用者无法在限定时间内完成或被调用者收到调用者取消调用的通知时，也能结束调用并释放资源。

接下来，我们就来看看 Go 标准库 context 包是如何解决上述两个问题的。

### 3. context 包的构成

Go 将对上面两个问题 “传值与控制” 的解决方案统一放到了 context 包下的一个名为 Context 接口类型中了：

```
// $GOROOT/src/context/context.go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key any) any
}


```

> 注：“上下文” 本没有统一标准，很多第三方包也有自己 Context 的定义，但 Go 1.7 之后都逐渐转为使用 Go 标准库的 context.Context 了。

如果你读懂了前面 context 包要解决的问题，你大致也能将 Context 接口类型中的方法分为两类，第一类就是 Value 方法，用于解决 “传值” 的问题；其他三个方法 (Deadline、Done 和 Err) 划归为第二类，用于解决 “传递控制” 的问题。

如果仅仅是定义 Context 这样一个接口类型，统一了对 Context 的抽象，那事情就未得到彻底解决 (但[也比 log 包做的要好了](https://tonybai.com/2022/10/30/first-exploration-of-slog))，Go context 包 “好人做到底”，还提供了一系列便利的函数以及若干内置的 Context 接口的实现。下面我们逐一来看一下。

#### 1) WithValue 函数

首先我们看一下**用于传值**的 WithValue 函数。

```
// $GOROOT/src/context/context.go
func WithValue(parent Context, key, val any) Context


```

WithValue 函数基于 parent Context 创建一个新的 Context，这个新的 Context 既保存了一份 parent Context 的副本，同时也保存了 WithValue 函数接受的那个 key-val 对。 WithValue 其实返回一个名为 * valueCtx 类型的实例，*valueCtx 实现了 Context 接口，它由三个字段组成：

```
// $GOROOT/src/context/context.go

type valueCtx struct {
    Context
    key, val any
}


```

结合 WithValue 的实现逻辑，valueCtx 中的 Context 被赋值为 parent Context，key 和 val 分别保存了 WithValue 传入的 key 和 val。

在新 Context 创建成功后，处理函数后续将基于该新 Context 进行上下文中的值信息的传递，我们来看一个例子：

```
// github.com/bigwhite/experiments/tree/master/context-examples/with_value/main.go

package main

import (
    "context"
    "fmt"
)

func f3(ctx context.Context, req any) {
    fmt.Println(ctx.Value("key0"))
    fmt.Println(ctx.Value("key1"))
    fmt.Println(ctx.Value("key2"))
}

func f2(ctx context.Context, req any) {
    ctx2 := context.WithValue(ctx, "key2", "value2")
    f3(ctx2, req)
}

func f1(ctx context.Context, req any) {
    ctx1 := context.WithValue(ctx, "key1", "value1")
    f2(ctx1, req)
}

func handle(ctx context.Context, req any) {
    ctx0 := context.WithValue(ctx, "key0", "value0")
    f1(ctx0, req)
}

func main() {
    rootCtx := context.Background()
    handle(rootCtx, "hello")
}


```

在上面这段代码中，handle 是负责处理 “请求” 的入口函数，它接受一个由 main 函数创建的 root Context 以及请求内容本身(“hello”)，之后 handle 函数基于传入的 ctx，通过 WithValue 函数创建了一个包含了自己附加的 key0-value0 对的新 Context，这个新 Context 将在调用 f1 函数时作为上下文传给 f1；依次类推，f1、f2 都基于传入的 ctx 通过 WithValue 函数创建了包含自己附加的值信息的新 Context，在函数调用链的末端，f3 通过 Context 的 Value 方法从传入的 ctx 中尝试取出上下文中的各种值信息，我们用一幅示意图来展示一下这个过程：

![](https://tonybai.com/wp-content/uploads/understand-go-context-by-example-4.png)

我们运行一下上述代码看看结果：

```
$go run main.go
value0
value1
value2


```

我们看到，f3 不仅从上下文中取出了 f2 附加的 key2-value2，还可以取出 handle、f1 等函数附加的值信息。这得益于满足 Context 接口的 * valueCtx 类型 “顺藤摸瓜” 的实现：

```
// $GOROOT/src/context/context.go

func (c *valueCtx) Value(key any) any {
    if c.key == key {
        return c.val
    }
    return value(c.Context, key)
}

func value(c Context, key any) any {
    for {
        switch ctx := c.(type) {
        case *valueCtx:
            if key == ctx.key {
                return ctx.val
            }
            c = ctx.Context
        case *cancelCtx:
            if key == &cancelCtxKey {
                return c
            }
            c = ctx.Context
        case *timerCtx:
            if key == &cancelCtxKey {
                return &ctx.cancelCtx
            }
            c = ctx.Context
        case *emptyCtx:
            return nil
        default:
            return c.Value(key)
        }
    }
}


```

我们看到在 * valueCtx case 中，如果 key 与当前 ctx 的 key 不同，就会继续沿着 parent Ctx 路径继续查找，直到找到为止。

我们看到：WithValue 用起来不难，也好理解。不过**由于每个 valueCtx 仅能保存一对 key-val**，这样即便在一个函数中添加多个值信息，其使用模式也必须是这样的：

```
ctx1 := WithValue(parentCtx, key1, val1)
ctx2 := WithValue(ctx1, key2, val2)
ctx3 := WithValue(ctx2, key3, val3)
nextCall(ctx3, req)


```

而不能是

```
ctx1 := WithValue(parentCtx, key1, val1)
ctx1 = WithValue(parentCtx, key2, val2)
ctx1 = WithValue(parentCtx, key3, val3)
nextCall(ctx1, req)


```

否则 ctx1 中仅会保存最后一次的 key3-val3 的信息，而 key1、key2 都会被覆盖掉。

valueCtx 的这种设计也导致了 Value 方法的查找 key 的效率不是很高，是个 O(n) 的查找。在一些对性能敏感的 Web 框架中，valueCtx 和 WithValue 可能难有用武之地。

在上面的例子中，我们说到了 root Context，下面简单说一下 root Context 的构建。

#### 2) root Context 构建

root Context，也称为 top-level Context，即最顶层的 Context，通常在 main 函数、初始化函数、请求处理的入口 (某个 Handle 函数) 中创建。 Go 提供了两种 root Context 的构建方法 Background 和 TODO：

```
// $GOROOT/src/context/context.go

var (
    background = new(emptyCtx)
    todo       = new(emptyCtx)
)

func Background() Context {
    return background
}

func TODO() Context {
    return todo
}


```

我们看到，虽然标准库提供了两种 root Context 的创建方法，但它们本质是一样的，底层都返回的是一个与程序同生命周期的 emptyCtx 类型的实例。有小伙伴可能会问：Go 所有代码共享一个 root Context 会不会有问题呢？

答案是不会！因为 root Context 啥 “实事” 也不做，就像 “英联邦国王” 一样，仅具有名义上的象征意义，它既不会存储上下文值信息，也不会携带上下文控制信息，整个生命周期内它都不会被改变。它只是作为二级上下文 parent Context 的指向，真正具有 “功能” 作用的 Context 是类似于首相或总理的 second-level Context：

![](https://tonybai.com/wp-content/uploads/understand-go-context-by-example-5.png)

通常我们都会使用 Background() 函数构造 root Context，而按照 context 包 TODO 函数的注释来看，TODO 仅在不清楚应该使用哪个 Context 的情况下临时使用。

#### 3) WithCancel 函数

WithCancel 函数为上下文提供了第一种控制机制：可取消 (cancel)，它也是整个 context 包控制机制的基础。我们先直观感受一下 WithCancel 的作用，下面是 [Go context 包文档](https://pkg.go.dev/context)中的一个例子：

```
package main

import (
    "context"
    "fmt"
)

func main() {
    gen := func(ctx context.Context) <-chan int {
        dst := make(chan int)
        n := 1
        go func() {
            for {
                select {
                case <-ctx.Done():
                    return // returning not to leak the goroutine
                case dst <- n:
                    n++
                }
            }
        }()
        return dst
    }

    ctx, cancel := context.WithCancel(context.Background())
    defer cancel() // cancel when we are finished consuming integers

    for n := range gen(ctx) {
        fmt.Println(n)
        if n == 5 {
            break
        }
    }
}


```

在这个例子，main 函数通过 WithCancel 创建了一个具有可取消属性的 Context 实例，然后在调用 gen 函数时传入了该实例。WithCancel 函数除了返回一个具有可取消属性的 Context 实例外，还返回了一个 cancelFunc，这个 cancelFunc 就是握在调用者手里的那个 “按钮”，一旦按下该“按钮”，即调用者发出“取消” 信号，异步调用中启动的 goroutine 就应该放下手头工作，老老实实地退出。

就像上面这个示例一样，main 函数将 cancel Context 传给 gen 后，gen 函数启动了一个新 goroutine 用于生成一组数列，而 main 函数则从 gen 返回的 channel 中读取这些数列中的数。main 函数在读完第 5 个数字后，按下了 “按钮”，即调用了 cancel Function。这时那个生成数列的 goroutine 会监听到 Done channel 有事件，然后完成 goroutine 的退出。

这就是前面说过的那种调用者和被调用者 (以及调用者创建的新 goroutine) 之间应具备的那种 “默契”，这种“默契” 要求两者都要基于上下文按一定的 “套路” 进行处理，在这个例子中就体现在**调用者适时调用 cancel Function，而 gen 启动的 goroutine 要监听可取消 Context 实例的 Done channel**。

并且通常，我们在创建完一个 cancel Context 后，立即会通过 defer 将 cancel Function 注册到 deferred function stack 中去，以防止因未调用 cancel Function 导致的资源泄露！在这个例子中，如果不调用 cancel Function，gen 函数创建的那个 goroutine 就会一直运行，虽然它生成的数字已经不会再有其他 goroutine 消费。

相较于 WithValue 函数，WithCancel 的实现略复杂：

```
// $GOROOT/src/context/context.go

func WithCancel(parent Context) (ctx Context, cancel CancelFunc) {
    if parent == nil {
        panic("cannot create context from nil parent")
    }
    c := newCancelCtx(parent)
    propagateCancel(parent, &c)
    return &c, func() { c.cancel(true, Canceled) }
}

func newCancelCtx(parent Context) cancelCtx {
    return cancelCtx{Context: parent}
}


```

其复杂就复杂在 propagateCancel 这个调用上：

```
// propagateCancel arranges for child to be canceled when parent is.
func propagateCancel(parent Context, child canceler) {
    done := parent.Done()
    if done == nil {
        return // parent is never canceled
    }

    select {
    case <-done:
        // parent is already canceled
        child.cancel(false, parent.Err())
        return
    default:
    }

    if p, ok := parentCancelCtx(parent); ok {
        p.mu.Lock()
        if p.err != nil {
            // parent has already been canceled
            child.cancel(false, p.err)
        } else {
            if p.children == nil {
                p.children = make(map[canceler]struct{})
            }
            p.children[child] = struct{}{}
        }
        p.mu.Unlock()
    } else {
        atomic.AddInt32(&goroutines, +1)
        go func() {
            select {
            case <-parent.Done():
                child.cancel(false, parent.Err())
            case <-child.Done():
            }
        }()
    }
}


```

propagateCancel 通过 parentCancelCtx 向上顺着 parent 路径查找，之所以可以这样，是因为 Value 方法具备沿着 parent 路径查找的特性：

```
func parentCancelCtx(parent Context) (*cancelCtx, bool) {
    done := parent.Done()
    if done == closedchan || done == nil {
        return nil, false
    }
    p, ok := parent.Value(&cancelCtxKey).(*cancelCtx) // 沿着parent路径查找第一个cancelCtx
    if !ok {
        return nil, false
    }
    pdone, _ := p.done.Load().(chan struct{})
    if pdone != done {
        return nil, false
    }
    return p, true
}


```

如果找到一个 cancelCtx，就将自己加入到该 cancelCtx 的 child map 中：

```
type cancelCtx struct {
    Context

    mu       sync.Mutex            // protects following fields
    done     atomic.Value          // of chan struct{}, created lazily, closed by first cancel call
    children map[canceler]struct{} // set to nil by the first cancel call
    err      error                 // set to non-nil by the first cancel call
}


```

> 注：接口类型值是支持比较的，如果两个接口类型值的动态类型相同且动态类型的值相同，那么两个接口类型值就相同。这也是 children 这个 map 用 canceler 接口作为 key 的原因。

这样当其 parent cancelCtx 的 cancel Function 被调用时，cancel function 会调用 cancelCtx 的 cancel 方法，cancel 方法会遍历所有 children cancelCtx，然后调用 child 的 cancel 方法以达到关联取消的目的，同时该 parent cancelCtx 会与所有 children cancelCtx 解除关系！

```
func (c *cancelCtx) cancel(removeFromParent bool, err error) {
    if err == nil {
        panic("context: internal error: missing cancel error")
    }
    c.mu.Lock()
    if c.err != nil {
        c.mu.Unlock()
        return // already canceled
    }
    c.err = err
    d, _ := c.done.Load().(chan struct{})
    if d == nil {
        c.done.Store(closedchan)
    } else {
        close(d)
    }
    for child := range c.children { // 遍历children，调用cancel方法
        // NOTE: acquiring the child's lock while holding parent's lock.
        child.cancel(false, err)
    }
    c.children = nil // 解除与children的关系
    c.mu.Unlock()

    if removeFromParent {
        removeChild(c.Context, c)
    }
}


```

我们用一个例子来演示一下：

```
// github.com/bigwhite/experiments/tree/master/context-examples/with_cancel/cancelctx_map.go

package main

import (
    "context"
    "fmt"
    "time"
)

// 直接使用parent cancelCtx
func f1(ctx context.Context) {
    go func() {
        select {
        case <-ctx.Done():
            fmt.Println("goroutine created by f1 exit")
        }
    }()
}

// 基于parent cancelCtx创建新的cancelCtx
func f2(ctx context.Context) {
    ctx1, _ := context.WithCancel(ctx)
    go func() {
        select {
        case <-ctx1.Done():
            fmt.Println("goroutine created by f2 exit")
        }
    }()
}

// 使用基于parent cancelCtx创建的valueCtx
func f3(ctx context.Context) {
    ctx1 := context.WithValue(ctx, "key3", "value3")
    go func() {
        select {
        case <-ctx1.Done():
            fmt.Println("goroutine created by f3 exit")
        }
    }()
}

// 基于parent cancelCtx创建的valueCtx之上创建cancelCtx
func f4(ctx context.Context) {
    ctx1 := context.WithValue(ctx, "key4", "value4")
    ctx2, _ := context.WithCancel(ctx1)
    go func() {
        select {
        case <-ctx2.Done():
            fmt.Println("goroutine created by f4 exit")
        }
    }()
}

func main() {
    valueCtx := context.WithValue(context.Background(), "key0", "value0")
    cancelCtx, cf := context.WithCancel(valueCtx)
    f1(cancelCtx)
    f2(cancelCtx)
    f3(cancelCtx)
    f4(cancelCtx)

    time.Sleep(3 * time.Second)
    fmt.Println("cancel all by main")
    cf()
    time.Sleep(10 * time.Second) // wait for log output
}


```

上面这个示例演示了四种情况：

*   f1: 直接使用 parent cancelCtx
*   f2: 基于 parent cancelCtx 创建新的 cancelCtx
*   f3: 使用基于 parent cancelCtx 创建的 valueCtx
*   f4: 使用基于 parent cancelCtx 创建的 valueCtx 之上创建的 cancelCtx

运行这个示例，我们得到：

```
cancel all by main
goroutine created by f1 exit
goroutine created by f2 exit
goroutine created by f3 exit
goroutine created by f4 exit


```

我们看到，无论是直接使用 parent cancelCtx，还是使用基于 parent cancelCtx 创建的其他各种 Ctx，当 parent cancelCtx 的 cancel Function 被调用后，所有监听对应 child Done channel 的 goroutine 都能正确收到通知并退出。

当然这种 “取消通知” 只能由 parent 通知到下面的 children，反过来则不行，parent cancelCtx 不会因为 child Context 的 cancel function 被调用而被 cancel 掉。另外如果某个 children cancelCtx 的 cancel Function 被调用后，该 children 会与其 parent cancelCtx 解绑。

在前面贴出的 propagateCancel 函数的实现中，我们还看到了另外一个分支，即 parentCancelCtx 函数返回的 ok 为 false 时，propagateCancel 函数会启动一个新的 goroutine 监听 parent Done channel 和自身的 Done channel。什么情况下会走到这个执行分支下呢？这种情况似乎不多！我们来看一个自定义 cancelCtx 的情况：

```
package main

import (
    "context"
    "fmt"
    "runtime"
    "time"
)

func f1(ctx context.Context) {
    ctx1, _ := context.WithCancel(ctx)
    go func() {
        select {
        case <-ctx1.Done():
            fmt.Println("goroutine created by f1 exit")
        }
    }()
}

type myCancelCtx struct {
    context.Context
    done chan struct{}
    err  error
}

func (ctx *myCancelCtx) Done() <-chan struct{} {
    return ctx.done
}

func (ctx *myCancelCtx) Err() error {
    return ctx.err
}

func WithMyCancelCtx(parent context.Context) (context.Context, context.CancelFunc) {
    var myCtx = &myCancelCtx{
        Context: parent,
        done:    make(chan struct{}),
    }

    return myCtx, func() {
        myCtx.done <- struct{}{}
        myCtx.err = context.Canceled
    }
}

func main() {
    valueCtx := context.WithValue(context.Background(), "key0", "value0")
    fmt.Println("before f1:", runtime.NumGoroutine())

    myCtx, mycf := WithMyCancelCtx(valueCtx)
    f1(myCtx)
    fmt.Println("after f1:", runtime.NumGoroutine())

    time.Sleep(3 * time.Second)
    mycf()
    time.Sleep(10 * time.Second) // wait for log output
}


```

在这个例子中，我们 “部分逃离” 了 context cancelCtx 的体系并自定义了一个实现了 Context 接口的 myCancelCtx，在这样的情况下，当 f1 函数基于 myCancelCtx 构建自己的 child CancelCtx 时，由于向上找不到 * cancelCtx 类型，所以它 WithCancel 启动了一个 goroutine 既监听自己的 Done channel，也监听其 parent Ctx(即 myCancelCtx)的 Done channel。

当 myCancelCtx 的 cancel Function 在 main 函数中被调用时 (mycf())，新建的 goroutine 会调用 child 的 cancel 函数实现操作取消。运行上面示例，我们得到如下结果：

```
$go run custom_cancelctx.go
before f1: 1
after f1: 3  // 在context包中新创建了一个goroutine
goroutine created by f1 exit


```

由此，我们看到，除了 “业务” 层面可能导致的资源泄露之外，cancel Context 的实现中也会有一些资源 (比如上面这个新建的 goroutine) 需要及时释放，否则也会导致“泄露”。

一些小伙伴可能会问这样一个问题：在被调用函数 (callee) 中，到底是继续传递原 cancelCtx 给新建的 goroutine，还是基于 parent cancelCtx 创建一个新的 cancelCtx 再传给 goroutine 用呢？这让我想起了装修时遇到的一个问题：是否在水管某些地方加阀门？

![](https://tonybai.com/wp-content/uploads/understand-go-context-by-example-6.png)

加上阀门，可以单独控制一路的关闭！同样在代码中，基于 parent cancelCtx 创建新的 cancelCtx 可以做单独取消操作，而不影响 parentCtx，这就看业务层代码是否需要这么做了。

到这里，我们已经 get 到了 context 包提供的取消机制，但实际中，**我们很难拿捏好 cancel Function 调用的时机**。为此，context 包提供了另外一个建构在 cancelCtx 之上的实用控制机制：timerCtx。接下来，我们就来看看 timerCtx。

#### 4) WithDeadline 和 WithTimeout 函数

timerCtx 基于 cancelCtx 提供了一种基于 deadline 的取消控制机制：

```
type timerCtx struct {
    cancelCtx
    timer *time.Timer // Under cancelCtx.mu.

    deadline time.Time
}


```

context 包提供了两个创建 timerCtx 的 API：WithDeadline 和 WithTimeout 函数：

```
// $GOROOT/src/context/context.go

func WithDeadline(parent Context, d time.Time) (Context, CancelFunc) {
    if parent == nil {
        panic("cannot create context from nil parent")
    }
    if cur, ok := parent.Deadline(); ok && cur.Before(d) {
        // The current deadline is already sooner than the new one.
        return WithCancel(parent)
    }
    c := &timerCtx{
        cancelCtx: newCancelCtx(parent),
        deadline:  d,
    }
    propagateCancel(parent, c)
    dur := time.Until(d)
    if dur <= 0 {
        c.cancel(true, DeadlineExceeded) // deadline has already passed
        return c, func() { c.cancel(false, Canceled) }
    }
    c.mu.Lock()
    defer c.mu.Unlock()
    if c.err == nil {
        c.timer = time.AfterFunc(dur, func() {
            c.cancel(true, DeadlineExceeded)
        })
    }
    return c, func() { c.cancel(true, Canceled) }
}

func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc) {
    return WithDeadline(parent, time.Now().Add(timeout))
}


```

从实现来看，WithTimeout 就是 WithDeadline 的再包装！我们弄懂 WithDeadline 即可。从 WithDeadline 的实现来看，该函数通过 time.AfterFunc 设置了一个定时器，定时器 fire 后的执行逻辑就是执行该 ctx 的 cancel Function。也就是说 timerCtx 既支持手工 cancel(原 cancelCtx 的机制)，也支持定时 cancel，并且通常由定时器来完成 cancel。

有了 cancelCtx 的基础，timerCtx 就不难理解了。不要要注意的一点时，即便有了定时器来 cancel 操作，我们也不要忘记显式调用 WithDeadline 和 WithTimeout 返回的 cancel function，及早释放资源不是更好么！

### 4. 小结

本文对 Go 标准库 context 包要解决的问题、context 包构成以及传值和传递控制的原理做了简要讲解，相信读完这些内容后，你再回头去看你写过的运用 context 包的代码肯定会有更为深刻的理解。

context 包目前在 Go 生态内得到广泛应用，较为典型的是在 http handler 中传递值信息、在 tracing 框架中通过在上下文中的 trace ID 来整合 tracing 信息等。

Go 社区对 context 包的声音也不全是正面，其中 context.Context 具有 “病毒般” 的传染性就是被集中诟病的方面。[Go 官方也有一个 issue 记录了 Go 社区对 context 包的反馈和优化建议](https://github.com/golang/go/issues/28342)，有兴趣的小伙伴可以去翻翻。

本文的 context 包源码来自 [Go 1.19.1 版本](https://tonybai.com/2022/08/22/some-changes-in-go-1-19)，与老版本 Go 或 Go 的未来版本可能会有差别。

本文的源码在[这里](https://tonybai.com/2022/11/08/understand-go-context-by-example/https//github.com/bigwhite/experiments/tree/master/context-examples)可以下载。

### 5. 参考资料

*   context 包文档手册 – https://pkg.go.dev/context
*   Go Concurrency Patterns: Context – https://go.dev/blog/context

[“Gopher 部落” 知识星球](https://wx.zsxq.com/dweb2/index/group/51284458844544)旨在打造一个精品 Go 学习和进阶社群！高品质首发 Go 技术文章，“三天” 首发阅读权，每年两期 Go 语言发展现状分析，每天提前 1 小时阅读到新鲜的 Gopher 日报，网课、技术专栏、图书内容前瞻，六小时内必答保证等满足你关于 Go 语言生态的所有需求！2022 年，Gopher 部落全面改版，将持续分享 Go 语言与 Go 应用领域的知识、技巧与实践，并增加诸多互动形式。欢迎大家加入！

Gopher Daily(Gopher 每日新闻) 归档仓库 – https://github.com/bigwhite/gopherdaily

我的联系方式：

*   微博：https://weibo.com/bigwhite20xx
*   博客：tonybai.com
*   github: https://github.com/bigwhite

© 2022, [bigwhite](https://tonybai.com/). 版权所有.

Related posts:

1.  [Go coding in go way](https://tonybai.com/2017/04/20/go-coding-in-go-way/ "Go coding in go way")
2.  [Go 语言回顾：从 Go 1.0 到 Go 1.13](https://tonybai.com/2019/09/07/go-retrospective/ "Go语言回顾：从Go 1.0到Go 1.13")
3.  [Go 语言错误处理](https://tonybai.com/2015/10/30/error-handling-in-go/ "Go语言错误处理")
4.  [GoCN 社区 Go 读书会第二期：《Go 语言精进之路》](https://tonybai.com/2022/07/07/gocn-community-go-book-club-issue2-go-programming-from-beginner-to-master/ "GoCN社区Go读书会第二期：《Go语言精进之路》")
5.  [Go 编程语言与环境：万字长文复盘导致 Go 语言成功的那些设计决策 [译]](https://tonybai.com/2022/05/04/the-paper-of-go-programming-language-and-environment/ "Go编程语言与环境：万字长文复盘导致Go语言成功的那些设计决策[译]")
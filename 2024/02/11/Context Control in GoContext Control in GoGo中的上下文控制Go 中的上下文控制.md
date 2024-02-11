> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zenhorace.dev](https://zenhorace.dev/blog/context-control-go/)

Context Control in Go Go 中的上下文控制
================================

### Best practices for handling context plumbing.  
处理上下文管道的最佳实践。

##### Published: 2024-02-03 发布日期： 2024-02-03

![](https://zenhorace.dev/img/context-go.png)

> tl;dr TL的;博士  
> There are three main rules to observe when handling context plumbing in Go: only entry-point functions should create new contexts, contexts are only passed down the call chain, and don’t store contexts or otherwise use them after the function returns.  
> 在 Go 中处理上下文管道时，需要遵守三个主要规则：只有入口点函数才能创建新的上下文，上下文只在调用链中向下传递，并且不要存储上下文或在函数返回后以其他方式使用它们。

Context is one of the foundational building blocks in Go. Anyone with even a cursory experience with the language is likely to have encountered it, as it’s the first argument passed to functions that accept contexts. I see the purpose of context as twofold:  
上下文是 Go 中的基础构建块之一。任何对这门语言有粗略经验的人都可能遇到过它，因为它是传递给接受上下文的函数的第一个参数。我认为上下文的目的有两个：  
1. Provide a [control-flow](https://en.wikipedia.org/wiki/Control_flow) mechanism across API boundaries with signals.  
1. 提供跨 API 边界的信号控制流机制。  
1. Carrying request-scoped data across API boundaries.  
1. 跨 API 边界携带请求范围的数据。

This post will focus on good practices for leveraging contexts with control-flow operations.  
本文将重点介绍通过控制流操作利用上下文的良好做法。

A couple of rules of thumb to start.  
从几个经验法则开始。
-------------------------------------------------

1.  Only entry-point functions (the one at the top of a call chain) should create an empty context (i.e., `context.Background()`). For example, `main()`, `TestXxx()`. The HTTP library creates a custom context for each request, which you should access and pass. Of course, mid-chain functions can create child contexts to pass along if they need to share data or have flow control over the functions they call.  
    只有入口点函数（位于调用链顶部的函数）才应创建一个空上下文（即 `context.Background()` ）。例如， `main()` ， `TestXxx()` .HTTP 库为每个请求创建一个自定义上下文，您应该访问并传递该上下文。当然，如果中间链函数需要共享数据或对它们调用的函数进行流控制，它们可以创建子上下文来传递。
    
2.  Contexts are (only) passed down the call chain. If you’re not in an entry-point function and you need to call a function that takes a context, your function should accept a context and pass that along. But what if, for some reason, you can’t currently get access to the context at the top of the chain? In that case, use `context.TODO()`. This signals that the context is not yet available, and further work is required. Perhaps maintainers of another library you depend on will need to extend their functions to accept a context so that you, in turn, can pass it on. Of course, a function should never be returning a context.  
    上下文（仅）在调用链中向下传递。如果您不在入口点函数中，并且需要调用采用上下文的函数，则您的函数应接受上下文并传递该上下文。但是，如果由于某种原因，您目前无法访问链顶部的上下文怎么办？在这种情况下，请使用 `context.TODO()` .这表明上下文尚不可用，需要进一步的工作。也许你所依赖的另一个库的维护者需要扩展他们的功能来接受一个上下文，这样你就可以反过来传递它。当然，函数永远不应该返回上下文。
    

There are three key rules of thumb when handling context. The first two mentioned above are relatively straightforward. The third rule is the reason for this post, as I encountered it this week.  
在处理上下文时，有三个关键的经验法则。上面提到的前两个相对简单。第三条规则是这篇文章的原因，正如我本周遇到的那样。

Storytime 故事时间
--------------

The [documentation](https://pkg.go.dev/context) for context states:  
上下文文档指出：

> Do not store Contexts inside a struct type; instead, pass a Context explicitly to each function that needs it.  
> 不要将 Contexts 存储在结构类型中;相反，将 Context 显式传递给每个需要它的函数。

I _thought_ I understood this implicitly, and it sounds easy to obey. So I was surprised and confused earlier this week when I received a comment on a code review telling me, “Don’t store contexts.”  
我以为我含蓄地理解了这一点，这听起来很容易服从。因此，本周早些时候，当我收到一条关于代码审查的评论时，我感到惊讶和困惑，告诉我，“不要存储上下文”。  
“I’m not, though…” was my first thought. No context was in my struct!  
“不过，我不是......”是我的第一个想法。我的结构中没有上下文！

What was I doing wrong? Let me set the context (pun intended). If you just want the third rule sans preamble, skip to the next section.  
我做错了什么？让我设置上下文（双关语）。如果您只想要第三条无序言的规则，请跳到下一节。

_Note: The code samples discussed below are simplified approximations of the issue I faced. While the examples should be fine, there may be typos.  
注意：下面讨论的代码示例是我所面临问题的简化近似值。虽然示例应该没问题，但可能会有错别字。_

Imagine a long-running routine that makes requests to some source and relays the data it receives to a PubSub service. It keeps doing this until the caller tells the routine to stop. This relatively common system may look something like this:  
想象一个长时间运行的例程，它向某个源发出请求，并将它收到的数据中继到 PubSub 服务。它一直这样做，直到调用方告诉例程停止。这个相对常见的系统可能看起来像这样：

```
type Worker struct {
  quit chan struct{}
  // internal details
}

// New configures and returns a Worker.
func New(ctx context.Context, ...) (*Worker, error)

func (w *Worker) Run(ctx context.Context)

func (w *Worker) Stop()

```

This is fine. However, I (in my infinite wisdom) thought that I could simplify things. I knew that:  
这很好。然而，我（以我无限的智慧）认为我可以简化事情。我知道：  
- the caller of this routine would always want to run it asynchronously (I wrote the only caller), and  
- 这个例程的调用方总是希望异步运行它（我写了唯一的调用方），并且  
- once the routine had been started, the only action the caller would need was to stop the routine.  
- 启动例程后，调用方唯一需要的操作就是停止例程。

So, I came up with this:  
所以，我想出了这个：

```
type worker struct {
  quit chan struct{}
  // other internal details
}

func Start(ctx context.Context, ...) (cancel func()){
  // Configure setup. Details elided.
  w := &worker{...}

  go w.run(ctx context.Context)
  return w.stop
}

func (w *worker) run(ctx context.Context) {
  ticker := time.NewTicker(time.Minute)
  defer ticker.Stop()
  for {
    select {
    case <- w.quit:
      // perform cleanup
    case <-ticker.C:
      cctx, cancel := context.WithTimeout(ctx, 30 * time.Second)
      w.doWork(cctx)
      cancel()
    }
  }
}

func (w *worker) stop() {
  close(w.quit)
}

```

Now, most seasoned Go devs would leap out of their seats to tell you it’s an anti-pattern for libraries to start their own goroutines. Best practices dictate that you should perform your work synchronously and let the caller decide if they want it to be asynchronous. Despite knowing this, I figured, “I’m writing the caller; it’ll be fine.” Now, instead of calling `New()`, then `Run()`, I can simply call `Start()`, which returns a cancel function. And I no longer need to export anything except `Start()` (I’m a sucker for tiny API surfaces).  
现在，大多数经验丰富的 Go 开发人员都会从座位上跳起来告诉你，对于图书馆来说，启动自己的 goroutine 是一种反模式。最佳做法规定，应同步执行工作，并让调用方决定是否希望异步工作。尽管知道这一点，但我想，“我正在写来电者;会没事的。现在，我可以简单地调用 ，而不是调用 `New()` `Start()` ， `Run()` 它返回一个取消函数。而且我不再需要导出任何东西，除了 `Start()` （我是微小的 API 表面的傻瓜）。

After I did that, I realized, “Oh… I need to ensure I also respect context cancellations.” So I made this change to `run()`:  
在我这样做之后，我意识到，“哦......我需要确保我也尊重上下文取消。所以我把这个改成了 `run()` ：

```
func (w *worker) run(ctx context.Context) {
  ticker := time.NewTicker(time.Minute)
  defer ticker.Stop()
  for {
    select {
    case <- w.quit:
      // perform cleanup
    case <- ctx.Done():
      // perform cleanup
    case <-ticker.C:
      // do work
    }
  }
}

```

Again, this should’ve been another indication that my hack wasn’t such a stroke of genius. I had the same logic to handle context cancellations and calls to `stop`. However, I was still too thrilled with my work, so I abstracted the cleanup logic to its own method and moved on.  
同样，这应该是另一个迹象，表明我的黑客并不是天才之举。我有相同的逻辑来处理上下文取消和对 `stop` .但是，我仍然对自己的工作感到非常兴奋，所以我将清理逻辑抽象为它自己的方法并继续前进。

Anyway, can you spot how I was storing context even though I only passed it to functions and never put it in a struct?  
无论如何，您能发现我是如何存储上下文的，即使我只将其传递给函数而从未将其放在结构中？  
The issue is that `Start()` takes a context, passes it to a goroutine, and returns. Even after returning, the context it’s passed is still being used, breaking the lifecycle expectations like I had stashed it in a struct.  
问题在于，它 `Start()` 获取一个上下文，将其传递给 goroutine，然后返回。即使在返回后，它传递的上下文仍在使用，打破了生命周期预期，就像我将其隐藏在结构中一样。

So, I assessed my code with some [rubber duck debugging](https://en.wikipedia.org/wiki/Rubber_duck_debugging):  
因此，我通过一些橡皮鸭调试来评估我的代码：

**Me**: So, does this mean I have to toss this whole thing and start over??  
我：所以，这是否意味着我必须抛弃整个事情并重新开始？？

**Rubber Duck**: There were some interesting ideas in there. This can work with some tweaks. First off, stop ignoring best practices and make the work synchronous.  
橡皮鸭：这里面有一些有趣的想法。这可以通过一些调整来工作。首先，不要再忽视最佳实践，让工作同步。

**Me**: That makes sense. It’s two extra characters for the caller to make it async - not much work. But wait! If `Start()` is blocking, how will the caller access `Stop()`? I’ll have to go back to the `New() -> [Run, Stop]` way…  
我：有道理。调用方需要两个额外的字符才能使其异步 - 工作量不大。但是等等！如果 `Start()` 被阻止，呼叫者将如何访问 `Stop()` ？我得回去了 `New() -> [Run, Stop]` ......

**Rubber Duck**: Well, you currently have two stopping mechanisms that do identical work.  
Rubber Duck：嗯，你目前有两个停止机制，它们做着相同的工作。

**Me**: You’re right! A cancellable context is an excellent [inversion of control](https://en.wikipedia.org/wiki/Inversion_of_control) mechanism. I don’t need to create a custom Stop function.  
我：你说得对！可取消上下文是一种极好的控制反转机制。我不需要创建自定义 Stop 函数。

```
type worker struct {
  // internal details. no stop channel.
}

// Start configures and runs the worker.
// Blocks until context cancellation.
func Start(ctx context.Context, ...){
  // Configure setup. Details elided.
  w := worker{...}
  // blocking call to run
  w.run(ctx context.Context)
}

func (w *worker) run(ctx context.Context) {
  ticker := time.NewTicker(time.Minute)
  defer ticker.Stop()
  for {
    select {
    case <- ctx.Done:
      // perform cleanup
    case <-ticker.C:
      // do work
    }
  }
}

```

By trying to be a little less clever, the final solution was cleaner, simpler, and less error-prone.  
通过尝试变得不那么聪明，最终的解决方案更干净、更简单、更不容易出错。

Rule 3: Don’t store contexts  
规则 3：不要存储上下文
-------------------------------------------

The core of the rule is:  
该规则的核心是：

> When a function takes a context parameter, that context should only be used for the duration of the call, not after it returns.  
> 当函数采用上下文参数时，该上下文应仅在调用期间使用，而不应在返回后使用。

The rationale is that once a function returns, the caller often cancels the context. Then, any calls made with that context will be canceled before they even begin, causing errors. These can be some of the most obscure bugs to root cause, so it’s best to eliminate the possibility.  
其基本原理是，一旦函数返回，调用方通常会取消上下文。然后，使用该上下文进行的任何调用都将在开始之前被取消，从而导致错误。这些可能是一些最晦涩难懂的 bug 根本原因，因此最好消除这种可能性。

**_Fin. 鳍。_**

Additional References on this topic:  
有关此主题的其他参考：  
- [Contexts and structs](https://go.dev/blog/context-and-structs)  
- 上下文和结构  
- [Google Go style decisions](https://google.github.io/styleguide/go/decisions#contexts)  
- Google Go 风格决策  
- [Google Go style best practices](https://google.github.io/styleguide/go/best-practices#contexts)  
- Google Go 风格最佳实践
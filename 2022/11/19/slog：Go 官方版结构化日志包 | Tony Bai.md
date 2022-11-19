> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [tonybai.com](https://tonybai.com/2022/10/30/first-exploration-of-slog/)

> 本文永久链接 -

![](https://tonybai.com/wp-content/uploads/first-exploration-of-slog-1.png)

[本文永久链接](https://tonybai.com/2022/10/30/first-exploration-of-slog) – https://tonybai.com/2022/10/30/first-exploration-of-slog

Go 自诞生以来就在其[标准库](https://tonybai.com/2022/10/25/the-modules-that-go-standard-library-depend-on)内置了 log 包作为 Go 源码[输出日志](https://tonybai.com/2022/03/05/go-logging-practice)的标准组件，该包被广泛应用于 Go 标准库自身以及 Go 社区项目中。

不过，针对 Go 标准库 log 包，Go 社区要求改进的声音始终不断，主流声音聚焦在以下几点：

*   log 包是为了方便人类可读而设计的，不支持便于机器解析的结构化日志 (比如[像 zap 那样输出 json 格式的日志](https://tonybai.com/2021/07/14/uber-zap-advanced-usage))；
*   不支持日志级别 (log level)；
*   log 包采用专有日志输出格式，又没有提供可供 Go 社区共同遵循的 Logger 接口类型，导致 Go 社区项目使用的 log 输出格式五花八门，相互之间又难以兼容。

Go 社区曾经[尝试过合力改进标准库 log 包](https://groups.google.com/g/golang-dev/c/F3l9Iz1JX4g/m/t0J0loRaDQAJ)，并[撰写了 Proposal 设计初稿](https://docs.google.com/document/d/1nFRxQ5SJVPpIBWTFHV-q5lBYiwGrfCMkESFGNzsrvBU/edit)，但最终因各种原因都没有被 Go 核心团队接受。

2022 年 8 月末，Go 团队的 [Jonathan Amsterdam](https://github.com/jba) [发起 discussion](https://github.com/golang/go/discussions/54763)，意在和社区讨论[为 Go 标准库添加结构化的、支持日志级别的日志包相关事宜](https://github.com/golang/go/discussions/54763)，并形成一个一致的 Proposal。

Jonathan Amsterdam 将该结构化日志包命名为 slog，计划放在 log/slog 下。他还在 golang.org/x/exp 下面给出了 [slog 的初始实现](https://github.com/golang/exp/slog)，这几天[该 Proposal 正式进入 review 阶段](https://go-review.googlesource.com/c/proposal/+/444415/3/design/56345-structured-logging.md)。至于何时能正式落地到 Go 正式版本中还不可知。

在这篇文章中，我们就来简单看一下 slog 的 proposal 以及它的初始实现。

### 1. slog 的设计简介

slog 的设计之初对社区目前的一些应用广泛的 log 包进行了详细调研，比如 [uber zap](https://tonybai.com/2021/07/14/uber-zap-advanced-usage)、[zerolog](https://github.com/rs/zerolog) 等，因此 slog 的设计也算是 “站在前人的肩膀上”，尤其是 uber zap。

Jonathan Amsterdam 为此次 slog 的设计设定了如下目标 (摘自 slog 的 proposal)：

*   易用性

通过对现有日志包的调查发现，程序员们希望有一套简洁且易于理解的 logging API。在此 proposal 中，我们将采用目前最流行的方式来表达键值对，即交替传入键和值。

*   高性能高

该 log API 的设计将尽量减少内存分配和加锁。它提供了一个交替输入键和值的方法，虽略繁琐，但速度更快；

*   可以与运行时跟踪 (tracing) 集成

Go 团队正在开发一个改进的运行时跟踪 (runtime tracing) 系统。本软件包的日志将可以被无缝集成到这个跟踪系统中，使开发者能够将他们的程序行为与运行时的行为联系起来。

这里基于 slog proposal 和 golang.org/x/exp/slog 的源码，画了一幅 slog 的结构示意图：

![](https://tonybai.com/wp-content/uploads/first-exploration-of-slog-2.png)

简单解释一下这个图：

slog 从逻辑上分为前端 (front) 和后端(backend)。

slog 前端就是 slog 提供给使用者的 API，不过，很遗憾 slog 依旧像 log 那样没有抽取出 Logger 接口，而是定义了一个 Logger 结构体，并提供了如图中的那些方法，这也意味着**我们依旧无法在整个 Go 社区统一前端 API**；

通过前端方法，slog 将日志内容以及相关属性信息封装成一个 slog.Record 类型实例，然后传递给 slog 的后端。

如果你使用的是 Go 社区的第三方 log 包的前端方法，比如 zap，如果要使用 slog 后端，你同样需要对 zap 等进行封装，让其输出 slog.Record 并传递给 slog 的后端 (目前尚没有这方面示例)。

slog 将后端抽象为 slog.Handler 接口，接口如下：

```
//
// Any of the Handler's methods may be called concurrently with itself
// or with other methods. It is the responsibility of the Handler to
// manage this concurrency.
type Handler interface {
    // Enabled reports whether the handler handles records at the given level.
    // The handler ignores records whose level is lower.
    // Enabled is called early, before any arguments are processed,
    // to save effort if the log event should be discarded.
    Enabled(Level) bool

    // Handle handles the Record.
    // It will only be called if Enabled returns true.
    // Handle methods that produce output should observe the following rules:
    //   - If r.Time is the zero time, ignore the time.
    //   - If an Attr's key is the empty string, ignore the Attr.
    Handle(r Record) error

    // WithAttrs returns a new Handler whose attributes consist of
    // both the receiver's attributes and the arguments.
    // The Handler owns the slice: it may retain, modify or discard it.
    WithAttrs(attrs []Attr) Handler

    // WithGroup returns a new Handler with the given group appended to
    // the receiver's existing groups.
    // The keys of all subsequent attributes, whether added by With or in a
    // Record, should be qualified by the sequence of group names.
    //
    // How this qualification happens is up to the Handler, so long as
    // this Handler's attribute keys differ from those of another Handler
    // with a different sequence of group names.
    //
    // A Handler should treat WithGroup as starting a Group of Attrs that ends
    // at the end of the log event. That is,
    //
    //     logger.WithGroup("s").LogAttrs(slog.Int("a", 1), slog.Int("b", 2))
    //
    // should behave like
    //
    //     logger.LogAttrs(slog.Group("s", slog.Int("a", 1), slog.Int("b", 2)))
    WithGroup(name string) Handler
}


```

接口类型的存在，让 slog 的后端扩展性更强，我们除了可以使用 slog 提供的两个内置 Handler 实现：TextHandler 和 JSONHandler 之外，还可以基于第三方 log 包定义或完全自定义后端 Handler 的实现。

slog 内置两个最常用的 Handler：TextHandler 和 JSONHandler。TextHandler 顾名思义，像标准库 log 包那样将日志以一行文本那样输出；而 JSONHandler 则是以 JSON 格式输出 log 内容与各个属性，我们看一下作者给的例子：

```
// github.com/bigwhite/experiments/tree/master/slog-examples/demo1/main.go
package main

import (
    "net"

    "golang.org/x/exp/slog"
)

func main() {
    slog.SetDefault(slog.New(slog.NewTextHandler(os.Stderr)))
    slog.Info("hello", "name", "Al")
    slog.Error("oops", net.ErrClosed, "status", 500)
    slog.LogAttrs(slog.ErrorLevel, "oops",
        slog.Int("status", 500), slog.Any("err", net.ErrClosed))
}


```

这是一个使用内置 TextHandler 的示例，我们运行一下看看结果：

```
time=2022-10-23T18:41:35.074+08:00 level=INFO msg=hello name=Al
time=2022-10-23T18:41:35.074+08:00 level=ERROR msg=oops status=500 err="use of closed network connection"
time=2022-10-23T18:41:35.074+08:00 level=ERROR msg=oops status=500 err="use of closed network connection"


```

我们看到，输出的日志以 “key1=value1 key2=value2 … keyN=valueN” 形式呈现，time 和 level 两个 key 是必选，调用 Error 方法时，err 这个 key 也是必选的。

接下来，我们将 TextHandler 换成 JSONHandler：

```
slog.SetDefault(slog.New(slog.NewTextHandler(os.Stderr)))

改为：

slog.SetDefault(slog.New(slog.NewJSONHandler(os.Stderr)))


```

运行修改后的程序，我们得到：

```
{"time":"2022-10-23T18:45:26.2131+08:00","level":"INFO","msg":"hello","name":"Al"}
{"time":"2022-10-23T18:45:26.213287+08:00","level":"ERROR","msg":"oops","status":500,"err":"use of closed network connection"}
{"time":"2022-10-23T18:45:26.21331+08:00","level":"ERROR","msg":"oops","status":500,"err":"use of closed network connection"}


```

我们看到，每条日志以一条 json 记录的形式呈现，这样的结构化日志非常适合机器解析。

如果我们去掉上面 SetDefault 那一行代码，再来运行一下程序：

```
2022/10/23 18:47:51 INFO hello name=Al
2022/10/23 18:47:51 ERROR oops status=500 err="use of closed network connection"
2022/10/23 18:47:51 ERROR oops status=500 err="use of closed network connection"


```

我们得到了不同于 TextHandler 和 JSONHandler 的日志样式，不过这个日志样式非常眼熟！这不和 log 包的输出样式相同么！没错，如果没有显式将新创建的 Logger 设置为默认 Logger，slog 会使用 defaultHandler，而 defaultHandler 的 output 函数就是 log.Output：

```
// slog项目

// logger.go
var defaultLogger atomic.Value

func init() {
    defaultLogger.Store(Logger{
        handler: newDefaultHandler(log.Output), // 这里直接使用了log.Output
    })
} 

// handler.go

type defaultHandler struct {
    ch *commonHandler
    // log.Output, except for testing
    output func(calldepth int, message string) error
}

func newDefaultHandler(output func(int, string) error) *defaultHandler {
    return &defaultHandler{
        ch:     &commonHandler{json: false},
        output: output,
    }
}


```

slog 的前端是 “固定格式” 的，因此没什么可定制的。但后端这块倒是有不少玩法，接下来我们重点看一下后端。

### 2. Handler 选项 (HandlerOptions)

slog 提供了 HandlerOptions 结构：

```
// handler.go

// HandlerOptions are options for a TextHandler or JSONHandler.
// A zero HandlerOptions consists entirely of default values.
type HandlerOptions struct {
    // Add a "source" attribute to the output whose value is of the form
    // "file:line".
    // This is false by default, because there is a cost to extracting this
    // information.
    AddSource bool

    // Ignore records with levels below Level.Level().
    // The default is InfoLevel.
    Level Leveler

    // If set, ReplaceAttr is called on each attribute of the message,
    // and the returned value is used instead of the original. If the returned
    // key is empty, the attribute is omitted from the output.
    //
    // The built-in attributes with keys "time", "level", "source", and "msg"
    // are passed to this function first, except that time and level are omitted
    // if zero, and source is omitted if AddSourceLine is false.
    //
    // ReplaceAttr can be used to change the default keys of the built-in
    // attributes, convert types (for example, to replace a `time.Time` with the
    // integer seconds since the Unix epoch), sanitize personal information, or
    // remove attributes from the output.
    ReplaceAttr func(a Attr) Attr
}


```

通过该结构，我们可以为输出的日志添加 source 信息，即输出日志的文件名与行号，下面就是一个例子：

```
// github.com/bigwhite/experiments/tree/master/slog-examples/demo2/main.go
package main

import (
    "net"
    "os"

    "golang.org/x/exp/slog"
)

func main() {
    opts := slog.HandlerOptions{
        AddSource: true,
    }

    slog.SetDefault(slog.New(opts.NewJSONHandler(os.Stderr)))
    slog.Info("hello", "name", "Al")
    slog.Error("oops", net.ErrClosed, "status", 500)
    slog.LogAttrs(slog.ErrorLevel, "oops",
        slog.Int("status", 500), slog.Any("err", net.ErrClosed))
}


```

运行上述程序，我们将得到：

```
{"time":"2022-10-23T21:46:25.718112+08:00","level":"INFO","source":"/Users/tonybai/go/src/github.com/bigwhite/experiments/slog-examples/demo2/main.go:16","msg":"hello","name":"Al"}
{"time":"2022-10-23T21:46:25.718324+08:00","level":"ERROR","source":"/Users/tonybai/go/src/github.com/bigwhite/experiments/slog-examples/demo2/main.go:17","msg":"oops","status":500,"err":"use of closed network connection"}
{"time":"2022-10-23T21:46:25.718352+08:00","level":"ERROR","source":"/Users/tonybai/go/src/github.com/bigwhite/experiments/slog-examples/demo2/main.go:18","msg":"oops","status":500,"err":"use of closed network connection"}


```

我们也可以通过 HandlerOptions 实现日志级别的动态设置，比如下面例子：

```
// github.com/bigwhite/experiments/tree/master/slog-examples/demo3/main.go
func main() {
    var lvl = &slog.AtomicLevel{}
    lvl.Set(slog.DebugLevel)
    opts := slog.HandlerOptions{
        Level: lvl,
    }
    slog.SetDefault(slog.New(opts.NewJSONHandler(os.Stderr)))

    slog.Info("before resetting log level:")

    slog.Info("hello", "name", "Al")
    slog.Error("oops", net.ErrClosed, "status", 500)
    slog.LogAttrs(slog.ErrorLevel, "oops",
        slog.Int("status", 500), slog.Any("err", net.ErrClosed))

    slog.Info("after resetting log level to error level:")
    lvl.Set(slog.ErrorLevel)
    slog.Info("hello", "name", "Al")
    slog.Error("oops", net.ErrClosed, "status", 500)
    slog.LogAttrs(slog.ErrorLevel, "oops",
        slog.Int("status", 500), slog.Any("err", net.ErrClosed))

}


```

slog.HandlerOptions 的字段 Level 是一个接口类型变量，其类型为 slog.Leveler：

```
type Leveler interface {
    Level() Level
}


```

实现了 Level 方法的类型都可以赋值给 HandlerOptions 的 Level 字段，slog 提供了支持并发访问的 AtomicLevel 供我们直接使用，上面的 demo3 使用的就是 AtomicLevel，初始时设置的是 DebugLevel，于是第一次调用 Info、Error 等 API 输出的日志都会得到输出，之后重置日志级别为 ErrorLevel，这样 Info API 输出的日志将不会被呈现出来，下面是 demo3 程序的运行结果：

```
{"time":"2022-10-23T21:58:48.467666+08:00","level":"INFO","msg":"before resetting log level:"}
{"time":"2022-10-23T21:58:48.467818+08:00","level":"INFO","msg":"hello","name":"Al"}
{"time":"2022-10-23T21:58:48.467825+08:00","level":"ERROR","msg":"oops","status":500,"err":"use of closed network connection"}
{"time":"2022-10-23T21:58:48.467842+08:00","level":"ERROR","msg":"oops","status":500,"err":"use of closed network connection"}
{"time":"2022-10-23T21:58:48.467846+08:00","level":"INFO","msg":"after resetting log level to error level:"}
{"time":"2022-10-23T21:58:48.46785+08:00","level":"ERROR","msg":"oops","status":500,"err":"use of closed network connection"}
{"time":"2022-10-23T21:58:48.467854+08:00","level":"ERROR","msg":"oops","status":500,"err":"use of closed network connection"}


```

HandlerOptions 的第三个字段 ReplaceAttr 有什么功用，就留给大家自己探索一下。

除了利用 HandleOptions 做一些定制之外，我们也可以完全自定义 Handler 接口的实现，下面我们就用一个示例来说明一下。

### 3. 自定义 Handler 接口的实现

我们来定义一个新 Handler：ChanHandler，该 Handler 实现将日志写入 channel 的行为（用来模拟日志写入 kafka)，我们来建立该 ChanHandler：

```
// github.com/bigwhite/experiments/tree/master/slog-examples/demo4/main.go
type ChanHandler struct {
    slog.Handler
    ch  chan []byte
    buf *bytes.Buffer
}

func (h *ChanHandler) Enabled(level slog.Level) bool {
    return h.Handler.Enabled(level)
}

func (h *ChanHandler) Handle(r slog.Record) error {
    err := h.Handler.Handle(r)
    if err != nil {
        return err
    }
    var nb = make([]byte, h.buf.Len())
    copy(nb, h.buf.Bytes())
    h.ch <- nb
    h.buf.Reset()
    return nil
}

func (h *ChanHandler) WithAttrs(as []slog.Attr) slog.Handler {
    return &ChanHandler{
        buf:     h.buf,
        ch:      h.ch,
        Handler: h.Handler.WithAttrs(as),
    }
}

func (h *ChanHandler) WithGroup(name string) slog.Handler {
    return &ChanHandler{
        buf:     h.buf,
        ch:      h.ch,
        Handler: h.Handler.WithGroup(name),
    }
}

func NewChanHandler(ch chan []byte) *ChanHandler {
    var b = make([]byte, 256)
    h := &ChanHandler{
        buf: bytes.NewBuffer(b),
        ch:  ch,
    }

    h.Handler = slog.NewJSONHandler(h.buf)

    return h
}


```

我们看到 ChanHandler 内嵌了 slog.JSONHandler，对 slog.Handler 接口的实现多半由内嵌的 JSONHandler 去完成，唯一不同的是 Handle 方法，这里要把 JSONHandler 处理完的日志 copy 出来并发送到 channel 中。下面是该 demo 的 main 函数：

```
// github.com/bigwhite/experiments/tree/master/slog-examples/demo4/main.go

func main() {
    var ch = make(chan []byte, 100)
    attrs := []slog.Attr{
        {Key: "field1", Value: slog.StringValue("value1")},
        {Key: "field2", Value: slog.StringValue("value2")},
    }
    slog.SetDefault(slog.New(NewChanHandler(ch).WithAttrs(attrs)))
    go func() { // 模拟channel的消费者，用来消费日志
        for {
            b := <-ch
            fmt.Println(string(b))
        }
    }()

    slog.Info("hello", "name", "Al")
    slog.Error("oops", net.ErrClosed, "status", 500)
    slog.LogAttrs(slog.ErrorLevel, "oops",
        slog.Int("status", 500), slog.Any("err", net.ErrClosed))

    time.Sleep(3 * time.Second)
}


```

运行上述程序，我们将得到下面输出结果：

```
{"time":"2022-10-23T23:09:01.358702+08:00","level":"INFO","msg":"hello","field1":"value1","field2":"value2","name":"Al"}

{"time":"2022-10-23T23:09:01.358836+08:00","level":"ERROR","msg":"oops","field1":"value1","field2":"value2","status":500,"err":"use of closed network connection"}

{"time":"2022-10-23T23:09:01.358856+08:00","level":"ERROR","msg":"oops","field1":"value1","field2":"value2","status":500,"err":"use of closed network connection"}


```

### 4. slog 的性能

我们再来看看 slog 的性能，我们直接使用了 slog 源码中自带的与 zap 的性能对比数据，使用 benchstat 工具查看对比结果如下：

```
$ benchstat zapbenchmarks/zap.bench slog.bench
name                              old time/op    new time/op    delta
Attrs/async_discard/5_args-8         348ns ± 2%      88ns ± 2%   -74.77%  (p=0.008 n=5+5)
Attrs/async_discard/10_args-8        570ns ± 2%     280ns ± 2%   -50.94%  (p=0.008 n=5+5)
Attrs/async_discard/40_args-8       1.84µs ± 2%    0.91µs ± 3%   -50.37%  (p=0.008 n=5+5)
Attrs/fastText_discard/5_args-8      476ns ± 2%     200ns ±45%   -57.92%  (p=0.008 n=5+5)
Attrs/fastText_discard/10_args-8     822ns ± 7%     524ns ± 2%   -36.27%  (p=0.008 n=5+5)
Attrs/fastText_discard/40_args-8    2.70µs ± 3%    2.01µs ± 3%   -25.76%  (p=0.008 n=5+5)

name                              old alloc/op   new alloc/op   delta
Attrs/async_discard/5_args-8          320B ± 0%        0B       -100.00%  (p=0.008 n=5+5)
Attrs/async_discard/10_args-8         640B ± 0%      208B ± 0%   -67.50%  (p=0.008 n=5+5)
Attrs/async_discard/40_args-8       2.69kB ± 0%    1.41kB ± 0%   -47.64%  (p=0.008 n=5+5)
Attrs/fastText_discard/5_args-8       320B ± 0%        0B       -100.00%  (p=0.008 n=5+5)
Attrs/fastText_discard/10_args-8      641B ± 0%      208B ± 0%   -67.55%  (p=0.008 n=5+5)
Attrs/fastText_discard/40_args-8    2.70kB ± 0%    1.41kB ± 0%   -47.63%  (p=0.029 n=4+4)

name                              old allocs/op  new allocs/op  delta
Attrs/async_discard/5_args-8          1.00 ± 0%      0.00       -100.00%  (p=0.008 n=5+5)
Attrs/async_discard/10_args-8         1.00 ± 0%      1.00 ± 0%      ~     (all equal)
Attrs/async_discard/40_args-8         1.00 ± 0%      1.00 ± 0%      ~     (all equal)
Attrs/fastText_discard/5_args-8       1.00 ± 0%      0.00       -100.00%  (p=0.008 n=5+5)
Attrs/fastText_discard/10_args-8      1.00 ± 0%      1.00 ± 0%      ~     (all equal)
Attrs/fastText_discard/40_args-8      1.00 ± 0%      1.00 ± 0%      ~     (all equal)


```

我们看到，slog 的性能相对于本就以高性能著称的 zap 还要好上不少，内存分配也减少很多。

### 5. 小结

通过对 slog 的初步探索，感觉 slog 整体上借鉴了 zap 等第三方 log 包的设计，都采用前后端分离的策略，但似乎又比 zap 好理解一些。

前面示例中提到了使用起来很方便的前端 API，谈到了 slog 的高性能，slog 设计目标中与 runtime tracing 集成在 proposal 中提及不多，更多谈到的是其与 context.Context 的集成 (通过 slog.WithContext 和 slog.FromContext 等)，也许这就是与 runtime tracing 集成的基础吧。

Jonathan Amsterdam 在 proposal 也提到过，每个第三方 log 包都有其特点，不指望 slog 能替换掉所有第三方 log 包，只是希望 slog 能与第三方 log 包充分交互，实现每个程序有一个共同的 “后端”。 一个有许多依赖关系的应用程序可能会发现，它已经连接了许多日志包。如果所有的包都支持 slog 提出的 Handler 接口，那么应用程序就可以创建一个单一的 Handler 并为每个日志库安装一次，以便在其所有的依赖中获得一致的日志。

个人观点：等 slog 加入标准库后，新项目推荐使用 slog。

本文涉及的示例代码可以在[这里](https://github.com/bigwhite/experiments/tree/master/slog-examples)下载。

### 6. 参考资料

*   Proposal: Structured Logging review – https://go-review.googlesource.com/c/proposal/+/444415/3/design/56345-structured-logging.md
*   discussion: structured, leveled logging – https://github.com/golang/go/discussions/54763
*   proposal: log/slog: structured, leveled logging – https://github.com/golang/go/issues/56345
*   slog 实验性实现 – https://github.com/golang/exp/tree/master/slog
*   logr – https://github.com/go-logr/logr
*   Go Logging Design Proposal – Ross Light – https://docs.google.com/document/d/1nFRxQ5SJVPpIBWTFHV-q5lBYiwGrfCMkESFGNzsrvBU/edit
*   Standardization around logging and related concerns – https://groups.google.com/g/golang-dev/c/F3l9Iz1JX4g/m/t0J0loRaDQAJ
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/4HN0-m8AlzSmDCuF9hYU4Q)

在大约一年前，我就写下了《[slog：Go 官方版结构化日志包 [1]](http://mp.weixin.qq.com/s?__biz=MzIyNzM0MDk0Mg==&mid=2247493688&idx=1&sn=b1dd65e5fc91ef96f7624f4e2492b559&chksm=e8600fd9df1786cf9fc4ac72bd47a72394617fa54823348f295d808dc1c16f3319e8f60b6c8a&scene=21#wechat_redirect)》一文，文中介绍了 Go 团队正在设计并计划在下一个 Go 版本中落地的 Go 官方结构化日志包：slog[2]。但 slog 并未如预期在 [Go 1.20 版本](http://mp.weixin.qq.com/s?__biz=MzIyNzM0MDk0Mg==&mid=2247494023&idx=1&sn=d70511791640a4ae1da21c6a8b4eb431&chksm=e8600e66df17877071a28cb9a74ddc20c265ebde32b1a7320cc154056b6926c67d70a3050c86&scene=21#wechat_redirect) [3] 中落地，而是在 golang.org/x/exp/slog 下面给出了 slog 的初始实现供社区体验。

时光飞逝，slog 在 golang.org/x/exp/slog 下经历了 1 年多时间的改善和演进，终于在最近发布的 [Go 1.21 版本](http://mp.weixin.qq.com/s?__biz=MzIyNzM0MDk0Mg==&mid=2247495344&idx=1&sn=e713d2d4121c71fe6da575874ceb324d&chksm=e8600951df1780477babe1e86028033dd01010fe7009a6ac7115916cbd0a170a02a09e3c4aed&scene=21#wechat_redirect) [4] 中以 log/slog 的包导入路径正式加入 Go 标准库。

正式版的 slog 在结构上并未作较大变化，依旧是分为前端和后端，因此讲 exp/slog 时的那幅图依然适用：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/cH6WzfQ94mZteWrIcUr9lKXCuT7hFEL7Dxvibj0f56RWvuGRD13D5upNUAu52sGCVUuyEpSFQzXiaKsey0x2SgjQ/640?wx_fmt=png)

不过，正式版的 slog 与当初那篇文章中的 exp/slog 在一些类型与 API 上已有不同。在这篇文章中，我就来简要说明一下。我这里讲述的思路大致是将《slog：Go 官方版结构化日志包 [5]》一文中的例子用 log/slog 改造一遍，这个过程可以让我们看到正式版 log/slog 与 exp/slog 的差异。

1. slog 快速入门
------------

### 1.1 slog 的 "hello, world"

如果仅是想以最快速的方式开始使用 slog，那么下面可以算是 slog 的 "hello, world" 版本：

```
//slog-examples-go121/demo0/main.go

package main
  
import (
    "log/slog"
)

func main() {
    slog.Info("my first slog msg", "greeting", "hello, slog")
}


```

运行这段程序，会得到下面输出：

```
$go run main.go
2023/08/29 05:01:36 INFO my first slog msg greeting="hello, slog"


```

### 1.2 TextHandler 和 JSONHandler

默认情况下，slog 输出的格式仅是普通 text 格式，而并非 JSON 格式，也不是以 key=value 形式呈现的文本。

slog 提供了以 JSON 格式输出的 JSONHandler 和以 key=value 形式呈现的文本形式的 TextHandler。不过要使用这两种 Handler 进行日志输出，我们需要显式创建它们：

```
//slog-examples-go121/demo1/main.go

h := slog.NewTextHandler(os.Stderr, nil)
l := slog.New(h)
l.Info("greeting", "name", "tony")
l.Error("oops", "err", net.ErrClosed, "status", 500)

h1 := slog.NewJSONHandler(os.Stderr, nil)
l1 := slog.New(h1)
l1.Info("greeting", "name", "tony")
l1.Error("oops", "err", net.ErrClosed, "status", 500)


```

> 注：相对于 exp/slog，正式版的 log/slog 的 NewTextHandler 和 NewJSONHandler 增加了一个新的 opts *HandlerOptions 参数。

上述代码分别创建了一个使用 TextHandler 的 slog.Logger 实例以及一个使用 JSONHandler 的 slog.Logger 实例，执行这段代码后将输出如下日志：

```
$go run main.go
time=2023-08-29T05:34:27.370+08:00 level=INFO msg=greeting name=tony
time=2023-08-29T05:34:27.370+08:00 level=ERROR msg=oops err="use of closed network connection" status=500
{"time":"2023-08-29T05:34:27.370306+08:00","level":"INFO","msg":"greeting","name":"tony"}
{"time":"2023-08-29T05:34:27.370315+08:00","level":"ERROR","msg":"oops","err":"use of closed network connection","status":500}


```

如果觉得每次还得使用 l 或 l1 来调用 Info、Error 等输出日志的函数不便利，可以将 l 或 l1 设置为 Default Logger，这样无论在任何包内都可以直接通过 slog 包级函数，如 Info、Error 等直接输出日志：

```
//slog-examples-go121/demo1/main.go

time=2023-08-29T05:40:08.503+08:00 level=INFO msg="textHandler after setDefault" name=tony age=30
{"time":"2023-08-29T05:40:08.503672+08:00","level":"INFO","msg":"jsonHandler after setDefault","name":"tony","age":30}


```

> 注：相对于 exp/slog，正式版的 log/slog 提供了带有 Context 的 Info、Error 日志输出函数：DebugContext、InfoContext、ErrorContext 等。

### 1.3 HandlerOption

通过在创建 Handler 时传入自定义的 HandlerOption，我们可以设置 Logger 的日志级别和是否输出 Source，比如下面示例：

```
//slog-examples-go121/demo2/main.go

opts := slog.HandlerOptions{
AddSource: true,
Level:     slog.LevelError,
}

slog.SetDefault(slog.New(slog.NewJSONHandler(os.Stderr, &opts)))
slog.Info("open file for reading", "name", "foo.txt", "path", "/home/tonybai/demo/foo.txt")
slog.Error("open file error", "err", os.ErrNotExist, "status", 2)


```

上述代码通过 HandlerOption 设置了 Handler 仅输出 Error 级别日志，并在输出的日志中带上 Source 信息，运行这段程序，会得到下面输出：

```
$go run main.go
{"time":"2023-08-29T05:18:18.068213+08:00","level":"ERROR","source":{"function":"main.main","file":"/Users/tonybai/Go/src/github.com/bigwhite/experiments/slog-examples-go121/demo2/main.go","line":17},"msg":"open file error","err":"file does not exist","status":2}


```

我们看到通过 Info 函数输出的日志并没有被仅处理 Error 级别的 Handler 输出到 console 上。另外在输出的日志中，我们看到了 source 这个 key，以及它的值，即输出日志的那行代码在源代码文件中位置。

### 1.4 属性字段

我们日常输出的日志都有一些共同的字段，比如上面的 level、time，这些字段被称为属性。slog 支持带有属性 (attribute) 的日志输出，slog 内置了若干属性，如下面代码所示：

```
// log/slog/handler.go

// Keys for "built-in" attributes.
const (
    // TimeKey is the key used by the built-in handlers for the time
    // when the log method is called. The associated Value is a [time.Time].
    TimeKey = "time"
    // LevelKey is the key used by the built-in handlers for the level
    // of the log call. The associated value is a [Level].
    LevelKey = "level"
    // MessageKey is the key used by the built-in handlers for the
    // message of the log call. The associated value is a string.
    MessageKey = "msg"
    // SourceKey is the key used by the built-in handlers for the source file
    // and line of the log call. The associated value is a string.
    SourceKey = "source"
)


```

当然 slog 也支持自定义属性：

```
//slog-examples-go121/demo2/main.go

l := slog.Default().With("attr1", "attr1_value", "attr2", "attr2_value")
l.Error("connect server error", "err", net.ErrClosed, "status", 500)
l.Error("close conn error", "err", net.ErrClosed, "status", 501)


```

在上面的代码中，我们定义了两个属性：attr1 和 attr2，以及它们的值，这样当我们用带有这两个属性的 Logger 输出日志时，每行日志都会包含这两个属性：

```
{"time":"2023-08-29T05:28:39.322014+08:00","level":"ERROR","source":{"function":"main.main","file":"/Users/tonybai/Go/src/github.com/bigwhite/experiments/slog-examples-go121/demo2/main.go","line":23},"msg":"connect server error","attr1":"attr1_value","attr2":"attr2_value","err":"use of closed network connection","status":500}
{"time":"2023-08-29T05:28:39.322028+08:00","level":"ERROR","source":{"function":"main.main","file":"/Users/tonybai/Go/src/github.com/bigwhite/experiments/slog-examples-go121/demo2/main.go","line":24},"msg":"close conn error","attr1":"attr1_value","attr2":"attr2_value","err":"use of closed network connection","status":501}


```

当然你也可以通过 slog.LogAttrs 做 “一次性” 的属性输出：

```
//slog-examples-go121/demo2/main.go

l.LogAttrs(context.Background(), slog.LevelError, "log with attribute once", slog.String("attr3", "attr3_value"))
l.Error("reconnect error", "err", net.ErrClosed, "status", 502)


```

这两行输出如下日志：

```
{"time":"2023-08-29T05:32:00.419772+08:00","level":"ERROR","source":{"function":"main.main","file":"/Users/tonybai/Go/src/github.com/bigwhite/experiments/slog-examples-go121/demo2/main.go","line":26},"msg":"log with attribute once","attr1":"attr1_value","attr2":"attr2_value","attr3":"attr3_value"}
{"time":"2023-08-29T05:32:00.419778+08:00","level":"ERROR","source":{"function":"main.main","file":"/Users/tonybai/Go/src/github.com/bigwhite/experiments/slog-examples-go121/demo2/main.go","line":27},"msg":"reconnect error","attr1":"attr1_value","attr2":"attr2_value","err":"use of closed network connection","status":502}


```

我们看到通过 LogAttrs 输出的 attr3 属性仅出现一次。

> 注：相对于 exp/slog，正式版的 log/slog 提供的 LogAttrs 方法多了一个 context.Context 参数。

### 1.5 Group 形式的日志输出

slog 支持 group 形式的日志输出，这点保持了与 exp/slog 的一致。下面是一个输出 group log 的例子：

```
//slog-examples-go121/demo2/main.go

gl := l.WithGroup("response")
gl.Error("http post response", "code", 403, "status", "server not response", "server", "10.10.121.88")


```

我们先创建一个名为 “response" 的 group logger，然后调用 Error 输出日志。Error 会将所有 attribute 之外的字段放入 response 这个 group 中呈现，我们看一下运行结果：

```
{"time":"2023-08-29T12:54:07.623002+08:00","level":"ERROR","source":{"function":"main.main","file":"/Users/tonybai/Go/src/github.com/bigwhite/experiments/slog-examples-go121/demo2/main.go","line":30},"msg":"http post response","attr1":"attr1_value","attr2":"attr2_value","response":{"code":403,"status":"server not response","server":"10.10.121.88"}}


```

2. 动态调整日志级别
-----------

exp/slog 使用 slog.AtomicLevel 实现 Logger 级别的动态调整。在正式版 slog 中，我们则使用 slog.LevelVar 来实现 Logger 日志级别的动态调整，使用方法差不多，看下面这个例子：

```
// slog-examples-go121-demo3/main.go

func main() {
    var lvl slog.LevelVar
    lvl.Set(slog.LevelDebug)
    opts := slog.HandlerOptions{
        Level: &lvl,
    }
    slog.SetDefault(slog.New(slog.NewJSONHandler(os.Stderr, &opts)))

    slog.Info("before resetting log level:")

    slog.Info("greeting", "name", "tony")
    slog.Error("oops", "err", net.ErrClosed, "status", 500)
    slog.LogAttrs(context.Background(), slog.LevelError, "oops",
        slog.Int("status", 500), slog.Any("err", net.ErrClosed))

    slog.Info("after resetting log level to error level:")
    lvl.Set(slog.LevelError)
    slog.Info("greeting", "name", "tony")
    slog.Error("oops", "err", net.ErrClosed, "status", 500)
    slog.LogAttrs(context.Background(), slog.LevelError, "oops",
        slog.Int("status", 500), slog.Any("err", net.ErrClosed))

}


```

结合 LevelVar 和 HandlerOption，我们实现了 Logger 日志级别的动态调整，这里是由 LevelDebug 调整为 LevelError。上面示例的输出结果如下：

```
{"time":"2023-08-29T06:15:26.103022+08:00","level":"INFO","msg":"before resetting log level:"}
{"time":"2023-08-29T06:15:26.103197+08:00","level":"INFO","msg":"greeting","name":"tony"}
{"time":"2023-08-29T06:15:26.103203+08:00","level":"ERROR","msg":"oops","err":"use of closed network connection","status":500}
{"time":"2023-08-29T06:15:26.103222+08:00","level":"ERROR","msg":"oops","status":500,"err":"use of closed network connection"}
{"time":"2023-08-29T06:15:26.103226+08:00","level":"INFO","msg":"after resetting log level to error level:"}
{"time":"2023-08-29T06:15:26.103232+08:00","level":"ERROR","msg":"oops","err":"use of closed network connection","status":500}
{"time":"2023-08-29T06:15:26.103236+08:00","level":"ERROR","msg":"oops","status":500,"err":"use of closed network connection"}


```

我们看到，动态调整到 LevelError 后，Info 函数打印的日志将不再输出到 console 了。

3. 自定义后端 Handler
----------------

在《slog：Go 官方版结构化日志包 [6]》一文中，我们就举例说明了如何自定义一个后端 Handler，正式版 slog 在自定义 Handler 这方面变化不大，都是通过实现 slog.Handler 接口的方式达成的。大家可自行查看 slog-examples-go121/demo4 中的代码，这里就不赘述了。

此外，log/slog 的作者 Jonathan Amsterdam 还提供了一篇 “slog 自定义 handler 指南”[7] 供大家参考。

4. 验证 handler
-------------

Go 1.21 正式版提供了一个 testing/slogtest 包可以用来辅助测试自定义后端 Handler，我们就以 slog-examples-go121/demo4 中自定义的 ChanHandler 为例，用 slogtest 包对其进行一下测试：

```
// slog-examples-go121/demo4/handler_test.go

func TestChanHandlerParsing(t *testing.T) {
    var ch = make(chan []byte, 100)
    h := NewChanHandler(ch)

    results := func() []map[string]any {
        var ms []map[string]any
        ticker := time.NewTicker(time.Second)
    loop:
        for {
            select {
            case line := <-ch:
                if len(line) == 0 {
                    break
                }
                var m map[string]any
                if err := json.Unmarshal(line, &m); err != nil {
                    t.Fatal(err)
                }
                ms = append(ms, m)
            case <-ticker.C:
                break loop
            }
        }
        return ms
    }
    err := slogtest.TestHandler(h, results)
    if err != nil {
        log.Fatal(err)
    }
}


```

slogtest 仅提供一个导出函数 TestHandler，它会自动基于你提供的 Handler 创建 Logger 并向 Logger 写入一些日志，然后通过传入的 results 函数对写入的日志进行格式验证，主要是 json 反序列化，如果成功，会记录在 map[string]any 类型的切片中。最后 TestHandler 会比对写入日志条数与反序列化成功的条数，如果一致，说明测试 ok，反之则测试失败。

> 注：基于这个 TestHandler，还真测试出原 ChanHandler 的一个问题，已经 fix。

5. 性能 tips
----------

按官方 benchmark 结果，log/slog 的性能要高于 Go 社区常用的结构化日志包，比如 zap[8] 等。

即便如此，log 在 go 应用中带来的延迟依旧不可忽视。slog 的 proposal design[9] 中给出了一些关于性能的考量和 tip，大家可以在日后使用 slog 时借鉴：

*   使用 Logger.With 避免重复格式化公共属性字段，这使得处理程序可以缓存格式化结果。
    
*   将昂贵的计算推迟到日志输出时再进行，例如传递指针而不是格式化后的字符串。这可以避免在禁用的日志行上进行不必要的工作。
    
*   对于昂贵的值，可以实现 LogValuer 接口，这样在输出时可以进行 lazy 加载计算。
    

```
// log/slog/value.go
  
// A LogValuer is any Go value that can convert itself into a Value for logging.
//
// This mechanism may be used to defer expensive operations until they are
// needed, or to expand a single value into a sequence of components.
type LogValuer interface {
    LogValue() Value
}


```

最后，内置的 Handler 已经处理了原子写入的加锁。自定义 Handler 应该实现自己的加锁。

6. 小结
-----

总体来说，slog 正式版与之前实现相比，接口变化不大，功能也基本保持不变，但代码质量、性能、文档等有较大改进，符合预期。

slog 填补了 Go 标准库在结构化日志支持上的短板，提供了简洁、易用、易扩展的 API。相信随着 slog 的推广，可以逐步统一 Go 社区中的日志实践，也让更多人受益。

个人建议：新项目如果没有使用第三方日志包，可以直接采用 slog，无需再考虑 zap、zerolog 等第三方选择。对于没有升级到 Go 1.21 版本的新项目，也可以使用 exp/slog，目前 exp/slog 也已经与 log/slog 保持了同步。

本文涉及的示例代码可以在这里 [10] 下载。

7. 参考资料
-------

*   Proposal: Structured Logging - https://go.googlesource.com/proposal/+/master/design/56345-structured-logging.md
    
*   slog 包手册 - https://pkg.go.dev/log/slog
    
*   Structured Logging with slog - https://go.dev/blog/slog
    
*   A Guide to Writing slog Handlers - https://github.com/golang/example/blob/master/slog-handler-guide/README.md
    

* * *

### 参考资料

[1] 

slog：Go 官方版结构化日志包: _https://tonybai.com/2022/10/30/first-exploration-of-slog_

[2] 

slog: _https://pkg.go.dev/log/slog_

[3] 

Go 1.20 版本: _https://tonybai.com/2023/02/08/some-changes-in-go-1-20/_

[4] 

Go 1.21 版本: _https://tonybai.com/2023/08/20/some-changes-in-go-1-21/_

[5] 

slog：Go 官方版结构化日志包: _https://tonybai.com/2022/10/30/first-exploration-of-slog_

[6] 

slog：Go 官方版结构化日志包: _https://tonybai.com/2022/10/30/first-exploration-of-slog_

[7] 

“slog 自定义 handler 指南”: _https://github.com/golang/example/blob/master/slog-handler-guide/guide.md_

[8] 

zap: _https://tonybai.com/2021/07/14/uber-zap-advanced-usage_

[9] 

proposal design: _https://go.googlesource.com/proposal/+/master/design/56345-structured-logging.md_

[10] 

这里: _https://github.com/bigwhite/experiments/tree/master/slog-examples-go121_

[11] 

“Gopher 部落” 知识星球: _https://public.zsxq.com/groups/51284458844544_

[12] 

链接地址: _https://m.do.co/c/bff6eed92687_
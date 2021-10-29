> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/zhounixing/article/details/105815910)

一、背景介绍

        由于在微服务架构中，服务之间的调用关系多而复杂，所以有必要对它们之间的调用链路进行追踪、分析，判断是哪里出了问题，或者哪里耗时过多。

        最近接到了这个需求，添加全链路追踪，所以研究并实践了一下，还不太深刻，若有错误的地方欢迎指正。

二、OpenTracing 相关概念介绍

        首先，要实现全链路追踪，必须先理解 OpenTracing 的一些基本概念。OpenTracing 为分布式链路追踪制定了一个统一的标准。只要是按照此标准实现的服务，就能够完整的进行分布式追踪。

    1. Span

        Span 可以被翻译为跨度，可以理解为一次方法调用，一个程序块的调用，或者一次 RPC / 数据库访问。

        Span 之间是有关系的，child of 和 follow of。比如一次 RPC 的调用，RPC 客户端和服务端的 span 就形成了父子关系。

    2. Trace

        Trace 表示一个调用链，比如在分布式服务中，一个客户端的请求，在后台可能经过了层层的调用，那么每一次调用就相当于一个 span，而这一整条调用链路，可以理解成一个 trace。

        Trace 有一个全局唯一的 ID。

三、Go2sky 简介

        Go2sky 是 Golang 提供给开发者实现 SkyWalking agent 探针的包，可以通过它来实现向 SkyWalking Collector 上报数据。

        快速入门：[GitHub-Go2Sky](https://github.com/SkyAPM/go2sky)

        1. 创建 Reporter、Tracer

            SkyWalking 支持 http 和 gRpc 两种方式收集数据，在 Go2sky 中，想要上报数据，先创建一个 GRPCReporter.

            Tracer 代表了本程序中的一条调用链路。

            ![](https://img-blog.csdnimg.cn/20200428160958852.png)

            本程序中的所有 span 都会与服务名为 example 的服务相关联。

        2. 创建 Span

            Span 有三种类型：LocalSpan、EntrySpan、ExitSpan。

            LocalSpan：可以用来表示本程序内的一次调用。

            EntrySpan：用来从下游服务提取 context 信息。

            ExitSpan：  用来向上游服务注入 context 信息。

            ![](https://img-blog.csdnimg.cn/20200428161926671.png)

            在创建 span 时，上下文参数传入 context.Backround() ，就表示它是 root span。

        3. 创建 sub span

            在创建 LocalSpan 和 EntrySpan 的时候，返回值会返回一个 context 信息 (ctx)，通过它来创建 sub span，来与 root span 形成父子关系。

            ![](https://img-blog.csdnimg.cn/20200428162448317.png)

        4. End Span

            必须要确保结束 span，它们才可以被上传给 skywalking。

            ![](https://img-blog.csdnimg.cn/20200428173403142.png)

        5. 关联 Span

            我们在程序中创建的 span，是怎么关联起来形成一个调用链的呢。

            在同一个程序中，向上面那样，创建 root span 和 sub span 即可。

            在不同的程序中，下游服务使用 ExitSpan 向上游注入 context 信息，上游服务使用 EntrySpan 从下游提取 context 信息。Entry 和 Exit 使得 skywalking 可以分析，从而生成拓扑图和度量指标。

            ![](https://img-blog.csdnimg.cn/20200428163215305.png)

四、实战 -- 跨程序追踪 RPC 调用

        看到这里，有了基本的概念，以及 Go2sky 的基本用法，但是仍然不能够对 RPC 进行有效的追踪。

        因为上图中的例子使用的是 http 请求，它本身就封装了 Get 和 Set 方法，可以很轻松的注入和提取 context 信息。但是 RPC 请求并没有，想要追踪别的类型跨程序的调用也没有。

        所以我们要自己将 context 信息在进行调用的时候，从下游服务传给上游服务，然后自己定义注入和提取的方法。

        下面只贴出了链路追踪部分的代码，其它的比如 rpc 相关的部分代码省略了 (不然又臭又长，还难看)。

    1. Client 端 (下游服务)

        定义请求信息的结构体：

```
type Req struct {
    A       int
    Header  string        // 添加此字段，用于传递context信息
}
```

        定义 context 信息的注入方法：

```
func (p *Req) Set(key, value string) error {
    p.Header = fmt.Sprintf("%s:%s", key, value)
    return nil
}
```

        创建 reporter 和 tracer：

```
r, err = reporter.NewGRPCReporter("192.168.204.130:11800")
if err != nil {
    logs.Info("[New GRPC Reporter Error]: [%v]", err)
    return
}
 
// 这个程序中所有的span都会跟服务名叫RTS_Test的服务关联起来
tracer, err = go2sky.NewTracer("RTS_Test", go2sky.WithReporter(r), go2sky.WithInstance("RTS_Test_1"))
if err != nil {
    logs.Info("[New Tracer Error]: [%v]", err)
    return
}
tracer.WaitUntilRegister()
```

        rpc 调用以及创建 span：

        在创建 ExitSpan 的时候，传入了一个函数，函数实现就是我们定义的如何注入 context 信息的函数。

        它会在 CreateExitSpan() 函数的内部被调用，header 的值不需要我们管，它在 CreateExitSpan 函数内部生成的。我们只需要负责在上游服务中把它提取出来即可。

        我目前的理解是，只需要在下游服务中负责把这个 header 按一定规则拼接，传给上游服务，然后在上游服务中按照规则将 header 解析出来，skywalking 通过分析，即可将上下游的 span 关联起来。

```
func OnSnapshot() {
    // client := GetClinet()
    // 表示收到客户端请求，因为只追踪后台服务之间的链路，所以这里不需要提取context信息
    span2, ctx, err := tracer.CreateEntrySpan(context.Background(), "/API/Snapshot", func() (string, error){
        return "", nil
    })
    if err != nil {
        logs.Info("[Create Exit Span Error]: [%v]", err)
        return
    }
    span2.SetComponent(5200)
	
    // 表示rpc调用的span，这里需要向上游服务注入context信息，即参数中的header
    req := Req{3, ""}
    span1, err := tracer.CreateExitSpan(ctx, "/Service/OnSnapshot", "RTS_Server", func(header string) error{
        return req.Set(propagation.Header, header)
    })
    if err != nil {
        logs.Info("[Create Exit Span Error]: [%v]", err)
        return
    }
    span1.SetComponent(5200)    // Golang程序使用范围是[5000, 6000)，还要在skywalking中配置，config目录下的component-libraries.yml文件
 
    var res Res
    // rpc调用
    err = conn.Call("Req.Snapshot", req, &res)
    if err != nil {
        logs.Info("[RPC Call Snapshot Error]: [%v]", err)
        return
    } else {
        logs.Info("[RPC Call Snapshot Success]: [%s]", res)
    }
 
    span1.End()
    span2.End()    // 一定要确保span被结束
 
    // s1 := ReportedSpan(span1)
    // s2 := ReportedSpan(span2)
    // spans := []go2sky.ReportedSpan{s1, s2}
    // r.Send(spans)
}
```

    2. Server 端 (上游服务)

        定义请求信息的结构体：

```
type ReqBody struct {
    A       int
    Header  string
}
```

        定义 context 信息的提取方法：

```
func (p *ReqBody) Get(key string) string {
    subs := strings.Split(p.Header, ":")
    if len(subs) != 2 || subs[0] != key {
	    return ""
    }
 
    return subs[1]
}
```

        创建 reporter 和 tracer：

```
r, err = reporter.NewGRPCReporter("192.168.204.130:11800")
if err != nil {
    logs.Info("[New GRPC Reporter Error]: [%v]\n", err)
    return
}
 
tracer, err = go2sky.NewTracer("Service_Test", go2sky.WithReporter(r), go2sky.WithInstance("Service_Test_1"))
if err != nil {
    logs.Info("[New Tracer Error]: [%v]\n", err)
    return
}
tracer.WaitUntilRegister()
```

        创建 span：

        在创建 EntrySpan 时，调用 Get() 方法提取 context 信息

```
func (p *Req)Snapshot(req ReqBody, res *Res) error {
    // 表示收到 rpc 客户端的请求，这里需要提取context信息
    span1, ctx, err := tracer.CreateEntrySpan(context.Background(), "/Service/OnSnapshot/QueringSnapshot", func() (string, error){
	return req.Get(propagation.Header), nil
    })
    if err != nil {
	    logs.Info("[Create Exit Span Error]: [%v]\n", err)
	    return err
    }
    span1.SetComponent(5200)
    // span1.SetPeer("Service_Test")
 
    // 表示去请求了一次数据库
    span2, err := tracer.CreateExitSpan(ctx, "/database/QuerySnapshot", "APIService", func(header string) error {
	    return nil
    })
    span2.SetComponent(5200)
 
    time.Sleep(time.Millisecond * 6)
    *res = "Return Snapshot Info"
 
    span2.End()
    span1.End()
 
    // s1 := ReportedSpan(span1)
    // s2 := ReportedSpan(span2)
    // spans := []go2sky.ReportedSpan{s1, s2}
    // r.Send(spans)
 
    return nil
}
```

    3. 结果展示 

        链路追踪：

        ![](https://img-blog.csdnimg.cn/20200428172223895.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pob3VuaXhpbmc=,size_16,color_FFFFFF,t_70)

         拓扑图：

        ![](https://img-blog.csdnimg.cn/20200428172845642.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pob3VuaXhpbmc=,size_16,color_FFFFFF,t_70)
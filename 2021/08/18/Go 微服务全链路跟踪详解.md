> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5OTA1MDUyMA==&mid=2655462989&idx=4&sn=687226956640f0f4684c870184e65e0e&chksm=bd72ea3a8a05632c330b40f943e6de2f44e104e947247f18008c9a9e4a06128f757b78290e5c&scene=21#wechat_redirect)

【导读】微服务架构下，全链路追踪 OpenTracing 打印的调用链信息可以帮助我们快速定位和排查问题。本文介绍了 OpenTracing 的用法。  

在微服务架构中，调用链是漫长而复杂的，要了解其中的每个环节及其性能，你需要全链路跟踪。它的原理很简单，你可以在每个请求开始时生成一个唯一的 ID，并将其传递到整个调用链。该 ID 称为 CorrelationID，你可以用它来跟踪整个请求并获得各个调用环节的性能指标。简单来说有两个问题需要解决。第一，如何在应用程序内部传递 ID; 第二，当你需要调用另一个微服务时，如何通过网络传递 ID。

什么是 OpenTracing?
----------------

现在有许多开源的分布式跟踪库可供选择，其中最受欢迎的库可能是 Zipkin 和 Jaeger。  
选择哪个是一个令人头疼的问题，因为你现在可以选择最受欢迎的一个，但是如果以后有一个更好的出现呢？OpenTracing  
可以帮你解决这个问题。它建立了一套跟踪库的通用接口，这样你的程序只需要调用这些接口而不被具体的跟踪库绑定，将来可以切换到不同的跟踪库而无需更改代码。Zipkin 和 Jaeger 都支持 OpenTracing。

如何跟踪服务端口 (server endpoints)?
----------------------------

在下面的程序中我使用 “Zipkin” 作为跟踪库，用 “OpenTracing” 作为通用跟踪接口。跟踪系统中通常有四个组件，下面我用 Zipkin 作为示例：

*   recorder(记录器)：记录跟踪数据
    
*   Reporter (or collecting agent)(报告器或收集代理)：从记录器收集数据并将数据发送到 UI 程序
    
*   Tracer：生成跟踪数据
    
*   UI：负责在图形 UI 中显示跟踪数据
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/IgylNib7ZE2J9ehzUvrQkJC5pAJmLkbaD5CVDMU956cdxbd9nEdZfjSQKIbUaJibeGckvjy1hrZW5GDqa9PekByg/640?wx_fmt=jpeg)

上面是 Zipkin 的组件图，你可以在 Zipkin Architecture 中找到它。

有两种不同类型的跟踪，一种是进程内跟踪（in-process），另一种是跨进程跟踪（cross-process）。我们将首先讨论跨进程跟踪。

### 客户端程序:

我们将用一个简单的 gRPC 程序作为示例，它分成客户端和服务器端代码。我们想跟踪一个完整的服务请求，它从客户端到服务端并从服务端返回。以下是在客户端创建新跟踪器的代码。它首先创建 “HTTP Collector”(the agent) 用来收集跟踪数据并将其发送到 “Zipkin” UI， “endpointUrl” 是“Zipkin” UI 的 URL。其次，它创建了一个记录器 (recorder) 来记录端口上的信息，“hostUrl”是 gRPC(客户端)呼叫的 URL。第三，它用我们新建的记录器创建了一个新的跟踪器 (tracer)。最后，它为“OpenTracing” 设置了“GlobalTracer”，这样你可以在程序中的任何地方访问它。

```
const (
    endpoint_url = "http://localhost:9411/api/v1/spans"
    host_url = "localhost:5051"
    service_name_cache_client = "cache service client"
    service_name_call_get = "callGet"
)

func newTracer () (opentracing.Tracer, zipkintracer.Collector, error) {
 collector, err := openzipkin.NewHTTPCollector(endpoint_url)
 if err != nil {
  return nil, nil, err
 }
 recorder :=openzipkin.NewRecorder(collector, true, host_url, service_name_cache_client)
 tracer, err := openzipkin.NewTracer(
  recorder,
  openzipkin.ClientServerSameSpan(true))

 if err != nil {
  return nil,nil,err
 }
 opentracing.SetGlobalTracer(tracer)

 return tracer,collector, nil
}
```

以下是 gRPC 客户端代码。它首先调用上面提到的函数 “newTrace()” 来创建跟踪器，然后，它创建一个包含跟踪器的 gRPC 调用连接。接下来，它使用新建的 gRPC 连接创建缓存服务 (Cache service) 的 gRPC 客户端。最后，它通过 gRPC 客户端来调用缓存服务的 “Get” 函数。

```
key:="123"
 tracer, collector, err :=newTracer()
 if err != nil {
  panic(err)
 }
 defer collector.Close()
 connection, err := grpc.Dial(host_url,
  grpc.WithInsecure(), grpc.WithUnaryInterceptor(otgrpc.OpenTracingClientInterceptor(tracer, otgrpc.LogPayloads())),
  )
 if err != nil {
  panic(err)
 }
 defer connection.Close()
 client := pb.NewCacheServiceClient(connection)
 value, err := callGet(key, client)
```

### Trace 和 Span:

在 OpenTracing 中，一个重要的概念是 “trace”，它表示从头到尾的一个请求的调用链，它的标识符是 “traceID”。  
一个 “trace” 包含有许多跨度 (span)，每个跨度捕获调用链内的一个工作单元，并由“spanId” 标识。每个跨度具有一个父跨度，并且一个 “trace” 的所有跨度形成有向无环图(DAG)。以下是跨度之间的关系图。你可以从 The OpenTracing Semantic Specification 中找到它。

![](https://mmbiz.qpic.cn/mmbiz_jpg/IgylNib7ZE2J9ehzUvrQkJC5pAJmLkbaDgcyhax4nOesMWDiaL08Efzmxq2Be1LiaA5aO20paIk9xNBsfclGsUUVg/640?wx_fmt=jpeg)

以下是函数 “callGet” 的代码，它调用了 gRPC 服务端的“Get" 函数。在函数的开头，OpenTracing 为这个函数调用开启了一个新的 span，整个函数结束后，它也结束了这个 span。

```
const service_name_call_get = "callGet"

func callGet(key string, c pb.CacheServiceClient) ( []byte, error) {
    span := opentracing.StartSpan(service_name_call_get)
    defer span.Finish()
    time.Sleep(5*time.Millisecond)
    // Put root span in context so it will be used in our calls to the client.
    ctx := opentracing.ContextWithSpan(context.Background(), span)
    //ctx := context.Background()
    getReq:=&pb.GetReq{Key:key}
    getResp, err :=c.Get(ctx, getReq )
    value := getResp.Value
    return value, err
}
```

### 服务端代码:

下面是服务端代码，它与客户端代码类似，它调用了 “newTracer()”(与客户端“newTracer()” 函数几乎相同)来创建跟踪器。然后，它创建了一个 “OpenTracingServerInterceptor”，其中包含跟踪器。最后，它使用我们刚创建的拦截器(Interceptor) 创建了 gRPC 服务器。

```
    connection, err := net.Listen(network, host_url)
 if err != nil {
  panic(err)
 }
 tracer,err  := newTracer()
 if err != nil {
  panic(err)
 }
 opts := []grpc.ServerOption{
  grpc.UnaryInterceptor(
   otgrpc.OpenTracingServerInterceptor(tracer,otgrpc.LogPayloads()),
  ),
 }
 srv := grpc.NewServer(opts...)
 cs := initCache()
 pb.RegisterCacheServiceServer(srv, cs)

 err = srv.Serve(connection)
 if err != nil {
  panic(err)
 } else {
  fmt.Println("server listening on port 5051")
 }
```

以下是运行上述代码后在 Zipkin 中看到的跟踪和跨度的图片。在服务器端，我们不需要在函数内部编写任何代码来生成 span，我们需要做的就是创建跟踪器（tracer），服务器拦截器自动为我们生成 span。

![](https://mmbiz.qpic.cn/mmbiz_jpg/IgylNib7ZE2J9ehzUvrQkJC5pAJmLkbaDSkffWxztNOOTcJjB3bzYWNdF4bWR4RlCh2mTIu2VeE8Wq0OnDFAuCw/640?wx_fmt=jpeg)

怎样跟踪函数内部?
---------

上面的图片没有告诉我们函数内部的跟踪细节， 我们需要编写一些代码来获得它。

以下是服务器端 “get” 函数，我们在其中添加了跟踪代码。它首先从上下文（context）获取跨度 (span)，然后创建一个新的子跨度并使用我们刚刚获得的跨度作为父跨度。接下来，它执行一些操作(例如数据库查询)，然后结束(mysqlSpan.Finish()) 子跨度。

```
const service_name_db_query_user = "db query user"

func (c *CacheService) Get(ctx context.Context, req *pb.GetReq) (*pb.GetResp, error) {
    time.Sleep(5*time.Millisecond)
    if parent := opentracing.SpanFromContext(ctx); parent != nil {
        pctx := parent.Context()
        if tracer := opentracing.GlobalTracer(); tracer != nil {
            mysqlSpan := tracer.StartSpan(service_name_db_query_user, opentracing.ChildOf(pctx))
            defer mysqlSpan.Finish()
            //do some operations
            time.Sleep(time.Millisecond * 10)
        }
    }
    key := req.GetKey()
    value := c.storage[key]
    fmt.Println("get called with return of value: ", value)
    resp := &pb.GetResp{Value: value}
    return resp, nil

}
```

以下是它运行后的图片。现在它在服务器端有一个新的跨度 “db query user”。

![](https://mmbiz.qpic.cn/mmbiz_jpg/IgylNib7ZE2J9ehzUvrQkJC5pAJmLkbaDhqg6LDicTekyic21wvz18YoVTBzM9icMM6e8eGzmdGt7XbicOHibiaxic3HOg/640?wx_fmt=jpeg)

以下是 zipkin 中的跟踪数据。你可以看到客户端从 8.016ms 开始，服务端也在同一时间启动。服务器端完成大约需要 16ms。

![](https://mmbiz.qpic.cn/mmbiz_jpg/IgylNib7ZE2J9ehzUvrQkJC5pAJmLkbaDkGmDiaB0obkUhDevEZK5zDsMxGKe2xxqwnDHnj98pRtyVHy8m97e6HA/640?wx_fmt=jpeg)

怎样跟踪数据库?
--------

怎样才能跟踪数据库内部的操作？首先，数据库的驱动程序需要有跟踪功能，另外你需要将跟踪器 (tracer) 传递到数据库函数中。如果数据库驱动程序不支持跟踪怎么办？现在已经有几个开源的驱动程序封装器(Wrapper)，它们可以封装任何数据库驱动程序并使其支持跟踪。其中一个是 instrumentedsql(另外两个是 luna-duclos/instrumentedsql 和 ocsql/driver.go)。  
我简要地看了一下他们的代码，他们的原理基本相同。它们都为底层数据库的每个函数创建了一个封装 (Wrapper)，  
并在每个数据库操作之前启动一个新的跨度，在操作完成后结束跨度。但是所有这些都只封装了 “database/sql” 接口，  
这就意味着 NoSQL 数据库没有办法使用他们。如果你的 NoSQL 数据库（例如 MongoDB) 的驱动程序不支持 OpenTracing，你可能需要自己编写一个封装 (Wrapper), 它并不困难。

一个问题是 “如果我在程序中使用 OpenTracing 和 Zipkin 而数据库驱动程序使用 Openeracing 和 Jaeger，那会有问题吗？" 这其实不会发生。我上面提到的大部分封装都支持 OpenTracing。在使用封装时，你需要注册（register）封装了的 SQL 驱动程序，其中包含跟踪器。在 SQL 驱动程序内部，所有跟踪函数都只调用了 OpenTracing 的接口，因此它们甚至不知道底层实现是 Zipkin 还是 Jaeger。现在使用 OpenTarcing 的好处终于体现出来了。在应用程序中创建全局跟踪器时 (Global tracer)，你需要决定是使用 Zipkin 还是 Jaeger，但这之后，应用程序或第三方库中的每个函数都只调用 OpenTracing 接口，  
已经与具体的跟踪库 (Zipkin 或 Jaeger) 没关系了。

怎样跟踪服务调用 \？
-----------

假设我们需要在 gRPC 服务中调用另外一个微服务 (例如 RESTFul 服务)，该如何跟踪？

简单来说就是使用 HTTP 头作为媒介（Carrier）来传递跟踪信息 (traceID)。无论微服务是 gRPC 还是 RESTFul，它们都使用 HTTP 协议。如果是消息队列(Message Queue)，则将跟踪信息(traceID) 放入消息报头中 (Zipkin B3-propogation 有“single header” 和“multiple header”有两种不同类型的跟踪信息，但 JMS 仅支持“single header”)。

一个重要的概念是 “跟踪上下文(trace context)”，它定义了传播跟踪所需的所有信息，例如 traceID，parentId(父 spanId) 等。有关详细信息，请阅读跟踪上下文 (trace context)。OpenTracing 提供了两个处理“跟踪上下文(trace context)” 的函数：“extract(format，carrier)”和 “inject(SpanContext，format，carrier)”.“extarct()” 从媒介（通常是 HTTP 头）获取跟踪上下文。“inject”将跟踪上下文放入媒介，来保证跟踪链的连续性。  
以下是我从 Zipkin 获取的 b3-propagation 图。

![](https://mmbiz.qpic.cn/mmbiz_jpg/IgylNib7ZE2J9ehzUvrQkJC5pAJmLkbaDhcCL8Jjk5S0LROluRE9O35oQuXTVJurQdkRILDYCezyZY6RDZXHerQ/640?wx_fmt=jpeg)

但是为什么我们没有在上面的例子中调用这两个函数呢？让我们再来回顾一下代码。在客户端，在创建 gRPC 客户端连接时，我们调用了一个名为 “OpenTracingClientInterceptor” 的函数。以下是 “OpenTracingClientInterceptor” 的部分代码，我从 otgrpc 包中的 “client.go” 中得到了它。它已经从 Go context¹² 获取了跟踪上下文并将其注入 HTTP 头，因此我们不需要再次调用 “inject” 函数。

```
func OpenTracingClientInterceptor(tracer opentracing.Tracer, optFuncs ...Option) 
  grpc.UnaryClientInterceptor {
    ...
    ctx = injectSpanContext(ctx, tracer, clientSpan)
    ...
  }

  func injectSpanContext(ctx context.Context, tracer opentracing.Tracer, clientSpan opentracing.Span) 
    context.Context {
      md, ok := metadata.FromOutgoingContext(ctx)
      if !ok {
        md = metadata.New(nil)
      } else {
        md = md.Copy()
      }
      mdWriter := metadataReaderWriter{md}
      err := tracer.Inject(clientSpan.Context(), opentracing.HTTPHeaders, mdWriter)
      // We have no better place to record an error than the Span itself :-/
      if err != nil {
        clientSpan.LogFields(log.String("event", "Tracer.Inject() failed"), log.Error(err))
      }
      return metadata.NewOutgoingContext(ctx, md)
}
```

在服务器端，我们还调用了一个函数 “otgrpc.OpenTracingServerInterceptor”，其代码类似于客户端的“OpenTracingClientInterceptor”。它不是调用“inject” 写入跟踪上下文，而是从 HTTP 头中提取（extract）跟踪上下文并将其放入 Go 上下文（Go context）中。这就是我们不需要再次手动调用 “extract（）” 的原因。但对于其他基于 HTTP 的服务（如 RESTFul 服务），情况就并非如此，因此我们需要写代码从服务器端的 HTTP 头中提取跟踪上下文。当然，你也可以使用拦截器或过滤器。

跟踪库之间的互兼容性
----------

你也许会问 “如果我的程序使用 OpenTracing 和 Zipkin 而需要调用的第三方微服务使用 OpenTracing 与 Jaeger，它们会兼容吗？"它看起来于我们之前询问的数据库问题类似，但实际上很不相同。对于数据库，因为应用程序和数据库在同一个进程中，它们可以共享相同的全局跟踪器，因此更容易解决。对于微服务，这种方式将不兼容。因为 OpenTracing 只标准化了跟踪接口，它没有标准化跟踪上下文。万维网联盟(W3C) 正在制定跟踪上下文 (trace context) 的标准，并于 2019-08-09 年发布了候选推荐标准。OpenTracing 没有规定跟踪上下文的格式，而是把决定权留给了实现它的跟踪库。结果每个库都选择了自己独有的的格式。例如，Zipkin 使用 “X-B3-TraceId” 作为跟踪 ID，Jaeger 使用 “uber-trace-id”，因此使用 OpenTracing 并不意味着不同的跟踪库可以进行跨网互操作。对于“Jaeger” 来说有一个好处是你可以选择使用 “Zipkin 兼容性功能" 来生成 Zipkin 跟踪上下文， 这样就可以与 Zipkin 相互兼容了。对于其他情况，你需要自己进行手动格式转换(在“inject” 和“extract”之间)。

全链路跟踪设计
-------

### 尽量少写代码

一个好的全链路跟踪系统不需要用户编写很多跟踪代码。最理想的情况是你不需要任何代码，让框架或库负责处理它，当然这比较困难。全链路跟踪分成三个跟踪级别：

*   跨进程跟踪 (cross-process)(调用另一个微服务)
    
*   数据库跟踪
    
*   进程内部的跟踪 (in-process)(在一个函数内部的跟踪)
    

跨进程跟踪是最简单的。你可以编写拦截器或过滤器来跟踪每个请求，它只需要编写极少的代码。数据库跟踪也比较简单。如果使用我们上面讨论过的封装器 (Wrapper)，你只需要注册 SQL 驱动程序封装器(Wrapper) 并将 go-context(里面有跟踪上下文) 传入数据库函数。你可以使用依赖注入(Dependency Injection)，这样就可以用比较少的代码来完成此操作。

进程内跟踪是最困难的，因为你必须为每个单独的函数编写跟踪代码。现在还没有一个很好的方法，可以编写一个通用的函数来跟踪应用程序中的每个函数 (拦截器不是一个好选择，因为它需要每个函数的参数和返回值都必须是一个泛型类型 (interface {}) )。幸运的是，对于大多数人来说，前两个级别的跟踪应该已经足够了。

有些人可能会使用服务网格 (service mesh) 来实现分布式跟踪，例如 Istio 或 Linkerd。它确实是一个好主意，跟踪最好由基础架构（infrastructure）实现，而不是将业务逻辑代码与跟踪代码混在一起，不过你将遇到我们刚才谈到的同样问题。服务网格只负责跨进程跟踪，函数内部或数据库跟踪依然需要你来编写代码。不过一些服务网格可以通过提供与流行跟踪库的集成，来简化不同跟踪库跨网跟踪时的上下文格式转换。

### 跟踪设计:

精心设计的跨度 (span)，服务名称(service name)，标签(tag) 能充分发挥全链路跟踪的作用，并使之简单易用。有关信息  
请阅读语义约定 (Semantic Conventions)¹⁴。

### 将 Trace ID 记录到日志

将跟踪与日志记录集成是一个常见的需求，最重要的是将跟踪 ID 记录到整个调用链的日志消息中。目前 OpenTracing 不提供访问 traceID 的方法。你可以将 “OpenTracing.SpanContext” 转换为特定跟踪库的 “SpanContext”(Zipkin 和 Jaeger 都可以通过各自的“SpanContext” 访问 traceID)或将 “OpenTracing.SpanContext” 转换为字符串并解析它以获取 traceID。转换为字符串更好，因为它不会破坏程序的依赖关系。幸运的是不久的将来你就不需要它了，因为 OpenTracing 将提供访问 traceID 的方法，请阅读这里  github.com/opentracing/specification/blob/master/rfc/trace_identifiers.md。

OpenTracing 和 OpenCensus
------------------------

OpenCensus 不是另一个通用跟踪接口，它是一组库，可以用来与其他跟踪库集成以完成跟踪功能，因此它经常拿来与 OpenTracing 进行比较。那么它与 OpenTracing 兼容吗？答案是否定的。因此，在选择跟踪接口时 (不论是 OpenTracing 还是 OpenCensus) 需要小心，以确保你需要调用的其他库支持它。一个好消息是，你不需要在将来做出选择，因为它们会将项目合并为一个 ¹⁶。

结论:
---

全链路跟踪包括不同的场景，例如在函数内部跟踪，数据库跟踪和跨进程跟踪。每个场景都有不同的问题和解决方案。如果你想设计更好的跟踪解决方案或为你的应用选择最适合的跟踪工具或库，那你需要对每种情况都有清晰的了解。

源码:
---

完整源码的 github 链接 https://github.com/jfeng45/grpcservice

> 转自：倚天码农
> 
> zhuanlan.zhihu.com/p/79419529

- EOF -
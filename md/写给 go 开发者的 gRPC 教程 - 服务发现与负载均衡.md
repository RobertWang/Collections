> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/GqtY2Z21biGqxicv1IJvJQ)

> 相当全面的最新的 gRPC 服务发现与负载均衡介绍，尤其介绍了目前互联网上资料非常少的 gRPC 新的 xDS 方案。

本篇为【写给 go 开发者的 gRPC 教程】系列第九篇

第一篇：[protobuf 基础](https://mp.weixin.qq.com/s?__biz=MzIyMTI4OTY3Mw==&mid=2247483986&idx=1&sn=0142813fe900a594e8a0a0d34e33e025&chksm=e83e4ff4df49c6e2548ce10b6cf440e88b2837e2fd2dcf7ef15517bcb790a86383bd59c42994&scene=21#wechat_redirect)

第二篇：[通信模式](https://mp.weixin.qq.com/s?__biz=MzIyMTI4OTY3Mw==&mid=2247484078&idx=1&sn=91e32c7af22112849263905ec79cf863&chksm=e83e4f08df49c61eef7c67b821bab91ab1eb9b77937f297c3a5b12bd9d2b4993b686c90f52d2&scene=21#wechat_redirect)

第三篇：[拦截器](https://mp.weixin.qq.com/s?__biz=MzIyMTI4OTY3Mw==&mid=2247484098&idx=1&sn=b8263d6885e57bd19ed33dea72e5b6d1&chksm=e83e4f64df49c672b7e443e0a4456be5f31acab4b85f0d724cb61b59f089bd6048be67528080&scene=21#wechat_redirect)

第四篇：[错误处理](https://mp.weixin.qq.com/s?__biz=MzIyMTI4OTY3Mw==&mid=2247484115&idx=1&sn=681de56485e73f4b2ee330901e3cca68&chksm=e83e4f75df49c66361751531b35aed0dbe59b1c173fce790034361dd39d58a6d91d09209d8e4&scene=21#wechat_redirect)

第五篇：[metadata](https://mp.weixin.qq.com/s?__biz=MzIyMTI4OTY3Mw==&mid=2247484298&idx=1&sn=f76997b5d6b875e5ddc8cbe5f83c603f&chksm=e83e4e2cdf49c73abb4cce748c8473c489d3a55ae6a5c8d4995f68884b9eef7b80e58dbc8abb&scene=21#wechat_redirect)

第六篇：[超时控制](https://mp.weixin.qq.com/s?__biz=MzIyMTI4OTY3Mw==&mid=2247484343&idx=1&sn=8ab50ff69290be1bb30a041aa754fcb8&chksm=e83e4e11df49c707a300c413a422bcb3b9de96fffd7403f07787125b4284e6aa9a9ee436e294&scene=21#wechat_redirect)

第七篇：[安全](https://mp.weixin.qq.com/s?__biz=MzIyMTI4OTY3Mw==&mid=2247484466&idx=1&sn=d5bd634937a39a9d8d77b3bcaf8617ba&chksm=e83e4994df49c082b3a2d3d75c5142a0f2f1f3c0827a067aa4c3e1e2ba1b4700169056a23c08&scene=21#wechat_redirect)

第八篇：[用户认证](https://mp.weixin.qq.com/s?__biz=MzIyMTI4OTY3Mw==&mid=2247484475&idx=1&sn=2c562149dc7306847740d35971f9176c&chksm=e83e499ddf49c08bfc85cec9e4b34c414c81b6b6ffa6997b1e8b5a1c2c08ab95f30a4dde4380&scene=21#wechat_redirect)

**第九篇：服务发现与负载均衡** 👈

本系列将持续更新，欢迎关注 👏 获取实时通知

* * *

对于一个客户端创建请求的过程

```
conn, err := grpc.Dial("example:8009", grpc.WithInsecure())
if err != nil {
  panic(err)
}


```

1.  gRPC 客户端通过服务发现解析请求，将名称解析为一个或多个 IP 地址，以及服务配置
    
2.  客户端使用上一步的服务配置、ip 列表、实例化负载均衡策略
    
3.  负载均衡策略为每个服务器地址创建一个子通道（channel），并监测每一个子通道状态
    
4.  当有 rpc 请求时，负载均衡策略决定那个子通道即 gRPC 服务器将接收请求，当可用服务器为空时客户端的请求将被阻塞
    

gRPC 官方提供了基本的服务发现和负载均衡逻辑，并提供了接口供扩展用于开发自定义的服务发现与负载均衡

服务发现
----

用通俗易懂的方式来解释下什么是服务发现。通常情况下客户端需要知道服务端的 IP + 端口号才能建立连接，但服务端的 IP 和端口号并不是那么容易记忆。还有更重要的，在云部署的环境中，服务端的 IP 和端口可能随时会发生变化。

所以我们可以给某一个服务起一个名字，客户端通过名字创建与服务端的连接，客户端底层使用**服务发现**系统，解析这个名字来获取真正的 IP 和端口，并在服务端的 IP 和端口发生变化时，重新建立连接。这样的系统通常也会被叫做`name-system`（名字服务）

gRPC 中的默认`name-system`是 DNS，同时在客户端以插件形式提供了自定义`name-system`的机制。

### 名字格式

gRPC 采用的名字格式遵循的`RFC 3986`中定义的 URI 语法

```
scheme:[//[user[:password]@]host[:port]][/path][?query][#fragment]


```

例如

![](https://mmbiz.qpic.cn/mmbiz_png/kWF4acROZUAjC0ETh519H6LwQiboQ8iaaUrcjpykKMzQuCknFjffDoKhNj3dWEyhe6dWdWl9oMTWoFdv6ib5rEoWw/640?wx_fmt=png)URI 示例

gRPC 关注其中两部分

*   **URI scheme**：第一个`:`前面的标识。对于 gRPC 客户端来说，它指示要使用的服务发现解析器，如果未指定前缀或方案未知，则默认使用 DNS 方案
    
*   **URI path**：指示要解析的名字
    

大部分 gRPC 实现默认支持以下的 _URI schemes_

*   `dns:[//authority/]host[:port]` -- DNS (default)
    
*   `unix:path`, `unix://absolute_path` -- Unix domain sockets (Unix systems only)
    
*   `unix-abstract:abstract_path` -- Unix domain socket in abstract namespace (Unix systems only)
    

### 服务的服务注册

如果 gRPC 服务端的地址是静态的，可以在客户端服务发现时直接解析为静态的地址

如果 gRPC 服务端的地址是动态的，可以有两种选择

*   自注册：当 gRPC 的服务启动后，向一个集中的注册中心进行注册
    
*   平台的服务发现：使用 k8s 平台时，平台会感知 gPRC 实例的变化
    

关于服务注册这里不在做更多介绍了

### 客户端的服务发现

自定义服务发现需要在客户端启动前，注册一个服务解析器（`Resolve`）

Golang 中使用`google.golang.org/grpc/resolver.Register(resolver.Builder)`注册，这个函数不是直接接收一个解析器，而是使用工厂模式接收一个解析器的构造器

```
type Builder interface {
 // Build creates a new resolver for the given target.
 //
 // gRPC dial calls Build synchronously, and fails if the returned error is
 // not nil.
 Build(target Target, cc ClientConn, opts BuildOptions) (Resolver, error)
 // Scheme returns the scheme supported by this resolver.
 // Scheme is defined at https://github.com/grpc/grpc/blob/master/doc/naming.md.
 Scheme() string
}


```

✨ `Scheme()`需要返回的就是名字格式中提到的 URI scheme

✨ `Build(...)`需要返回一个服务发现解析器`google.golang.org/grpc/resolver.Resolver`

✨✨`cc ClientConn`代表客户端与服务端的连接，其拥有的`cc.UpdateState(State) error`可以让我们更新链接的状态

```
type Resolver interface {
 // ResolveNow will be called by gRPC to try to resolve the target name
 // again. It's just a hint, resolver can ignore this if it's not necessary.
 //
 // It could be called multiple times concurrently.
  // ResolveNow 尝试再一次对域名进行解析，在个人实践中，服务端进程挂掉会触发该调用
 ResolveNow(ResolveNowOptions)
  // Close closes the resolver.
  // 资源释放。
 Close()
}


```

解析器需要有能力从注册中心获取解析结果，并更新客户端中连接 (`cc ClientConn`) 的信息。还可以持续 watch 一个名字的解析结果，实时的更新客户端中连接的信息

*   解析出来的地址列表（包含 ip 和 port）
    
*   针对连接的服务配置，如负载均衡等，来想覆盖全局的负责均衡配置
    
*   每一个地址可以包含一系列的属性（kv），他们可以用来支持后续的负载均衡策略
    

### 示例代码

#### gRPC resolver 原理

🌲 在 `init()` 阶段时

*   创建并注册 Builder 实例，将其注册到 grpc 内部的 resolveBuilder 表中（其实是一个全局 map，key 为协议名；value 为构造的 resolveBuilder）
    

🌲 客户端启动时通过自定义`Dail()`方法构造`grpc.ClientConn`单例

*   `grpc.DialContext()` 方法内部解析 URI，分析协议类型，并从 resolveBuilder 表中查找协议对应的 resolverBuilder
    
*   将地址作为 `Target` 、conn 单例作为`resolver.ClientConn`参数调用 `resolver.Build` 方法实例化出 `Resolver`
    
*   用户 `Resolver`实现中调用 `cc.UpdateState` 传入 `State.Addresses` 地址，gRPC 使用这个地址建立连接，传入`State.ServiceConfig`，gRPC 使用这份服务配置覆盖默认配置
    

![](https://mmbiz.qpic.cn/mmbiz_png/kWF4acROZUAjC0ETh519H6LwQiboQ8iaaURaEyOyhpc1MEiaJqYgqlyOPDLVj2B4AInnxNyk0NqrD5Y8MAibX96TbQ/640?wx_fmt=png) name-reslover 原理

#### 代码

```
# builder.go
package resolver

import "google.golang.org/grpc/resolver"

var _ resolver.Builder = Builder{}

type Builder struct {
 addrsStore map[string][]string
}

func NewResolverBuilder(addrsStore map[string][]string) *Builder {
 return &Builder{addrsStore: addrsStore}
}

func (b Builder) Build(target resolver.Target, cc resolver.ClientConn, opts resolver.BuildOptions) (resolver.Resolver, error) {
 r := &Resolver{
  target:     target,
  cc:         cc,
  addrsStore: b.addrsStore,
 }
 r.Start()
 return r, nil
}

func (b Builder) Scheme() string {
 return "example"
}


```

```
# resolver.go
package resolver

import (
 "google.golang.org/grpc/resolver"
)

var _ resolver.Resolver = &Resolver{}

// impl google.golang.org/grpc/resolver.Resolver
type Resolver struct {
 target resolver.Target
 cc     resolver.ClientConn

 addrsStore map[string][]string
}

func (r *Resolver) Start() {
 // 在静态路由表中查询此 Endpoint 对应 addrs
 var addrs []resolver.Address
 for _, addr := range r.addrsStore[r.target.URL.Opaque] {
  addrs = append(addrs, resolver.Address{Addr: addr})
 }

 r.cc.UpdateState(resolver.State{
  Addresses: addrs,
    // 设置负载均衡策略为round_robin
  ServiceConfig: r.cc.ParseServiceConfig(
    `{"loadBalancingPolicy":"round_robin"}`),
 })
}

func (r *Resolver) ResolveNow(resolver.ResolveNowOptions) {

}

func (r *Resolver) Close() {

}


```

```
# main.go
package main

import (
 "context"
 "log"

 rs "github.com/liangwt/note/grpc/name_resolver_lb_example/client/resolver"
 pb "github.com/liangwt/note/grpc/name_resolver_lb_example/ecommerce"
 "google.golang.org/grpc"
 "google.golang.org/grpc/resolver"
)

func main() {
 resolver.Register(rs.NewResolverBuilder(map[string][]string{
  "cluster@callee": {
   "127.0.0.1:8009",
  },
 }))

 conn, err := grpc.Dial("example:cluster@callee", grpc.WithInsecure())
 if err != nil {
  panic(err)
 }
  
 // ...
}


```

负载均衡
----

同样来通俗易懂的解释下什么负载均衡。为了提高系统的负载能力和稳定性，我们的服务端往往具有多台服务器，负载均衡的目的就是希望请求能分散到不同的服务器，从服务器列表中选择一台服务器的算法就是负载均衡的策略，常见的轮循、加权轮询等

负载均衡器要在多台服务器之间选择，**所以通常情况下负载均衡器是具备服务发现的能力的**

根据负载均衡实现所在的位置不同，通常可分为以下三种解决方案：

**1、集中式负载均衡（Proxy Model）**

在客户端和服务端之间有一个独立的 LB，通常是专门的硬件设备如 F5，或者基于软件如 LVS，HAproxy，Nginx 等实现。LB 使用负载均衡策略将请求转发到目标服务

因为所有服务调用流量都经过 LB，当服务数量和调用量大的时候，LB 容易成为瓶颈；一旦 LB 发生故障影响整个系统；客户端、服务端之间增加了一级，有一定性能开销

![](https://mmbiz.qpic.cn/mmbiz_png/kWF4acROZUAjC0ETh519H6LwQiboQ8iaaU02euAh59yO0lwb9TXrDKtoM5icAibF9E5iabArjVWJ73WR1I8cPkfnkjA/640?wx_fmt=png)集中式负载均衡

**2、客户端负载均衡（Balancing-aware Client）**

客户端负载将 LB 的功能集成到客户端进程里，然后使用负载均衡策略选择一个目标服务地址，向目标服务发起请求。LB 能力被分散到每一个服务消费者的进程内部，同时服务消费方和服务提供方之间是直接调用，没有额外开销，性能比较好。

但如果有多种不同的语言栈，就要配合开发多种不同的客户端，有一定的研发和维护成本；后续如果要对客户库进行升级，势必要求服务调用方修改代码并重新发布，升级较复杂。

![](https://mmbiz.qpic.cn/mmbiz_png/kWF4acROZUAjC0ETh519H6LwQiboQ8iaaUGfgQs8lQ0UfbcCp7M9OGY49ibBgaDndCMu0biapKYDWx41nD7ibHJ1F3w/640?wx_fmt=png)客户端负载均衡

**3、独立负载均衡进程（External Load Balancing Service）**

将 LB 从进程内移出来，变成主机上的一个独立进程。主机上的一个或者多个服务要访问目标服务时，他们都通过同一主机上的独立 LB 进程做负载均衡

此方案有两种模式

第一种是直接由 LB 进行转发请求，被称为 sidecar 方案

第二种是从 LB 获取到 IP 后依旧由客户端发起请求，gRPC 曾经支持过此方案叫 lookaside 方案，目前已废弃

该方案也是一种分布式方案没有单点问题，一个 LB 进程挂了只影响该主机上的客户端；客户端和 LB 之间是本地调用调用性能好；同时该方案还简化了客户端，不需要为不同语言开发客户库，LB 的升级不需要服务调用方改代码。该方案主要问题：部署较复杂，环节多，出错调试排查问题不方便

![](https://mmbiz.qpic.cn/mmbiz_png/kWF4acROZUAjC0ETh519H6LwQiboQ8iaaU5KOarU0PomJhlVfvNe2LWWh8AibKMYCpiaibOZXPK7j5EPJxibaszLYr2Q/640?wx_fmt=png)独立负载均衡进程

### gRPC 的负载均衡

上文介绍的三种负载均衡方式，集中式负载均衡和 gRPC 无关，属于外部的基础设施，因此我们不再介绍

gRPC 中的负载平衡是以每次调用为基础，而不是以每个连接为基础。换句话说，即使所有的请求都来自一个客户端，它仍能在所有的服务器上实现负载平衡

gRPC 目前内置四种策略

🌲 `pick_first`：默认策略，选择第一个

🌲 `round_robin`：轮询

使用默认的负载均衡器很简单，只需要在建立连接的时候指定负载均衡策略即可。

> ⚠️ 注意
> 
> 旧版本 gRPC 使用 `grpc.WithBalancerName("round_robin"),`已经被废弃，使用`grpc.WithDefaultServiceConfig`。
> 
> `grpc.WithDefaultServiceConfig`可以被上文服务发现中提到的`cc.UpdateState(State) error`覆盖配置

```
 conn, err := grpc.Dial("example:cluster@callee",
  grpc.WithInsecure(),
  grpc.WithDefaultServiceConfig(
   `{"loadBalancingPolicy":"round_robin"}`,
  ),
 )


```

🌲 `grpclb`：已废弃

它属于上文介绍的负载均衡中独立负载均衡进程第二种。**不必直接在客户端中添加新的 LB 策略**，而只实现诸如 round-robin 之类的简单算法，任何更复杂的算法都将由 lookaside 负载平衡器提供

![](https://mmbiz.qpic.cn/mmbiz_png/kWF4acROZUAjC0ETh519H6LwQiboQ8iaaUUGvENMvicJVia0EB8tgCep3iceUKLCEkT4LAvYV0tI7WkYehxUAfsgFMQ/640?wx_fmt=png)gRPC 的 grpclb 方案

🌲 `xDS`

如果接触过 servicemesh 那么对 xDS 并不会陌生，xDS 本身是 Envoy 中的概念，现在已经发展为用于配置各种数据平面软件的标准，最新版本的 gRPC 已经支持 xDS。不同于单纯的负载均衡策略，xDS 在 gRPC 包含了服务发现和负载均衡的概念

这里简单的理解下 xDS，本质上 xDS 就是一个标准的协议

它规定了 xDS 客户端和 xDS 服务端的交互流程，即 API 调用的顺序

我们在 xDS 服务端实现服务发现，配置负载均衡策略等等，支持 xDS 的客户端连接到 xDS 服务端并通过 xDS api 来获取各种需要的数据和配置

xDS 主要应用于 servicemesh 中，在 mesh 中由 sidecar 连接到 xDS server 进行数据交互，同时由 sidecar 来控制流量的分发。也就是上文提到的独立负载均衡进程的第一种模式

![](https://mmbiz.qpic.cn/mmbiz_png/kWF4acROZUAjC0ETh519H6LwQiboQ8iaaU5GZvqZLnDQoVliaNuWY2m8wO3xeJiayYTaajVVbID0jbM6Bgg0HBB9dg/640?wx_fmt=png)servicemesh 的负载均衡

gRPC 使用 xDS 是一种无 proxy 的客户端负载均衡方案。对比 Mesh 方案，性能更好。我们把 servicemesh 的负载均衡和 grpc 的 xds 负载均衡放在一起感受下区别

![](https://mmbiz.qpic.cn/mmbiz_png/kWF4acROZUAjC0ETh519H6LwQiboQ8iaaUlbJdEX7z6bNxr2YODzzscLK5moGEC7SayF8J0L06sdtjLyy4G50CKA/640?wx_fmt=png)gRPC 的 xDS 的方案

📖 这意味着，`grpclb`废弃后，gRPC 内置的`pick_first`、`round_robin`、`xDS`三种模式都属于客户端的负载均衡模式。`pick_first`、`round_robin`是单纯的负载均衡策略；`xDS`包含了服务发现等一系列能力，并且还在不断拓展中，而我们控制 gRPC 的行为也被转移到了 xDS server 上了。

### xDS 模式的使用

xDS 内容较多，又比较新，单独开个章节介绍下 xDS 的使用

gRPC xDS 架构中出现了三个服务

*   **xDS server**：它是我们的控制面
    
*   **gRPC client**：它既是 gRPC 服务的客户端，同时也是 xDS 的客户端
    
*   **gRPC server**：它是 gRPC 服务的服务端，它需要具备被 xDS server 发现的能力
    

我们可以使用 Envoy go-control-plane 库来实现 xDS server，这部分的开发类似于 servicemesh，就不介绍太多了，如果位于 k8s 平台内可以参考下 istio 中控制面的实现

gRPC server 需要自注册或者托管到 k8s 平台，如果托管到 k8s 则可以继续参考 istio 中控制面的实现

因为 xDS 也包含了服务发现的部分，因此对于 client 来说第一步需要先开发自定义的服务发现和负载均衡配置。幸运的是 gRPC 官方已经为我们开发了对应实现，只需要引入包即可，在 init 阶段会注册 xDS 的解析器和负载均衡器

```
_ "google.golang.org/grpc/xds" // To install the xds resolvers and balancers.


```

随后只需要把 gRPC client 连接到 xDs server 即可，这部分与非 xDS 并无不同。只是目标服务的地址的 URI scheme 为`xds`

```
conn, err := grpc.Dial("xds:///localhost:50051", grpc.WithTransportCredentials(creds))
if err != nil {
  panic(err)
}

ctx, cancelFn := context.WithCancel(context.Background())
defer cancelFn()

c := pb.NewOrderManagementClient(conn)
res, err := c.AddOrder(ctx, &order)


```

完整代码可以参考：https://github.com/grpc/grpc-go/tree/master/examples/features/xds

### 自定义负载均衡器

自定义负载均衡器需要使用`google.golang.org/grpc/balancer.Register`提前注册，此函数和服务发现一样接受工厂函数

```
// Builder creates a balancer.
type Builder interface {
 // Build creates a new balancer with the ClientConn.
 Build(cc ClientConn, opts BuildOptions) Balancer
 // Name returns the name of balancers built by this builder.
 // It will be used to pick balancers (for example in service config).
 Name() string
}


```

✨ `Name()`是负载均衡策略的名字

✨ `Build(...)`需要返回负载均衡器

✨✨ `cc ClientConn`代表客户端与服务端的连接，其拥有一系列函数可以让我们更新链接的状态

```
type Balancer interface {
 // UpdateClientConnState is called by gRPC when the state of the ClientConn
 // changes.  If the error returned is ErrBadResolverState, the ClientConn
 // will begin calling ResolveNow on the active name resolver with
 // exponential backoff until a subsequent call to UpdateClientConnState
 // returns a nil error.  Any other errors are currently ignored.
 UpdateClientConnState(ClientConnState) error
 // ResolverError is called by gRPC when the name resolver reports an error.
 ResolverError(error)
 // UpdateSubConnState is called by gRPC when the state of a SubConn
 // changes.
 UpdateSubConnState(SubConn, SubConnState)
 // Close closes the balancer. The balancer is not required to call
 // ClientConn.RemoveSubConn for its existing SubConns.
 Close()
}


```

负载均衡器需要实现一系列的函数用于 gRPC 在不同场景下调用

**类 RR 算法负载均衡器**

如果要实现一个类 round_robin 的负载均衡策略，gRPC 官方实现提供了一个`baseBuilder`，它已经实现了大部`Balancer`接口，可以大幅简化了我们创建 RR 策略的逻辑。使用`google.golang.org/grpc/balancer/base.NewBalancerBuilder`创建负载均衡的工厂

```
func NewBalancerBuilder(name string, pb PickerBuilder, config Config) balancer.Builder 


```

*   `name string`负载均衡器的名字
    
*   `pb PickerBuilder`是我们要实现的负载均衡策略逻辑的地方
    
*   `config Config`可以配置是否要进行健康检查
    

```
// PickerBuilder creates balancer.Picker.
type PickerBuilder interface {
 // Build returns a picker that will be used by gRPC to pick a SubConn.
 Build(info PickerBuildInfo) balancer.Picker
}


```

```
type Picker interface {
  // 子连接选择
 Pick(info PickInfo) (PickResult, error)
}


```

于是借助`base.NewBalancerBuilder`我们仅需要实现`Picker`一个函数即可实现类 RR 的负载均衡策略了

#### 代码

利用`Picker`接口来实现一个随机选择策略

```
# builder.go
package balancer

import (
 "google.golang.org/grpc/balancer"
 "google.golang.org/grpc/balancer/base"
)

var _ base.PickerBuilder = &Builder{}

type Builder struct {
}

func NewBalancerBuilder() balancer.Builder {
 return base.NewBalancerBuilder("random_picker", &Builder{}, base.Config{HealthCheck: true})
}

func (b *Builder) Build(info base.PickerBuildInfo) balancer.Picker {
 if len(info.ReadySCs) == 0 {
  return base.NewErrPicker(balancer.ErrNoSubConnAvailable)
 }

 var scs []balancer.SubConn
 for subConn := range info.ReadySCs {
  scs = append(scs, subConn)
 }

 return &Picker{
  subConns: scs,
 }
}


```

```
// picker.go
package balancer

import (
 "math/rand"

 "google.golang.org/grpc/balancer"
)

var _ balancer.Picker = &Picker{}

type Picker struct {
 subConns []balancer.SubConn
}

func (p *Picker) Pick(info balancer.PickInfo) (balancer.PickResult, error) {
 index := rand.Intn(len(p.subConns))
 sc := p.subConns[index]
 return balancer.PickResult{SubConn: sc}, nil
}


```

```
#client.go
package main

import (
 "context"
 "log"

 bl "github.com/liangwt/note/grpc/name_resolver_lb_example/client/balancer"
 rs "github.com/liangwt/note/grpc/name_resolver_lb_example/client/resolver"
 pb "github.com/liangwt/note/grpc/name_resolver_lb_example/ecommerce"
 "google.golang.org/grpc"
 "google.golang.org/grpc/balancer"
 "google.golang.org/grpc/resolver"
)

func main() {
 resolver.Register(rs.NewResolverBuilder(map[string][]string{
  "cluster@callee": {
   "127.0.0.1:8009",
   "127.0.0.1:8010",
  },
 }))

 balancer.Register(bl.NewBalancerBuilder())

 conn, err := grpc.Dial("example:cluster@callee",
  grpc.WithInsecure(),
  grpc.WithDefaultServiceConfig(
   `{"loadBalancingPolicy":"random_picker"}`,
  ),
 )
 if err != nil {
  panic(err)
 }
 
  ....
}


```

* * *

参考资料  

[1]

URI 语法: _https://zh.wikipedia.org/wiki/%E7%BB%9F%E4%B8%80%E8%B5%84%E6%BA%90%E6%A0%87%E5%BF%97%E7%AC%A6_

[2]

go-control-plane: _https://github.com/envoyproxy/go-control-plane_

[3]

gRPC Name Resolution: _https://github.com/grpc/grpc/blob/master/doc/naming.md_

[4]

Load Balancing in gRPC: _https://github.com/grpc/grpc/blob/master/doc/load-balancing.md_

[5]

https://github.com/grpc/grpc-go: _https://github.com/grpc/grpc-go_

[6]

gRPC Go 服务发现与负载均衡: _https://blog.cong.moe/post/2021-03-06-grpc-go-discovery-lb/_

[7]

[转]gRPC 服务发现 & 负载均衡: _https://colobu.com/2017/03/25/grpc-naming-and-load-balance/_

[8]

浅淡 xDS 协议在 gRPC 中的应用: _http://limeng.org/2020/03/08/xds-in-grpc.html_
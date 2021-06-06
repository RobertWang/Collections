> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxNjk4ODE4OQ==&mid=2247500376&idx=2&sn=6d50495a55ed2541e0c3d8813afa9dda&chksm=9beee32aac996a3c62d1960604b7e90f67bd8fe1048a6fae1a6776aa5ec560f41a713890cb5e&scene=21#wechat_redirect)

随着微服务架构的流行，服务按照不同的维度进行拆分，一次请求往往需要涉及到多个服务。  

互联网应用构建在不同的软件模块集上，这些软件模块，有可能是由不同的团队开发、可能使用不同的编程语言来实现、有可能布在了几千台服务器，横跨多个不同的数据中心。

因此，就需要一些可以帮助理解系统行为、用于分析性能问题的工具，以便发生故障的时候，能够快速定位和解决问题。

全链路监控组件就在这样的问题背景下产生了。最出名的是谷歌公开的论文提到的 Google Dapper。

想要在这个上下文中理解分布式系统的行为，就需要监控那些横跨了不同的应用、不同的服务器之间的关联动作。

所以，在复杂的微服务架构系统中，几乎每一个前端请求都会形成一个复杂的分布式服务调用链路。一个请求完整调用链可能如下图所示：

![](https://mmbiz.qpic.cn/mmbiz/yNKv1P4Q9eXAD3BGZWPJxT7DFIvSxRUeJKwenxBHrOyb5PBmez71ib0VlMsNYVFib2u3XQCjuA7m3ngY40XbCfqA/640?wx_fmt=jpeg)

那么在业务规模不断增大、服务不断增多以及频繁变更的情况下，面对复杂的调用链路就带来一系列问题：

1.  如何快速发现问题？
    
2.  如何判断故障影响范围？
    
3.  如何梳理服务依赖以及依赖的合理性？
    
4.  如何分析链路性能问题以及实时容量规划？
    

同时我们会关注在请求处理期间各个调用的各项性能指标，比如：吞吐量（TPS）、响应时间及错误记录等。

1.  吞吐量，根据拓扑可计算相应组件、平台、物理设备的实时吞吐量。
    
2.  响应时间，包括整体调用的响应时间和各个服务的响应时间等。
    
3.  错误记录，根据服务返回统计单位时间异常次数。
    

全链路性能监控从整体维度到局部维度展示各项指标，将跨应用的所有调用链性能信息集中展现，可方便度量整体和局部性能，并且方便找到故障产生的源头，生产上可极大缩短故障排除时间。

有了全链路监控工具，我们能够达到：

1.  请求链路追踪，故障快速定位：可以通过调用链结合业务日志快速定位错误信息。
    
2.  可视化：各个阶段耗时，进行性能分析。
    
3.  依赖优化：各个调用环节的可用性、梳理服务依赖关系以及优化。
    
4.  数据分析，优化链路：可以得到用户的行为路径，汇总分析应用在很多业务场景。
    

1 目标要求
------

如上所述，那么我们选择全链路监控组件有哪些目标要求呢？Google Dapper 中也提到了，总结如下：

### 1、**探针的性能消耗**

APM 组件服务的影响应该做到足够小。服务调用埋点本身会带来性能损耗，这就需要调用跟踪的低损耗，实际中还会通过配置采样率的方式，选择一部分请求去分析请求路径。在一些高度优化过的服务，即使一点点损耗也会很容易察觉到，而且有可能迫使在线服务的部署团队不得不将跟踪系统关停。

### **2、代码的侵入性**

即也作为业务组件，应当尽可能少入侵或者无入侵其他业务系统，对于使用方透明，减少开发人员的负担。

对于应用的程序员来说，是不需要知道有跟踪系统这回事的。如果一个跟踪系统想生效，就必须需要依赖应用的开发者主动配合，那么这个跟踪系统也太脆弱了，往往由于跟踪系统在应用中植入代码的 bug 或疏忽导致应用出问题，这样才是无法满足对跟踪系统 “无所不在的部署” 这个需求。

### **3、可扩展性**

一个优秀的调用跟踪系统必须支持分布式部署，具备良好的可扩展性。能够支持的组件越多当然越好。或者提供便捷的插件开发 API，对于一些没有监控到的组件，应用开发者也可以自行扩展。

### **4、数据的分析**

数据的分析要快 ，分析的维度尽可能多。跟踪系统能提供足够快的信息反馈，就可以对生产环境下的异常状况做出快速反应。分析的全面，能够避免二次开发。

2 功能模块
------

一般的全链路监控系统，大致可分为四大功能模块：

### 1. 埋点与生成日志

埋点即系统在当前节点的上下文信息，可以分为 客户端埋点、服务端埋点，以及客户端和服务端双向型埋点。埋点日志通常要包含以下内容 traceId、spanId、调用的开始时间，协议类型、调用方 ip 和端口，请求的服务名、调用耗时，调用结果，异常信息等，同时预留可扩展字段，为下一步扩展做准备；

> 不能造成性能负担：一个价值未被验证，却会影响性能的东西，是很难在公司推广的！
> 
> 因为要写 log，业务 QPS 越高，性能影响越重。通过采样和异步 log 解决。

### 2. 收集和存储日志

主要支持分布式日志采集的方案，同时增加 MQ 作为缓冲；

1.  每个机器上有一个 deamon 做日志收集，业务进程把自己的 Trace 发到 daemon，daemon 把收集 Trace 往上一级发送；
    
2.  多级的 collector，类似 pub/sub 架构，可以负载均衡；  
    
3.  对聚合的数据进行 实时分析和离线存储；  
    
4.  离线分析 需要将同一条调用链的日志汇总在一起；
    

### 3. 分析和统计调用链路数据，以及时效性

调用链跟踪分析：把同一 TraceID 的 Span 收集起来，按时间排序就是 timeline。把 ParentID 串起来就是调用栈。

抛异常或者超时，在日志里打印 TraceID。利用 TraceID 查询调用链情况，定位问题。

**依赖度量：**

1.  强依赖：调用失败会直接中断主流程
    
2.  高度依赖：一次链路中调用某个依赖的几率高
    
3.  频繁依赖：一次链路调用同一个依赖的次数多
    

**离线分析：**按 TraceID 汇总，通过 Span 的 ID 和 ParentID 还原调用关系，分析链路形态。

**实时分析：**对单条日志直接分析，不做汇总，重组。得到当前 QPS，延迟。

### 4. 展现以及决策支持

3 Google Dapper
---------------

### 3.1 Span

基本工作单元，一次链路调用（可以是 RPC，DB 等没有特定的限制）创建一个 span，通过一个 64 位 ID 标识它，uuid 较为方便，span 中还有其他的数据，例如描述信息，时间戳，key-value 对的（Annotation）tag 信息，parent_id 等，其中 parent-id 可以表示 span 调用链路来源。

![](https://mmbiz.qpic.cn/mmbiz/yNKv1P4Q9eXAD3BGZWPJxT7DFIvSxRUeHGjALlxd0x7fvBkZtw9PUY0FAUk3DZdtbl9DNQF6LYZnKex5s66xMA/640?wx_fmt=jpeg)

上图说明了 span 在一次大的跟踪过程中是什么样的。Dapper 记录了 span 名称，以及每个 span 的 ID 和父 ID，以重建在一次追踪过程中不同 span 之间的关系。如果一个 span 没有父 ID 被称为 root span。所有 span 都挂在一个特定的跟踪上，也共用一个跟踪 id。

### Span 数据结构：

### 3.2 Trace

类似于 树结构的 Span 集合，表示一次完整的跟踪，从请求到服务器开始，服务器返回 response 结束，跟踪每次 rpc 调用的耗时，存在唯一标识 trace_id。比如：你运行的分布式大数据存储一次 Trace 就由你的一次请求组成。

![](https://mmbiz.qpic.cn/mmbiz/yNKv1P4Q9eXAD3BGZWPJxT7DFIvSxRUeC2wOFbLsIaQ07ib2hA2QWMuiaeqPR6Xmaia9UZEVBPy4ATficKoCVibOcicw/640?wx_fmt=jpeg)

Trace

每种颜色的 note 标注了一个 span，一条链路通过 TraceId 唯一标识，Span 标识发起的请求信息。树节点是整个架构的基本单元，而每一个节点又是对 span 的引用。节点之间的连线表示的 span 和它的父 span 直接的关系。虽然 span 在日志文件中只是简单的代表 span 的开始和结束时间，他们在整个树形结构中却是相对独立的。

### 3.3 Annotation

注解，用来记录请求特定事件相关信息（例如时间），一个 span 中会有多个 annotation 注解描述。通常包含四个注解信息：

(1) cs：Client Start，表示客户端发起请求  
(2) sr：Server Receive，表示服务端收到请求  
(3) ss：Server Send，表示服务端完成处理，并将结果发送给客户端  
(4) cr：Client Received，表示客户端获取到服务端返回信息

Annotation 数据结构：

### 3.4 调用示例

#### 1. 请求调用示例

1.  当用户发起一个请求时，首先到达前端 A 服务，然后分别对 B 服务和 C 服务进行 RPC 调用；
    
2.  B 服务处理完给 A 做出响应，但是 C 服务还需要和后端的 D 服务和 E 服务交互之后再返还给 A 服务，最后由 A 服务来响应用户的请求；
    

  

![](https://mmbiz.qpic.cn/mmbiz/yNKv1P4Q9eXAD3BGZWPJxT7DFIvSxRUeGtx1LTVC82j1WcD5m8hOpUrD1I2OCHP6Nibf6KbgTecDfq14jCqtneA/640?wx_fmt=jpeg)

请求调用示例

#### 2. 调用过程追踪

1.  请求到来生成一个全局 TraceID，通过 TraceID 可以串联起整个调用链，一个 TraceID 代表一次请求。
    
2.  除了 TraceID 外，还需要 SpanID 用于记录调用父子关系。每个服务会记录下 parent id 和 span id，通过他们可以组织一次完整调用链的父子关系。
    
3.  一个没有 parent id 的 span 成为 root span，可以看成调用链入口。
    
4.  所有这些 ID 可用全局唯一的 64 位整数表示；  
    
5.  整个调用过程中每个请求都要透传 TraceID 和 SpanID。  
    
6.  每个服务将该次请求附带的 TraceID 和附带的 SpanID 作为 parent id 记录下，并且将自己生成的 SpanID 也记录下。  
    
7.  要查看某次完整的调用则 只要根据 TraceID 查出所有调用记录，然后通过 parent id 和 span id 组织起整个调用父子关系。
    

![](https://mmbiz.qpic.cn/mmbiz/yNKv1P4Q9eXAD3BGZWPJxT7DFIvSxRUeic1oC8scnsQuCCSv8A4nMKEVYroYpOMrDABRrRsBQVzHDa3ZD2Bkglw/640?wx_fmt=jpeg)

整个调用过程追踪

#### 3. 调用链核心工作

1.  调用链数据生成，对整个调用过程的所有应用进行埋点并输出日志。
    
2.  调用链数据采集，对各个应用中的日志数据进行采集。
    
3.  调用链数据存储及查询，对采集到的数据进行存储，由于日志数据量一般都很大，不仅要能对其存储，还需要能提供快速查询。
    
4.  指标运算、存储及查询，对采集到的日志数据进行各种指标运算，将运算结果保存起来。
    
5.  告警功能，提供各种阀值警告功能。
    

#### 4. 整体部署架构

![](https://mmbiz.qpic.cn/mmbiz/yNKv1P4Q9eXAD3BGZWPJxT7DFIvSxRUegWBMMuDNzibiarEZOJqdYoK55VAHQ666J5Iy1yicyPqvibO5JUL0BuDVGg/640?wx_fmt=jpeg)

#### 5. AGENT 无侵入部署

通过 AGENT 代理无侵入式部署，将性能测量与业务逻辑完全分离，可以测量任意类的任意方法的执行时间，这种方式大大提高了采集效率，并且减少运维成本。根据服务跨度主要分为两大类 AGENT：

1.  服务内 AGENT，这种方式是通过 Java 的 agent 机制，对服务内部的方法调用层次信息进行数据收集，如方法调用耗时、入参、出参等信息。
    
2.  跨服务 AGENT，这种情况需要对主流 RPC 框架以插件形式提供无缝支持。并通过提供标准数据规范以适应自定义 RPC 框架：
    

#### 6. 调用链监控好处

1.  准确掌握生产一线应用部署情况；
    
2.  从调用链全流程性能角度，识别对关键调用链，并进行优化；
    
3.  提供可追溯的性能数据，量化 IT 运维部门业务价值；
    
4.  快速定位代码性能问题，协助开发人员持续性的优化代码；
    
5.  协助开发人员进行白盒测试，缩短系统上线稳定期；
    

4 方案比较
------

市面上的全链路监控理论模型大多都是借鉴 Google Dapper 论文，本文重点关注以下三种 APM 组件：

1.  Zipkin：由 Twitter 公司开源，开放源代码分布式的跟踪系统，用于收集服务的定时数据，以解决微服务架构中的延迟问题，包括：数据的收集、存储、查找和展现。
    
2.  Pinpoint：一款对 Java 编写的大规模分布式系统的 APM 工具，由韩国人开源的分布式跟踪组件。
    
3.  Skywalking：国产的优秀 APM 组件，是一个对 JAVA 分布式应用程序集群的业务运行情况进行追踪、告警和分析的系统。
    

以上三种全链路监控方案需要对比的项提炼出来：

1. 探针的性能  
主要是 agent 对服务的吞吐量、CPU 和内存的影响。微服务的规模和动态性使得数据收集的成本大幅度提高。

2.collector 的可扩展性  
能够水平扩展以便支持大规模服务器集群。

3. 全面的调用链路数据分析  
提供代码级别的可见性以便轻松定位失败点和瓶颈。

4. 对于开发透明，容易开关  
添加新功能而无需修改代码，容易启用或者禁用。

5. 完整的调用链应用拓扑  
自动检测应用拓扑，帮助你搞清楚应用的架构

### 4.1 探针的性能

比较关注探针的性能，毕竟 APM 定位还是工具，如果启用了链路监控组建后，直接导致吞吐量降低过半，那也是不能接受的。对 skywalking、zipkin、pinpoint 进行了压测，并与基线（未使用探针）的情况进行了对比。

选用了一个常见的基于 Spring 的应用程序，包含 Spring Boot, Spring MVC，redis 客户端，mysql。监控这个应用程序，每个 trace，探针会抓取 5 个 span（1 Tomcat, 1 SpringMVC, 2 Jedis, 1 Mysql）。这边基本和 skywalkingtest 的测试应用差不多。

模拟了三种并发用户：500，750，1000。使用 jmeter 测试，每个线程发送 30 个请求，设置思考时间为 10ms。使用的采样率为 1，即 100%，这边与生产可能有差别。pinpoint 默认的采样率为 20，即 50%，通过设置 agent 的配置文件改为 100%。zipkin 默认也是 1。组合起来，一共有 12 种。下面看下汇总表：

![](https://mmbiz.qpic.cn/mmbiz/yNKv1P4Q9eXAD3BGZWPJxT7DFIvSxRUeU35sjickonfpKl1VdA1cN1cAGO4ickaX0T6kMuwYZPsGj0qpczudzMyg/640?wx_fmt=jpeg)

从上表可以看出，在三种链路监控组件中，skywalking 的探针对吞吐量的影响最小，zipkin 的吞吐量居中。pinpoint 的探针对吞吐量的影响较为明显，在 500 并发用户时，测试服务的吞吐量从 1385 降低到 774，影响很大。然后再看下 CPU 和 memory 的影响，在内部服务器进行的压测，对 CPU 和 memory 的影响都差不多在 10% 之内。

### 4.2 collector 的可扩展性

collector 的可扩展性，使得能够水平扩展以便支持大规模服务器集群。

**1. zipkin**

开发 zipkin-Server（其实就是提供的开箱即用包），zipkin-agent 与 zipkin-Server 通过 http 或者 mq 进行通信，http 通信会对正常的访问造成影响，所以还是推荐基于 mq 异步方式通信，zipkin-Server 通过订阅具体的 topic 进行消费。这个当然是可以扩展的，多个 zipkin-Server 实例进行异步消费 mq 中的监控信息。

![](https://mmbiz.qpic.cn/mmbiz/yNKv1P4Q9eXAD3BGZWPJxT7DFIvSxRUeZYdwMzO1TPW0iciaxeO7AqWDj1Uh4iaJT6qkpeFm1N15icxVIKMUXRujog/640?wx_fmt=jpeg)

zipkin

**2. skywalking**

skywalking 的 collector 支持两种部署方式：单机和集群模式。collector 与 agent 之间的通信使用了 gRPC。

**3. pinpoint**

同样，pinpoint 也是支持集群和单机部署的。pinpoint agent 通过 thrift 通信框架，发送链路信息到 collector。

### 4.3 全面的调用链路数据分析

全面的调用链路数据分析，提供代码级别的可见性以便轻松定位失败点和瓶颈。

**zipkin**

![](https://mmbiz.qpic.cn/mmbiz/yNKv1P4Q9eXAD3BGZWPJxT7DFIvSxRUeUW0N5Elgjs50v9icplpUp4dNQz4aQ8W5ZTsicawKPokyicF1ZXHGJsqyA/640?wx_fmt=jpeg)

zipkin 链路调用分析

zipkin 的链路监控粒度相对没有那么细，从上图可以看到调用链中具体到接口级别，再进一步的调用信息并未涉及。

**skywalking**

![](https://mmbiz.qpic.cn/mmbiz/yNKv1P4Q9eXAD3BGZWPJxT7DFIvSxRUeSdGMyIeNkxYPeIKnFHibJDdsC5LGibdVkDaMb9QcC8phc51E2sUDvk8Q/640?wx_fmt=jpeg)

skywalking 链路调用分析

skywalking 还支持 20 + 的中间件、框架、类库，比如：主流的 dubbo、Okhttp，还有 DB 和消息中间件。上图 skywalking 链路调用分析截取的比较简单，网关调用 user 服务，由于支持众多的中间件，所以 skywalking 链路调用分析比 zipkin 完备些。

**pinpoint**

![](https://mmbiz.qpic.cn/mmbiz/yNKv1P4Q9eXAD3BGZWPJxT7DFIvSxRUeOdMnskZxJsG8iaEL3hvnTrOrqGY5JRtlRZIiaJFLlAbvq4usavl2qLibA/640?wx_fmt=jpeg)

pinpoint 链路调用分析

pinpoint 应该是这三种 APM 组件中，数据分析最为完备的组件。提供代码级别的可见性以便轻松定位失败点和瓶颈，上图可以看到对于执行的 sql 语句，都进行了记录。还可以配置报警规则等，设置每个应用对应的负责人，根据配置的规则报警，支持的中间件和框架也比较完备。

### 4.4 对于开发透明，容易开关

对于开发透明，容易开关，添加新功能而无需修改代码，容易启用或者禁用。我们期望功能可以不修改代码就工作并希望得到代码级别的可见性。

对于这一点，Zipkin 使用修改过的类库和它自己的容器 (Finagle) 来提供分布式事务跟踪的功能。但是，它要求在需要时修改代码。skywalking 和 pinpoint 都是基于字节码增强的方式，开发人员不需要修改代码，并且可以收集到更多精确的数据因为有字节码中的更多信息。

### 4.5 完整的调用链应用拓扑

自动检测应用拓扑，帮助你搞清楚应用的架构。

![](https://mmbiz.qpic.cn/mmbiz/yNKv1P4Q9eXAD3BGZWPJxT7DFIvSxRUesfopOXUuCeaVZC8OWg4VVQIicyqcqTR5MNtdCSAKZclzS2TkBCCELCQ/640?wx_fmt=jpeg)

pinpoint 链路拓扑

![](https://mmbiz.qpic.cn/mmbiz/yNKv1P4Q9eXAD3BGZWPJxT7DFIvSxRUewGd7pVKicKQ4eOXEpHbG7iadZVRwOiadoicicXZ3m87dBbYePk0vrkMHQ6Q/640?wx_fmt=jpeg)

skywalking 链路拓扑

![](https://mmbiz.qpic.cn/mmbiz/yNKv1P4Q9eXAD3BGZWPJxT7DFIvSxRUe4Gyia2LqlXynN0M4Wf7tIweJiagibT8xIWRoamHkD5XWicbLlzYLPaJJqA/640?wx_fmt=jpeg)

zipkin 链路拓扑

上面三幅图，分别展示了 APM 组件各自的调用拓扑，都能实现完整的调用链应用拓扑。相对来说，pinpoint 界面显示的更加丰富，具体到调用的 DB 名，zipkin 的拓扑局限于服务于服务之间。

### 4.6 Pinpoint 与 Zipkin 细化比较

#### 4.6.1 Pinpoint 与 Zipkin 差异性

1.  Pinpoint 是一个完整的性能监控解决方案：有从探针、收集器、存储到 Web 界面等全套体系；而 Zipkin 只侧重收集器和存储服务，虽然也有用户界面，但其功能与 Pinpoint 不可同日而语。反而 Zipkin 提供有 Query 接口，更强大的用户界面和系统集成能力，可以基于该接口二次开发实现。
    
2.  Zipkin 官方提供有基于 Finagle 框架（Scala 语言）的接口，而其他框架的接口由社区贡献，目前可以支持 Java、Scala、Node、Go、Python、Ruby 和 C# 等主流开发语言和框架；但是 Pinpoint 目前只有官方提供的 Java Agent 探针，其他的都在请求社区支援中（请参见 #1759 和 #1760）。
    
3.  Pinpoint 提供有 Java Agent 探针，通过字节码注入的方式实现调用拦截和数据收集，可以做到真正的代码无侵入，只需要在启动服务器的时候添加一些参数，就可以完成探针的部署；而 Zipkin 的 Java 接口实现 Brave，只提供了基本的操作 API，如果需要与框架或者项目集成的话，就需要手动添加配置文件或增加代码。
    
4.  Pinpoint 的后端存储基于 HBase，而 Zipkin 基于 Cassandra。
    

#### 4.6.2 Pinpoint 与 Zipkin 相似性

Pinpoint 与 Zipkin 都是基于 Google Dapper 的那篇论文，因此理论基础大致相同。两者都是将服务调用拆分成若干有级联关系的 Span，通过 SpanId 和 ParentSpanId 来进行调用关系的级联；最后再将整个调用链流经的所有的 Span 汇聚成一个 Trace，报告给服务端的 collector 进行收集和存储。

即便在这一点上，Pinpoint 所采用的概念也不完全与那篇论文一致。比如他采用 TransactionId 来取代 TraceId，而真正的 TraceId 是一个结构，里面包含了 TransactionId, SpanId 和 ParentSpanId。而且 Pinpoint 在 Span 下面又增加了一个 SpanEvent 结构，用来记录一个 Span 内部的调用细节（比如具体的方法调用等等），因此 Pinpoint 默认会比 Zipkin 记录更多的跟踪数据。

但是理论上并没有限定 Span 的粒度大小，所以一个服务调用可以是一个 Span，那么每个服务中的方法调用也可以是个 Span，这样的话，其实 Brave 也可以跟踪到方法调用级别，只是具体实现并没有这样做而已。

#### 4.6.3 字节码注入 vs API 调用

Pinpoint 实现了基于字节码注入的 Java Agent 探针，而 Zipkin 的 Brave 框架仅仅提供了应用层面的 API，但是细想问题远不那么简单。字节码注入是一种简单粗暴的解决方案，理论上来说无论任何方法调用，都可以通过注入代码的方式实现拦截，也就是说没有实现不了的，只有不会实现的。但 Brave 则不同，其提供的应用层面的 API 还需要框架底层驱动的支持，才能实现拦截。

比如，MySQL 的 JDBC 驱动，就提供有注入 interceptor 的方法，因此只需要实现 StatementInterceptor 接口，并在 Connection String 中进行配置，就可以很简单的实现相关拦截；而与此相对的，低版本的 MongoDB 的驱动或者是 Spring Data MongoDB 的实现就没有如此接口，想要实现拦截查询语句的功能，就比较困难。

因此在这一点上，Brave 是硬伤，无论使用字节码注入多么困难，但至少也是可以实现的，但是 Brave 却有无从下手的可能，而且是否可以注入，能够多大程度上注入，更多的取决于框架的 API 而不是自身的能力。

#### 4.6.4 难度及成本

经过简单阅读 Pinpoint 和 Brave 插件的代码，可以发现两者的实现难度有天壤之别。在都没有任何开发文档支撑的前提下，Brave 比 Pinpoint 更容易上手。Brave 的代码量很少，核心功能都集中在 brave-core 这个模块下，一个中等水平的开发人员，可以在一天之内读懂其内容，并且能对 API 的结构有非常清晰的认识。

Pinpoint 的代码封装也是非常好的，尤其是针对字节码注入的上层 API 的封装非常出色，但是这依然要求阅读人员对字节码注入多少有一些了解，虽然其用于注入代码的核心 API 并不多，但要想了解透彻，恐怕还得深入 Agent 的相关代码，比如很难一目了然的理解 addInterceptor 和 addScopedInterceptor 的区别，而这两个方法就是位于 Agent 的有关类型中。

因为 Brave 的注入需要依赖底层框架提供相关接口，因此并不需要对框架有一个全面的了解，只需要知道能在什么地方注入，能够在注入的时候取得什么数据就可以了。就像上面的例子，我们根本不需要知道 MySQL 的 JDBC Driver 是如何实现的也可以做到拦截 SQL 的能力。但是 Pinpoint 就不然，因为 Pinpoint 几乎可以在任何地方注入任何代码，这需要开发人员对所需注入的库的代码实现有非常深入的了解，通过查看其 MySQL 和 Http Client 插件的实现就可以洞察这一点，当然这也从另外一个层面说明 Pinpoint 的能力确实可以非常强大，而且其默认实现的很多插件已经做到了非常细粒度的拦截。

针对底层框架没有公开 API 的时候，其实 Brave 也并不完全无计可施，我们可以采取 AOP 的方式，一样能够将相关拦截注入到指定的代码中，而且显然 AOP 的应用要比字节码注入简单很多。

以上这些直接关系到实现一个监控的成本，在 Pinpoint 的官方技术文档中，给出了一个参考数据。如果对一个系统集成的话，那么用于开发 Pinpoint 插件的成本是 100，将此插件集成入系统的成本是 0；但对于 Brave，插件开发的成本只有 20，而集成成本是 10。从这一点上可以看出官方给出的成本参考数据是 5:1。但是官方又强调了，如果有 10 个系统需要集成的话，那么总成本就是 10 * 10 + 20 = 120，就超出了 Pinpoint 的开发成本 100，而且需要集成的服务越多，这个差距就越大。

#### 4.6.5 通用性和扩展性

很显然，这一点上 Pinpoint 完全处于劣势，从社区所开发出来的集成接口就可见一斑。

Pinpoint 的数据接口缺乏文档，而且也不太标准（参考论坛讨论帖），需要阅读很多代码才可能实现一个自己的探针（比如 Node 的或者 PHP 的）。而且团队为了性能考虑使用了 Thrift 作为数据传输协议标准，比起 HTTP 和 JSON 而言难度增加了不少。

#### 4.6.6 社区支持

这一点也不必多说，Zipkin 由 Twitter 开发，可以算得上是明星团队，而 Naver 的团队只是一个默默无闻的小团队（从 #1759 的讨论中可以看出）。虽然说这个项目在短期内不太可能消失或停止更新，但毕竟不如前者用起来更加放心。而且没有更多社区开发出来的插件，让 Pinpoint 只依靠团队自身的力量完成诸多框架的集成实属困难，而且他们目前的工作重点依然是在提升性能和稳定性上。

#### 4.6.7 其他

Pinpoint 在实现之初就考虑到了性能问题，www.naver.com 网站的后端某些服务每天要处理超过 200 亿次的请求，因此他们会选择 Thrift 的二进制变长编码格式、而且使用 UDP 作为传输链路，同时在传递常量的时候也尽量使用数据参考字典，传递一个数字而不是直接传递字符串等等。这些优化也增加了系统的复杂度：包括使用 Thrift 接口的难度、UDP 数据传输的问题、以及数据常量字典的注册问题等等。

相比之下，Zipkin 使用熟悉的 Restful 接口加 JSON，几乎没有任何学习成本和集成难度，只要知道数据传输结构，就可以轻易的为一个新的框架开发出相应的接口。

另外 Pinpoint 缺乏针对请求的采样能力，显然在大流量的生产环境下，不太可能将所有的请求全部记录，这就要求对请求进行采样，以决定什么样的请求是我需要记录的。Pinpoint 和 Brave 都支持采样百分比，也就是百分之多少的请求会被记录下来。但是，除此之外 Brave 还提供了 Sampler 接口，可以自定义采样策略，尤其是当进行 A/B 测试的时候，这样的功能就非常有意义了。

#### 4.6.8 总结

从短期目标来看，Pinpoint 确实具有压倒性的优势：无需对项目代码进行任何改动就可以部署探针、追踪数据细粒化到方法调用级别、功能强大的用户界面以及几乎比较全面的 Java 框架支持。但是长远来看，学习 Pinpoint 的开发接口，以及未来为不同的框架实现接口的成本都还是个未知数。

相反，掌握 Brave 就相对容易，而且 Zipkin 的社区更加强大，更有可能在未来开发出更多的接口。在最坏的情况下，我们也可以自己通过 AOP 的方式添加适合于我们自己的监控代码，而并不需要引入太多的新技术和新概念。而且在未来业务发生变化的时候，Pinpoint 官方提供的报表是否能满足要求也不好说，增加新的报表也会带来不可以预测的工作难度和工作量。

5 Tracing 和 Monitor 区别
----------------------

Monitor 可分为系统监控和应用监控。系统监控比如 CPU，内存，网络，磁盘等等整体的系统负载的数据，细化可具体到各进程的相关数据。这一类信息是直接可以从系统中得到的。应用监控需要应用提供支持，暴露了相应的数据。比如应用内部请求的 QPS，请求处理的延时，请求处理的 error 数，消息队列的队列长度，崩溃情况，进程垃圾回收信息等等。Monitor 主要目标是发现异常，及时报警。

Tracing 的基础和核心都是调用链。相关的 metric 大多都是围绕调用链分析得到的。Tracing 主要目标是系统分析。提前找到问题比出现问题后再去解决更好。

Tracing 和应用级的 Monitor 技术栈上有很多共同点。都有数据的采集，分析，存储和展式。只是具体收集的数据维度不同，分析过程不一样。

来源 | https://www.jianshu.com/p/92a12de11f18
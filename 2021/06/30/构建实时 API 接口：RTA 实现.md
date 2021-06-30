> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Qnw3HlokkXSnfRcoyjPPRw)

RTA 背景
------

RTA 是最近两年在国内兴起的一种投放模式，目前国内主流的流量媒体方都推出该项能力。如腾讯 / 头条在 2020 年正式对外推出该项服务，信也科技作为其中的广告主一方也几乎在同期开始进行陆续接入，以此来帮助我们进一步提升广告的精准投放效果。

RTA 接入要求
--------

*   接口响应时间在 60ms 内（包括网络和处理时间）
    
*   超时率要低于 2%
    
*   预计高点流量可达 20w/s
    

RTA 实现
------

### _实现原则_

基于 RTA 高并发且实时的业务要求，我们在前期和运维 / DBA / 基础组件的同学沟通，确保该并发的条件下我们的基础设施可以有效地承载，同时在一些设施上面进行有效的资源隔离，以防止 RTA 影响到其他业务。另一方面，在系统设计上，我们的处理逻辑不能有依赖慢设备或者慢操作的行为，资源上也需要能支持高并发的访问。综合考量后，我们作出如下的选择和主要设计原则：

*   数据存储：使用内存型设施（Redis、JVM 内存）
    
*   避免阻塞 / 耗时操作：可采用异步化手段
    
*   资源成本考虑：对于那些需要降低下游 QPS 的地方，可采用队列、缓存、批量操作等手段来进行优化
    
*   资源保护：比如跨系统边界的调用、有风险的本地代码块等，都可当成资源进行保护并提供有效的降级机制
    

### _系统视图_

应用的核心视图如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/7ianxQYSjic0oAedLeEOE72EROkEwDXMicicQK6pVvmicCT6llErQZUnIs3uNbN4iabpib6XYNzfcGPs44WgYcHaZwXTA/640?wx_fmt=png)

主要有 3 个服务，对于每个服务解释如下：

*   API 接口服务：提供实时接口供媒体方调用
    
*   消息处理、JOB 任务处理：处理业务数据，处理后的结果存储到 Redis 或者配置中心
    
*   Admin 服务：管理系统配置信息、业务规则等，处理结果存储到数据库，或作进一步转化后存储到配置中心
    

业务人员对每个 RTA 请求是否曝光（参竞）制订策略，不同媒体的策略规则和配置信息不一样，它们决定着媒体的 RTA 请求是否参竞以及对应的结果信息。其中，规则信息可简单理解为对逻辑运算的抽象（操作对象和操作符，以及操作结果之间的关系），可以让业务人员进行可视化的配置，配置后最终转换为一份规则契约，供 RTA 接口解析。借助配置中心的数据变更通知，API 实例可以拉取到变更后的信息、并直接转为 Java 内存对象供业务使用，因此运行态 API 代码所需的数据均直接来源于 JVM 内存或者 Redis，这样就保证了获取数据的速度。

下面是 API 接口的主要处理流程：

![](https://mmbiz.qpic.cn/mmbiz_png/7ianxQYSjic0oAedLeEOE72EROkEwDXMicicKl3QY50FoTt3A81FwCnBeSuicHEYxDoCtDpnE1NXlMSFcInvHU4wSIg/640?wx_fmt=png)

API 直接依赖的 2 个数据源是 Redis 和 JVM 内存，为保证 HTTP 的线程不阻塞，尽可能优先采用异步处理方式。通过上面的各种手段，我们可以满足实时性的要求。

### _数据存取_

由于业务上的特点，我们会面临大量的数据存储需求，业务上一个很小的规则可能会使用很大的存储资源。这要求我们谨慎设计数据存储，寻找有效的存取结构。

业务上用的数据，可以归结为如下 2 类存储：

*   JVM 本地存储：系统业务配置、业务规则策略、业务的控制信息、热 KEY 和黑名单等
    
*   Redis 存储：策略计算需要很多不同的数据，数据量比较大，每份数据以亿计
    

对于 JVM 本地存储，以对**热 key** 的处理为例进行说明。**热 key** 是指同一个设备号的曝光请求被媒体反复下发。在业务上线的初期，我们发现很多设备请求被下发很多次，有的每日可达上千万次，浪费了处理资源，需要某种策略进行应对。为此，我们设计了**收集反馈**的方式，具体流程：`API实例本地LFU队列收集 -> 上报 -> 统计 -> 反馈到API实例`，如下图所示

![](https://mmbiz.qpic.cn/mmbiz_png/7ianxQYSjic0oAedLeEOE72EROkEwDXMicicicNIlxSNFe4Rcx1COfjwUSJD0HhnGPlwVMhibYaM1OuWZiaaeB8ia3Rxhw/640?wx_fmt=png)

> _**LFU（Least Frequently Used）**_ 算法根据数据的历史访问频率来淘汰数据，其核心思想是 “**如果数据过去被访问多次，那么将来被访问的频率也更高**”。

对于使用 Redis 存储的业务数据，早期上线一段时间后耗费的内存空间超过 1.6TB，存储了超过 60 亿条的数据记录，数据稳定后还在以每日超过千万级的量增长。按这样的增长速度，不久之后成本将无法被覆盖。

出现该问题的主要原因一方面在于早期严重低估了数据量，所以直接采用了简单的`k/v`结构进行存储，另一方面在于数据稳定后的增长速度超出预期。所以在和业务方沟通如何针对策略保留最有效数据的同时，寻找更有效的存取手段。

我们在 Redis 中存储数据时，使用`protobuf`进行压缩，并让 Redis 使用`ziplist`作为`hash`和`zset`的底层实现，从而大大节省了内存空间。技术细节，可参考 Redis 官方文档中的存储优化章节。

目前我们的很多场景下，都用到整型的 k/v 存储结果。测试结果表明，在使用了改进的存储结构后，1 亿的数据只需要占用 1.1G 左右的 Redis 空间，存储优化率可高达 10 倍左右，效果很好，所以在大多数情况下我们尽量使用整型数据。对于`protobuf`也是一样，尽可能的使用整型数据类型，可以达到最好的压缩效果。

结合业务上的数据特点，我们使用了`hash/zset`存储结构，一方面节省内存，另一面将不同数据的 keys 划分到不同的连续区间内，方便管理数据。比如，知道某个范围内存储的是什么数据，就可以方便地进行 key 的清理，直接清理该区间即可。Redis 的 key 区间划分大致如下：

<table><thead><tr data-style="box-sizing: border-box; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204);"><th data-darkmode-bgcolor-16250231136857="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16250231136857="#fff|rgb(240, 240, 240)" data-style="box-sizing: border-box; text-align: left; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240);">业务信息</th><th data-darkmode-bgcolor-16250231136857="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16250231136857="#fff|rgb(240, 240, 240)" data-style="box-sizing: border-box; text-align: left; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240);">存储结构</th><th data-darkmode-bgcolor-16250231136857="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16250231136857="#fff|rgb(240, 240, 240)" data-style="box-sizing: border-box; text-align: left; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240);">key 区间</th></tr></thead><tbody><tr data-style="box-sizing: border-box; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204);"><td data-style="box-sizing: border-box; border-color: rgb(204, 204, 204);">业务数据 1</td><td data-style="box-sizing: border-box; border-color: rgb(204, 204, 204);">hash(ziplist)</td><td data-style="box-sizing: border-box; border-color: rgb(204, 204, 204);">0~200w</td></tr><tr data-darkmode-bgcolor-16250231136857="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16250231136857="#fff|rgb(248, 248, 248)" data-style="box-sizing: border-box; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-bgcolor-16250231136857="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16250231136857="#fff|rgb(248, 248, 248)" data-style="box-sizing: border-box; border-color: rgb(204, 204, 204);">业务数据 2</td><td data-darkmode-bgcolor-16250231136857="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16250231136857="#fff|rgb(248, 248, 248)" data-style="box-sizing: border-box; border-color: rgb(204, 204, 204);">hash(ziplist)</td><td data-darkmode-bgcolor-16250231136857="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16250231136857="#fff|rgb(248, 248, 248)" data-style="box-sizing: border-box; border-color: rgb(204, 204, 204);">200w~2000w</td></tr><tr data-style="box-sizing: border-box; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204);"><td data-style="box-sizing: border-box; border-color: rgb(204, 204, 204);">业务数据 3</td><td data-style="box-sizing: border-box; border-color: rgb(204, 204, 204);">zset(ziplist)</td><td data-style="box-sizing: border-box; border-color: rgb(204, 204, 204);">2000w~5000w</td></tr><tr data-darkmode-bgcolor-16250231136857="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16250231136857="#fff|rgb(248, 248, 248)" data-style="box-sizing: border-box; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-bgcolor-16250231136857="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16250231136857="#fff|rgb(248, 248, 248)" data-style="box-sizing: border-box; border-color: rgb(204, 204, 204);">业务数据 4</td><td data-darkmode-bgcolor-16250231136857="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16250231136857="#fff|rgb(248, 248, 248)" data-style="box-sizing: border-box; border-color: rgb(204, 204, 204);">zset(ziplist)</td><td data-darkmode-bgcolor-16250231136857="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16250231136857="#fff|rgb(248, 248, 248)" data-style="box-sizing: border-box; border-color: rgb(204, 204, 204);">5000w~Nw</td></tr></tbody></table>

下面举例说明 Redis 的存储场景。比如，我们需要对曝光的设备做一些策略，限制媒体每个设备每日到达多少量后不再进行曝光，这依赖于对设备进行计数。我们对接了多个媒体，总设备曝光请求数据每日高达上百亿次，预计每日会有数十亿的去重设备量。结合业务上的的特点，设计如下：

*   设备有并发，所以一定要原子操作，只能选择`INCR`命令（string、hash、zset）
    
*   媒体设备分别计数，每日每个媒体计数业务规则上有上限（业务上可满足）。这意味着每个计数器可能达到的最大计数值是确定的，亦即计数器所需的位数是有限的、固定的。
    

Redis 中对于整数类型采用的内部编码是`int`编码，对应 Java 里的`long`类型，占 8 个字节。在评估安全的并发进位问题后，我们将一个 8 字节拆开，取合适的 bit 数量作为某个媒体计数，结合 hash 存储后我们还可以获得数倍的空间节省，存储结构如下：

<table><thead><tr data-style="box-sizing: border-box; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204);"><th data-darkmode-bgcolor-16250231136857="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16250231136857="#fff|rgb(240, 240, 240)" data-style="box-sizing: border-box; text-align: center; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240);">进制</th><th data-darkmode-bgcolor-16250231136857="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16250231136857="#fff|rgb(240, 240, 240)" data-style="box-sizing: border-box; text-align: center; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240);">备用</th><th data-darkmode-bgcolor-16250231136857="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16250231136857="#fff|rgb(240, 240, 240)" data-style="box-sizing: border-box; text-align: center; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240);">备用</th><th data-darkmode-bgcolor-16250231136857="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16250231136857="#fff|rgb(240, 240, 240)" data-style="box-sizing: border-box; text-align: center; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); word-break: break-all;">媒体 6</th><th data-darkmode-bgcolor-16250231136857="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16250231136857="#fff|rgb(240, 240, 240)" data-style="box-sizing: border-box; text-align: center; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); word-break: break-all;">媒体 5</th><th data-darkmode-bgcolor-16250231136857="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16250231136857="#fff|rgb(240, 240, 240)" data-style="box-sizing: border-box; text-align: center; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); word-break: break-all;">媒体 4</th><th data-darkmode-bgcolor-16250231136857="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16250231136857="#fff|rgb(240, 240, 240)" data-style="box-sizing: border-box; text-align: center; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); word-break: break-all;">媒体 3</th><th data-darkmode-bgcolor-16250231136857="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16250231136857="#fff|rgb(240, 240, 240)" data-style="box-sizing: border-box; text-align: center; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); word-break: break-all;">媒体 2</th><th data-darkmode-bgcolor-16250231136857="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16250231136857="#fff|rgb(240, 240, 240)" data-style="box-sizing: border-box; text-align: center; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); word-break: break-all;">媒体 1</th></tr></thead><tbody><tr data-style="box-sizing: border-box; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204);"><td data-style="box-sizing: border-box; border-color: rgb(204, 204, 204); text-align: center;">0x</td><td data-style="box-sizing: border-box; border-color: rgb(204, 204, 204); text-align: center;">7F</td><td data-style="box-sizing: border-box; border-color: rgb(204, 204, 204); text-align: center;">FF</td><td data-style="box-sizing: border-box; border-color: rgb(204, 204, 204); text-align: center;">FF</td><td data-style="box-sizing: border-box; border-color: rgb(204, 204, 204); text-align: center;">FF</td><td data-style="box-sizing: border-box; border-color: rgb(204, 204, 204); text-align: center;">FF</td><td data-style="box-sizing: border-box; border-color: rgb(204, 204, 204); text-align: center;">FF</td><td data-style="box-sizing: border-box; border-color: rgb(204, 204, 204); text-align: center;">FF</td><td data-style="box-sizing: border-box; border-color: rgb(204, 204, 204); text-align: center;">FF</td></tr></tbody></table>

计数规则如下：

![](https://mmbiz.qpic.cn/mmbiz_png/7ianxQYSjic0oAedLeEOE72EROkEwDXMicicicEib2jRZvZ0PhB6FbjbM5Gia5jCA2kThW6Y2faRADyvTmAbCNRQzsx8w/640?wx_fmt=png)

### _  
  
网络问题_

我们上面提到接口的响应时间要在 60ms 以内，因此网络的问题影响很大。

事实上我们的时间大部分花费在网络上，距离的远近直接影响着网络时延。以目前对接的一些媒体来看，在接口消耗时间上，上海市内的来回网络耗时达 7ms 左右，深圳到上海的达 35ms 左右。在上线前差点因为网络耗时长的问题而上不了线，最后发现是测试工具默认没有开启 http 长连接导致测试不达标。

在上线后，当媒体方请求量增大时，接口超时严重无法达标，在反复确认业务代码的耗时没问题后，最后发现是负载均衡达到了瓶颈，在更换更大带宽的设备后，问题消失。

网络的延迟问题很难解决，但可以对接口返回的网络字节大小进行优化。除了用`protobuf`对接口报文的 body 部分进行压缩外，还可以擦除不必要的 http 请求头，达到更好的优化效果。比如在 protobuf 数据格式下我们发现响应头里会默认塞入`X-Protobuf-Schema`和`X-Protobuf-Message`两个请求头的数据，在 UTF-8 的编码下，每个英文字母会占用 1 个字节，擦除这 2 个响应头后可以节省大量的网络带宽。开始时，我们尝试在请求的 filter 的里进行擦除，但没有成功；最后通过深入探究 http 的 protobuf 消息的处理过程，重写了 protobuf 消息转换逻辑，达到想要的效果，代码如下：

```
public ProtobufHttpMessageConverter protobufHttpMessageConverter() {
	return new ProtobufHttpMessageConverter() {
		@Override
		protected void writeInternal(Message message, HttpOutputMessage outputMessage)
            	throws IOException, HttpMessageNotWritableException {
			MediaType contentType = outputMessage.getHeaders().getContentType();
			if (contentType == null) {
				contentType = getDefaultContentType(message);
			}
			if (PROTOBUF.isCompatibleWith(contentType)) {
				FileCopyUtils.copy(message.toByteArray(), outputMessage.getBody());
			} else {
				super.writeInternal(message, outputMessage);
			}
		}
	};
}

```

### _资源保护_

对资源进行保护和有效降级（限流熔断）非常重要。保护点主要基于业务上考虑来确定，可以是任意的代码片段，并尽可能提供降级手段，以保证我们的主业务不受影响。

我们定义了很多的保护点，在生产上遇到过几次问题，这些保护点机制都发挥了很大的作用，具体过程不详细展开。

总结
--

我们分享了前期在构建实时接口 RTA 方面的一些思考和实现点。目前 RTA 在生产上已经持续稳定运行了一段时间，每日请求数超过 170 亿次，QPS 达到了 30w/s，Redis 的访问达到 90w/s，处理的响应时间低于 7ms，我们的健康指标多次得到了媒体方的认可和赞许。

文中提及的方法，尤其是对 Redis 的存储优化，可供感兴趣的技术人员参考采用，很多场景下，节省空间的同时也带来了时间和 CPU 的消耗，可谓鱼与熊掌不可兼得。RTA 还在持续的构建和改进中，我们还面临着更多的数据存储和复杂的策略，适当的时候我们会继续分享所遇到的问题和解决方案。

作者介绍
----

yy，信也科技后端研发专家，目前负责市场信息流投放相关业务。如有问题随时沟通！
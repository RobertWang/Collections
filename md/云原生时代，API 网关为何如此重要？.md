> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg4NDQwNTI0OQ==&mid=2247567058&idx=2&sn=45155849852ce554aba4259846e7ce03&chksm=cfbb23bcf8ccaaaa15bfaaa4752dfd5606a2644aa817250021f2a46eeb69de944ce3df76dece&mpshare=1&scene=1&srcid=102922txmbMgRBzdp9LPBCur&sharer_sharetime=1666978387456&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

作者 | 温铭，Apache APISIX PMC 主席       

责编 | 张红月

出品 | CSDN（ID：CSDNnews）

API 是各个不同的应用程序和系统之间互相调用和传输数据的标准方式。在很多的开发团队中都是使用 API-first 的模式，围绕着 API 来进行产品的迭代，包括测试、Mock、文档、API 网关、Dev Portal 等，这就是 API 全生命周期管理（Full Life Cycle API Management）。

![](https://mmbiz.qpic.cn/mmbiz_png/VeJKXItpwPWf7XS3tqT5j3m4RicVvkJia4KmRKa57BIIr7XAJCfTuGCQc83zviaBP0D9fSpeQyWLGiaCicYnkm2WpTw/640?wx_fmt=png)

**API 解决的问题**

在 API 出现之前，数据的传输和交换并没有标准的方式，大多数情况下是通过数据库、Excel 表格、文本，或者是 FTP，不同的系统和程序通过各种五花八门的方式来沟通。这些混乱的背后，隐藏了巨大的开发成本和安全隐患：权限控制、数据精细管控、限流限速、审计等，都只能用笨重的方式来解决。这就是计算机世界中的 “巴别塔”（Tower of Babel），因此只有解决了不同语言开发的系统以及不同存储方案带来的问题，才有机会构建足够复杂的产品。

而 API 的出现，则成功地解决了巴别塔问题，开发者只需要关心其他系统对外暴露的 API 即可，无需关心底层实现和细节。

我们熟知的手机 App、网络游戏、视频直播、远程会议和 IoT 设备，都离不开终端设备与服务端的连接和数据传输，这些都是通过 API 完成的。

![](https://mmbiz.qpic.cn/mmbiz_png/VeJKXItpwPWf7XS3tqT5j3m4RicVvkJia4wIV1EdGtAngSs2lGEDj9K0Bzl65dkVbBichNGPxc5r8HmsgJBbtD8zA/640?wx_fmt=png)

### 

**为什么需要 API 网关**

API 网关是 API 全生命周期管理的关键基础组件，负责生产环境中 API 的配置、发布、版本回滚、安全、负载均衡等。API 网关是所有终端流量的入口，负责把终端的 API 请求路由到正确的上游服务进行处理，然后再把返回的数据返回给原始请求方，同时保证整个过程的安全、可靠和低延迟。

![](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAuj4QHxVgslkic9t9euibZSlbyPP9svEKu5u7d9sMt4t8BfIOH72DVre3rsrzACmG8OGo9X8vRicWxuhA/640?wx_fmt=png)

在最开始 API 数量不多的情况下，API 网关往往是一个由 Web Server 和上游服务两部分拼接而成的虚拟组件：由 Apache 和 NGINX 等组件完成最简单的路由转发、反向代理和和负载均衡，其他功能比如身份认证、限流限速等，则依靠上游服务自身进行实现。

但随着 API 的数量越来越多，喜欢 “偷懒” 的开发者们发现了一个严重的问题：他们需要在不同的上游服务中，重复实现身份认证、限流限速、日志等通用的功能，这不仅会浪费开发资源，而且一旦需要修改这部分的代码，就需要修改很多代码，这是版本管理和升级维护的噩梦。怎么办呢？聪明的开发者们很快就找到了解决方案：把这些通用的功能抽象到一个统一的组件中，把上述的两层结构改为一层结构，从上游的逻辑代码中剥离与业务无关的功能，然后增强 Apache 和 NGINX 这类组件。这就是第一代 API 网关的诞生过程。

让 API 网关承载更多非业务逻辑的功能，这就是 API 网关一直以来的进化方向。在这个过程中，前端开发者和后端开发者们，为了让产品能够更快的迭代，就对 API 网关提出了越来越多的需求，并不仅仅局限于负载均衡、反向代理、身份认证等传统功能，还在 gRPC、GraphQL、可观测性等方面提出了很多新的功能。

![](https://mmbiz.qpic.cn/mmbiz_png/VeJKXItpwPWf7XS3tqT5j3m4RicVvkJia4wQ2vZgibWoq4ibczickNm1Yl4kkrSMRAscsfyzYOxSOMB0HYdTSmE9hlA/640?wx_fmt=png)

### 

**API 网关的作用**

为了让 API 网关更灵活高效，开发者们在底层上也做了非常多创新，比如：

*   功能插件化。随着 API 网关上承载的功能越来越多，如何让这些功能之间更好的隔离以及让二次开发变得更加简单？插件，是完美的解决方案。因此，现在主流的 API 网关均采用插件来实现各种功能，比如在 Apache APISIX 中叫做 Plugins，在 Envoy 中则称之为 Filters，它们的含义是相同的。插件化可以让网关的开发者不再关心底层实现，用较少的开发资源就可以实现一个新的功能。
    
*   数据面和控制面的分离。在第一代 API 网关的实现中，数据面和控制面是在同一个进程中实现的，这样足够简单，但也带来了很大的安全隐患。由于数据面是直接对外提供服务的，如果被黑客入侵，就有机会获取到控制面的数据（比如 SSL 证书）和控制权，造成更大的破坏。因此，现在大部分的开源 API 网关，都是将两者分别部署的，中间通过关系型数据库或者 etcd 来进行配置的管理和同步。
    

以 Apache APISIX 为例，下面的架构图，诠释了以上两个创新：

![](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAuj4QHxVgslkic9t9euibZSlbyU1MJ3zvDOQDgATNZzM7A0M0iaDN04NMW9cAlB6saY4YKzEOiamzWjW3w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/VeJKXItpwPWf7XS3tqT5j3m4RicVvkJia4JxlFASkab3MUTtghhicJhaHhcJor04HtIFOZBVPvucx14Q4FaU24VTA/640?wx_fmt=png)

**云原生时代下的挑战  
**

过去十年，IT 领域最大的技术变革就是云原生。诞生于 2013 年的 Docker 拉开了云原生的帷幕，从此裸金属、虚拟机开始被容器所替代，单体架构开始被微服务所替代。但是云原生并不是简单的技术革命，其背后的推动力主要来自互联网产品的快速发展和激烈竞争，云原生相关的技术生逢其时，迅速流行并替代了之前的很多技术组件和方案。

具体到 API 网关在云原生中的挑战，主要来自以下两个方面：

**单体架构到微服务的转型**

在微服务架构逐渐被开发者认可和落地后，该架构释放了巨大的技术红利：每个微服务可以按照自己的节奏进行升级和发布，不需要担心与其他服务的耦合。产品的迭代因此变得敏捷，每天都可以进行几十次甚至几百次的发布。

但与此同时，微服务的发展也带来了一些副作用，比如：

*   API 和微服务的数量从最初的几十个，增长到几千个，甚至几万个；
    

*   出现故障时如何快速定位是哪一个 API 引起的？
    

*   如何保证 API 的安全？
    

*   如何做到服务熔断和服务降级？
    

API 网关无法独立解决安全性、可观察性、灰度发布等问题。它需要与 Prometheus、Zipkin、Skywalking、Datadog、Okta 等众多开源项目和 SaaS 服务合作，为企业提供更好的解决方案。

**动态和集群化管理**

容器和 Kubernetes 的普及，让动态成为所有云原生基础组件的标准特性。在 Kubernetes 的环境中，容器在不断的生成和销毁，弹性伸缩成为一个必选项而不是可选项。

想象一个场景：一家互联网电商公司做了一次促销，大量的用户在一个小时内涌入，促销结束后就会离开。对于传统架构的公司来说，他们需要事先采购一批物理服务器，来应对这些高峰时候的 API 流量；但是对于云原生架构的公司来说，就可以随时使用公有云上的弹性资源，根据 API 请求的数量，自动的调整网络、计算、数据库等资源的规模即可。那么伴随着容器弹性伸缩而来的技术挑战如下：

*   上游服务不断更换 IP 地址和端口;
    
*   IP 黑白名单的频繁更新;
    
*   服务健康的及时检测和异常处理;
    
*   API 的频繁发布;
    
*   服务注册和发现的及时性;
    
*   SSL 证书的热更新和自动轮转。
    

想要解决上述这些挑战，均需要依赖于动态。以 NGINX 为代表的第一代 API 网关，动态能力是非常弱的。因为 NGINX 是本地静态配置文件驱动的，所以变更任何配置都需要重启 NGINX 服务才能生效，这在云原生时代是不能被企业接受的。这就是第一代 API 网关的技术痛点之一。

在中国，有一家类似微软 Office 365 的 SaaS 办公软件公司 -- WPS，他们有数百台物理机在运行着 Apache APISIX，有近万核 CPU 在处理来自客户端的 API 请求，每天处理数百亿次 API 请求。

在这个超大规模的 API 网关环境下，开发者不可能去逐个修改每一个 API 网关的配置然后 Reload，他们希望有一个统一的控制台来操作整个集群。可惜的是，第一代 API 网关诞生的年代，并没有这么大的实例规模，也就没有考虑集群管理的需求。

![](https://mmbiz.qpic.cn/mmbiz_png/VeJKXItpwPWf7XS3tqT5j3m4RicVvkJia49OwAgFgic4BrtAgvibArIZwSlM6qsTFhyrHxUO9YXvLiaicV860eXL12ew/640?wx_fmt=png)

### 

**下一代 API **网关的**发展**

上述挑战和痛点，逐渐催生了新一代的 API 网关。

和第一代 API 网关不同的是，云原生时代诞生的下一代 API 网关是在开源社区的驱动下快速成长的。借助社区和众多开源贡献者的力量，这些 API 网关有机会形成一个正向的迭代和进化：

*   更快速的收集一线开发者及用户的需求和痛点
    
*   在开源项目中尝试解决这些问题
    
*   开源项目变得更加好用，吸引更多开发者使用
    

于是，我们看到下一代 API 网关突破了传统网关的负载均衡和反向代理的定位，而是承担起了 API 和流量的连接、调度、过滤、分析、协议转换、治理、集成等更多的职责。

![](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAuj4QHxVgslkic9t9euibZSlbydbZPVcqOiaeNdMzQ48nhLr99s2KibQ68LP73Wx3jtDklKdnxas01Qx8A/640?wx_fmt=png)

**支持更低成本的二次开发**

同时，让开发者能够以更低的成本进行二次开发，也成为了下一代 API 网关的亮点。集成是 API 网关的重要功能之一，对于下游是协议解析和各种协议的转换，包括 GraphQL、gRPC、Dubbo 等；对于上游是集成 Okta、Keycloak、Datadog、Prometheus 等身份认证、可观测性服务，以及公司内部的认证、日志、审计等服务。API 网关不可能覆盖集成过程中所有的组件，这时候不可避免地需要开发者通过插件的方式进行二次开发，来满足自己的业务需求。不同的 API 网关提供了不同的二次开发的编程语言和开发方式，Apache APISIX 和 Kong 都可以使用 Lua 来编写原生插件，Envoy 是使用 C++ 编写原生插件。同时，Apache APISIX 还可以使用 Go、Python、Node、Java 和 Wasm 来编写插件，这些主流的开发语言已经可以覆盖绝大部分开发者了。

![](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAuj4QHxVgslkic9t9euibZSlbyVvdRpKe8dok2102J8Uxh6Va8aiaXjmr4pia0HA7NltXGR8gicJvcJkgfA/640?wx_fmt=png)

开发者不必去学习 Lua 和 C++，就可以使用自己熟悉的编程语言在下一代 API 网关上进行开发，这让基础组件的开发变得更加简单。开源和易于二次开发，是下一代的 API 网关最重要的特点，它把更多的选择权留给了开发者自身，同时，开发者也可以更加放心的在多云、混合云的环境下使用 API 网关，不用担心被云供应商锁定。

![](https://mmbiz.qpic.cn/mmbiz_png/VeJKXItpwPWf7XS3tqT5j3m4RicVvkJia4ABve0LaC4reQDV11Bm7xdO7fmutic0r26CoIicQmrtvSYWlDqFrQq4AA/640?wx_fmt=png)

**基于双十一的 API 网关流量处理场景**

这里，我们用一个具体的例子来解释下现代 API 网关的作用。

在双十一时，电商都会有各种各样的商品促销活动，这段时间的 API 请求量是平时的几十倍。让我们先来看下如果没有 API 网关将会是怎样的技术架构：

![](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAuj4QHxVgslkic9t9euibZSlbyvibKVP28A0JqsZusYAZ8PiajYHeicdUC4uF6ejibOBFrUk2y807ev3Apjw/640?wx_fmt=png)

可以看到，在 Order 和 Payment 服务中，身份认证和日志记录功能是重复的。一个电商的服务，一般会有数千个不同的服务组成，这时候就会有大量的代码和功能是重复开发的。

下面是增加了 API 网关之后的架构图：

![](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAuj4QHxVgslkic9t9euibZSlbyvibKVP28A0JqsZusYAZ8PiajYHeicdUC4uF6ejibOBFrUk2y807ev3Apjw/640?wx_fmt=png)

从上图可以看到，我们在 API 网关层统一了公共的服务，后端服务只需要关心自身业务，为弹性伸缩提供了更多可能。

当促销开始时，客户端大量 API 请求涌入 API 网关的时候，后端服务需要进行快速的弹性伸缩，为了保障关键业务不受突发流量的影响，我们需要在 API 网关上识别恶意爬虫并实现限流限速、服务降级和熔断。此时，我们可以暂时关闭部分服务，比如商品评价、快递查询等。

但是库存信息、购买功能、支付功能等核心业务是绝对不能出现故障的，因此我们需要通过 K8s 来管理容器服务，再生成更多的服务副本来保证它的正常运行。此时 API 网关需要将客户端的 API 请求路由到新生成的副本服务，并且自动移除出现故障的服务，如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAuj4QHxVgslkic9t9euibZSlbyIe7hQF1uQRlD7p5YFrXvIyVItBXZN9YOLJfy3eb3GLlYic2IOQs43uw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/VeJKXItpwPWf7XS3tqT5j3m4RicVvkJia4CZGcD3icx8R99eaKJgQtlDSaSbffJBKRqyq9M6uFAje5hYW6E8NOmpw/640?wx_fmt=png)

**总结**  

API 网关并不是一个新的基础中间件，而是在产品快速迭代和技术架构的变迁中，变得越来越重要。而下一代云原生 API 网关的出现则解决了企业用户在集群管理，动态，生态，可观测性以及安全性等方面的痛点。

API 网关不仅可以处理 API 的流量，也可以来处理 Kubernetes Ingress 和服务网格的流量，进一步降低开发者的学习成本，帮助企业更好的统一管理流量。[](http://mp.weixin.qq.com/s?__biz=MjM5MjAwODM4MA==&mid=2650939625&idx=1&sn=511745022dac8030cb9b499532309cc1&chksm=bd5a513a8a2dd82cc199951071ac523eb22d4fa4a41ea080416af62d0cd701a9e449ee4f5cdb&scene=21#wechat_redirect)
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653556379&idx=1&sn=84feb71bb8d2eb9329c024001244d91c&chksm=81399f03b64e16154321a91f308d666c3e93cb2136f16fa9cb84358340d4bba29dc52744683f&scene=21#wechat_redirect)

导读：

**1. AfterShip API Gateway 需要解决的问题**

1.1 服务治理诉求

1.2 服务暴露的痛点

**2. AfterShip API Gateway  整体架构**

2.1 为什么选择使用 Kong

2.2 基于 Kong 改造后的 API Gateway 架构

2.3 稳定性保障实践

2.4 扩展性与易用性

**3. Kong 的一些实践经验**

3.1 基于 Kong 所面临的问题
------------------

3.2 Kong 配置管理和快速暴露 API
----------------------

3.3 如何对 Kong 进行性能优化

API Gateway 是随 “微服务” 概念而兴起的一种架构模式，原本一个庞大的单体应用系统被拆分成许多微服务系统并行独立维护和部署，服务拆分带来的变化是 API 规模成倍增长，所以使用 API Gateway 发布和管理 API 逐渐成为一种趋势。

AfterShip 是一家业务和团队遍布全球的电商 B2B SaaS 公司，从 2012 年成立至今，AfterShip 已经与全球 800 多家物流公司，包括 Amazon、 Microsoft、 Wish、 eBay、Shopify、迪卡侬在内的 10 万家客户达成了合作，成为了连接物流公司和电商平台的桥梁，并进入了全球 200 多个国家地区。AfterShip 从成立至今，共发布了 10 多款 to B 产品和 2 款 to C 产品，并且已经初步构建完成了电商 SaaS 产品矩阵。AfterShip 系统整体基于微服务架构之上，随着越来越多新的产品发布，及其产品之间的依赖越来越多，在服务治理上也面临越来越大的挑战。在此基础上 AfterShip 期望引入统一的 API Gateway 来解决 统一鉴权、版本管理、灰度、流控、降级等一系列问题，同时也期望兼具有更好的扩展能力以应对特殊定制的需求。

AfterShip 最终选型使用 Kong 作为 API Gateway 方案。业务研发人员通过配置，即可简单对外开放功能和数据，无需在每个业务系统都实现通用的 API 管理功能，能更聚焦在业务逻辑本身。下面将进一步介绍 AfterShip API Gateway 的诞生和演进过程。如何兼顾效率和稳定性，平衡功能复杂性和系统性能，并向云原生架构进行演进。

**1. AfterShip API Gateway 需要解决的问题**

**1.1 服务治理诉求**
--------------

基于微服务的架构，降低了服务的耦合度，但也增加了服务管理的难度。基于这样的架构考虑，有一个全局的入口来统一管理这些服务的诉求就越来越大。

在业务发展的早期，AfterShip 的鉴权、流控等基础能力是在每个业务服务中单独实现，但是随着业务的不断发展及其更多新的业务产品的出现，系统面临越来越大的挑战。作为服务全球客户的电商 SaaS 公司，安全、合规及其隐私保护方面一直被我们高度重视。我们通过了 ISO 27001 认证体系，并且也在严格遵循 GDPR 和 CCPA 相关的规范要求。我们需要有更好的机制来统一管理 API，在更好服务业务的同时，也尽可能多的规避潜在的安全风险。

![](https://mmbiz.qpic.cn/mmbiz_png/8XkvNnTiapOP1ud3QHVic4eysCuICXww9uGKT5TAAO18ic4T6XwrTdtUWI5eEZrRkY1BcSfKB8NaAXFxGlw8MMIgQ/640?wx_fmt=png)

 注: 引用自（https://github.com/Kong/kong）

**1.2 服务暴露的痛点**
---------------

公司的第一款产品 AfterShip 于 2012 年正式发布上线，随后陆续发布了 10 多款产品，to B 和 to C 的产品均包括在内。系统也经历多次的迭代演变，同时也由于有一些历史原因，整体的的服务管理方式不太统一。

### _南北流量_

![](https://mmbiz.qpic.cn/mmbiz_png/8XkvNnTiapOP1ud3QHVic4eysCuICXww9ubFeIE2FUTrNClyKZL4yhyb7WyGESNfVlZJLrkWiaJzM963A7tc7AOFg/640?wx_fmt=png)

早期因为历史包袱的问题，南北流向的服务暴露的管理方式不太统一：

*   一开始使用 GCE Ingress Loadbalancer 暴露服务，但功能较单一，基本上就是 7 层透明代理，没法做一些 Rewrite、Ratelimit 等策略；
    
*   部分服务使用 Nginx Ingress 暴露服务，解决一些特殊需求。同时通过 Nginx SideCar 的方式做 SSO 等鉴权，对不同 OpenAPI 的场景，也是由不同的业务方自己做鉴权。比如对现有一些服务，需要包一层 API 作为服务入口，同时注入 Nginx  SideCar。这种方式带来的问题是升级维护比较麻烦，特别是当需要不断迭代改进 API 的管理策略时。
    

_东西流量_

![](https://mmbiz.qpic.cn/mmbiz_png/8XkvNnTiapOP1ud3QHVic4eysCuICXww9uqzwQpQHbIjPfAXkOzbJUJ0BnHdJ1xv8ONt8ern3rqG6LDPDRwmRKsQ/640?wx_fmt=png)

之前服务之间的 API 调用整体也不太统一，有通过 K8S Service 的，也有通过 GCE Ingress 暴露的，授权方面也是通过服务自定义的鉴权方式 。

这种方式的问题，主要是行为不太统一，导致不太好做自动化授权。无论是服务提供方还是使用方，都需要关注服务的鉴权问题，严重影响开发效率。同时，服务的接入和监控层面都面临不少挑战。比如想知道当前服务被哪些服务所调用，比如想做一些 RateLimit 等的策略，这些服务治理方面的就比较难得到支持。

**2. AfterShip API Gateway 整体架构**

**2.1 为什么选择使用 Kong**
--------------------

AfterShip API Gateway 需要考虑能够支撑：

*   **接入层能力**：多协议支持，安全防护，流量治理，流量识别，流量转发，流量观察告警等 ;
    
*   **服务治理**：内部鉴权，熔断，重试，缓存等；
    
*   **扩展能力**：具备良好的插件扩展能力，方便我们自定义扩展满足需求。
    

AfterShip 自 2019 年初开始做网关，当时开源主要是有几个方向，基于 Nginx 的 OpenResty，Envoy ，还有基于 Go 实现的网关 Tyk 等。

因为 Nginx 性能较好，服务稳定，扩展能力不错，且公司团队对 Nginx 有比较丰富的维护经验，所以更偏向于基于 OpenResty 相关生态的开源解决方案。

行业基于 OpenResty 有几个开源项目：

*   Orange：处于半停滞状态；
    
*   APISIX：在 2019 年刚开始开源，当时还没正式的 release，最近我们也有在跟 APISIX 交流，也看到 APISIX 从正式发布后发展非常快，我们也在保持关注中；
    
*   Nginx Ingress ：定位做流量接入，如果需要做服务的治理和鉴权等，需要自己做大量的扩展；
    
*   Kong ：优势是技术成熟度高，易迁移性强，扩展性好，性能满足需求，同时提供很多开箱即用的插件，生态也不错。基于 OpenResty 可在现有基础上扩展，实现更复杂的特性。比如 AfterShip 定制的鉴权插件，以及一些解决特殊的场景的插件。不过 Kong 也有些不足，如本身一些实现较复杂，性能方面相比于其他解决方案来说没有优势。
    

**2.2 基于 Kong 改造后的 API Gateway 架构**

![](https://mmbiz.qpic.cn/mmbiz_png/8XkvNnTiapOP1ud3QHVic4eysCuICXww9uHgMGibyicwIcgGTgo83mkNntdLibCo9K9RhpDSgbaWlZD4h3RickOOYPww/640?wx_fmt=png)

### _南北流量_

我们将原来 Nginx SideCar 的方式，替换成统一的  Kong + GCE Ingress 方式暴露。同时提供多种鉴权方式：对应不同端和使用方，提供不同的鉴权和接入方式。

一个服务若想暴露 API 给外部使用，只需遵守对应的 API 规范便可。这样既可减少开发一层 API Service，又能通过扩展，保证统一鉴权和用户的登录状态。

### _东西流量_

AfterShip 暂时没有引入 Service Mesh 的方式，而是使用 Kong 的 Keyauth 做统一鉴权。对 AfterShip 而言，痛点是统一的鉴权方案和 API 服务治理能力，同时需要兼顾服务的改造成本和维护成本等。

**2.3 稳定性保障实践**
---------------

### _API  Gateway 的 HA 机制_

![](https://mmbiz.qpic.cn/mmbiz_png/8XkvNnTiapOP1ud3QHVic4eysCuICXww9uC2ljrh2kf27HuficwtJkiaeod5Gw0jkoGtK4Z4Rozl8nUSwL4Ma6ib8og/640?wx_fmt=png) 

使用 GCE Ingress Load Balancer 接入流量，处理 TLS 的接入，可减少 Kong 处理 TLS 性能损耗。通过 GCP 给提供 NEG 的方式，可直接连接到 Kong 实例的 Pod IP，减少一层  iptables 转发。同时稳定的接入层，可在 Kong 异常的时候发现问题，GCE Ingress Load Balancer 也是主备的架构，当出现问题的时候，可以通过工具脚本直接起到另一组实例。

多 AZ 部署 Kong , 同时通过 ClusterIP 的方式，代理到后端的服务。发布时，可通过灰度发布的方式，慢慢更新 Kong 实例。我们启用时，还没有 Kong Ingress Controller，直接采用了 ClusterIP 的方式。当然了，这块如果量比较大的场景下，还是会有 iptables 性能上的问题，不过可以方便做调整。

根据不同业务做集群隔离，不同端做不同实例的部署，防止单个业务的异常引发所有服务的不可用。

Kong 支持故障自愈，服务端接入了弹性伸缩模块，可根据 CPU 流量等指标进行快速扩容。除此之外，还支持快速摘除问题节点，以及更细粒度的问题组件摘除。

_灰度流量_

![](https://mmbiz.qpic.cn/mmbiz_png/8XkvNnTiapOP1ud3QHVic4eysCuICXww9uPPXdI9vhCpmZn0hcN3UiaSjMQCpGsJkia6MLakHljXdBTtSJQOKrf1FQ/640?wx_fmt=png)

注: 引用自（https://argoproj.github.io/argo-rollouts/architecture.html）

目前结合 Argo Rollout 做灰度流量的支持，可通过多个 Deployment 的方式控制 Pod 的数量来控制流量比例，也可在 API Gateway 里支持基于按流量比例灰度的场景。

_服务监控与日志管理_

![](https://mmbiz.qpic.cn/mmbiz_png/8XkvNnTiapOP1ud3QHVic4eysCuICXww9uSpvib6OxNicSveT3tr79B8BhJEUw5lcpo6CIpJiaGG8JYzcicicbib7Ya5ibw/640?wx_fmt=png)

我们通过结合多种监控策略来确保监控的完善性：

*   通过 Prometheus 和 Grafana 进行多级监控，实时上报请求调用信息，提供 Overview、GCE Ingress Load Balancer、Kong、Service、Consumer 等多个维度的监控，以覆盖链路的关键组件；
    
*   通过 Pingdom + Blackbox Export 做存活检查，Pingdom 支持在全球不同的节点检查服务的状态，除了可以监控服务本身的问题，也可以监控外部不同国家链路状态；Blackbox Export 则是更多的检查不同的内网服务检查链路的的存活和 TCP DNS 的一些状态，确保内网健康；
    
*   通过 GCP Stackdriver 做日志监控，异常的日志我们会存放到 GCP Stackdriver 上，然后通过监控日志异常关键字来发现问题，比如 5xx 状态码、Nginx 异常的日志等；
    
*   通过 NewRelic + OpenTelemetry 全链路追踪监控。
    

而告警方面，我们通过整合 PagerDuty（https://www.pagerduty.com/） 来作为我们的告警服务。

**2.4 扩展性与易用性**
---------------

### _自定义 Kong Plugins_

Kong 默认自带的插件集，按照功能的不同大致可以分为六大类：

*   Authentication 认证
    
*   Security 安全
    
*   Traffic Control 流量控制
    
*   Analytics & Monitoring 分析监控
    
*   Transformations 请求报文处理
    
*   Logging 日志
    

Kong 除了提供众多开箱即用的插件外，且有易于扩展的自定义插件接口，用户可使用 Lua 自行开发插件，同时支持 Go 插件的方式。但我们测试发现 Go 开发插件时性能并不好，主要是它通过 hook 到 Go Server 上做协议解析时比较消耗性能，所以我们基本还是用 Lua 进行自定义插件的开发集成。

我们自定义了插件，去支持较特殊的流控机制、JWT 鉴权机制、签名验证机制、协议转换机制、API Logging 等等。

### _权限管理平台_

每个服务都有自己的 API Key，一般服务可通过 API Key 访问，但还是需要服务授权才能被访问。AfterShip 开发了自助的 Access Platform ，服务可申请访问权限，只要服务的提供方审批通过，就可以。

![](https://mmbiz.qpic.cn/mmbiz_png/8XkvNnTiapOP1ud3QHVic4eysCuICXww9uNAJ7br4faEPHSKXYs6lLP6cibofY9PA7sE7nPnzJhlxXS9O26vP1lFw/640?wx_fmt=png)

为了保证安全性，所有的访问都有对应的审计日志，关联到网关的 Access Log。

**3. Kong 的一些实践经验**

**3.1 基于 Kong 所面临的问题**
----------------------

### _Konga 的权限问题_

目前社区 Dashboard  Konga 在权限设置做得较粗，没法控制到 Service 级别。企业版有一个付费的 Dashboard，但收费较贵。所以我们根据自己的权限体系，开发了一个后台系统支持 Service 级别的权限管理。

### _Prometheus Plugins 性能_

早期版本 Prometheus Plugins 的性能不太好，同时支持的 Metrics 的指标也不够多。我们通过扩展 HTTP Log Plugins 的方式，以类似 Log By Metrics 的方式，实现了 Metrics。

### _Admin 接口的保护_

Admin API 接口这块，官方镜像没有做保护，要暴露到内网，需添加一个鉴权的方式。

### _防止直接访问 SVC_

为了防止服务调用跳过网关，直接通过 SVC 的方式访问服务，可以使用 K8S NetworkPolicy 的方式：

![](https://mmbiz.qpic.cn/mmbiz_png/8XkvNnTiapOP1ud3QHVic4eysCuICXww9uwaj0RRhKxVuaagpscTXEUyYiayFnHyGJ4ZWvwlynMGtCgicpEbJj4qqg/640?wx_fmt=png)

### _内网和外网网络隔离_

因为 Kong 本身扩展较多，但会带来一些潜在的安全隐患，要做好网络隔离以避免安全问题。如通过 IP 白名单，防止因内部漏洞引发的安全风险。

**3.2 Kong 配置管理和快速暴露 API**
--------------------------

AfterShip 当前没有使用 Ingress 的方式，因早期引入 Kong 时还不支持。现在引入对架构调整较大，社区有几个工具可以做配置管理 Deck + Konga。

在较早期时，AfterShip 基于 Deck 这个工具做了扩展去管理配置，主要是扩展了基于 Service 级别划分文件，基于环境区分目录。好处是修改时，比较容易看出做了哪些修改，以及做 diff 配置，同时通过模板和脚本的方式，快速暴露 API。

Konga 主要是给业务看相关的配置和规则，这是因为 Konga 不太好做基于 Service 的权限管理。后续为了满足服务的需求，我们做了进一步优化，基于 Service 维度的权限管理平台，这样业务就可以快速管理部分插件，如 RateLimit 和 Ban 等策略。同时通过包装一层的服务暴露的自助平台，帮助业务暴露服务。

**3.3 如何对 Kong 进行性能优化**
-----------------------

虽然 Nginx 本身有着不错的性能，但 Kong 在 Nginx 之上扩展了很多插件，容易引入性能问题。下面介绍对 Kong 性能分析和优化的几种方式。

方式一：Perf

如果是 C 语言层面的性能定位，可用 Perf + 火焰图的方式，工具帮助我们去排查问题。需要特别注意，编译参数上需添加一些符号表的编译参数 -g -o0，不然会出现显示不了函数名，要自行计算服务号位。

方式二：SystemTap

OpenResty 提供了 openresty-systemtap-toolkit, stap++ 两个工具可以动态的 trace Nginx 和解析 LuaVM 的 Stack。不过这两个工具都是基于 SystemTap，原理也是通过编译 Linux 动态内核模块的方式加载到系统上跑，对环境有不少的依赖，需要内核的 debuginfo。

方式三：eBPF

基于 Linux 内核的 uprobes or kprobes 机制， 可通过 kubectl trace 的方式注入到宿主机中运行。但如果需要 Profile Lua 的堆栈，需自己写代码去解析 LuaVM 的一些堆栈。

方式四：OpenResty XRay

![](https://mmbiz.qpic.cn/mmbiz_png/8XkvNnTiapOP1ud3QHVic4eysCuICXww9ugy9hH4SJ9qRvLy05XiappxEw9ialHDV4Ztg8ibLSbHDERZpB7pUR1xQYQ/640?wx_fmt=png)

注: 引用自（https://blog.openresty.com.cn/cn/lua-cpu-flame-graph/?src=xray_web）

通过 Openresty XRay 可以比较容易获取到 Lua 的堆栈，同时分析内存使用的模块做进一步的优化。

**总结**

AfterShip 基于 Kong 构建了 API Gateway，架构在一定程度上满足了现有业务需求。我们也逐步聚焦在解决 API 如何更好的治理方面，及如何做到更好的安全和稳定保障方面。

Kong 有着活跃的社区，不错的扩展能力，不过它也有一些不足。随着业务发展，以及 Cloud Native 技术的演进，我们也持续在做相应的架构演进，同时也在逐步推动回馈开源社区，为开源社区做更多的贡献，相互的促进。
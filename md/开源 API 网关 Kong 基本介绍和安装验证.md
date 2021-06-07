> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1623031534&ver=3115&signature=Ei1gbqPKxLw7Vf0BXZl*88kw4PrrQ5scpfXcMbM-NkpEtP0KkWbIpUMby74e4mPCz-SaKamgnvmdnRJts5XDoeVcGkkxezKIHYsr3MDlJjOT5L4bvgml6W2wDgHLbXXR&new=1)

今天准备介绍下开源 API 网关 Kong，在 Gtihub 搜索 API 网关类的开源产品，可以看到 Kong 网关常年都是排第一的位置，而且当前很多都有一定研发能力的企业在 API 网关产品选型的时候基本也会选择 Kong 网关，并基于 Kong 网关进行二次开发和定制。

API 网关概述
========

![](https://mmbiz.qpic.cn/mmbiz_png/RQueXibgo0KPiazMIRREKJniacibc8micxeN8ibRnicDh32Qx90lxlslYWxzJj6WnWvoLEk7a5wOyMOBURFtoHCXDib9kA/640?wx_fmt=png)

简单来说 API 网关就是将所有的微服务提供的 API 接口服务能力全部汇聚进来，统一接入进行管理，也正是通过统一拦截，就可以通过网关实现对 API 接口的安全，日志，限流熔断等共性需求。如果再简单说下，通过网关实现了几个关键能力。

*   内部的微服务对外部访问来说位置透明，外部应用只需和网关交互
    
*   统一拦截接口服务，实现安全，日志，限流熔断等需求
    

从这里，我们就可以看到 API 网关和传统架构里面的 ESB 总线是类似的，这些关键能力本身也是 ESB 服务总线的能力，但是 ESB 服务总线由于要考虑遗留系统的接入，因此增加了：

*   大量适配器实现对遗留系统的遗留接口适配，多协议转换能力
    
*   进行数据的复制映射，路由等能力
    

对于两者，我原来做过一个简单的对比，大家可以参考。

![](https://mmbiz.qpic.cn/mmbiz_png/RQueXibgo0KPiazMIRREKJniacibc8micxeN8ibhv540LY10CLhkClV7Acfbk0yswADujWnzBgqrz0tG4d1bJ1mJ4IPQ/640?wx_fmt=png)

对于 API 网关进一步的功能介绍，可以我前面参考文章：

一文详细讲解 API 网关核心功能和 API 管理扩展

基于 Openresty 开发 API 网关
======================

![](https://mmbiz.qpic.cn/mmbiz_jpg/RQueXibgo0KPiazMIRREKJniacibc8micxeN8nkglaAtGHQXfuWDHDTZanQs4N0ud8MqFJWmBG4NzTaUXZs0oGJsjGg/640?wx_fmt=jpeg)

在谈 API 网关前，我们先谈下 Openresty。在前面文章谈到过，当前适合用于 API 网关的架构，一种是基于 Openresty 的，一种是基于 Go 语言的。

OpenResty® 是一个基于 Nginx 与 Lua 的高性能 Web 平台，其内部集成了大量精良的 Lua 库、第三方模块以及大多数的依赖项。用于方便地搭建能够处理超高并发、扩展性极高的动态 Web 应用、Web 服务和动态网关。

OpenResty® 通过汇聚各种设计精良的 Nginx 模块（主要由 OpenResty 团队自主开发），从而将 Nginx 有效地变成一个强大的通用系统 Web 应用平台。这样，Web 开发人员和系统工程师可以使用 Lua 脚本语言调动 Nginx 支持的各种 C 以及 Lua 模块，快速构造出足以胜任 10K 乃至 1000K 以上单机并发连接的高性能 Web 应用系统。

OpenResty® 的目标是让你的 Web 服务直接跑在 Nginx 服务内部，充分利用 Nginx 的非阻塞 I/O 模型，不仅仅对 HTTP 客户端请求, 甚至于对远程后端诸如 MySQL、PostgreSQL、Memcached 以及 Redis 等都进行一致的高性能响应。

为何 Openresty 适合用于 API 网关开发？

对于 API 网关，有一个最基础的核心功能即需要实现接口服务代理路由，而这个本身也是 Nginx 反向代理能够提供的一个标准功能。

如果应用场景比较简单，仅仅是实现统一接口对外暴露，可以看到很多企业实际并没有采用 API 网关，而是直接采用了 Nginx 来替代 API 网关的服务代理路由功能。

其次，对于常规的 API 网关的中间件研发，在研发完成后本身还要依托一个 Web 容器进行部署，同时还需要自己去实现类似路由代理等各种能力。

在这种场景下可以看到，依托于 Nginx，在其内部来扩展一个 Web 容器能力，既可以充分的利用 Nginx 本身的代理路由和性能优势就是一个重要的选择。而 OpenResty 本身可以看做是基于 Nginx 服务器，在其 worker 里面内嵌了一个 LuaJVM，通过这种方式来实现了两者的融合。同时又可以通过开发和定制各种 Lua 库来进行快速的功能扩展。

也正是这样原因，基于 Openresty 来扩展动态网关功能是一个很好的选择。

OpenResty 的安装

在 Centos 下可以通过 yum 很方便的安装 OpenResty，具体如下：

```
//安装yum-utils
# sudo yum install yum-utils
//添加 openresty 仓库
sudo yum-config-manager --add-repo https://openresty.org/package/centos/openresty.repo
//安装openresty
# sudo yum install openresty
//安装命令行工具
# sudo yum install openresty-resty
//至此安装成功，默认安装在  /usr/local/openresty
//测试
# sudo /sbin/service openresty start
# sudo /sbin/service openresty stop

```

注意在安装完成后如果通过浏览器访问，需要先关闭防火墙或者打开放行 80 端口。

Kong 网关概述
=========

![](https://mmbiz.qpic.cn/mmbiz_png/RQueXibgo0KPiazMIRREKJniacibc8micxeN8bJ8CJWTgpkGiaicwytuLicZ54VgGDf2A5wbXKZ8UiasKoNWoah4XEqlCUw/640?wx_fmt=png)

首先我们看下 GitHub 上对 Kong 网关的一些介绍

当我们决定对应用进行微服务改造时，应用客户端如何与微服务交互的问题也随之而来，毕竟服务数量的增加会直接导致部署授权、负载均衡、通信管理、分析和改变的难度增加。

面对以上问题，API GATEWAY 是一个不错的解决方案，其所提供的访问限制、安全、流量控制、分析监控、日志、请求转发、合成和协议转换功能，可以解放开发者去把精力集中在具体逻辑的代码，而不是把时间花费在考虑如何解决应用和其他微服务链接的问题上。

> Kong 网关是一款基于 OpenResty（Nginx + Lua 模块）编写的高可用、易扩展的，由 Mashape 公司开源的 API Gateway 项目。Kong 是基于 NGINX 和 Apache Cassandra 或 PostgreSQL 构建的，能提供易于使用的 RESTful API 来操作和配置 API 管理系统，所以它可以水平扩展多个 Kong 服务器，通过前置的负载均衡配置把请求均匀地分发到各个 Server，来应对大批量的网络请求。

KONG 本身提供包括 HTTP 基本认证、密钥认证、CORS、TCP、UDP、文件日志、API 请求限流、请求转发及 NGINX 监控等基本功能。目前，Kong 在 Mashape 管理了超过 15,000 个 API，为 200,000 开发者提供了每月数十亿的请求支持。

Kong 网关核心组件

*   Kong Server ：基于 nginx 的服务器，用来接收 API 请求。
    
*   Apache Cassandra/PostgreSQL ：用来存储操作数据。
    
*   Kong dashboard：官方推荐 UI 管理工具，也可以使用开源的 Konga 平台
    

Kong 采用插件机制进行功能定制，当前本身已经具备了类似安全，限流，日志，认证，数据映射等基础插件能力，同时也可以很方便的通过 Lua 定制自己的插件。插件完全是一种可以动态插拔的模式，通过插件可以方便的实现 Kong 网关能力的扩展。

Kong 网关核心组件关键特性

*   Cloud-Native 云原生：与平台无关，Kong 可以在从裸机到容器的任何平台上运行，并且可以在每个云上本地运行。
    
*   Kubernetes 集成能力：使用官方的 Ingress Controller 通过本地 Kubernetes CRD 声明性地配置 Kong，以路由和连接所有 L4 + L7 层网络流量。
    
*   动态负载均衡：在多个上游服务之间平衡流量。
    
*   基于哈希的负载均衡：具有一致的哈希 / 粘性会话的负载均衡。
    
*   断路器：智能跟踪不健康的上游服务。
    
*   健康检查：主动和被动监视上游服务。
    
*   服务发现：在 Consul 之类的第三方 DNS 解析器中解析 SRV 记录。
    
*   Serverless：直接从 Kong 调用和保护 AWS Lambda 或 OpenWhisk 功能。
    
*   WebSockets：通过 WebSockets 与您的上游服务进行通信。
    
*   gRPC：与 gRPC 服务进行通信，并通过日志记录和可观察性插件观察流量
    
*   OAuth2.0：轻松将 OAuth2.0 身份验证添加到您的 API。
    
*   日志功能：通过 HTTP，TCP，UDP 或磁盘 Log 对系统的请求和响应。
    
*   安全性：ACL，僵尸程序检测，允许 / 拒绝 IP 等...
    
*   Syslog：登录到系统日志。
    
*   SSL：为基础服务或 API 设置特定的 SSL 证书。
    
*   监控：对关键负载和性能服务器指标提供动态实时监控。
    
*   转发代理：使 Kong 连接到透明的中介 HTTP 代理。
    
*   认证：HMAC，JWT，Basic 等。
    
*   限流：基于许多变量的阻止和限制请求。
    
*   转换：添加，删除或处理 HTTP 请求和响应。
    
*   缓存：在代理层缓存并提供响应。
    
*   CLI：从命令行控制 Kong 群集。
    
*   开放 API 接口：Kong 可以使用其 RESTful API 进行操作，以实现最大的灵活性。
    
*   跨区域复制：跨不同区域的配置始终是最新的。
    
*   故障检测和恢复：如果您的 Cassandra 节点之一发生故障，则 Kong 不会受到影响。
    
*   群集：所有 Kong 节点自动加入群集，并在各个节点之间更新其配置。
    
*   可扩展性：Kong 本质上分布，只需添加节点即可水平扩展。
    
*   性能：Kong 通过扩展和使用 NGINX 作为核心轻松处理负载。
    
*   插件：可扩展的体系结构，用于向 Kong 和 API 添加功能。
    

Kong 网关的主要功能分析

![](https://mmbiz.qpic.cn/mmbiz_png/RQueXibgo0KPiazMIRREKJniacibc8micxeN8D5e8aAEy4yS6TQrdC2SlxBGb6VtPF1NRr5xraYliaoYAbhcqPORSibFQ/640?wx_fmt=png)

对于 Kong 网关本身基于 OpenResty 构建，因此前面介绍的 OpenResty 本身的特性也就成了 Kong 网关的特性，同时 Kong 网关本身天然和 OpenResty 融合在一起，不再需要依赖其他的中间件或 Web 容器进行部署。

其次就是 Kong 网关本身的插件化开发机制，通过插件对 Http 接口请求和响应进行拦截，在拦截处就可以实现各种 API 接口管控和治理的要求。插件通过 Lua 语言开发，可以动态插拔。

要实现和 Kong 网关本身的集成和管理本身也很简单，Kong 网关提供各类的 Rest API 接口，可以很方便的实现和 Kong 网关能力的对接。也就是说你完全可以自己开发要给 API 管控治理平台，而底层引擎用 Kong 网关。

支持高可用集群，节点之间的发现采用的 gossip 协议，当 Kong 节点启动后，会将自己的 IP 信息加入到 node 列表，广播给任何其他的 Kong 节点，当另一个 Kong 节点启动后，会去读取数据库中的 node 列表，并将自己的信息加入到列表中。

另外再来看下 Kong API 网关提供的一些基本功能：

API 注册和服务代理

对于原始的 API 接口进行注册，并提供服务代理能力，即不同的 API 接口注册到网关后，网关可以提供一个统一的访问地址和接口。这和 ESB 总线的代理服务道理是一致的，在服务注册到 API 网关后，所有的内部服务对外界都是透明的，外部的服务访问必须要通过 API 网关进行。

该部分对应到 Kong API 网关的服务注册和 Routing 部分的功能和能力上。

负载均衡

由于 Kong API 网关本身底层也是基于 Nginx 的，因此对应负载均衡的能力实际上是通过底层的 Nginx 来完成的。要在 Kong 上面实现负载均衡和 API 注册代理实际上需要分开为两个步骤进行。

即首先是要配置负载均衡能力，建立 Upstream 组，并添加多个 Target，其次才是进行 API 注册。

AUTHENTICATION 实现

通过 OAuth 2.0 Authentication 插件实现 user 端口的用户访问限制。其具体步骤如下：

*   注册 Oauth2 插件，配置参考
    
*   添加 Consumer 及 Consumer 对应的 credentials
    
*   申请 accesstoken 并访问，如果不带 token 访问将被拒绝
    

安全和访问控制

支持最基本的基于 IP 的安全访问控制和黑白名单设置。即为相应的端口添加 IP Restriction 插件扩展，并设置白名单（只有名单内的 IP 可以访问 API）。

这和我们当前 ESB 的基于 IP 的访问控制和授权是一个道理。但是我们的 ESB 更加灵活，多了业务系统这一层，即可以直接对业务系统这层统一进行授权。

如果没有授权，在进行访问的时候将返回 Your IP address is not allowed 的访问错误。

流量控制 - Traffic Control

流量控制在当前 Kong 网关上，从 GitHub 上的参考来看，主要是实现了可以基于单位时间内的访问次数进行流量控制，如果超过了这个访问次数，则直接提示流控约束且无法访问的提示。当前的流量控制，暂时不支持基于数据量的流控。

当前的流控暂时没有看到微服务网关常用的熔断操作，即服务并发或服务响应时间大过某个临界值的时候，直接对服务进行熔断和下线操作。

日志 Logging 实现

通过 File-log 插件实现对于每次访问日志的获取，需要注意为日志文件写权限，日志格式参考 Log Format。具体包括两个步骤。

*   为端口添加 File-log 插件，并设置为日志文件路径设为:/tmp/file.log
    
*   添加日志插件后，每次访问都会被记录
    

当前 Kong 网关日志都是写入到文件系统中，但是这块可以很方便的定制日志插件将日志写入到缓存库，时序数据库或分布式数据库。另外当前的日志 LOGGING 是没有提供日志查询的前端功能界面的，如果需要的话还需要自己开发对应的日志查询功能。

这个日志功能和我们当前的 ESB 总线的日志 Log 完全是类似的。但是我们 ESB 这块的能力更加强，包括后续的服务运行日志的统计分析和报表查看等。

Kong 网关插件说明
===========

![](https://mmbiz.qpic.cn/mmbiz_png/RQueXibgo0KPiazMIRREKJniacibc8micxeN8RxvIibEsa80B2so1Jhgibu82X9N824eQ67VSaZggkfpFnreI7LWLaZEw/640?wx_fmt=png)

从上图可以看到，Kong 网关是基于 OpenResty 应用服务器，OpenResty 是一个基于 Nginx 与 Lua 的高性能 Web 平台，其内部集成了大量精良的 Lua 库、第三方模块以及大多数的依赖项。用于方便地搭建能够处理超高并发、扩展性极高的动态 Web 应用、Web 服务和动态网关。而 Kong 核心基于 OpenResty 构建，并且拥有强大的插件扩展功能。

在 Http 请求到达 Kong 网关后，转发给后端应用之前，可以通过网关的各种插件对请求进行流量控制，安全，日志等各方面的处理能力。当前 Kong 的插件分为开源版和社区版，社区版还有更多的定制功能，但是社区版是要收费的。

目前，KONG 开源版本一共开放 28 个插件，如下：

> acl、aws-lambda、basic-auth、bot-detection、correlation-id、cors、datadog、file-log、galileo、hmac-auth、http-log、ip-restriction、jwt、key-auth、ldap-auth、loggly、oauth2、rate-limiting、request-size-limiting、request-termination、request-transformer、response-ratelimiting、response-transformer、runscope、statsd、syslog、tcp-log、udp-log。

以上这些插件主要分五大类，Authentication 认证，Security 安全，Traffic Control 流量控制，Analytics & Monitoring 分析 & 监控，Logging 日志，其他还有请求报文处理类。插件类似 AOP 开发中的横切功能，可以灵活的配置进行拦截控制，下面选择一些关键性的插件进行简单的说明。

黑白名单控制能力 - ip-restriction

Kong 提供的 IP 黑白名单控制能力还算相当强，从配置项里面可以看到主要可以针对两个维度进行配置，一个是针对所有的 API 接口还是针对特定的 API 接口，一个是针对所有的消费方还是特定的某个消费方。

对于 IP 配置可以是一个区段，也可以是特定的 IP 地址。但是黑白名单不能同时配置，其次当前没有一个功能是针对某一个系统提供的所有服务都启用黑名单或白名单功能。

日志记录能力 - syslog, file-log，http-log

这里主要日志的插件比较多，一个是 sysLog 在配置后可以直接将 Kong 产生的日志写入到应用服务器的系统日志文件中。如果配置了 file-log 则是单独写入到你指定的 file 文件中。对于 http-log 则是对于 http 服务请求，可以详细的记录请求的输入和输出报文信息，但是具体是记录到哪里，需要通过 config.http_endpoint 配置。

如果需要将 API 接口调用消息报文日志写入到分布式存储或数据库中，则需要自己定义相应的日志插件来接入写入问题。

熔断插件 - request-termination

该插件用来定义指定请求或服务不进行上层服务，而直接返回指定的内容，用来为指定的请求或指定的服务进行熔断。注意 Kong 的熔断插件感觉是临时对服务的禁用，而不是说当达到某一种监控阈值的时候自动触发熔断。从官方文档的应用场景也可以看到这点。

如果仅仅是这种方式的熔断话，实际上意义并不是很大。但是可用的地方就在于当某个业务系统进行发版部署的时候我们可以对该业务系统或该业务系统所提供的所有服务进行熔断。

限流插件 - rate-limiting

Kong 当前提供的限流相对来说还是比较弱，即主要是控制某一个 API 接口服务在单位时间内最多只能够调用多少次，如果超过这个次数那么网关就直接拒绝访问并返回错误提示信息。而在前面我讲限流和流量控制的时候经常会说到，就是限流实际上一个是根据服务调用次数，一个是根据服务调用数据量，需要在这两个方面进行限流。而里面更加重要的反而是数据量的限流，因为大数据量报文往往更加容易造成内存溢出异常。

安全认证类插件

当前 Kong 网关提供 basic-auth，key-auth、ldap-auth，hmac-auth 多种认证插件。

Basic-auth 基本认证插件，即我们根据用户名和密码来生成一个 base64 编码，同时将该编码和目标服务绑定，这样在消费目标服务的时候就需要在报文头填写这个 Base64 编码信息。

Key-auth 认证插件则是利用提前预设好的关键字名称，如下面设置的 keynote = apices，然后为 consumer 设置一个 key-auth 密钥，假如 key-auth=test@keyauth。在请求 api 的时候，将 apikey=test@keyauth，作为一个参数附加到请求 url 后，或者放置到 headers 中。

Hmac-auth 插件是设置绑定的 service 和 rout，以启动 hmac 验证。然后在 Consumers 页面中 Hmac credentials of Consumer 设置中添加一个 username 和 secret。

请求报文容量限制 - request-size-limiting

该插件用于限制请求报文的数据量大小，可以限制单个服务，也可以显示所有的 API 接口服务。这个实际上是很有用的一个功能，可以防止大消息报文调用导致整个 API 网关内存溢出。

支持 OAuth2.0 身份认证 - oauth2

Kong 网关支持 OAuth2.0 身份认证，OAuth2.0 协议根据使用不同的适用场景，定义了用于四种授权模式。即 Authorization code（授权码模式），Implicit Grant（隐式模式），Resource Owner Password Credentials（密码模式），Client Credentials（客户端模式）。

在四种方式中经常采用的还是授权码这种标准的 Server 授权模式，非常适合 Server 端的 Web 应用。一旦资源的拥有者授权访问他们的数据之后，他们将会被重定向到 Web 应用并在 URL 的查询参数中附带一个授权码（code）。

在客户端里， code 用于请求访问令牌（access_token）。并且该令牌交换的过程是两个服务端之前完成的，防止其他人甚至是资源拥有者本人得到该令牌。另外，在该授权模式下可以通过 refresh_token 来刷新令牌以延长访问授权时间，也是最为复杂的一种方式。

简单转换能力

对于简单转换能力通过 request-transformer 和 response transformer 两个插件来完成。Kong 网关提供对输入和输出报文简单转换的能力，这部分内容后续再详细展开介绍。从当前配置来看，主要是对消息报文提供了 Add, Replace,Rename,Append 等各种简单操作能力。

Kong 网关和其它网关对比

对于开源的 Kong 网关和其它网关的对比如下。

![](https://mmbiz.qpic.cn/mmbiz_jpg/RQueXibgo0KPiazMIRREKJniacibc8micxeN89T64QTXjldbJh3n6AAaxBAmPDjXMEX6uE4bfkHA1JNABq5UaLVdZSw/640?wx_fmt=jpeg)

从上面对比图也可以看到，Kong 网关在功能，性能，插件可扩展性各方面都能够更好的满足企业 API 网关的需求。因此我们也是基于 Konga 来进一步定制对 Kong 网关的管控治理平台。

![](https://mmbiz.qpic.cn/mmbiz_jpg/RQueXibgo0KPiazMIRREKJniacibc8micxeN82eYl1ASftBice1AKiasHIcVpfaBnYN60ww9O7glUoicmAleZAFR8k1D3A/640?wx_fmt=jpeg)

在整个定制中增加了基于 DB 适配的 Http Rest API 接口的自动发布，API 服务自动化注册，服务日志采集和服务日志查询，常见映射模板定制，接口服务的自动化测试等方面的能力。

Kong 网关的安装
==========

![](https://mmbiz.qpic.cn/mmbiz_png/RQueXibgo0KPiazMIRREKJniacibc8micxeN8L0arFRaFWCwnibMGciaNYlJ7VWoKNvQKfbPeZicMib2via8dLggqgBO5vUw/640?wx_fmt=png)

在这里以在 Centos7 下安装 Kong2.2 版本说明。注意 centos7 安装 Postgresql 不再列出，可以参考网上的文章进行安装。

```
# 安装kong-yum源
$wget https://bintray.com/kong/kong-rpm/rpm -O bintray-kong-kong-rpm.repo
$export major_version=`grep -oE '[0-9]+\.[0-9]+' /etc/redhat-release | cut -d "." -f1`
$sed -i -e 's/baseurl.*/&\/centos\/'$major_version''/ bintray-kong-kong-rpm.repo
$mv bintray-kong-kong-rpm.repo /etc/yum.repos.d/
  
# 清空缓存，重新生成缓存
$yum clean all && yum makecache
# 查看yum源信息
$yum repolist
# 更新yum源
$yum update -y

#############################
#通过yum安装kong
#注意如果出现和openresty不兼容，需要先下载老版本的openresty
$yum install -y kong
# 查看配置文件位置
$whereis kong
$kong: /etc/kong /usr/local/bin/kong /usr/local/kong

#############################
#安装postgresql数据库（略）
#############################

#数据库创建kong用户
$su - postgres
$psql -U postgres
 CREATE USER kong; 
 CREATE DATABASE kong OWNER kong;
 ALTER USER kong with encrypted password 'kong';  
#修改kong,postgresql连接配置
$cp -r /etc/kong/kong.conf.default /etc/kong/kong.conf
$vi /etc/kong/kong.conf
    database = postgres
    pg_host = 172.28.102.62
    pg_port = 5432
    pg_timeout = 5000
    pg_user = kong
    pg_password = kong
    pg_database = kong
    
# 初始化数据库
$kong migrations bootstrap [-c /etc/kong/kong.conf]
注意执行过程如遇到 failed to retrieve PostgreSQL server_version_num: FATAL: Ident authentication failed for user "kong",请给该用户设定密码，并修改postgres的授权方式为MD5，操作如下:
$alter user kong with password  'kong';
# 启动kong
$kong start [-c /etc/to/kong.conf]
#检查 KONG 是否正确运行
$curl -i http://localhost:8001/
或者
$kong health
#停止 KONG
$ kong stop

```

Kong-Dashboard 安装
=================

对于 Kong 网关本身也提供了 Kong Dashboard 管理页面，下面再看下 Dashboard 的安装。在安装 Dashboard 前需要首先安装 node.js，具体如下：

```
#解压安装包
$wget http://nodejs.org/dist/v8.1.0/node-v8.1.0.tar.gz 
$tar zxvf node-v8.1.0.tar.gz 
$cd node-v8.1.0

#进行编译和安装（注意编译需要很长时间）
$./configure –prefix=/usr/local/node/*
$make
$make install

ln -s /usr/local/node/bin/* /usr/sbin/  

#npm 配置
npm set prefix /usr/local
echo -e '\nexport PATH=/usr/local/lib/node_modules:$PATH' >> ~/.bashrc
source ~/.bashrc

```

在 node.js 安装完成后可以安装 Dashboard

```
#从git库克隆
git clone https://github.com/PGBI/kong-dashboard

#安装Kong Dashboard:
npm install -g kong-dashboard@v2

#启动kong-dashboard，注意和老版本有变化
#启动时候可以自己指定端口号如9001
kong-dashboard start -p 9001

#访问kong-dashboard
http://localhost:9001

```

在启动后再配置 Dashboard 需要绑定的 Kong Server，如下：

![](https://mmbiz.qpic.cn/mmbiz_png/RQueXibgo0KPiazMIRREKJniacibc8micxeN8n8mxv46ZbRy8EH3R6zUEZlOKTGYDkzY7sVpqEic4EjwudwPPia6rhccA/640?wx_fmt=png)

访问 Dashboard 界面如下：

![](https://mmbiz.qpic.cn/mmbiz_png/RQueXibgo0KPiazMIRREKJniacibc8micxeN8MyNzqZTicfHicqLtynEuJ6BnwWot8KQG1VAbTHevP1uH2yHmq5YzhDZw/640?wx_fmt=png)

可以看到 Dashboard2.0 版本的界面已经和 V1 版本有了较大的变化，在 Dashboard 上可以实现最近的 API 接口注册接入，路由，安全管理，限流熔断配置的。由于本身还有另外一个开源的 kong 网关管理平台 Konga，因此后续准备基于 Konga 来对网关的关键功能进一步做说明和介绍。

来源：

https://www.toutiao.com/a6899586550972301836/
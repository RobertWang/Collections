> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU0OTE4MzYzMw==&mid=2247548121&idx=2&sn=35fb944aaeedc529d7eab813e73685ea&chksm=fbb1b527ccc63c314a9d9fd9af8bad7c17ddd0b9cc054d37150e0954490e78c04029e775565c&mpshare=1&scene=1&srcid=10079erzbuwSwuVQDx7rpB7n&sharer_sharetime=1665141075879&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

**Serverless 架构的应用场景**

Serverless 架构的应用场景通常是由其特性决定的，所支持的触发器决定具体场景。如图 1-1 所示，CNCF Serverless Whitepaper v1.0 描述的关于 Serverless 架构所适合的用户场景如下。

*   异步并发，组件可独立部署和扩展的场景。
    
*   突发或服务使用量不可预测的场景。
    
*   短暂、无状态的应用，对冷启动时间不敏感的场景。
    
*   需要快速开发、迭代的业务。
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1sUGlEeLwOdsna6RWnNAQbHQcf6ISNzpu5C6mPAKULpzDFv4UdFKXickggt40DRMUM3oSORkvLo7ug/640?wx_fmt=png)

1-1 CNCF Serverless Whitepaper v1.0 描述的 Serverless 架构所适合的用户场景

CNCF 除基于 Serverless 架构的特性给出 4 个适用的用户场景之外，还结合常见的触发器提供了详细的例子。

*   响应数据库更改（插入、更新、触发、删除）的执行逻辑。
    
*   对物联网传感器输入消息（如 MQTT 消息）进行分析。
    
*   执行流处理（分析或修改动态数据）。
    
*   数据单次提取、转换和存储需要在短时间内进行大量处理（ETL）。
    
*   通过聊天机器人界面提供认知计算（异步）。
    
*   调度短时间内执行的任务，例如 CRON 或批处理的调用。
    
*   机器学习和人工智能模型。
    
*   持续集成管道，按需为构建作业提供资源。
    

CNCF Serverless Whitepaper v1.0 基于 Serverless 架构的特点，从理论上描述了 Serverless 架构适合的场景或业务。云厂商则站在自身业务角度来描述 Serverless 架构的典型应用场景。

通常情况下，当对象存储作为 Serverless 相关产品触发器时，典型的应用场景包括视频处理、数据 ETL 处理等；API 网关更多会为用户赋能对外的访问链接以及相关联的功能等，当 API 网关作为 Serverless 相关产品的触发器时，典型的应用场景就是后端服务，包括 App 后端服务、网站后端服务甚至微信小程序等相关产品的后端服务。

一些智能音箱也会开放相关接口，这个接口也可以通过 API 网关触发云函数，获得相应的服务等；除了对象存储触发以及 API 网关触发，常见的触发器还有消息队列触发器、Kafka 触发器、日志触发器等。

**1. Web 应用或移动应用后端**

如果 Serverless 架构和云厂商所提供的其他云产品结合，开发者能够构建可弹性扩展的移动应用或 Web 应用程序 ，轻松创建丰富的无服务器后端。而且这些程序在多个数据中心可用。图 1-2 所示为 Web 应用后端处理示例。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1sUGlEeLwOdsna6RWnNAQbH9OyADCK9a99ZjpL2mD4ia8PwGVK2v25AcAibTHd7Sib9Ypql63r0oSV1Q/640?wx_fmt=png)

1-2 Web 应用后端处理示例

**2. 实时文件 / 数据处理**

在视频应用、社交应用等场景下，用户上传的图片、音视频往往总量大、频率高，对处理系统的实时性和并发能力都有较高要求。此时，对于用户上传的图片，我们可以使用多个函数对其分别处理，包括图片的压缩、格式转换等，以满足不同场景下的需求。图 1-3 所示为实时文件处理示例。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1sUGlEeLwOdsna6RWnNAQbHX9FYzn6IsJsKyJUNy4zevnjpbW6OKcFZL9NH66GEzlUtO3euOE85hw/640?wx_fmt=png)

1-3 实时文件处理示例

我们可以通过 Serverless 架构所支持的丰富的事件源、事件触发机制、代码和简单的配置对数据进行实时处理，例如：对对象存储压缩包进行解压、对日志或数据库中的数据进行清洗、对 MNS 消息进行自定义消费等。图 1-4 所示为实时数据处理示例。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1sUGlEeLwOdsna6RWnNAQbHYicic1ONlofFGAZqIVEutwMIkfBIDjdWPIiaWBsAwMO1St2Tib3fSuFguA/640?wx_fmt=png)1-4 实时数据处理示例

**3. 离线数据处理**

例如：某证券公司每 12 小时统计一次该时段的交易情况并整理出该时段交易量 top5；每天处理一遍秒杀网站的交易流日志获取因售罄而产生的错误，以便准确分析商品热度和趋势等。函数计算近乎无限扩容的能力可以使用户轻松地进行大容量数据的计算。

利用 Serverless 架构可以对源数据并发执行 mapper 和 reducer 函数，在短时间内完成工作。相比传统的工作方式，使用 Serverless 架构更能避免资源的闲置，从而节省成本。

数据 ETC 处理流程可以简化为图 1-5。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1sUGlEeLwOdsna6RWnNAQbHDbEw8A3qVfGWJpt210SSaCfhepzibDIgubIyaptC9A5EwYfMgh0cVcA/640?wx_fmt=png)

1-5 数据 ETL 处理示例

**4. 人工智能领域**

在 AI 模型完成训练，对外提供推理服务时，基于 Serverless 架构，将数据模型包装在调用函数中，在实际用户的请求到达时再运行代码。

相对于传统的推理预测，这样做的好处是无论是函数模块还是后端的 GPU 服务器，以及对接的其他相关机器学习服务，都可以进行按量付费以及自动伸缩，从而在保证性能的同时确保服务的稳定。图 1-6 为机器学习（AI 推理预测）处理示例。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1sUGlEeLwOdsna6RWnNAQbHYPVxCXZoFYkxRyaalpjpfkGfVVCeoLdlIhdHLDY8NoTRTRh6ibz543A/640?wx_fmt=png)

1-6 机器学习（AI 推理预测）处理示例

**5. 物联网（IoT）领域**

目前，很多厂商都在推出自己的智能音箱产品—用户对智能音箱说一句话，智能音箱通过互联网将这句话传递给后端服务，然后得到反馈结果，再返给用户。通过 Serverless 架构，厂商可以将 API 网关、云函数以及数据库产品结合起来，以替代传统的服务器或者虚拟机等。

Serverless 架构一方面可以确保资源能按量付费，即用户只有在使用的时候，函数部分才会计费；另一方面当用户量增加时，通过 Serverless 架构实现的智能音箱系统的后端也会进行弹性伸缩，保证用户侧的服务稳定，且对其中某个功能的维护相当于对单个函数的维护，并不会给主流程带来额外风险，相对来说会更加安全、稳定等。图 1-7 为 IoT 后端处理示例。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1sUGlEeLwOdsna6RWnNAQbHvbKpSZAibia3HnHDnz0icEs0Qf5hkI2I8xMPICc75JTFOAvytQV3rCOZA/640?wx_fmt=png)

图 1-7 IoT 后端处理示例

**6. 监控与自动化运维**

在实际生产中，我们经常需要做一些监控脚本来监控网站服务或者 API 服务是否健康，包括是否可用、响应速度等。传统的方法是通过一些网站监控平台（例如 DNSPod 监控、360 网站服务监控，以及阿里云监控等）进行监控和告警。

这些监控平台的原理是用户自己设置要监控的网站和预期的时间阈值，由监控平台部署在各地区的服务器定期发起请求，进而判断网站或服务的可用性。当然，这些服务器虽然说通用性很强，但实际上并不一定适合。

例如，现在需要监控某网站状态码和不同区域的延时，同时设置一个延时阈值，当网站状态异常或者延时过大时，平台通过邮件等进行通知和告警。目前，针对这样一个定制化需求，大部分监控平台很难直接实现，所以开发一个网站状态监控工具就显得尤为重要。

除此之外，在实际生产运维中，对所使用的云服务进行监控和告警也非常有必要。例如：在使用 Hadoop、Spark 时要对节点的健康进行监控；在使用 Kubernetes 时要对 API Server、ETCD 等的指标进行监控；在使用 Kafka 时要对数据积压量，以及 Topic、Consumer 等的指标进行监控。

对于这些服务的监控，我们往往不能通过简单的 URL 以及某些状态进行判断。在传统的运维中，我们通常会在额外的机器上设置一个定时任务，以对相关服务进行旁路监控。

Serverless 架构的一个很重要的应用场景就是运维、监控与告警，即通过与定时触发器结合使用，实现对某些资源健康状态的监控与感知。图 1-8 为网站监控告警示例。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1sUGlEeLwOdsna6RWnNAQbHsek2T0oRqYmMDVE9KZDcqMpoMaHkTdvntsTuApaDdFgAVbehOXB5Zw/640?wx_fmt=png)

1-8 网站监控告警示例
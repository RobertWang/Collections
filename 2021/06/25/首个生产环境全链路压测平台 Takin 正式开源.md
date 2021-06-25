> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653556037&idx=1&sn=82c7fa25dfee918c4483af48b7d7ede8&chksm=81399cddb64e15cbac531c0702d354f0c89d60bdf56a2209d97fe634e7daa5492d6a6e90a122&mpshare=1&scene=1&srcid=06252pb8qyoGrIG0xiyfaZFj&sharer_sharetime=1624598069090&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

6 月 25 日，国内知名的系统高可用专家数列科技宣布开源旗下核心产品能力，对外开放生产全链路压测平台产品的源代码，并正式命名为 Takin。  

**什么是生产环境全链路压测？**

IT 系统是工程师结合具体的业务场景基于一系列的基础组件进行编码搭建而成的，基础组件本身的局限性，以及代码的不确定性，会使整个系统存在很大的不确定性，这种不确定性会让系统在面临一系列 “风险” 场景（高峰场景）时，表现得很脆弱，那该如何让系统具备反脆弱能力呢？

![](https://mmbiz.qpic.cn/mmbiz_jpg/9J2bE1YLwXjwa72HZOJdibKr6siaRhMtqm9ic6JZKoibyIhHvibOanwlDa8844gD7Lss8BBBlNvAxPxvznX3OueibbPA/640?wx_fmt=jpeg)

通过生产环境全链路压测，真实模拟 “风险” 业务行为场景，实时监控系统表现，提前识别和快速定位系统的中的不确定因素，并对不确定因素进行处理，优化系统资源配比，使用最低硬件成本，使系统从容面对各种 “风险” 场景，达到预期的系统性能目标。通过这种方法，在生产环境上落地常态化稳定压测体系，实现 IT 系统的长期性能稳定治理。

![](https://mmbiz.qpic.cn/mmbiz_png/9J2bE1YLwXjwa72HZOJdibKr6siaRhMtqmkTLll1muAl6pibRehHk6cnyzWBRWLS62OlbXJr5IhCPmaROvJZusyvw/640?wx_fmt=png)

性能测试经历了从线下到线上演变的四个阶段：

**1**

**需求驱动压测阶段**

需求驱动压测，大多采用简单的工具进行单接口或者单系统压测，也能进行一些简单的性能问题分析，但很多时候都没有专门的测试团队，需要开发进行自主压测。

**2**

**性能回归体系阶段**

组建专门的性能测试团队搭建线下性能测试质量平台，具备复杂场景全链路压测能力、性能问题定位能力。

![](https://mmbiz.qpic.cn/mmbiz_png/9J2bE1YLwXjwa72HZOJdibKr6siaRhMtqmq7Yempia4iaVDL0tnJDkah5ibN2Cte92KMasp2yugZoJWrahXPUyjZ0mA/640?wx_fmt=png)

> 在这一阶段有三个问题比较有代表性：
> 
> *   很多公司线下做了性能测试，但到了线上还是存在很多问题，以测试环境的压测结果来评估线上环境，效果不佳。
>     
> 
> *   业务增长、营销活动增加使测试工程师对活动保障心里没底，每逢营销活动问题频发影响公司形象。
>     
> 
> *   性能压测效率无法满足增长的性能压测需求，导致部分项目没有性能压测直接上线，线上故障频发。
>     

为了解决测试环境性能压测的不确定性，性能压测开始向生产环境进行演变，进入生产环境性能压测阶段。

**3**

**生产只读业务压测阶段**

在测试环境回归体系阶段上增加了生产只读业务的性能压测，对生产环境压测进行实践，搭建生产环境性能压测回归体系，具备只读业务生产压测的性能问题分析能力。

**4**

**全业务全链路压测阶段**

在上一个阶段的基础上增加写入业务的性能压测，进而开展对全业务实行全链路压测，具备全业务的性能压测能力、问题定位能力，做的更好一些还会增加系统防护能力，比如降级、限流、故障演练等。

**为什么要开源？**

正如数列科技 CEO 曹学锋在接受 InfoQ 专访时表示 “**我们开源 Takin 的初衷其实很简单，就是想让更多的企业用上好的产品，帮助企业提供更好的用户服务体验，释放更多的精力去拓展业务。相信大家的使用反馈对于产品本身的发展迭代也是具有正向作用的，互惠互利实现良性循环。**”

目前大多数企业仍在使用传统的性能压测方式，但随着分布式、微服务架构的发展，这种方式已经无法满足系统性能的保障，数列科技决定把这款生产环境全链路压测产品开源出来并正式命名为 Takin。

当然 Takin 要做的不止于此，开源最大的特性在于开放包容与创新。希望产品开源能以开放的工作方式激发技术创新，吸引更多业界优秀的开发人员加入到生产环境全链路压测技术的共创团队中，让技术更落地，连接不同的使用场景。

**什么是 Takin？**

Takin 是基于 Java 语言开发的一套生产全链路压测的系统，可以在**无业务代码侵入**的情况下，嵌入到各个应用程序节点，**实现生产环境的全链路性能测试，适用于复杂的微服务架构系统。**

![](https://mmbiz.qpic.cn/mmbiz_png/9J2bE1YLwXjwa72HZOJdibKr6siaRhMtqmNZBibsw8D8xLTH0bgtmBpYNf61aE1pJaaCfTUrmsqHwBhxicvicVguzGQ/640?wx_fmt=png)

Takin 架构图  

Takin 具备以下 4 个特点：  

（1）**业务代码 0 侵入**：在接入、采集和实现逻辑控制时，不需要修改任何业务代码；

（2）**数据安全隔离**：可以在不污染生产环境业务数据情况下进行全链路性能测试，可以在生产环境对写类型接口进行直接的性能测试；

（3）**安全性能压测**：在生产环境进行性能压测，对业务不会造成影响；

（4）**性能瓶颈快速定位**：性能测试结果直接展现业务链路中性能瓶颈的节点。

![](https://mmbiz.qpic.cn/mmbiz_png/9J2bE1YLwXjwa72HZOJdibKr6siaRhMtqmVCLDoN33sXiadGejicI9AhS7OJibUFufYpJfJz2LhEfrypVzkHNVa2YtQ/640?wx_fmt=png)

Takin 界面

**Takin 开源了哪些内容？**

**Takin 开源内容主要包括三个部分：Agent 探针、控制台以及大数据模块。**在 Java 应用程序中植入探针（Agent），它能收集性能数据、控制测试流量的流向，将数据上报给大数据模块，大数据模块会进行一些实时计算分析并对数据进行存储，控制台则负责这些业务流程的管理和展现。三个部分各司其职，为业务提供无代码侵入的、常态化的生产环境全链路压测服务。

![](https://mmbiz.qpic.cn/mmbiz_png/9J2bE1YLwXjwa72HZOJdibKr6siaRhMtqmq2oEQ2x22gZpCGgQ1fZWU7vLN7MuIMn7DyT4Wd6Xyykibs26HkoIMuw/640?wx_fmt=png)

GitHub 开源地址如下：

**Takin：**

**https://github.com/shulieTech/Takin**

**开源社区：**

https://news.shulie.io/?p=3024（了解详细操作文档）

未来任重而道远，秉承着帮助企业解决微服务架构治理及性能问题的初心，Takin 可以较大程度地帮助企业降低生产全链路压测平台的开发难度，真正做到为更多企业系统的性能和稳定性提供保障。

数列衷心希望 Takin 能和业界携手，共建更完整、更标准化的生产全链路压测生态圈。
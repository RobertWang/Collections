> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU2NDgwNjEyMA==&mid=2247484878&idx=1&sn=0f92d6dc646c5d85956a5bcafcbcbb14&chksm=fc442d49cb33a45f6c23ada1431798353f43f5093e59e315e632c90429fad9fa9fe01e6da61b&mpshare=1&scene=1&srcid=0129VAiAoSW2HPiUzJk28CMP&sharer_shareinfo=0a76dc9d506a911228cc6e33f483a707&sharer_shareinfo_first=0a76dc9d506a911228cc6e33f483a707#rd)

    1 月 15 日，咱们围绕 ERP 为核心，简析了 “多系统流向关系分析——ERP 和 CRM、MRP、PLM、APS、MES、WMS、SRM 的关系”，私信留言的朋友指出 1 个问题，也和大家致歉，即：总图里没有体现 ERP 与 SRM 的集成内容，但分项介绍却有。

    另一位私信留言的朋友问：能否讲一下以 MES 为核心，其与 ERP、SCM、WMS、APS、SCADA、PLM、QMS 等的集成关系。今晚宁宁宝宝有点兴奋，睡得晚，她妈妈好困啊，所以咱概述分享一下哈：

![](https://mmbiz.qpic.cn/mmbiz_png/I5Jo7NnTd79LTibRtQtwic7QxjGlwib8G9ZQOaA80jtcX6VmHAC3JEzLHJIdgGlW9nUfqt6ly1RgZzOBiaicQYvbw6A/640?wx_fmt=png&from=appmsg)

**1. MES 与 ERP 系统**：

    **关系**：MES 提供实时生产数据和状态，与 ERP 系统的生产计划、库存管理等模块进行协同。ERP 提供基础数据给 MES 以做同步。

    **区别**：MES 专注于制造过程的实时监控、作业调度和质量管理，而 ERP 系统更侧重于整体业务财务的管理和规划，包括财务、采购、销售、库存、生产等方面。

**2、MES 与 SCM 系统**：

  **关系：**MES 提供实时的生产数据、要货信息、状态等，帮助 SCM 系统实现更准确的库存管理和交付时间估算。SCM 给 MES 反馈到货时间。

 **区别：**MES 主要关注内部制造过程的监控和调度，而 SCM 系统则涵盖了物流、库存和订单等供应链活动的协调和优化。

**3、MES 与 WMS 系统**：

 **关系：**MES 将生产完成的产品信息、配送需求等传递给 WMS 进行库存管理和出入库操作，并更新库存状态。WMS 系统也会向 MES 提供相关的物料库存、配送任务和备件信息。

 **区别：**MES 聚焦于制造过程和生产计划的实时监控，而 WMS 系统则专注于仓储和物流管理，包括库存管理、订单拣选等。

**4、MES 与 APS 系统**：  

    **关系**：MES 将派工单执行结果与异常报警等信息给到 APS，APS 将派工单、调度指令给到 MES。  

    **区别**：MES 是制造执行系统，APS 是高级计划与排产系统，二者有时候在一款产品中为包含关系，MES 与 APS 结合可实现计划与车间执行闭环管理模式。

**5、MES 与 SCADA**：

    **关系**：MES 将任务信息、标准工艺参数传递给 SCADA，SCADA 将设备状态、运行参数等传递给 MES。  

    **区别**：SCADA 在工厂自动化系统中，处于 PLC 与 MES 之间。SCADA 主要是以设备为主，透过资料撷取与监控，来产生各种设备资料，提供给 MES；MES 是重视现正发生的现场生产情况，能够即时解决现在的生产资源瓶颈问题；就目标的区隔来看，SCADA 着重的是设备，MES 则是制程。

**6、MES 与 PLM 系统**：

 **关系**：PLM 将工艺路线、工艺参数、图纸、作业指导书、变更通知等传递给 MES。

 **区别**：MES 主要关注制造过程中的实时监控和控制，PLM 主要关注产品的设计、开发和营销等生命周期的各个阶段；MES 一般是制造现场人员应用，PLM 则是研发设计人员应用。

**7、MES 与 QMS 系统**：

 **关系：**MES 在质量管理方面扮演重要角色，通过记录和分析生产过程中的实时数据，支持质量监测、缺陷追溯和质量改进活动。MES 与 QMS 系统的集成可以实现更全面的质量管理和追溯能力。

 **区别：**MES 除了质量管理外，还包括作业调度、资源管理等功能，而 QMS 系统专注于质量控制、质量检验和质量改进的管理。但很多产品中，都把 QMS 作为 MES/MOM 的其中一个大模块。

![](https://mmbiz.qpic.cn/mmbiz_png/I5Jo7NnTd79LTibRtQtwic7QxjGlwib8G9ZyguMpLgx94qib0jL8GicgrkNt7HUQKRkcysprEYa30ib2R5uw6kHwDhZQ/640?wx_fmt=png&from=appmsg)

    MES 是一个技术、业务与管理要求非常高的方案，要做好一个项目不但需要优秀的软件架构设计与开发人员，更需要像工业工程、企业管理、流程专家及项目管理专家等专业人士的参与。因此，我们一直说：管理无小事，交付需谨慎。
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [news.shulie.io](https://news.shulie.io/?p=3024)

> 微服务架构在现代系统架构中已被普遍使用，业务复杂性和系统复杂性双重作用使得保障和维持整个系统的高可用性变得困难异常，同时对研发效率也有较大负面影响。 为了解决性能瓶颈保证系统的高可...

  
微服务架构在现代系统架构中已被普遍使用，业务复杂性和系统复杂性双重作用使得保障和维持整个系统的高可用性变得困难异常，同时对研发效率也有较大负面影响。

[![](https://news.shulie.io/wp-content/uploads/2021/06/image-17-1024x468.png)](https://news.shulie.io/wp-content/uploads/2021/06/image-17-1024x468.png)

为了解决性能瓶颈保证系统的高可用，需要对系统实施性能测试，但传统的性能测试有仿真性、局部性和黑盒性三大问题。

[![](https://news.shulie.io/wp-content/uploads/2021/06/image-18-1024x330.png)](https://news.shulie.io/wp-content/uploads/2021/06/image-18-1024x330.png)

  
在生产环境进行性能压测是公认的最优解决方案，但这也是一件极具挑战性的事情，容易污染现网的数据库、日志等数据，进行生产环境测试数据清理时操作复杂且危险性高，为此，生产环境全链路压测技术应运而生。

  
Takin 作为首款生产环境全链路压测开源产品，可以较大程度地帮助企业降低生产全链路压测平台的开发复杂度，在无业务代码侵入的情况下，获得链路治理、数据隔离、性能瓶颈定位等生产压测核心能力。  

### **什么是 Takin？**  

Takin 是基于 Java 的开源系统，可以在无业务代码侵入的情况下，嵌入到各个应用程序节点，实现生产环境的全链路性能测试，适用于复杂的微服务架构系统。  

[![](https://news.shulie.io/wp-content/uploads/2021/06/1-1024x420.png)](https://news.shulie.io/wp-content/uploads/2021/06/1-1024x420.png)

Takin 架构图  

**Takin 具备以下 4 个特点：**  
（1）业务代码 0 侵入：在接入、采集和实现逻辑控制时，不需要修改任何业务代码；  
（2）数据隔离：可以在不污染生产环境数据和日志的情况下实施性能测试，可以在生产环境对写类型接口进行直接的性能测试；  
（3）链路治理：能够帮助业务和微服务架构分析业务链路，以技术方式获得功能视角的链路信息；  
（4）性能瓶颈定位：性能测试结果可以直接展现整个链路中存在性能瓶颈的微服务架构节点。

[![](https://news.shulie.io/wp-content/uploads/2021/06/222-1024x596.png)](https://news.shulie.io/wp-content/uploads/2021/06/222-1024x596.png)

Takin 界面  

**Takin 的开源内容**  

Takin 开源内容主要包括三个部分：Agent 探针、控制中台以及大数据模块。在 Java 应用程序中植入探针（agent），它能收集性能数据、控制测试流量的流向，将数据上报给大数据模块，大数据模块会进行一些实时计算并对数据进行存储，控制台则负责这些业务流程的管理和展现。三个部分各司其职，为业务提供无代码侵入的、常态化的生产环境全链路压测服务。  

[![](https://news.shulie.io/wp-content/uploads/2021/06/333-1024x404.png)](https://news.shulie.io/wp-content/uploads/2021/06/333-1024x404.png)

  
GitHub 开源地址如下：  
[https://github.com/shulieTech/Takin](https://github.com/shulieTech/Takin)

[](https://news.shulie.io/wp-content/uploads/2021/06/98634e4a7a1aa02d109f2741bcb29b4a.jpg)

*   [关于 Takin](https://docs.shulie.io/docs/forcecop/Takin-about)
*   [社区版使用文档](https://news.shulie.io/?p=2987)
*   [社区版使用视频](https://news.shulie.io/?p=3065)

发布者：shulieNews，转载请注明出处：数列科技
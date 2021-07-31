> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI4NTM1NDgwNw==&mid=2247494528&idx=2&sn=fffb2f95ff1fc370a84c44a8f69f5d2e&chksm=ebefd5d8dc985cceaa9255d1d7b400e9d8bc30568fad6a575e4a9d6478b7838b1128e5fb6dd0&scene=21#wechat_redirect)

**大家好，我是磊哥。**

1、什么是微服务
========

**1.1、架构演进**

架构的发展历程是从单体式架构，到分布式架构，到 SOA 架构，再到微服务架构。

图 1：架构演进

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsggic4544RPGcAiaJJQeAYhmOOmHFo82GcZrxlAIhmyoN0BLGg3zN76jPFkM3yUyjqVhCzicxr4Qu8gw/640?wx_fmt=png)

单体架构：未做任何拆分的 Java Web 程序

图 2：单体架构示意图

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsggic4544RPGcAiaJJQeAYhmOXvFovvmWAPWy4OYKrKWRLL5x8urLtZ17XgbcGYCvqOMbyeQeRwbzGg/640?wx_fmt=png)

分布式架构: 按照业务垂直划分，每个业务都是单体架构，通过 API 互相调用。

图 3：分布式架构示意图

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsggic4544RPGcAiaJJQeAYhmO6O2yZACPUg6AgMhGvDOw9MGROkOLoyRLa3PyDkMk3UmVFHB8nFhTWg/640?wx_fmt=png)

SOA 架构：SOA 是一种面向服务的架构。其应用程序的不同组件通过网络上的通信协议向其它组件提供服务或消费服务，所以也是分布式架构的一种。

图 4：SOA 架构示意图

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsggic4544RPGcAiaJJQeAYhmOtW1Yw0IAgzD9JXwiblUFDNVDNMqYEjMsyscMz4E6MRmWARa0Q6nIh3w/640?wx_fmt=png)

**1.2、微服务架构**
-------------

微服务架构在某种程度上是 SOA 架构的进一步的发展。

微服务目前并没有比较官方的定义。微服务 Microservices 之父，马丁. 福勒，对微服务大概的概述如下：

> 就目前而言，对于微服务业界并没有一个统一的、标准的定义（While there is no precise definition of this architectural style ) 。
> 
> 但通常在其而言，微服务架构是一种架构模式或者说是一种架构风格，它提倡将单一应用程序划分成一组小的服务，每个服务运行独立的自己的进程中，服务之间互相协调、互相配合，为用户提供最终价值。
> 
> 服务之间采用轻量级的通信机制互相沟通（通常是基于 HTTP 的 RESTful API ) 。每个服务都围绕着具体业务进行构建，并且能够被独立地部署到生产环境、类生产环境等。
> 
> 另外，应尽量避免统一的、集中式的服务管理机制，对具体的一个服务而言，应根据业务上下文，选择合适的语言、工具对其进行构建，可以有一个非常轻量级的集中式管理来协调这些服务。可以使用不同的语言来编写服务，也可以使用不同的数据存储。

图 5：微服务定义思维导图

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsggic4544RPGcAiaJJQeAYhmObARfpCYGvN5F05FlPNIUmlRBib9ypOomkzH8On37uW5v5A6JN04wyKg/640?wx_fmt=png)

图 6：微服务架构示意图

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsggic4544RPGcAiaJJQeAYhmOtotGNHF80zTBGlhp25cPrMV5E4zYEhslmGIE6eyzXKvguw1EQZc1TQ/640?wx_fmt=png)

**1.3、微服务解决方案**
---------------

目前最流行的两种微服务解决方案是 Spring Cloud 和 Dubbo。

2、SpringCloud 概览
================

**2.0、什么是 SpringCloud**
-----------------------

Spring Cloud 作为 Java 言的微服务框架，它依赖于 Spring Boot ，有快速开发、持续交付和容易部署等特点。Spring Cloud 的组件非常多，涉及微服务的方方面面，井在开源社区 Spring、Netflix Pivotal 两大公司的推动下越来越完善。

Spring Cloud 是一系列组件的有机集合。

图 7：SpringCloud 技术体系

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsggic4544RPGcAiaJJQeAYhmOX2HGicH1bn3dFBkY2ia4RibOibvfEUsLX5Eib4qymF5ibQTVcw6fhma2841w/640?wx_fmt=png)

图 8：SpringCloud 技术体系思维导图

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsggic4544RPGcAiaJJQeAYhmOO2icMFjaFEC8SHDJO1Jjs5lq34MiaH4zxeuvdicgymyyGF7KbrCzZKHkQ/640?wx_fmt=png)

**2.1、SpringCloud 主要组件**
------------------------

### **2.1.1、Eureka**

Netflix Eureka 是由 Netflix 开源的一款基于 REST 的服务发现组件，包括 Eureka Server 及 Eureka Client。

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsggic4544RPGcAiaJJQeAYhmOF2u8BHBPHgToqDfp3hEPMd3xsr8cOnyc2EWJhMSqubRW4UibjVg1hdQ/640?wx_fmt=png)

### **2.1.2、Ribbon**

Ribbon Netflix 公司开源的一个负载均衡的组件。

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsggic4544RPGcAiaJJQeAYhmOLib4jhdnNmZaJHlibvdaibR9FP4oxDdOaxDNxVuwfOJb0UIkvmmwgpKSQ/640?wx_fmt=png)

### **2.1.3、Feign**

Feign 是是一个声明式的 Web Service 客户端。

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsggic4544RPGcAiaJJQeAYhmORaiax4pDGoREakSPHLXgY0ev3draG6EjhRACwwppP92Jkt1agvU70sg/640?wx_fmt=png)

### **2.1.4、Hystrix**

Hystrix 是 Netstflix 公司开源的一个项目，它提供了熔断器功能，能够阻止分布式系统中出现联动故障。

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsggic4544RPGcAiaJJQeAYhmOLqDbcjR99vsw59B51mB7icbGjOpnjeNwV2zcQkbJOpSic6AzYZ0aEkbA/640?wx_fmt=png)

### **2.1.5、Zuul**

Zuul 是由 Netflix 孵化的一个致力于 “网关 “解决方案的开源组件。

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsggic4544RPGcAiaJJQeAYhmOkX3JrBV18hZ8NC9Yp7ibKWD3wicnPkq31av12iaklDAtYddeT6D9RBeIw/640?wx_fmt=png)

### **2.1.6、Gateway**

Spring Cloud Gateway 是 Spring 官方基于 Spring 5.0、 Spring Boot 2.0 和 Project Reactor 等技术开发的网关， Spring Cloud Gateway 旨在为微服务架构提供简单、 有效且统一的 API 路由管理方式。

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsggic4544RPGcAiaJJQeAYhmOZibjsBicoGANZwT2kODEp77Kh9lt4n2RiczlsOYBXFEqBpKOkn2vtibbww/640?wx_fmt=png)

### **2.1.7、Config**

Spring Cloud 中提供了分布式配置中 Spring Cloud Config ，为外部配置提供了客户端和服务器端的支持。

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsggic4544RPGcAiaJJQeAYhmOkDQN1LWYO1sRMHbP3oIAgVw2PvleER24UMZz7ibktuGujq49ECosM1Q/640?wx_fmt=png)

### **2.1.8、 Bus**

使用 Spring Cloud Bus, 可以非常容易地搭建起消息总线。

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsggic4544RPGcAiaJJQeAYhmOt7dCL3jtRwgnNujuwnEmu469LhCnZ4Q8ZNibbCoLM1ghQJ2Zau3kYGA/640?wx_fmt=png)

### **2.1.9、OAuth2**

Sprin Cloud 构建的微服务系统中可以使用 Spring Cloud OAuth2 来保护微服务系统。

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsggic4544RPGcAiaJJQeAYhmOH4zdHPo7oLB2qZqOgQdT1bWVGP6vpRYya2sI4Mr2CPbdbuU4aibd6cw/640?wx_fmt=png)

### **2.1.10、Sleuth**

Spring Cloud Sleuth 是 Spring Cloud 个组件，它的主要功能是在分布式系统中提供服务链路追踪的解决方案。

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsggic4544RPGcAiaJJQeAYhmO4GHibmHibG6CcByr7yp7ic8GNEoVfmoY1nosGoYKibATT8rQOhibke87CUw/640?wx_fmt=png)

本文中对架构的演进及 Spring Cloud 构建微服务的基本组件进行了概览。

![](https://mmbiz.qpic.cn/mmbiz_png/UtWdDgynLdZYlL1A5TuKX9Xic9WY0sD5mjdl7yeGkbyWm67PRv3ejL4ZDS5aPAWe5XOOAVFibaaswdDTk8YCWqoQ/640?wx_fmt=png)

```
作者：三分恶
```

链接：cnblogs.com/three-fighter/p/13485459.html
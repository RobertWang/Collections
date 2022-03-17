> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA3OTc0MzY1Mg==&mid=2247530155&idx=3&sn=cef7fc03dc26e3cd0db6648044ce4616&chksm=9face960a8db6076d32868378192a9640550be98d19504a1c4688c75ddf29d3346063381b64d&mpshare=1&scene=1&srcid=0316CdHicK2b6R8fgi1rEIEa&sharer_sharetime=1647400989195&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

我们生活中都听说了 DDD，也了解了 DDD，那么怎么将一个新项目从头开始按照 DDD 的过程进行划分与架构设计呢？  

**一、专业术语**

各种服务

IAAS：基础设施服务，Infrastructure-as-a-service

PAAS：平台服务，Platform-as-a-service

SAAS：软件服务，Software-as-a-service

**二、架构演变**

![](https://mmbiz.qpic.cn/mmbiz_png/fEsWkVrSk54nVNmKDIqgKCh5Wfk6Z6iaB3NWRWkibiaFMc0K4AjqlJLx5QnRAeqLGtSw2NgnAlI8mIhhoNEI5uNFg/640?wx_fmt=png)

从图中已经可以很容易看出架构的演进过程，通过对三个层的举例来进行说明：

**SAAS**：比如我们最早的就是单体应用，多个业务之间可能都没有进行分层，之后我们业务多了，都各自混淆在一起，后来我们就通过 MVC、SSM、分层等方式进行业务拆分，保证业务与业务之间解耦

**PAAS**：随者业务的增长，我们打算分离出一个子系统，但是成本太高，每次都需要从头搭建一个子系统，效率低下。这时我们就抽取除了一些通用技术，比如 mesh、SOA、微服务等方式来隔离系统，且对通用技术复用来快速搭建一个系统

**IAAS**：比如订单服务并发量高，单台服务器已经无法满足要求，这时我们需要多台服务器，可能有 windows 的、linux、mac，想要快速部署就需要屏蔽 OS，于是就有了 VM、Docker、K8S 等技术来屏蔽 OS

**三、限界上下文**

**限界上下文概念**

![](https://mmbiz.qpic.cn/mmbiz_png/fEsWkVrSk54nVNmKDIqgKCh5Wfk6Z6iaBhVZ6rOcvfsqv8NKUavVUhlboK2WFN6M15FwmfgnLP8HZqrPbRlRcibQ/640?wx_fmt=png)

**BC 与业务的关系**：

通过对业务的划分，比如订单系统，订单是一个子域；库存是一个子域；

其中商品再不同的子域中所表示的意义也不同，比如在订单上下文中的商品表示商品的单价、折扣等等；而在库存的上下文中商品表示商品的库存量、成本、存放位置等。

**BC 与技术的关系**：

多个子域之间必须需要在应用层进行聚合，而聚合的过程中就引出了技术方案，比如订单到库存到支付，他们应该采用同步方式；这几个子域调用通知都应该是异步，那么可能就需要消息中间件或其它技术方案

**限界上下文划分规则**

![](https://mmbiz.qpic.cn/mmbiz_png/fEsWkVrSk54nVNmKDIqgKCh5Wfk6Z6iaBdVsLTDm9SMy9QNYj9s2mbfxIlLMoYQrn7sibA4UnjUsIxibOohAVEr6Q/640?wx_fmt=png)

一般来说，先考虑团队规模，来决定最终需要划分到多细粒度的 BC，如果团队规模过小而 BC 过细，则对后期的运维、部署、上线都会造成很大的负担；

在确定好粒度后，可以对语义相关性、功能相关性 - 业务方向、功能相关性 - 非业务方向进行划分

按照以上的规则划分之后就得到了多个 BC 啦

**一个 BC 代表一个微服务吗？**

![](https://mmbiz.qpic.cn/mmbiz_png/fEsWkVrSk54nVNmKDIqgKCh5Wfk6Z6iaBNkdIDcoX0jsXAnKgzUptHEkic7CxuQzF9K3tNsQauWjZwWk0GGBPiaSw/640?wx_fmt=png)

**概念**：微服务一般是指将高度相关功能的一个开发部署单元，有自己的技术自治性、技术选型、弹性扩缩容、发布上下频率等，说白了就是各自维护一个业务，然后多个业务组成一个系统，多个业务之间各自管理

关系：这里的 BC 其实就是一个领域或一个模块或一个业务，如果两个领域相关性很高，就可以包含多个 BC，或者如果一个领域访问量非常大，则需要部署在一个微服务中以提高性能

**四、领域驱动设计的四重边界**

![](https://mmbiz.qpic.cn/mmbiz_png/fEsWkVrSk54nVNmKDIqgKCh5Wfk6Z6iaBnLTPWa9icE939Ahmj0kPqCibsuZxVoRyIEXSe29cr9c8icwP3vWFicOE6Q/640?wx_fmt=png)

根据上图所示，我们通过四重来进行架构设计：

**分而治之**：DDD 通过规划四重边界，把领域知识做了合理的固化和分层。业务有核心领域和支持域、业务域中又拆分成多个限界上下文（BC），一个 BC 中又根据领域知识核心与否进行分层，领域层中按照多个业务（子域）的强相关性进行聚合成一个子域

【第一重边界】确定项目的愿景与目标，确定问题空间，确定核心子领域、通用子领域（多个子领域可以复用）、支撑子领域（额外功能，如数据统计、导出报表）

【第二重边界】解决方案空间里的限界上下文就是一道进程隔离层面的物理边界

【第三重边界】每个限界上下文内，使用分层架构划分为：接口层、领域层、应用层、基础设施层之间的最小隔离

【第四重边界】领域层里为了保证各个领域的完整性和一致性，引入聚合的设计作为隔离领域模型的最小单元

**五、整洁分层架构**

![](https://mmbiz.qpic.cn/mmbiz_png/fEsWkVrSk54nVNmKDIqgKCh5Wfk6Z6iaBZOuFWr1cXicf3zKicJkZPD8sUWocxrRKj8fW9m8GDkzvuM1mPDq7WZ7w/640?wx_fmt=png)

具体说明看图中备注，总的来说就是通过实现与接口分离，让 domain 层尽量独立，而不耦合与任何模块，这里面包含了领域模型的业务逻辑代码，但不会依赖于具体技术实现，可以很方便更换基础设施层，提供给第三方 web 调用 service

**六、六边形架构**

![](https://mmbiz.qpic.cn/mmbiz_png/fEsWkVrSk54nVNmKDIqgKCh5Wfk6Z6iaBgX4eWVr1Sz59NNMicDIqU3FxjGRaewXnoOdmJxiakVbzHlcwib89sD2bA/640?wx_fmt=png)

**主动适配**：指来⾃于 UI、命令⾏等输⼊型命令， controller 就是⼀种端⼝，端⼝的具体实现就是应⽤逻

辑⾃身。因此端⼝和具体实现都在应⽤系统的内部。

**被动适配**：指访问存储设备，外部服务等。每种访问就是⼀种端⼝，具体实现是各个具体的中间件。因

此端⼝在整个应⽤系统的⾥部，具体实现在系统的外部。

每⼀种输⼊和输出都是⼀个端⼝，每个端⼝都有具体的实现逻辑，因此整个应⽤系统的架构就是⼀些列

的端⼝ + 适配逻辑组成，架构图就是⼀个多边形形状。有⼏个端⼝需要根据应⽤系统的具体情况⽽定，

只是六个端⼝⽐较形象⽽得名为六边形架构。

特点：1. 外层依赖内层使得依赖更合理。端⼝就是接⼝，依赖接⼝编程。借此保证了应⽤和实现细节之

间的隔离。2. 可测试更好

**七、洋葱架构**

![](https://mmbiz.qpic.cn/mmbiz_png/fEsWkVrSk54nVNmKDIqgKCh5Wfk6Z6iaBXhKpfE9zxicvyylwLKbBgpwveH0nnxdguLib5nPzLcB6kb79Fo5wzaYQ/640?wx_fmt=png)

洋葱架构针对六边形架构更进⼀步把内层的业务逻辑分为了 DDD 概念的应⽤服务层、领域服务层和领域

模型层。

特点：

（1）围绕独⽴的领域模型构建应⽤

（2）内层定义接⼝，外层实现接⼝

（3）依赖的⽅向指向圆⼼（注意：洋葱架构提倡不破坏耦合⽅向的依赖都是合理的，外层可以依赖直接内层，也可以依赖更⾥⾯的层）

（4）所有的应⽤代码可以独⽴于基础设施编译和运⾏

**八、总结**

目前领域驱动设计是目前比较流行的一种架构设计，只需要按照领域驱动设计的四重边界进行架构设计，就能够很好的对各个领域解耦，对后期的业务垂直扩展、功能的水平扩展提供了良好的基础。

出处：https://blog.csdn.net/qq_32828253/article/details/110673205
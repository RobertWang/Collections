> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653555980&idx=1&sn=2ae362b2b3d0552ef83d40db92588e5a&chksm=81399c94b64e15825e3cb9c5bfc6142b3f169de097c0f45aa04cc5f18924645ceff1ba333d5b&mpshare=1&scene=1&srcid=0622HPO35BDG1n8PkP9UISzw&sharer_sharetime=1624348641071&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

**1. 请简单介绍自己**

好多人以为我名字是于雨，其实是 2012 年使用微信时用的微信名字，后来入职阿里时也用作花名，现在已经成了我的常用名了。

从业十一年来一直在服务端基础架构研发一线，工作内容广泛，涵盖  RPC、NoSQL 存储、实时监控告警、即时通信、消息推送等方面。研究内容也挺广泛，2018 年到 2019 年在 Google 内搜索 RocksDB，个人的技术博客一直在 first page，我大概也是国内最早的 gogo protobuf 和 pulsar 研究和鼓吹者。

起家工作语言是 C/C++，2013 年在 chinaunix 上接触到 Go 语言，跟着 go 官方博客开始学习，所以在 Go 语言方面我算是野路子出身，没有人正式带过我。个人热爱开源，从 2015 年为 Redis 贡献代码开始，2015 年使用 C++11 重构 Muduo，2016 年构建 dubbogo，2018 年接触 Pika。当时除了给 pika 修复 bug，主要构建 Pika  和 Redis 之间数据代理 PikaPort，PikaPort 先后在 360、新浪、脉脉、全民快乐、顺丰、快手、好未来等公司投入使用，直到今年为止不少好未来和新浪不少 DBA 还因为这个工具找我交流。

目前业余主要精力在 dubbogo 社区推动 dubbogo 项目。业余时间，除了 Dubbogo 社区，还在即时通讯圈子与从业者进行技术交流，IM 方面个人有七八年实战项目经验积累。

**2. 聊聊你最近一年正在做的项目，它的技术价值怎样？它的行业发展状况是怎样？你负责项目的技术亮点和挑战能否展开讲讲？**

个人从 2018 下半年开始全力构建 dubbogo 社区，其主要开源项目当然是 https://github.com/apache/dubbo-go。

这个项目的技术价值，对我个人而言，是我打通我的即时通信技术体系的关键，我个人认为技术是触类旁通的，微服务治理技术和 MQ 技术比起来，仅仅少了一块存储技术，其他无差。它自身的技术价值就更不用说了，个人认为它是 Dubbo 迈进云原生技术时代的基石。

就其行业发展状况而言，dubbogo 目前发展非常好，目前登记的使用方有 35 个，实际上有更多，行业涵盖了电子商城、即时通信、交通出行、游戏、第三方支付、快递、旅游、车联网、IoT、健康医疗、云平台、视频网站、数字货币、情感交流 APP、少儿教育、教育电子设备、营销培训、税务办理、汽车交易、视频聊天、文档办公等方方面面，详见 https://github.com/apache/dubbo-go/issues/2 。

dubbogo 社区有贡献者 100 多人，apache dubbo committer 23 人，apache dubbo PMC 5 人。社区基础项目在 https://github.com/dubbogo，孵化成熟后即捐献到 https://github.com/apache，已经总体给 apache 组织贡献了 5 个 项目。总体项目【包含 getty\gost\hessian2\triple\v3router  等】代码有 17 万行之多，其中不少代码被国内其他同行项目不打招呼 “借鉴” 过去。

dubbogo 首先是 dubbo 的 go 语言实现，其第一要义就是全面兼容 dubbo。dubbo 底层的网络库是 netty，其原生序列化协议库是 hessian2，2016 年 4 月开始构建 dubbogo 时我首先要做的就是 使用 go 语言实现一个类似于 netty 的网络库 getty [https://github.com/apache/dubbo-getty]，其次是构建 hessian2 协议的 go 语言版本 [ https://github.com/apache/dubbo-go-hessian2 ]，总之 dubbogo 从底层到上层全部都自成体系。目前 dubbogo 除了对齐所有 dubbo 版本，目前与 dubbo 齐头并进。

dubbogo 还有第二个使命：充分利用作为云原生时代第一语言 ---Go 语言的优势，扩展 dubbo 的能力。dubbogo 最早实现了把 k8s 作为注册中心的微服务治理方案，目前的 dubbo 的 k8s 注册中心解决方案即脱胎于此。

dubbogo 第三个使命是互联互通。dubbogo 自身的宗旨是 “bridging the gap between Java and Go"，除了与 dubbo 保持兼容外，目前也已经与 gRPC  和 兼容 REST 协议的框架如 Spring Cloud Alibaba 打通。

近期社区主要精力在于构建 dubbogo 生态矩阵，其中的关键就是最新迁移到 apache 的网关项目 https://github.com/apache/dubbo-go-pixiu，它将逐步打通 dubbo/SpringCloud/gRPC/RocketMQ，初步验证个人对云原生未来形态的设想【 https://mp.weixin.qq.com/s/EWR6xwbf53XHZ8O3m2bdVA 】。

**3. 在技术方案落地的过程中，你通常关注哪些问题？如何保证技术方案顺利实施？**

技术方案的落地，个人主要关注其价值，从技术价值和客户价值出发考虑其收益、人力资源成本、技术风险等问题。一旦目标确定，接下来要做的就是目标的拆解、寻找靠谱的工程师以及工程进度的实时跟踪，一旦面临项目进度或者未预料到的技术风险，首先要及时暴露给利益方相关人，保证能在可控时间范围内让问题收敛。

如果无法收敛，在大公司就需要考虑把问题上升，向上管理，利用各种资源来推动问题被解决。如果在中小公司内，以当前的资源【包括人力资源】无法保证问题收敛，能做到的就是及时止损。

**4. 架构师在最近的技术变化的浪潮中，需要面对的挑战都有哪些？如何应对这些挑战？**

个人在当前公司两年多一直是一线搬砖的工程师，自感没有资格以架构师的高度谈论这个话题。

不过在以前公司工作时 title 是架构师，如果把话题范围不局限在 “最近的技术变化的浪潮” 内，我倒是可以谈论下当时的面临的挑战以及应对之道。当时面临的挑战是公司的基础设施能力是零，当时的同事面临线上故障的第一反应是把最有可能出问题的服务全部重启一遍，如果还不能解决问题就把相关链路上所有的服务的版本回退并重启一遍。个人的应对方法就是基于个人的工作经验，借鉴以往工作中成熟技术方案，带着我组内的几个同事把公司的轻量级实时监控系统、日志收集系统、即时通信系统以及 KV 存储系统全部从零实现出来。

诚所谓功不唐捐，个人工作前五年一直跟着各路师傅搬砖，同时也积极参与各种开源项目，算是吃过一些猪肉也见过很多猪的走路姿势，积攒的经验在当时这些系统实施过程中很好地保证了这些系统快速高效的实施出来。

**5. 在做技术选型的过程中，你经常考虑的问题有些？**

以个人的愚见，技术人做技术选型一定不能只站在技术角度看待问题，一定要从价值角度或者说是老板的角度看待问题。老板的角度，个人感觉大概有三点：快速实施、系统稳定和成本最低。

所谓快速实施，就是在相关技术能在最短时间内把问题搞定，完成业务目标。系统稳定，首先要保证系统的稳定可靠运行，其次要保证架构三年【起码一年】不用翻来覆去不断地推到重来，必要的打补丁可以接受。做到前两点的前提下，尽可能地让人力资源、机器成本、维护成本等各种成本保持最低，让业务挣到钱，起码不要亏钱。

**6. 云原生领域你看好哪个项目或技术，为什么？**

云原生领域的具体项目个人首先看好 dubbogo，因为我个人在项目上已经坚持到了第六个年头，初心和热情仍在。

就云原生技术方向而言，这个话题太宽泛，个人积累甚少，还需要很长时间的学习和积累，自觉无资格评点。稍微能说的是，个人预感以开源服务平台为基础的跨云【如阿里云、腾讯云、华为云】多 k8s 集群管理是个很好的方向，如跨供应商进行服务部署。例如，https://github.com/gardener 这个项目，sap 和 pingcap 都在使用，是一个 k8s 多集群管理者，支持 aliyun/aws/openstack 等，同时能让一个物理机被多个 k8s 集群管理，有命令行也有 web 界面，这是妥妥的未来的混合云 多 k8s 集群的范例，可以使用 gardener 做多云平台管理，然后依赖 dapr 或者类似产品【如蚂蚁最近开源的 Layotto 以及 未来的 dubbo-go-pixiu】做公共服务部署。总之，个人感觉这块未来市场空间很大，可想象的地方很多。

**8. 请介绍下你这次在 giac 演讲的议题或者负责的专题内容**

个人本次演讲的内容主要围绕 dubbogo 3.0 展开。除了大家通过公开渠道已经知道的 dubbogo 3.0 的应用注册服务发现、新的路由方案以及新的注册通信协议外，还会着重讲解下官方对 dubbogo 3.0 未来的一些规划：dubbogo 微服务平台在云原生时代的容量与资源消耗评估、自适应限流与自适应负载均衡、Proxyless Service Mesh 等。

当然，我还会补充一些 dubbogo 在阿里内部的落地形态。总之，让一个有着 14 年历史的微服务治理框架在云原生时代与时俱进成为一个微服务治理平台， 是致敬前辈开拓者和当下用户信任的最佳方式，也为国内其他开源项目树立一个持续演进长期维护的标杆。

**9. 对本次 giac 有什么寄语**

这是 dubbogo 社区第二次派人参加 giac 大会了，希望 dubbogo 和 giac 都在各自的领域发展地越来越好。更希望通过 giac 这个平台，让更多人认识 dubbogo 这个强大的微服务基础平台，促进国内微服务技术的进步，创造出更多的社会价值与技术红利。

**参考阅读**  

*   [3-11 倍性能提升, 从 BTree 到 Polar Index](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653555932&idx=1&sn=fb9366ca0a628c88d3dc44351be0d051&chksm=81399d44b64e145235034bdf7bab907cfaaf13eeda7e6c4fa8be9044aaa215f691b06d02e76e&scene=21#wechat_redirect)
    
*   [Kvrocks: 一款开源的企业级磁盘 KV 存储服务](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653555924&idx=1&sn=e018ed7e8dabc2a99bcdc322f9598211&chksm=81399d4cb64e145aa758742083ef7d4180e86d14cd6802875a0bd7f64f4ccec831e5419fd0a9&scene=21#wechat_redirect)
    
*   [异地多活 paxos 实现：Multi-Master-Paxos-3](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653555895&idx=1&sn=6e1237d48487b36adebecde28704db61&chksm=81399d2fb64e14396d52aa71cd13a7e6b7bf02de218508905f61bc7d6234443fa2a081803baf&scene=21#wechat_redirect)  
    
*   [百度大规模 Service Mesh 落地实践](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653555866&idx=1&sn=8a8734e8d1d9b4093ee38206105189b7&chksm=81399d02b64e1414a83c3673f72be7e268e13f518bd5e82db3c5c870c0e9e2cf10fb7f69910f&scene=21#wechat_redirect)  
    
*   [春节红包活动如何应对 10 亿级流量？看大佬复盘总结](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653555824&idx=1&sn=48b13160ac74e851e654c95a9f9d8bfc&chksm=81399de8b64e14fe1bce230c88f0624bb57c6f2dbd1e43b60216526e49bbff9b21615f7de14d&scene=21#wechat_redirect)
    
*   [百度 C++ 工程师如何实现极致并发优化](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653555974&idx=1&sn=e7d33fa87bb6d6030b8af0bccd78b986&chksm=81399c9eb64e1588a50ed9c22cf2d8c05b967f30f57088a81f5cf8a7f88353f1d5e7118fdfda&scene=21#wechat_redirect)
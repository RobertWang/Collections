> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI5OTI4OTY5NQ==&mid=2247495791&idx=3&sn=af2599ca915c6693223911ae643aeaa3&chksm=ec9a6d5fdbede449cd3aed1561283d70a262fda992f343b916f48ac110a198a1cdff62ae4e21&mpshare=1&scene=1&srcid=0713ljimgwdlIETgVIfi1hfE&sharer_sharetime=1626149588243&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

*   1、为什么选择 Go 语言
    
*   2、Go 语言能做什么
    
*   3、国内外有哪些企业或项目使用 Go 语言
    
*   4、写在最后
    

* * *

1、为什么选择 Go 语言
=============

选择 Go 语言的原因可能会有很多，关于 Go 语言的特性、优势等，我们在之前的文档中也已经介绍了很多了。但是最主要的原因，应该是基于以下两方面的考虑：

**执行性能**

缩短 API 的响应时长，解决批量请求访问超时的问题。在 Uwork 的业务场景下，一次 API 批量请求，往往会涉及对另外接口服务的多次调用，而在之前的 PHP 实现模式下，要做到并行调用是非常困难的，串行处理却不能从根本上提高处理性能。而 GO 语言不一样，通过协程可以方便的实现 API 的并行处理，达到处理效率的最大化。依赖 Golang 的高性能 HTTP Server，提升系统吞吐能力，由 PHP 的数百级别提升到数千里甚至过万级别。

**开发效率**

GO 语言使用起来简单、代码描述效率高、编码规范统一、上手快。通过少量的代码，即可实现框架的标准化，并以统一的规范快速构建 API 业务逻辑。能快速的构建各种通用组件和公共类库，进一步提升开发效率，实现特定场景下的功能量产。

2、Go 语言能做什么
===========

Go 语言从发布 1.0 版本以来备受众多开发者关注并得到广泛使用，Go 语言的简单、高效、并发特性吸引了众多传统语言开发者的加入，而且人数越来越多。

鉴于 Go 语言的特点和设计的初衷，Go 语言作为服务器编程语言，很适合处理日志、数据打包、虚拟机处理、文件系统、分布式系统、数据库代理等；网络编程方面，Go 语言广泛应用于 Web 应用、API 应用、下载应用等；除此之外，Go 语言还适用于内存数据库和云平台领域，目前国外很多云平台都是采用 Go 开发。

服务器编程，以前你如果使用 C 或者 C++ 做的那些事情，用 Go 来做很合适，例如处理日志、数据打包、虚拟机处理、文件系统等。

分布式系统、数据库代理器、中间件等，例如 Etcd。

网络编程，这一块目前应用最广，包括 Web 应用、API 应用、下载应用，而且 Go 内置的 net/http 包基本上把我们平常用到的网络功能都实现了。

**数据库操作**

开发云平台，目前国外很多云平台在采用 Go 开发

3、国内外有哪些企业或项目使用 Go 语言
=====================

Go 发布之后，很多公司特别是云计算公司开始用 Go 重构他们的基础架构，很多都是直接采用 Go 进行了开发，最近热火朝天的 Docker 就是采用 Go 开发的。

![](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZfct3Ar49cOe9WSDCQftpsG3DUqnxja5Oa5lI19xZzm8rAaAxhx8zgIpgmDeicufXquQCcfoNfXziakA/640?wx_fmt=jpeg)

使用 Go 语言开发的开源项目非常多。早期的 Go 语言开源项目只是通过 Go 语言与传统项目进行 C 语言库绑定实现，例如 Qt、Sqlite 等；后期的很多项目都使用 Go 语言进行重新原生实现，这个过程相对于其他语言要简单一些，这也促成了大量使用 Go 语言原生开发项目的出现。

**云计算基础设施领域**代表项目：docker、kubernetes、etcd、consul、cloudflare CDN、七牛云存储等。

**基础软件**代表项目：tidb、influxdb、cockroachdb 等。

**微服务**代表项目：go-kit、micro、monzo bank 的 typhon、bilibili 等。

**互联网基础设施**代表项目：以太坊、hyperledger 等。采用 Go 的一些国外公司，如 Google、Docker、Apple、Cloud Foundry、CloudFlare、Couchbase、CoreOS、Dropbox、MongoDB、AWS 等公司；采用 Go 开发的国内企业：如阿里云 CDN、百度、小米、七牛、PingCAP、华为、金山软件、猎豹移动、饿了么等公司。

**Docker**Docker 是一种操作系统层面的虚拟化技术，可以在操作系统和应用程序之间进行隔离，也可以称之为容器。Docker 可以在一台物理服务器上快速运行一个或多个实例。基于 lxc 的一个虚拟打包工具，能够实现 PAAS 平台的组建。例如，启动一个 CentOS 操作系统，并在其内部命令行执行指令后结束，整个过程就像自己在操作系统一样高效。

项目链接：https://github.com/docker/docker

**go 语言** Go 语言自己的早期源码使用 C 语言和汇编语言写成。从 Go 1.5 版本后，完全使用 Go 语言自身进行编写。Go 语言的源码对了解 Go 语言的底层调度有极大的参考意义，建议希望对 Go 语言有深入了解的读者读一读。

项目链接：https://github.com/golang/go

**Kubernetes**Google 公司开发的构建于 Docker 之上的容器调度服务，用户可以通过 Kubernetes 集群进行云端容器集群管理。

项目链接：https://github.com/kubernetes/kubernetes

**etcd** 一款分布式、可靠的 KV 存储系统，可以快速进行云配置。

项目链接：https://github.com/coreos/etcd

**beego**beego 是一个类似 Python 的 Tornado 框架，采用了 RESTFul 的设计思路，使用 Go 语言编写的一个极轻量级、高可伸缩性和高性能的 Web 应用框架。

项目链接：https://github.com/astaxie/beego

**martini** 一款快速构建模块化的 Web 应用的 Web 框架。项目链接：https://github.com/go-martini/martini

**codis** 国产的优秀分布式 Redis 解决方案。项目链接：https://github.com/CodisLabs/codis

**Facebook**Facebook 也在用，为此他们还专门在 Github 上建立了一个开源组织 facebookgo，大家可以通过 https://github.com/facebookgo 访问查看 facebook 开源的项目，比如著名的是平滑升级的 grace。

**腾讯**腾讯作为国内的大公司，还是敢于尝试的，尤其是 Docker 容器化这一块，他们在 15 年已经做了 docker 万台规模的实践，具体可以参考 http://www.infoq.com/cn/articles/tencent-millions-scale-docker-application-practice 。

**百度**目前所知的百度的使用是在运维这边，是百度运维的一个 BFE 项目，负责前端流量的接入。他们的负责人在 2016 年有分享，大家可以看下这个 http://www.infoq.com/cn/presentations/application-of-golang-in-baidu-frontend 。

其次就是百度的消息系统。负责公司手百消息通讯系统服务器端开发及维护。

**京东**京东云消息推送系统、云存储，以及京东商城等都有使用 Go 做开发。

**小米**小米对 Golang 的支持，莫过于运维监控系统的开源，也就是 http://open-falcon.com/ 。此外，小米互娱、小米商城、小米视频、小米生态链等团队都在使用 Golang。

**360**360 对 Golang 的使用也不少，一个是开源的日志搜索系统 Poseidon，托管在 Github 上，https://github.com/Qihoo360/poseidon. 还有 360 的推送团队也在使用，他们还写了篇博文在 Golang 的官方博客上 https://blog.golang.org/qihoo。

**七牛云**七牛云用了近 50 万行代码，来实现整个产品。七牛云存储产品网址：http://qiniu.com/。上线时间：2011-9-1。应用范围：整个产品（包括基础服务、Web 端、统计平台、各类小工具等等）Go 代码行数占比：99.9% 日 PV：保密

**美团**美团后台流量支撑程序。应用范围：支撑主站后台流量（排序，推荐，搜索等），提供负载均衡，cache，容错，按条件分流，统计运行指标（qps，latency）等功能。

**滴滴**基础服务平台。

**金山微看**应用范围：服务接口，后台流程服务，消息系统，图片系统

**weico** 服务端所有代码都是用 Go 实现。

**仙侠道**应用范围：游戏服务端（通讯、逻辑、数据存储）

**快玩游戏**应用范围：实时消息系统、用户认证、用户会话、统一统计接口

**盛大云 CDN**CDN 的调度系统、分发系统、监控系统、短域名服务，CDN 内部开放平台、运营报表系统以及其他一些小工具等

**Bmob 移动后端云服务平台**应用范围：Restful API(使用 Beego)、统计分析平台、常用服务如发邮件、队列异步处理、统计用户空间和接口请求

**群策**统一团队沟通，高效完成工作 应用范围：全系统

**BiddingX** DSP 广告投放系统 应用范围：竞价投放、曝光统计、点击跳转

**宅豆**自筑最美家，宅豆随你搭

**实验楼**第一家以实验为核心的 IT 在线教育平台

**新浪微博**中间件和弹性调度用 Java 和 Go 编写，微博视频转码及存储服务用 Go 编写。

**爱奇艺** VR 后台系统中间件，VR 端的 HTTP 接口。

**网易**网易蜂巢容器公有云。

**哔哩哔哩**弹幕

**巨人网络**部分手机游戏的服务端。

**今日头条** Nsq：Nsq 是由 Go 语言开发的高性能、高可用消息队列系统，性能非常高，每天能处理数十亿条的消息；Packer: 用来生成不同平台的镜像文件，例如 VM、vbox、AWS 等，作者是 vagrant 的作者 Skynet：分布式调度框架 Doozer：分布式同步工具，类似 ZooKeeper Heka：mazila 开源的日志处理系统 Cbfs：couchbase 开源的分布式文件系统 Tsuru：开源的 PAAS 平台，和 SAE 实现的功能一模一样 Groupcache：memcahe 作者写的用于 Google 下载系统的缓存系统 God：类似 redis 的缓存系统，但是支持分布式和扩展性 Gor：网络流量抓包和重放工具

还有很多，比如阿里中间件、聚美优品、高升控股、探探、斗鱼直播、人人车、亚信、Udesk、方付通、招财猫、三一集团、美餐网等。一般的选择，都是选择用于自己公司合适的产品系统来做，比如消息推送的、监控的、容器的等，Golang 特别适合做网络并发的服务，这是他的强项，所以也是被优先用于这些项目。Go 语言作为一门大型项目开发语言，在很多大公司相继使用，甚至完全转向 Go 开发。

4、写在最后
======

当然，一个技术能不能发展起来，关键还要看三点。

** 有没有一个比较好的社区。** 像 C、C++、Java、Python 和 JavaScript 的生态圈都是非常丰富和火爆的。尤其是有很多商业机构参与的社区那就更为人气爆棚了，比如 Linux 的社区。

** 有没有一个工业化的标准。** 像 C、C++、Java 都是有标准化组织的。尤其是 Java，其在架构上还搞出了像 J2EE 这样的企业级标准。

** 有没有一个或多个杀手级应用。**C、C++ 和 Java 的杀手级应用不用多说了，就算是对于 PHP 这样还不能算是一个好的编程语言来说，因为是 Linux 时代的第一个杀手级解决方案 LAMP 中的关键技术，所以，也发展起来了。

上述的这三点是非常关键的，新的技术只需要占到其中一到两点就已经很不错了，何况有的技术，比如 Java，是三点全占到了，所以，Java 的发展是如此好。当然，除了上面这三点重要的，还有一些其它的影响因素，比如：

** 学习曲线是否低，上手是否快。** 这点非常重要，C++ 在这点上越做越不好了。** 有没有一个不错的提高开发效率的开发框架。** 如：Java 的 Spring 框架，C++ 的 STL 等。** 是否有一个或多个巨型的技术公司作为后盾。** 如：Java 和 Linux 后面的 IBM、Sun……** 有没有解决软件开发中的痛点。** 如：Java 解决了 C 和 C++ 的内存管理问题。

用这些标尺来量一下 Go 语言，我们可以清楚地看到：

*   Go 语言容易上手；
    
*   Go 语言解决了并发编程和写底层应用开发效率的痛点；
    
*   Go 语言有 Google 这个世界一流的技术公司在后面；
    
*   Go 语言的杀手级应用是 Docker，而 Docker 的生态圈在这几年完全爆棚了。
    

所以，Go 语言的未来是不可限量的。当然，我个人觉得，Go 可能会吞食很多 C、C++、Java 的项目。不过，Go 语言所吞食主要的项目应该是中间层的项目，既不是非常底层也不会是业务层。

也就是说，Go 语言不会吞食底层到 C 和 C++ 那个级别的，也不会吞食到高层如 Java 业务层的项目。Go 语言能吞食的一定是 PaaS 上的项目，比如一些消息缓存中间件、服务发现、服务代理、控制系统、Agent、日志收集等等，没有复杂的业务场景，也到不了特别底层（如操作系统）的中间平台层的软件项目或工具。而 C 和 C++ 会被打到更底层，Java 会被打到更上层的业务层。

好了，我们再用上面的标尺来量一下 Go 语言的杀手级应用 Docker，你会发现基本是一样的。

*   Docker 上手很容易。
    
*   Docker 解决了运维中的环境问题以及服务调度的痛点。
    
*   Docker 的生态圈中有大公司在后面助力。比如 Google。
    
*   Docker 产出了工业界标准 OCI。
    
*   Docker 的社区和生态圈已经出现像 Java 和 Linux 那样的态势。……
    

所以，虽然几年前的 Docker ，当时的坑儿还很多，但是，相对于这些大的因素来说，那些小坑儿都不是问题。只是需要一些时间，这些小坑儿在未来 5-10 年就可以完全被填平了。

![](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZfct3Ar49cOe9WSDCQftpsG3KLY9zTeDhk2AunhT74MCbWwKp3ziaaGFViaGbSuNp70icUGjGl274VYVQ/640?wx_fmt=jpeg)

同样，我们可以看到 Kubernetes 作为服务和容器调度的关键技术一定会是最后的赢家。

最后，我还要说一下，为什么要早一点地进入这些新技术，而不是等待这些技术成熟了后再进入。原因有这么几个。

技术的发展过程非常重要。因为你可以清楚地看到了这种新技术的生态圈发展过程。让我们收获最大的并不是这些技术本身，而是一个技术的变迁和行业的发展。

从中，我们看到了非常具体的各种思潮和思路，这些东西比起 技术本身来说更有价值。因为，这不但让我们重新思考已经掌握的技术以及如何更好地解决已有的问题，而且还让我看到了未来。不但有了技术优势，而且这些知识还让我们的技术生涯多了很多的可能性。

这些关键新技术，可以让你拿到技术的先机。这些对一个需要技术领导力的个人或公司来说都是非常重要的。

一个公司或是个人能够占有技术先机，就会比其它公司或个人有更大的影响力。一旦未来行业需求引爆，那么这个公司或是个人的影响力就会形成一个比较大的护城河，并可以快速地产生经济利益。

Go 的应用范围一直在扩大，云计算，微服务，区块链，哪里都有用 Go 写的重量级项目。docker/kubernetes 生态圈，几百 / 千万行代码，基本统治了云原生应用市场。去年大热的区块链，以太坊的 geth，比特币的 btcd，闪电网络的 lnd，都是 Go 语言开发。还是那句话，多看看各种语言的生态，或许都并没有你想象的那么不堪。。。Go 语言设计上确实不够 “先进”，但也是另一种 “务实”。其实 go 不管在国内还是国外已经很受待见了，国外 google 用的很多，uber 也在用，国内有著名的今日头条，每日千亿级的访问妥妥的。多少语言终其一生都没有这么大的应用场景。
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA3OTc0MzY1Mg==&mid=2247515151&idx=3&sn=4023908bb0821d026abd08323114d974&chksm=9fac23c4a8dbaad2a8d699d23e86d738ec6b44608b24d631478f7f19cd95c265554c84cb353a&mpshare=1&scene=1&srcid=0819eU0e5O2dg0lYUkcDqFhd&sharer_sharetime=1629345287511&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

**导读**  

  

本文以零售行业为例探究中台的建设。中台是一套先进的架构理念，通过持续提炼可复用的能力，达到快速响应客户需求的目的。但是中台不能简单地照搬照抄，需要结合企业的业务特点，沉淀一套自身的架构方法论。中台建设是手段不是目的，目的始终是快速响应需求取得成功。

01

中台建设基础

  

零售 SaaS 业务架构分为前台业务、中台业务、后台业务。业务架构会驱动应用架构的设计与建设，应用架构最上层是业务架构的内容，应用前台负责支撑整个业务架构，应用中台主要包括业务中台、数据中台，应用中台为应用前台提可复用的业务能力，帮助应用前台快速支撑业务发展，技术中台为应用前台、中台提供可复用的技术能力。

但是整个架构需要建立在一套标准的架构方法论之上，在建设中台的过程中，它为团队提供统一的原则和标准，帮助团队更好的理解整个架构体系，进而持续迭代和演进整体架构。没有这套方法论，中台建设只是空中楼阁，无法顺利落地。

![](https://mmbiz.qpic.cn/mmbiz_png/qvnwYKoABd6GzMF8EqGGPdMUTgDg2TiaSBwwicx3dPGqQozjRugoiaPLKhvtQCagZg8AwKvXelga9NhuJpze9maSQ/640?wx_fmt=png)

02

抽象分层标准化

  

搭建中台前会用到的一个重要工具是抽象分层。为什么要进行抽象分层？

*   人脑处理信息的能力有限。如果信息量超过了人脑的处理能力，那么人会失去对该事物的理解。零售业务的信息量非常庞大，没有人能够同一时间处理这么庞大的信息量，需要找到一种概括、抽象的方法去理解零售业务。在面对高度复杂的问题时，我们可以通过不断提升抽象层次来解决更为复杂的问题。
    
*   沟通协作的需要。团队需要基于统一的设计语言来进行沟通，否则在面的大量复杂问题时，沟通经常会不在一个频道上，出现理解偏差、反复沟通的情况。
    
*   让架构设计思路更加清晰有条理。不管是梳理已有业务，还是梳理创新型业务，如果能基于标准化的抽象层次梳理，整个过程将会更加有条理。
    
*   有利于领域知识的沉淀和传承。在复杂业务系统的迭代过程中，知识沉淀是尤为重要的，否则业务系统很快就变成一个 “大泥球” 系统，没有人理得清内部的业务逻辑和业务关系。然而知识沉淀的前提是要有一个知识框架，否则知识没有附着的地方，而标准化的抽象层次就是一个最基础的知识框架。
    

![](https://mmbiz.qpic.cn/mmbiz_png/qvnwYKoABd6GzMF8EqGGPdMUTgDg2TiaScZKb9Rvv11LzVRnIUjJN3N1INdYGIhdItUtrfWibn8IibibiaHeVQ4P5yg/640?wx_fmt=png)  

03

中台业务架构

  

ToB 产品和 ToC 产品有一个显著的区别是，ToB 产品面向的客户是企业，ToC 产品面向的客户是个人。企业内部有复杂的组织架构，会设立公司、部门、岗位。各个公司、部门、岗位会有不同的职责和权力，三者相互之间会产生错综复杂的关系。

所以在获取用户需求时，首先需要梳理清楚企业的组织架构，然后按照组织架构的脉络循序渐进地进行需求调研。在访谈不同层级的用户时，访谈的要点也要有所侧重。

  

![](https://mmbiz.qpic.cn/mmbiz_png/qvnwYKoABd6GzMF8EqGGPdMUTgDg2TiaSaYw1KicIia6TOS57ujwdYNgTJibkx72TSianljBkhVf0jTgBG7MYNgBlsQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/qvnwYKoABd6GzMF8EqGGPdMUTgDg2TiaShE4znicedeEu1m3PXeGiaaFKf78ZAxCMIicm27ETpXJoCcHJld727unpA/640?wx_fmt=png)

04

中台数据架构

  

数据架构分析，这里展示的是一部分财务中台业务域的领域模型图，它所处的抽象层次是业务子域层次，每个虚线框代表一个业务子域，业务子域之间有上下游关系，业务子域内部是领域模型以及模型之间的关系，包含一个个的聚合，每个聚合都有一个聚合根，其他实体都会依附于某一个聚合根存在。

领域模型设计是非常重要的架构设计工作，因为领域模型是每个业务域和业务子域的核心，后续要实现的各种服务能力，都是围绕领域模型来组织的。领域模型没有设计好，对业务发展的影响是非常长远的，一方面会影响团队对业务的理解，导致系统可维护性越来越差；另一方面如果要建设数据中台，也会让数据中台的建设困难重重。

  

![](https://mmbiz.qpic.cn/mmbiz_png/qvnwYKoABd6GzMF8EqGGPdMUTgDg2TiaSb16FLyQBUGZ8bOSBQ18qfKVPAbAibHu3vV0WOjfokmh7QB3XZdRdydw/640?wx_fmt=png)

05

中台应用架构

  

应用架构需要定义整个产品需要建设哪些业务系统，系统间如何集成，系统内是否需要划分应用容器，每个应用容器提供哪些服务能力，哪些服务需要下沉为领域服务，哪些服务直接为前端提供业务服务。

![](https://mmbiz.qpic.cn/mmbiz_png/qvnwYKoABd6GzMF8EqGGPdMUTgDg2TiaSH98X9v7iaibThFoaYGmpOmpFmc5T7G4eVibGWqU75qLFzCiacObT1bg0hQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/qvnwYKoABd6GzMF8EqGGPdMUTgDg2TiaSyMgkrCmgDPkFSBjgc3QYUPxKNqDbASavf9FQBvTl0P9pOkMaldicIBQ/640?wx_fmt=png)
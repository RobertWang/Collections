> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [paul.pub](https://paul.pub/dds-and-fastrtps/)

> DDS 与 FastRTPS, Automotive, Distribution,Realtime,Publisher,Subscribe,Topic, DDS 是一套通信协议和 API 标准，它提供了以数据为中心的连接服务。

DDS 是一套通信协议和 API 标准，它提供了以数据为中心的连接服务。Fast-RTPS 是 DDS 的开源实现，借助它可以方便的开发出高效，可靠的分布式系统。本文是对 DDS 以及 Fast RTPS 的介绍文章。

*   [从《变形金刚》电影说起](#id-从变形金刚电影说起)
*   [DDS 介绍](#id-dds介绍)
    *   [DDS 可以降低系统复杂度](#id-dds可以降低系统复杂度)
    *   [DDS 应用范围](#id-dds应用范围)
    *   [DDS 提供商](#id-dds-提供商)
    *   [DDS 与 RTPS](#id-dds与rtps)
*   [DDS 与汽车行业](#id-dds与汽车行业)
*   [Fast-RTPS 介绍](#id-fast-rtps-介绍)
*   [源码与编译](#id-源码与编译)
*   [Fast-RTPS 概述](#id-fast-rtps概述)
*   [对象与数据结构](#id-对象与数据结构)
    *   [Publish-Subscriber 模块](#id-publish-subscriber模块)
    *   [RTPS 模块](#id-rtps模块)
    *   [配置 Attributes](#id-配置attributes)
*   [Domain](#id-domain)
*   [发现](#id-发现)
*   [传输控制](#id-传输控制)
*   [FastRTPS 代码示例](#id-fastrtps代码示例)
*   [FASTRTPSGEN](#id-fastrtpsgen)
*   [发布者 - 订阅者层](#id-发布者-订阅者层)
*   [读者 - 写者层](#id-读者-写者层)
*   [持久化](#id-持久化)
*   [QoS](#id-qos)
*   [实操测试](#id-实操测试)
*   [参考资料与推荐读物](#id-参考资料与推荐读物)

![](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-07-dds-and-fastrtps/bg.jpg)

这里要提到的是 2011 年的真人版电影，变形金刚第三部《Transformers: Dark of the Moon》。

这是一篇技术文章，为什么要扯到《变形金刚》电影呢？这是因为这部电影的主要内容与本文所提到的技术有一定的相关性。

在这部电影中，御天敌背叛了擎天柱，与霸天虎合作。在地球的各地布置了许多的能量柱，他试图借助这些能量柱将赛博坦星球传送到地球上，以此来重建自己的家园。

这些能量柱必须组合起来才能完成传输工作，并且在这其中有一个红色的能量柱比较特殊，因为它负责控制其他的传送柱。

![](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-07-dds-and-fastrtps/t1.png)

由此可见，这是一个大型的分布式系统。在这个系统中，这个红色的能量柱被称之为 “中心节点”，中心节点正如其名称那样，它是整个系统的中心。对于带有中心节点的分布式系统来说，一旦中心节点被摧毁，整个系统都将无法工作。

因此电影的后来，自然是擎天柱摧毁了这个中心节点，使得御天敌的传送计划彻底失败。

![](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-07-dds-and-fastrtps/t2.png)

从设计上来说，对于一个如此大型的系统，却存在一个非常薄弱和重要的中心节点，这并不是一个好的方案。

而本文介绍的 DDS 就是一个去中心化的分布式技术。因此在这类系统中，不存在负责总控制的中心节点，所有节点都完全对等。任何一个节点的异常都不会影响整个系统的运行。

DDS 全称是 Data Distribution Service，这是一套通信协议和 API 标准，它提供了以数据为中心的连接服务，基于发布者 - 订阅者模型。这是一套中间件，它提供介于操作系统和应用程序之间的功能，使得组件之间可以互相通信。并且提供了低延迟，高可靠的通信以及可扩展的架构。

或许，你已经知道很多种网络通信协议，对于发布 - 订阅这些概念也很熟悉。那 DDS 到底有什么特别之处呢？

下图展示了 4 个时代的数据通信方式：

![](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-07-dds-and-fastrtps/gen.png)

*   （第一代）点对点的 CS（Client-Server）结构，这是大家最为熟悉的：一个服务器角色被许多的客户端使用，每次通信时，通信双方必须建立一条连接。当通信节点增多时，通信的连接数也会增多。并且，每个客户端都必须知道服务器的具体地址和所提供的服务。一旦服务器地址发生变化，所有客户端都会受到影响。
*   （第二代）Broker 模型：存在一个中间人，它负责初步处理大家的请求，并进一步找到真正能响应服务的角色，这就好像存在一个经纪人。这为客户端提供了一层抽象，使得服务器的具体地址变得不重要了。服务端地址如果发生变化，只需要告诉 Broker 就可以了。但这个模型的问题在于，Broker 变成了模型的中心，它的处理速度会影响所有人的效率，这就好像城市中心的路口，当系统规则增长到一定程度，Broker 终究会成为瓶颈。更糟糕的是，如果 Broker 瘫痪了，可能整个系统都将无法运转。
*   （第三代）广播模型：所有人都可以在通道上广播消息，并且所有人都可以收到消息。这个模型解决了服务器地址的问题，且通信双方不用单独建立连接，但它存在的问题是：广播通道上的消息太多，太嘈杂，所有人都必须关心每条消息是否与自己有关。这就好像全公司一千号人坐在同一个房间里面办公一样。
*   （第四代）DDS 模型：这种模型与广播模型有些类似，所有人都可以在 DataBus 上发布和读取消息。但它更进一步的是，通信中包含了很多并行的通路，每个人可以只关心自己感兴趣的消息，自动忽略自己不需要的消息。

下图展示了 DDS 在网络栈中的位置，它位于传输层的上面，并且以 TCP，UDP 为基础。

![](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-07-dds-and-fastrtps/stack.png)

> 这个图之所以是沙漏形状是因为：两头的技术变化都发展很快，但是中间的却鲜有变化。

对比大家常见的 Socker API，DDS 有如下特点：

<table><thead><tr><th>特性</th><th>Socket API</th><th>DDS</th></tr></thead><tbody><tr><td>架构</td><td>TCP：点对点<br>UDP：点对点，广播，多播</td><td>Publish-subscribe 模型</td></tr><tr><td>平台独立</td><td>需要为不同硬件，操作系统和编程语言编写不同的代码</td><td>所有硬件，操作系统和编程语言使用相同的 API</td></tr><tr><td>发现</td><td>需要硬编码 IP 地址和端口号</td><td>动态发现，无需关注端点所在位置</td></tr><tr><td>类型安全</td><td>没有类型安全，应用需要将字节流转换成正确类型</td><td>强类型安全，<code>write()</code>和<code>read()</code>针对特定数据类型</td></tr><tr><td>通信行为定制</td><td>需要通过自定义的代码来实现</td><td>通过 QoS 策略来完成</td></tr><tr><td>互操作性</td><td>不支持</td><td>具有公认的互操作性的开放标准</td></tr></tbody></table>

关于 DDS 的更多特性，可以点击这个链接：[《What is DDS？》](https://www.dds-foundation.org/what-is-dds-3/)。

DDS 可以降低系统复杂度
-------------

对于分布式系统来说，有很多复杂的逻辑需要处理，例如：如何发现其他节点，如何为每个节点分配地址，如何配置消息的可靠性等。这使得应用程序变得臃肿。

而如果说通信的中间件能够完全处理好这些逻辑，则应用程序将可以集中处理自己的业务，变得更加敏捷。

下图是两种情况的对比：

![](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-07-dds-and-fastrtps/complex.png)

如果考虑系统的演化，问题会更突出。

由于分布式系统中包含了许多的角色需要互相通信，随着角色数量的不断增长，其通信的通道数量会以爆炸式增长。

![](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-07-dds-and-fastrtps/traditioan.png)

而如果有统一的 DataBus，则即便新增了通信角色其通信模型也不会变得更加复杂。

![](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-07-dds-and-fastrtps/dds_arch.png)

DDS 应用范围
--------

尽管可能你原先没有听过 DDS 这个术语，但其实它的应用非常广泛，广泛到它涉及到了我们每天都要依赖的许多重要行业，例如：航空，国防，交通，医疗，能源等等。

下图是一些示例：

![](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-07-dds-and-fastrtps/application.png)

DDS 提供商
-------

DDS 本身是一套标准。由 [Object Management Group](https://www.omg.org/)（简称 OMG）维护。

OMG 是一个开放性的非营利技术标准联盟，由许多大型 IT 公司组成：包括 IBM，Apple Computer，Sun Microsystems 等。

但 OMG 仅仅负责制定标准，而标准的实现则由其他服务提供商完成。

目前 DDS 的提供商包括下面这些：

*   [Vortex OpenSplice](https://www.adlinktech.com/en/vortex-opensplice-data-distribution-service.aspx)
*   [eProsima Fast RTPS](http://www.eprosima.com/)
*   [Hamersham](https://hamersham.com/)
*   [Company Summary Kongsberg Gallium](http://www.kongsberggallium.com/)
*   [MilSOFT](http://dds.milsoft.com.tr/en/dds-home.php)
*   [Object Computing OpenDDS](https://objectcomputing.com/products/opendds)
*   [Remedy IT](http://www.remedy.nl/)
*   [RTI](http://www.rti.com/)
*   [Twin Oaks Computing, Inc.](http://www.twinoakscomputing.com/)

DDS 与 RTPS
----------

在 DDS 规范中，有两个描述标准的基本文档：

*   [DDS Specification](https://www.omg.org/spec/DDS/)：描述了以数据为中心的发布 - 订阅模型。该规范定义了 API 和通信语义（行为和服务质量），使消息从消息生产者有效地传递到匹配的消费者。DDS 规范的目的可以概括为：“能够在正确的时间将正确的信息高效，可靠地传递到正确的位置”。
*   [DDSI-RTPS](https://www.omg.org/spec/DDSI-RTPS/2.2/)：描述了 RTPS（Real Time Publish Subscribe Protocol）协议。该协议通过 UDP 等不可靠的传输，实现最大努力（Best-Effort）和可靠的发布 - 订阅通信。RTPS 是 DDS 实现的标准协议，它的目的和范围是确保基于不同 DDS 供应商的应用程序可以实现互操作。

![](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-07-dds-and-fastrtps/spec.png)

对于汽车行业来说，汽车开放系统架构（AUTomotive Open System ARchitecture）已经在 [AUTOSAR Adaptive Platform 18.03 中包含了 DDS 协议](https://www.rti.com/blog/autosar-adaptive-platform-18.03-now-with-dds)。

[ROS2 架构也以 DDS 为基础](https://design.ros2.org/articles/ros_on_dds.html)。

另外，DDS 的实时特性可能特别适合自动驾驶系统。在这类系统中，通常会存在感知，预测，决策和定位模块，这些需要非常高速和频繁的交换数据。借助 DDS，可以很好的满足它们的通信需求。

![](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-07-dds-and-fastrtps/autonomous.png)

[Fast-RTPS](https://fast-rtps.docs.eprosima.com/en/latest/) 是 eprosima 对于 RTPS 的 C++ 实现，这是一个免费开源软件，遵循 Apache License 2.0。

eProsima Fast RTPS 在性能，功能和对最新版本 RTPS 标准（RTPS 2.2）的遵守方面均处于领先地位。关于 Fast RTPS 的性能可以查看这个链接：[eProsima Fast RTPS Performance](https://www.eprosima.com/index.php/resources-all/performance/40-eprosima-fast-rtps-performance)。

它最为被大家知道的可能是因为被 [ROS2](https://www.ros.org/) 设定为默认的消息中间件。

Fast-RTPS 支持平台包括：Windows, Linux, Mac OS, QNX, VxWorks, iOS, Android, Raspbian。

Fast-RTPS 具有以下优点：

*   对于实时应用程序来说，可以在 Best-Effort 和可靠通信两种策略上进行配置。
*   即插即用的连接性，使得网络的所有成员自动发现其他新的成员。
*   模块化和可扩展性允许网络中设备不断增长。
*   可配置的网络行为和可互换的传输层：为每个部署选择最佳协议和系统输入 / 输出通道组合。
*   两个 API 层：一个简单易用的发布者 - 订阅者层和一个提供对 RTPS 协议内部更好控制的 Writer-Reader 层。

Fast-RTPS 的源码位于 Github 上：[eProsima/Fast-RTPS](https://github.com/eProsima/Fast-RTPS)。

可以通过下面这条命令获取其源码：

```
git clone https://github.com/eProsima/Fast-RTPS.git


```

关于如何编译 Fast-RTPS 可以参见这个链接：[Fast RTPS Installation from Sources](https://fast-rtps.docs.eprosima.com/en/latest/sources.html)。

Fast-RTPS 提供了两个层次的 API：

*   Publisher-Subscriber 层：RTPS 上的简化抽象。
*   Writer-Reader 层：对于 RTPS 端点的直接控制。

相较而言，后者更底层。两个层次的核心角色如下图所示：

![](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-07-dds-and-fastrtps/architecture.png)

Publisher-Subscriber 层为大多数开发者提供了一个方便的抽象。它允许定义与 Topic 关联的发布者和订阅者，以及传输 Topic 数据的简单方法。

Writer-Reader 层更接近于 RTPS 标准中定义的概念，并且可以进行更精细的控制，但是要求开发者直接与每个端点的历史记录缓存进行交互。

Fast RTPS 是并发且基于事件的。每个参与者都会生成一组线程来处理后台任务，例如日志记录，消息接收和异步通信。

事件系统使得 Fast RTPS 能够响应某些条件并安排定期活动。用户中几乎不用感知它们，因为这些事件大多数仅仅与 RTPS 元数据有关。

下面是 Fast-RTPS 实现中的核心结构。

Publish-Subscriber 模块
---------------------

RTPS 标准的高层类型。

*   Domain：用来创建，管理和销毁 Participants。
*   Participant：包括 Publisher 和 Subscriber，并管理它们的配置。
    *   ParticipantAttributes：创建 Participant 的配置参数。
    *   ParticipantListener：可以让开发者实现 Participant 的回调函数。
*   Publisher：在 Topic 上发布数据的对象。
    *   PublisherAttributes：创建 Publisher 的配置参数。
    *   PublisherListener：可以让开发者实现 Publisher 的回调函数。
*   Subscriber：在 Topic 上接受数据的对象。
    *   SubscriberAttributes：创建 Subscriber 的配置参数。
    *   SubscriberListener：可以让开发者实现 Subscriber 的回调函数。

RTPS 模块
-------

RTPS 的底层模型。包含下面几个子模块：

*   RTPS Common
    *   CacheChange_t：描述 Topic 上的变更，存储在历史 Cache 中。
    *   Data：Cache 变化的负载。
    *   Message：RTPS 消息。
    *   Header：RTPS 协议的头信息。
    *   Sub-Message Header：标识 RTPS 的订阅消息。
    *   MessageReceiver：反序列化和处理接受到的 RTPS 消息。
    *   RTPSMessageCreator：构建 RTPS 消息。
*   RTPS Domain
    *   RTPSDomain：用来创建，管理和销毁底层的 RTPSParticipants。
    *   RTPSParticipant：包括 Writer 和 Reader。
*   RTPS Reader
    *   RTPSReader：读者的基类。
    *   ReaderAttributes：包含 RTPS 读者的配置参数。
    *   ReaderHistory：存储 Topic 变化的历史数据。
    *   ReaderListener：读者的回调类型。
*   RTPS Writer
    *   RTPSWriter：写者的基类。
    *   WriterAttributes：包含 RTPS 写者的配置参数。
    *   WriterHistory：存储写者的历史数据。

配置 Attributes
-------------

上面的数据结构中看到了许多`Attributes`后缀的类名。这些类包含了对协议或者对象的配置参数，很多特性都需要设置这些属性来完成。

这些类的定义基本都位于下面三个文件夹中：

*   [include/fastrtps/attributes](https://github.com/eProsima/Fast-RTPS/tree/master/include/fastrtps/attributes)
*   [include/fastrtps/rtps/attributes](https://github.com/eProsima/Fast-RTPS/tree/master/include/fastrtps/rtps/attributes)
*   [include/fastdds/rtps/attributes](https://github.com/eProsima/Fast-RTPS/tree/master/include/fastdds/rtps/attributes)

Fast RTPS 支持非常多的配置参数，并且参数的结构常常是嵌套的。

通过代码去配置这些参数会产生很多啰嗦的代码，而且最大的问题在于：每次更改配置参数都需要重新编译。这个问题并非 Fast RTPS 才有，只要包含大量配置参数的软件都会这样的问题。通常的解决方法就是：提供文本格式的配置文件的方式来配置参数。因此对于 Fast-RTPS 来说，除了支持通过代码配置参数，它也支持通过 XML 文件的方式来进行配置。

有了配置文件之后，在代码中直接读取就好了，例如：

```
Participant *participant = Domain::createParticipant("participant_xml_profile");


```

在这之后，如果需要调整配置，只需要修改配置文件，不用在改动代码，自然也不用重新编译。这对于项目部署是很重要的。

Fast-RTPS 支持的配置项，以及这些配置项说明和默认值都可以到这个链接中查看：[XML profiles](https://fast-rtps.docs.eprosima.com/en/latest/xmlprofiles.html)。

RTPS 中的通信参数者之间，通过 Domain 进行隔离。

同一时刻可能有多个 Domain 同时存在，一个 Domain 中可以包含任意数目的消息发送者和接受者。

其结构如下图所示：

![](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-07-dds-and-fastrtps/RTPS-structure.png)

开发者可以通过 [domainId](https://fast-rtps.docs.eprosima.com/en/latest/xmlprofiles.html#built-in-parameters) 来指定参与者所属 Domain。

如果没有指定，默认的`domainId = 80`。

作为 DDS 的实现，Fast-RTPS 提供了 Publisher 和 Subscriber 自动发现和匹配的功能。在实现上，这分为两个步骤来完成：

*   Participant Discovery Phase （PDP）：在这个阶段，参与者互相通知彼此的存在。为了达到这个目的，每个参与者需要定时发送公告消息。公告消息通过周知的多播地址和端口发送（根据 domain 计算得到）。
*   Endpoint Discovery Phase （EDP）：在这个阶段，Publisher 和 Subscriber 互相确认。为此，参与者使用在 PDP 期间建立的通信通道，彼此共享有关其发布者和订阅者的信息。 该信息包含了 Topic 和数据类型。为了使两个端点匹配，它们的 Topic 和数据类型必须一致。 一旦发布者和订阅者匹配，他们就发送 / 接收数据了。

这两个阶段对应了两个独立的协议：

*   Simple Participant Discovery Protocol：指定参与者如何在网络中发现彼此。
*   Simple Endpoint Discovery Protocol：定义了已经互相发现的参与者交换信息的协议。

Fast-RTPS 提供了四种发现机制：

*   Simple：这是默认机制。它在 PDP 和 EDP 阶段均使用 RTPS 标准，因此可与任何其他 DDS 和 RTPS 实现兼容。
*   Static：此机制在 PDP 阶段使用 Simple Participant Discovery Protocol。如果所有发布者和订阅者的地址以及端口和主题信息是事先知道的，则允许跳过 EDP 阶段。
*   Server-Client：这种发现机制使用集中式发现结构，由服务器充当发现机制的 Hub。
*   Manual：此机制仅与 RTPSDomain 层兼容。它禁用了 PDP 阶段，使用户可以使用其选择的任何外部元信息通道手动匹配和取消匹配 RTPS 参与者，读者和写者。

不同的发现机制具有一些共同的配置：

<table><thead><tr><th>名称</th><th>描述</th><th>默认值</th></tr></thead><tbody><tr><td>Ignore Participant flags</td><td>在必要的时候，可以选择忽略一些参与者。<br>例如：另一台主机上的，另一个进程的或者同一个进程的。</td><td>NO_FILTER</td></tr><tr><td>Lease Duration</td><td>指定远程参与者在多少时间内认为本地参与者还活着。</td><td>20 s</td></tr><tr><td>Announcement Period</td><td>指定参与者的 PDP 公告消息的周期。</td><td>3s</td></tr></tbody></table>

关于发现机制的更多信息可以浏览这个链接：[Discovery](https://fast-rtps.docs.eprosima.com/en/latest/advanced.html#discovery)。

Fast-RTPS 实现了可插拔的传输架构，这意味着每一个参与者可以随时加入和退出。

在传输上，Fast-RTPS 支持以下五种传输方式：

*   UDPv4
*   UDPv6
*   TCPv4
*   TCPv6
*   SHM（Shared Memory）

默认的，当`Participant`创建时，会自动的配置两个传输通道：

*   SHM：用来与同一个机器上的参与者通信。
*   UDPv4：同来与跨机器的参与者通信。

> 当然，开发者可以改变这个默认行为，通过 C++ 接口或者 XML 配置文件都可以。

SHM 要求所有参与者位于同一个系统上，它是借助了操作系统提供的共享内存机制实现。共享内存的好处是：支持大数据传输，减少了数据拷贝，并且也减少系统负载。因此通常情况下，使用 SHM 会获得更好的性能。使用 SHM 时，可以配置共享内存的大小。

网络通信包含了非常多的参数需要配置，例如：Buffer 大小，端口号，超时时间等等。框架本身为参数设置了默认值，大部分情况下开发者不用调整它们。但是知道这些默认值是什么，在一些情况下可能会对分析问题有所帮助。关于这些配置的默认值，以及如果配置可以查看这个链接：[Transport descriptors](https://fast-rtps.docs.eprosima.com/en/latest/xmlprofiles.html#transport-descriptors)。

与 UDP 不同，TCP 传输是面向连接的，因此，Fast-RTPS 必须在发送 RTPS 消息之前建立 TCP 连接。TCP 传输可以具有两种行为：充当 TCP 服务器或充当 TCP 客户端。服务器打开一个 TCP 端口以侦听传入的连接，然后客户端尝试连接到服务器。服务器和客户端的概念独立于 RTPS 概念，例如：Publisher，Subscriber，Reader 或 Writer。它们中的任何一个都可以用作 TCP 服务器或 TCP 客户端，因为这些实体仅用于建立 TCP 连接，而 RTPS 协议可以在该 TCP 连接上工作。

如果要使用 TCP 传输，开发者需要做更多的配置，关于这部分内容可以继续阅读[官方文档](https://fast-rtps.docs.eprosima.com/en/latest/advanced.html#tcp-transport)，这里不再赘述。

FastRTPS 不仅有[框架文档](https://fast-rtps.docs.eprosima.com/en/latest/introduction.html)，[API Reference](http://www.eprosima.com/docs/fast-rtps/1.9.3/html/index.html)，还有丰富的代码示例。

对于开发者来说，浏览这些代码可能是上手最快捷的方法。

你可以在这里浏览这些示例：[Fast-RTPS/examples/C++/](https://github.com/eProsima/Fast-RTPS/tree/master/examples/C%2B%2B)。

FASTRTPSGEN 是一个 Java 程序。用来为在 Topic 上传输的数据类型生成源码。

开发者通过接口描述语言（Interface Definition Language）定义需要传输的数据类型。然后通过 FASTRTPSGEN 生成 C++ 编译需要的源文件。

可以通过下面的方法获取和编译 FASTRTPSGEN。

```
git clone --recursive https://github.com/eProsima/Fast-RTPS-Gen.git
cd Fast-RTPS-Gen
gradle assemble


```

编译完成之后可执行文件位于`./scripts/`目录。如果需要，可以将该路径添加到`$PATH`变量中。

关于如何通过 IDL 定义数据类型请参见这里：[Defining a data type via IDL](https://fast-rtps.docs.eprosima.com/en/latest/genuse.html#idl-types)。

以下面这个示例文件为例：

```
struct TestData 
{
char char_type;
octet octet_type;
long long_type;
string string_type;

float float_array[4];

sequence<double> double_list;
};


```

我们将其保存到文件名为`data_type.idl`的文件中。然后通过下面这条命令生成 C++ 文件：

```
~/Fast-RTPS-Gen/scripts/fastrtpsgen data_type.idl 


```

最后会得到下面四个文件：

```
data_type.cxx
data_type.h
data_typePubSubTypes.cxx
data_typePubSubTypes.h


```

前两个文件定义的是实际存储数据的结构，后两个文件定义的类是`eprosima::fastrtps::TopicDataType`的子类。用来在参与者上注册类型：

```
/**
 * Register a type in a participant.
 * @param part Pointer to the Participant.
 * @param type Pointer to the Type.
 * @return True if correctly registered.
 */
RTPS_DllAPI static bool registerType(
        Participant* part,
        fastdds::dds::TopicDataType * type);


```

每一套通信系统中通常都会包含一个或多个自定义的数据类型。

可以通过 [HelloWorldExample](https://github.com/eProsima/Fast-RTPS/tree/master/examples/C%2B%2B/HelloWorldExample) 来熟悉发布者 - 订阅者层接口。

该目录下文件列表如下：

```
-rw-r--r--  1 paul  staff   1.8K  3 16 13:36 CMakeLists.txt
-rw-r--r--  1 paul  staff   2.8K  3 16 13:36 HelloWorld.cxx
-rw-r--r--  1 paul  staff   6.1K  3 16 13:36 HelloWorld.h
-rw-r--r--  1 paul  staff    62B  3 16 13:36 HelloWorld.idl
-rw-r--r--  1 paul  staff   4.4K  3 16 13:36 HelloWorldPubSubTypes.cxx
-rw-r--r--  1 paul  staff   1.7K  3 16 13:36 HelloWorldPubSubTypes.h
-rw-r--r--  1 paul  staff   4.6K  3 16 13:36 HelloWorldPublisher.cpp
-rw-r--r--  1 paul  staff   1.7K  3 16 13:36 HelloWorldPublisher.h
-rw-r--r--  1 paul  staff   3.8K  3 16 13:36 HelloWorldSubscriber.cpp
-rw-r--r--  1 paul  staff   1.8K  3 16 13:36 HelloWorldSubscriber.h
-rw-r--r--  1 paul  staff   2.0K  3 16 13:36 HelloWorld_main.cpp
-rw-r--r--  1 paul  staff   3.1K  3 16 13:36 Makefile
-rw-r--r--  1 paul  staff   203B  3 16 13:36 README.txt


```

这其中：

*   README.txt 是工程说明
*   CMakeLists.txt 与 Makefile 是编译文件
*   HelloWorld_main.cpp 包含了生成可执行文件的`main`函数
*   HelloWorld.idl 是待传输的数据结构定义
*   HelloWorld.h，HelloWorld.cxx，HelloWorldPubSubTypes.h 和 HelloWorldPubSubTypes.cxx 是由 HelloWorld.idl 文件生成
*   HelloWorldPublisher.h 和 HelloWorldPublisher.cpp 是发布者的实现
*   HelloWorldSubscriber.h 和 HelloWorldSubscriber.cpp 是订阅者的实现

熟悉一个 C++ 工程可以先从`main`入手，HelloWorld_main.cpp 中的主要逻辑就是根据用户输入的参数是`"publisher"`还是`"subscriber"`来确定启动哪个模块。

```
switch(type)
{
    case 1:
        {
            HelloWorldPublisher mypub;
            if(mypub.init())
            {
                mypub.run(count, sleep);
            }
            break;
        }
    case 2:
        {
            HelloWorldSubscriber mysub;
            if(mysub.init())
            {
                mysub.run();
            }
            break;
        }
}


```

接下来我们直接看 HelloWorldPublisher 和 HelloWorldSubscriber 就好。

`HelloWorldPublisher::init`中主要是为 Publisher 的对象设置参数：

```
bool HelloWorldPublisher::init()
{
    m_Hello.index(0);
    m_Hello.message("HelloWorld");
    ParticipantAttributes PParam;
    PParam.rtps.builtin.discovery_config.discoveryProtocol = DiscoveryProtocol_t::SIMPLE;
    PParam.rtps.builtin.discovery_config.use_SIMPLE_EndpointDiscoveryProtocol = true;
    PParam.rtps.builtin.discovery_config.m_simpleEDP.use_PublicationReaderANDSubscriptionWriter = true;
    PParam.rtps.builtin.discovery_config.m_simpleEDP.use_PublicationWriterANDSubscriptionReader = true;
    PParam.rtps.builtin.domainId = 0;
    PParam.rtps.builtin.discovery_config.leaseDuration = c_TimeInfinite;
    PParam.rtps.setName("Participant_pub");
    mp_participant = Domain::createParticipant(PParam);

    if(mp_participant==nullptr)
        return false;
    //REGISTER THE TYPE

    Domain::registerType(mp_participant,&m_type);

    //CREATE THE PUBLISHER
    PublisherAttributes Wparam;
    Wparam.topic.topicKind = NO_KEY;
    Wparam.topic.topicDataType = "HelloWorld";
    Wparam.topic.topicName = "HelloWorldTopic";
    Wparam.topic.historyQos.kind = KEEP_LAST_HISTORY_QOS;
    Wparam.topic.historyQos.depth = 30;
    Wparam.topic.resourceLimitsQos.max_samples = 50;
    Wparam.topic.resourceLimitsQos.allocated_samples = 20;
    Wparam.times.heartbeatPeriod.seconds = 2;
    Wparam.times.heartbeatPeriod.nanosec = 200*1000*1000;
    Wparam.qos.m_reliability.kind = RELIABLE_RELIABILITY_QOS;
    mp_publisher = Domain::createPublisher(mp_participant,Wparam,(PublisherListener*)&m_listener);
    if(mp_publisher == nullptr)
        return false;

    return true;

}


```

这里的参数配置请参阅 API 说明：

*   [RTPSParticipantAttributes](http://www.eprosima.com/docs/fast-rtps/1.9.3/html/classeprosima_1_1fastrtps_1_1rtps_1_1_r_t_p_s_participant_attributes.html)
*   [BuiltinAttributes](http://www.eprosima.com/docs/fast-rtps/1.9.3/html/classeprosima_1_1fastrtps_1_1rtps_1_1_builtin_attributes.html)
*   [PublisherAttributes](http://www.eprosima.com/docs/fast-rtps/1.9.3/html/classeprosima_1_1fastrtps_1_1_publisher_attributes.html)
*   [TopicAttributes](http://www.eprosima.com/docs/fast-rtps/1.9.3/html/classeprosima_1_1fastrtps_1_1_topic_attributes.html)
*   [WriterQos](http://www.eprosima.com/docs/fast-rtps/1.9.3/html/classeprosima_1_1fastrtps_1_1_writer_qos.html)
*   [WriterTimes](http://www.eprosima.com/docs/fast-rtps/1.9.3/html/structeprosima_1_1fastrtps_1_1rtps_1_1_writer_times.html)
*   [WriterQos](http://www.eprosima.com/docs/fast-rtps/1.9.3/html/classeprosima_1_1fastrtps_1_1_writer_qos.html)

Publisher 发送消息的逻辑很简单：

```
bool HelloWorldPublisher::publish(bool waitForListener)
{
    if(m_listener.firstConnected || !waitForListener || m_listener.n_matched>0)
    {
        m_Hello.index(m_Hello.index()+1);
        mp_publisher->write((void*)&m_Hello);
        return true;
    }
    return false;
}


```

注意，这里`write`的对象是通过 idl 文件生成的类型。

Subscriber 的初始化和 Publisher 是类似的：

```
bool HelloWorldSubscriber::init()
{
    ParticipantAttributes PParam;
    PParam.rtps.builtin.discovery_config.discoveryProtocol = DiscoveryProtocol_t::SIMPLE;
    PParam.rtps.builtin.discovery_config.use_SIMPLE_EndpointDiscoveryProtocol = true;
    PParam.rtps.builtin.discovery_config.m_simpleEDP.use_PublicationReaderANDSubscriptionWriter = true;
    PParam.rtps.builtin.discovery_config.m_simpleEDP.use_PublicationWriterANDSubscriptionReader = true;
    PParam.rtps.builtin.domainId = 0;
    PParam.rtps.builtin.discovery_config.leaseDuration = c_TimeInfinite;
    PParam.rtps.setName("Participant_sub");
    mp_participant = Domain::createParticipant(PParam);
    if(mp_participant==nullptr)
        return false;

    //REGISTER THE TYPE

    Domain::registerType(mp_participant,&m_type);
    //CREATE THE SUBSCRIBER
    SubscriberAttributes Rparam;
    Rparam.topic.topicKind = NO_KEY;
    Rparam.topic.topicDataType = "HelloWorld";
    Rparam.topic.topicName = "HelloWorldTopic";
    Rparam.topic.historyQos.kind = KEEP_LAST_HISTORY_QOS;
    Rparam.topic.historyQos.depth = 30;
    Rparam.topic.resourceLimitsQos.max_samples = 50;
    Rparam.topic.resourceLimitsQos.allocated_samples = 20;
    Rparam.qos.m_reliability.kind = RELIABLE_RELIABILITY_QOS;
    Rparam.qos.m_durability.kind = TRANSIENT_LOCAL_DURABILITY_QOS;
    mp_subscriber = Domain::createSubscriber(mp_participant,Rparam,(SubscriberListener*)&m_listener);

    if(mp_subscriber == nullptr)
        return false;


    return true;
}


```

当然，Subscriber 有自己的配置参数类型：

*   [SubscriberAttributes](http://www.eprosima.com/docs/fast-rtps/1.9.3/html/classeprosima_1_1fastrtps_1_1_subscriber_attributes.html)
*   [ReaderQos](http://www.eprosima.com/docs/fast-rtps/1.9.3/html/classeprosima_1_1fastrtps_1_1_reader_qos.html)
*   [ReaderTimes](http://www.eprosima.com/docs/fast-rtps/1.9.3/html/classeprosima_1_1fastrtps_1_1rtps_1_1_reader_times.html)

需要注意的是，Subscriber 与 Publisher 的通信是建立在 Topic 上的，因此对于 Topic 标识的配置要保持一致：

```
Wparam.topic.topicDataType = "HelloWorld";
Wparam.topic.topicName = "HelloWorldTopic";


```

有了 Topic 的这个抽象概念，使得 Subscriber 与 Publisher 不用物理地址上有任何关联，也屏蔽了硬件和操作系统的差异：同样的代码，其编译产物可以一个跑在 x86 的 Mac 系统上，一个跑在 ARM 架构的 Android 设备上。

Subscriber 通过`void HelloWorldSubscriber::SubListener::onNewDataMessage(Subscriber* sub)`方法来处理接受到的数据。在示例的实现中，就是将消息体打印出来：

```
void HelloWorldSubscriber::SubListener::onNewDataMessage(Subscriber* sub)
{
    if(sub->takeNextData((void*)&m_Hello, &m_info))
    {
        if(m_info.sampleKind == ALIVE)
        {
            this->n_samples++;
            // Print your structure data here.
            std::cout << "Message "<<m_Hello.message()<< " "<< m_Hello.index()<< " RECEIVED"<<std::endl;
        }
    }

}


```

Publisher 与 Subscriber 各自有一些回调，开发者可以利用它们来进行需要的处理：

<table><thead><tr><th>回调</th><th>Publisher</th><th>Subscriber</th></tr></thead><tbody><tr><td>onNewDataMessage</td><td>-</td><td>√</td></tr><tr><td>onSubscriptionMatched</td><td>-</td><td>√</td></tr><tr><td>on_requested_deadline_missed</td><td>-</td><td>√</td></tr><tr><td>on_liveliness_changed</td><td>-</td><td>√</td></tr><tr><td>onPublicationMatched</td><td>√</td><td>-</td></tr><tr><td>on_offered_deadline_missed</td><td>√</td><td>-</td></tr><tr><td>on_liveliness_lost</td><td>√</td><td>-</td></tr></tbody></table>

读者 - 写者层是相对于发布者 - 订阅者层更底层的 API。

它提供了更多的控制，但也意味着使用起来会稍微麻烦一些。

两个层次在几个核心概念上存在一一对应的关系，如下表所示：

<table><thead><tr><th>Publisher-Subscriber Layer</th><th>Writer-Reader Layer</th></tr></thead><tbody><tr><td>Domain</td><td>RTPSDomain</td></tr><tr><td>Participant</td><td>RTPSParticipant</td></tr><tr><td>Publisher</td><td>RTPSWriter</td></tr><tr><td>Subscriber</td><td>RTPSReader</td></tr></tbody></table>

如果你浏览 Fast-RTPS 的源码，你会发现其实发布者 - 订阅者层的实现就是依赖读者 - 写者层的。

想要很快的熟悉读者 - 写者层的使用可以浏览下面三个代码示例：

*   [RTPSTest_registered](https://github.com/eProsima/Fast-RTPS/tree/master/examples/C%2B%2B/RTPSTest_registered)
*   [RTPSTest_persistent](https://github.com/eProsima/Fast-RTPS/tree/master/examples/C%2B%2B/RTPSTest_persistent)
*   [RTPSTest_as_socket](https://github.com/eProsima/Fast-RTPS/tree/master/examples/C%2B%2B/RTPSTest_as_socket)

RTPSParticipant，RTPSWriter 和 RTPSReader 都通过 RTPSDomain 创建。

相对于发布者 - 订阅层不一样的是，这一层不支持通过 XML 的形式配置参数。开发者必须通过代码的形式配置所有的参数，例如：

```
//CREATE PARTICIPANT
RTPSParticipantAttributes PParam;
PParam.builtin.discovery_config.discoveryProtocol = eprosima::fastrtps::rtps::DiscoveryProtocol::SIMPLE;
PParam.builtin.use_WriterLivelinessProtocol = true;
mp_participant = RTPSDomain::createParticipant(PParam);
if(mp_participant==nullptr)
	return false;

//CREATE WRITERHISTORY
HistoryAttributes hatt;
hatt.payloadMaxSize = 255;
hatt.maximumReservedCaches = 50;
mp_history = new WriterHistory(hatt);

//CREATE WRITER
WriterAttributes watt;
watt.endpoint.reliabilityKind = BEST_EFFORT;
mp_writer = RTPSDomain::createRTPSWriter(mp_participant,watt,mp_history,&m_listener);


```

这里的逻辑主要就是设置参数和创建 RTPSParticipant，RTPSWriter 对象。并且，RTPSParticipant 将被用来注册 RTPSWriter：

```
TopicAttributes Tatt;
Tatt.topicKind = NO_KEY;
Tatt.topicDataType = "string";
Tatt.topicName = "exampleTopic";
ReaderQos Rqos;
return mp_participant->registerReader(mp_reader, Tatt, Rqos);


```

在 RTPS 协议中，Reader 和 Writer 将有关 Topic 的数据保存在其关联的历史记录中。每个数据段都由一个变更表示，对应的实现是`CacheChange_t`。

更改通过历史记录管理。读者和写者的历史是两种类型：

*   `eprosima::fastrtps::rtps::WriterHistory`;
*   `eprosima::fastrtps::rtps::ReaderHistory`;

对于 Writer 来说，发送消息是往历史中添加变更：

```
//Request a change from the history
CacheChange_t* change = writer->new_change([]() -> uint32_t { return 255;}, ALIVE);
//Write serialized data into the change
change->serializedPayload.length = sprintf((char*) change->serializedPayload.data, "My example string %d", 2)+1;
//Insert change back into the history. The Writer takes care of the rest.
history->add_change(change);


```

而对于 Reader 来说，新消息会被放入到历史中，读取完了可以将其删除：

```
void TestReaderRegistered::MyListener::onNewCacheChangeAdded(
        RTPSReader* reader,
        const CacheChange_t* const change)
{
    printf("Received: %s\n", change->serializedPayload.data);
    reader->getHistory()->remove_change((CacheChange_t*)change);
    n_received++;
}


```

框架会根据消息触发 Reader 的回调。

默认情况下，Writer 的历史在其生命周期以内可以被 Reader 访问。这意味着，一旦 Writer 退出，则其历史就没有了。但如果需要，你可以配置持久化，这使得即便 Writer 重启了，仍然可以维护早先的历史。

使用持久化功能可以保护端点的状态免受意外故障的影响，因为端点在重新启动后仍会继续通信，就像它们刚从网络断开连接一样。

你可以通过 [RTPSTest_persistent](https://github.com/eProsima/Fast-RTPS/tree/master/examples/C%2B%2B/RTPSTest_persistent) 这个示例来了解如何使用这个功能。

要使用持久化功能，Writer 和 Reader 需要进行以下设置：

*   `durabilityKind`设置为`TRANSIENT`
*   `persistence_guid`不能是全 0
*   为 Writer，Reader 或者 RTPSParticipant 设置持久化插件。目前内置的插件是 SQLITE3。

下面是一段代码示例：

```
PropertyPolicy property_policy;
property_policy.properties().emplace_back("dds.persistence.plugin", "builtin.SQLITE3");
property_policy.properties().emplace_back("dds.persistence.sqlite3.filename", "test.db");

//CREATE WRITER
WriterAttributes watt;
watt.endpoint.reliabilityKind = BEST_EFFORT;
watt.endpoint.durabilityKind = TRANSIENT;
watt.endpoint.persistence_guid.guidPrefix.value[11] = 1;
watt.endpoint.persistence_guid.entityId.value[3] = 1;
watt.endpoint.properties = property_policy;

mp_writer = RTPSDomain::createRTPSWriter(mp_participant, watt, mp_history, &m_listener);


```

`durabilityKind`参数定义了 Writer 与新 Reader 匹配时对于已发送的数据的行为，该参数有三个选项：

*   VOLATILE（默认值）：丢掉所有已经发送的数据。
*   TRANSIENT_LOCAL：保存最近发送的 k 条数据。
*   TRANSIENT：与 TRANSIENT_LOCAL 类似，但是还将消息保存到持久化存储中。这就使得即便它的进程异常退出了，其数据不会丢失。

对于读者来说，其配置方法是类似的：

```
PropertyPolicy property_policy;
property_policy.properties().emplace_back("dds.persistence.plugin", "builtin.SQLITE3");
property_policy.properties().emplace_back("dds.persistence.sqlite3.filename", "test.db");

//CREATE READER
ReaderAttributes ratt;
Locator_t loc(22222);
ratt.endpoint.unicastLocatorList.push_back(loc);
ratt.endpoint.durabilityKind = TRANSIENT;
ratt.endpoint.persistence_guid.guidPrefix.value[11] = 2;
ratt.endpoint.persistence_guid.entityId.value[3] = 1;
ratt.endpoint.properties = property_policy;
mp_reader = RTPSDomain::createRTPSReader(mp_participant, ratt, mp_history, &m_listener);


```

使用 Fast-RTPS，你有非常多的 QoS 策略可以配置。

![](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-07-dds-and-fastrtps/QoS.png)

它们主要可以分为下面几类：

*   `durability`：定义了 Writer 与新 Reader 匹配时对于已发送的数据的行为，“持久化” 一节已经提到过。
*   `liveliness`：定义 Publisher 的活跃程度。例如：多长时间发布一次公告消息。
*   `reliability`：定义消息的可靠性。它有两个选项：1、`BEST_EFFORT`，发送消息时，接收者（订阅者）没有到达确认。速度快，但是消息可能会丢失。2、`RELIABLE`，发送方（发布者）期望接收方（订阅者）进行到达确认。速度较慢，但可以防止数据丢失。
*   `partition`：可以在 domain 的物理分区上建立逻辑分区。
*   `deadline`：指定消息的更新频率，当新消息的频率降至某个阈值以下时，会发出警报。这对于需要定期更新数据的场景很有用。
*   `lifespan`：指定 Publisher 发布数据的最大有效期限。当使用寿命到期时，数据将从历史记录中删除。
*   `disablePositiveAcks`：指定是否需要取消确认消息。在不需要严格可靠的通信且带宽受限时，这么做可以减少网络流量。

在实现中，QoS 包含了一系列的类，它们继承自 QoSPolicy 父类：

![](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-07-dds-and-fastrtps/classeprosima_1_1fastrtps_1_1_qos_policy__inherit__graph.png)

之所以提供如此多的选择，是因为不同的系统对于消息的质量有不同的要求。

在实际系统中，并非每个端点都需要本地存储所有数据。DDS 在发送信息方面很聪明，如果消息不一定总是到达预期的目的地，则中间件将保证需要的可靠性。当系统发生更改时，Fast-RTPS 会动态地找出将哪些数据发送到何处，并智能地将更改通知参与者。如果总数据量巨大，则 DDS 会智能过滤并仅发送每个端点真正需要的数据。当需要快速更新时，DDS 发送多播消息以一次更新许多远程应用程序。当数据格式变更时，DDS 会跟踪系统各个部分使用的版本并自动进行转换。对于安全性至关重要的应用程序，DDS 控制访问，强制执行数据流路径并实时加密数据。

当系统要满足：以极高的速度，在动态，苛刻且不可预测的环境工作时，DDS 的真正威力就会显现。

文章的最后，我们通过实际运行程序来进行一些实验。虽然 Fast-RTPS 支持非常多的操作系统，但在 Ubuntu 系统上验证可能是最方便的。

Fast-RTPS 是面向分布式系统的，这意味着在一个系统上验证它的功能意义不大。但另一方面我们大部分人并没有同时拥有多个设备。

在这种情况下，我们可以借助 [docker](https://www.docker.com/)，它可以在同一个系统上运行多个独立的虚拟系统。然后我们就可以在这些独立的系统上进行测试了，这样就模拟了分布式的环境。

Fast-RTPS 提供了包含依赖环境的 Docker 容器，我们只要下载和运行这些容器，就可以拥有多个独立的系统了。

不过，在这运行下面这些示例之前，你需要配置好 docker 环境。关于 [docker](https://www.docker.com/) 的基本使用已经超过了本文的范畴，你可以浏览这个链接：[Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)。

Fast-RTPS 需要的文件可以到官网下载：[https://eprosima.com/index.php/downloads-all](https://eprosima.com/index.php/downloads-all)。

点击上面这个链接，然后输入个人信息就可以进入下载页面了。你可以选择最新版本的 Docker 和 Fast-RTPS 包进行下载：

![](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-07-dds-and-fastrtps/download.png)

考虑到国内的网络状况，下载的速度可能非常慢。我下载需要的文件耗费的好几个小时，为了节省你的时间，我已经将下载好的文件放在了这里：

*   [eProsima_FastRTPS-1.9.3-Linux.tgz](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-07-dds-and-fastrtps/eProsima_FastRTPS-1.9.3-Linux.tgz)：包含了 FastRTPS 的源码和编译命令，用来在 Ubuntu 系统上安装环境。
*   [ubuntu-fast-rtps v1.9.3.tar](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-07-dds-and-fastrtps/ubuntu-fast-rtps%20v1.9.3.tar)：已经预装了 Fast-RTPS 环境，可以在上面运行 Fast-RTPS 的程序，用来进行测试。

在 Ubuntu 系统中，先将 eProsima_FastRTPS-1.9.3-Linux.tgz 解压缩，为了编译它，还需要安装一些依赖，相关命令如下：

```
sudo apt install cmake g++
sudo apt install libasio-dev libtinyxml2-dev
mkdir fast-rtps
tar -xvf eProsima_FastRTPS-1.9.3-Linux.tgz -C fast-rtps/
cd fast-rtps/
chmod a+x install.sh
sudo ./install.sh


```

在这之后你就可以转到 / fast-rtps/src/fastrtps/examples / 目录下编译示例了。不过这个目录下的 CMakeList.txt 似乎存在问题，我在这个文件的开头增加了下面一行才完成编译：

```
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -pthread")


```

编译完成之后，我们并非是在 Ubuntu 系统上运行程序，而是将这些可执行文件放到 docker 容器中，以分布式的环境来运行它们。

所以需要启动 docker 容器：

```
$ docker load -i ubuntu-fast-rtps.tar
$ docker run -it ubuntu-fast-rtps:v1.9.3


```

你可以通过`docker run -it ...`同时启动多个 docker 容器以进行测试（每个容器对应一个通信的参与者。当然，你需要同时打开多个 shell 窗口）。

例如我启动了两个 docker 容器：

```
CONTAINER ID        IMAGE                     COMMAND             CREATED             STATUS              PORTS               NAMES
2125504ee62f        ubuntu-fast-rtps:v1.9.3   "/bin/bash"         5 minutes ago       Up 5 minutes                            mystifying_jennings
b17517fefecd        ubuntu-fast-rtps:v1.9.3   "/bin/bash"         23 minutes ago      Up 23 minutes                           stoic_leavitt


```

运行`docker run -it ...`之后会直接进入 docker 的 shell 中，你可以在根目录创建`fastrtps`目录用来存放测试程序。

然后在 Ubuntu 系统上将编译出的示例程序拷贝到 docker 中：

```
sudo docker cp ./ b17517fefecd:/fastrtps
sudo docker cp ./ 2125504ee62f:/fastrtps


```

在这之后就可以转到 docker 容器的 shell 中运行测试程序了。

例如，在两个 docker 上运行 HelloWorld 的示例：

*   下面是 Publisher 程序：

```
root@b17517fefecd:/fastrtps/HelloWorldExample# ./HelloWorldExample publisher
Publisher running 10 samples.
Publisher matched
Message: HelloWorld with index: 1 SENT
Message: HelloWorld with index: 2 SENT
Message: HelloWorld with index: 3 SENT
Message: HelloWorld with index: 4 SENT
Message: HelloWorld with index: 5 SENT
Message: HelloWorld with index: 6 SENT
Message: HelloWorld with index: 7 SENT
Message: HelloWorld with index: 8 SENT
Message: HelloWorld with index: 9 SENT
Message: HelloWorld with index: 10 SENT


```

*   下面是 Subscriber 程序：

```
root@2125504ee62f:/fastrtps/HelloWorldExample# ./HelloWorldExample subscriber
Starting 
Subscriber running. Please press enter to stop the Subscriber
Subscriber matched
Message HelloWorld 1 RECEIVED
Message HelloWorld 2 RECEIVED
Message HelloWorld 3 RECEIVED
Message HelloWorld 4 RECEIVED
Message HelloWorld 5 RECEIVED
Message HelloWorld 6 RECEIVED
Message HelloWorld 7 RECEIVED
Message HelloWorld 8 RECEIVED
Message HelloWorld 9 RECEIVED
Message HelloWorld 10 RECEIVED
Subscriber unmatched


```

接下来是 Benchmark 程序：

*   Benchmark subscriber 端：

```
root@b17517fefecd:/fastrtps/Benchmark# ./Benchmark subscriber
Subscriber running...
Subscriber matched
Publisher matched
Subscriber unmatched
Publisher unmatched


```

*   Benchmark publisher 端：

```
root@2125504ee62f:/fastrtps/Benchmark# ./Benchmark publisher
Publisher running...
Subscriber matched
Publisher matched. Test starts...
RESULTS after 10000 milliseconds:
COUNT: 53951
SAMPLES: 0,771,668,548,582,716,700,706,408,440,592,636,738,698,648,574,706,776,690,584,638,556,750,740,640,584,572,542,526,560,552,528,608,504,630,478,598,708,620,528,660,718,578,646,702,528,652,528,450,508,566,544,516,616,652,584,532,434,542,678,752,696,412,544,654,766,736,612,496,470,662,580,566,634,674,568,532,546,528,552,552,528,490,508,598,620,672,506,468,654,


```

在运行的过程中，你可以感受到借助 Fast-RTPS，不同系统上的参与者是多么快速的发现了对方并完成了通信的。 当然，你可以运行更多的用例，或者修改代码进行你想要的测试。

*   [What is DDS?](https://www.dds-foundation.org/what-is-dds-3/)
*   [Where Can I Get DDS?](https://www.dds-foundation.org/where-can-i-get-dds/)
*   [PDF: What can DDS do for You?](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-07-dds-and-fastrtps/CoreDX_DDS_Why_Use_DDS.pdf)
*   [Data Distribution Services Performance Evaluation Framework](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-07-dds-and-fastrtps/Kri.pdf)
*   [Using DDS with TSN and Adaptive AUTOSAR](http://www.ieee802.org/1/files/public/docs2018/dg-leigh-autosar-dds-tsn-use-case-1218-v02.pdf)
*   [Object Management Group: Data Distribution Service™](https://www.youtube.com/watch?v=6iICap5G7rw&t=3s)
*   [DDS Interoperability Wire Protocol](https://www.omg.org/spec/DDSI-RTPS/2.2/)
*   [eProsima Fast RTPS Documentation](https://fast-rtps.docs.eprosima.com/)
*   [RTPS Introduction](https://www.eprosima.com/index.php/resources-all/rtps)
*   [eProsima Fast RTPS: PubSub Hello World](https://www.youtube.com/watch?v=JW9yWhekpW4)
*   [Github: Fast-RTPS](https://github.com/eProsima/Fast-RTPS)
*   [DDS in a Nutshell](https://www.youtube.com/watch?v=u-saogMmKOo)
*   [Data Distribution Service](https://www.youtube.com/watch?v=Hp9hzT94M_Q)
*   [What’s the difference between DDS and SOME/IP?](https://stackoverflow.com/questions/51182471/whats-the-difference-between-dds-and-some-ip)
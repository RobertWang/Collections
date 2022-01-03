> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzkzMDE5MDgwMQ==&mid=2247484074&idx=1&sn=665b93529f904f8b3a27f38970118240&chksm=c27f4614f508cf02960a751a7142936b3c80e14921ca346c4e91a7715f7fb93299eeab0611f2&scene=21#wechat_redirect)

业务背景
====

> 随着 Shopee 业务不断扩张，为了更加了解用户对产品的行为反馈，更好地决策产品特性，各团队内部涌现出大量数据分析的需求。例如：客户端用户行为分析（如跳转行为、页面留存等），业务核心指标分析（购买量、购买品类），甚至于 A/B Test 的结果数据分析，都需要一套数据体系来支撑。而通过传统离线数据产出已然不能满足实时运营、活动投放、异常问题发现等需求。
> 
> 为了支持这些实时数据分析能力，我们团队开发了 Boussole——多维数据实时分析系统，旨在通过低成本的方式支撑海量多维数据实时分析。本文将详细描述系统中的实时分析查询引擎 Boussole Engine 作为多维数据分析的核心一环，是如何通过对引擎的设计支撑毫秒级实时数据分析结果返回。

1. 介绍
=====

Boussole 作为多维分析平台，与大多数实时分析系统有类似的数据流向。从数据源拉取数据并经过前置清洗，通过用户在平台中定义的指标和维度以及汇聚方式实时聚合后，将产生的结果数据落入持久化存储，用户通过平台前端配置的相关视图及 Dashboard 实时观测这些最新汇聚出的数据结果。

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn040pVxB0z5JM0h8VnicGP7veLlWW9MXiahlIDn0KPQOiaJpmdc7dRseZKA1dLkDaGYjZApQomtbVroLA/640?wx_fmt=png)

图 1 系统数据流概览

我们在整个数据流中的每个阶段都投入了不少的设计精力，来应对海量数据带来的压力，**本文仅就其中核心的数据查询引擎来介绍设计思路和具体架构**。团队内部启动时面临的首要问题是如何设计一种前后端查询数据和交互的协议，使用户能方便地在前端通过自己的需求查询多维数据。我们在初期调研了一些主流时序数据分析产品，它们主要分为以下几类：

*   类 SQL 的时序数据查询方式，主要有 TimescalesDB[1] 和基于 InfluxQL 实现的 InfluxDB[2]，核心思路是通过 SQL 的方式将维度筛选、维度汇聚、指标间运算和时间过滤等标准的时序数据操作通过 SQL 描述并将结果返回给用户。
    
*   通过 JSON 自定义查询 Schema，主要有 OpenTSD[3] 和 KairosDB[4]，客户端需要查询的指标和维度明确指定在 JSON 字段中，服务端将查询的时序数据结果按要求返回。
    
*   自定义语言实现的查询，主要有 Promethues 的 PromQL[5] 和基于 Flux[6] 实现的 InfluxDB，它们各自都有一套独立的查询语法定义，并且能较好地支持筛选、指标计算和维度汇聚。
    

在选型上，我们最终使用了 PromQL 来作为前后端查询协议，核心原因是它的功能和易用性及业界的使用广泛程度。作为一种表现形式良好的时序数据查询语言，它能满足在前端查询时维度筛选、汇聚和指标计算的所有需求。并且，它的表现形式简单，在有复杂的汇聚需求（多维度复合指标运算、时序子查询等）时能通过自定义查询能力分析现有数据，相比于 SQL 的复杂表述和 OpenTSDB 过于简单的查询功能，PromQL 更符合需求。

要想做到实时分析查询，在项目初期就应该对未来能达到的效果有明确规划。我们希望不论有多少原始数据上报，在查询响应速度方面都能达到毫秒级，下文将详细描述我们是如何设计系统并达到这一目标的。

2. 存储模型
=======

在了解如何实现查询流程前，先介绍一下 Boussole 底层的多维时序数据存储模型。关于多维时序数据的存储，业界大部分实现都是类似的，核心思路是将多维数据细化到粒度最小的单个维度转化为 KV 格式，再通过保存单维度与多维度之间的映射关系，从而将多维时序数据映射在持久存储中。

这里以温度为一个指标举例，说明系统内部如何处理多维时序数据：每个城市都有一个温度采集站，会定时收集此地的温度数据，将数据上报至气象局。并且，由于温度垂直递减的关系，采温站并不会只采集一个高度的数据，而是一批高度的数据。这些数据是不同的，通常情况下在对流层中海拔越高气温越低。这样，温度随时间、高度、地域的变化就形成了一组多维时序数据。

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn040pVxB0z5JM0h8VnicGP7ve3rL5wRwfJoxSeH4CTYuAUDrIkRaRdAQYr1FNy4ibPqybQzd5LNPXtEQ/640?wx_fmt=png)

图 2 多维时序数据转化为 KV 格式存储

如上图所示，采集好的多维数据降维后转换成 KV 格式，方便落地在后端的持久存储中，这样做的好处是不论有多少维度，最终存储的格式是相同的。

按照这个思路，其实能够选型的具体存储引擎有很多，考虑到运维成本和社区的成熟度，最终我们选用 HBase 作为后端存储工具。引擎底层为了适配不同的存储类型，实现了一个存储适配层，使得系统可以在 Redis、Memcache、RocksDB、TiKV 等类似存储作为后端时快速对接，这种做法参考了 Promethues-Remote-Storage-Integrations[7]。

但以这种数据模型存储，是很难查询的。如果不加以处理，多维分析指定维度进行查询时，需要扫描整个以 Temperature 为前缀的所有数据，挑选出用户指定维度后再进行过滤。如果原数据维度组合有很多，这样做的 IO 开销会非常大。为了加速查询过程，系统会对原始数据做预聚合操作。并且为了实现用户在实际使用中维度筛选的便捷性，系统在汇聚时会将某个维度下存在哪些具体维度保存下来，方便后续的筛选聚合分析操作。

2.1 指标的预汇聚
----------

预汇聚的主要目的是当用户以某个维度做聚合操作时可以直接返回数据而不需要做二次计算。用上述采温站的例子来说，此时用户想看到全地域高度为 5 的平均温度，或想看到北京市所有高度的平均温度。若想加快这些数据的返回速度，预聚合是非常关键的一个步骤，它决定了查询引擎在执行时的具体方法。如果要加速这两个条件查询，预汇聚需要的配置及效果如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn040pVxB0z5JM0h8VnicGP7veGgmckXDtWzKd3yM4pQVtWsQ7jNyeStOFP4S3k6InJK3ByZdVMTQGfQ/640?wx_fmt=png)

图 3 预汇聚流程

预汇聚产生的一个问题就是存储放大，这种放大效果会随着维度值的数量和具体的预汇聚规则而发生变化（一个维度个数为 N 的原始数据，如果开启全排列，加速所有条件下的查询，存储会放大为原来的 2^N），选择预汇聚的维度组合需要用户基于其具体使用场景的理解；在数据接入时评估数据模型，也需要对具体分析场景有预先了解。后续的章节将会详述系统实际使用中是如何通过预汇聚和二次汇聚交叉使用来平衡存储和查询速度带来的影响。

2.2 指标的存储
---------

当然这也不是数据在存储中的最终保存形式，落地存储时还需对这些数据做一些转换。系统在汇聚逻辑最终产生数据结果后会将 Metric 和 Tags 的部分通过 FNV64a 进行 Hash，对时间戳进行 uint32 编码，值作为 float64 保存，具体落盘的 KV 格式如下：

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn040pVxB0z5JM0h8VnicGP7veA9zwgttRhkE406rrY6wUyQskNcv3H7b7TL3BfRy8mibZMqx0yOJ7HuQ/640?wx_fmt=png)

图 4 Metric Tab Encoding Format

在指标存储时系统对指标和维度明细进行了 Hash，主要是为了保证 Metric 表中 Key 是定长的，这样在 Range 提取时序曲线过程中不会出问题，防止其他脏数据混入。其次是因为有些数据的维度个数可能很多，导致 Key 较长，影响对存储量的评估。

2.3 维度的存储
---------

系统将维度存下来是因为在前端查询时，用户需要用到维度筛选和维度过滤功能。在实际的存储系统中，每个维度值是一个没有 Value 的 KV 对，因为只用到了 Key 这个属性来筛选和去重。实际使用时，用户只需要知道指标、维度和分析时间区间，就可以获取这段时间存在的维度值列表。

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn040pVxB0z5JM0h8VnicGP7vebz2qZbGhvcXDrA60dTLfpUOWV6lc0MFBRvUCAEF3t76d00nqOZAJnA/640?wx_fmt=png)

图 5 Tag Table 生成逻辑

这里有一个细节，存入维度表时的时间和指标上报的时间不一样了，存入维度的时间比维度实际出现时间早了一些（例如图中存入时间就比实际出现时间少了 10 秒）。其实在存储 Tag 的过程中，系统会强制将 Tag 的时间左对齐到每个整点小时。这么做是由于在时序分析场景中，用户不关心某个维度值在某个时间点是否出现，取而代之的是一段时间内，这个维度下有哪些维度值，通过预先对齐到小时节省了大量的存储空间（一小时内重复出现的维度值不会被写入，假设某个维度值在一小时内都稳定出现，没有断流，预聚合时间粒度为十秒，大约能节省 `(3600-10)/3600 ≈ 99.72%` 的存储空间）。

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn040pVxB0z5JM0h8VnicGP7veynhdRBxVmCEzuiaGIdhR9G42HsWOuZC8uL6H9xrLANmkHqTWQL4UDPw/640?wx_fmt=png)

图 6 维度存储的时间对齐

采用上述整点对齐的存储方式也引入了新的问题，在查询某维度下具体的维度值时可能会混入一些脏数据。如上图所示，存储落地时会把这两个 Location 维度值同时存储在 10:00-11:00 这个区间内。此时，如果用户想查询 10:30-11:00 期间的数据，`Locaiton = Beijing` 这个维度值会被扫描出来，然而实际情况是这段时间并不存在这个数据点。后面的章节将详细描述如何处理掉这些脏维度，并且使它们不在数据查询时返回。

3. 分析查询流程
=========

时序数据的查询流程概括来说是用户输入一个 Query，系统返回一系列带标签的曲线组合。通常用户不仅会查看在存储里的原始汇聚信息，也会对这些信息做上卷、筛选聚合、运算等一系列操作，最终得到自己想要的数据结果，整个查询引擎的工作流程都是围绕这些功能展开的。

在系统中一次查询主要经历以下几个阶段：首先是 PromQL 的 Parser 和 Optimizer，这里直接使用了开源 MetricsQL，相比 Promethues 原生的 PromQL，它具有更多的拓展能力，方便以后在查询过程中的各种定制化拓展。早期的 Boussole 版本在拿到解析器生成的 AST 后就直接开始数据获取和数据加工流程，首要的工作就是数据抽取，此时需要知道存储里具体哪些曲线是一次查询所需要的。

具体哪些细化的维度时序需要从存储中抽取出来，取决于用户在前端进行的维度筛选和维度展开。这里的维度筛选对应到上述温度采集的例子中，具体的使用场景是只查看位置为北京的数据，或查看高度不等于 200 的所有数据。维度筛选通常来说是比较复杂的，明确且固定值的维度筛选可以在数据获取时少查一次存储（不需要确定这个 Key 是否存在，直接能够拼接完 Metric 表中的 Key），除此之外诸如大于、不等于、正则匹配等各种非确定性查询都需要再次获取全量维度值来逐一进行匹配，命中的维度值需要加入待抽取指标数据的维度列表中。

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn040pVxB0z5JM0h8VnicGP7vezrlOXichymQXUIEHOt2ubibYESGhiaFtIzydU0icUzGibbU4lmoy6BXSDYQ/640?wx_fmt=png)

图 7 维度筛选导出抽取指标流程

例如用户在发起查询时指定了筛选条件 `location=(Beijing||Shanghai)，height!=200`，在筛选待抽取数据列表时整个流程如上图，最后得到的待抽取指标数据维度列表就是需要在底层存储查询的具体曲线。

维度汇聚也影响着需要拉取的数据集的大小。在多维时序分析中，用户查询到的结果往往不止一条曲线，而是在某个维度下钻或上卷的结果，或是某几个维度下钻或上卷的结果。并且，维度汇聚和维度选择会产生一定的关系。如果汇聚和筛选作用在同一个维度中，那么筛选的优先级是比汇聚高的，这时需要先排除用户筛选掉的维度后再汇聚数据才会产生正确结果。

准备好待抽取指标数据列表后，需要处理的就是聚合逻辑以及指标间运算。本质上来说这些操作都是对一批带标签的曲线集合进行数学运算。但由于曲线带上了标签，所以一些处理逻辑变得有些复杂。比如在聚合逻辑中，按照一个维度下钻并对其他所有维度取 Max 操作，最终，除了此维度以外其他维度都不会保留下来，曲线的标签发生了变化。在指标间运算过程中，只有相同标签的曲线才会参与计算。例如计算以 URL 维度展开的成功率，需要用成功数除以总数，只有维度完全相同时，曲线逐点计算才有意义。不过在指标与实数计算的过程中，实数会忽略标签，与所有维度标签一起计算，计算作用于每条指标曲线中，所以可以认为实数计算时是带有任意标签的。

在实际场景中做到以上的分析查询功能其实已经满足了绝大部分需求，但在能力拓展上仍留有很大空间，比如：需要支持一些特定的时序处理逻辑时会自定义时序处理函数，并在前端提示这些可用的函数用法。

在下表中我们将简述 MetricsQL 和 FLux 的区别。如果最初选用 Flux 作为前后端的查询协议，可以在发起查询时让用户自定义这些函数，在发起时直接提交。虽然有较高的自由度，但最初选型时我们并没有使用 Flux，核心原因是它是一种新的查询语言，理解并学习需要花费较高成本。并且，未经优化的 Flux 语句可能会导致额外的资源消耗，这些 Query 提交至后台处理时，系统需要在资源限制和超时控制上做一些额外工作，才能保证执行性能和稳定性符合预期。

<table data-darkmode-color-164122365909410="rgb(163, 163, 163)" data-darkmode-original-color-164122365909410="#fff|rgb(0,0,0)"><thead data-darkmode-color-164122365909410="rgb(163, 163, 163)" data-darkmode-original-color-164122365909410="#fff|rgb(0,0,0)"><tr data-darkmode-color-164122365909410="rgb(163, 163, 163)" data-darkmode-original-color-164122365909410="#fff|rgb(0,0,0)" data-darkmode-bgcolor-164122365909410="rgb(25, 25, 25)" data-darkmode-original-bgcolor-164122365909410="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-darkmode-color-164122365909410="rgb(77, 77, 77)" data-darkmode-original-color-164122365909410="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-164122365909410="rgb(189, 190, 193)" data-darkmode-original-bgcolor-164122365909410="#fff|rgb(255,255,255)|rgb(247, 249, 253)" data-style="text-align: left; font-size: 14px; border-right: 0.2px; border-left: 0.2px; color: rgb(77, 77, 77); min-width: 95px; background-color: rgb(247, 249, 253); border-bottom-color: rgb(255, 255, 255); border-top-width: 1px; border-top-color: rgb(255, 255, 255); border-top-left-radius: 8px; border-bottom-left-radius: 8px;">查询语言</th><th data-darkmode-color-164122365909410="rgb(77, 77, 77)" data-darkmode-original-color-164122365909410="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-164122365909410="rgb(189, 190, 193)" data-darkmode-original-bgcolor-164122365909410="#fff|rgb(255,255,255)|rgb(247, 249, 253)" data-style="text-align: left; font-size: 14px; border-right: 0.2px; border-left: 0.2px; color: rgb(77, 77, 77); min-width: 130px; background-color: rgb(247, 249, 253); border-bottom-color: rgb(255, 255, 255); border-top-width: 1px; border-top-color: rgb(255, 255, 255);">拓展能力</th><th data-darkmode-color-164122365909410="rgb(77, 77, 77)" data-darkmode-original-color-164122365909410="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-164122365909410="rgb(189, 190, 193)" data-darkmode-original-bgcolor-164122365909410="#fff|rgb(255,255,255)|rgb(247, 249, 253)" data-style="text-align: left; font-size: 14px; border-right: 0.2px; border-left: 0.2px; color: rgb(77, 77, 77); min-width: 175px; background-color: rgb(247, 249, 253); border-bottom-color: rgb(255, 255, 255); border-top-width: 1px; border-top-color: rgb(255, 255, 255);">学习成本</th><th data-darkmode-color-164122365909410="rgb(77, 77, 77)" data-darkmode-original-color-164122365909410="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-164122365909410="rgb(189, 190, 193)" data-darkmode-original-bgcolor-164122365909410="#fff|rgb(255,255,255)|rgb(247, 249, 253)" data-style="text-align: left; font-size: 14px; border-right: 0.2px; border-left: 0.2px; color: rgb(77, 77, 77); min-width: 160px; background-color: rgb(247, 249, 253); border-bottom-color: rgb(255, 255, 255); border-top-width: 1px; border-top-color: rgb(255, 255, 255); border-top-right-radius: 8px; border-bottom-right-radius: 8px;">资源消耗控制</th></tr></thead><tbody data-darkmode-color-164122365909410="rgb(163, 163, 163)" data-darkmode-original-color-164122365909410="#fff|rgb(0,0,0)"><tr data-darkmode-color-164122365909410="rgb(163, 163, 163)" data-darkmode-original-color-164122365909410="#fff|rgb(0,0,0)" data-darkmode-bgcolor-164122365909410="rgb(25, 25, 25)" data-darkmode-original-bgcolor-164122365909410="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-164122365909410="rgb(153, 153, 153)" data-darkmode-original-color-164122365909410="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-164122365909410="rgb(25, 25, 25)" data-darkmode-original-bgcolor-164122365909410="#fff|rgb(255,255,255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px;"><strong data-darkmode-color-164122365909410="rgb(163, 163, 163)" data-darkmode-original-color-164122365909410="#fff|rgb(0,0,0)|rgb(77, 77, 77)|rgb(0,0,0)" data-darkmode-bgcolor-164122365909410="rgb(25, 25, 25)" data-darkmode-original-bgcolor-164122365909410="#fff|rgb(255,255,255)" data-style="color: black;">MetricsQL</strong></td><td data-darkmode-color-164122365909410="rgb(153, 153, 153)" data-darkmode-original-color-164122365909410="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-164122365909410="rgb(25, 25, 25)" data-darkmode-original-bgcolor-164122365909410="#fff|rgb(255,255,255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px;">较弱。需要在语言层面定义新功能。</td><td data-darkmode-color-164122365909410="rgb(153, 153, 153)" data-darkmode-original-color-164122365909410="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-164122365909410="rgb(25, 25, 25)" data-darkmode-original-bgcolor-164122365909410="#fff|rgb(255,255,255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px;">较低。PromQL 语法简单。2014 年发布后广泛应用在监控领域，能接触到的资料较多。</td><td data-darkmode-color-164122365909410="rgb(153, 153, 153)" data-darkmode-original-color-164122365909410="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-164122365909410="rgb(25, 25, 25)" data-darkmode-original-bgcolor-164122365909410="#fff|rgb(255,255,255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px;">可控程度高。能严格控制每个执行计划中的计算量和数据量。</td></tr><tr data-darkmode-color-164122365909410="rgb(163, 163, 163)" data-darkmode-original-color-164122365909410="#fff|rgb(0,0,0)" data-darkmode-bgcolor-164122365909410="rgb(32, 32, 32)" data-darkmode-original-bgcolor-164122365909410="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-164122365909410="rgb(153, 153, 153)" data-darkmode-original-color-164122365909410="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-164122365909410="rgb(25, 25, 25)" data-darkmode-original-bgcolor-164122365909410="#fff|rgb(248, 248, 248)|rgb(255, 255, 255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; background-color: rgb(255, 255, 255);"><strong data-darkmode-color-164122365909410="rgb(163, 163, 163)" data-darkmode-original-color-164122365909410="#fff|rgb(0,0,0)|rgb(77, 77, 77)|rgb(0,0,0)" data-darkmode-bgcolor-164122365909410="rgb(25, 25, 25)" data-darkmode-original-bgcolor-164122365909410="#fff|rgb(248, 248, 248)|rgb(255, 255, 255)" data-style="color: black;">Flux</strong></td><td data-darkmode-color-164122365909410="rgb(153, 153, 153)" data-darkmode-original-color-164122365909410="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-164122365909410="rgb(25, 25, 25)" data-darkmode-original-bgcolor-164122365909410="#fff|rgb(248, 248, 248)|rgb(255, 255, 255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; background-color: rgb(255, 255, 255);">强。支持自定义函数，有大量包可以引用。</td><td data-darkmode-color-164122365909410="rgb(153, 153, 153)" data-darkmode-original-color-164122365909410="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-164122365909410="rgb(25, 25, 25)" data-darkmode-original-bgcolor-164122365909410="#fff|rgb(248, 248, 248)|rgb(255, 255, 255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; background-color: rgb(255, 255, 255);">较高。自定义了一套类似 Lambda 的流处理语言，2018 年发布后使用在 InfluxDB 上运行。</td><td data-darkmode-color-164122365909410="rgb(153, 153, 153)" data-darkmode-original-color-164122365909410="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-164122365909410="rgb(25, 25, 25)" data-darkmode-original-bgcolor-164122365909410="#fff|rgb(248, 248, 248)|rgb(255, 255, 255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; background-color: rgb(255, 255, 255);">可控程度低。引入自定义函数无法精确控制数据在集群中实际的资源消耗。</td></tr></tbody></table>

4. 查询条件与预汇聚规则
=============

Boussole 在窗口汇聚时并不会将所选维度的所有组合都进行预汇聚计算，在配置数据源时会让用户选择一些预先需要查询的维度组合进行预汇聚，从而在查询时能够快速返回结果。**预先设定维度组合进行汇聚计算**是预汇聚统计里常用的一种方式，它在查询速度和存储大小之间做出了一定平衡。存储空间不足时，适当减少预汇聚的维度组合数，能减少存储开销。相反，如果开启全部维度组合的预汇聚，能够使用户在任意维度下自由组合查询并且保持快速的响应时间。如下图中预汇聚的结果：

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn040pVxB0z5JM0h8VnicGP7vehcKpGXgs52vFH8LMEJmXHTVXiarb8NDqPaicgkyTPKdUXsVPM1Hypzkw/640?wx_fmt=png)

图 8 不同预聚合规则产生的结果

在上图中，命中预汇聚规则时，如果用户查询条件 `A=1,C=3` 下 MetricX 的和值，存储会直接返回 12，但如果命中没有命中预汇聚的查询，例如用户这时查询 `D=12` 下 Metric 的和值或 `A=1` 下 Metric 的和值，都是无法通过现有存储直接返回的，所以引擎必须要实现二次汇聚，通过现有汇聚好的数据进行二次加工得到用户想要查询的结果。

其实这里的实现思路比较简单：选择一个预汇聚结果中相对于目标查询维度最匹配的汇聚结果进行二次汇聚，例如用户想查询 `A=1` 下的值，通过组合 [A,B] 汇聚结果直接可以取出三条数据，并将这三条数据合并得到结果 `Sum(MetricX){A=1}=40`。但这个结果并不是最优的，因为通过组合 [A,C] 只需要两条数据就能汇聚出相同结果。所以这里定义的最优匹配其实是为了汇聚目标结果所需要获取最小数据量的预汇聚集合。当然为了保证用户的每个查询都是有结果的，系统设计在预汇聚时必须开启一个全部维度的组合（如例子中的 [A,B,C,D]），这样不论用户需要查询任何子维度集，都会是这个全集的子集。

通用化一些，用户需要查询维度集 X 的汇聚结果，此时有预汇聚维度集列表 `YL=[Y1,Y2,……Yn]`，系统需要先判断 `X∈YL`，如果成立则直接去底层查询结果数据，不需要二次汇聚。如果不存在，则需要逐一计算 YL 中所有成员与 X 的差集 `DYL=[DY1,DY2……DYn]`，如果这个结果存在且非空，逐一在维度表中查询这些维度下的维度值个数，选取乘积最小的一组差集，并追回导出它的 Yx，这个预汇聚组合 Yx 就是查询维度集 X 的最优的二次汇聚数据来源。

在实际生产中从 `X => Yx` 的关系推导损耗是比较大的，核心耗时主要是花费在计算某一维度下的值个数有多少（对应存储的 RowCount 操作），为了加速后续相同维度组合的二次汇聚查询，引擎会把这种对应关系缓存下来以备后用。在缓存的生存时间选择上，我们采用了与 TCP 慢启动机制类似的策略，如果缓存过期后下次的推导结果没有发生变化，则说明这个指标的维度数目相对稳定，系统会翻倍此缓存时间，防止频繁计算汇聚关系导致的额外性能开销。

5. 抽样和清空
========

Boussole 目前提供给用户可选的汇聚最小时间粒度为 10s，受限于所拥有的存储资源的大小，系统将存储的最长保存期限设定为一年半，日常使用时用户经常会查询近一个月的数据来观察数据波动，这是一个很常见的需求，而如此细粒度的数据在做用户展示时也有不小的压力。

这种压力来自两方面，一个是前端渲染给浏览器带来的压力，另外是查询的结果请求数据很大，普通客户带宽传输就需要较长时间等待。以一个二十条曲线汇聚统计图为例，假设汇聚粒度为 10s，查询近 60 天的数据。前端共需要渲染的点个数为 10,368,000 个，如果以纯二进制数据在 Web 中传输，忽略维度信息和请求头尾，一个 uint32 类型时间戳 4Byte 和一个 float64 类型的值 8Byte，整个包大约需要 118.65MB，开启 Delta-of-Delta 压缩后需要 15.1MB 的传输大小。这个体积的返回如果需要用户在发起请求后 350ms 返回，就算忽略服务端的处理时间，用户需要 345M 的带宽才能保证响应时间达标。

在查看长期趋势图时，用户不关心是否每个点都能展示，这时用户实际观察的是曲线的波动及大体趋势。在查看趋势时，如果某个细节出现了异常，用户通常会放大这个时间区间，观察区间中某些异常点的具体值，这时需要对这些数据点进行明确返回。所以区间时间长意味着需要忽略局部细节，时间短则要全量展示。为了平衡这两者之间的关系，需要控制单条曲线能显示点的个数，在后端做抽样逻辑处理，无论查询的时间多长，保持抽样的输出结果大小即可。

在实际生产中，系统配置的抽样原则是保留 3840 个点，原因是这个数字是目前的显示设备横向分辨率值的普遍大小，可以让前端渲染出图在一个 4K 显示器全屏展示而不失真，尽可能利用设备的显示优势展示每一个数据点。以 3840 来预估刚才的例子，60 天的曲线数据开启压缩后大约为 117.9KB，不仅加速了传输，加快了端上的渲染速度，同时也降低了服务端出口带宽的压力。

6. Distributed PromQL Executor 架构设计
===================================

上线一段时间后，随着业务上报的维度组合数变多，我们通过对系统性能和资源进行监控，发现了一些有趣的现象：

*   某些查询节点的资源使用会由于一个复杂 Query 突然升高。由于每个 Query 在查询节点中都是单独处理的，在动辄几万甚至上十万维度的汇聚，涉及到子查询和多指标间计算时，单个节点的资源消耗会飙升。
    
*   一次查询会向存储发送大量拉取请求，导致内核 TCP 缓存队列缓存阻塞。由于每个 Query 在维度筛选和汇聚后需要查询的基础数据可能会达到上万至十万条，每条曲线都会涉及对存储进行一次区间扫描，短时间内大量 RPC 请求直接影响了查询的响应时间。
    

第一个问题是系统的隐患，查询资源无法平均分配，在整体利用率不高的情况下偶尔单节点快速打满，使得系统的上限不稳定。第二个问题则更严重，影响了单个请求的响应时间，并且机器可能由于 TCP 内核阻塞影响其他查询请求，出现雪崩现象。

为了解决这些问题，Boussole Engine 参考了 CockroachDB(CRDB) Distributed SQL 的设计思路，实现了一个简单的 Distributed PromQL Executor。CRDB 的分布式 SQL 实现比较复杂，它采用查询和存储节点绑定的方式，能将适合的执行计划移动到距离存储更近的节点执行。尽管实现上由于架构的不同存在一些差异，不过解决问题的思路是相同的，都会将查询请求转化为分布式处理计划，将单个查询绑定到集群中的多个节点上，由收到原始请求的节点经过最后的一系列处理（后续会提到抽样及清空逻辑）返回给客户端。

例如一个计算 URL 可用性的简单表达式，它用到了简单的指标间运算，需要拉取两个指标来进行除法运算，最后通过聚合函数在 URL 维度上聚合曲线，具体的执行计划如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn040pVxB0z5JM0h8VnicGP7ve0sEVAQGPpALb920CicobaXMPtjmjf9h8bic6jvAtGqdOQcD6CT9N6OMQ/640?wx_fmt=png)

图 9 计算 URL 维度展开的执行计划

由于数据获取涉及的操作对单机网络可能造成的影响，在引擎设计时让某些步骤强制分配到其他节点执行，而有些简单的过滤和汇总在当前节点计算，具体的决策取决于系统在执行时评估要计算的数据量。

启动分布式查询之后，资源飙升的现象在集群中有所缓解，各个执行节点的资源使用也趋于平均，集群内节点资源利用率日内最大差异由 896.28% 下降到 171.86%。但正如预期，由于分布式执行造成了额外的网络通信，导致整体执行时间变长。我们统计了一周内用户的查询情况后发现，原来平均一次的查询，额外增加了 2.2 次 RPC 访问，由于节点之间的数据移动在两端编解码的额外开销，导致整体查询时间平均增加了 31.9ms。

实际上对于简单查询做分布式处理确实是存在额外开销的，我们做查询分布式处理的初衷是为了平衡资源，但一些简单 Query 并不会引起性能资源的额外消耗。相反，启用分布式查询后耗时增加了。为此需要寻找一个开启分布式查询的临界点，将简单查询和复杂汇聚区别处理，做到开销与收益的平衡。这个临界点是基于查询细粒度曲线的个数和时长决定的，总体上这也反映了需要查询数据集的包大小。具体的值如何设定，是根据集群所能容忍的资源不平衡度决定的，实际生产中大多数用户查询的简单 Query 都在 144,000 个点的数据体积之内，所以系统将这个值定为是否开启分布式查询的条件。

为了应对第二个问题，我们首先在机器层面进行了调整，启动了网卡多队列，并且增大了 TCP 缓冲区大小等参数。但这些调整并不能直接解决问题，本质上只是在网络层面抛出异常和等待时长方面做一定的周旋。根本的解决方法是引入了 HBase Coprocessor 来将大量请求组合成单个请求，并在 Coprocessor 中启用了 Delta-of-Delta[8] 时序压缩算法，在实际生产中对 90 万条一小时的时序曲线进行压缩测试，Delta-of-Delta 可以实现 13.1% 的压缩比，节约大量传输带宽。

7. 未来展望
=======

作为一套落地实际应用场景中的查询分析引擎，Boussole Engine 仍处于起步阶段，有很多需要打磨和优化的细节，同时也有大量的遗留工作需要完成。现阶段的成果一部分是对开源产品的参考，一部分是业界相关领域通用解决方案的落地，还有一部分是团队内部在实际使用时发现问题的修复补充及优化。其实方向和目标是非常明确的，我们希望它能够在支持更多功能特性的情况下 blazing fast and low cost。

随着业务的发展，与日俱增的数据量及每天高频次实时分析需求对整个系统的设计和迭代都带来了不小的考验。与此同时团队内部也总结了许多时序数据查询处理的经验，基于实际场景中出现的问题进行针对性优化，让它成为业务和用户真正觉得好用的产品，这也使得平台在业务内部被广泛使用。未来我们还会继续优化引擎速度，提高跨节点数据传输效率，分析反馈学习用户预聚合维度加速查询，尝试新的时序存储方式和模型，降低成本且提升查询效率。

参考资料
====

[1]

TimescalesDB: PostgreSQL for time‑series. _https://www.timescale.com/_

[2]

InfluxDB: Open Source Time Series, Analytics Database. _https://www.influxdata.com/_

[3]

OpenTSD: OpenTSDB: ADistributed, Scalable Monitoring System. _http://opentsdb.net/_

[4]

KairosDB: KariosDB: fast distributed scalable time series database. _https://kairosdb.github.io/_

[5]

PromQL: Prometheus Query Language Querying basics. _https://prometheus.io/docs/prometheus/latest/querying/basics/_

[6]

Flux: InfluxData’s functional data scripting language. _https://docs.influxdata.com/influxdb/v2.0/query-data/get-started/_

[7]

Promethues-Remote-Storage-Integrations: _https://prometheus.io/docs/prometheus/latest/storage/_

[8]

Delta-of-Delta: T. Pelkonen et al., Gorilla: "A fast scalable in-memory time series database", Proc. VLDB Endowment, vol. 8, no. 12, pp. 1816-1827, 2015.

> **本文作者**
> 
> Zhuo，后端开发工程师。主要从事实时多维时序数据存储及分析相关工作，来自 SeaMoney Data 团队。

> **加入我们**
> 
> SeaMoney Data 团队为 SeaMoney 旗下多款业务提供多样的数据服务和应用，包括数据集市和报表、实时多维指标分析、实时特征、用户画像等。每日接收和计算来自于多个市场多个业务系统的海量数据，为业务提供准确、及时、高可用的离线和实时数据服务。
> 
> 目前团队大量岗位持续招聘中，诚招数据开发、后端开发、前端、产品、测试，感兴趣的同学可以将简历发送至：jacky.zhou@shopee.com（邮件主题请注明：SeaMoney 数据服务 - 来自技术博客）。
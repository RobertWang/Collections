> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/7Mdq1hvn0fAwPCXBrVESew)

> 一文带大家深入了解分布式数据库 TiDB 的背景和原理

> **分布式数据库的诞生背景**  

随着互联网的飞速发展，业务量可能在短短的时间内爆发式地增长，对应的数据量可能快速地从几百 GB 涨到几百个 TB，传统的单机数据库提供的服务，在系统的可扩展性、性价比方面已经不再适用。比如 MySQL 数据库，缺点是没法做到水平扩展。MySQL 要想能做到水平扩展，唯一的方法就业务层的分库分表或者使用中间件等方案。但是，这些中间层方案也有很大局限性，执行计划不是最优，分布式事务，跨节点 join，扩容复杂等。

> **分布式数据库 TiDB 的介绍**

TiDB 是 一个开源分布式 NewSQL 数据库。实现了自动的水平伸缩，强一致性的分布式事务，基于 Raft 算法的多副本复制等重要 NewSQL 特性。TiDB 结合了 RDBMS 和 NoSQL 的优点，部署简单，在线弹性扩容和异步表结构变更不影响业务， 真正的异地多活及自动故障恢复保障数据安全，同时兼容 MySQL 协议，使迁移使用成本降到极低。

TiDB 具备如下 NewSQL 核心特性：

*   SQL 支持 （TiDB 是 MySQL 兼容的）
    
*   水平线性弹性扩展
    
*   分布式事务
    
*   跨数据中心数据强一致性保证
    
*   故障自恢复的高可用
    

TiDB 的设计目标是 100% 的 OLTP 场景和 80% 的 OLAP 场景。

TiDB 对业务没有任何侵入性，能优雅的替换传统的数据库中间件、数据库分库分表等 Sharding 方案。同时它也让开发运维人员不用关注数据库 Scale 的细节问题，专注于业务开发，极大的提升研发的生产力。

> **TiDB 整体架构**

要深入了解 TiDB 的水平扩展和高可用特点，首先需要了解 TiDB 的整体架构。

**TiDB Server**

TiDB Server 负责接收 SQL 请求，处理 SQL 相关的逻辑，并通过 PD 找到存储计算所需数据的 TiKV 地址，与 TiKV 交互获取数据，最终返回结果。TiDB Server 是无状态的，其本身并不存储数据，只负责计算，可以无限水平扩展，可以通过负载均衡组件（如 LVS、HAProxy 或 F5）对外提供统一的接入地址。

**PD Server**

Placement Driver (简称 PD) 是整个集群的管理模块，其主要工作有三个：一是存储集群的元信息（某个 Key 存储在哪个 TiKV 节点）；二是对 TiKV 集群进行调度和负载均衡（如数据的迁移、Raft group leader 的迁移等）；三是分配全局唯一且递增的事务 ID。

PD 是一个集群，需要部署奇数个节点，一般线上推荐至少部署 3 个节点。

**TiKV Server**

TiKV Server 负责存储数据，从外部看 TiKV 是一个分布式的提供事务的 Key-Value 存储引擎。存储数据的基本单位是 Region（区域），每个 Region 负责存储一个 Key Range （从 StartKey 到 EndKey 的左闭右开区间）的数据，每个 TiKV 节点会负责多个 Region 。TiKV 使用 Raft 协议做复制，保持数据的一致性和容灾。副本以 Region 为单位进行管理，不同节点上的多个 Region 构成一个 Raft Group，互为副本。数据在多个 TiKV 之间的负载均衡由 PD 调度，这里也是以 Region 为单位进行调度。

> **核心特性**

**水平扩展**  

无限水平扩展是 TiDB 的一大特点，这里说的水平扩展包括两方面：计算能力和存储能力。TiDB Server 负责处理 SQL 请求，随着业务的增长，可以简单的添加 TiDB Server 节点，提高整体的处理能力，提供更高的吞吐。TiKV 负责存储数据，随着数据量的增长，可以部署更多的 TiKV Server 节点解决数据 Scale 的问题。PD 会在 TiKV 节点之间以 Region 为单位做调度，将部分数据迁移到新加的节点上。所以在业务的早期，可以只部署少量的服务实例，随着业务量的增长，按照需求添加 TiKV 或者 TiDB 实例。

**高可用**

高可用是 TiDB 的另一大特点，TiDB/TiKV/PD 这三个组件都能容忍部分实例失效，不影响整个集群的可用性

> **TIDB 的实现原理**  

TiDB 架构是 SQL 层和 KV 存储层分离，相当于 InnoDB 插件存储引擎与 MySQL 的关系。从下图可以看出整个系统是高度分层的，最底层选用了当前比较流行的存储引擎 RocksDB，RockDB 性能很好但是是单机的，为了保证高可用所以写多份，上层使用 Raft 协议来保证单机失效后数据不丢失不出错。保证有了比较安全的 KV 存储的基础上再去构建多版本，再去构建分布式事务，这样就构成了存储层 TiKV。有了 TiKV，TiDB 层只需要实现 SQL 层，再加上 MySQL 协议的支持，应用程序就能像访问 MySQL 那样去访问 TiDB 了。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/48MFTQpxichm3npcpzicCmnhyTnnaxjsdwdVcKSwNibUz3eCBybicQtBYlzzuAXSKLch6fTFj3Yd3IpHtadw7Qj3Gw/640?wx_fmt=jpeg)

> **应用场景**  

TiDB 的四个主要应用场景，一是 MySQL 分片与合并；二是直接替换 MySQL；三是用做数据仓库；四是作为其他系统的一个模块。

用例 1：MySQL 分片与合并

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/48MFTQpxichm3npcpzicCmnhyTnnaxjsdwtLQZjTU60lkZUrnrHCEZAn72QnG1jrNqeZQnxt7r0r66TjGwC8ARLQ/640?wx_fmt=jpeg)

TiDB 应用的第一类场景是 MySQL 的分片与合并。对于已经在用 MySQL 的业务，分库、分表、分片、中间件是常用手段，随着分片的增多，跨分片查询是一大难题。TiDB 在业务层兼容 MySQL 的访问协议，PingCAP 做了一个数据同步的工具——Syncer，它可以把 TiDB 作为一个 MySQL Slave，将 TiDB 作为现有数据库的从库接在主 MySQL 库的后方，在这一层将数据打通，可以直接进行复杂的跨库、跨表、跨业务的实时 SQL 查询。过去的数据库都是一主多从，有了 TiDB 以后，可以反过来做到多主一从。

用例 2：直接替换 MySQL

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/48MFTQpxichm3npcpzicCmnhyTnnaxjsdwb1H3OqCica74tPq4ib06Zaxdng8rtkpW9ibibFOTUrvNXOgyRZxLnkMAXw/640?wx_fmt=jpeg)

第二类场景是用 TiDB 直接去替换 MySQL。如果你的 IT 架构在搭建之初并未考虑分库分表的问题，全部用了 MySQL，随着业务的快速增长，海量高并发的 OLTP 场景越来越多，如何解决架构上的弊端呢?

在一个 TiDB 的数据库上，所有业务场景不需要做分库分表，所有的分布式工作都由数据库层完成。TiDB 兼容 MySQL 协议，所以可以直接替换 MySQL，而且基本做到了开箱即用，完全不用担心传统分库分表方案带来繁重的工作负担和复杂的维护成本，友好的用户界面让常规的技术人员可以高效地进行维护和管理。另外，TiDB 具有 NoSQL 类似的扩容能力，在数据量和访问流量持续增长的情况下能够通过水平扩容提高系统的业务支撑能力，并且响应延迟稳定。

**用例 3：数据仓库**

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/48MFTQpxichm3npcpzicCmnhyTnnaxjsdwectTvFvmJkKPHtBraXADKhv86qsw56VWl99ZxMjfUXTulEBPgx8GZw/640?wx_fmt=jpeg)

TiDB 本身是一个分布式系统，第三种使用场景是将 TiDB 当作数据仓库使用。TPC-H 是数据分析领域的一个测试集，TiDB 2.0 在 OLAP 场景下的性能有了大幅提升，原来只能在数据仓库里面跑的一些复杂的 Query，在 TiDB 2.0 里面跑，时间基本都能控制在 10 秒以内。当然，因为 OLAP 的范畴非常大，TiDB 的 SQL 也有搞不定的情况，为此 PingCAP 开源了 TiSpark，TiSpark 是一个 Spark 插件，用户可以直接用 Spark SQL 实时地在 TiKV 上做大数据分析。

用例 4：作为其他系统的模块

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/48MFTQpxichm3npcpzicCmnhyTnnaxjsdwJ2L6RFlCO2UJaGia75FIgx6wxlP03ybANzVXvQiaS0PnJ1OwWicutkTYA/640?wx_fmt=jpeg)

TiDB 是一个传统的存储跟计算分离的项目，其底层的 Key-Value 层，可以单独作为一个 HBase 的 Replacement 来用，它同时支持跨行事务。TiDB 对外提供两个 API 接口，一个是 ACID Transaction 的 API，用于支持跨行事务；另一个是 Raw API，它可以做单行的事务，换来的是整个性能的提升，但不提供跨行事务的 ACID 支持。用户可以根据自身的需求在两个 API 之间自行选择。例如有一些用户直接在 TiKV 之上实现了 Redis 协议，将 TiKV 替换一些大容量，对延迟要求不高的 Redis 场景。

> **总结**

本篇文章介绍了分布式数据库 tidb 特性、架构、应用场景，但是是否使用在我们现有系统中需要从长计议。tidb 的功能很强大，但强大的背后意味着大量硬件资源的支撑。结合实际情况选择最适合的才是最好的方案。
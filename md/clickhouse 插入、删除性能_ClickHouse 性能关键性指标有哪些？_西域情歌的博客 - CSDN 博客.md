> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_42527589/article/details/113581995)

如果您紧跟数据库领域的最新发展，则可能对 ClickHouse 已经耳熟能详了，它是专为 OLAP 设计的列式数据库管理系统。ClickHouse 由 Yandex 开发，于 2016 年开源，这使其成为最新的列式数据库管理系统之一，当前被作为开源数据库被广泛应用。

因为 ClickHouse 支持实时，高速报告，所以它是一个功能强大的工具，特别是对于需要即时，快速和灵活的数据分析方式的现代 DevOps 团队而言。

但是，与大多数 DevOps 工具一样，ClickHouse 仅在受到正确管理和监控的情况下才能提供巨大的价值。即使该工具是为高性能而设计的，实际上要实现高性能也需要仔细注意系统的运行状况。

考虑到这一需求，本文 (由三部分组成的系列文章的第一部分) 说明了如何通过识别要监视的 ClickHouse 指标类型来开始制定监视策略。

ClickHouse 监控必要的指标
------------------

第一种也是最广泛的关键指标是那些反映 ClickHouse 内部事件的指标。在这个类别中有几个特定类型的事件需要监控:

**「查询总数 (clickhouse.query.count) :」**  此数字表示 ClickHouse 集成中的查询总数。这是评估 ClickHouse 系统总体活动水平的关键指标。

**「插入行 (clickhouse.insert.rows):」** 此度量标准表示在所有表中插入的行数，并反映数据库中的活动级别以及数据库大小。

**「插入字节数 (clickhouse.insert.bytes):」**  所有表中插入的未压缩字节数, 也反映了活动级别和数据库大小。

![](https://img-blog.csdnimg.cn/img_convert/375a87d637e9f113668d053cbc8de6d2.png)

**「合并的行 (clickhouse.merge.rows):」**  后台合并读取的行，这表示合并前的行数。

**「合并未压缩的字节 (clickhouse.merge.bytes.uncompressed):」** 后台合并读取的未压缩字节, 表示合并前的数字。

![](https://img-blog.csdnimg.cn/img_convert/31996ca0db4ea413acfc07d6790b7520.png)

ClickHouse 网络指标
---------------

尽管 ClickHouse 不是联网工具，但它依靠网络来传输信息。因此，网络指标提供了一种评估 ClickHouse 性能和运行状况的有用方法。特别是，您将需要监控以下指标：

**「TCP 连接数 (clickhouse.connection.tcp.count):」** 与 TCP 服务器的连接总数, 帮助衡量 ClickHouse 的负载。

**「HTTP 连接数 (clickhouse.connection.http.count):」** 与 HTTP 服务器的连接数, 也反映了负载。

**「服务器间连接数 (clickhouse.connection.interserver.count):」** 此度量标准表示从其他副本到获取 Part 的连接数。它与整体系统负载没有直接关系，但是对于评估和优化 ClickHouse 的性能很有用。

![](https://img-blog.csdnimg.cn/img_convert/1edd1f9ec5515786545dd95731007fff.png)

Zookeeper 指标
------------

ClickHouse 使用 Apache Zookeeper 来帮助管理数据，因此监视 Zookeeper 对于确保 ClickHouse 正常运行很重要。您可以监视 ZooKeeper 指标 以帮助了解 ClickHouse 安装状态。在此类别中，要遵循的关键指标如下：

**「Zookeepr 监视数 (clickhouse.zk.watches):」** ZooKeeper 中 watches 的数量 (例如，事件订阅)

**「Zookeeper 等待时间 (clickhouse.zk.wait.time):」** 等待 ZooKeeper 操作所花费的时间

**「Zookeeper 请求数 (clickhouse.zk.requests):」** 正在处理的 ZooKeeper 的请求数。

![](https://img-blog.csdnimg.cn/img_convert/c30eb1b443126b9cfcd761a803494c85.png)

异步指标
----

以下异步指标是要监视的其他的基本 ClickHouse 指标：

**「最大 Part 数 (clickhouse.part.count.max):」** 此度量标准表示 ClickHouse 分区中活动 Part 的最大数量。如果 Part 处于活动状态，则在表中使用它；否则，它将被删除。合并后，不活动的 Part 部分仍然保留。

![](https://img-blog.csdnimg.cn/img_convert/a753ce1ce260433c674f8a0aa10dbec4.png)

数据 Part 指标
----------

除了上述异步 ClickHouse 指标之外，您还需要监视以下 MergeTree 数据 Part 指标：

**「活动 Part 数 (clickhouse.mergetree.table.parts):」** MergeTree 表中活动 Part 的数量;

**「行数 (clickhouse.mergetree.table.rows):」** MergeTree 表中的行数。

![](https://img-blog.csdnimg.cn/img_convert/87ae6f3c5b2c5ac582715b52adc25c1b.png)

副本状态
----

最后介绍并非最重要的副本状态衡量指标:

**「副本队列大小 (clickhouse.replica.queue.size):」** 此度量标准表示等待执行操作的队列的大小。在这种情况下，操作包括插入数据块，合并和某些其他操作。

![](https://img-blog.csdnimg.cn/img_convert/eff2d05433179c331cc72242aa53a500.png)

结论
--

有效的 ClickHouse 监视要求跟踪各种指标，以反映 ClickHouse 安装的可用性，活动级别和性能。上述那些仅代表要监视的最重要的指标。要获取所有 ClickHouse 指标的较长列表，可以参阅此文档页面。
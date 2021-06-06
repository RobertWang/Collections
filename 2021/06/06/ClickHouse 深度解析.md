> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1622876254&ver=3111&signature=yBE1c0ZvTq1rFrjEvqlMyzEIIBIIeL-cG-dvIk8hlkOirqkbjGKu7FihdrXfpc4Vn-p59VM6ouxSpS18VzO1n7NaK15iBY3X2wwRgWTROTzTs9tmo*G6Cd5SXOLjBSxx&new=1)

  

ClickHouse 由俄罗斯第一大搜索引擎 Yandex 于 2016 年 6 月发布，开发语言为 C++，ClickHouse 是一个面向联机分析处理 (OLAP) 的开源的面向列式存储的 DBMS，简称， 与 Hadoop、Spark 这些巨无霸组件相比，ClickHouse 很轻量级，查询性能非常好，使用之后会被它的性能折服，非常值得安利。

****-**** ****适用场景 -****

1. 日志数据行为分析

2. 标签画像的分析

3. 数据集市分层

4. 广告系统和实时竞价广告

5. 电商和金融行业

6. 实时监控和遥感测量

7. 商业智能

8. 在线游戏

9. 信息安全

10. 所有的互联网场景

****-**** ****特性**** ****-****

1. 真正的列式数据库

2. 数据压缩

3. 数据的磁盘存储

4. 多核并行处理

5. 多服务器分布式处理（数据保存在不同的 shard 上，每一个 shard 都由一组用于容错的副本组成，可并行查询所有 shard）

6. 向量引擎（按列的一部分进行处理，高效实用 CPU）

7. 实时的数据更新（支持在表中定义主键, 数据增量有序存储在 mergeTree 中）

8. 索引（按照主键对数据进行排序，毫秒内完成对数据的查找）

9. 适合在线查询

10. 支持近似计算（允许牺牲精度的情况下低延迟查询）

11. 支持数据复制和数据完整性（异步多主复制技术）

****-**** ****缺陷 -****

1. 没有完整的事务支持

2. 缺少高频率低延迟的修改或删除数据的能力

3. 不适合通过其检索单行的点查询

4. 联机事物处理

5. 二进制数据或文件存储

6. 键值对数据高效率访问请求

****-**** ****核心概念**** ****-****

**1. 表引擎（Engine）**

表引擎决定了数据在文件系统中的存储方式，常用的也是官方推荐的存储引擎是 MergeTree 系列，如果需要数据副本的话可以使用 ReplicatedMergeTree 系列，相当于 MergeTree 的副本版本。读取集群数据需要使用分布式表引擎 Distribute。

**2. 表分区（Partition）**

表中的数据可以按照指定的字段分区存储，每个分区在文件系统中都是都以目录的形式存在。常用时间字段作为分区字段，数据量大的表可以按照小时分区，数据量小的表可以在按照天分区或者月分区，查询时，使用分区字段作为 Where 条件，可以有效的过滤掉大量非结果集数据。

**3. 分片（Shard）**

一个分片本身就是 ClickHouse 一个实例节点，分片的本质就是为了提高查询效率，将一份全量的数据分成多份（片），从而降低单节点的数据扫描数量，提高查询性能。

**4. 复制集（Replication）**

简单理解就是相同的数据备份，在 CK 中通过复制集，我们实现保障了数据可靠性外，也通过多副本的方式，增加了 CK 查询的并发能力。这里一般有 2 种方式：（1）基于 ZooKeeper 的表复制方式；（2）基于 Cluster 的复制方式。由于我们推荐的数据写入方式本地表写入，禁止分布式表写入，所以我们的复制表只考虑 ZooKeeper 的表复制方案。

**5. 集群（Cluster）**

可以使用多个 ClickHouse 实例组成一个集群，并统一对外提供服务。

****-**** ****主要表引擎深入解析**** ****-****

**1.TinyLog**

最简单的表引擎，用于将数据存储在磁盘上，每列都存储在单独的压缩文件中，写入时，数据附加到文件末尾.

缺点：

（1）没有并发控制（没有做优化，同时写会数据会损坏，报错） 

（2）不支持索引 

（3）数据存储在磁盘上

优点：（1）小表节省空间 （2）数据写入，只查询，不做增删改操作

创建表：

![](https://mmbiz.qpic.cn/mmbiz_png/nLVgocKDbmIKhibat60pwQ6DFwGm8v9n7icmLqzjrxibz4D7k3nW9O16RmLLqh2yn2TAZcO861U4EzPB1ZVGXm3dQ/640?wx_fmt=png)

**2. Memory**

内存引擎，数据以未压缩的原始形式直接保存在内存中，服务器重启，数据会消失，读写操作不会相互阻塞，不支持索引。

建议上限 1 亿行的场景。

优点：简单查询下有非常高的性能表现（超过 10G/s）

创建表:

![](https://mmbiz.qpic.cn/mmbiz_png/nLVgocKDbmIKhibat60pwQ6DFwGm8v9n7lBXZQjHObYDmlxX3NLKbI867W9xPKwzHsffwdRRymJy6ocx2ibKmM1A/640?wx_fmt=png)

**3.Merge**

本身不存储数据，但可用于同时从任意多个其他的表中读取数据，读是自动并行的，不支持写入，读取时，那些真正被读取到数据的表的索引（如果有的话）会被占用, 默认是本地表，不能跨机器。

参数：一个数据库名和一个用于匹配表名的正则表达式

创建表：

![](https://mmbiz.qpic.cn/mmbiz_png/nLVgocKDbmIKhibat60pwQ6DFwGm8v9n7ibQMbDsDUcFjWNTZPIDKUmicwLVEV4kApOJIPySfMYqoYmdBN77j2new/640?wx_fmt=png)

**4.MergeTree**

ck 中最强大的表引擎 MergeTree(合并树) 和该系列（*MergeTree）中的其他引擎。

使用场景：有巨量数据要插入到表中，高效一批批写入数据片段，并希望这些数据片段在后台按照一定规则合并。相比在插入时不断修改（重写）数据进行存储，会高效很多。

优点：

（1）数据按主键排序 （2）可以使用分区（如果指定了主键）

（3）支持数据副本 （4）支持数据采样

创建表：

![](https://mmbiz.qpic.cn/mmbiz_png/nLVgocKDbmIKhibat60pwQ6DFwGm8v9n7b2Z5zNQVK0OBVH4D6LibFojEnGkSAGPunVRZuC9tA5qXjLX4yeyva3g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/nLVgocKDbmIKhibat60pwQ6DFwGm8v9n77zUYiaSAJkaJ97Y4siatTbIlGJEX7D5qCQpDrRDSLscelguegS9iaibiaZw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/nLVgocKDbmIKhibat60pwQ6DFwGm8v9n7x1xGU3Y4ickLOjF5vFA4IaLGjEkJgbN8qCJKIolazcic5rvgkUFlsQQw/640?wx_fmt=png)

插入数据：

![](https://mmbiz.qpic.cn/mmbiz_png/nLVgocKDbmIKhibat60pwQ6DFwGm8v9n7lR0DBn49a1cnZHRnXZRyX8dSbp2t3mPue1aZhsRG0sxQTS6TTkQbCw/640?wx_fmt=png)

**5.ReplacingMergeTree**

在 MergeTree 的基础上，增加了 “处理重复数据” 的功能，和 MergeTree 的不同之处在于他会删除具有相同主键的重复项，数据的去重只会在合并的过程中出现，合并会在未知的时间在后台进行，所以你无法预先做出计划，有一些数据可能仍未被处理，适用于在后台清除重复的数据以节省空间，但是不保证没有重复的数据出现。

创建表：

![](https://mmbiz.qpic.cn/mmbiz_png/nLVgocKDbmIKhibat60pwQ6DFwGm8v9n7erBatVxwFliaVNt6bfNaFFMw61KdS94Ml28rMmdNibTvRNm1qkmGqSow/640?wx_fmt=png)

**6.SummingMergeTree**

继承自 MergeTree, 区别在于，当合并 SummingMergeTree 表的数据片段时，ck 会把具有相同主键的行合并为一行，该行包含了被合并的行中具有数值数据类型的列的汇总值，如果主键的组合方式使得单个键值对应于大量的行，则可以显著的减少存储空间并加快数据查询的速度，对于不可加的列，会取一个最先出现的值。

![](https://mmbiz.qpic.cn/mmbiz_png/nLVgocKDbmIKhibat60pwQ6DFwGm8v9n73n26AluMw4exfx0hTLtCw8K1f7tpdzypkmN0nGyvV9RpxL61YhtBPQ/640?wx_fmt=png)

**7.Distributed(重点)**

分布式引擎，本身不存储数据，但可以在多个服务器上进行分布式查询，读是自动并行的，读取时，远程服务器的索引（如果有的话）会被使用。

![](https://mmbiz.qpic.cn/mmbiz_png/nLVgocKDbmIKhibat60pwQ6DFwGm8v9n7vdj7pYaST4iaYksTczLibjg2LrxxZurN4U75RF2ibyRibsagyXO3rlznrQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/nLVgocKDbmIKhibat60pwQ6DFwGm8v9n7w1ORR4DViaTDuYiaOD046H8lghDvnKkRk6nkIICJNKxCxYgmDOjM9oNg/640?wx_fmt=png)

****-**** ****最佳实践 -****

**1.HDFS 数据导入 Clickhouse**

**1.1 创建 hdfs_engine**

![](https://mmbiz.qpic.cn/mmbiz_png/nLVgocKDbmIKhibat60pwQ6DFwGm8v9n7thuKrQhzAAcWY892EO0WWp90NyBa5jPo9yuMib4y620SH2QIhYtIXdw/640?wx_fmt=png)

**1.2 File file**

![](https://mmbiz.qpic.cn/mmbiz_png/nLVgocKDbmIKhibat60pwQ6DFwGm8v9n7afJ6Q78mtMoFLTFSficicXwdAyaBy5Iq9XHIdribEZOGoc4yCcKBWCpxA/640?wx_fmt=png)

**1.3 数据查询**

![](https://mmbiz.qpic.cn/mmbiz_png/nLVgocKDbmIKhibat60pwQ6DFwGm8v9n7euxmvRPOkg9YFkibtJcO4Jic2lxoAkI0Rq6Dfm9MdDGicibmOrtulpADbg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/nLVgocKDbmIKhibat60pwQ6DFwGm8v9n7buMyrXibg8IbicuSuicQs4uicqbKNgzB1jUcdialwKl3OO7LgcYebxRw1DQ/640?wx_fmt=png)

**2. 每分钟亿级别海量用户行为日志实时自助查询**

**2.1 数据链路**

![](https://mmbiz.qpic.cn/mmbiz_jpg/nLVgocKDbmIKhibat60pwQ6DFwGm8v9n7VBuTtBiaec6SLh2KvwAiclicB0nlFjGHtdqI5ttJ9gu7zFia9hLUjG3AwA/640?wx_fmt=jpeg)

**2.2 系统实现**

在 flink 端动态设置 schema 信息，ETL 处理数据，动态生成宽表，数据存入 Clickhouse，按天分区，Clickhouse 使用 Distributed 表引擎，数据保留 7 天，避免数据过度膨胀，导致查询性能降低，使用 Redash 报表工具，分析人员可以写 SQL 自助查询，结果自定义图表展示。

**2.3 系统成果**

每分钟乙级的数据量，整个数据链路数据延迟在毫秒，数据查询响应在秒级别，动态设置 schema 生成宽表，做到整个系统的复用性，避免重复开发，查询性能比 Hive 快几百倍，满足了实时性的要求。

****-**** ****大厂适用场景 -****

**1. 头条：**用户行为分析系统，上报日志 大宽表，减少 join，增加 map 数据类型，展平模型，支持动态 scheam

![](https://mmbiz.qpic.cn/mmbiz_png/nLVgocKDbmIKhibat60pwQ6DFwGm8v9n7ZTIuF4mroGUI7rczTDOXV7hmVlVRRPKwEdJFabfjHibd2ibHgAicOVL6Q/640?wx_fmt=png) 

**2. 腾讯：**游戏数据分析

**3. 携程：**内部从 18 年 7 月份开始接入试用，目前 80% 的业务都跑在 ClickHouse 上。每天数据增量十多亿，近百万次查询请求

**4. 快手：**内部也在使用 ClickHouse，存储总量大约 10PB， 每天新增 200TB， 90% 查询小于 3S

**5. 国外：**Yandex 内部有数百节点用于做用户点击行为分析，CloudFlare、Spotify 等头部公司也在使用

****-**** ****结语 -****

ClickHouse 优秀的性能，适合实时 OLAP 场景，是一颗冉冉升起的星星，社区活跃度高，未来在大数据领域能发挥更闪耀的价值，非常期待。

版权声明：本文为 CSDN 博主「追风 dylan」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。

原文链接：

https://blog.csdn.net/weixin_38255219/article/details/106809690
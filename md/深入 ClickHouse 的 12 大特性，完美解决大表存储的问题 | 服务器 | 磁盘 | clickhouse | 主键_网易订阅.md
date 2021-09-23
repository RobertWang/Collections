> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.163.com](https://www.163.com/dy/article/FCHRD80F0525KFNV.html)

> 深入 ClickHouse 的 12 大特性，完美解决大表存储的问题, 服务器, 磁盘, clickhouse, 主键

前言

ClickHouse 作为联机分析（OLAP）列式数据库的佼佼者，其大批量读写性能得到了开发者的青睐。很多大数据存储业务场景开始都使用 Clickhouse 解决，特别是涉及到单表过亿的数据的时候，ClickHouse 的性能完爆任何数据库。**那 ClickHouse 的高性能是如何实现的呢？ClickHouse 具有哪些特性呢？本文将重点讲解这两个问题**。

![](https://nimg.ws.126.net/?url=http%3A%2F%2Fdingyue.ws.126.net%2F2020%2F0513%2F5091609ej00qa9xkk000fc000hs00b4c.jpg&thumbnail=650x2147483647&quality=80&type=jpg)  
真正的列式数据库管理系统  

在一个真正的列式数据库管理系统中，除了数据本身外不应该存在其他额外的数据。这意味着为了避免在值旁边存储它们的长度，数据库必须支持固定长度数值类型。**例如，10 亿个 UInt8 类型的数据在未压缩的情况下大约消耗 1GB 左右的空间，如果不是这样的话，这将对 CPU 的使用产生强烈影响**。即使是在未压缩的情况下，紧凑的存储数据也是非常重要的，因为解压缩的速度主要取决于未压缩数据的大小。

这是非常值得注意的，因为在一些其他系统中也可以将不同的列分别进行存储，但由于对其他场景进行的优化，使其无法有效的处理分析查询。**例如： HBase，BigTable，Cassandra，HyperTable。在这些系统中，我们可以得到每秒数十万的吞吐能力，但是无法得到每秒几亿行的吞吐能力。ClickHouse 不单单是一个数据库， 它是一个数据库管理系统。因为它允许在运行时创建表和数据库、加载数据和运行查询，而无需重新配置或重启服务。**

数据压缩

在一些列式数据库管理系统中 (例如：InfiniDB CE 和 MonetDB) 并没有使用数据压缩。但是, 若想达到比较优异的性能，数据压缩确实起到了至关重要的作用。

![](https://nimg.ws.126.net/?url=http%3A%2F%2Fdingyue.ws.126.net%2F2020%2F0513%2F1e4fca89j00qa9xkk000nc000hs007oc.jpg&thumbnail=650x2147483647&quality=80&type=jpg)  
数据的磁盘存储  

许多的列式数据库 (如 SAP HANA, Google PowerDrill) 只能在内存中工作，这种方式会造成比实际更多的设备预算。**ClickHouse 被设计用于工作在传统磁盘上的系统，它提供每 GB 更低的存储成本，但如果有可以使用 SSD 和内存，它也会合理的利用这些资源**。

多核心并行处理

ClickHouse 会使用服务器上一切可用的资源，从而以最自然的方式并行处理大型查询。

![](https://nimg.ws.126.net/?url=http%3A%2F%2Fdingyue.ws.126.net%2F2020%2F0513%2F49eb211bj00qa9xkk000sc000hs009pc.jpg&thumbnail=650x2147483647&quality=80&type=jpg)  
多服务器分布式处理  

上面提到的列式数据库管理系统中，几乎没有一个支持分布式的查询处理。

**在 ClickHouse 中，数据可以保存在不同的 shard 上，每一个 shard 都由一组用于容错的 replica 组成，查询可以并行地在所有 shard 上进行处理。这些对用户来说是透明的**

支持 SQL

ClickHouse 支持基于 SQL 的声明式查询语言，该语言大部分情况下是与 SQL 标准兼容的。

**支持的查询包括 GROUP BY，ORDER BY，IN，JOIN 以及非相关****子****查询。不支持窗口函数和相关****子****查询。**

向量引擎

为了高效的使用 CPU，数据不仅仅按列存储，同时还按向量 (列的一部分) 进行处理，这样可以更加高效地使用 CPU。

![](https://nimg.ws.126.net/?url=http%3A%2F%2Fdingyue.ws.126.net%2F2020%2F0513%2F465e40abj00qa9xkk000rc000hs00bvc.jpg&thumbnail=650x2147483647&quality=80&type=jpg)  

实时的数据更新

ClickHouse 支持在表中定义主键。**为了使查询能够快速在****主键****中进行范围查找，数据总是以增量的方式有序的存储在 MergeTree 中**。因此，数据可以持续不断地高效的写入到表中，并且写入的过程中不会存在任何加锁的行为。

索引

按照主键对数据进行排序，这将帮助 ClickHouse 在几十毫秒以内完成对数据特定值或范围的查找。

适合在线查询

在线查询意味着在没有对数据做任何预处理的情况下以极低的延迟处理查询并将结果加载到用户的页面中。

支持近似计算

ClickHouse 提供各种各样在允许牺牲数据精度的情况下对查询进行加速的方法：

**用于近似计算的各类聚合函数，如：distinct values, medians, quantiles**。基于数据的部分样本进行近似查询。这时，仅会从磁盘检索少部分比例的数据。不使用全部的聚合条件，通过随机选择有限个数据聚合条件进行聚合。这在数据聚合条件满足某些分布条件下，在提供相当准确的聚合结果的同时降低了计算资源的使用。

![](https://nimg.ws.126.net/?url=http%3A%2F%2Fdingyue.ws.126.net%2F2020%2F0513%2F5ea22a09j00qa9xkk000cc000hs00aqc.jpg&thumbnail=650x2147483647&quality=80&type=jpg)  
支持数据复制和数据完整性  

ClickHouse 使用异步的多主复制技术。当数据被写入任何一个可用副本后，系统会在后台将数据分发给其他副本，以保证系统在不同副本上保持相同的数据。在大多数情况下 ClickHouse 能在故障后自动恢复，在一些少数的复杂情况下需要手动恢复。

特别声明：以上内容 (如有图片或视频亦包括在内) 为自媒体平台 “网易号” 用户上传并发布，本平台仅提供信息存储服务。

Notice: The content above (including the pictures and videos if any) is uploaded and posted by a user of NetEase Hao, which is a social media platform and only provides information storage services.
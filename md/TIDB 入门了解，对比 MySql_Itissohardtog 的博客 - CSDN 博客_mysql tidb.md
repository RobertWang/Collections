> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/Itissohardtog/article/details/103197730?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~aggregatepage~first_rank_ecpm_v1~rank_v31_ecpm-1-103197730.pc_agg_new_rank&utm_term=mysql+tidb+%E5%AF%B9%E6%AF%94&spm=1000.2123.3001.4430)

[TIDB](https://so.csdn.net/so/search?q=TIDB) 是什么？MySql 是什么？
===========================================================

**TiDB** 是 PingCAP 公司受 Google Spanner / F1 论文启发而设计的开源分布式 HTAP (Hybrid [Transactional](https://so.csdn.net/so/search?q=Transactional) and Analytical Processing) 数据库，结合了传统的 [RDBMS](https://blog.csdn.net/belalds/article/details/84029384) 和 [NoSQL](https://blog.csdn.net/belalds/article/details/84029384) 的最佳特性。TiDB 兼容 MySQL，支持无限的水平扩展，具备**强一致性**和**高可用性**。TiDB 的目标是为 OLTP(Online Transactional [Processing](https://so.csdn.net/so/search?q=Processing)) 和 OLAP (Online Analytical Processing) 场景提供一站式的解决方案。

TiDB 是一个分布式 [NewSQL](https://blog.csdn.net/belalds/article/details/84029384) 数据库。它支持水平弹性扩展、ACID 事务、标准 SQL、MySQL 语法和 MySQL 协议，具有数据强一致的高可用特性，是一个不仅适合 OLTP 场景还适合 OLAP 场景的混合数据库。

**MySQL** 是一个关系型数据库管理系统，由瑞典 MySQL AB 公司开发，目前属于 Oracle 旗下产品。MySQL 是最流行的关系型数据库管理系统之一，在 WEB 应用方面，MySQL 是最好的 RDBMS (Relational Database Management System，关系数据库管理系统) 应用软件之一。  
MySQL 是一种关系数据库管理系统，关系数据库将数据保存在不同的表中，而不是将所有数据放在一个大仓库内，这样就增加了速度并提高了灵活性。  
MySQL 所使用的 SQL 语言是用于访问数据库的最常用标准化语言。MySQL 软件采用了双授权政策，分为社区版和商业版，由于其体积小、速度快、总体拥有成本低，尤其是开放源码这一特点，一般中小型网站的开发都选择 MySQL 作为网站数据库。

TIDB 适用场景；MySql 适用场景：
=====================

###### TIDB 适用场景：

1.  原业务的 MySql 的业务逻辑遇到单机容量或者性能瓶颈时，可以考虑适用 TiDB 无缝替换 MySql。
2.  大数据量下，MySql 复杂查询很慢。
3.  大数据量下，数据增长很快，接近单机处理的极限，不想分库分表或者使用数据库中间件等对业务侵入性较大、对业务有约束的 Sharding 方案。
4.  大数据量下，有高并发实时写入、实时查询、实时统计分析的需求。
5.  有分布式事务、多数据中心的数据 100% 强一致性、auto-failover 的高可用的需求。

###### MySql 适用场景：

1.  Web 网站系统
2.  日志记录系统
3.  数据仓库系统
4.  嵌入式系统

###### TIDB 不适用场景：

1.  单机 MySql 能满足的场景用不到 TiDB。
2.  数据条数少于 5000W 的场景下通常用不到 TiDB，TiDB 是为大规模的数据场景设计的。
3.  如果你的应用数据量小（所有数据千万级别行以下），且没有高可用、强一致性或者多数据中心复制等要求，那么就不适合适用 TiDB。

###### MySql 不适用场景

1.  单机 MySql 无法满足的场景。
2.  数据条数大于 5000W 的场景下 MySql 通常无法满足。
3.  应用数据量特别大，请求次数频繁。

TIDB 和 MySql 的核心特点：
===================

###### TIDB 核心特点：

1.  高度兼容 MySql  
    大多数情况下，无需修改代码即可从 MySql 轻松迁移至 TiDB，分库分表后的 MySql 集群亦可通过 TiDB 工具进行实时迁移。
    
2.  水平弹性扩展  
    通过简单地增加新节点即可实现 TiDB 的水平扩展，按需扩展吞吐或存储，轻松应对高并发、海量数据场景。
    
3.  分布式事务  
    TiDB 100% 支持标准的 ACID 事务。
    
4.  真正金融级高可用  
    相比于传统主从（M-S）复制方案，基于 Raft 的多数派选举协议可以提供金融级的 100% 数据强一致性保证，且在不丢失大多数读本的前提下，可以实现故障的自动恢复（auto-failover），无需人工介入。
    
5.  一站式 HTAP 解决方案  
    TiDB 作为典型的 OLTP（联机事务处理）行存数据库，同事兼具强大的 OLAP（联机分析处理）性能，配合 TiSpark，可提供一站式 HTAP 解决方案，一份存储同时处理 OLTP&OLAP 无需传统繁琐的 [ETL](https://www.cnblogs.com/yjd_hycf_space/p/7772722.html) 过程。
    
6.  云原生 SQL 数据库  
    TiDB 是为云而设计的数据库，同 [Kubernetes](http://www.dockone.io/article/932) 深度耦合，支持公有云、私有云和混合云，使部署、配置和维护变得十分简单。  
    TiDB 的设计目标是 100% 的 OLTP 场景和 80% 的 OLAP 场景，更复杂的 OLAP 分析可以通过 TiSpark 项目来完成。TiDB 对业务没有任何侵入性，能优雅的替换传统的数据库中间件、数据库分表分库等 Sharding 方案。同时它也让开发运维人员不用关注数据库 Scale 的细节问题，专注于业务开发，极大的提升研发的生产力。
    

###### MySql 核心特点：

1.  开源、免费、成本低
2.  性能高、移植性也好
3.  体积小，便于安装

TiDB 和 MySql 的差异：
=================

###### TiDB 事务和 MySql 事物的差异

###### MySql 事务和 TiDB 事务对比

![](https://img-blog.csdnimg.cn/20191122160405987.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0l0aXNzb2hhcmR0b2c=,size_16,color_FFFFFF,t_70)

在 TiDB 中执行的事务 b，返回影响条数是 1（认为已经修改成功），但是提交后查询，status 却不是事务 b 修改的值，而是事务 a 修改的值。

可见，MySql 事务和 TiDB 事务存在这样的差异：**MySql 事务中，可以通过影响行数，作为写入（或修改）是否成功的依据；而 TiDB 中，这却是不可行的！**

作为开发者我们需要考虑下面的问题：

同步 RPC 调用中，如果需要严格依赖影响条数以确认返回值，那将如何是好？  
多表操作中，如果需要严格依赖某个主表数据更新结果，作为是否更新（或写入）其他表的判断依据，那将如何是好？

原因分析及解决方案

*   **对于 MySql，当更新某条记录时，会先获取该记录对应的行级锁（排它锁），获取成功则进行后续的事务操作，获取失败则阻塞等待。**
*   **对于 TiDB，使用 Percolator 事务模型：可以理解为乐观锁实现，事务开启、事务中都不会加锁，而是在提交时才加锁。**

其简要流程如下：  
![](https://img-blog.csdnimg.cn/20191122161836301.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0l0aXNzb2hhcmR0b2c=,size_16,color_FFFFFF,t_70)  
在事务提交的 PreWrite 阶段，当 “锁检查” 失败时：如果开启冲突重试，事务提交将会进行重试；如果未开启冲突重试，将会抛出写入冲突异常。

可见，对于 MySql，由于在写入操作时加上了排他锁，变相将并行事务从逻辑上串行化；而对于 TiDB，属于乐观锁模型，在事务提交时才加锁，并使用事务开启时获取的 “全局时间戳” 作为 “锁检查” 的依据。

所以，在业务面避免 TiDB 事务差异的本质在于避免锁冲突，即，当事务执行时，不产生别的事务时间戳（无其他事务并行）。处理方式为事务串行化。

TiDB 事务串行化  
在业务层，可以借助分布式锁，实行串行化处理，如下：  
![](https://img-blog.csdnimg.cn/20191122171203937.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0l0aXNzb2hhcmR0b2c=,size_16,color_FFFFFF,t_70)  
基于 Spring 和分布式锁的事务管理器拓展  
在 Spring 生态下，spring-tx 中定义了统一的事务管理器接口：PlatformTransactionManager，其中有获取事务（getTransaction）、提交（commit）、回滚（rollback）三个基本方法；使用装饰器模式，事务串行化组件可做如下设计：  
![](https://img-blog.csdnimg.cn/20191122171817862.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0l0aXNzb2hhcmR0b2c=,size_16,color_FFFFFF,t_70)  
其中，关键点有：

超时时间：为避免死锁，锁必须有超时时间；为避免锁超时导致事务并行，事务必须有超时时间，而且锁超时时间必须大于事务超时时间（时间差最好在秒级）。

加锁时机：TiDB 中 “锁检查” 的依据是事务开启时获取的“全局时间戳”，所以加锁时机必须在事务开启前。

事务模板接口设计  
隐藏复杂的事务重写逻辑，暴露简单友好的 API：

```
public interface LockTransactionTemplate {

	/**
	 * 阻塞模式事务模板
	 * @param action 事务回调action
	 * @return
	 */
	public <T> T lockableExecute(TransactionCallback<T> action, LockKeys<?>...LockKeys);
}

```

```
/**
	 * 非阻塞模式事务模板
	 * @param action 事务回调action
	 * @return
	 */
	public <T> T lockableExecute(NonBlockingAcquireCallback<T> action, LockKeys<?>...LockKeys);

```

TiDB 查询和 MySQL 的差异  
在 TiDB 使用过程中，偶尔会有这样的情况，某几个字段建立了索引，但是查询过程还是很慢，甚至不经过索引检索。

索引混淆型（举例）  
表结构：

```
CREATE TABLE `t_test` (
      `id` bigint(20) NOT NULL DEFAULT '0' COMMENT '主键id',
      `a` int(11) NOT NULL DEFAULT '0' COMMENT 'a',
      `b` int(11) NOT NULL DEFAULT '0' COMMENT 'b',
      `c` int(11) NOT NULL DEFAULT '0' COMMENT 'c',
      PRIMARY KEY (`id`),
      KEY `idx_a_b` (`a`,`b`),
      KEY `idx_c` (`c`)
    ) ENGINE=InnoDB;

```

查询：如果需要查询 (a=1 且 b=1）或 c=2 的数据，在 MySQL 中，sql 可以写为：

```
SELECT id from t_test where (a=1 and b=1) or (c=2);

```

MySQL 做查询优化时，会检索到 idx_a_b 和 idx_c 两个索引；但是在 TiDB（v2.0.8-9）中，这个 sql 会成为一个慢 SQL，需要改写为：

```
SELECT id from t_test where (a=1 and b=1) UNION SELECT id from t_test where (c=2);

```

小结：导致该问题的原因，可以理解为 TiDB 的 sql 解析还有优化空间。

冷热数据型（举例）  
表结构：

```
CREATE TABLE `t_job_record` (
      `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键id',
      `job_code` varchar(255) NOT NULL DEFAULT '' COMMENT '任务code',
      `record_id` bigint(20) NOT NULL DEFAULT '0' COMMENT '记录id',
      `status` tinyint(3) NOT NULL DEFAULT '0' COMMENT '执行状态:0 待处理',
      `execute_time` bigint(20) NOT NULL DEFAULT '0' COMMENT '执行时间（毫秒）',
      PRIMARY KEY (`id`),
      KEY `idx_status_execute_time` (`status`,`execute_time`),
      KEY `idx_record_id` (`record_id`)
    ) ENGINE=InnoDB COMMENT='异步任务job'

```

数据说明：

*   冷数据，status=1 的数据（已经处理过的数据）；
    
*   热数据，status=0 并且 execute_time<= 当前时间的数据。
    

慢查询：对于热数据，数据量一般不大，但是查询频率度很高，假设当前（毫秒级）时间为：1546361579646，则在 MySql 中，查询 sql 为：

```
SELECT * FROM t_job_record where status=0 and execute_time<= 1546361579646

```

这个在 MySQL 中很高效的查询，在 TiDB 中虽然也可从索引检索，但其耗时却不尽人意（百万级数据量，耗时百毫秒级）。

原因分析：在 TiDB 中，底层索引结构为 LSM-Tree，如下图：  
![](https://img-blog.csdnimg.cn/20191122175655687.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0l0aXNzb2hhcmR0b2c=,size_16,color_FFFFFF,t_70)  
当从内存级的 C0 层查询不到数据时，会逐层扫描硬盘中各层；且 merge 操作为异步操作，索引数据更新会存在一定的延迟，可能存在无效索引。由于逐层扫描和异步 merge，使得查询效率较低。

优化方式：尽可能缩小过滤范围，比如结合异步 job 获取记录频率，在保证不遗漏数据的前提下，合理设置 execute_time 筛选区间，例如 1 小时，sql 改写为：

```
SELECT * FROM t_job_record  where status=0 and execute_time>1546357979646  and execute_time<= 1546361579646

```

优化效果：耗时 10 毫秒级别（以下）。

关于查询的启发  
在基于 TiDB 的业务开发中，先摒弃传统关系型数据库带来的对 sql 先入为主的理解或经验，谨慎设计每一个 sql，如 DBA（数据库管理员）所提倡：**设计 sql 时务必关注执行计划，必要时请教 DBA。**

和 MySql 相比，TiDB 的底层存储和结构决定了其特殊性和差异性；但是，TiDB 支持 MySql 协议，他们也存在一些共同之处，比如在 TiDB 中使用 “预编译” 和“批处理”，同样可以获得一定的性能提升。

服务端预编译  
在 MySql 中，可以使用 PREPARE stmt_name FROM preparable_stm 对 sql 语句进行预编译，然后使用 EXECUTE stmt_name [USING @var_name [,@var_name]…] 执行预编译语句。如此，同一 sql 的多次操作，可以获得比常规 sql 更高得性能。

mysql-jdbc 源码中，实现了标准的 Statement 和 PreparedStatement 的同时，还有一个 ServerPreparedStatement 实现，ServerPreparedStatement 属于 PreparedStatement 的拓展，三者对比如下：  
![](https://img-blog.csdnimg.cn/20191123090921172.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0l0aXNzb2hhcmR0b2c=,size_16,color_FFFFFF,t_70)  
容易发现，PreparedStatement 和 Statement 的区别主要区别在于参数处理，而对于发送数据包，调用服务器端的处理逻辑是一样（或类似）的；经测试，二者速度相当。其实 PreparedStatement 并不是服务器端预处理的；ServerPreparedStatement 才是真正的服务器端预处理，速度也较 PreparedStatement 快；其使用场景一般是：频繁的数据库访问，sql 数量有限（有缓存淘汰策略，使用不宜会导致两次 IO）。

批处理：  
对于多条数据写入，常用 sql 为 insert…values(…),(…); 而对于多条数据更新，亦可以使用 update…case…when…then…end 来减少 IO 次数，但他们都有一个特点，数据条数越多，sql 越复杂，sql 解析成本也更高，耗时增长可能高于线性增长。而批处理，可以复用一条简单的 sql，实现批量数据的写入或更新，为系统带来更低、更稳定的耗时。

对于批处理，作为客户端，java.sql.Statement 主要定义了两个接口方法，addBatch 和 executeBatch 来支持批处理。

批处理的简要流程说明如下：

![](https://img-blog.csdnimg.cn/20191123105753489.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0l0aXNzb2hhcmR0b2c=,size_16,color_FFFFFF,t_70)  
经业务中实践，使用批处理方式的写入（或更新），比常规 insert … values(…),(…)（或 update … case … when… then… end）性能更稳定，耗时也更低。

TiDB 和 MySql 的整体架构：
===================

###### TiDB 整体架构：

![](https://img-blog.csdnimg.cn/20191123115010771.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0l0aXNzb2hhcmR0b2c=,size_16,color_FFFFFF,t_70)

### TiDB Server

TiDB Server 负责接收 SQL 请求，处理 SQL 相关的逻辑，并通过 PD 找到存储计算所需数据的 TiKV 地址，与 TiKV 交互获取数据，最终返回结果。 TiDB Server 是无状态的，其本身并不存储数据，只负责计算，可以无限水平扩展，可以通过负载均衡组件（如 LVS、HAProxy 或 F5）对外提供统一的接入地址。TiDB 采用 go 语言编写。

### PD Server

Placement Driver (简称 PD) 是整个集群的管理模块，其主要工作有三个： 一是存储集群的元信息（某个 Key 存储在哪个 TiKV 节点）；二是对 TiKV 集群进行调度和负载均衡（如数据的迁移、Raft group leader 的迁移等）；三是分配全局唯一且递增的事务 ID。PD 通过内嵌 etcd 来支持数据分布和容错。PD 采用 go 语言编写。

PD 是一个集群，需要部署奇数个节点，一般线上推荐至少部署 3 个节点。PD 在选举的过程中无法对外提供服务，这个时间大约是 3 秒。

### TiKV Server

TiKV Server 负责存储数据，从外部看 TiKV 是一个分布式的提供事务的 Key-Value 存储引擎，基于 Google Spanner, HBase 设计，但脱离了底层较为复杂的 HDFS。通过 RocksDB 将 key-value 值存在本地地盘，使用 Raft 协议做复制，保持数据的一致性和容灾。存储数据的基本单位是 Region，每个 Region 负责存储一个 Key Range （从 StartKey 到 EndKey 的左闭右开区间）的数据，每个 TiKV 节点会负责多个 Region 。TiKV 使用 Raft 协议做复制，保持数据的一致性和容灾。副本以 Region 为单位进行管理，不同节点上的多个 Region 构成一个 Raft Group，互为副本。数据在多个 TiKV 之间的负载均衡由 PD 调度，这里也是以 Region 为单位进行调度。TiKV 采用 Rust 语言编写。

### TiSpark

TiSpark 作为 TiDB 中解决用户复杂 OLAP 需求的主要组件，将 Spark SQL 直接运行在 TiDB 存储层上，同时融合 TiKV 分布式集群的优势，并融入大数据社区生态。至此，TiDB 可以通过一套系统，同时支持 OLTP 与 OLAP，免除用户数据同步的烦恼。

### TiDB Operator

TiDB Operator 提供在主流云基础设施（Kubernetes）上部署管理 TiDB 集群的能力。它结合云原生社区的容器编排最佳实践与 TiDB 的专业运维知识，集成一键部署、多集群混部、自动运维、故障自愈等能力，极大地降低了用户使用和管理 TiDB 的门槛与成本。

### syncer syncer 通过 binlog 能方便地将 MySQL 的数据增量导入到 TiDB。

![](https://img-blog.csdnimg.cn/2019112312001011.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0l0aXNzb2hhcmR0b2c=,size_16,color_FFFFFF,t_70)

### TiDB-Binlog 将数据备份到其他数据库，也可以 TiDB 集群故障时恢复。

![](https://img-blog.csdnimg.cn/20191123141005404.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0l0aXNzb2hhcmR0b2c=,size_16,color_FFFFFF,t_70)

###### MySql 整体架构：

![](https://img-blog.csdnimg.cn/20191123141548413.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0l0aXNzb2hhcmR0b2c=,size_16,color_FFFFFF,t_70)

### MySQL 向外提供的交互接口（Connectors）

Connectors 组件，是 MySQL 向外提供的交互组件，如 java,.net,php 等语言可以通过该组件来操作 SQL 语句，实现与 SQL 的交互。

### 管理服务组件和工具组件 (Management Service & Utilities)

提供对 MySQL 的集成管理，如备份 (Backup), 恢复(Recovery), 安全管理(Security) 等

### 连接池组件 (Connection Pool)

负责监听对客户端向 MySQL Server 端的各种请求，接收请求，转发请求到目标模块。每个成功连接 MySQL Server 的客户请求都会被  
创建或分配一个线程，该线程负责客户端与 MySQL Server 端的通信，接收客户端发送的命令，传递服务端的结果信息等。

### SQL 接口组件 (SQL Interface)

接收用户 SQL 命令，如 DML,DDL 和存储过程等，并将最终结果返回给用户。

### 查询分析器组件 (Parser)

首先分析 SQL 命令语法的合法性，并尝试将 SQL 命令分解成数据结构，若分解失败，则提示 SQL 语句不合理。

### 优化器组件（Optimizer）

对 SQL 命令按照标准流程进行优化分析。

### 缓存主件（Caches & Buffers）

缓存和缓冲组件

### MySQL 存储引擎

1.  什么是 MySQL 存储引擎？  
    MySQL 属于关系型数据库，而关系型数据库的存储是以表的形式进行的，对于表的创建，数据的存储，检索，更新等都是由 MySQL  
    存储引擎完成的，这也是 MySQL 存储引擎在 MySQL 中扮演的重要角色。  
    研究过 SQL Server 和 Oracle 的读者可能很清楚，这两种数据库的存储引擎只有一个，而 MySQL 的存储引擎种类比较多，如 MyISAM 存储  
    引擎，InnoDB 存储引擎和 Memory 存储引擎。  
    MySQL 之所以有多种存储引擎，是因为 MySQL 的开源性决定的。MySQL 存储引擎，从种类上来说，大致可归结为官方存储引擎和第三  
    方存储引起。MySQL 的开源性，允许第三方基于 MySQL 骨架，开发适合自己业务需求的存储引擎。
    
2.  MySQL 存储引擎作用？  
    MySQL 存储引擎在 MySQL 中扮演重要角色，其作比较重要作用，大致归结为如下两方面：  
    作用一：**管理表创建，数据检索，索引创建等**  
    作用二：**满足自定义存储引擎开发。**
    
3.  MySQL 引擎种类？  
    不同种类的存储引擎，在存储表时的存储引擎表机制也有所不同，从 MySQL 存储引擎种类上来说，可以分为官方存储引擎和第三方存储引擎。  
    当前，也存在多种 MySQL 存储引擎，如 MyISAM 存储引擎，InnoDB 存储引擎，NDB 存储引擎，Archive 存储引擎，Federated 存储引擎，Memory  
    存储引擎，Merge 存储引擎，Parter 存储引擎，Community 存储引擎，Custom 存储引擎和其他存储引擎。  
    其中，比较常用的存储引擎包括 InnoDB 存储引擎，MyISAM 存储引擎和 Momery 存储引擎。
    
4.  几种典型 MySQL 存储引擎比较？
    

![](https://img-blog.csdnimg.cn/20191123142347622.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0l0aXNzb2hhcmR0b2c=,size_16,color_FFFFFF,t_70)

### 物理文件（File System）

实际存储 MySQL 数据库文件和一些日志文件等的系统，如 Linux，Unix,Windows 等。

TiDB 原理和 MySql 原理
=================

###### [TiDB 原理](http://www.sohu.com/a/138003618_736949)

![](https://img-blog.csdnimg.cn/20191123143230865.png)  
TiDB 架构是 SQL 层和 KV 存储层分离，相当于 InnoDB 插件存储引擎与 MySQL 的关系。从下图可以看出整个系统是高度分层的，最底层选用了当前比较流行的存储引擎 RocksDB，RockDB 性能很好但是是单机的，为了保证高可用所以写多份，上层使用 Raft 协议来保证单机失效后数据不丢失不出错。保证有了比较安全的 KV 存储的基础上再去构建多版本，再去构建分布式事务，这样就构成了存储层 TiKV。有了 TiKV，TiDB 层只需要实现 SQL 层，再加上 MySQL 协议的支持，应用程序就能像访问 MySQL 那样去访问 TiDB 了。

###### [MySql 原理](https://www.2cto.com/database/201801/715716.html)

在上面的 "MySql 整体架构" 基本上都讲过了，再看一下别人写的吧！  
[时光传送门](https://www.2cto.com/database/201801/715716.html)
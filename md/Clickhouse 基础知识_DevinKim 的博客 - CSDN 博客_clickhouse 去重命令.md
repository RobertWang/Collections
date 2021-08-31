> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/Yanxu_Jin/article/details/108829848)

Clickhouse 基础知识
===============

本文参考以下博文：

https://clickhouse.tech/docs/zh/ 【clickhouse 中文社区文档】

https://www.jianshu.com/p/5f7809b1965e

https://blog.csdn.net/jmx_bigdata/article/details/108568278

https://blog.csdn.net/jmx_bigdata/article/details/108719569

一. Clickhouse 简介
----------------

Clickhouse 是一个开源的面向联机分析处理（OLAP, On-Line Analytical Processing）的列式存储数据库管理系统。

<table><thead><tr><th>优点</th><th>缺点</th></tr></thead><tbody><tr><td>写入快、查询快</td><td>不支持事务</td></tr><tr><td>SQL 支持</td><td>不适合典型的 K/V 存储</td></tr><tr><td>简单方便，不依赖 Hadoop 技术栈</td><td>不适合 Blob/Document 存储</td></tr><tr><td>支持线性扩展</td><td>不支持完整的 Update/Delete 操作</td></tr><tr><td>深度列存储</td><td>非跨平台</td></tr><tr><td>向量化查询执行</td><td>并非查询资源控制不好处理</td></tr><tr><td>数据压缩</td><td>不支持二级索引</td></tr><tr><td>并行和分布式查询</td><td></td></tr><tr><td>实时数据更新</td><td></td></tr></tbody></table>

### 1. ClickHouse 的大数据处理架构优点

<table><thead><tr><th>ClickHouse 的大数据处理架构</th><th>典型的大数据处理架构</th></tr></thead><tbody><tr><td>链路简单</td><td>数据处理流程复杂，链路多</td></tr><tr><td>数据准实时入库</td><td>Spark/Flink 占用资源多</td></tr><tr><td>清单数据随用随查</td><td>基于原始数据的即席查询延迟大</td></tr><tr><td>基于原始数据的统计分析效率高</td><td>清单数据查询成本高</td></tr><tr><td>硬件投资少</td><td>硬件投资多</td></tr><tr><td>维护简单</td><td>维护成本高</td></tr></tbody></table>

### 2. ClickHouse 与 Hbase、Elasticsearch 对比

**资源占用**

*   Hbase 依赖 hadoop，占用资源一般。
*   Elasticsearch 基于内存，占用资源多。
*   ClickHouse 占用资源少，约为 Elasticsearch 的 5%-10%。

**清单查询**

*   Hbase 适用于 rowkey 返回少量数据的查询，非 rowkey 字段需全表扫描。
*   Elasticsearch 支持索引字段查询，速度快。
*   ClickHouse 支持针对所有字段的清单查询，速度快。建议使用主键索引查询，节约资源。

**数据分析**

*   Hbase 不适合用于分析的数据源。
*   Elasticsearch 分析功能有限，不能多表关联。
*   ClickHouse 适合 OLAP 分析，速度快。

**查询语法**

*   Hbase 专用 API，默认不支持 SQL。
    
*   Elasticsearch 专用 DSL，有限的 SQL 支持。
    
*   ClickHouse 支持复杂 SQL。
    

**读写阻塞**

*   Hbase 高并发读写容易产生 region 热点。
*   Elasticsearch 高并发读写容易产生热点。
*   ClickHouse 读写相互不影响。

**可维护性**

*   Hbase 维护复杂，元数据损坏可能导致数据不可用。
*   Elasticsearch 维护一般。
*   ClickHouse 易于扩展，数据容易恢复。

**硬件成本**

*   Hbase 依赖 Hadoop 生态。
    
*   ElasticsearchCPU 密集型, 单节点内存不超过 64GB，超出浪费。
    
*   ClickHouseCPU 密集型，能充分利用主机资源。
    

**集群架构**

*   Hbase 和 Elasticsearch 为主从架构。
*   ClickHouse 为多主架构，完全自主实现高可用。

二. 单机版安装
--------

### 1. 系统检查

```
# 查看 Linux 版本
[root@ck01 ~]# cat /etc/redhat-release
CentOS Linux release 7.7.1908 (Core)

# 检查系统是否支持clickhouse安装
[root@ck01 ~]# grep -q sse4_2 /proc/cpuinfo && echo “SSE 4.2 supported” || echo “SSE 4.2 not supported”
“SSE 4.2 supported”

```

若显示为 SSE4.2suported 则可以继续安装。

### 2. 下载与安装

第三方机构 Altinity 提供了完整的 rpm 包，支持在 Centos 下安装。  
网址： https://packagecloud.io/Altinity/clickhouse  
以下 4 个 rpm 包即可：

```
mkdir -p /baicdt/softwares/ck
cd /baicdt/softwares/ck/

wget --content-disposition https://packagecloud.io/Altinity/clickhouse/packages/el/7/clickhouse-server-common-20.6.6.7-1.el7.x86_64.rpm/download.rpm

wget --content-disposition https://packagecloud.io/Altinity/clickhouse/packages/el/7/clickhouse-server-20.6.6.7-1.el7.x86_64.rpm/download.rpm

wget --content-disposition https://packagecloud.io/Altinity/clickhouse/packages/el/7/clickhouse-common-static-20.6.6.7-1.el7.x86_64.rpm/download.rpm

wget --content-disposition https://packagecloud.io/Altinity/clickhouse/packages/el/7/clickhouse-client-20.6.6.7-1.el7.x86_64.rpm/download.rpm

#安装
rpm -ivh *.rpm

```

> 1.  安装完后默认的配置文件是`/etc/clickhouse-server/config.xml`，其中默认的数据目录、临时目录、日志目录为：
>     *   `/var/lib/clickhouse` 默认数据目录
>     *   `/var/lib/clickhouse/tmp/` 默认临时目录
>     *   `/var/log/clickhouse-server` 默认日志目录
> 2.  服务端启动脚本：`/etc/init.d/clickhouse-server`

### 3. 启动与验证

```
# 启动 clickhouse-server
systemctl start clickhouse-server

# 查看 clickhouse-server 状态
systemctl status clickhouse-server

# 开机启动 clickhouse-server 状态
systemctl enable clickhouse-server

# 进入clickhouse客户端: 命令行clickhouse-client –h host –u –p
clickhouse-client

```

三. 集群安装
-------

<table><thead><tr><th>IP</th><th>主机名</th></tr></thead><tbody><tr><td>192.168.52.210</td><td>ck01.hadoop.com</td></tr><tr><td>192.168.52.220</td><td>ck02.hadoop.com</td></tr><tr><td>192.168.52.230</td><td>ck03.hadoop.com</td></tr></tbody></table>

    ClickHouse 集群安装非常简单，首先重复上面步骤，分别在其他机器上安装 ClickHouse，然后再分别配置一下`/etc/clickhouse-server/config.xml`和`/etc/metrika.xml`两个文件即可。注意：ClickHouse 集群的依赖于 Zookeeper，所以要保证先安装好 Zookeeper 集群。本文演示三个节点的 ClickHouse 集群安装，具体步骤如下：

### 1. 三台服务器上进行单机的部署

### 2. 创建 metrika.xml 文件

```
# 每台服务器都要创建
vim /etc/metrika.xml

```

```
<yandex>
	<!-- 该标签与config.xml的<remote_servers incl="clickhouse_remote_servers" >保持一致--> 
	<clickhouse_remote_servers>
		<!-- 自定义的标签名，也就是集群名称，可以修改 -->
		<perftest_3shards_1replicas>
			<!-- 配置三个分片，每个分片对应一台机器-->
			<!-- 数据分片1  -->
			<shard>
				<!-- 定义数据分片的数据权重。比如有两个分片，第一个权重为9，第二个权重为10，
				则该批次的行将会有9/19的数据被发送到第一个分片，10/19的数据被发送到第二分片。-->
				<weight>1</weight>
				<!-- 是否启用内部复制。true 代表写入数据时选择第一个健康的副本进行写入，其余
				副本以该表本身进行复制，保证复制表的一致性。false(默认) 代表将数据直接写入所
				有副本，因为没有检查复制表的一致性，而且随着时间的推移，它们将包含略微不同的
				数据。 -->
				<internal_replication>true</internal_replication>
				<!-- 指定分片数据副本，可为每个服务器指定参数主机、端口和可选的用户、密码、安全、压缩等 -->
				<replica>
					<!-- 数据分片远程服务地址（支持IPv6 ）。如果指定了域名（domain），
					服务器在启动时发出DNS请求并且只要服务器在运行就能一直保存结果。如
					果DNS请求失败，服务器不会启动。如果更改DNS记录，请重新启动服务器。 -->
					<host>192.168.52.210</host>
					<!-- TCP端口(对应config.xml的"tcp_port"，通常设置为9000)。不要将其与http_port混淆。 -->
					<port>9000</port>
				</replica>
			</shard>
			<!-- 数据分片2  -->
			<shard>
				<replica>
					<internal_replication>true</internal_replication>
					<host>192.168.52.220</host>
					<port>9000</port>
				</replica>
			</shard>
			<!-- 数据分片3  -->
			<shard>
				<internal_replication>true</internal_replication>
				<replica>
					<host>192.168.52.230</host>
					<port>9000</port>
				</replica>
			</shard>
		</perftest_3shards_1replicas>
	</clickhouse_remote_servers>

	<!-- zookeeper相关配置: 该标签与config.xml的 <zookeeper incl="zookeeper-servers" optional="true" /> 保持一致 -->
	<zookeeper-servers>
		<node index="1">
			<host>192.168.52.210</host>
			<port>2181</port>
		</node>
		<node index="2">
			<host>192.168.52.220</host>
			<port>2181</port>
		</node>
		<node index="3">
			<host>192.168.52.230</host>
			<port>2181</port>
		</node>
	</zookeeper-servers>

	<!-- 分片和副本标识: shard标签配置分片编号; <replica>配置分片副本主机名(需要修改对应主机上的配置)-->
	<!-- 宏定义,子标签有：
	         1、{layer} - ClickHouse集群的昵称，用于区分不同集群之间的数据。
			 2、{shard} - 分片编号或符号引用。
			 3、{replica} - 副本的名称（唯一），通常与主机名匹配macros为可选定义。
		配置文件中定义了在创建表时每台服务器就可以使用相同的建表DDL。否则要ReplicatedMergeTree指定zk路径和replica值。 -->
	<macros>
		<shard>01</shard>
		<replica>192.168.52.210</replica>
	</macros>
	
	<!-- 监听网络，::/0代表监听所有ip -->
	<networks>
		<ip>::/0</ip>
	</networks>
	
	<!-- 数据压缩算法：
			 ClickHouse检查{min_part_size}和{min_part_size_ratio}，并处理与这些条件匹配的{case}块。
			 如果<case>不匹配，ClickHouse应用lz4压缩算法。{method}指压缩方法，可接受值:lz4或zstd(实验值)。
	-->
	<clickhouse_compression>
		<case>
			<min_part_size>10000000000</min_part_size>
			<min_part_size_ratio>0.01</min_part_size_ratio>
			<method>lz4</method>
		</case>
	</clickhouse_compression>

</yandex>

```

> **配置说明来自** https://www.jianshu.com/p/5f7809b1965e

ck02 服务器 macros 标签属性

```
	<macros>
		<shard>02</shard>
		<replica>192.168.52.220</replica>
	</macros>

```

ck03 服务器 macros 标签属性

```
	<macros>
		<shard>03</shard>
		<replica>192.168.52.230</replica>
	</macros>

```

### 3. 修改 config.xml 配置

在`/etc/clickhouse-server/config.xml`中放开远程主机监听，每台服务器都要修改

```
    <listen_host>::1</listen_host>
    <listen_host>0.0.0.0</listen_host>

```

> 遇到问题：
> 
> 登录时若报异常`Code: 210. DB::NetException: Connection refused (localhost:9000, ::1)`  
> 解决: 需要开放`<listen_host>::1</listen_host>`。另外 zookeeper 集群没有正常启动也会造成建表时报此类错误。

### 4. 重启与验证

```
# 每台服务器重启clickhouse-server
systemctl restart clickhouse-server
# 查看 clickhouse-server 状态
systemctl status clickhouse-server
# 启动客户端，-m参数支持多行输入
clickhouse-client -m

```

可以查询系统表验证集群配置是否已被加载：

```
ck01.hadoop.com :) select cluster,shard_num,replica_num,host_name,port,user from system.clusters;
结果如下：
┌─cluster───────────────────────────┬─shard_num─┬─replica_num─┬─host_name──────┬─port─┬─user────┐
│ perftest_3shards_1replicas        │         1 │           1 │ 192.168.52.210 │ 9000 │ default │
│ perftest_3shards_1replicas        │         2 │           1 │ 192.168.52.220 │ 9000 │ default │
│ perftest_3shards_1replicas        │         3 │           1 │ 192.168.52.230 │ 9000 │ default │
│ test_cluster_two_shards           │         1 │           1 │ 127.0.0.1      │ 9000 │ default │
│ test_cluster_two_shards           │         2 │           1 │ 127.0.0.2      │ 9000 │ default │
│ test_cluster_two_shards_localhost │         1 │           1 │ localhost      │ 9000 │ default │
│ test_cluster_two_shards_localhost │         2 │           1 │ localhost      │ 9000 │ default │
│ test_shard_localhost              │         1 │           1 │ localhost      │ 9000 │ default │
│ test_shard_localhost_secure       │         1 │           1 │ localhost      │ 9440 │ default │
│ test_unavailable_shard            │         1 │           1 │ localhost      │ 9000 │ default │
│ test_unavailable_shard            │         2 │           1 │ localhost      │    1 │ default │
└───────────────────────────────────┴───────────┴─────────────┴────────────────┴──────┴─────────┘

```

查看集群的分片信息 (宏变量)：分别在各自机器上执行下面命令：

```
ck01.hadoop.com :) select * from system.macros;
┌─macro───┬─substitution───┐
│ replica │ 192.168.52.210 │
│ shard   │ 01             │
└─────────┴────────────────┘

```

### 5. 分布式 DDL 入门操作

    在分布式集群中，我们通常需要**先创建本地表（又叫：分片表、复制表）再创建分布式表**。因为分布式表只是作为一个查询引擎，本身不存储任何数据，查询时将 sql 发送到所有集群分片，然后进行进行处理和聚合后将结果返回给客户端。创建什么样的表，需要根据实际的使用场景决定在创建表指定什么样的表引擎。

### （1）创建本地表了

```
CREATE TABLE IF NOT EXISTS user_local 
(
    id Int32,
    name String
)ENGINE = MergeTree()
ORDER BY id
PARTITION BY id
PRIMARY KEY id;

```

我们先在一台机器上，对`user_local`表进行插入数据，然后再查询`user_cluster`表

### （2）创建分布式表

    默认情况下，CREATE、DROP、ALTER、RENAME 操作仅仅在当前执行该命令的 server 上生效。在集群环境下，可以使用 **ON CLUSTER** 语法，这样就可以在整个集群发挥作用。

比如创建一张分布式表：

```
CREATE TABLE IF NOT EXISTS user_cluster ON CLUSTER perftest_3shards_1replicas
(
    id Int32,
    name String
)ENGINE = Distributed(perftest_3shards_1replicas, default, user_local,id);

```

**表引擎的定义规则:**

`Distributed(cluster_name, database_name, table_name, [sharding_key])`

> *   **cluster_name**：集群名称，与集群配置中的自定义名称相对应。
> *   **database_name**：数据库名称
> *   **table_name**：本地表名称
> *   **sharding_key**：用于分片的 key 值，在数据写入的过程中，分布式表会依据分片 key 的规则，将数据分布到各个节点的本地表。**可选。**

**注意：**

    创建分布式表是**读时检查的机制，对创建分布式表和本地表的先后顺序没有强制要求**。

    同样值得注意的是，在上面的语句中使用了 ON CLUSTER 分布式 DDL，这意味着在集群的每个分片节点上，都会创建一张 Distributed 表，这样便可以从其中任意一端发起对所有分片的读、写请求。

### （3）测试：插入 | 查询

```
-- 插入数据
ck03.hadoop.com :) INSERT INTO user_local VALUES(1,'tom'),(2,'jack');

-- 任意节点查询user_cluster表,可查询到插入的数据
ck03.hadoop.com :) select * from user_cluster;
┌─id─┬─name─┐
│  1 │ tom  │
└────┴──────┘
┌─id─┬─name─┐
│  2 │ jack │
└────┴──────┘

-- 其他服务器查询分布式表
ck02.hadoop.com :) select * from user_cluster;
┌─id─┬─name─┐
│  2 │ jack │
└────┴──────┘
┌─id─┬─name─┐
│  1 │ tom  │
└────┴──────┘

```

然后，我们再向`user_cluster`中插入一些数据，观察`user_local`表数据变化，可以发现数据被分散存储到了其他节点上了。

```
-- 向user_cluster插入数据
ck03.hadoop.com :) INSERT INTO user_cluster VALUES(3,'lilei'),(4,'lihua'); 
-- 个服务器查看user_local数据，会发现新插入的数据分散到了其他节点
ck03.hadoop.com :) select * from user_local;
┌─id─┬─name─┐
│  1 │ tom  │
└────┴──────┘
┌─id─┬─name─┐
│  2 │ jack │
└────┴──────┘

```

四. Clickhouse 表引擎
-----------------

### 1. 表引擎的作用

*   决定表存储在哪里以及以何种方式存储
*   支持哪些查询以及如何支持
*   并发数据访问
*   索引的使用
*   是否可以执行多线程请求
*   数据复制参数

### 2. 表引擎分类

<table><thead><tr><th>引擎分类</th><th>引擎名称</th></tr></thead><tbody><tr><td>MergeTree 系列</td><td>MergeTree 、ReplacingMergeTree 、SummingMergeTree 、 AggregatingMergeTree CollapsingMergeTree 、 VersionedCollapsingMergeTree 、GraphiteMergeTree</td></tr><tr><td>Log 系列</td><td>TinyLog 、StripeLog 、Log</td></tr><tr><td>Integration Engines</td><td>Kafka 、MySQL、ODBC 、JDBC、HDFS</td></tr><tr><td>Special Engines</td><td>Distributed 、MaterializedView、 Dictionary 、Merge 、File、Null 、Set 、Join 、 URL View、Memory 、 Buffer</td></tr></tbody></table>

### 3. Log 系列表引擎

#### 应用场景

    Log 系列表引擎功能相对简单，主要用于快速写入小表 (1 百万行左右的表)，然后全部读出的场景。即一次写入多次查询。

#### Log 系列表引擎的特点

*   共性特点
    
    1.  数据存储在磁盘上
    2.  当写数据时，将数据追加到文件的末尾
    3.  不支持并发读写，当向表中写入数据时，针对这张表的查询会被阻塞，直至写入动作结束
    4.  不支持索引
    5.  不支持原子写：如果某些操作 (异常的服务器关闭) 中断了写操作，则可能会获得带有损坏数据的表
    6.  不支持 ALTER 操作 (这些操作会修改表设置或数据，比如 delete、update 等等)
*   区别
    
    1.  **TinyLog** 是 Log 系列引擎中功能简单、性能较低的引擎。它的存储结构由数据文件和元数据两部分组成。其中，数据文件是按列独立存储的，也就是说每一个列字段都对应一个文件。除此之外，TinyLog 不支持并发数据读取。
    2.  **StripLog** 支持并发读取数据文件，当读取数据时，ClickHouse 会使用多线程进行读取，每个线程处理一个单独的数据块。另外，StripLog 将所有列数据存储在同一个文件中，减少了文件的使用数量。
    3.  **Log** 支持并发读取数据文件，当读取数据时，ClickHouse 会使用多线程进行读取，每个线程处理一个单独的数据块。Log 引擎会将每个列数据单独存储在一个独立文件中。

#### TinyLog 表引擎使用

    适用于**一次写入，多次读取的场景**。对于处理小批数据的中间表可以使用该引擎。值得注意的是，使用大量的小表存储数据，性能会很低。

```
-- 建表
CREATE TABLE emp_tinylog (
  emp_id UInt16 COMMENT '员工id',
  name String COMMENT '员工姓名',
  work_place String COMMENT '工作地点',
  age UInt8 COMMENT '员工年龄',
  depart String COMMENT '部门',
  salary Decimal32(2) COMMENT '工资'
  )ENGINE=TinyLog();

-- 插入数据
INSERT INTO emp_tinylog 
VALUES (1,'tom','上海',25,'技术部',20000),(2,'jack','上海',26,'人事部',10000);
INSERT INTO emp_tinylog
VALUES (3,'bob','北京',33,'财务部',50000),(4,'tony','杭州',28,'销售事部',50000);

-- 查询数据
ck01.hadoop.com :) SELECT * FROM emp_tinylog
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart───┬───salary─┐
│      1 │ tom  │ 上海       │  25 │ 技术部   │ 20000.00 │
│      2 │ jack │ 上海       │  26 │ 人事部   │ 10000.00 │
│      3 │ bob  │ 北京       │  33 │ 财务部   │ 50000.00 │
│      4 │ tony │ 杭州       │  28 │ 销售事部 │ 50000.00 │
└────────┴──────┴────────────┴─────┴──────────┴──────────┘

```

进入默认数据存储目录，查看底层数据存储形式, 可以看出：TinyLog 引擎表每一列都对应的文件

```
[root@ck01 ~]# cd /var/lib/clickhouse/data/default/emp_tinylog
[root@ck01 emp_tinylog]# ll -ah
total 28K
drwxr-x--- 2 clickhouse clickhouse 131 Sep 24 10:56 .
drwxr-x--- 5 clickhouse clickhouse  63 Sep 24 10:56 ..
-rw-r----- 1 clickhouse clickhouse  56 Sep 24 10:56 age.bin
-rw-r----- 1 clickhouse clickhouse  97 Sep 24 10:56 depart.bin
-rw-r----- 1 clickhouse clickhouse  60 Sep 24 10:56 emp_id.bin
-rw-r----- 1 clickhouse clickhouse  70 Sep 24 10:56 name.bin
-rw-r----- 1 clickhouse clickhouse  68 Sep 24 10:56 salary.bin
-rw-r----- 1 clickhouse clickhouse 185 Sep 24 10:56 sizes.json
-rw-r----- 1 clickhouse clickhouse  80 Sep 24 10:56 work_place.bin

```

*   **注：**`sizes.json`文件内使用 JSON 格式记录了每个`.bin`文件内对应的数据大小的信息

当我们执行 **ALTER 操作**时会报错，说明该表引擎不支持 ALTER 操作

```
-- 以下操作会报错：
-- DB::Exception: Mutations are not supported by storage TinyLog.
ALTER TABLE emp_tinylog DELETE WHERE emp_id = 5;
ALTER TABLE emp_tinylog UPDATE age = 30 WHERE emp_id = 4;

```

#### StripLog 表引擎使用

    相比 TinyLog 而言，StripeLog 拥有更高的查询性能（拥有. mrk 标记文件，支持并行查询），同时其使用了更少的文件描述符（所有数据使用同一个文件保存）。

```
-- 建表
CREATE TABLE emp_stripelog (
  emp_id UInt16 COMMENT '员工id',
  name String COMMENT '员工姓名',
  work_place String COMMENT '工作地点',
  age UInt8 COMMENT '员工年龄',
  depart String COMMENT '部门',
  salary Decimal32(2) COMMENT '工资'
  )ENGINE=StripeLog;
-- 插入数据  
INSERT INTO emp_stripelog
VALUES (1,'tom','上海',25,'技术部',20000),(2,'jack','上海',26,'人事部',10000);
INSERT INTO emp_stripelog 
VALUES (3,'bob','北京',33,'财务部',50000),(4,'tony','杭州',28,'销售事部',50000);

-- 查询数据，由于是分两次插入数据，所以查询时会有两个数据块
ck01.hadoop.com :) select * from emp_stripelog;
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart─┬───salary─┐
│      1 │ tom  │ 上海       │  25 │ 技术部 │ 20000.00 │
│      2 │ jack │ 上海       │  26 │ 人事部 │ 10000.00 │
└────────┴──────┴────────────┴─────┴────────┴──────────┘
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart───┬───salary─┐
│      3 │ bob  │ 北京       │  33 │ 财务部   │ 50000.00 │
│      4 │ tony │ 杭州       │  28 │ 销售事部 │ 50000.00 │
└────────┴──────┴────────────┴─────┴──────────┴──────────┘

```

进入默认数据存储目录，查看底层数据存储形式

```
[root@ck01 emp_tinylog]# cd /var/lib/clickhouse/data/default/emp_stripelog
[root@ck01 emp_stripelog]# ll -h
total 12K
-rw-r----- 1 clickhouse clickhouse 673 Sep 24 11:05 data.bin
-rw-r----- 1 clickhouse clickhouse 281 Sep 24 11:05 index.mrk
-rw-r----- 1 clickhouse clickhouse  69 Sep 24 11:05 sizes.json

```

可以看出 StripeLog 表引擎对应的存储结构包括三个文件：

*   `data.bin`：数据文件，所有的列字段使用同一个文件保存，它们的数据都会被写入`data.bin`。
*   `index.mrk`：数据标记，保存了数据在`data.bin`文件中的位置信息 (每个插入数据块对应列的 offset)，利用数据标记能够使用多个线程，以并行的方式读取`data.bin内`的压缩数据块，从而提升数据查询的性能。
*   `sizes.json`：元数据文件，记录了`data.bin`和`index.mrk`大小的信息

> **注意：**
> 
>     StripeLog 引擎将所有数据都存储在了一个文件中，对于每次的 **INSERT** 操作，ClickHouse 会将**数据块**追加到表文件的末尾，StripeLog 引擎同样不支持`ALTER UPDATE` 和`ALTER DELETE` 操作。

#### Log 表引擎使用

    Log 引擎表适用于临时数据，一次性写入、测试场景。Log 引擎结合了 TinyLog 表引擎和 StripeLog 表引擎的长处，是 Log 系列引擎中性能最高的表引擎。

```
-- 建表
CREATE TABLE emp_log (
  emp_id UInt16 COMMENT '员工id',
  name String COMMENT '员工姓名',
  work_place String COMMENT '工作地点',
  age UInt8 COMMENT '员工年龄',
  depart String COMMENT '部门',
  salary Decimal32(2) COMMENT '工资'
  )ENGINE=Log;

-- 插入数据
INSERT INTO emp_log VALUES (1,'tom','上海',25,'技术部',20000),(2,'jack','上海',26,'人事部',10000);
INSERT INTO emp_log VALUES (3,'bob','北京',33,'财务部',50000),(4,'tony','杭州',28,'销售事部',50000);

-- 查询数据，由于是分两次插入数据，所以查询时会有两个数据块
SELECT * FROM emp_log;

```

进入默认数据存储目录，查看底层数据存储形式

```
[root@ck01 emp_log]# ll -h
total 32K
-rw-r----- 1 clickhouse clickhouse  56 Sep 24 11:12 age.bin
-rw-r----- 1 clickhouse clickhouse  97 Sep 24 11:12 depart.bin
-rw-r----- 1 clickhouse clickhouse  60 Sep 24 11:12 emp_id.bin
-rw-r----- 1 clickhouse clickhouse  96 Sep 24 11:12 __marks.mrk
-rw-r----- 1 clickhouse clickhouse  70 Sep 24 11:12 name.bin
-rw-r----- 1 clickhouse clickhouse  68 Sep 24 11:12 salary.bin
-rw-r----- 1 clickhouse clickhouse 215 Sep 24 11:12 sizes.json
-rw-r----- 1 clickhouse clickhouse  80 Sep 24 11:12 work_place.bin

```

Log 引擎的存储结构包含三部分：

*   `.bin`：数据文件，数据文件按列单独存储
*   `__marks.mrk`：数据标记，统一保存了数据在各个`.bin`文件中的位置信息。利用数据标记能够使用多个线程，以并行的方式读取。`.bin`内的压缩数据块，从而提升数据查询的性能。
*   `sizes.json`：记录了`.bin`和`__marks.mrk`大小的信息

> ** 注意：**Log 表引擎会将每一列都存在一个文件中，对于每一次的 INSERT 操作，都会对应一个数据块

### 4. MergeTree 系列引擎

    在所有的表引擎中，最为核心的当属 MergeTree 系列表引擎，这些表引擎拥有最为强大的性能和最广泛的使用场合。对于非 MergeTree 系列的其他引擎而言，主要用于特殊用途，场景相对有限。而 MergeTree 系列表引擎是官方主推的存储引擎，支持几乎所有 ClickHouse 核心功能。

#### MergeTree 表引擎（重要）

    更详细的讲解参阅： https://clickhouse.tech/docs/zh/engines/table-engines/mergetree-family/mergetree/

    MergeTree 在写入一批数据时，数据总会以数据片段的形式写入磁盘，且数据片段不可修改。为了避免片段过多，ClickHouse 会通过后台线程，定期合并这些数据片段，属于相同分区的数据片段会被合成一个新的片段。这种数据片段往复合并的特点，也正是合并树名称的由来。

MergeTree 作为家族系列最基础的表引擎，主要有以下特点：

*   存储的数据按照主键排序：允许创建稀疏索引，从而加快数据查询速度
    
*   支持分区，可以通过 PRIMARY KEY 语句指定分区字段。
    
    在相同数据集和相同结果集的情况下 ClickHouse 中某些带分区的操作会比普通操作更快。查询中指定了分区键时 ClickHouse 会自动截取分区数据。这也有效增加了查询性能。
    
*   支持数据副本
    
*   支持数据采样
    

**建表语法**

```
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1] [TTL expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2] [TTL expr2],
    ...
    INDEX index_name1 expr1 TYPE type1(...) GRANULARITY value1,
    INDEX index_name2 expr2 TYPE type2(...) GRANULARITY value2
) ENGINE = MergeTree()
ORDER BY expr
[PARTITION BY expr]
[PRIMARY KEY expr]
[SAMPLE BY expr]
[TTL expr [DELETE|TO DISK 'xxx'|TO VOLUME 'xxx'], ...]
[SETTINGS name=value, ...]

```

*   `ENGINE` – 引擎名和参数。 `ENGINE = MergeTree()`. `MergeTree` 引擎没有参数。
    
*   `ORDER BY`– 排序键。**必选**。
    
    *   可以是一组列的元组或任意的表达式。 例如: `ORDER BY (CounterID, EventDate)` 。
    *   如果没有使用 `PRIMARY KEY` 显式的指定主键，ClickHouse 会使用排序键作为主键。
    *   如果不需要排序，可以使用 `ORDER BY tuple()`. 这种情况下，ClickHouse 会按照插入的顺序存储数据。
*   `PARTITION BY`– 分区键 。**可选**。
    
    要按月分区，可以使用表达式 `toYYYYMM(date_column)` ，这里的 `date_column` 是一个 [Date](https://clickhouse.tech/docs/zh/engines/table-engines/mergetree-family/mergetree/) 类型的列。分区名的格式会是 `"YYYYMM"` 。
    
*   `PRIMARY KEY` – 主键。如果排序字段与主键不一致，可以单独指定主键字段。否则默认主键是排序字段。**可选**。
    
*   `SAMPLE BY`– 用于抽样的表达式。**可选**。
    
    如果指定了该字段，那么主键中也必须包含该字段。比如`SAMPLE BY intHash32(UserID) ORDER BY (CounterID, EventDate, intHash32(UserID))`。
    
*   `TTL`–指定行存储的持续时间并定义数据片段在硬盘和卷上的移动逻辑的规则列表。** 可选。** 当时间到达时，如果是列字段级别的 TTL，则会删除这一列的数据；如果是表级别的 TTL，则会删除整张表的数据。**可选**。
    
*   `SETTINGS` — 控制 `MergeTree` 行为的额外参数。**可选**。
    

**建表示例**

```
-- 建表
DROP TABLE IF EXISTS emp_mergetree;
CREATE TABLE emp_mergetree (
  emp_id UInt16 COMMENT '员工id',
  name String COMMENT '员工姓名',
  work_place String COMMENT '工作地点',
  age UInt8 COMMENT '员工年龄',
  depart String COMMENT '部门',
  salary Decimal32(2) COMMENT '工资'
)ENGINE=MergeTree()
ORDER BY emp_id
PARTITION BY work_place;

-- 插入数据 每个INSERT插入语句都会根据分区字段生成新的分区
INSERT INTO emp_mergetree 
VALUES (1,'tom','上海',25,'技术部',20000),(2,'jack','上海',26,'人事部',10000);
INSERT INTO emp_mergetree 
VALUES (3,'bob','北京',33,'财务部',50000),(4,'tony','杭州',28,'销售事部',50000); 

-- 查询数据
-- 按work_place进行分区,分了三个区
SELECT * FROM emp_mergetree;
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart─┬───salary─┐
│      1 │ tom  │ 上海       │  25 │ 技术部 │ 20000.00 │
│      2 │ jack │ 上海       │  26 │ 人事部 │ 10000.00 │
└────────┴──────┴────────────┴─────┴────────┴──────────┘
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart───┬───salary─┐
│      4 │ tony │ 杭州       │  28 │ 销售事部 │ 50000.00 │
└────────┴──────┴────────────┴─────┴──────────┴──────────┘
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart─┬───salary─┐
│      3 │ bob  │ 北京       │  33 │ 财务部 │ 50000.00 │
└────────┴──────┴────────────┴─────┴────────┴──────────┘

```

查看一下数据存储格式，也可以看出，存在三个分区文件夹，每一个分区文件夹内存储了对应分区的数据

```
[root@ck01 ~]# cd /var/lib/clickhouse/data/default/emp_mergetree
[root@ck01 emp_mergetree]# ll -h
total 16K
drwxr-x--- 2 clickhouse clickhouse 4.0K Sep 24 14:03 1c89a3ba9fe5fd53379716a776c5ac34_3_3_0
drwxr-x--- 2 clickhouse clickhouse 4.0K Sep 24 14:03 40d45822dbd7fa81583d715338929da9_1_1_0
drwxr-x--- 2 clickhouse clickhouse 4.0K Sep 24 14:03 a6155dcc1997eda1a348cd98b17a93e9_2_2_0
drwxr-x--- 2 clickhouse clickhouse    6 Sep 24 14:02 detached
-rw-r----- 1 clickhouse clickhouse    1 Sep 24 14:02 format_version.txt

# 进入一个分区目录查看
[root@ck01 1c89a3ba9fe5fd53379716a776c5ac34_3_3_0]# ll -h
total 76K
-rw-r----- 1 clickhouse clickhouse   27 Sep 24 14:03 age.bin
-rw-r----- 1 clickhouse clickhouse   48 Sep 24 14:03 age.mrk2
-rw-r----- 1 clickhouse clickhouse  591 Sep 24 14:03 checksums.txt
-rw-r----- 1 clickhouse clickhouse  138 Sep 24 14:03 columns.txt
-rw-r----- 1 clickhouse clickhouse    1 Sep 24 14:03 count.txt
-rw-r----- 1 clickhouse clickhouse   39 Sep 24 14:03 depart.bin
-rw-r----- 1 clickhouse clickhouse   48 Sep 24 14:03 depart.mrk2
-rw-r----- 1 clickhouse clickhouse   28 Sep 24 14:03 emp_id.bin
-rw-r----- 1 clickhouse clickhouse   48 Sep 24 14:03 emp_id.mrk2
-rw-r----- 1 clickhouse clickhouse   14 Sep 24 14:03 minmax_work_place.idx
-rw-r----- 1 clickhouse clickhouse   31 Sep 24 14:03 name.bin
-rw-r----- 1 clickhouse clickhouse   48 Sep 24 14:03 name.mrk2
-rw-r----- 1 clickhouse clickhouse    7 Sep 24 14:03 partition.dat
-rw-r----- 1 clickhouse clickhouse    4 Sep 24 14:03 primary.idx
-rw-r----- 1 clickhouse clickhouse   30 Sep 24 14:03 salary.bin
-rw-r----- 1 clickhouse clickhouse   48 Sep 24 14:03 salary.mrk2
-rw-r----- 1 clickhouse clickhouse   33 Sep 24 14:03 work_place.bin
-rw-r----- 1 clickhouse clickhouse   48 Sep 24 14:03 work_place.mrk2

```

*   **checksums.txt**：校验文件，使用二进制格式存储。它保存了余下各类文件 (primary. idx、count.txt 等) 的 size 大小及 size 的哈希值，用于快速校验文件的完整性和正确性。
    
*   **columns.txt**：列信息文件，使用明文格式存储。用于保存此数据分区下的列字段信息。`cat columns.txt`
    
*   **count.txt**：计数文件，使用明文格式存储。用于记录当前数据分区目录下数据的总行数
    
*   **primary.idx**：一级索引文件，使用二进制格式存储。用于存放稀疏索引，一张 MergeTree 表只能声明一次一级索引，**即通过 ORDER BY 或者 PRIMARY KEY** 指定字段。借助稀疏索引，在数据查询的时能够排除主键条件范围之外的数据文件，从而有效减少数据扫描范围，加速查询速度。
    
*   **列. bin**：数据文件，使用压缩格式存储，默认为 LZ4 压缩格式，用于存储某一列的数据。由于 MergeTree 采用列式存储，所以每一个列字段都拥有独立的`.bin`数据文件，并以列字段名称命名。
    
*   **列. mrk2**：列字段标记文件，使用二进制格式存储。标记文件中保存了`.bin`文件中数据的偏移量信息
    
*   **partition.dat 与 minmax_[Column].idx**：如果指定了分区键，则会额外生成 partition.dat 与 minmax 索引文件，它们均使用二进制格式存储。**partition.dat** 用于保存当前分区下分区表达式最终生成的值，即分区字段值；而 **minmax** 索引用于记录当前分区下分区字段对应原始数据的最小和最大值。比如当使用 EventTime 字段对应的原始数据为 2020-09-17、2020-09-30，分区表达式为 PARTITION BY toYYYYMM(EventTime)，即按月分区。partition.dat 中保存的值将会是 2019-09，而 minmax 索引中保存的值将会是 2020-09-17 2020-09-30。
    

**注意：多次插入数据，会生成多个分区文件**

```
-- 新插入两条数据  虽然北京分区已经存在,但新插入时,还是生成了新的新的分区,这也说明了每个INSERT插入语句都会根据分区字段生成新的分区
INSERT INTO emp_mergetree
VALUES (5,'robin','北京',35,'财务部',50000),(6,'lilei','北京',38,'销售事部',50000);

-- 查询结果
ck01.hadoop.com :) SELECT * FROM emp_mergetree;
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart─┬───salary─┐
│      3 │ bob  │ 北京       │  33 │ 财务部 │ 50000.00 │
└────────┴──────┴────────────┴─────┴────────┴──────────┘
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart───┬───salary─┐
│      4 │ tony │ 杭州       │  28 │ 销售事部 │ 50000.00 │
└────────┴──────┴────────────┴─────┴──────────┴──────────┘
┌─emp_id─┬─name──┬─work_place─┬─age─┬─depart───┬───salary─┐
│      5 │ robin │ 北京       │  35 │ 财务部   │ 50000.00 │
│      6 │ lilei │ 北京       │  38 │ 销售事部 │ 50000.00 │
└────────┴───────┴────────────┴─────┴──────────┴──────────┘
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart─┬───salary─┐
│      1 │ tom  │ 上海       │  25 │ 技术部 │ 20000.00 │
│      2 │ jack │ 上海       │  26 │ 人事部 │ 10000.00 │
└────────┴──────┴────────────┴─────┴────────┴──────────┘

```

```
# 查看一下数据存储格式，也可以看出，又新生成了一个分区文件夹 a6155dcc1997eda1a348cd98b17a93e9_4_4_0
[root@ck01 emp_mergetree]# ll /var/lib/clickhouse/data/default/emp_mergetree
total 20
drwxr-x--- 2 clickhouse clickhouse 4096 Sep 24 14:03 1c89a3ba9fe5fd53379716a776c5ac34_3_3_0
drwxr-x--- 2 clickhouse clickhouse 4096 Sep 24 14:03 40d45822dbd7fa81583d715338929da9_1_1_0
drwxr-x--- 2 clickhouse clickhouse 4096 Sep 24 14:03 a6155dcc1997eda1a348cd98b17a93e9_2_2_0
drwxr-x--- 2 clickhouse clickhouse 4096 Sep 24 14:19 a6155dcc1997eda1a348cd98b17a93e9_4_4_0
drwxr-x--- 2 clickhouse clickhouse    6 Sep 24 14:02 detached
-rw-r----- 1 clickhouse clickhouse    1 Sep 24 14:02 format_version.txt

```

    新插入的数据新生成了一个数据块，并没有与原来的分区数据在一起，我们可以执行 optimize 命令，执行合并操作

```
-- 仅对'北京'分区内的数据执行合并操作, 合并操作也会根据合并的分区数在数据存储目录中生成相应分区数的分区文件夹
OPTIMIZE TABLE emp_mergetree PARTITION '北京';

-- 再次执行查询
ck01.hadoop.com :) SELECT * FROM emp_mergetree;             
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart─┬───salary─┐
│      1 │ tom  │ 上海       │  25 │ 技术部 │ 20000.00 │
│      2 │ jack │ 上海       │  26 │ 人事部 │ 10000.00 │
└────────┴──────┴────────────┴─────┴────────┴──────────┘
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart───┬───salary─┐
│      4 │ tony │ 杭州       │  28 │ 销售事部 │ 50000.00 │
└────────┴──────┴────────────┴─────┴──────────┴──────────┘
┌─emp_id─┬─name──┬─work_place─┬─age─┬─depart───┬───salary─┐
│      3 │ bob   │ 北京       │  33 │ 财务部   │ 50000.00 │
│      5 │ robin │ 北京       │  35 │ 财务部   │ 50000.00 │
│      6 │ lilei │ 北京       │  38 │ 销售事部 │ 50000.00 │
└────────┴───────┴────────────┴─────┴──────────┴──────────┘

```

```
# 查看一下数据存储格式，可以看出，又新生成了一个分区文件夹，原来的分区文件夹不变。 a6155dcc1997eda1a348cd98b17a93e9_2_4_1
[root@ck01 emp_mergetree]# ll /var/lib/clickhouse/data/default/emp_mergetree
total 24
drwxr-x--- 2 clickhouse clickhouse 4096 Sep 24 14:03 1c89a3ba9fe5fd53379716a776c5ac34_3_3_0
drwxr-x--- 2 clickhouse clickhouse 4096 Sep 24 14:03 40d45822dbd7fa81583d715338929da9_1_1_0
drwxr-x--- 2 clickhouse clickhouse 4096 Sep 24 14:03 a6155dcc1997eda1a348cd98b17a93e9_2_2_0
drwxr-x--- 2 clickhouse clickhouse 4096 Sep 24 14:24 a6155dcc1997eda1a348cd98b17a93e9_2_4_1
drwxr-x--- 2 clickhouse clickhouse 4096 Sep 24 14:19 a6155dcc1997eda1a348cd98b17a93e9_4_4_0
drwxr-x--- 2 clickhouse clickhouse    6 Sep 24 14:02 detached
-rw-r----- 1 clickhouse clickhouse    1 Sep 24 14:02 format_version.txt

```

**在 MergeTree 中主键并不用于去重，而是用于索引，加快查询速度**

```
-- 插入一条相同主键的数据,会发现该条数据可以插入
INSERT INTO emp_mergetree VALUES (1,'sam','杭州',35,'财务部',50000);

-- 查询发现emp_id=1的数据查到了两条，由此可知，并不会对主键进行去重
ck01.hadoop.com :) SELECT * FROM emp_mergetree WHERE emp_id = 1;
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart─┬───salary─┐
│      1 │ tom  │ 上海       │  25 │ 技术部 │ 20000.00 │
└────────┴──────┴────────────┴─────┴────────┴──────────┘
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart─┬───salary─┐
│      1 │ sam  │ 杭州       │  35 │ 财务部 │ 50000.00 │
└────────┴──────┴────────────┴─────┴────────┴──────────┘

```

#### ReplacingMergeTree 表引擎

    上文最后提到 MergeTree 表引擎无法对相同主键的数据进行去重，ClickHouse 提供了 ReplacingMergeTree 引擎，它能够在合并分区时删除相同的数据分区内重复数据。

**建表语法**

```
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
    ...
) ENGINE = ReplacingMergeTree([ver])
[PARTITION BY expr]
[ORDER BY expr]
[PRIMARY KEY expr]
[SAMPLE BY expr]
[SETTINGS name=value, ...]

```

*   `ver` – 列的版本。** 可选。** 类型为 `UInt*`, `Date` 或 `DateTime`。
    
    在数据合并的时候，`ReplacingMergeTree` 从所有具有相同排序键的行中选择一行留下：
    
    *   如果 `ver` 列未指定，保留最后一条。
    *   如果 `ver` 列已指定，保留 `ver` 值最大的版本

**建表示例：验证合并分区时，只有相同的数据分区内重复的数据才会被去重**

```
-- 建表  ORDER BY (emp_id,name)
DROP TABLE IF EXISTS emp_replacingmergetree;
CREATE TABLE emp_replacingmergetree (
  emp_id UInt16 COMMENT '员工id',
  name String COMMENT '员工姓名',
  work_place String COMMENT '工作地点',
  age UInt8 COMMENT '员工年龄',
  depart String COMMENT '部门',
  salary Decimal32(2) COMMENT '工资'
)ENGINE=ReplacingMergeTree()
ORDER BY (emp_id,name)
PRIMARY KEY emp_id
PARTITION BY work_place;

 -- 插入数据   数据存储目分区数：4
INSERT INTO emp_replacingmergetree VALUES (1,'tom','上海',25,'技术部',20000);
INSERT INTO emp_replacingmergetree VALUES (2,'jack','上海',26,'人事部',10000);
INSERT INTO emp_replacingmergetree VALUES (3,'bob','北京',33,'财务部',50000);
INSERT INTO emp_replacingmergetree VALUES (4,'tony','杭州',28,'销售事部',50000); 

-- 查询
select * from emp_replacingmergetree;
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart─┬───salary─┐
│      2 │ jack │ 上海       │  26 │ 人事部 │ 10000.00 │
└────────┴──────┴────────────┴─────┴────────┴──────────┘
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart───┬───salary─┐
│      4 │ tony │ 杭州       │  28 │ 销售事部 │ 50000.00 │
└────────┴──────┴────────────┴─────┴──────────┴──────────┘
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart─┬───salary─┐
│      1 │ tom  │ 上海       │  25 │ 技术部 │ 20000.00 │
└────────┴──────┴────────────┴─────┴────────┴──────────┘
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart─┬───salary─┐
│      3 │ bob  │ 北京       │  33 │ 财务部 │ 50000.00 │
└────────┴──────┴────────────┴─────┴────────┴──────────┘

-- 再次向该表插入具有相同主键的数据  数据存储目分区数：6
-- (1,'tom','北京',26,'技术部',10000) -- emp_id和name相同，满足orderby字段重复，但区分不同，不会被去重
-- (1,'sam','上海',25,'技术部',20000)  -- 满足分区相同，主键相同，但orderby字段不同，不会被去重
-- (1,'tom','上海',25,'技术部',50000)  -- emp_id、name、分区都相同，满足分区相同，orderby字段相同，会被去重
INSERT INTO emp_replacingmergetree VALUES (1,'tom','北京',26,'技术部',10000),(1,'sam','上海',25,'技术部',20000),(1,'tom','上海',25,'技术部',50000);

-- 查询数据
select * from emp_replacingmergetree;
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart─┬───salary─┐
│      1 │ sam  │ 上海       │  25 │ 技术部 │ 20000.00 │
│      1 │ tom  │ 上海       │  25 │ 技术部 │ 50000.00 │
└────────┴──────┴────────────┴─────┴────────┴──────────┘
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart───┬───salary─┐
│      4 │ tony │ 杭州       │  28 │ 销售事部 │ 50000.00 │
└────────┴──────┴────────────┴─────┴──────────┴──────────┘
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart─┬───salary─┐
│      3 │ bob  │ 北京       │  33 │ 财务部 │ 50000.00 │
└────────┴──────┴────────────┴─────┴────────┴──────────┘
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart─┬───salary─┐
│      1 │ tom  │ 上海       │  25 │ 技术部 │ 20000.00 │
└────────┴──────┴────────────┴─────┴────────┴──────────┘
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart─┬───salary─┐
│      1 │ tom  │ 北京       │  26 │ 技术部 │ 10000.00 │
└────────┴──────┴────────────┴─────┴────────┴──────────┘
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart─┬───salary─┐
│      2 │ jack │ 上海       │  26 │ 人事部 │ 10000.00 │
└────────┴──────┴────────────┴─────┴────────┴──────────┘

-- 执行合并操作，数据存储目分区数：9
optimize table emp_replacingmergetree final;

-- 再次查询，相同主键的数据，保留最近插入的数据，旧的数据被清除
select * from emp_replacingmergetree;    
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart─┬───salary─┐
│      1 │ sam  │ 上海       │  25 │ 技术部 │ 20000.00 │
│      1 │ tom  │ 上海       │  25 │ 技术部 │ 50000.00 │
│      2 │ jack │ 上海       │  26 │ 人事部 │ 10000.00 │
└────────┴──────┴────────────┴─────┴────────┴──────────┘
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart───┬───salary─┐
│      4 │ tony │ 杭州       │  28 │ 销售事部 │ 50000.00 │
└────────┴──────┴────────────┴─────┴──────────┴──────────┘
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart─┬───salary─┐
│      1 │ tom  │ 北京       │  26 │ 技术部 │ 10000.00 │
│      3 │ bob  │ 北京       │  33 │ 财务部 │ 50000.00 │
└────────┴──────┴────────────┴─────┴────────┴──────────┘

```

#### SummingMergeTree 表引擎

    该引擎继承了 MergeTree 引擎，推荐将该引擎和 MergeTree 一起使用。例如，将完整的数据存储在 MergeTree 表中，并且使用 SummingMergeTree 来存储聚合数据。这种方法可以避免因为使用不正确的主键组合方式而丢失数据。

    如果用户只需要查询数据的汇总结果，不关心明细数据，并且数据的汇总条件是预先明确的，即 **GROUP BY 的分组字段是确定的**，可以使用该表引擎。

**建表语法**

```
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
    ...
) ENGINE = SummingMergeTree([columns]) -- 指定合并时汇总字段
[PARTITION BY expr]
[ORDER BY expr]
[SAMPLE BY expr]
[SETTINGS name=value, ...]

```

**建表示例**

```
-- 建表  SummingMergeTree(salary)当合并时,相同分区如果有排序字段重复的数据会被去重,重复数据的salary会被求和,作为去重后的salary字段的值
DROP TABLE IF EXISTS emp_summingmergetree;
CREATE TABLE emp_summingmergetree (
  emp_id UInt16 COMMENT '员工id',
  name String COMMENT '员工姓名',
  work_place String COMMENT '工作地点',
  age UInt8 COMMENT '员工年龄',
  depart String COMMENT '部门',
  salary Decimal32(2) COMMENT '工资'
)ENGINE=SummingMergeTree(salary)
ORDER BY (emp_id,name)
PRIMARY KEY emp_id
PARTITION BY work_place;

 -- 插入数据   数据存储目分区数：3
INSERT INTO emp_summingmergetree
VALUES (1,'tom','上海',25,'技术部',20000),(2,'jack','上海',26,'人事部',10000);
INSERT INTO emp_summingmergetree
VALUES (3,'bob','北京',33,'财务部',50000),(4,'tony','杭州',28,'销售事部',50000); 

-- 再次插入数据   数据存储目分区数：5
-- (1,'tom','上海',25,'信息部',10000) 同一分区内排序字段重复,会被去重且合并salary,保留最新的记录(即此条记录),但salary字段会与重复字段合并salary的值应为30000
-- (1,'jerry','上海',25,'信息部',10000) 同一分区内,主键字段重复但排序字段不重复,不会被去重
-- (1,'tom','北京',26,'人事部',10000) 不同分区内,排序字段重复,不会被去重
INSERT INTO emp_summingmergetree VALUES 
(1,'tom','上海',25,'信息部',10000),
(1,'jerry','上海',25,'信息部',10000),
(1,'tom','北京',26,'人事部',10000);

-- 执行合并操作   数据存储目分区数：5
optimize table emp_summingmergetree final;

-- 查询
select * from emp_summingmergetree;       
┌─emp_id─┬─name──┬─work_place─┬─age─┬─depart─┬───salary─┐
│      1 │ jerry │ 上海       │  25 │ 信息部 │ 10000.00 │   -- 同一分区排序字段不同不会被去重
│      1 │ tom   │ 上海       │  25 │ 技术部 │ 30000.00 │   -- salary值为30000
│      2 │ jack  │ 上海       │  26 │ 人事部 │ 10000.00 │
└────────┴───────┴────────────┴─────┴────────┴──────────┘
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart───┬───salary─┐
│      4 │ tony │ 杭州       │  28 │ 销售事部 │ 50000.00 │
└────────┴──────┴────────────┴─────┴──────────┴──────────┘
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart─┬───salary─┐
│      1 │ tom  │ 北京       │  26 │ 人事部 │ 10000.00 │    -- 分区不同,未被去重
│      3 │ bob  │ 北京       │  33 │ 财务部 │ 50000.00 │
└────────┴──────┴────────────┴─────┴────────┴──────────┘

```

#### Aggregatingmergetree 表引擎

该表引擎继承自 MergeTree，可以使用 AggregatingMergeTree 表来做增量数据统计聚合。如果要按一组规则来合并减少行数，则使用 AggregatingMergeTree 是合适的。AggregatingMergeTree 是通过预先定义的聚合函数计算数据并通过二进制的格式存入表内。

与 SummingMergeTree 的区别在于：SummingMergeTree 对非主键列进行 sum 聚合，而 AggregatingMergeTree 则可以指定各种聚合函数。

**建表示例**

AggregatingMergeTree 通常作为物化视图的表引擎，与普通 MergeTree 搭配使用。

```
-- 创建一个MereTree引擎的明细表,用于存储全量的明细数据,对外提供实时查询
DROP TABLE IF EXISTS tbl_emp_mergetree;
CREATE TABLE tbl_emp_mergetree (
  emp_id UInt16 COMMENT '员工id',
  name String COMMENT '员工姓名',
  work_place String COMMENT '工作地点',
  age UInt8 COMMENT '员工年龄',
  depart String COMMENT '部门',
  salary Decimal32(2) COMMENT '工资'
)ENGINE=MergeTree()
ORDER BY (emp_id,name)
PARTITION BY work_place;
  
-- 创建一张物化视图,使用AggregatingMergeTree表引擎
DROP VIEW IF EXISTS view_emp_agg;
CREATE MATERIALIZED VIEW view_emp_agg
ENGINE = AggregatingMergeTree()
PARTITION BY emp_id
ORDER BY (emp_id,name)
AS SELECT
     emp_id,
     name,
     sumState(salary) AS salary
FROM tbl_emp_mergetree
GROUP BY emp_id,name;

-- 向基础明细表emp_mergetree_base插入数据
INSERT INTO tbl_emp_mergetree
VALUES (1,'tom','上海',25,'技术部',20000),(1,'tom','上海',26,'人事部',10000);

-- 查询物化视图
SELECT emp_id, name, sumMerge(salary) 
FROM view_emp_agg
GROUP BY emp_id,name;
-- 结果
┌─emp_id─┬─name─┬─sumMerge(salary)─┐
│      1 │ tom  │         30000.00 │
└────────┴──────┴──────────────────┘

```

#### CollapsingMergeTree 表引擎

    CollapsingMergeTree【collapsing 译：折叠、崩塌】就是一种 == 通过以增代删的思路，支持行级数据修改和删除的表引擎。== 它通过定义一个 sign 标记位字段，记录数据行的状态。如果 sign 标记为 1，则表示这是一行有效的数据；如果 sign 标记为 - 1，则表示这行数据需要被删除。CollapsingMergeTree 会异步的删除（折叠）这些除了特定列 `Sign` 有 `1` 和 `-1` 的值以外，其余所有字段的值都相等的成对的行。没有成对的行会被保留。更多的细节请看本文的折叠部分。

    每次需要新增数据时，写入一行 sign 标记为 1 的数据；需要删除数据时，则写入一行 sign 标记为 - 1 的数据。

**建表语法**

```
-- sign是一个Int8类型的字段
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
    ...
) ENGINE = CollapsingMergeTree(sign)
[PARTITION BY expr]
[ORDER BY expr]
[SAMPLE BY expr]
[SETTINGS name=value, ...]

```

**建表示例**

```
-- 建表  CollapsingMergeTree同样是以ORDER BY排序键作为判断数据唯一性的依据
DROP TABLE IF EXISTS emp_collapsingmergetree;
CREATE TABLE emp_collapsingmergetree (
  emp_id UInt16 COMMENT '员工id',
  name String COMMENT '员工姓名',
  work_place String COMMENT '工作地点',
  age UInt8 COMMENT '员工年龄',
  depart String COMMENT '部门',
  salary Decimal32(2) COMMENT '工资',
  sign Int8
)ENGINE=CollapsingMergeTree(sign)
ORDER BY (emp_id,name)
PARTITION BY work_place;

-- 插入新增数据,sign=1表示正常数据
INSERT INTO emp_collapsingmergetree 
VALUES (1,'tom','上海',25,'技术部',20000,1);

-- 更新上述的数据
-- 首先插入一条与原来相同的数据(ORDER BY字段一致),并将sign置为-1
INSERT INTO emp_collapsingmergetree 
VALUES (1,'tom','上海',25,'技术部',20000,-1);

-- 再插入更新之后的数据
INSERT INTO emp_collapsingmergetree 
VALUES (1,'tom','上海',25,'技术部',30000,1);

-- 查看一下结果
cdh04 :) select * from emp_collapsingmergetree ;

SELECT *
FROM emp_collapsingmergetree

┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart─┬───salary─┬─sign─┐
│      1 │ tom  │ 上海       │  25 │ 技术部 │ 30000.00 │    1 │
└────────┴──────┴────────────┴─────┴────────┴──────────┴──────┘
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart─┬───salary─┬─sign─┐
│      1 │ tom  │ 上海       │  25 │ 技术部 │ 20000.00 │   -1 │
└────────┴──────┴────────────┴─────┴────────┴──────────┴──────┘
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart─┬───salary─┬─sign─┐
│      1 │ tom  │ 上海       │  25 │ 技术部 │ 20000.00 │    1 │
└────────┴──────┴────────────┴─────┴────────┴──────────┴──────┘

-- 执行分区合并操作
optimize table emp_collapsingmergetree;
-- 再次查询，sign=1与sign=-1的数据相互抵消了，即被删除
select * from emp_collapsingmergetree ;
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart─┬───salary─┬─sign─┐
│      1 │ tom  │ 上海       │  25 │ 技术部 │ 30000.00 │    1 │
└────────┴──────┴────────────┴─────┴────────┴──────────┴──────┘

```

**注意：**

1、分区数据折叠不是实时的，需要后台进行 Compaction 操作，用户也可以使用手动合并命令，但是效率会很低，一般不推荐在生产环境中使用。当进行汇总数据操作时，可以通过改变查询方式，来过滤掉被删除的数据。

```
SELECT 
    emp_id, 
    name, 
    sum(salary * sign)
FROM emp_collapsingmergetree
GROUP BY 
    emp_id, 
    name
HAVING sum(sign) > 0;

```

2、CollapsingMergeTree 对于写入数据的顺序有着严格要求，否则导致无法正常折叠。

    如果数据的写入程序是单线程执行的，则能够较好地控制写入顺序；如果需要处理的数据量很大，数据的写入程序通常是多线程执行的，那么此时就不能保障数据的写入顺序了。在这种情况下，CollapsingMergeTree 的工作机制就会出现问题。但是**可以通过 VersionedCollapsingMergeTree 的表引擎得到解决**。

#### VersionedCollapsingMergeTree 表引擎

    上面提到 CollapsingMergeTree 表引擎对于数据写入乱序的情况下，不能够实现数据折叠的效果。VersionedCollapsingMergeTree 表引擎的作用与 CollapsingMergeTree 完全相同，它们的不同之处在于，VersionedCollapsingMergeTree 对数据的写入顺序没有要求，在同一个分区内，任意顺序的数据都能够完成折叠操作。VersionedCollapsingMergeTree 使用 version 列来实现乱序情况下的数据折叠。

**建表语法**

```
-- 该引擎除了需要指定一个sign标识之外，还需要指定一个UInt8类型的version版本号。
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
    ...
) ENGINE = VersionedCollapsingMergeTree(sign, version)
[PARTITION BY expr]
[ORDER BY expr]
[SAMPLE BY expr]
[SETTINGS name=value, ...]

```

**建表示例**

```
DROP TABLE IF EXISTS emp_collapsingmergetree;
CREATE TABLE emp_versioned (
  emp_id UInt16 COMMENT '员工id',
  name String COMMENT '员工姓名',
  work_place String COMMENT '工作地点',
  age UInt8 COMMENT '员工年龄',
  depart String COMMENT '部门',
  salary Decimal32(2) COMMENT '工资',
  sign Int8,
  version Int8
)ENGINE=VersionedCollapsingMergeTree(sign, version)
ORDER BY (emp_id,name)
PARTITION BY work_place;
  
-- 先插入需要被删除的数据，即sign=-1的数据
INSERT INTO emp_versioned VALUES (1,'tom','上海',25,'技术部',20000,-1,1);
-- 再插入sign=1的数据
INSERT INTO emp_versioned VALUES (1,'tom','上海',25,'技术部',20000,1,1);
-- 在插入一个新版本数据
INSERT INTO emp_versioned VALUES (1,'tom','上海',25,'技术部',30000,1,2);

-- 手动合并
optimize table emp_versioned;

-- 再次查询
select * from emp_versioned;
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart─┬───salary─┬─sign─┬─version─┐
│      1 │ tom  │ 上海       │  25 │ 技术部 │ 30000.00 │    1 │       2 │
└────────┴──────┴────────────┴─────┴────────┴──────────┴──────┴─────────┘

```

上述案例虽然插入的数据是乱序的，依然能够实现折叠的效果，因为在定义 version 字段之后，VersionedCollapsingMergeTree 会自动将 version 作为排序条件并增加到 ORDER BY 的末端，就上述的例子而言，最终的排序字段为 ORDER BY emp_id,name，version desc。

#### GraphiteMergeTree 表引擎

    该引擎用来对 Graphite 数据进行’瘦身’及汇总。对于想使用 CH 来存储 Graphite 数据的开发者来说可能有用。

    如果不需要对 Graphite 数据做汇总，那么可以使用任意的 CH 表引擎；但若需要，那就采用 GraphiteMergeTree 引擎。它能减少存储空间，同时能提高 Graphite 数据的查询效率。

### 5. 外部集成表引擎

    ClickHouse 提供了许多与外部系统集成的方法，包括一些表引擎。这些表引擎与其他类型的表引擎类似，可以用于将外部数据导入到 ClickHouse 中，或者在 ClickHouse 中直接操作外部数据源。

    例如直接读取 HDFS 的文件或者 MySQL 数据库的表。这些表引擎只负责元数据管理和数据查询，而它们自身通常并不负责数据的写入，数据文件直接由外部系统提供。目前 ClickHouse 提供了下面的外部集成表引擎：

*   ODBC：通过指定 odbc 连接读取数据源; [暂不演示，可查看官方文档](https://clickhouse.tech/docs/zh/engines/table-engines/integrations/odbc/)
*   JDBC：通过指定 jdbc 连接读取数据源；[暂不演示，可查看官方文档](https://clickhouse.tech/docs/zh/engines/table-engines/integrations/jdbc/)
*   MySQL：将 MySQL 作为数据存储，直接查询其数据
*   HDFS：直接读取 HDFS 上的特定格式的数据文件；
*   Kafka：将 Kafka 数据导入 ClickHouse
*   RabbitMQ：与 Kafka 类似

#### HDFS

语法：`ENGINE = HDFS(URI, format)`

*   `URI`：HDFS 文件路径
*   `format`：文件格式，比如 CSV、JSON、TSV 等

**使用示例**

```
-- 建表
DROP TABLE IF EXISTS hdfs_engine_table;
CREATE TABLE hdfs_engine_table(
  emp_id UInt16 COMMENT '员工id',
  name String COMMENT '员工姓名',
  work_place String COMMENT '工作地点',
  age UInt8 COMMENT '员工年龄',
  depart String COMMENT '部门',
  salary Decimal32(2) COMMENT '工资'
) ENGINE=HDFS('hdfs://192.168.52.100:8020/clickhouse/hdfs_engine_table.csv', 'CSV');

-- 写入数据
INSERT INTO hdfs_engine_table 
VALUES (1,'tom','上海',25,'技术部',20000),(2,'jack','上海',26,'人事部',10000);
-- 再次插入就会失败，报文件已存在。一般是csv文件已经在hdfs中存在了，我们直接建表直接去读
INSERT INTO hdfs_engine_table 
VALUES (3,'bob','北京',33,'财务部',50000),(4,'tony','杭州',28,'销售事部',50000); 

-- 查询数据
select * from hdfs_engine_table;
┌─emp_id─┬─name─┬─work_place─┬─age─┬─depart─┬───salary─┐
│      1 │ tom  │ 上海       │  25 │ 技术部 │ 20000.00 │
│      2 │ jack │ 上海       │  26 │ 人事部 │ 10000.00 │
└────────┴──────┴────────────┴─────┴────────┴──────────┘

```

    这种方式与使用 Hive 类似，我们直接可以将 HDFS 对应的文件映射成 ClickHouse 中的一张表，这样就可以使用 SQL 操作 HDFS 上的文件了。** 注意：**ClickHouse 并不能够删除 HDFS 上的数据，当我们在 ClickHouse 客户端中删除了对应的表，只是删除了表结构，HDFS 上的文件并没有被删除，这一点跟 Hive 的外部表十分相似。

#### MySQL

    MySQL 引擎用于将远程的 MySQL 服务器中的表映射到 ClickHouse 中，并允许您对表进行`INSERT`和`SELECT`查询，以方便您在 ClickHouse 与 MySQL 之间进行数据交换。

**建表语法**

```
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1] [TTL expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2] [TTL expr2],
    ...
) ENGINE = MySQL('host:port', 'database', 'table', 'user', 'password'[, replace_query, 'on_duplicate_clause']);

```

**建表示例**

```
-- 连接MySQL中clickhouse数据库的test表
CREATE TABLE mysql_engine_table(
    id Int32,
    name String
) ENGINE = MySQL(
 '192.168.52.100:3306',
 'clickhouse',
 'test', 
 'root', 
 '123456');
 
-- 查询数据
SELECT * FROM mysql_engine_table;
┌─id─┬─name─────┐
│  1 │ zhangsan │
│  2 │ lisi     │
│  3 │ wangwu   │
└────┴──────────┘

-- 插入数据，会将数据插入MySQL对应的表中
-- 在MySQL中查询数据时，也会发现新增了一条数据
INSERT INTO mysql_engine_table VALUES(4,'robin');

```

**数据类型映射关系**

<table><thead><tr><th>MySQL</th><th>ClickHouse</th></tr></thead><tbody><tr><td>UNSIGNED TINYINT</td><td><a href="https://clickhouse.tech/docs/en/sql-reference/data-types/int-uint/">UInt8</a></td></tr><tr><td>TINYINT</td><td><a href="https://clickhouse.tech/docs/en/sql-reference/data-types/int-uint/">Int8</a></td></tr><tr><td>UNSIGNED SMALLINT</td><td><a href="https://clickhouse.tech/docs/en/sql-reference/data-types/int-uint/">UInt16</a></td></tr><tr><td>SMALLINT</td><td><a href="https://clickhouse.tech/docs/en/sql-reference/data-types/int-uint/">Int16</a></td></tr><tr><td>UNSIGNED INT, UNSIGNED MEDIUMINT</td><td><a href="https://clickhouse.tech/docs/en/sql-reference/data-types/int-uint/">UInt32</a></td></tr><tr><td>INT, MEDIUMINT</td><td><a href="https://clickhouse.tech/docs/en/sql-reference/data-types/int-uint/">Int32</a></td></tr><tr><td>UNSIGNED BIGINT</td><td><a href="https://clickhouse.tech/docs/en/sql-reference/data-types/int-uint/">UInt64</a></td></tr><tr><td>BIGINT</td><td><a href="https://clickhouse.tech/docs/en/sql-reference/data-types/int-uint/">Int64</a></td></tr><tr><td>FLOAT</td><td><a href="https://clickhouse.tech/docs/en/sql-reference/data-types/float/">Float32</a></td></tr><tr><td>DOUBLE</td><td><a href="https://clickhouse.tech/docs/en/sql-reference/data-types/float/">Float64</a></td></tr><tr><td>DATE</td><td><a href="https://clickhouse.tech/docs/en/sql-reference/data-types/date/">Date</a></td></tr><tr><td>DATETIME, TIMESTAMP</td><td><a href="https://clickhouse.tech/docs/en/sql-reference/data-types/datetime/">DateTime</a></td></tr><tr><td>BINARY</td><td><a href="https://clickhouse.tech/docs/en/sql-reference/data-types/fixedstring/">FixedString</a></td></tr></tbody></table>

其他的 MySQL 数据类型将全部都转换为[字符串](https://clickhouse.tech/docs/zh/sql-reference/data-types/string/)。同时以上的所有类型都支持[可为空](https://clickhouse.tech/docs/zh/sql-reference/data-types/nullable/)。

#### Kafka

**建表语法**

```
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
    ...
) ENGINE = Kafka()
SETTINGS
    kafka_broker_list = 'host:port',
    kafka_topic_list = 'topic1,topic2,...',
    kafka_group_name = 'group_name',
    kafka_format = 'data_format'[,]
    [kafka_row_delimiter = 'delimiter_symbol',]
    [kafka_schema = '',]
    [kafka_num_consumers = N,]
    [kafka_max_block_size = 0,]
    [kafka_skip_broken_messages = N,]
    [kafka_commit_every_batch = 0,]
    [kafka_thread_per_consumer = 0]

```

必要参数：

*   `kafka_broker_list` – 以逗号分隔的 brokers 列表 (`localhost:9092`)。
*   `kafka_topic_list` – topic 列表 (`my_topic`)。
*   `kafka_group_name` – Kafka 消费组名称 (`group1`)。如果不希望消息在集群中重复，请在每个分片中使用相同的组名。
*   `kafka_format` – 消息体格式。使用与 SQL 部分的 `FORMAT` 函数相同表示方法，例如 `JSONEachRow`。了解详细信息，请参考 `Formats` 部分。

可选参数：

*   `kafka_row_delimiter` - 每个消息体（记录）之间的分隔符。
*   `kafka_schema` – 如果解析格式需要一个 schema 时，此参数必填。例如，[普罗托船长](https://capnproto.org/) 需要 schema 文件路径以及根对象 `schema.capnp:Message` 的名字。
*   `kafka_num_consumers` – 单个表的消费者数量。默认值是：`1`，如果一个消费者的吞吐量不足，则指定更多的消费者。消费者的总数不应该超过 topic 中分区的数量，因为每个分区只能分配一个消费者。

**建表示例**

在 kafka 中创建 ck_topic 主题，并向该主题写入数据

```
# 启动kafka
cd /baicdt/servers/kafka_2.11-1.0.0/
nohup bin/kafka-server-start.sh config/server.properties 2>&1 &

# 查看Kafka当中已存在的主题
/baicdt/servers/kafka_2.11-1.0.0/bin/kafka-topics.sh \
--list \
--zookeeper 192.168.52.100:2181

#创建一个新的topic，名称为ck_topic
/baicdt/servers/kafka_2.11-1.0.0/bin/kafka-topics.sh \
--create \
--zookeeper 192.168.52.100:2181 \
--replication-factor 1 \
--partitions 3 \
--topic ck_topic


# 模拟生产者生产数据  {"id":1,"name":"zhangsan"}   {"id":2,"name":"lisi"}
/baicdt/servers/kafka_2.11-1.0.0/bin/kafka-console-producer.sh --broker-list 192.168.52.100:9092 --topic ck_topic

# 模拟消费者进行消费数据
/baicdt/servers/kafka_2.11-1.0.0/bin/kafka-console-consumer.sh \
--from-beginning \
--topic ck_topic  \
--zookeeper 192.168.52.100:2181

# 删除topic
/baicdt/servers/kafka_2.11-1.0.0/bin/kafka-topics.sh \
--zookeeper 192.168.52.100:2181 \
--delete \
--topic ck_topic

```

在 clickhouse 中建 kafka_table 表消费数据

```
-- 建表 kafka_format = 'JSONEachRow' 会自动解析字符串{"id":1,"name":"zhangsan"}
-- 如果 kafka_format = 'CSV' 则用来解析 1,"zhangsan" 这种格式的字符串
DROP TABLE IF EXISTS kafka_table;
CREATE TABLE kafka_table (
    id Int32,
    name String
  ) ENGINE = Kafka()
    SETTINGS
    kafka_broker_list = '192.168.52.100:9092',
    kafka_topic_list = 'ck_topic',
    kafka_group_name = 'custom_group_01',
    kafka_format = 'JSONEachRow';

-- 查询，数据查询到一次后再次查询不到了，即无法重复消费
select * from kafka_table;

```

** 注意：** 当我们一旦查询完毕之后，ClickHouse 会删除表内的数据，其实 Kafka 表引擎只是一个数据管道，我们可以通过物化视图的方式访问 Kafka 中的数据。

如何更合理的使用 Kafka 表引擎呢？

*   首先创建一张 Kafka 表引擎的表，用于从 Kafka 中读取数据
*   然后再创建一张普通表引擎的表，比如 MergeTree，面向终端用户使用
*   最后创建物化视图，用于将 Kafka 引擎表实时同步到终端用户所使用的表中

```
--  创建Kafka引擎表
 CREATE TABLE kafka_table_consumer (
    id UInt64,
    name String
  ) ENGINE = Kafka()
    SETTINGS
    kafka_broker_list = '192.168.52.100:9092',
    kafka_topic_list = 'ck_topic',
    kafka_group_name = 'custom_group_02',
    kafka_format = 'JSONEachRow';

-- 创建一张终端用户使用的表
CREATE TABLE kafka_table_mergetree (
  id UInt64 ,
  name String
)ENGINE=MergeTree()
ORDER BY id;
  
-- 创建物化视图，同步数据
CREATE MATERIALIZED VIEW view_consumer TO kafka_table_mergetree
    AS SELECT id,name FROM kafka_table_consumer;
    
-- 查询，多次查询，已经被查询的数据依然会被输出
select * from kafka_table_mergetree;

```

### 6. 其他表引擎

#### Distributed 表引擎

    Distributed 表引擎是分布式表的代名词，它自身不存储任何数据，数据都分散存储在某一个分片上，能够自动路由数据至集群中的各个节点，所以 Distributed 表引擎需要和其他数据表引擎一起协同工作。

    所以，一张分布式表底层会对应多个本地分片数据表，由具体的分片表存储数据，分布式表与分片表是一对多的关系。

    建表语法和使用示例请参照 第三章第 5 节。

#### Memory 表引擎

    Memory 表引擎直接将数据保存在内存中，数据既不会被压缩也不会被格式转换。当 ClickHouse 服务重启的时候，Memory 表内的数据会全部丢失。一般在测试时使用。

```
 CREATE TABLE table_memory (
    id UInt64,
    name String
  ) ENGINE = Memory();

```
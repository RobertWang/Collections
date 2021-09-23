> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1632366767&ver=3331&signature=hjqrTE5EAuIZ2TFQoMLs8*eklrI*r9hH*vmPvQ9tkPJ1JAa2osy1kwft8FTRWou*k9bKMBSDIXr-IY7wkGAi4Oe1IYlXpUWzxCvfdDOYq4FoolbZL5oL8JTVQp68fUKx&new=1)

> 本文介绍了 ClickHouse 的安装、常用命令及引擎，同时总结了对于 ClickHouse 存在的优劣势以及使用优化总结。

作者介绍

周安行，4 年大数据从业经验，3 年数据库、自动化测试经验，主导万亿级自研 OLAP 数据库 HydrogenSQL 测试实施与定制专项测试。

ClickHouse 是俄罗斯的重要网络服务门户之一 Yandex 所开源的一套针对数据仓库场景的多维数据存储与检索工具，一个用于联机分析 (OLAP) 的列式数据库管理系统 (DBMS), 它通过针对性的设计力图解决海量多维度数据的查询性能问题。  

下面，enjoy：  

![](https://mmbiz.qpic.cn/mmbiz_jpg/b56DXVoSgl9ewE8NQUqiaONwMxpyrIzwwBnrlZChLTgdUG9qe0a4f3IbhZdQyiaAYmIiarU5DIGVbWVTHuQuzLbfA/640?wx_fmt=jpeg)

**数据库的行存与列存**

在传统的行式数据库系统中，数据按顺序存储：

处于同一行中的数据总是被物理的存储在一起，常见的行式数据库系统有：MySQL、Postgres 和 MS SQL Server。

![](https://mmbiz.qpic.cn/mmbiz_gif/b56DXVoSgl9ewE8NQUqiaONwMxpyrIzwwsAbcnCE6t1nMR2Uv9ibZ2M2pxYiaL3KTk39XEQ4iaXtVwEUtukib88coFg/640?wx_fmt=gif)

在列式数据库系统中，来自不同列的值被单独存储，来自同一列的数据被存储在一起。列式数据库更适合于 OLAP 场景 (对于大多数查询而言，处理速度至少提高了 100 倍)。新兴的 Hbase、HP Vertica、EMC Greenplum 等分布式数据库均采用列式存储。

![](https://mmbiz.qpic.cn/mmbiz_gif/b56DXVoSgl9ewE8NQUqiaONwMxpyrIzwwicXmoj8vD1K3CN8uKicmp4jicojv18ROicibByTXObDdqawAwbqJaHic7ybQ/640?wx_fmt=gif)

ClickHouse 采取的就是列示存储的方式。

![](https://mmbiz.qpic.cn/mmbiz_jpg/b56DXVoSgl9ewE8NQUqiaONwMxpyrIzwwwiaazByFdM8OE2okxWbfOkrP9m7a6vRVR1HBNCeb1K8KOhSsSH6Dt8Q/640?wx_fmt=jpeg)

**ClickHouse 安装及常用命令参数**

**1.ClickHouse 支持的操作系统和硬件环境**

只要是 Linux，64 位都可以。优先支持 Ubuntu，Ubuntu 有官方编译好的安装包可以使用。其次是 CentOS 和 RedHat，有第三方组织编译好的 rpm 包可以使用。

如果是其他 Linux 系统，需要自己编译源码。

而且，机器的 CPU 必须支持 SSE 4.2 指令集。

[root@localhost ~]#  grep -q sse4_2 /proc/cpuinfo && echo “SSE 4.2 supported” || echo “SSE 4.2 not supported”

**2.ClickHouse 的安装方法**

**(1)RPM 安装包**

推荐使用 CentOS、RedHat 和所有其他基于 rpm 的 Linux 发行版的官方预编译 rpm 包。

首先，您需要添加官方存储库：

sudo yum install yum-utils

sudo rpm --import https://repo.clickhouse.tech/CLICKHOUSE-KEY.GPG

sudo yum-config-manager --add-repo https://repo.clickhouse.tech/rpm/stable/x86_64

然后运行命令安装：

sudo yum install clickhouse-server clickhouse-client

**(2) 设置系统参数**

CentOS 取消打开文件数限制

在 / etc/security/limits.conf、/etc/security/limits.d/90-nproc.conf 这 2 个文件的末尾加入以下内容：

* soft nofile 65536

* hard nofile 65536

* soft nproc 131072

* hard nproc 131072

CentOS 取消 SELINUX

修改 / etc/selinux/config 中的 SELINUX=disabled 后重启

关闭防火墙

Centos 6.X：

service iptables stop

service ip6tables stop

Centos 7.X：

chkconfig iptables off

chkconfig ip6tables off

**(3) 启动连接**

启动服务：service clickhouse-server start

连接客户端：clickhouse-client  

**3.ClickHouse 常用命令参数**

**(1) 进入交互式客户端**

用 clickhouse-client 连接本机 clickhouse-server 服务器：

clickhouse-client -m

用本机 clickhouse-client 连接远程 clickhouse-server 服务器：

clickhouse-client –host 192.168.x.xxx –port 9000 –database default–user default –password ””

启动失败可以查看日志，日志的目录默认为 / var/log/clickhouse-server / 下。

**(2) 服务**

停止：

service clickhouse-server stop

启动：

service clickhouse-server start

重启：

service clickhouse-server restart

**(3) 设置数据目录**

vi /etc/clickhouse-server/config.xml  

![](https://mmbiz.qpic.cn/mmbiz_png/b56DXVoSgl9ewE8NQUqiaONwMxpyrIzww2FndVYF2wvXKG9ia8ricX4XaaQ3olGStQicC8fCibYJR2TK6cAIOZYc8hg/640?wx_fmt=png)

**(4) 放开远程访问**

vi /etc/clickhouse-server/config.xml

修改服务器的配置文件 / etc/clickhouse-server/config.xml，第 65 行，放开注释即可。

![](https://mmbiz.qpic.cn/mmbiz_png/b56DXVoSgl9ewE8NQUqiaONwMxpyrIzwwuaiceIiaU0ib07GYSePUX3L6egmLwWHkqtcynXFURRiaRbS779ibG4LkSgg/640?wx_fmt=png)

**(5) 内存限制设置**

vi /etc/clickhouse-server/users.xml

![](https://mmbiz.qpic.cn/mmbiz_png/b56DXVoSgl9ewE8NQUqiaONwMxpyrIzwwJXV7pBibzfF2xhUhTsYheNRE3unwgZN3d9bRb6XOc7D4egNUFQy2IqQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_jpg/b56DXVoSgl9ewE8NQUqiaONwMxpyrIzww68e5cOtsaLIdIq6H0oToQlLftowg8UhubLADo88L3DavfVGXbVxdiaw/640?wx_fmt=jpeg)

**ClickHouse 引擎介绍**

ClickHouse 提供了大量的数据引擎，分为数据库引擎、表引擎，根据数据特点及使用场景选择合适的引擎至关重要。

**1.ClickHouse 引擎分类**

<table><tbody><tr><td data-darkmode-bgcolor-16323667706786="rgb(79, 129, 189)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(79, 129, 189)" data-style="padding: 5px 10px; border-width: 1px 1px 3px; border-color: rgb(79, 129, 189) rgb(79, 129, 189) rgb(255, 255, 255); background: rgb(79, 129, 189);"><p data-darkmode-bgcolor-16323667706786="rgb(79, 129, 189)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(79, 129, 189)"><strong data-darkmode-bgcolor-16323667706786="rgb(79, 129, 189)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(79, 129, 189)"><span data-darkmode-bgcolor-16323667706786="rgb(79, 129, 189)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(79, 129, 189)" data-darkmode-color-16323667706786="rgb(255, 255, 255)" data-darkmode-original-color-16323667706786="#fff|rgb(255, 255, 255)">引擎分类</span></strong></p></td><td colspan="2" data-darkmode-bgcolor-16323667706786="rgb(79, 129, 189)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(79, 129, 189)" data-style="padding: 5px 10px; border-left: none; border-right-width: 1px; border-right-color: rgb(79, 129, 189); border-top-width: 1px; border-top-color: rgb(79, 129, 189); border-bottom-width: 3px; border-bottom-color: rgb(255, 255, 255); background: rgb(79, 129, 189);"><p data-darkmode-bgcolor-16323667706786="rgb(79, 129, 189)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(79, 129, 189)"><strong data-darkmode-bgcolor-16323667706786="rgb(79, 129, 189)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(79, 129, 189)"><span data-darkmode-bgcolor-16323667706786="rgb(79, 129, 189)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(79, 129, 189)" data-darkmode-color-16323667706786="rgb(255, 255, 255)" data-darkmode-original-color-16323667706786="#fff|rgb(255, 255, 255)">引擎名称</span></strong></p></td></tr><tr><td data-darkmode-bgcolor-16323667706786="rgb(174, 193, 216)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(184, 204, 228)" data-style="padding: 5px 10px; border-left-width: 1px; border-left-color: rgb(79, 129, 189); border-right-width: 1px; border-right-color: rgb(79, 129, 189); border-top: none; border-bottom-width: 1px; border-bottom-color: rgb(79, 129, 189); background: rgb(184, 204, 228);"><p data-darkmode-bgcolor-16323667706786="rgb(174, 193, 216)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(184, 204, 228)"><span data-darkmode-bgcolor-16323667706786="rgb(174, 193, 216)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(184, 204, 228)" data-darkmode-color-16323667706786="rgb(0, 0, 0)" data-darkmode-original-color-16323667706786="#fff|rgb(0, 0, 0)">数据库引擎</span></p></td><td colspan="2" data-darkmode-bgcolor-16323667706786="rgb(174, 193, 216)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(184, 204, 228)" data-style="padding: 5px 10px; border-left: none; border-right-width: 1px; border-right-color: rgb(79, 129, 189); border-top: none; border-bottom-width: 1px; border-bottom-color: rgb(79, 129, 189); background: rgb(184, 204, 228);"><p data-darkmode-bgcolor-16323667706786="rgb(174, 193, 216)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(184, 204, 228)"><span data-darkmode-bgcolor-16323667706786="rgb(174, 193, 216)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(184, 204, 228)" data-darkmode-color-16323667706786="rgb(0, 0, 0)" data-darkmode-original-color-16323667706786="#fff|rgb(0, 0, 0)">Ordinary/Dictionary/Memory/Lazy/MySQL</span></p></td></tr><tr><td rowspan="4" data-darkmode-bgcolor-16323667706786="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(255, 255, 255)" data-style="padding: 8px 14px; border-left-width: 1px; border-left-color: rgb(79, 129, 189); border-right-width: 1px; border-right-color: rgb(79, 129, 189); border-top: none; border-bottom-width: 1px; border-bottom-color: rgb(79, 129, 189); background: rgb(255, 255, 255);"><p data-darkmode-bgcolor-16323667706786="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(255, 255, 255)"><span data-darkmode-bgcolor-16323667706786="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(255, 255, 255)" data-darkmode-color-16323667706786="rgb(163, 163, 163)" data-darkmode-original-color-16323667706786="#fff|rgb(0, 0, 0)" data-style="font-family: 微软雅黑; color: rgb(0, 0, 0); font-size: 16px;">数据表引擎</span></p></td><td data-darkmode-bgcolor-16323667706786="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(255, 255, 255)" data-style="padding: 5px 10px; border-left: none; border-right-width: 1px; border-right-color: rgb(79, 129, 189); border-top: none; border-bottom-width: 1px; border-bottom-color: rgb(79, 129, 189); background: rgb(255, 255, 255);"><p data-darkmode-bgcolor-16323667706786="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(255, 255, 255)"><span data-darkmode-bgcolor-16323667706786="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(255, 255, 255)" data-darkmode-color-16323667706786="rgb(163, 163, 163)" data-darkmode-original-color-16323667706786="#fff|rgb(0, 0, 0)" data-style="font-family: 微软雅黑; color: rgb(0, 0, 0); font-size: 16px;">MergeTree 系列</span></p></td><td data-darkmode-bgcolor-16323667706786="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(255, 255, 255)" data-style="padding: 5px 10px; border-left: none; border-right-width: 1px; border-right-color: rgb(79, 129, 189); border-top-width: 1px; border-top-color: rgb(79, 129, 189); border-bottom-width: 1px; border-bottom-color: rgb(79, 129, 189); background: rgb(255, 255, 255);"><p data-darkmode-bgcolor-16323667706786="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(255, 255, 255)"><span data-darkmode-bgcolor-16323667706786="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(255, 255, 255)" data-darkmode-color-16323667706786="rgb(163, 163, 163)" data-darkmode-original-color-16323667706786="#fff|rgb(0, 0, 0)" data-style="font-family: 微软雅黑; line-height: 150%; color: rgb(0, 0, 0); font-size: 12px;">MergeTree 、ReplacingMergeTree 、SummingMergeTree 、 AggregatingMergeTree CollapsingMergeTree 、 VersionedCollapsingMergeTree 、GraphiteMergeTree</span></p></td></tr><tr><td data-darkmode-bgcolor-16323667706786="rgb(174, 193, 216)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(184, 204, 228)" data-style="padding: 8px 14px; border-left: none; border-right-width: 1px; border-right-color: rgb(79, 129, 189); border-top: none; border-bottom-width: 1px; border-bottom-color: rgb(79, 129, 189); background: rgb(184, 204, 228);"><p data-darkmode-bgcolor-16323667706786="rgb(174, 193, 216)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(184, 204, 228)"><span data-darkmode-bgcolor-16323667706786="rgb(174, 193, 216)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(184, 204, 228)" data-darkmode-color-16323667706786="rgb(0, 0, 0)" data-darkmode-original-color-16323667706786="#fff|rgb(0, 0, 0)">Log 系列</span></p></td><td data-darkmode-bgcolor-16323667706786="rgb(174, 193, 216)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(184, 204, 228)" data-style="padding: 5px 10px; border-left: none; border-right-width: 1px; border-right-color: rgb(79, 129, 189); border-top: none; border-bottom-width: 1px; border-bottom-color: rgb(79, 129, 189); background: rgb(184, 204, 228);"><p data-darkmode-bgcolor-16323667706786="rgb(174, 193, 216)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(184, 204, 228)"><span data-darkmode-bgcolor-16323667706786="rgb(174, 193, 216)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(184, 204, 228)" data-darkmode-color-16323667706786="rgb(0, 0, 0)" data-darkmode-original-color-16323667706786="#fff|rgb(0, 0, 0)">TinyLog 、StripeLog 、Log</span></p></td></tr><tr><td data-darkmode-bgcolor-16323667706786="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(255, 255, 255)" data-style="padding: 8px 14px; border-left: none; border-right-width: 1px; border-right-color: rgb(79, 129, 189); border-top: none; border-bottom-width: 1px; border-bottom-color: rgb(79, 129, 189); background: rgb(255, 255, 255);"><p data-darkmode-bgcolor-16323667706786="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(255, 255, 255)"><span data-darkmode-bgcolor-16323667706786="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(255, 255, 255)" data-darkmode-color-16323667706786="rgb(163, 163, 163)" data-darkmode-original-color-16323667706786="#fff|rgb(0, 0, 0)" data-style="font-family: 微软雅黑; line-height: 150%; color: rgb(0, 0, 0); font-size: 15px;">Integration Engines</span></p></td><td data-darkmode-bgcolor-16323667706786="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(255, 255, 255)" data-style="padding: 5px 10px; border-left: none; border-right-width: 1px; border-right-color: rgb(79, 129, 189); border-top: none; border-bottom-width: 1px; border-bottom-color: rgb(79, 129, 189); background: rgb(255, 255, 255);"><p data-darkmode-bgcolor-16323667706786="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(255, 255, 255)"><span data-darkmode-bgcolor-16323667706786="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(255, 255, 255)" data-darkmode-color-16323667706786="rgb(163, 163, 163)" data-darkmode-original-color-16323667706786="#fff|rgb(0, 0, 0)" data-style="font-family: 微软雅黑; line-height: 150%; color: rgb(0, 0, 0); font-size: 12px;">Kafka 、MySQL、ODBC 、JDBC、HDFS</span></p></td></tr><tr><td data-darkmode-bgcolor-16323667706786="rgb(174, 193, 216)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(184, 204, 228)" data-style="padding: 8px 14px; border-left: none; border-right-width: 1px; border-right-color: rgb(79, 129, 189); border-top: none; border-bottom-width: 1px; border-bottom-color: rgb(79, 129, 189); background: rgb(184, 204, 228);"><p data-darkmode-bgcolor-16323667706786="rgb(174, 193, 216)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(184, 204, 228)"><span data-darkmode-bgcolor-16323667706786="rgb(174, 193, 216)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(184, 204, 228)" data-darkmode-color-16323667706786="rgb(0, 0, 0)" data-darkmode-original-color-16323667706786="#fff|rgb(0, 0, 0)">Special Engines</span></p></td><td data-darkmode-bgcolor-16323667706786="rgb(174, 193, 216)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(184, 204, 228)" data-style="padding: 5px 10px; border-left: none; border-right-width: 1px; border-right-color: rgb(79, 129, 189); border-top: none; border-bottom-width: 1px; border-bottom-color: rgb(79, 129, 189); background: rgb(184, 204, 228);"><p data-darkmode-bgcolor-16323667706786="rgb(174, 193, 216)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(184, 204, 228)"><span data-darkmode-bgcolor-16323667706786="rgb(174, 193, 216)" data-darkmode-original-bgcolor-16323667706786="#fff|rgb(184, 204, 228)" data-darkmode-color-16323667706786="rgb(0, 0, 0)" data-darkmode-original-color-16323667706786="#fff|rgb(0, 0, 0)">Distributed 、MaterializedView、 Dictionary 、Merge 、File、Null 、Set 、Join 、 URL View、Memory 、 Buffer</span></p></td></tr></tbody></table>

在以下几种情况下，ClickHouse 使用自己的数据库引擎：

*   决定表存储在哪里以及以何种方式存储
    
*   支持哪些查询以及如何支持
    
*   并发数据访问
    
*   索引的使用
    
*   是否可以执行多线程请求
    
*   数据复制参数
    

在所有的表引擎中，最为核心的当属 MergeTree 系列表引擎，这些表引擎拥有最为强大的性能和最广泛的使用场合。对于非 MergeTree 系列的其他引擎而言，主要用于特殊用途，场景相对有限。而 MergeTree 系列表引擎是官方主推的存储引擎，支持几乎所有 ClickHouse 核心功能。

MergeTree 作为家族系列最基础的表引擎，主要有以下特点：

*   存储的数据按照主键排序：允许创建稀疏索引，从而加快数据查询速度
    
*   支持分区，可以通过 PRIMARY KEY 语句指定分区字段
    
*   支持数据副本
    
*   支持数据采样
    

**2. 建表引擎参数**

**ENGINE：**ENGINE = MergeTree()，MergeTree 引擎没有参数。  

**ORDER BY：**order by 设定了分区内的数据按照哪些字段顺序进行有序保存。

order by 是 MergeTree 中唯一一个必填项，甚至比 primary key 还重要，因为当用户不设置主键的情况，很多处理会依照 order by 的字段进行处理。

要求：主键必须是 order by 字段的前缀字段。

如果 ORDER BY 与 PRIMARY KEY 不同，PRIMARY KEY 必须是 ORDER BY 的前缀 (为了保证分区内数据和主键的有序性)。

ORDER BY 决定了每个分区中数据的排序规则;

PRIMARY KEY 决定了一级索引 (primary.idx);

ORDER BY 可以指代 PRIMARY KEY, 通常只用声明 ORDER BY 即可。

**PARTITION BY：**分区字段，可选。如果不填：只会使用一个分区。

分区目录：MergeTree 是以列文件 + 索引文件 + 表定义文件组成的，但是如果设定了分区那么这些文件就会保存到不同的分区目录中。

**PRIMARY KEY：**指定主键，如果排序字段与主键不一致，可以单独指定主键字段。否则默认主键是排序字段。可选。

**SAMPLE BY：**采样字段，如果指定了该字段，那么主键中也必须包含该字段。比如 SAMPLE BY intHash32(UserID) ORDER BY (CounterID, EventDate, intHash32(UserID))。可选。

**TTL：**数据的存活时间。在 MergeTree 中，可以为某个列字段或整张表设置 TTL。当时间到达时，如果是列字段级别的 TTL，则会删除这一列的数据；如果是表级别的 TTL，则会删除整张表的数据。可选。

**SETTINGS：**额外的参数配置。可选。

**3. 建表示例**  

![](https://mmbiz.qpic.cn/mmbiz_png/b56DXVoSgl9ewE8NQUqiaONwMxpyrIzwwm42tKMWkeRnuefxugLlic1RxCPA0ZicibaFlQice4oD1V8MJd1PYlibbGhw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/b56DXVoSgl9ewE8NQUqiaONwMxpyrIzwwonKj0VZ4vcyg3XmDe9xH4viabudwXo6HicmqyDNAbiaWZudWPfeiatzlhA/640?wx_fmt=png)

**4. 数据导入**

clickhouse-client --query "INSERT INTO default.emp_mgetree FORMAT CSV" --max_insert_block_size=100000 < test_data.csv

虽然 clickhouse 底层将 DateTime 存储为时间戳 Long 类型，但不建议直接存储 Long 类型，因为 DateTime 不需要经过函数转换处理，执行效率高、可读性好。

官方已经指出 Nullable 类型几乎总是会拖累性能，因为存储 Nullable 列时需要创建一个额外的文件来存储 NULL 的标记，并且 Nullable 列无法被索引。因此除非极特殊情况，应直接使用字段默认值表示空，或者自行指定一个在业务中无意义的值（例如用 - 1 表示没有商品 ID）。

必须指定索引列，clickhouse 中的索引列即排序列，通过 order by 指定，一般在查询条件中经常被用来充当筛选条件的属性被纳入进来；可以是单一维度，也可以是组合维度的索引；通常需要满足高级列在前、查询频率大的在前原则；还有基数特别大的不适合做索引列，如用户表的 userid 字段；通常筛选后的数据满足在百万以内为最佳。

clickhouse 不支持设置多数据目录，为了提升数据 io 性能，可以挂载虚拟券组，一个券组绑定多块物理磁盘提升读写性能；多数查询场景 SSD 盘会比普通机械硬盘快 2-3 倍。
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [my.oschina.net](https://my.oschina.net/actiontechoss/blog/10092464)

> 作者通过一个案例详细说明了 MySQL 8.0 字段信息统计机制的相关参数和使用方式。

作者通过一个案例详细说明了 MySQL 8.0 字段信息统计机制的相关参数和使用方式。

> 作者：杨奇龙
> 
> 网名 “北在南方”，资深 DBA，主要负责数据库架构设计和运维平台开发工作，擅长数据库性能调优、故障诊断。
> 
> 本文来源：原创投稿
> 
> 爱可生开源社区出品，原创内容未经授权不得随意使用，转载请联系小编并注明来源。

前几天有同事在咨询一个问题：某个业务基于 `INFORMATION_SCHEMA` 统计表的信息（比如最大值）向表里面插入数据。

**请问 `INFORMATION_SCHEMA.TABLES` 中的 `AUTO_INCREMENT` 会不会及时的更新呢？**

先说结论：**可以！**

> 这里涉及到 [信息统计机制](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F8.0%2Fen%2Finformation-schema-optimization.html "信息统计机制") 或者说频率问题，主要由参数 `information_schema_stats_expiry` 控制。

表信息更新的基本逻辑
----------

默认情况下，MySQL 会高效的从 系统表 `mysql.index_stats` 和 `mysql.table_stats` 中检索这些列的缓存值，而不是直接从存储引擎中获取统计信息。如果缓存的统计信息不可用或已过期，MySQL 将从存储引擎中检索最新的统计信息，并将其统计信息更新并缓存在 `mysql.index_stats` 和 `mysql.table_stats` 字典表中。后续查询将检索缓存的统计信息，直到缓存的统计数据过期。

值得注意的是：**MySQL 重新启动或第一次打开 `mysql.index_stats` 和 `mysql.table_stats` 表不会自动更新缓存的统计信息。**

核心参数
----

核心参数 `information_schema_stats_expiry` 默认是 86400 秒。也就是说每隔一天自动收集一次相关统计信息到 `information_schema` 中的如下表字段中：

```
STATISTICS.CARDINALITY
TABLES.AUTO_INCREMENT
TABLES.AVG_ROW_LENGTH
TABLES.CHECKSUM
TABLES.CHECK_TIME
TABLES.CREATE_TIME
TABLES.DATA_FREE
TABLES.DATA_LENGTH
TABLES.INDEX_LENGTH
TABLES.MAX_DATA_LENGTH
TABLES.TABLE_ROWS
TABLES.UPDATE_TIME


```

参数 `information_schema_stats_expiry` 的值决定再次收集表的统计信息的时间间隔，默认 86400 秒。如果设置为 0 ，则表示**实时**更新统计信息，当然势必会影响一部分性能。

在以下情况中，查询统计信息列不会在 `mysql.index_stats` 和 `mysql.table_stats` 字典表中存储或更新统计信息：

1.  缓存的统计信息尚未过期时。
2.  当 `information_schema_stas_expiry` 设置为 0 时。
3.  当 MySQL server 处于只读、超级只读、事务只读或 `innodb_read_only` 模式时。
4.  查询还获取 `Performance Schema` 的数据时。

`information_schema_stas_experity` 支持全局和会话级别，每个会话都可以定义自己的过期值。从存储引擎中检索并由一个会话缓存的统计信息可用于其他会话。

本文以 MySQL 8.0.30 为例，进行分析。

2.1 测试准备
--------

```
 CREATE TABLE `sbtest1` (
 `id` int NOT NULL AUTO_INCREMENT,
 `k` int NOT NULL DEFAULT '0',
 `c` char(120) NOT NULL DEFAULT '',
 `pad` char(60) NOT NULL DEFAULT '',
 PRIMARY KEY (`id`)
) ENGINE=InnoDB ;


```

2.2 测试
------

查看 `information_schema.tables` 中 `sbtest1` 的当前信息，最大值 1200006，表结构定义中自增最大值也是 1200006。

```
master [localhost:22031] {msandbox} (test) > show variables like 'information_schema_stats_expiry' ;
+-----------------------------------------+---------+
| Variable_name                   | Value |
+-----------------------------------------+---------+
| information_schema_stats_expiry | 86400 |
+-----------------------------------------+---------+
1 row in set (0.02 sec)

master [localhost:22031] {msandbox} (test) > select  table_name, AUTO_INCREMENT  from information_schema.tables where  table_name='sbtest1';
+---------------+--------------------+
| TABLE_NAME | AUTO_INCREMENT |
+---------------+--------------------+
| sbtest1    |        1200006 |
+---------------+--------------------+
1 row in set (0.01 sec)

master [localhost:22031] {msandbox} (test) > show create table  sbtest1 \G
*************************** 1. row ***************************
       Table: sbtest1
Create Table: CREATE TABLE `sbtest1` (
  `id` int NOT NULL AUTO_INCREMENT,
  `k` int NOT NULL DEFAULT '0',
  `c` char(120) NOT NULL DEFAULT '',
  `pad` char(60) NOT NULL DEFAULT '',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1200006 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
1 row in set (0.00 sec)


```

插入新的数据，自增值加 1。

```
master [localhost:22031] {msandbox} (test) > insert into sbtest1(k,c,pad) values(1,'c','cc');
Query OK, 1 row affected (0.00 sec)

master [localhost:22031] {msandbox} (test) > show create table  sbtest1 \G
*************************** 1. row ***************************
       Table: sbtest1
Create Table: CREATE TABLE `sbtest1` (
  `id` int NOT NULL AUTO_INCREMENT,
  `k` int NOT NULL DEFAULT '0',
  `c` char(120) NOT NULL DEFAULT '',
  `pad` char(60) NOT NULL DEFAULT '',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1200007 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
1 row in set (0.00 sec)


```

但是 `information_schema.tables` 中的值并未发生变化。

```
master [localhost:22031] {msandbox} (test) > select  table_name, AUTO_INCREMENT  from information_schema.tables where  table_name='sbtest1';
+
| TABLE_NAME | AUTO_INCREMENT |
+
| sbtest1    |        1200006 |
+
1 row in set (0.00 sec)


```

### 设置为实时更新

修改 `information_schema_stats_expiry` 为 0。

```
master [localhost:22031] {msandbox} (test) > set  information_schema_stats_expiry=0;
Query OK, 0 rows affected (0.00 sec)

master [localhost:22031] {msandbox} (test) > show variables like 'information_schema_stats_expiry' ;
+---------------------------------+-------+
| Variable_name                   | Value |
+---------------------------------+-------+
| information_schema_stats_expiry | 0     |
+---------------------------------+-------+
1 row in set (0.00 sec)

master [localhost:22031] {msandbox} (test) > select  table_name, AUTO_INCREMENT  from information_schema.tables where  table_name='sbtest1';
+------------+----------------+
| TABLE_NAME | AUTO_INCREMENT |
+------------+----------------+
| sbtest1    |        1200007 |
+------------+----------------+
1 row in set (0.00 sec)

master [localhost:22031] {msandbox} (test) > show create table  sbtest1 \G
*************************** 1. row ***************************
       Table: sbtest1
Create Table: CREATE TABLE `sbtest1` (
  `id` int NOT NULL AUTO_INCREMENT,
  `k` int NOT NULL DEFAULT '0',
  `c` char(120) NOT NULL DEFAULT '',
  `pad` char(60) NOT NULL DEFAULT '',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1200007 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
1 row in set (0.00 sec)


```

插入数据自增加 1 ，查询 `information_schema.tables` 中的 **自增列统计值** 也是实时更新。

```
master [localhost:22031] {msandbox} (test) > insert into sbtest1(k,c,pad) values(1,'c','cc');
Query OK, 1 row affected (0.00 sec)

master [localhost:22031] {msandbox} (test) > select  table_name, AUTO_INCREMENT  from information_schema.tables where  table_name='sbtest1';
+------------+----------------+
| TABLE_NAME | AUTO_INCREMENT |
+------------+----------------+
| sbtest1    |        1200008 |
+------------+----------------+
1 row in set (0.00 sec)

master [localhost:22031] {msandbox} (test) > show create table  sbtest1 \G
*************************** 1. row ***************************
       Table: sbtest1
Create Table: CREATE TABLE `sbtest1` (
  `id` int NOT NULL AUTO_INCREMENT,
  `k` int NOT NULL DEFAULT '0',
  `c` char(120) NOT NULL DEFAULT '',
  `pad` char(60) NOT NULL DEFAULT '',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1200008 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
1 row in set (0.00 sec)


```

手工修改自增列，也是可以实时更新。

```
master [localhost:22031] {msandbox} (test) > alter table sbtest1  AUTO_INCREMENT=1200010;
Query OK, 0 rows affected (0.00 sec)
Records: 0  Duplicates: 0  Warnings: 0

master [localhost:22031] {msandbox} (test) > select  table_name, AUTO_INCREMENT  from information_schema.tables where  table_name='sbtest1';
+
| TABLE_NAME | AUTO_INCREMENT |
+
| sbtest1    |        1200010 |
+
1 row in set (0.00 sec)


```

MySQL 8.0 对于表字段的统计信息提供更多的技术特性来支持。统计有效性时长，字段本身的直方图，使用起来越来越便利。

回过头来看这个需求，其实如果是业务监控或者数据库监控信息比如监控表的主键最大值是否溢出，比较实际的建议还是查询具体的表的最大 `id` 值，比较实际，查询频率看可控。

更多技术文章，请访问：[https://opensource.actionsky.com/](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fopensource.actionsky.com%2F)

关于 SQLE
-------

爱可生开源社区的 SQLE 是一款面向数据库使用者和管理者，支持多场景审核，支持标准化上线流程，原生支持 MySQL 审核且数据库类型可扩展的 SQL 审核工具。

SQLE 获取
-------

<table><thead><tr><th>类型</th><th>地址</th></tr></thead><tbody><tr><td>版本库</td><td><a href="https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Factiontech%2Fsqle" target="_blank">https://github.com/actiontech/sqle</a></td></tr><tr><td>文档</td><td><a href="https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Factiontech.github.io%2Fsqle-docs%2F" target="_blank">https://actiontech.github.io/sqle-docs/</a></td></tr><tr><td>发布信息</td><td><a href="https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Factiontech%2Fsqle%2Freleases" target="_blank">https://github.com/actiontech/sqle/releases</a></td></tr><tr><td>数据审核插件开发文档</td><td><a href="https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Factiontech.github.io%2Fsqle-docs%2Fdocs%2Fdev-manual%2Fplugins%2Fhowtouse" target="_blank">https://actiontech.github.io/sqle-docs/docs/dev-manual/plugins/howtouse</a></td></tr></tbody></table>
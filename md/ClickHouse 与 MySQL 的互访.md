> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1622873955&ver=3111&signature=XOEuACIdHzgV32JW2yCORjrCQIQact-u5lAIjYmEZEdL*qgdDOlMNnR1TCKwZILuV0vV-fWclbVQvDzsf*CQ8xoITk9xNeFA7v-6rdcscPaJFKMVhdmjKhz6I8gYt4wa&new=1)

ClickHouse 与 MySQL 的互访
======================

前言
--

如果你熟悉 PostgreSQL，推荐你看 [ClickHouse 与 PostgreSQL 互访](https://mp.weixin.qq.com/s?__biz=MzU1NTg2ODQ5Nw==&mid=2247486139&idx=1&sn=a14b3691b529ee0f950e921087872126&chksm=fbcc8285ccbb0b9323159cdd1de89b74cd809eb45aa726b09bac8a80c78282abc803c04cbca3&token=1219143568&lang=zh_CN&scene=21#wechat_redirect)这篇文章。

[ClickHouse vs MariaDB ColumnStore](https://mp.weixin.qq.com/s?__biz=MzU1NTg2ODQ5Nw==&mid=2247486043&idx=1&sn=e2209c000466959667d60d1218c410bd&chksm=fbcc8265ccbb0b73d0b3121d36d874294717fed575b726dac23c9b84dfe1104c5869d7d4616b&token=1219143568&lang=zh_CN&scene=21#wechat_redirect) 文章部分 MySQL 没有资格入场 PK，但合作还是可以。今天介绍的这篇文章 ClickHouse and MySQL – Better Together[1]（作者：Vadim Tkachenko）， 主要是让熟悉 MySQL 的人也可以快速上手 ClickHouse

ClickHouse 提供了两个和 MySQL 有关的功能，

1.  使用 MySQL 协议和 MySQL 客户端连接到 ClickHouse
    
2.  使用 MySQL 表选择和关联 ClickHouse 表
    

ClickHouse Server config.xml 默认添加了 MySQL 协议的支持

```
<!-- Compatibility with MySQL protocol.
         ClickHouse will pretend to be MySQL for applications connecting to this port.
    -->
    <mysql_port>9004</mysql_port>
```

启动 ClickHouse 后会看到这样的日志

```
<Information> Application: Listening for MySQL compatibility protocol: [::1]:9004
<Information> Application: Listening for MySQL compatibility protocol: 127.0.0.1:9004
```

接下来，就可以使用 MySQL 客户端登录了，

MySQL 客户端通过 MySQL 协议访问 ClickHouse
---------------------------------

*   登录
    

除了端口和 clickhouse 客户端访问时不一样外，其余一样，

```
$ mysql -h127.0.0.1 -P9004 -udefault -pqwert
```

*   查看版本信息
    

```
select version();
+-------------+
| version()   |
+-------------+
| 21.3.1.6016 |
+-------------+
1 row in set (0.005 sec)
```

很明细，这是我系统上 ClickHose 的版本信息。

接下来以 ontime[2] 数据（本例起止时间为 2017 年 1 月～ 2019 年 11 月) 为例做些尝试，

*   切换数据库及显示表名
    

```
mysql> use default;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+--------+
| name   |
+--------+
| ontime |
+--------+
1 row in set (0.00 sec)
Read 1 rows, 31.00 B in 0.000 sec., 2038 rows/sec., 61.71 KiB/sec.
```

*   简单的查下总行数
    

```
select count(*) from ontime;
+----------+
| count()  |
+----------+
| 19684341 |
+----------+
1 row in set (0.00 sec)
Read 1 rows, 4.01 KiB in 0.000 sec., 2150 rows/sec., 8.42 MiB/sec.
```

*   top 10 离港的城市
    

```
mysql> SELECT Origin,count(*) cnt FROM ontime GROUP BY Origin ORDER BY cnt DESC LIMIT 10;
+--------+---------+
| Origin | cnt     |
+--------+---------+
| ATL    | 1117414 |
| ORD    |  911572 |
| DFW    |  739334 |
| DEN    |  689847 |
| LAX    |  637070 |
| CLT    |  562160 |
| SFO    |  507377 |
| PHX    |  488398 |
| IAH    |  467543 |
| LAS    |  463627 |
+--------+---------+
10 rows in set (0.03 sec)
Read 19684341 rows, 93.86 MiB in 0.027 sec., 726992466 rows/sec., 3.39 GiB/sec.
```

*   最热门的 10 条航线
    

```
mysql> SELECT Origin,Dest,count(*) cnt FROM ontime GROUP BY Origin,Dest ORDER BY cnt DESC LIMIT 10;
+--------+-------+-------+
| Origin | Dest  | cnt   |
+--------+-------+-------+
| SFO    | LAX   | 44419 |
| LAX    | SFO   | 44204 |
| LGA    | ORD   | 39276 |
| ORD    | LGA   | 39141 |
| LAX    | JFK   | 37309 |
| JFK    | LAX   | 37256 |
| LAX    | LAS   | 33780 |
| LAS    | LAX   | 33757 |
| OGG    | HNL   | 29619 |
| HNL    | OGG   | 29616 |
+--------+-------+-------+
10 rows in set (0.05 sec)
Read 19684341 rows, 187.73 MiB in 0.053 sec., 372125496 rows/sec., 3.47 GiB/sec.
```

与 MySQL 表关联
-----------

在 ClickHouse 中使用字典从 MySQL 获取数据的功能早就实现了，但是它并不方便，导致使用了非标准的 SQL 扩展。从那时起，ClickHouse 中实现了两个新功能：

*   支持 JOIN 语法
    
*   支持外部表
    

这使我们可以结合使用 MySQL 和 ClickHouse 表来运行更熟悉的查询。在转到示例之前，让我们回顾一下为什么需要这样做。

典型的数据分析设计假定存在大量事实表，其中引用了维度表（如果使用 ClickHouse 词典，则为字典）。有关详细示例，请参见星形模式 [3]。

维度表的更改（更新）频率可能更高，而 ClickHouse 并不擅长它，因为它以更多类似追加的方式运行。因此，我们可能希望将纬度表存储在 OLTP 数据库（如 MySQL）中。顺便说一句，这也有助于 GDPR 政策删除个性化数据。如果将所有个人数据存储在非规范化的事实表中，那么删除它会变得非常复杂。相反，如果个人数据存储在维度表中，则很容易将其匿名化。

因此，让我们回到我们的示例。

ontime（事实表）有一个参考字段 AirlineID，这并不是很有用，因为我们想在报告中查看航空公司名称。

可以在 BTS 网站 [4] 上找到 AirlineID – AirlineName 查找（纬度）表的数据。

看起来像这样：

![](https://mmbiz.qpic.cn/mmbiz_png/IcShsSp5mXunZxouKhnka41vjmov0InvJ1XMLCUzWAl6sMpoxHdibQxGB20enE4HmgKxYB08zf60y4fjvrrgIow/640?wx_fmt=png)

我们将此数据加载到 MySQL 表中：  

```
CREATE TABLE airlines
  id INT NOT NULL PRIMARY KEY,
  name VARCHAR(255)
)

select count(*) from airlines;
+----------+
| count(*) |
+----------+
|     1671 |
+----------+
```

现在，我们如何将此表连接到 ClickHouse？为此，在 ClickHouse 中，我们使用`MySQL table engine`创建一个表：

Clickhouse->（我们可以使用 mysql 客户端工具连接到它，请参阅第一部分）。

```
CREATE TABLE airlinesclick
(
    `id` Int32,
    `name` String
)
ENGINE = MySQL('127.0.0.1:3306', 'click', airlines, 'sbtest', 'sbtest');
```

现在，我们可以执行一个查询，该查询将 ClickHouse 事实表 (ontime) 和 MySQL 维度表 (airlinesclick) 按航班 ID 关联起来，获得离港最多的十大航空公司：

```
SELECT name,count(*) cnt FROM ontime
JOIN airlinesclick ON ontime.AirlineID=airlinesclick.id
GROUP BY name ORDER BY cnt DESC LIMIT 10;
+-----------------------------+---------+
| name                        | cnt     |
+-----------------------------+---------+
| Southwest Airlines Co.: WN  | 3931500 |
| Delta Air Lines Inc.: DL    | 2783305 |
| American Airlines Inc.: AA  | 2680537 |
| SkyWest Airlines Inc.: OO   | 2245105 |
| United Air Lines Inc.: UA   | 1780665 |
| JetBlue Airways: B6         |  875858 |
| Alaska Airlines Inc.: AS    |  673652 |
| ExpressJet Airlines LLC: EV |  665911 |
| Republic Airline: YX        |  616362 |
| Envoy Air: MQ               |  595908 |
+-----------------------------+---------+
10 rows in set (0.05 sec)
Read 19686012 rows, 75.15 MiB in 0.050 sec., 394346280 rows/sec., 1.47 GiB/sec.
```

我们使用熟悉的 JOIN 语法在 ClickHouse 中执行了一个关联查询，只是被关联的两个表存储在不同服务器中：MySQL 和 ClickHouse。

![](https://mmbiz.qpic.cn/mmbiz_png/IcShsSp5mXunZxouKhnka41vjmov0InvwSZcibw1nXuSMJvEWjjPc9psSpIE41kVsq3LiaibhTY7rpQQQmYPWZXww/640?wx_fmt=png)

最后  

-----

和 ClickHouse 提供的 PostgreSQL 协议一样，很多第三方客户端都不支持，

*   使用 DBeaver 尝试连接会报错，
    

```
Unexpected driver error occurred while connecting to the database
```

*   Python 下使用 sqlalchemy 会报错,
    

```
list index out of range
```

### 参考资料

[1]

ClickHouse and MySQL – Better Together: _https://www.percona.com/blog/2020/02/03/clickhouse-and-mysql-better-together/_

[2]

ontime: _https://clickhouse.tech/docs/en/getting-started/example-datasets/ontime/_

[3]

星形模式: _https://en.wikipedia.org/wiki/Star_schema_

[4]

BTS 网站: _https://www.transtats.bts.gov/Download_Lookup.asp?Lookup=L_AIRLINE_ID_
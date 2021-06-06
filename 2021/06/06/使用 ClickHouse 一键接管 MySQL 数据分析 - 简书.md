> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/5bfb043a075d)

*   为啥有这篇文章？
    *   很多人好奇 ClickHouse，都听说过很快，但是到底有多恐怖？
    *   新建表还要理解 ClickHouse 的引擎和数据类型，好麻烦
    *   今天，用一个简单粗暴的功能，帮你一键导入 MySQL 的数据，无需人肉建表

数据导入
----

### 第一组

```
# du出的表大小
5.5G    article_clientuser_sum.ibd

# ClickHouse操作语句
CREATE TABLE article_clientuser_sum
ENGINE = MergeTree
ORDER BY id AS
SELECT *
FROM mysql('host:port', 'db', 'article_clientuser_sum', 'user', 'password') 

# 耗时和平均速度
0 rows in set. Elapsed: 137.251 sec. Processed 18.59 million rows, 7.34 GB (135.43 thousand rows/s., 53.48 MB/s.)
```

### 第二组

```
# 另一个表
20G     xx_httpcode_minf.ibd

CREATE TABLE xx_httpcode_minf
ENGINE = MergeTree
ORDER BY id AS
SELECT *
FROM mysql('host:port', 'db', 'tb', 'user', 'password') 

# 不知道为啥这表这么快就导入了 貌似是行少，但是表的总大小大啊
0 rows in set. Elapsed: 44.389 sec. Processed 13.03 million rows, 1.44 GB (293.44 thousand rows/s., 32.35 MB/s.)
```

PK 之 count(*)
-------------

### 第一组

```
# 1800w

# ClickHouse
SELECT count(*)
FROM article_clientuser_sum
┌──count()─┐
│ 18587381 │
└──────────┘

1 rows in set. Elapsed: 0.033 sec. Processed 18.59 million rows, 74.35 MB (556.76 million rows/s., 2.23 GB/s.)


# MySQL
mysql> select count(*) from article_clientuser_sum ;
+----------+
| count(*) |
+----------+
| 18587381 |
+----------+
1 row in set (39.48 sec)

# 性能 1196X
```

### 第二组

```
# 1300w

# ClickHouse
SELECT count(*)
FROM xx_httpcode_minf
┌──count()─┐
│ 13025469 │
└──────────┘
1 rows in set. Elapsed: 0.032 sec. Processed 13.03 million rows, 52.10 MB (406.68 million rows/s., 1.63 GB/s.)

# MySQL
mysql> SELECT count(*)
    -> FROM xx_httpcode_minf;
+----------+
| count(*) |
+----------+
| 13025469 |
+----------+
1 row in set (1 min 46.87 sec)

# 性能 3340X
```

PK 之复杂查询
--------

### 第一组

```
# ClickHouse

SELECT SUM(size) AS size
FROM xx_network_flow
WHERE (date >= '2018-01-01') AND (date <= '2018-01-31') AND (netstat = 0) AND (project LIKE '保密%')

Row 1:
──────
size: 4132888693

1 rows in set. Elapsed: 0.039 sec. Processed 841.66 thousand rows, 9.46 MB (21.67 million rows/s., 243.70 MB/s.)


# MySQL
+------------+
| size       |
+------------+
| 4132888693 |
+------------+
1 row in set (2.34 sec)

# 性能 60X
```

*   SQL 太长，截图示例
*   SQL 里的 xxx 均为脱敏数据

![](http://upload-images.jianshu.io/upload_images/1727239-e4438a1c9f8bc206.png)

```
# ClickHouse
┌─────size─┐
│ 76888224 │
└──────────┘

1 rows in set. Elapsed: 0.137 sec. Processed 841.66 thousand rows, 9.46 MB (6.13 million rows/s., 68.97 MB/s.)


# MySQL
+----------+
| size     |
+----------+
| 76888224 |
+----------+
1 row in set (2.86 sec)

# 性能 21X
```

### 第二组

```
# ClickHouse
SELECT
    project,
    idc,
    minf,
    http_code,
    sum(sumhit) AS num
FROM xx_httpcode_minf
WHERE (date = '2018-01-16') AND (httptype = 'download') AND \
(minf >= 0) AND (minf <= 288) AND \
(http_code IN ('200', '500', '404', '502', '503', '504'))
GROUP BY
    project,
    idc,
    minf,
    http_code
ORDER BY num DESC
LIMIT 3

┌─project─────────────────────────────────────┬─idc────┬─minf─┬─http_code─┬────num─┐
│ 域名1xxxx │ .1xx │  195 │ 200       │ 247522 │
│ 域名2xxxx │ .2xx │  185 │ 200       │ 246613 │
│ 域名3xxxx │ .3xx │  188 │ 200       │ 245808 │
└─────────────────────────────────────────────┴────────┴──────┴───────────┴────────┘

3 rows in set. Elapsed: 0.161 sec. Processed 13.03 million rows, 284.63 MB (80.94 million rows/s., 1.77 GB/s.)

# MySQL
+---------------------------------------------+--------+------+-----------+--------+
| project                                     | idc    | minf | http_code | num    |
+---------------------------------------------+--------+------+-----------+--------+
| 域名1xxxx| .1.xx |  195 | 200       | 247522 |
|  域名2xxxx | .2.xxx |  185 | 200       | 246613 |
|  域名3xxxx | .3xx |  188 | 200       | 245808 |
+---------------------------------------------+--------+------+-----------+--------+
3 rows in set (12.02 sec)

# 性能 75X
```

*   SQL 太长，截图示例
*   SQL 里的 xxx 均为脱敏数据
    
    ![](http://upload-images.jianshu.io/upload_images/1727239-d9ffefb8644cb1ec.png)

```
# ClickHouse

┌─project────────────────────────────┬─idc────┬─minf─┬─http_code─┬───num─┐
│  域名1 │ 1xxx│  154 │ 404       │ 10792 │
│  域名1 │ 2xxx │  155 │ 404       │ 10395 │
│ 域名1│ 3xxx │  272 │ 404       │ 10313 │
└────────────────────────────────────┴────────┴──────┴───────────┴───────┘

3 rows in set. Elapsed: 0.119 sec. Processed 13.03 million rows, 283.15 MB (109.10 million rows/s., 2.37 GB/s.)


# MySQL
+------------------------------------+--------+------+-----------+-------+
| project                            | idc    | minf | http_code | num   |
+------------------------------------+--------+------+-----------+-------+
|  域名1 | .1zz |  154 | 404       | 10792 |
|  域名1 | .3xx |  155 | 404       | 10395 |
|  域名1 | .3rr |  272 | 404       | 10313 |
+------------------------------------+--------+------+-----------+-------+
3 rows in set (2.19 sec)

# 性能 18X
```

压缩对比
----

<table><thead><tr><th>表名</th><th>MySQL 表容量</th><th>ClickHouse 表容量</th><th>压缩倍数</th></tr></thead><tbody><tr><td>article_clientuser_sum</td><td>5.5GB</td><td>1.2G</td><td>4.6</td></tr><tr><td>xx_httpcode_minf</td><td>20GB</td><td>243M</td><td>84</td></tr><tr><td>xx_network_flow</td><td>189MB</td><td>25M</td><td>7.56</td></tr></tbody></table>

*   **注**：xx_httpcode_minf 这个表的 MySQL 文件 20 个 G，应该是有大量的空洞造成的，这也就是 Facebook 的人开发 MyRocks 的原因：减少空洞，节省磁盘

风险
--

*   目前该功能还处于初级阶段，有不完善的地方，比如数据导入的方式比较粗暴，中间如果有异常，需要重新执行（使用的 ClickHouse 版本为：1.1.54342）
*   MySQL 的参数需要修改，如 max_allowed_packet
*   数据导入时需要注意带宽，实测可以达到 50MB/S
*   如果 MySQL 里的字段有 decimal 字符类型会怎么样？ClickHouse 没有双精度的类型
*   部分 SQL 需要改写
    *   如双引号改单引号

讨论
--

*   ClickHouse 为啥快？
    
    1.  MySQL 单条 SQL 是单线程的，只能跑满一个 core，ClickHouse 相反，有多少 CPU，吃多少资源，所以飞快
    2.  ClickHouse 不支持事务，不存在隔离级别。这里要额外说一下， 有人觉得，你一个数据库都不支持事务，不支持 ACID 还玩个毛。ClickHouse 的定位是分析性数据库，而不是严格的关系型数据库。又有人要问了，数据都不一致，统计个毛。举个例子，汽车的油表是 100% 准确么？为了获得一个 100% 准确的值，难道每次测量你都要停车检查么？统计数据的意义在于用大量的数据看规律，看趋势，而不是 100% 准确。
    3.  IO 方面，MySQL 是行存储，ClickHouse 是列存储，后者在 count() 这类操作天然有优势，同时，在 IO 方面，MySQL 需要大量随机 IO，ClickHouse 基本是顺序 IO。
    4.  有人可能觉得上面的数据导入的时候，数据肯定缓存在内存里了，这个的确，但是 ClickHouse 基本上是顺序 IO，用过就知道了，对 IO 基本没有太高要求，当然，磁盘越快，上层处理越快，但是 99% 的情况是，CPU 先跑满了（数据库里太少见了，大多数都是 IO 不够用）。
*   说到 MySQL 上跑的各种复杂查询，那是相当痛苦的回忆。从索引层面，很难对这些 SQL 进行优化，这也是我从 MySQL DBA 转做数据分析后要解决的第一个问题
    
*   专业的事情让专业的数据库来做，放开 MySQL 吧~
    
*   太™快了，还不赶紧来试试
    

Reference
---------

*   [MySQL のデータを ClickHouse で高速に集計する](https://links.jianshu.com/go?to=https%3A%2F%2Fqiita.com%2Fmikage%2Fitems%2Ffba19b3b8697fc6f73f0)
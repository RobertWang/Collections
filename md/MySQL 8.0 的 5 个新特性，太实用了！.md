> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU4NjQ1NDkyNQ==&mid=2247494824&idx=3&sn=10cdd8bac76ae0c36c82adaa2b4bfc9c&chksm=fdf9ae4aca8e275cd8465e842bfbdd372275bb9df4aaec82943ceaff8c1f506cc6476758e135&mpshare=1&scene=1&srcid=0601q3i9XtU2g7oXCpAuLHlw&sharer_sharetime=1622554606974&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

_作者：捏造的信仰  
原文：https://segmentfault.com/a/1190000013803247_

本文介绍几个 8.0 在关系数据库方面的主要新特性。

你可能已经知道 MySQL 从版本 5.7 开始提供了 NoSQL 存储的功能，在 8.0 中这部分功能也得到了一些改进，但鉴于这个在实际当中用得极少，本人也是没用过，所以本文不会介绍这方面的东西，而是关注其关系数据库方面。

1、隐藏索引
------

隐藏索引的特性对于性能调试非常有用。在 8.0 中，索引可以被 “隐藏” 和“显示”。当一个索引隐藏时，它不会被查询优化器所使用。

也就是说，我们可以隐藏一个索引，然后观察对数据库的影响。如果数据库性能有所下降，就说明这个索引是有用的，于是将其 “恢复显示” 即可；如果数据库性能看不出变化，说明这个索引是多余的，可以删掉了。

隐藏一个索引的语法是：

```
ALTER TABLE t ALTER INDEX i INVISIBLE;


```

恢复显示该索引的语法是：

```
ALTER TABLE t ALTER INDEX i VISIBLE;


```

当一个索引被隐藏时，我们可以从 `show index` 命令的输出中看到，该索引的 `Visible` 属性值为 `NO`。

> 注意：当索引被隐藏时，它的内容仍然是和正常索引一样实时更新的，这个特性本身是专门为优化调试使用。如果你长期隐藏一个索引，那还不如干脆删掉，因为毕竟索引的存在会影响插入、更新和删除的性能。

2、设置持久化
-------

MySQL 的设置可以在运行时通过 `SET GLOBAL` 命令来更改，但是这种更改只会临时生效，到下次启动时数据库又会从配置文件中读取。

MySQL 8 新增了 `SET PERSIST` 命令，例如：

```
SET PERSIST max_connections = 500;


```

MySQL 会将该命令的配置保存到数据目录下的 `mysqld-auto.cnf` 文件中，下次启动时会读取该文件，用其中的配置来覆盖缺省的配置文件。

3、UTF-8 编码
----------

从 MySQL 8 开始，数据库的缺省编码将改为 `utf8mb4`，这个编码包含了所有 emoji 字符。

多少年来我们使用 MySQL 都要在编码方面小心翼翼，生怕忘了将缺省的 latin 改掉而出现乱码问题。从此以后就不用担心了。

4、通用表表达式（Common Table Expressions）
----------------------------------

复杂的查询会使用嵌入式表，例如：

```
SELECT t1.*, t2.* FROM 
  (SELECT col1 FROM table1) t1,
  (SELECT col2 FROM table2) t2;


```

而有了 CTE，我们可以这样写：

```
WITH
  t1 AS (SELECT col1 FROM table1),
  t2 AS (SELECT col2 FROM table2)
SELECT t1.*, t2.* 
FROM t1, t2;


```

这样看上去层次和区域都更加分明，改起来也更清晰的知道要改哪一部分。

关于 CTE 的更详细介绍请看官方文档，关注公众号 Java 技术栈可以阅读更多 MySQL 系列教程及面试题。

5、窗口函数（Window Functions）
------------------------

MySQL 被吐槽最多的特性之一就是缺少 rank() 函数，当需要在查询当中实现排名时，必须手写 @ 变量。但是从 8.0 开始，MySQL 新增了一个叫窗口函数的概念，它可以用来实现若干新的查询方式。

窗口函数有点像是 `SUM()`、`COUNT()` 那样的集合函数，但它并不会将多行查询结果合并为一行，而是将结果放回多行当中。也就是说，窗口函数是不需要 `GROUP BY` 的。

假设我们有一张 “班级学生人数” 表：

```
mysql> select * from classes;
+--------+-----------+
| name   | stu_count |
+--------+-----------+
| class1 |        41 |
| class2 |        43 |
| class3 |        57 |
| class4 |        57 |
| class5 |        37 |
+--------+-----------+
5 rows in set (0.00 sec)


```

如果我要对班级人数从小到大进行排名，可以这样利用窗口函数：

```
mysql> select *, rank() over w as `rank` from classes
    -> window w as (order by stu_count);
+--------+-----------+------+
| name   | stu_count | rank |
+--------+-----------+------+
| class5 |        37 |    1 |
| class1 |        41 |    2 |
| class2 |        43 |    3 |
| class3 |        57 |    4 |
| class4 |        57 |    4 |
+--------+-----------+------+
5 rows in set (0.00 sec)


```

在这里我们创建了名为 w 的 window，规定它对 stu_count 字段进行排序，然后在 select 子句中对 w 执行 rank() 方法，将结果输出为 rank 字段。

其实，window 的创建是可选的。例如我要在每一行中加入学生总数，则可以这样：

```
mysql> select *, sum(stu_count) over() as total_count
    -> from classes;
+--------+-----------+-------------+
| name   | stu_count | total_count |
+--------+-----------+-------------+
| class1 |        41 |         235 |
| class2 |        43 |         235 |
| class3 |        57 |         235 |
| class4 |        57 |         235 |
| class5 |        37 |         235 |
+--------+-----------+-------------+
5 rows in set (0.00 sec)


```

这样做有什么用呢？

这样我们就可以一次性将每个班级的学生人数占比查出来了：

```
mysql> select *,
    -> (stu_count)/(sum(stu_count) over()) as rate
    -> from classes;
+--------+-----------+--------+
| name   | stu_count | rate   |
+--------+-----------+--------+
| class1 |        41 | 0.1745 |
| class2 |        43 | 0.1830 |
| class3 |        57 | 0.2426 |
| class4 |        57 | 0.2426 |
| class5 |        37 | 0.1574 |
+--------+-----------+--------+
5 rows in set (0.00 sec)


```

看来 MySQL 8.0 改进还挺人性化的，你用的哪个版本呢？
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/kendoziyu/p/14777470.html)

1. 查看数据库
========

```
show databases;
```

上面这条命令的作用是查看所有的数据库。效果等同于下面这条命令:

```
use information_schema;
select schema_name from schemata;
```

![](https://img2020.cnblogs.com/blog/1730512/202105/1730512-20210517151138626-906161907.jpg)

以纵向报表的形式输出结果，有利于阅读。

> 图中的四个数据库是 MySQL 安装成功以后自带的。

2. 查看 MySQL 服务器状态
=================

通常使用以下这条命令，来查看当前 MySQL 服务器的运行状态：

```
show status;
```

**加上 LIKE 关键字可以模糊筛选出你需要的属性值。**

★ 例如，查看 MySQL 服务器的正常运行时间：

```
show status like '%uptime%'
```

![](https://img2020.cnblogs.com/blog/1730512/202105/1730512-20210517152301027-2040946907.jpg)

> 如上图所示，表示自 MySQL 服务器启动以来，已正常运行 56779735 秒，共计 657 多天了。

★ 再例如，查看 MySQL 慢 SQL 的数量：

```
show status like '%slow%'
```

![](https://img2020.cnblogs.com/blog/1730512/202105/1730512-20210517153004285-1023544791.jpg)

> 如上图所示，Sql_queries 表示慢 SQL 查询的数量。即使没有开启慢 SQL 日志功能，该属性值也会照常计数。

★ 再比如，查看 MySQL 的表锁 / 行锁信息：

```
show status like '%lock%';
```

![](https://img2020.cnblogs.com/blog/1730512/202105/1730512-20210517162921092-43962729.jpg)

![](https://img2020.cnblogs.com/blog/1730512/202105/1730512-20210517162930047-1370590161.jpg)

> 如图所示，前缀为 Innodb_row_lock 的表示行锁，前缀为 Table_locks_ 表示表锁。

更多状态值，请移步 MySQL 5.7 官方文档之服务器状态变量 [跳转 click here](https://dev.mysql.com/doc/refman/5.7/en/server-status-variables.html)，进行查看。

3. 选择当前数据库
==========

```
use 数据库名称;
```

使用这条语句之后，相当于声明了接下来的 SQL 语句的默认缺省数据库。就不需要每条语句都带上表所在的数据库名称了。  
例如 `use information_schema`，当使用此命令后

```
select schema_name from information_schema.schemata;
```

可以简写为

```
select schema_name from schemata;
```

4. 查看数据库中的表
===========

```
show tables from 数据库名称;
```

例如，查看数据库 `information_schema` 中所有的表：

```
show tables from information_schema;
```

也可以写作

```
use information_schema;
show tables;
```

![](https://img2020.cnblogs.com/blog/1730512/202105/1730512-20210517154313896-1927131604.jpg)

> 如上图所示，这仅仅截取展示了一部分的表。

5. 查看表结构定义
==========

```
desc 表名称;
```

例如：

```
use information_schema;
desc engines;
```

![](https://img2020.cnblogs.com/blog/1730512/202105/1730512-20210517154955197-1288296170.jpg)

> 如上图所示，展示了数据库 `information_schema` 中的表 `engines` 的字段定义。
> 
> 具体包含的信息有：字段名称 Field，字段类型 Type，字段是否可以为空 Null，索引类型 Key，默认值 Default 等等...

6. 查看表状态
========

```
show table status from 数据库名称;
```

这条命令，查看的是数据库中所有表的状态。

例如，显示数据库 `information_schema` 中所有表的状态：

```
show table status from information_schema;
```

也可以写作

```
use information_schema;
show table status;
```

![](https://img2020.cnblogs.com/blog/1730512/202105/1730512-20210517160152263-1732956065.jpg)

> 如上图所示，包含的信息包括，表名称 Name，表引擎 Engine，行记录格式 Row_format，等等...

★ 如果，需要从所有的表状态中筛选出目标表状态，可以使用 `like` 关键字：

```
show table status from information_schema like 'engines';
```

★ 如果，需要模糊查询的话，可以加上通配符 `%` :

```
show table status from information_schema like '%innodb%';
```

7. 查看 MySQL 服务器系统变量
===================

```
show variables;
```

★ 例如，查看日志是否启动：

```
show variables like 'log%';
```

更多系统变量，请移步 MySQL 5.7 官方文档之服务器系统变量 [跳转 click here](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html)

参考文档
====

mysql 查看数据库、表的基本命令 [跳转 click here](https://www.cnblogs.com/niewd/p/11481030.html)
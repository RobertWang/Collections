> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jb51.net](https://www.jb51.net/article/165908.htm)

> 这篇文章主要介绍了 mysql 迁移到 clickhouse 的 5 种方法，本文给大家介绍的非常详细，具有一定的参考借鉴价值, 需要的朋友可以参考下

数据迁移需要从 mysql 导入 clickhouse, 总结方案如下，包括 clickhouse 自身支持的三种方式，第三方工具两种。

```
create table engin mysql

CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]

(

 name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1] [TTL expr1],

 name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2] [TTL expr2],

 ...

 INDEX index_name1 expr1 TYPE type1(...) GRANULARITY value1,

 INDEX index_name2 expr2 TYPE type2(...) GRANULARITY value2

) ENGINE = MySQL('host:port', 'database', 'table', 'user', 'password'[, replace_query, 'on_duplicate_clause']);
```

官方文档: [https://clickhouse.yandex/docs/en/operations/table_engines/mysql/](https://clickhouse.yandex/docs/en/operations/table_engines/mysql/)

**注意，实际数据存储在远端 mysql 数据库中，可以理解成外表。**

可以通过在 mysql 增删数据进行验证。

```
insert into select from

CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]

(

 name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],

 name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],

 ...

) ENGINE = engine

INSERT INTO [db.]table [(c1, c2, c3)] select 列或者* from mysql('host:port', 'db', 'table_name', 'user', 'password')
```

可以自定义列类型，列数，使用 clickhouse 函数对数据进行处理，比如

```
select toDate(xx) from mysql("host:port","db","table_name","user_name","password")

create table as select from

CREATE TABLE [IF NOT EXISTS] [db.]table_name

ENGINE =Log

AS

SELECT *

FROM mysql('host:port', 'db', 'article_clientuser_sum', 'user', 'password')
```

网友文章: [http://jackpgao.github.io/2018/02/04/ClickHouse-Use-MySQL-Data/](http://jackpgao.github.io/2018/02/04/ClickHouse-Use-MySQL-Data/)

不支持自定义列，参考资料里的博主写的 `ENGIN=MergeTree` 测试失败。

可以理解成 `create table` 和 `insert into select` 的组合

`Altinity/clickhouse-mysql-data-reader`

Altinity 公司开源的一个 python 工具，用来从 mysql 迁移数据到 clickhouse(支持 binlog 增量更新和全量导入)，但是官方 readme 和代码脱节，根据 quick start 跑不通。

```
## 创建表

clickhouse-mysql \

## 修改脚本

vim create_clickhouse_table_template.sql

## 导入建表

clickhouse-client -mn < create_clickhouse_table_template.sql

## 数据导入

clickhouse-mysql \
```

官方文档: [https://github.com/Altinity/clickhouse-mysql-data-reader#mysql-migration-case-1—migrate-existing-data](https://github.com/Altinity/clickhouse-mysql-data-reader#mysql-migration-case-1%E2%80%94migrate-existing-data)

注意，上述三种都是从 mysql 导入 clickhouse，如果数据量大，对于 mysql 压力还是挺大的。下面介绍两种离线方式 (streamsets 支持实时，也支持离线)

csv

```
## 忽略建表

clickhouse-client \

 -h host \
```

但是如果源数据质量不高，往往会有问题，比如包含特殊字符 (分隔符，转义符)，或者换行。被坑的很惨。

```
自定义分隔符,

遇到错误跳过而不中止，

csv 跳过空值(null) ，报 Code: 27. DB::Exception: Cannot parse input: expected , before: xxxx: (at row 69) ERROR: garbage after Nullable(Date): "8,0020205" sed ' :a;s/,,/,\\N,/g;ta' |clickhouse-client -h host

python clean_csv.py
```

clean_csv.py 参考我另外一篇 032-[csv 文件容错处理](https://www.jb51.net/article/165911.htm)

**streamsets**

streamsets 支持从 mysql 或者读 csv 全量导入，也支持订阅 binlog 增量插入，参考我另外一篇 025- [大数据 ETL 工具之 StreamSets 安装及订阅 mysql binlog](https://anjia0532.github.io/2019/06/10/cdh-streamsets/) 。

本文只展示从 mysql 全量导入 clickhouse

本文假设你已经搭建起 streamsets 服务

![](https://img.jbzj.com/file_images/article/201907/201907220940011.png)

**启用并重启服务**

![](https://img.jbzj.com/file_images/article/201907/201907220940022.png)

上传 mysql 和 clickhouse 的 jdbc jar 和依赖包

便捷方式，创建 pom.xml，使用 maven 统一下载

如果本地装有 maven，执行如下命令

```
mvn dependency:copy-dependencies -DoutputDirectory=lib -DincludeScope=compile
```

所有需要的 jar 会下载并复制到 lib 目录下

![](https://img.jbzj.com/file_images/article/201907/201907220940023.png)

然后拷贝到 `streamsets /opt/streamsets-datacollector-3.9.1/streamsets-libs-extras/streamsets-datacollector-jdbc-lib/lib/` 目录下

![](https://img.jbzj.com/file_images/article/201907/201907220940024.png)

重启 streamsets 服务

![](https://img.jbzj.com/file_images/article/201907/201907220940025.png) ![](https://img.jbzj.com/file_images/article/201907/201907220940036.png) ![](https://img.jbzj.com/file_images/article/201907/201907220940037.png) ![](https://img.jbzj.com/file_images/article/201907/201907220940038.png)![](https://img.jbzj.com/file_images/article/201907/201907220940039.png) ![](https://img.jbzj.com/file_images/article/201907/2019072209400410.png)  

**总结**

以上所述是小编给大家介绍的 mysql 迁移到 clickhouse 的 5 种方法, 希望对大家有所帮助，如果大家有任何疑问请给我留言，小编会及时回复大家的。在此也非常感谢大家对脚本之家网站的支持！  
如果你觉得本文对你有帮助，欢迎转载，烦请注明出处，谢谢！
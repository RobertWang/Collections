> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/zqr99/p/9939418.html)

centos 安装 clickhouse
====================

curl -s https://packagecloud.io/install/repositories/altinity/clickhouse/script.rpm.sh | sudo bash

sudo yum list 'clickhouse*'

sudo yum -y install clickhouse*

docker 安装可以直接克隆 
================

https://gitee.com/pyzy/cloudcompute

clickhouse 数据类型
===============

数据类型没有 boolean 其他基本和 hive 一样, 详细的看官网 

[clickhouse 数据类型](https://clickhouse.yandex/docs/en/data_types/int_uint/ "clickhouse数据类型")

clickhouse 引擎
=============

clickhouse 有很多引擎, 最常用的是 [MergeTree 家族](https://clickhouse.yandex/docs/en/operations/table_engines/mergetree/ "mergeTree家族") 还有 [Distributed 引擎](https://clickhouse.yandex/docs/en/operations/table_engines/distributed/ "distributed")

clickhouse 创建表
==============

clickhouse 可以创建本地表, 分布式表, 集群表

create table test() 为本地表

```
CREATE TABLE image_label_all AS image_label ENGINE = Distributed(distable, monchickey, image_label, rand()) 分布式表



```

create table test on cluster() 为集群表

贴一个完整的建表语句, 使用 ReplicatedMergeTree 引擎

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
 CREATE TABLE m.n_mdw_pcg (
 storekey Int32, 
 custkey Int32,  
cardholderkey Int32,  
pcg_main_cat_id Int32,  
pcg_main_cat_desc String,  
count Int32,  
quartly String
) ENGINE = ReplicatedMergeTree('/clickhouse/tables/m/n_mdw_pcg', '{replica}') PARTITION BY (quartly, pcg_main_cat_id) 
ORDER BY (storekey, custkey, cardholderkey)

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

![](https://img2018.cnblogs.com/blog/1186254/201811/1186254-20181110141716709-1825033874.png)

保证数据复制

clickhouse 数据操作
===============

增加可以使用 insert;

不能修改, 也不能指定删除;

可以删除分区, 会删除对应的数据 我使用 --help 看了一下有 truncate table, 但是没有具体使用过, 如果要全部删除数据可以删除表, 然后在建表查数据

可以使用脚本操作

```
database=$1
table=$2
echo "Truncate table "$2
create=`clickhouse-client --database=$1 --query="SHOW CREATE TABLE $table" | tr -d '\\'`
clickhouse-client --database=$1 --query="DROP TABLE $table"
clickhouse-client --database=$1 --query="$create"

```

再导入数据就可以了

导入数据, clickhouse 支持很多文件类型 详细的看官方文档, [文件导入导出](https://clickhouse.yandex/docs/en/interfaces/formats/ "输入和输出数据的格式")

贴两个经常用的文件的导入

tsv, 以 "\t" 隔开

```
 clickhouse-client -h badd52c42f08 --input_format_allow_errors_num=1 --input_format_allow_errors_ratio=0.1 --query="INSERT INTO tablename FORMAT TSV"<file

```

csv 以 "," 或者 "|" 隔开

```
clickhouse-client -h adc3eaba589c --format_csv_delimiter="|" --query='INSERT INTO tablename FORMAT CSV' < file

```

数据查询

clickhouse 的 查询 sql 表单查询基本和标准 sql 一样, 也支持 limit 分页, 但是 inner join 的查询写法不一样, 而且我用 4 亿 + 2000 万 inner join 的速度很慢

两个 sql 对比 inner join 要花费将近一分钟, 使用 in 子查询仅 3 秒, 建议都使用 in 查询, clickhouse 的单表查询速度很快, 3 亿数据 count distinct 仅 1 秒左右

其它的技巧和知识, 本人占时未做了解, 希望大家能一起学习, 一起进步
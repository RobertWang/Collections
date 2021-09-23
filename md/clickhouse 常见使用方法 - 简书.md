> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/a6888c1d0a55)

clickhouse 在用户画像中的使用
--------------------

> 将用户标签放在 clickhouse 中，并且用 bitmap 形式，可以减少用户空间，同时能够加快用户查询标签的效率，现在很多企业采用 clickhouse + bitmap 解决用户画像的问题

```
  //1:建立bitmap的分布式表 
    CREATE TABLE test.bitmap_test
   (
       `la_name` String,
       `la_value` String,
       `uid_bitmap` AggregateFunction(groupBitmap, UInt32),
       `dt` Date
   )
   ENGINE = Distributed('test', 'test', 'test', rand())
   //2:bitmap 查询符合条件的数据集合bitmap
   SELECT groupBitmapMergeState(uid_bitmap) FROM test.bitmap_test WHERE dt='2021-06-23'；
   //3: 将得到的bitmap 中uid的集合，找出对应的uid;
  with(SELECT groupBitmapMergeState(uid_bitmap) FROM test.bitmap_test WHERE dt='2021-06-23') as temp SELECT arrayJoin(bitmapToArray(temp)


```

clickhouse 在地理位置服务中的使用
----------------------

> 目前很多企业需要提供基于地理位置的服务，同时能够计算出几公里内的一些  
> 服务信息；  
> 查询 tb_distance 中距离坐标位置为（x,y）小于 5000 米的所有记录

```
select * from tb_distance where   greatCircleDistance(x,y,lnt,lat)<5000 


```

clickhouse 常见的 sql 优化
---------------------

sql 慢查大部分主要体现在 cpu 负载过高，io 过高，或者查询的列中无索引导致的；注意；clickhouse 本身不太支持高并发的场景，qps 过高会导致 clickhouse 服务器 cpu 过高，导致慢查

在这些情况下; 常见的考虑的是 sql 中是否有复杂的运算，查询的数量量是否过大，查询的列中索引是否有效；

### sql 查询特点：数量大，且分区跨度大

data 表格中有 8 亿多条数据，data 表按照 p_data_day 分区；

```
select sn,COUNT(1) as valueQt from data WHERE   sn='70A0600018109' and p_day >= '2017-01-01' and p_data_day < '2020-08-13'
group by sn;


```

数据会遍历整个分区，数据平均在 1s 左右分钟返回 ;

优化思路：减少不必要数据的遍历（分区）；充分利用 clickhouse 索引（group by 索引）

*   针对 sn 的查询，建立物化视图；将 8 亿条数据按照 sn 号以及 device_id（mac_code）建立 256 个分区；

```
create MATERIALIZED VIEW IF NOT EXISTS data_sn_materialized
engine = ReplicatedMergeTree('/clickhouse/tables/{ck_cluster}/data_sn_materialized', '{replica}')
PARTITION BY sn_sort_key ORDER BY (sn_sort_key,sn,p_day)
AS select halfMD5(_sn) % 256 as sn_sort_key,sn,p_day,count() as cnt
 from data group by sn_sort_key,sn,p_day;


```

优化后

```
查询语句；保持原来的出参和入参不变，数据能够在200ms以内返回，


```

### sql 查询特点：数量大，且分区跨度大

data 表格数据量在 10 亿多条，建表语句如下

```
CREATE TABLE data (
`data_day` Date,
 `flow_type` UInt32 DEFAULT CAST(0,
 'UInt32'),
.....
) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{ck_cluster}/data', '{replica}') PARTITION BY data_day ORDER BY (flow_type, data_day) SETTINGS index_granularity = 8192;


```

查询语句

```
select ... from data where data_day = '2020-09-11'


```

我们观察到查询数据的时候，总是会具体到昨天；而且历史的数据不会再使用；

优化思路： 使用 clickhouse 的 TTL，减少表容量，

```
CREATE TABLE dwrt.lc_order_flow (
`data_day` Date,
 .....
 `flow_type` UInt32 DEFAULT CAST(0,
 'UInt32'),
....
) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{ck_cluster}/data', '{replica}') PARTITION BY data_day ORDER BY (data_day, flow_type) TTL data_day + toIntervalDay(7) SETTINGS index_granularity = 8192;



```

参考链接：  
[clickhouse geo 函数](https://links.jianshu.com/go?to=https%3A%2F%2Fclickhouse.tech%2Fdocs%2Fzh%2Fsql-reference%2Ffunctions%2Fgeo%2F)  
[clickhouse 位图函数](https://links.jianshu.com/go?to=https%3A%2F%2Fclickhouse.tech%2Fdocs%2Fzh%2Fsql-reference%2Ffunctions%2Fbitmap-functions%2F)
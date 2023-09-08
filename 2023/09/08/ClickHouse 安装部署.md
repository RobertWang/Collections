> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Qvhbt-fpoxkDDl9x1xne8w)

> 一、ClickHouse 简介 ClickHouse 是俄罗斯的 Yandex 于 2016 年开源的用

**一、ClickHouse 简介**

    ClickHouse 是俄罗斯的 Yandex 于 2016 年开源的用于在线分析处理查询（OLAP :Online Analytical Processing）MPP 架构的列式存储数据库（DBMS：Database Management System），能够使用 SQL 查询实时生成分析数据报告。ClickHouse 的全称是 Click Stream，Data WareHouse。

clickhouse 可以做用户行为分析、流批一体、线性扩展和可靠性保障能够原生支持 shard + replication，clickhouse 没有走 hadoop 生态，采用 Local attached storage 作为存储

**二、ClickHouse 特点**

1、列式存储：

行式存储的好处：

想查找某个人所有的属性时，可以通过一次磁盘查找加顺序读取就可以；但是当想查所有人的年龄时，需要不停的查找，或者全表扫描才行，遍历的很多数据都是不需要的。

列式存储的好处

对于列的聚合、计数、求和等统计操作优于行式存储

由于某一列的数据类型都是相同的，针对于数据存储更容易进行数据压缩，每一列选择更优的数据压缩算法，大大提高了数据的压缩比重

数据压缩比更好，一方面节省了磁盘空间，另一方面对于 cache 也有了更大的发挥空间

列式存储不支持事务

2、DBMS 功能：

几乎覆盖了标准 SQL 的大部分语法，包括 DDL 和 DML、，以及配套的各种函数；用户管理及权限管理、数据的备份与恢复

3、多样化引擎：

目前包括合并树、日志、接口和其他四大类 20 多种引擎。

4、高吞吐写入能力：

ClickHouse 采用类 LSM Tree 的结构，数据写入后定期在后台 Compaction。通过类 LSM tree 的结构， ClickHouse 在数据导入时全部是顺序 append 写，写入后数据段不可更改，在后台 compaction 时也是多个段 merge sort 后顺序写回磁盘。顺序写的特性，充分利用了磁盘的吞吐能力。

5、数据分区与线程及并行：

ClickHouse 将数据划分为多个 partition，每个 partition 再进一步划分为多个 index granularity(索引粒度)，然后通过多个 CPU 核心分别处理其中的一部分来实现并行数据处理。在这种设计下， 单条 Query 就能利用整机所有 CPU。极致的并行处理能力，极大的降低了查询延时。

所以， ClickHouse 即使对于大量数据的查询也能够化整为零平行处理。但是有一个弊端就是对于单条查询使用多 cpu，就不利于同时并发多条查询。所以对于高 qps 的查询业务并不是强项。

6、ClickHouse 像很多 OLAP 数据库一样，单表查询速度优于关联查询，而且 ClickHouse 的两者差距更为明显。

关联查询：clickhouse 会将右表加载到内存。

**三、安装 ClickHouse  
**

```
下载安装包
https://repo.clickhouse.com/tgz/
安装包分别为：
clickhouse-common-static-21.9.6.24.tgz
clickhouse-common-static-dbg-21.9.6.24.tgz
clickhouse-server-21.9.6.24.tgz
clickhouse-client-21.9.6.24.tgz
依次将这四个安装包解压安装
tar zxf clickhouse-common-static-21.9.6.24.tgz
./clickhouse-common-static-21.9.6.24/install/doinst.sh
tar zxf clickhouse-common-static-dbg-21.9.6.24.tgz 
./clickhouse-common-static-dbg-21.9.6.24/install/doinst.sh
tar zxf clickhouse-server-21.9.6.24.tgz 
./clickhouse-server-21.9.6.24/install/doinst.sh
注：执行时会让输入密码和是否允许其他主机登录(如下图)
tar zxf clickhouse-client-21.9.6.24.tgz 
./clickhouse-client-21.9.6.24/install/doinst.sh

```

![](https://mmbiz.qpic.cn/mmbiz_png/NHZWiaTSnCOzVDdmXMBKeOYNjH2rKrrULaxHziczjNu5lI8cTuaIAmD0gtBcSTQv3iaUVEDreSRUohMia1YGq2sTEw/640?wx_fmt=png)

启动  

```
clickhouse start
  chown --recursive clickhouse '/var/run/clickhouse-server/'
  Will run su -s /bin/sh 'clickhouse' -c '/usr/bin/clickhouse-server --config-file /etc/clickhouse-server/config.xml --pid-file /var/run/clickhouse-server/clickhouse-server.pid --daemon'
  Waiting for server to start
  Waiting for server to start
  Server started

```

![](https://mmbiz.qpic.cn/mmbiz_png/NHZWiaTSnCOzVDdmXMBKeOYNjH2rKrrULkv4ep5aoF43DFghD592OyXTXb14VCLjsUcSNLVVtLoaEoIKJMJYhgg/640?wx_fmt=png)

连接  

```
clickhouse-client --host 127.0.0.1 --user default --password 123456
select * from system.databases;

```

![](https://mmbiz.qpic.cn/mmbiz_png/NHZWiaTSnCOzVDdmXMBKeOYNjH2rKrrULpw2ic8HuzdS463mt1ib0BJfP1S0ggrEojebR2iaZTZH8MVkK9IzYWEhDA/640?wx_fmt=png)

这里我们发现它默认存放位置是 / var/lib/clickhouse/

**四、修改默认存放位置和日志位置**

```
#创建存放位置目录
mkdir -p /data/clickhouse
#停止服务
clickhouse stop
#修改配置文件
cd /etc/clickhouse-server
sed -i 's#/var/lib/clickhouse#/data/clickhouse#g' config.xml
cat config.xml | grep /data/clickhouse
#将数据文件移动到/data/clickhouse
mv /var/lib/clickhouse/* /data/clickhouse/
#查看metadata目录下原有全部库的软链接
#说明：这些是原来库自动创建的软链接，以default为例子：default实际指向还是/var/lib/clickhouse/.../.../，需要重置为新位置的目录。
#由于步骤3导致/var/lib/clickhouse/.../.../的目录不存在，ll查看的时候有问题的软链接会呈现闪烁状态。
unlink system
unlink default
ln -s /data/clickhouse/store/ed8/ed824233-308b-4a10-ad82-4233308bea10/ default
ln -s /data/clickhouse/store/3c5/3c578aad-3fce-4a28-bc57-8aad3fce4a28/ system
chown -h clickhouse:clickhouse system  default
  -h: chown一定要加上-h参数，不然更改的权限目标实际上是软链接对应的真正的位置
cd /data/clickhouse/data/system
然后把这些的软链接出现链一下
unlink asynchronous_metric_log
ln -s /data/clickhouse/store/46e/46ebd7e2-8923-4a24-86eb-d7e289239a24/ asynchronous_metric_log
unlink metric_log 
ln -s /data/clickhouse/store/a74/a74fd533-c637-48a6-a74f-d533c63768a6/ metric_log
unlink query_log 
ln -s /data/clickhouse/store/de5/de5519d3-2a6f-48e6-9e55-19d32a6fd8e6/ query_log
unlink query_thread_log 
ln -s /data/clickhouse/store/4d6/4d6cf09c-4557-4959-8d6c-f09c45579959/ query_thread_log
unlink trace_log 
ln -s /data/clickhouse/store/4a5/4a5cb3a7-e718-4bfc-8a5c-b3a7e7180bfc/ trace_log
chown -h clickhouse:clickhouse asynchronous_metric_log metric_log query_log query_thread_log trace_log
  -h: chown一定要加上-h参数，不然更改的权限目标实际上是软链接对应的真正的位置
启动数据库
clickhouse start

```

验证：  

```
clickhouse-client --host 127.0.0.1 --user default --password 123456
select * from system.databases;

```

![](https://mmbiz.qpic.cn/mmbiz_png/NHZWiaTSnCOzVDdmXMBKeOYNjH2rKrrULibfoOJnqocEkCY5hLFoicGMPGlcCRPibjsibqzjYkpZehKAuxL8qALMk4A/640?wx_fmt=png)

修改日志存放位置

```
#修改日志文件位置
mkdir -p /data/clickhouse/log
chown -h clickhouse:clickhouse /data/clickhouse/log
#修改配置文件路径
sed -i 's#/var/log/clickhouse-server#/data/clickhouse/log#g' config.xml
#重启服务，查看日志

```

![](https://mmbiz.qpic.cn/mmbiz_png/NHZWiaTSnCOzVDdmXMBKeOYNjH2rKrrULFde2SMdXu5G5icp3ic9f6IMhialwUDR549k5EbrLCXdcqwjQ5zleakpVw/640?wx_fmt=png)

五、允许远程登录

```
vim /etc/clickhouse-server/config.xml
<listen_host>::</listen_host>
重启服务
clickhouse stop
clickhouse start
测试：
浏览器：http://192.168.1.30:8123/

```

![](https://mmbiz.qpic.cn/mmbiz_png/NHZWiaTSnCOzVDdmXMBKeOYNjH2rKrrULkh0jyyzgkbfay7SOdfHFkIticOpggFQoxuUB7WbyV7DniaBgKzvA8kmw/640?wx_fmt=png)

使用 DBeaver 连接

![](https://mmbiz.qpic.cn/mmbiz_png/NHZWiaTSnCOzVDdmXMBKeOYNjH2rKrrULWQo3W0nYpzrALk5ibN60aHsBDicRZ9GY63xVFDZfBBPGsZkUaBwIrjrQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/NHZWiaTSnCOzVDdmXMBKeOYNjH2rKrrULgrkEvjTfVsbECmX0Jk0ZzJiclPE023tgLyS8qTGia8KnHtib2UN3hgk8g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/NHZWiaTSnCOzVDdmXMBKeOYNjH2rKrrULMsia2FwpSJhOslLf3KgiaBHWaKPibTyVkBNGMSXPkVwAyE7hzUeFnGAVg/640?wx_fmt=png)
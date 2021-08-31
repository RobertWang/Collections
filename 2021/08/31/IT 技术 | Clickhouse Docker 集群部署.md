> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1630396366&ver=3285&signature=rtZn9aV-tHtXKCWaqWQyYpo1wKr0RljCMrRhqG5ykLne-cKyys9O-lOP7DonXWpKZquhNpnDDWl7AjUVXdL8GcC2FBKvquBsrpkziYwq9HuX2scKZvJwSxz19-Kd61gD&new=1)

  

****写在前面  
****

  

抽空来更新一下大数据的玩意儿了，起初架构选型的时候有考虑 Hadoop 那一套做数仓，但是 Hadoop 要求的服务器数量有点高，集群至少 6 台或以上，所以选择了 Clickhouse（后面简称 CH）。CH 做集群的话，3 台服务器起步就可以的，当然，不是硬性，取决于你的 zookeeper 做不做集群，其次 CH 性能更强大，对于量不是非常巨大的场景来说，单机已经足够应对 OLAP 多种场景了。

进入正题，相关环境：

<table height="329"><tbody><tr><td width="70" data-style="padding: 8px 14px; border-color: rgb(192, 192, 192); word-break: break-all; max-width: 100%; box-sizing: border-box; transition: all 0.5s ease 0s; border-collapse: collapse; min-width: 50px; overflow-wrap: break-word !important;">IP</td><td width="98" data-style="padding: 8px 14px; border-color: rgb(192, 192, 192); word-break: break-all; max-width: 100%; box-sizing: border-box; transition: all 0.5s ease 0s; border-collapse: collapse; min-width: 50px; overflow-wrap: break-word !important;">服务器名</td><td width="71" data-style="padding: 8px 14px; border-color: rgb(192, 192, 192); word-break: break-all; max-width: 100%; box-sizing: border-box; transition: all 0.5s ease 0s; border-collapse: collapse; min-width: 50px; overflow-wrap: break-word !important;">操作系统</td><td width="106" data-style="padding: 8px 14px; border-color: rgb(192, 192, 192); word-break: break-all; max-width: 100%; box-sizing: border-box; transition: all 0.5s ease 0s; border-collapse: collapse; min-width: 50px; overflow-wrap: break-word !important;">服务</td><td width="235" data-style="padding: 8px 14px; border-color: rgb(192, 192, 192); word-break: break-all; max-width: 100%; box-sizing: border-box; transition: all 0.5s ease 0s; border-collapse: collapse; min-width: 50px; overflow-wrap: break-word !important;">备注</td></tr><tr><td width="48" data-style="padding: 8px 14px; border-color: rgb(192, 192, 192); word-break: break-all; max-width: 100%; box-sizing: border-box; transition: all 0.5s ease 0s; border-collapse: collapse; min-width: 50px; overflow-wrap: break-word !important;">172.192.13.10</td><td width="77" data-style="padding: 8px 14px; border-color: rgb(192, 192, 192); word-break: break-all; max-width: 100%; box-sizing: border-box; transition: all 0.5s ease 0s; border-collapse: collapse; min-width: 50px; overflow-wrap: break-word !important;">server01</td><td width="50" data-style="padding: 8px 14px; border-color: rgb(192, 192, 192); word-break: break-all; max-width: 100%; box-sizing: border-box; transition: all 0.5s ease 0s; border-collapse: collapse; min-width: 50px; overflow-wrap: break-word !important;">Ubuntu20.04</td><td width="100" data-style="padding: 8px 14px; border-color: rgb(192, 192, 192); word-break: break-all; max-width: 100%; box-sizing: border-box; transition: all 0.5s ease 0s; border-collapse: collapse; min-width: 50px; overflow-wrap: break-word !important;">两个 Clickhouse 实例、Zookeeper</td><td width="214" data-style="padding: 8px 14px; border-color: rgb(192, 192, 192); word-break: break-all; max-width: 100%; box-sizing: border-box; transition: all 0.5s ease 0s; border-collapse: collapse; min-width: 50px; overflow-wrap: break-word !important;"><p>CH 实例 1 端口: tcp 9000, http 8123, 同步端口 9009，MySQL 9004, 类型: 主分片 1</p><p>CH 实例 2 端口: tcp 9000, http 8124, 同步端口 9010，MySQL 9005, 类型: server02 的副本</p></td></tr><tr><td width="70" data-style="padding: 8px 14px; border-color: rgb(192, 192, 192); word-break: break-all; max-width: 100%; box-sizing: border-box; transition: all 0.5s ease 0s; border-collapse: collapse; min-width: 50px; overflow-wrap: break-word !important;">172.192.13.11</td><td width="98" data-style="padding: 8px 14px; border-color: rgb(192, 192, 192); word-break: break-all; max-width: 100%; box-sizing: border-box; transition: all 0.5s ease 0s; border-collapse: collapse; min-width: 50px; overflow-wrap: break-word !important;">server02</td><td width="71" data-style="padding: 8px 14px; border-color: rgb(192, 192, 192); word-break: break-all; max-width: 100%; box-sizing: border-box; transition: all 0.5s ease 0s; border-collapse: collapse; min-width: 50px; overflow-wrap: break-word !important;">Ubuntu20.04</td><td width="106" data-style="padding: 8px 14px; border-color: rgb(192, 192, 192); word-break: break-all; max-width: 100%; box-sizing: border-box; transition: all 0.5s ease 0s; border-collapse: collapse; min-width: 50px; overflow-wrap: break-word !important;">两个 Clickhouse 实例、Zookeeper</td><td width="235" data-style="padding: 8px 14px; border-color: rgb(192, 192, 192); word-break: break-all; max-width: 100%; box-sizing: border-box; transition: all 0.5s ease 0s; border-collapse: collapse; min-width: 50px; overflow-wrap: break-word !important;"><p>CH 实例 3 端口: tcp 9000, http 8123, 同步端口 9009，MySQL 9004, 类型: 主分片 2</p><p>CH 实例 4 端口: tcp 9000, http 8124, 同步端口 9010，MySQL 9005, 类型: server03 的副本</p></td></tr><tr><td height="201" width="70" data-style="padding: 8px 14px; border-color: rgb(192, 192, 192); word-break: break-all; max-width: 100%; box-sizing: border-box; transition: all 0.5s ease 0s; border-collapse: collapse; min-width: 50px; overflow-wrap: break-word !important;">172.192.13.12</td><td height="201" width="98" data-style="padding: 8px 14px; border-color: rgb(192, 192, 192); word-break: break-all; max-width: 100%; box-sizing: border-box; transition: all 0.5s ease 0s; border-collapse: collapse; min-width: 50px; overflow-wrap: break-word !important;">server03</td><td width="71" height="201" data-style="padding: 8px 14px; border-color: rgb(192, 192, 192); word-break: break-all; max-width: 100%; box-sizing: border-box; transition: all 0.5s ease 0s; border-collapse: collapse; min-width: 50px; overflow-wrap: break-word !important;">Ubuntu20.04</td><td width="106" height="201" data-style="padding: 8px 14px; border-color: rgb(192, 192, 192); word-break: break-all; max-width: 100%; box-sizing: border-box; transition: all 0.5s ease 0s; border-collapse: collapse; min-width: 50px; overflow-wrap: break-word !important;">两个 Clickhouse 实例、Zookeeper</td><td height="201" width="235" data-style="padding: 8px 14px; border-color: rgb(192, 192, 192); word-break: break-all; max-width: 100%; box-sizing: border-box; transition: all 0.5s ease 0s; border-collapse: collapse; min-width: 50px; overflow-wrap: break-word !important;"><p>CH 实例 5 端口: tcp 9000, http 8123, 同步端口 9009，MySQL 9004, 类型: 主分片 3</p><p>CH 实例 6 端口: tcp 9000, http 8124, 同步端口 9010，MySQL 9005, 类型: server01 的副本</p></td></tr></tbody></table>

在每一台服务上都安装 docker，docker 里面分别安装有 3 个服务：ch-main，ch-sub，zookeeper_node，如图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/YLdd61RZjGUiakUCunPT7QVHAJmfljQvjl8BzwAveeVea91qS7hPt4DHnPkzAxRsXmefHuKRm9ElpeoN0GBTZ7g/640?wx_fmt=png)

  

****环境部署****

  

**1. 服务器环境配置**

　　在每一台服务器上执行： vim /etc/hosts ，打开 hosts 之后新增配置：

```
172.192.13.10 server01
172.192.13.11 server02
172.192.13.12 server03

```

**2. 安装 docker**

　　太简单，略...

  

****Zookeeper 集群部署  
****

  

在每个服务器上你想存放的位置，新建一个文件夹来存放 zk 的配置信息，这里是 /usr/soft/zookeeper/，在每个服务器上依次运行以下启动命令：
------------------------------------------------------------------------------

```
docker run -d -p 2181:2181 -p 2888:2888 -p 3888:3888 --name zookeeper_node --restart always \
-v /usr/soft/zookeeper/data:/data \
-v /usr/soft/zookeeper/datalog:/datalog \
-v /usr/soft/zookeeper/logs:/logs \
-v /usr/soft/zookeeper/conf:/conf \
--network host  \
-e ZOO_MY_ID=1  zookeeper

```

```
docker run -d -p 2181:2181 -p 2888:2888 -p 3888:3888 --name zookeeper_node --restart always \
-v /usr/soft/zookeeper/data:/data \
-v /usr/soft/zookeeper/datalog:/datalog \
-v /usr/soft/zookeeper/logs:/logs \
-v /usr/soft/zookeeper/conf:/conf \
--network host  \
-e ZOO_MY_ID=2  zookeeper

```

```
docker run -d -p 2181:2181 -p 2888:2888 -p 3888:3888 --name zookeeper_node --restart always \
-v /usr/soft/zookeeper/data:/data \
-v /usr/soft/zookeeper/datalog:/datalog \
-v /usr/soft/zookeeper/logs:/logs \
-v /usr/soft/zookeeper/conf:/conf \
--network host  \
-e ZOO_MY_ID=3  zookeeper

```

唯一的差别是： -e ZOO_MY_ID=* 而已。

其次，每台服务上打开 /usr/soft/zookeeper/conf 路径，找到 zoo.cfg 配置文件，修改为：

```
dataDir=/data
dataLogDir=/datalog
tickTime=2000
initLimit=5
syncLimit=2
clientPort=2181
autopurge.snapRetainCount=3
autopurge.purgeInterval=0
maxClientCnxns=60
server.1=172.192.13.10:2888:3888
server.2=172.192.13.11:2888:3888
server.3=172.192.13.12:2888:3888

```

然后进入其中一台服务器，进入 zk 查看是否配置启动成功：

```
docker exec -it zookeeper_node /bin/bash
./bin/zkServer.sh status

```

  

****Clickhouse 集群部署****

运行一个临时容器，目的是为了将配置、数据、日志等信息存储到宿主机上：

拷贝容器内的文件：

```
docker cp temp-ch:/etc/clickhouse-server/ /etc/
//https://www.cnblogs.com/EminemJK/p/15138536.html

```

```
//同时兼容IPV6，一劳永逸
<listen_host>0.0.0.0</listen_host>
//设置时区
<timezone>Asia/Shanghai</timezone>
//删除原节点<remote_servers>的测试信息
<remote_servers incl="clickhouse_remote_servers" />
//新增，和上面的remote_servers 节点同级
<include_from>/etc/clickhouse-server/metrika.xml</include_from>
//新增，和上面的remote_servers 节点同级
<zookeeper incl="zookeeper-servers" optional="true" />
//新增，和上面的remote_servers 节点同级
<macros incl="macros" optional="true" />

```

其他 listen_host 仅保留一项即可，其他 listen_host 则注释掉。

```
cp -rf /etc/clickhouse-server/ /usr/soft/clickhouse-server/main
cp -rf /etc/clickhouse-server/ /usr/soft/clickhouse-server/sub

```

main 为主分片，sub 为副本。

```
#拷贝配置到server02上
scp /usr/soft/clickhouse-server/main/ server02:/usr/soft/clickhouse-server/main/
scp /usr/soft/clickhouse-server/sub/ server02:/usr/soft/clickhouse-server/sub/ 
#拷贝配置到server03上
scp /usr/soft/clickhouse-server/main/ server03:/usr/soft/clickhouse-server/main/
scp /usr/soft/clickhouse-server/sub/ server03:/usr/soft/clickhouse-server/sub/

```

SCP 真香。

然后就可以删除掉临时容器： docker rm -f temp-ch 

![](https://mmbiz.qpic.cn/mmbiz_png/LGh7bn8KbYAlhwhDBf3EfdjY9DUvsmeicc4aantO4BvUzjLOH7TyCHaic80icvEia2bZRIqGnaFedgmZofJxQHoPvg/640?wx_fmt=png)

  

****配置集群  
****

  

这里三台服务器，每台服务器起 2 个 CH 实例，环状相互备份，达到高可用的目的，资源充裕的情况下，可以将副本 Sub 实例完全独立出来，修改配置即可，这个又是 Clickhouse 的好处之一，横向扩展非常方便。

**1. 修改配置**

进入 **server1** 服务器， /usr/soft/clickhouse-server/sub/conf 修改 config.xml 文件，主要修改内容：

```
原：
<http_port>8123</http_port>
<tcp_port>9000</tcp_port>
<mysql_port>9004</mysql_port>
<interserver_http_port>9009</interserver_http_port>
修改为：
<http_port>8124</http_port>
<tcp_port>9001</tcp_port>
<mysql_port>9005</mysql_port>
<interserver_http_port>9010</interserver_http_port>

```

修改的目的目的是为了和主分片 **main** 的配置区分开来，端口不能同时应用于两个程序。server02 和 server03 如此修改或 scp 命令进行分发。

**2. 新增集群配置文件 metrika.xml**

**server01，main 主分片配置：**

进入 /usr/soft/clickhouse-server/main/conf 文件夹内，新增 metrika.xml 文件（文件编码：utf-8）。

```
<yandex>
    <!-- CH集群配置,所有服务器都一样 -->
    <clickhouse_remote_servers>
        <cluster_3s_1r>
            <!-- 数据分片1  -->
            <shard>
                <internal_replication>true</internal_replication>
                <replica>
                    <host>server01</host>
                    <port>9000</port>
                    <user>default</user>
                    <password></password>
                </replica>
                <replica>
                    <host>server03</host>
                    <port>9001</port>
                    <user>default</user>
                    <password></password>
                </replica>
            </shard>
            <!-- 数据分片2  -->
            <shard>
                <internal_replication>true</internal_replication>
                <replica>
                    <host>server02</host>
                    <port>9000</port>
                    <user>default</user>
                    <password></password>
                </replica>
                <replica>
                    <host>server01</host>
                    <port>9001</port>
                    <user>default</user>
                    <password></password>
                </replica>
            </shard>
            <!-- 数据分片3  -->
            <shard>
                <internal_replication>true</internal_replication>
                <replica>
                    <host>server03</host>
                    <port>9000</port>
                    <user>default</user>
                    <password></password>
                </replica>
                <replica>
                    <host>server02</host>
                    <port>9001</port>
                    <user>default</user>
                    <password></password>
                </replica>
            </shard>
        </cluster_3s_1r>
    </clickhouse_remote_servers>
    <!-- zookeeper_servers所有实例配置都一样 -->
    <zookeeper-servers>
        <node index="1">
            <host>172.16.13.10</host>
            <port>2181</port>
        </node>
        <node index="2">
            <host>172.16.13.11</host>
            <port>2181</port>
        </node>
        <node index="3">
            <host>172.16.13.12</host>
            <port>2181</port>
        </node>
    </zookeeper-servers>
    <!-- marcos每个实例配置不一样 -->
    <macros>
        <layer>01</layer>
        <shard>01</shard>
        <replica>cluster01-01-1</replica>
    </macros>
    <networks>
        <ip>::/0</ip>
    </networks>
    <!-- 数据压缩算法  -->
    <clickhouse_compression>
        <case>
            <min_part_size>10000000000</min_part_size>
            <min_part_size_ratio>0.01</min_part_size_ratio>
            <method>lz4</method>
        </case>
    </clickhouse_compression>
</yandex>

```

<macros> 节点每个服务器每个实例不同，其他节点配置一样即可，下面仅列举 <macros> 节点差异的配置。

**server01，sub 副本配置：**

```
<macros>
    <layer>01</layer>
    <shard>02</shard>
    <replica>cluster01-02-2</replica>
</macros>

```

**server02，main 主分片配置：**

```
<macros>
    <layer>01</layer>
    <shard>02</shard>
    <replica>cluster01-02-1</replica>
</macros>

```

**server02，sub 副本配置：**  

```
<macros>
    <layer>01</layer>
    <shard>03</shard>
    <replica>cluster01-03-2</replica>
</macros>

```

**server03，main 主分片配置：**

```
<macros>
    <layer>01</layer>
    <shard>03</shard>
    <replica>cluster01-03-1</replica>
</macros>

```

**server03，sub 副本配置：**

```
<macros>
    <layer>01</layer>
    <shard>02</shard>
    <replica>cluster01-01-2</replica>
</macros>

```

至此，已经完成全部配置，其他的比如密码等配置，可以按需增加。

![](https://mmbiz.qpic.cn/mmbiz_png/LGh7bn8KbYAlhwhDBf3EfdjY9DUvsmeicc4aantO4BvUzjLOH7TyCHaic80icvEia2bZRIqGnaFedgmZofJxQHoPvg/640?wx_fmt=png)

  

****集群运行及测试****

  

在每一台服务器上依次运行实例，zookeeper 前面已经提前运行，没有则需先运行 zk 集群。  

运行 main 实例：

```
docker run -d --name=ch-main -p 8123:8123 -p 9000:9000 -p 9009:9009 --ulimit nofile=262144:262144 \
-v /usr/soft/clickhouse-server/main/data:/var/lib/clickhouse:rw \
-v /usr/soft/clickhouse-server/main/conf:/etc/clickhouse-server:rw \
-v /usr/soft/clickhouse-server/main/log:/var/log/clickhouse-server:rw \
--add-host server01:172.192.13.10 \
--add-host server02:172.192.13.11 \
--add-host server03:172.192.13.12 \
--hostname server01 \
--network host \
--restart=always \
 yandex/clickhouse-server

```

运行 sub 实例：

```
docker run -d --name=ch-sub -p 8124:8124 -p 9001:9001 -p 9010:9010 --ulimit nofile=262144:262144 \
-v /usr/soft/clickhouse-server/sub/data:/var/lib/clickhouse:rw \
-v /usr/soft/clickhouse-server/sub/conf:/etc/clickhouse-server:rw \
-v /usr/soft/clickhouse-server/sub/log:/var/log/clickhouse-server:rw \
--add-host server01:172.192.13.10 \
--add-host server02:172.192.13.11 \
--add-host server03:172.192.13.12 \
--hostname server01 \
--network host \
--restart=always \
 yandex/clickhouse-server

```

在每台服务器执行命令，唯一不同的参数是 hostname，因为我们前面已经设置了 hostname 来指定服务器，否则在执行 select * from system.clusters 查询集群的时候，将 is_local 列全为 0，则表示找不到本地服务，这是需要注意的地方。在每台服务器的实例都启动之后，这里使用正版 DataGrip 来打开：

![](https://mmbiz.qpic.cn/mmbiz_png/YLdd61RZjGUiakUCunPT7QVHAJmfljQvjJpp9c28orsMVpOd7u5OiclreCL1zVkSpBSx7DcxlJVwNN09G1Bgt3Ew/640?wx_fmt=png)

在任一实例上新建一个查询：

```
create table T_UserTest on cluster cluster_3s_1r
(
    ts  DateTime,
    uid String,
    biz String
)
    engine = ReplicatedMergeTree('/clickhouse/tables/{layer}-{shard}/T_UserTest', '{replica}')
        PARTITION BY toYYYYMMDD(ts)
        ORDER BY ts
        SETTINGS index_granularity = 8192;

```

cluster_3s_1r 是前面配置的集群名称，需一一对应上， /clickhouse/tables/ 是固定的前缀，相关语法可以查看官方文档了。

![](https://mmbiz.qpic.cn/mmbiz_png/YLdd61RZjGUiakUCunPT7QVHAJmfljQvjA0C8pRGIRvtTNq1hBiaibXbEVPTMNg465n0hJtVUiar0RcMgov1Q2VLzg/640?wx_fmt=png)

刷新每个实例，即可看到全部实例中都有这张 T_UserTest 表，因为已经搭建 zookeeper，很容易实现分布式 DDL。

继续新建 Distributed 分布式表：

```
CREATE TABLE T_UserTest_All ON CLUSTER cluster_3s_1r AS T_UserTest ENGINE = Distributed(cluster_3s_1r, default,  T_UserTest, rand())

```

每个主分片分别插入相关信息：

```
--server01
insert into  T_UserTest values ('2021-08-16 17:00:00',1,1)
--server02
insert into  T_UserTest values ('2021-08-16 17:00:00',2,1)
--server03
insert into  T_UserTest values ('2021-08-16 17:00:00',3,1)

```

然后查询分布式表 select * from T_UserTest_All ，

![](https://mmbiz.qpic.cn/mmbiz_png/YLdd61RZjGUiakUCunPT7QVHAJmfljQvjkia5GvgyVdopV9FqUjBMTEiaI2OHT0rq1xc4iaMrcB9hkb2dPdm1tAyibg/640?wx_fmt=png)

查询对应的副本表或者关闭其中一台服务器的 docker 实例，查询也是不受影响，时间关系不在测试。

![](https://mmbiz.qpic.cn/mmbiz_png/LGh7bn8KbYAlhwhDBf3EfdjY9DUvsmeicc4aantO4BvUzjLOH7TyCHaic80icvEia2bZRIqGnaFedgmZofJxQHoPvg/640?wx_fmt=png)

  

**最后**

  

下班。

> 本文作者：山治先生
> 
> 声明：本文为订阅号投稿作品，未经作者允许请勿转载。

![](https://mmbiz.qpic.cn/mmbiz_png/ou3VZp3NaRX6GpkIx5viczHvpFq0vGGe6CibjtRvnaYbjibsZ7QshFb0ib9VkndcCLzIJy1x8fSToaawyYFXC5U81A/640?wx_fmt=jpeg)

小宝贝，你没看错

订阅号的确是 “xLong 设计”，发的也的确是 IT 技术文章

如果喜欢就支持一下作者吧！
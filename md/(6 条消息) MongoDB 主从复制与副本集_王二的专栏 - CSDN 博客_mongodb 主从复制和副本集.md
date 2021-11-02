> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/dutsoft/article/details/78088779)

MongoDB 复制
----------

### 概念

### 复制的特点

*   保障数据的安全性
*   数据高可用性
*   灾难恢复
*   无需停机维护 (如备份，重建索引，压缩)
*   分布式读取数据 (提高读取能力)  
    mongodb 支持副本集和主从复制，主从复制官方已不再推荐（不支持自动故障切换）。

主从复制（Master-Slave）
------------------

主从复制是可用于备份，故障恢复和读扩展等。基本就是搭建一个主节点和一个或多个从节点，在数据库集群中要明确的知道谁是主服务器，主服务器只有一台。从服务器要知道自己的数据源也就是对应的主服务是谁。  
运行`mongod --master`启动主服务器。  
运行`mongod --slave --source master_address`启动从服务器。

### 调整配置

主要是指定了一下 dbPath 和 log path，master 配置如下：

```
$ cat master.conf
# mongod.conf

# for documentation of all options, see:
#   http://docs.mongodb.org/manual/reference/configuration-options/

# Where and how to store data.
storage:
  dbPath: /home/work/mongo/master
  journal:
    enabled: true
#  engine:
#  mmapv1:
#  wiredTiger:

# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /home/work/mongo/master/mongod.log

# network interfaces
net:
  port: 27017
  bindIp: 127.0.0.1
```

### 启动 mongo 实例

```
mongod -f master.conf --logappend --master --fork
mongod -f slave-1.conf --logappend --slave --fork --source localhost:27017
mongod -f slave-2.conf --logappend --slave --fork --source localhost:27017
```

所有的从节点都是从主节点复制信息，目前还不能从节点到从节点复制机制，原因是从节点没有自己的 oplog。  
一个集群中从节点没有明确的限制，但是多个节点对单点主机发起的查询也是吃不消的，不超过 12 个节点的集群可以良好运转。

### 功能测试

在主库中向 user 集合插入一条数据。

```
> use test
switched to db test
> db.user.insert({'u':'test', 'p':'pwd'})
WriteResult({ "nInserted" : 1 })
> db.user.find()
{ "_id" : ObjectId("59c8d10c836ac6ec4e425fc5"), "u" : "test", "p" : "pwd" }
```

登录从库，可以看到 user 集合插入的记录已经同步过来了。

```
> rs.slaveOk()
> show dbs
admin  0.000GB
local  0.000GB
test   0.000GB
> use test
switched to db test
> show collections;
user
> db.user.find()
{ "_id" : ObjectId("59c8d10c836ac6ec4e425fc5"), "u" : "test", "p" : "pwd" }
```

### 其他配置项

*   –only 从节点指定复制某个数据库, 默认是复制全部数据库
*   –slavedelay 从节点设置主数据库同步数据的延迟 (单位是秒)
*   –fastsync 从节点以主数据库的节点快照为节点启动从数据库
*   –autoresync 从节点如果不同步则从新同步数据库 (即选择当通过热添加了一台从服务器之后，从服务器选择是否更新主服务器之间的数据)
*   –oplogSize 主节点设置 oplog 的大小 (主节点操作记录存储到 local 的 oplog 中), 单位 MB

副本集（Replica Set）
----------------

副本集就是有**自动故障恢复功能的主从集群**，由一个 primary 节点和一个或多个 secondary 节点组成。主从集群和副本集最为明显的区别就是副本集没有固定的主节点：整个集群会选举出一个主节点，当其不能工作时，则变更到其它节点。  
副本集总会有一个活跃节点和一个或多个备份节点。

### 工作原理

1.  mongodb 的复制至少需要两个实例。其中一个是主节点，负责处理客户端请求，其余的都是从节点，负责复制主节点上的数据。主节点记录在其上的所有操作 oplog，从节点定期轮询主节点获取这些操作，然后对自己的数据副本执行这些操作，从而保证从节点的数据与主节点一致。
2.  主节点的操作记录称为 oplog(operation log)，存储在 local 数据库中 (local 数据库不会被复制，用来存放复制状态信息的)。oplog 中的每个文档代表着主节点上执行的操作。oplog 只作为从节点与主节点保持数据同步的机制。
3.  oplog.rs 是一个固定长度的 capped collection。默认情况下，64 位的实例将使用 oplog 5% 的可用空间，这个空间将在 local 数据库中分配，并在服务器启动时预先分配。
4.  如果从节点落后主节点很远了，oplog 日志从节点还没执行完，oplog 可能已经轮滚一圈了，那么从节点将会追赶不上主节点了，复制将会停止。从节点需要重新做完整的同步，可以用 {resync:1} 命令来手动执行重新同步或在启动从节点时指定–autoresync 选项让其自动重新同步。重新同步的代价昂贵，应尽量避免，避免的方法就是配置足够大的 oplog。

### 启动实例

rep1，2，3 分别以 27017，27018，27019 端口启动。

```
work@jdu4e00u53f7:~/mongo$ mongod -f rep1.conf --replSet rs0 --fork
work@jdu4e00u53f7:~/mongo$ mongod -f rep2.conf --replSet rs0 --fork
work@jdu4e00u53f7:~/mongo$ mongod -f rep3.conf --replSet rs0 --fork
work@jdu4e00u53f7:~/mongo$ mongo --port 27017
```

### 初始化副本集

```
> rsconf = {'_id':'rs0', 'members':[{'_id': 0, 'host':'127.0.0.1:27017'}, {'_id': 1, 'host': '127.0.0.1:27018'}, {'_id':2, 'host': '127.0.0.1:27019'}]}
{
    "_id" : "rs0",
    "members" : [
        {
            "_id" : 0,
            "host" : "127.0.0.1:27017"
        },
        {
            "_id" : 1,
            "host" : "127.0.0.1:27018"
        },
        {
            "_id" : 2,
            "host" : "127.0.0.1:27019"
        }
    ]
}
> rs.initiate(rsconf)
{ "ok" : 1 }
```

### 查看当前配置

```
rs0:OTHER> rs.conf()
{
    "_id" : "rs0",
    "version" : 1,
    "protocolVersion" : NumberLong(1),
    "members" : [
        {
            "_id" : 0,
            "host" : "127.0.0.1:27017",
            "arbiterOnly" : false,
            "buildIndexes" : true,
            "hidden" : false,
            "priority" : 1,
            "tags" : {

            },
            "slaveDelay" : NumberLong(0),
            "votes" : 1
        },
        {
            "_id" : 1,
            "host" : "127.0.0.1:27018",
            "arbiterOnly" : false,
            "buildIndexes" : true,
            "hidden" : false,
            "priority" : 1,
            "tags" : {

            },
            "slaveDelay" : NumberLong(0),
            "votes" : 1
        },
        {
            "_id" : 2,
            "host" : "127.0.0.1:27019",
            "arbiterOnly" : false,
            "buildIndexes" : true,
            "hidden" : false,
            "priority" : 1,
            "tags" : {

            },
            "slaveDelay" : NumberLong(0),
            "votes" : 1
        }
    ],
    "settings" : {
        "chainingAllowed" : true,
        "heartbeatIntervalMillis" : 2000,
        "heartbeatTimeoutSecs" : 10,
        "electionTimeoutMillis" : 10000,
        "catchUpTimeoutMillis" : 60000,
        "getLastErrorModes" : {

        },
        "getLastErrorDefaults" : {
            "w" : 1,
            "wtimeout" : 0
        },
        "replicaSetId" : ObjectId("59c8e527002aa17b30c3408c")
    }
}
```

在 mongodb 会将配置里的节点自动添加到副本集 rs0 中。

### 查看主节点

```
rs0:SECONDARY> rs.isMaster()
{
    "hosts" : [
        "127.0.0.1:27017",
        "127.0.0.1:27018",
        "127.0.0.1:27019"
    ],
    "setName" : "rs0",
    "setVersion" : 1,
    "ismaster" : false,
    "secondary" : true,
    "primary" : "127.0.0.1:27017",
    "me" : "127.0.0.1:27018",
    "lastWrite" : {
        "opTime" : {
            "ts" : Timestamp(1506338789, 1),
            "t" : NumberLong(1)
        },
        "lastWriteDate" : ISODate("2017-09-25T11:26:29Z")
    },
    "maxBsonObjectSize" : 16777216,
    "maxMessageSizeBytes" : 48000000,
    "maxWriteBatchSize" : 1000,
    "localTime" : ISODate("2017-09-25T11:26:37.835Z"),
    "maxWireVersion" : 5,
    "minWireVersion" : 0,
    "readOnly" : false,
    "ok" : 1
}
```

### 查看 oplog 状态

```
rs0:SECONDARY> db.printReplicationInfo()
configured oplog size:   1214.6957025527954MB
log length start to end: 862secs (0.24hrs)
oplog first event time:  Mon Sep 25 2017 19:14:47 GMT+0800 (CST)
oplog last event time:   Mon Sep 25 2017 19:29:09 GMT+0800 (CST)
now:                     Mon Sep 25 2017 19:29:15 GMT+0800 (CST)
```

功能测试
----

### 复制功能

在主库中插入一条数据。

```
rs0:PRIMARY> use testdb
switched to db testdb
rs0:PRIMARY> db.user.insert({'u':'test', 'p':'123'})
WriteResult({ "nInserted" : 1 })
rs0:PRIMARY> db.user.find()
{ "_id" : ObjectId("59c8e698c28239e0dfd6f578"), "u" : "test", "p" : "123" }
```

登录从库，可以看到数据已经同步过来了。

```
$ mongo --port 27018
rs0:SECONDARY> rs.slaveOk()
rs0:SECONDARY> show dbs;
admin   0.000GB
local   0.000GB
testdb  0.000GB
rs0:SECONDARY> use testdb
switched to db testdb
rs0:SECONDARY> db.user.find()
{ "_id" : ObjectId("59c8e698c28239e0dfd6f578"), "u" : "test", "p" : "123" }
```

### 自动故障转移

关闭主节点

```
# 连接到主节点primary
$ mongo --port 27017
# 切换到 admin 数据库（关掉数据库命令只能在 admin 数据库）
rs0:PRIMARY> use admin
switched to db admin
rs0:PRIMARY> db.shutdownServer()
server should be down...
```

查看节点状态

```
rs0:PRIMARY> rs.status()
{
    "set" : "rs0",
    "date" : ISODate("2017-09-25T11:31:22.492Z"),
    "myState" : 1,
    "term" : NumberLong(2),
    "heartbeatIntervalMillis" : NumberLong(2000),
    "optimes" : {
        "lastCommittedOpTime" : {
            "ts" : Timestamp(1506339074, 1),
            "t" : NumberLong(2)
        },
        "appliedOpTime" : {
            "ts" : Timestamp(1506339074, 1),
            "t" : NumberLong(2)
        },
        "durableOpTime" : {
            "ts" : Timestamp(1506339074, 1),
            "t" : NumberLong(2)
        }
    },
    "members" : [
        {
            "_id" : 0,
            "name" : "127.0.0.1:27017",
            "health" : 0,
            "state" : 8,
            "stateStr" : "(not reachable/healthy)",
            "uptime" : 0,
            "optime" : {
                "ts" : Timestamp(0, 0),
                "t" : NumberLong(-1)
            },
            "optimeDurable" : {
                "ts" : Timestamp(0, 0),
                "t" : NumberLong(-1)
            },
            "optimeDate" : ISODate("1970-01-01T00:00:00Z"),
            "optimeDurableDate" : ISODate("1970-01-01T00:00:00Z"),
            "lastHeartbeat" : ISODate("2017-09-25T11:31:21.532Z"),
            "lastHeartbeatRecv" : ISODate("2017-09-25T11:30:42.934Z"),
            "pingMs" : NumberLong(0),
            "lastHeartbeatMessage" : "Connection refused",
            "configVersion" : -1
        },
        {
            "_id" : 1,
            "name" : "127.0.0.1:27018",
            "health" : 1,
            "state" : 1,
            "stateStr" : "PRIMARY",
            "uptime" : 1057,
            "optime" : {
                "ts" : Timestamp(1506339074, 1),
                "t" : NumberLong(2)
            },
            "optimeDate" : ISODate("2017-09-25T11:31:14Z"),
            "infoMessage" : "could not find member to sync from",
            "electionTime" : Timestamp(1506339053, 1),
            "electionDate" : ISODate("2017-09-25T11:30:53Z"),
            "configVersion" : 1,
            "self" : true
        },
        {
            "_id" : 2,
            "name" : "127.0.0.1:27019",
            "health" : 1,
            "state" : 2,
            "stateStr" : "SECONDARY",
            "uptime" : 992,
            "optime" : {
                "ts" : Timestamp(1506339074, 1),
                "t" : NumberLong(2)
            },
            "optimeDurable" : {
                "ts" : Timestamp(1506339074, 1),
                "t" : NumberLong(2)
            },
            "optimeDate" : ISODate("2017-09-25T11:31:14Z"),
            "optimeDurableDate" : ISODate("2017-09-25T11:31:14Z"),
            "lastHeartbeat" : ISODate("2017-09-25T11:31:21.530Z"),
            "lastHeartbeatRecv" : ISODate("2017-09-25T11:31:21.725Z"),
            "pingMs" : NumberLong(0),
            "syncingTo" : "127.0.0.1:27018",
            "configVersion" : 1
        }
    ],
    "ok" : 1
}
```

可以看到`127.0.0.1:27017`已经是不可达状态；`127.0.0.1:27018`被选为新的主节点；自动故障转移测试成功。重新启动 27017 的实例，会发现`127.0.0.1:27017`会成为从节点。

### 其他配置

*   standard 常规节点: 参与投票有可能成为活跃节点
*   passive 副本节点: 参与投票, 但是不能成为活跃节点
*   arbiter 仲裁节点: 只是参与投票不复制节点也不能成为活跃节点
*   Priority 0 到 1000 之间 ,0 代表是副本节点 ,1 到 1000 是常规节点
*   arbiterOnly : true 仲裁节点

操作环境
----

京东入门级云服务器，Ubuntu16.04，Mongo3.4

参考
--

*   [MongoDB 的主从复制及副本集的 replSet 配置教程](https://www.teakki.com/p/57e234456ef03829195216ae)
*   [mongodb 副本集（Replica Set）搭建](https://deepzz.com/post/mongodb-replica-set.html)
*   [mognodb 副本集复制概念理解](https://deepzz.com/post/mongodb-replica-set-concept.html)
*   [MongoDB 复制集原理](https://www.qcloud.com/community/article/164816001481011848)
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/lishiming0308/article/details/78404863)

一、概述
====

       MongoDB 复本集解决了数据库的备份与自动故障转移，但是围绕数据库的业务中当前还有两个方面的问题变得越来越重要，一是海量数据如何存储，二是如何高效地读写海量数据。尽管复制集也可以实现读写分离，如在 primary 节点上写，在 secondary 节点上读，但在这种方式下客户端读出来的数据有可能不是最新的，因为 primary 节点到 secondary 节点间的数据同步会带来一定延迟，而且这种方式也不能处理大量数据。MongoDB 引入了分片机制，实现了海量数据的分布式存储与高效的读写分离。复制集中的每个成员是一个 mongod 实例，但在分片部署上，每一个分片可能就是一个复制集。

      MongoDB 使用内存映射文件的方式来读写数据库，对内存的管理由操作系统来负责。随着时间的推移，数据库的索引和数据文件会变得越来越大，对于单节点的机器来说，迟早会突破内存的限制。当磁盘上的数据文件和索引远大于内存的大小时，操作系统会频繁地进行内存交换，导致整个数据库系统的读写性能下降。

      因此对于大数据的处理，要时刻监控 MongoDB 的磁盘 I/O 性能、可用内存的大小，在数据库内使用率达到一定程度时就要考虑分片了，通过分片使整个数据库分布在各个片上，每个片拥有数据库的一部分数据，从而降低内存使用率，提高读写性能。

二、应用架构图
=======

![](https://img-blog.csdn.net/20171031164032317)

当副本集中 Priamary 节点不可用，Secondary 节点会变成 Primary 节点。仲裁节点不会被选为 Primary 节点。这个图是初始化是的有一个状态。

在运行中主节点和从节点是会互相转换的。

上图中是我们的业务系统访问 Mongo 数据库集群的一个比较典型的架构。首先我们需要三台 MongoDB 服务器节点。

<table border="1" cellpadding="0" cellspacing="0"><tbody><tr><td><p>名称</p></td><td><p>地址</p></td><td><p>操作系统版本</p></td><td><p>MongoDB 版本</p></td><td><p>作用</p></td></tr><tr><td><p>ServerA</p></td><td><p>192.168.80.150</p></td><td><p>Server2008SP2</p></td><td><p>3.6</p></td><td><p>MongoDB 数据库 Primary,Mongos 路由服务、Mongos 配置服务 Primary</p></td></tr><tr><td><p>ServerB</p></td><td><p>192.168.80.151</p></td><td><p>Server2008SP2</p></td><td><p>3.6</p></td><td><p>MongoDB 数据库 Secondary,Mogos 路由服务、Mongos 配置服务 Secondary</p></td></tr><tr><td><p>ServerC</p></td><td><p>192.168.80.152</p></td><td><p>Server2008SP2</p></td><td><p>3.6</p></td><td><p>MongoDB 数据库 arbiter,Mongos 路由、Mongos 配置服务器 Secondary</p></td></tr></tbody></table>

三、副本集架构原理
=========

    mongodb 的复制至少需要两个节点。其中一个是主节点，负责处理客户端请求，其余的都是从节点，负责复制主节点上的数据。

mongodb 各个节点常见的搭配方式为：一主一从、一主多从。

     主节点记录在其上的所有操作 oplog，从节点定期轮询主节点获取这些操作，然后对自己的数据副本执行这些操作，从而保证从节点的数据与主节点一致。

MongoDB 复制结构图如下所示：

![](https://img-blog.csdn.net/20171031164106317)

以上结构图中，客户端从主节点读取数据，在客户端写入数据到主节点时，主节点与从节点进行数据交互保障数据的一致性。

副本集特征：

N 个节点的集群

任何节点可作为主节点

所有写入操作都在主节点上

自动故障转移

自动恢复

四、分片架构图
=======

在 Mongodb 里面存在另一种集群，就是分片技术, 可以满足 MongoDB 数据量大量增长的需求。

当 MongoDB 存储海量的数据时，一台机器可能不足以存储数据，也可能不足以提供可接受的读写吞吐量。这时，我们就可以通过在多台机器上分割数据，使得数据库系统能存储和处理更多的数据

为什么使用分片

复制所有的写入操作到主节点

延迟的敏感数据会在主节点查询

单个副本集限制在 12 个节点

当请求量巨大时会出现内存不足。

本地磁盘不足

垂直扩展价格昂贵

![](https://img-blog.csdn.net/20171031164110239)

上图中主要有如下所述三个主要组件：

Shard:

用于存储实际的数据块，实际生产环境中一个 shard server 角色可由几台机器组个一个 replica set 承担，防止主机单点故障

ConfigServer:

mongod 实例，存储了整个 ClusterMetadata，其中包括 chunk 信息。

QueryRouters:

前端路由，客户端由此接入，且让整个集群看上去像单一数据库，前端应用可以透明使用

五、部署节点配置文件
==========

在三台服务器上安装 mongoDB 文件，这个安装很简单，单击安装文件一步一步安装即可。给个社区版本下载地址连接:[https://www.mongodb.com/download-center?jmp=nav#community](https://www.mongodb.com/download-center?jmp=nav#community)

笔者一般使用 Robo 作为 GUI 工具，Robo 分为免费版和专业版本。免费版本功能基本够用，下载地址连接:[https://robomongo.org/](https://robomongo.org/)

MongoDB 中设置数据库和日志路径一般不要放置在系统盘符下，以免系统崩溃而数据丢失。由笔者是在虚拟中运行就一个盘符就都放置在 C 盘中了。

安装后的目录截图如下

![](https://img-blog.csdnimg.cn/20190403145606358.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2hpbWluZzAzMDg=,size_16,color_FFFFFF,t_70)

首先进入 192.168.80.150 服务器做如下配置。其他两台服务器和这台服务器配置相同。

需要建立文件夹 Config、Mongos、Share1、Share2、Share3。每个文件下建 Data 和 Log 两个目录。分别用于存放数据和日志。

创建副本集和分片安全验证的秘钥文件 KeyFile.txt，文件中存储 6~1024 个字符用于 mongodb 集群中的内部验证。

需要手动建 5 个配置文件 mongoConfig.cfg、mongos.cfg、mongoShare1.cfg、mongoShare2.cfg、mongoShare3.cfg

mongoConfig.cfg: 用于配置 mongos 使用的配置服务数据库信息，配置数据库服务和正常的 Mongo 库无大区别，只是它会默认建 Config 数据库，用于存放分片的配置信息。

mongos.cfg:Mongo 分片的路由服务，mongos 路由进程是一个轻量级且非持久性的进程。它不会保存任何数据库中的数据，它只是将整个分片集群看成一个整体，使分片集群对整个客户端程序来说是透明的。当客户端发起读写操作时，由 mongos 路由进程将该操作路由到具体的分片上进行；为了实现对读写请求的路由，mongos 进行必须知道整个分片集群上所有数据库的分片情况，即元数据。这些信息是从配置服务器上同步过来的，每次进程启动时都会从 config server 服务器上读元信息，mongos 并非持久化保存这些信。

mongoShare1.cfg、mongoShare2、mongoShare3.cfg: 这是三个分片数据库的配置文件。笔者一般将片分为三个。**三是个很特别的数字，一生二，二生三，三生万物**。

![](https://img-blog.csdn.net/20180707170024884?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2hpbWluZzAzMDg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**注意: 当 mongodb 不支持远程访问时在配置文件中加入 bind_ip = 0.0.0.0 以支持远程 IP 地址访问。在 mongodb3.4 版本默认是支持远程 IP 访问的，在 3.5 以后版本中不默认开启需要明确指定。**

![](https://img-blog.csdn.net/20180707164953735?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2hpbWluZzAzMDg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![](https://img-blog.csdn.net/20180707165047697?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2hpbWluZzAzMDg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![](https://img-blog.csdn.net/20180707165109752?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2hpbWluZzAzMDg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![](https://img-blog.csdn.net/20180707165129905?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2hpbWluZzAzMDg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![](https://img-blog.csdn.net/20180707165148899?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2hpbWluZzAzMDg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

六、初始化副本集和分片
===========

1、安装服务
------

在三台服务器上分别执行下列命令安装服务

#安装配置服务 下边是一行名

"C:\Program Files\MongoDB\Server\3.6\bin\mongod" --config "C:\Program Files\MongoDB\Server\3.6\mongoConfig.cfg" --install

#启动配置服务

sc startMongoDBConfig

#安装 mongos 服务

"C:\Program Files\MongoDB\Server\3.6\bin\mongos" --config "C:\Program Files\MongoDB\Server\3.6\mongos.cfg" --install

#启动 mongos 服务

sc startMongoDBS

#安装副本集

"C:\Program Files\MongoDB\Server\3.6\bin\mongod" --config "C:\Program Files\MongoDB\Server\3.6\mongoShare1.cfg" --install

sc startMongoDBShare1

"C:\Program Files\MongoDB\Server\3.6\bin\mongod" --config "C:\Program Files\MongoDB\Server\3.6\mongoShare2.cfg" --install

sc startMongoDBShare2

"C:\Program Files\MongoDB\Server\3.6\bin\mongod" --config "C:\Program Files\MongoDB\Server\3.6\mongoShare3.cfg" --install

sc startMongoDBShare3

现在启动 mogs 的服务启动不成功，需要进入任意配置服务器配置副本集。

2、初始化副本集
--------

#进入 192.168.80.152:21000, 初始化配置服务器副本集

![](https://img-blog.csdnimg.cn/20190403172756348.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2hpbWluZzAzMDg=,size_16,color_FFFFFF,t_70)

use admin

config={  
    _id:"rscfg",  
    members:[  
    {_id:0,host:"192.168.80.150:21000",priority:10},  
    {_id:1,host:"192.168.80.151:21000",priority:9},  
    {_id:2,host:"192.168.80.152:21000",priority:9},  
    ],  
    configsvr:true  
    }  
rs.initiate(config)

服务安装成功后会有 5 个 Mongo 的后台服务

![](https://img-blog.csdn.net/20171031164142712)

#下边配置分片的副本集，在服务器 192.168.80.150 上使用 shell 进入每个分片端口

例如

mongo localhost:27017 

mongo localhost:27018

mongo localhost27019

#进入 mongo localhost:27017 

use admin

config={

    _id:"share1",

    members:[

   {_id:0,host:"192.168.80.150:27017",priority:10},

   {_id:1,host:"192.168.80.151:27017",priority:9},

   {_id:2,host:"192.168.80.152:27017",priority:0,arbiterOnly:true},

    ]

    }

 rs.initiate(config)

 # 创建管理员  
admin=db.getSiblingDB("admin")  
admin.createUser({  
"user":"admin",  
"pwd":"admin",  
roles:[  
{  
"role":"userAdminAnyDatabase",  
"db":"admin"  
},  
{  
"role":"clusterAdmin",  
"db":"admin"  
}  
]  
})![](https://img-blog.csdn.net/20171031164146788)

#进入 mongo localhost:27018

use admin

config={

    _id:"share2",

    members:[

   {_id:0,host:"192.168.80.150:27018",priority:10},

   {_id:1,host:"192.168.80.151:27018",priority:9},

   {_id:2,host:"192.168.80.152:27018",priority:0,arbiterOnly:true},

    ]

    }

 rs.initiate(config)

#创建管理员  
admin=db.getSiblingDB("admin")  
admin.createUser({  
"user":"admin",  
"pwd":"admin",  
roles:[  
{  
"role":"userAdminAnyDatabase",  
"db":"admin"  
},  
{  
"role":"clusterAdmin",  
"db":"admin"  
}  
]  
})

#进入 mongo localhost:27019

use admin

config={

    _id:"share3",

    members:[

   {_id:0,host:"192.168.80.150:27019",priority:10},

   {_id:1,host:"192.168.80.151:27019",priority:9},

   {_id:2,host:"192.168.80.152:27019",priority:0,arbiterOnly:true},

    ]

    }

 rs.initiate(config)

#创建管理员  
admin=db.getSiblingDB("admin")

use admin  
admin.createUser({  
"user":"admin",  
"pwd":"admin",  
roles:[  
{  
"role":"userAdminAnyDatabase",  
"db":"admin"  
},  
{  
"role":"clusterAdmin",  
"db":"admin"  
}  
]  
})

2、初始化分片

#进入 192.168.80.150:20000 添加分片

sh.addShard("share1/192.168.80.150:27017,192.168.80.151:27017")

sh.addShard("share2/192.168.80.150:27018,192.168.80.151:27018")

sh.addShard("share3/192.168.80.150:27019,192.168.80.151:27019")

#创建超级管理员管理员

use admin

db.createUser(   
  {   
    user: "admin",   
    pwd: "admin",   
    roles: [ {  
            "role" : "userAdminAnyDatabase",  
            "db" : "admin"  
        }, {role: "root", db: "admin"} ]   
  } );

#管理员登录

use admin  
db.auth("admin","admin")  
#查看分片  
sh.status()

![](https://img-blog.csdn.net/20171031164151372)

七、数据库集合启动分片
===========

集合启动分片后会将集合划分为更小的数据块 chunk, 每个 chunk 默认大小是 64M。可以使用如下命令修改 chunk 块的大小。

进入 Mongos 192.168.80.150：20000

use config

db.settings.save({_id: "chunksize", value : 4})

将 chunk 大小设置为 4M 这样方面我们插入测试数据，看看分片情况。生产环境应该设置成 64M。.

#使用数据库可以分片

sh.enableSharding("test")

#使数据中的集合可以分片, shardKey 的选择要慎重，要保证数据分布的均匀性，高随机性等。shardKey 设置之后无法修改。

sh.shardCollection("test.test",{uid:1})

#切换数据, 并插入测试数据

use test

for(var i=0;i<5000;i++)

{

db.test.insert({uid:i,name:"lsm",age:30})

}

在数据集合超过 64M 大小时才会触发数据重新划分到新片上的操作

#查看 Config 库下的 chunks 表

![](https://img-blog.csdn.net/20171031164155935)

分片的选择相当重要，不在本文叙述范围之内，大家可以自己去查阅相关资料如何选择分片的字段。本文中选择的 id 作为分片字段，只是一个示例，不适用于生成环境。

八、三方应用程序访问
==========

mongDB 提供了很多驱动供其他语言访问例如 C#、java、Php 等给一个 C# 的源码驱动地址连接：[https://github.com/mongodb/mongo-csharp-driver](https://github.com/mongodb/mongo-csharp-driver)

mongodb://192.168.80.150:20000,192.168.80.151:20000,192.168.80.152:20000/?slaveOk=true;safe=true;w=2;wtimeoutMS=2000

使用这个连接字符串就可以访问 Mongo 集群了。连接字符串中的具体意思详见：[http://www.runoob.com/mongodb/mongodb-connections.html](http://www.runoob.com/mongodb/mongodb-connections.html)
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI2OTQxMTM4OQ==&mid=2247523782&idx=4&sn=61fd9d86ece2eac10296ac9a5cebdfbb&chksm=eae26894dd95e182bd536848dbb6793992e3d94879422131931ad742422ac15a1b35de290221&mpshare=1&scene=1&srcid=0722ziUdVgBHL2ny1wo522Pd&sharer_sharetime=1658471328554&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

目录  

*   一、准备表 & 数据
    
*   二、500w 级数据测试
    

*   2.1 录入 500W 数据，自增 ID 节省一半磁盘空间
    
*   2.2 单个数据走索引查询，自增 id 和 uuid 相差不大
    
*   2.3 范围 like 查询，自增 ID 性能优于 UUID
    
*   2.4 写入测试，自增 ID 是 UUID 的 4 倍
    
*   2.5、备份和恢复，自增 ID 性能优于 UUID
    
*   500W 总结
    
*   1000W 总结
    
*   自增 ID 主键 + 步长，适合中等规模的分布式场景
    
*   UUID，适合小规模的分布式环境
    

一、准备表 & 数据
----------

`UC_USER`，自增 ID 为主键，表结构类似如下：

```
CREATE TABLE `UC_USER` (  `ID` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',  `USER_NAME` varchar(100) DEFAULT NULL COMMENT '用户名',  `USER_PWD` varchar(200) DEFAULT NULL COMMENT '密码',  `BIRTHDAY` datetime DEFAULT NULL COMMENT '生日',  `NAME` varchar(200) DEFAULT NULL COMMENT '姓名',  `USER_ICON` varchar(500) DEFAULT NULL COMMENT '头像图片',  `SEX` char(1) DEFAULT NULL COMMENT '性别, 1:男，2:女，3：保密',  `NICKNAME` varchar(200) DEFAULT NULL COMMENT '昵称',  `STAT` varchar(10) DEFAULT NULL COMMENT '用户状态，01:正常，02:冻结',  `USER_MALL` bigint(20) DEFAULT NULL COMMENT '当前所属MALL',  `LAST_LOGIN_DATE` datetime DEFAULT NULL COMMENT '最后登录时间',  `LAST_LOGIN_IP` varchar(100) DEFAULT NULL COMMENT '最后登录IP',  `SRC_OPEN_USER_ID` bigint(20) DEFAULT NULL COMMENT '来源的联合登录',  `EMAIL` varchar(200) DEFAULT NULL COMMENT '邮箱',  `MOBILE` varchar(50) DEFAULT NULL COMMENT '手机',  `IS_DEL` char(1) DEFAULT '0' COMMENT '是否删除',  `IS_EMAIL_CONFIRMED` char(1) DEFAULT '0' COMMENT '是否绑定邮箱',  `IS_PHONE_CONFIRMED` char(1) DEFAULT '0' COMMENT '是否绑定手机',  `CREATER` bigint(20) DEFAULT NULL COMMENT '创建人',  `CREATE_DATE` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '注册时间',  `UPDATE_DATE` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '修改日期',  `PWD_INTENSITY` char(1) DEFAULT NULL COMMENT '密码强度',  `MOBILE_TGC` char(64) DEFAULT NULL COMMENT '手机登录标识',  `MAC` char(64) DEFAULT NULL COMMENT 'mac地址',  `SOURCE` char(1) DEFAULT '0' COMMENT '1:WEB,2:IOS,3:ANDROID,4:WIFI,5:管理系统, 0:未知',  `ACTIVATE` char(1) DEFAULT '1' COMMENT '激活，1：激活，0：未激活',  `ACTIVATE_TYPE` char(1) DEFAULT '0' COMMENT '激活类型，0：自动，1：手动',  PRIMARY KEY (`ID`),  UNIQUE KEY `USER_NAME` (`USER_NAME`),  KEY `MOBILE` (`MOBILE`),  KEY `IDX_MOBILE_TGC` (`MOBILE_TGC`,`ID`),  KEY `IDX_EMAIL` (`EMAIL`,`ID`),  KEY `IDX_CREATE_DATE` (`CREATE_DATE`,`ID`),  KEY `IDX_UPDATE_DATE` (`UPDATE_DATE`)) ENGINE=InnoDB AUTO_INCREMENT=7122681 DEFAULT CHARSET=utf8 COMMENT='用户表'
```

`UC_USER_PK_VARCHAR`表，字符串 ID 为主键，采用 uuid：

```
CREATE TABLE `UC_USER_PK_VARCHAR_1` (  `ID` varchar(36) CHARACTER SET utf8mb4 NOT NULL DEFAULT '0' COMMENT '主键',  `USER_NAME` varchar(100) DEFAULT NULL COMMENT '用户名',  `USER_PWD` varchar(200) DEFAULT NULL COMMENT '密码',  `BIRTHDAY` datetime DEFAULT NULL COMMENT '生日',  `NAME` varchar(200) DEFAULT NULL COMMENT '姓名',  `USER_ICON` varchar(500) DEFAULT NULL COMMENT '头像图片',  `SEX` char(1) DEFAULT NULL COMMENT '性别, 1:男，2:女，3：保密',  `NICKNAME` varchar(200) DEFAULT NULL COMMENT '昵称',  `STAT` varchar(10) DEFAULT NULL COMMENT '用户状态，01:正常，02:冻结',  `USER_MALL` bigint(20) DEFAULT NULL COMMENT '当前所属MALL',  `LAST_LOGIN_DATE` datetime DEFAULT NULL COMMENT '最后登录时间',  `LAST_LOGIN_IP` varchar(100) DEFAULT NULL COMMENT '最后登录IP',  `SRC_OPEN_USER_ID` bigint(20) DEFAULT NULL COMMENT '来源的联合登录',  `EMAIL` varchar(200) DEFAULT NULL COMMENT '邮箱',  `MOBILE` varchar(50) DEFAULT NULL COMMENT '手机',  `IS_DEL` char(1) DEFAULT '0' COMMENT '是否删除',  `IS_EMAIL_CONFIRMED` char(1) DEFAULT '0' COMMENT '是否绑定邮箱',  `IS_PHONE_CONFIRMED` char(1) DEFAULT '0' COMMENT '是否绑定手机',  `CREATER` bigint(20) DEFAULT NULL COMMENT '创建人',  `CREATE_DATE` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '注册时间',  `UPDATE_DATE` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '修改日期',  `PWD_INTENSITY` char(1) DEFAULT NULL COMMENT '密码强度',  `MOBILE_TGC` char(64) DEFAULT NULL COMMENT '手机登录标识',  `MAC` char(64) DEFAULT NULL COMMENT 'mac地址',  `SOURCE` char(1) DEFAULT '0' COMMENT '1:WEB,2:IOS,3:ANDROID,4:WIFI,5:管理系统, 0:未知',  `ACTIVATE` char(1) DEFAULT '1' COMMENT '激活，1：激活，0：未激活',  `ACTIVATE_TYPE` char(1) DEFAULT '0' COMMENT '激活类型，0：自动，1：手动',  PRIMARY KEY (`ID`),  UNIQUE KEY `USER_NAME` (`USER_NAME`),  KEY `MOBILE` (`MOBILE`),  KEY `IDX_MOBILE_TGC` (`MOBILE_TGC`,`ID`),  KEY `IDX_EMAIL` (`EMAIL`,`ID`),  KEY `IDX_CREATE_DATE` (`CREATE_DATE`,`ID`),  KEY `IDX_UPDATE_DATE` (`UPDATE_DATE`)) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='用户表';
```

二、500w 级数据测试
------------

```
# 自增id为主键的表mysql> select count(1) from UC_USER;+----------+| count(1) |+----------+|  5720112 |+----------+1 row in set (0.00 sec) # uuid为主键的表mysql> select count(1) from UC_USER_PK_VARCHAR_1;+----------+| count(1) |+----------+|  5720112 |+----------+1 row in set (1.91 sec)
```

#### 2.1 录入 500W 数据，自增 ID 节省一半磁盘空间

占据的空间容量来看，自增 ID 比 UUID 小一半左右。

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbuePlVR02fhBTJJA4zlPqMlLTZzT1eSpQL8n0xic9hNmricvZ3IToV9iaVZH6Ate2LsYStAmOibJ16sItg/640?wx_fmt=png)

#### 2.2 单个数据走索引查询，自增 id 和 uuid 相差不大

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbuePlVR02fhBTJJA4zlPqMlLbqRQ87Bl9YoaBF3l9bDtQqiat6jupHYDxibpAcVjiaZiclZ0ApenLvcX8Q/640?wx_fmt=png)

#### 2.3 范围 like 查询，自增 ID 性能优于 UUID

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbuePlVR02fhBTJJA4zlPqMlLunavb51icic8jum1PTARicicSq4X9BX6dtXLnrliauKBiajIWicLUib5GCUahg/640?wx_fmt=png)

#### 2.4 写入测试，自增 ID 是 UUID 的 4 倍

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbuePlVR02fhBTJJA4zlPqMlLVyq3HnibGcKsTtxCWjUQpGBLv8uiaK0fXrbdBdmOcJRdzib90MFjOZMdg/640?wx_fmt=png)

#### 2.5、备份和恢复，自增 ID 性能优于 UUID

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbuePlVR02fhBTJJA4zlPqMlLlUveicduj6p1ibDfFKytQG0VMFINbJyLXTgJuCIkJ2aS46LPD1sCNohw/640?wx_fmt=png)

### 500W 总结

在 500W 记录表的测试下：

（1） 普通单条或者 20 条左右的记录检索，uuid 为主键的相差不大几乎效率相同；

（2） 但是范围查询特别是上百成千条的记录查询，自增 id 的效率要大于 uuid；

（3） 在范围查询做统计汇总的时候，自增 id 的效率要大于 uuid；

（4） 在存储上面，自增 id 所占的存储空间是 uuid 的 1/2；

（5） 在备份恢复上，自增 ID 主键稍微优于 UUID。

### 1000W 总结

在 1000W 记录表的测试下：

（1）普通单条或者 20 条左右的记录检索，自增主键效率是 uuid 主键的 2 到 3 倍；

（2）但是范围查询特别是上百成千条的记录查询，自增 id 的效率要大于 uuid；

（3）在范围查询做统计汇总的时候，自增 id 主键的效率是 uuid 主键 1.5 到 2 倍；

（4）在存储上面，自增 id 所占的存储空间是 uuid 的 1/2；

（5）在写入上面，自增 ID 主键的效率是 UUID 主键的 3 到 10 倍，相差比较明显，特别是 update 小范围之内的数据上面。

（6）在备份恢复上，自增 ID 主键稍微优于 UUID。

#### 自增 ID 主键 + 步长，适合中等规模的分布式场景

在每个集群节点组的 master 上面，设置（`auto_increment_increment`），让目前每个集群的起始点错开 1，步长选择大于将来基本不可能达到的切分集群数，达到将 ID 相对分段的效果来满足全局唯一的效果。

*   优点是：实现简单，后期维护简单，对应用透明。
    
*   缺点是：第一次设置相对较为复杂，因为要针对未来业务的发展而计算好足够的步长;
    

#### UUID，适合小规模的分布式环境

对于 InnoDB 这种聚集主键类型的引擎来说，数据会按照主键进行排序，由于 UUID 的无序性，InnoDB 会产生巨大的 IO 压力，而且由于索引和数据存储在一起，字符串做主键会造成存储空间增大一倍。

在存储和检索的时候，innodb 会对主键进行物理排序，这对`auto_increment_int`是个好消息，因为后一次插入的主键位置总是在最后。

但是对 uuid 来说，这却是个坏消息，因为 uuid 是杂乱无章的，每次插入的主键位置是不确定的，可能在开头，也可能在中间，在进行主键物理排序的时候，势必会造成大量的 IO 操作影响效率，在数据量不停增长的时候，特别是数据量上了千万记录的时候，读写性能下降的非常厉害。

*   优点：搭建比较简单，不需要为主键唯一性的处理。
    
*   缺点：占用两倍的存储空间（在云上光存储一块就要多花 2 倍的钱），后期读写性能下降厉害。
    

### _来源：blog.csdn.net/qq_30108237/article/details/106856051_
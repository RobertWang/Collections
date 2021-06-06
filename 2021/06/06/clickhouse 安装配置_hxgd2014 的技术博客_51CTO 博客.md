> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.51cto.com](https://blog.51cto.com/u_7965676/2678302)

> clickhouse 安装配置，clickhouse 数据库安装与集群配置

© 著作权归作者所有：来自 51CTO 博客作者 hxgd2014 的原创作品，如需转载，请注明出处，否则将追究法律责任  
https://blog.51cto.com/u_7965676/2678302

### 一、介绍

Clickhouse 是一个用于联机分析处理（OLAP）的列式数据库管理系统（columnar DBMS），并允许使用 SQL 查询实时生成分析报告。

### 二、clickhouse 安装

#### 1. 配置源

vim /etc/yum.repos.d/clickhouse.repo

```
[altinity_clickhouse]
name=altinity_clickhouse
baseurl=https://packagecloud.io/altinity/clickhouse/el/7/$basearch
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://packagecloud.io/altinity/clickhouse/gpgkey
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
[altinity_clickhouse-source]
name=altinity_clickhouse-source
baseurl=https://packagecloud.io/altinity/clickhouse/el/7/SRPMS
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://packagecloud.io/altinity/clickhouse/gpgkey
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
```

#### 2. clickhouse 安装

yum makecache  
yum install -y clickhouse-server clickhouse-client

#### 3. 修改配置

vim /etc/clickhouse-server/config.xml

```
#修改log路径
<log>/data/clickhouse/logs/clickhouse-server.log</log>
<errorlog>/data/clickhouse/logs/clickhouse-server.err.log</errorlog>
#http端口
<http_port>8123</http_port>
#tcp端口
<tcp_port>9000</tcp_port>
#副本同步端口
<interserver_http_port>9009</interserver_http_port>
#主机名
<interserver_http_host>clickhouse </interserver_http_host>
#监听地址
<listen_host>::</listen_host>
#数据目录
<path>/data/clickhouse/data</path>
#临时文件路径
<tmp_path>/data/clickhouse/tmp/</tmp_path>
#用户定义文件路径
<user_files_path>/data/clickhouse/user_files/</user_files_path>
<users_config>users.xml</users_config>
```

#### 4. 生成密码

PASSWORD=$(base64 < /dev/urandom | head -c8); echo "$PASSWORD"; echo -n "$PASSWORD" | sha256sum | tr -d '-'

#### 5. 配置用户

vim /etc/clickhouse-server/users.xml

```
<test>
<password_sha256_hex>6de608c69f21b017a6148f31cfae59aa0cc1e5abfc174c752b0c0abfd3274936</password_sha256_hex>
<networks incl="networks" replace="replace">
<ip>::/0</ip>
    </networks>
    <profile>default</profile>
    <quota>default</quota>
    <allow_databases>
      <database>test</database>
    </allow_databases>
</test> 
<readonly>
    <password></password>
    <networks incl="networks" replace="replace">
        <ip>::1</ip>
         <ip>127.0.0.1</ip>
     </networks>
     <profile>readonly</profile>
     <quota>default</quota>
</readonly>
```

#### 6. 启动服务

/etc/init.d/clickhouse-server start

#### 7. 验证

clickhouse-client -utest --password $PASSWORD  
![](https://s4.51cto.com/images/blog/202103/31/1723a786770930225f7ab696a344c8fa.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

### 三、集群配置

#### 1. 修改配置文件

vim /etc/clickhouse-server/config.xml

```
#添加
<include_from>/etc/clickhouse-server/metrika.xml</include_from>
```

#### 2. 集群配置

vim /etc/clickhouse-server/metrika.xml

```
<yandex>
<!--ck集群节点-->
<clickhouse_remote_servers>
    <test_ck_cluster>
        <!--分片1-->
        <shard>
        <weight>1</weight>
            <internal_replication>true</internal_replication>
            <replica>
                <host>172.16.50.238</host>
                <port>9000</port>
                <user>test</user>
                <password>polytest</password>
            </replica>
            <replica>
                <host>172.16.50.239</host>
                <port>9001</port>
                <user>test</user>
                <password>polytest</password>
            </replica>
        </shard>
        <!--分片2-->
        <shard>
        <weight>1</weight>
            <internal_replication>true</internal_replication>
            <replica>
                <host>172.16.50.239</host>
                <port>9000</port>
                <user>test</user>
                <password>polytest</password>
            </replica>
            <replica>
                <host>172.16.50.238</host>
                <port>9001</port>
                <user>test</user>
                <password>polytest</password>
            </replica>
        </shard>
        <!--分片3-->
    </test_ck_cluster>
</clickhouse_remote_servers>

<!--zookeeper相关配置-->
<zookeeper-servers>
    <node index="1">
        <host>172.16.50.125</host>
        <port>2181</port>
    </node>
</zookeeper-servers>

<macros>
    <layer>01</layer>
    <shard>01</shard>
    <replica>cluster01-01-1</replica> <!--当前节点IP-->
</macros>
<!--
<networks>
    <ip>::/0</ip>
</networks>
-->
<!--压缩相关配置-->
<clickhouse_compression>
    <case>
        <min_part_size>10000000000</min_part_size>
        <min_part_size_ratio>0.01</min_part_size_ratio>
        <method>lz4</method> <!--压缩算法lz4压缩比zstd快, 更占磁盘-->
    </case>
</clickhouse_compression>
</yandex>
```

#### 3. 验证

select * from system.clusters;  
![](https://s4.51cto.com/images/blog/202103/31/6b828fd659de9812f288d37b79639ab9.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)
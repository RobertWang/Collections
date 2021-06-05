> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1622874144&ver=3111&signature=lzYiiUMw3g034KrFevttwPD-yK-hDdH6Gbk0Ac4xxug1MfSs9OIBFuwr8etopLIX5TfWbhi8tDLsKmx1v0L8WClSBe6VNlQFri8Gy8z7z69LmicunafAbTwF*h72JDjV&new=1)

最近项目遇到个问题，就是 mysql 做数据分析高并发情况下，老是报超时错误，主要原因还是因为 mysql 进行大批量关联查询太耗时间，于是在网上查询了下资料，决定试一下 clickhouse

一、Clickhouse 简介
===============

Yandex 在 2016 年 6 月 15 日开源了一个数据分析的数据库，名字叫做 ClickHouse，这对保守俄罗斯人来说是个特大事。更让人惊讶的是，这个列式存储数据库的跑分要超过很多流行的商业 MPP 数据库软件，例如 Vertica。如果你没有听过 Vertica，那你一定听过 Michael Stonebraker，2014 年图灵奖的获得者，PostgreSQL 和 Ingres 发明者（Sybase 和 SQL Server 都是继承 Ingres 而来的）, Paradigm4 和 SciDB 的创办者。Michael Stonebraker 于 2005 年创办 Vertica 公司，后来该公司被 HP 收购，HP Vertica 成为 MPP 列式存储商业数据库的高性能代表，Facebook 就购买了 Vertica 数据用于用户行为分析。简单的说，ClickHouse 作为分析型数据库，有三大特点：一是跑分快，二是功能多，三是文艺范 

官网地址：

https://clickhouse.yandex/ 

官方文档：

https://clickhouse.yandex/docs/en/single/ 

二、安装配置
======

clickhouse 只要是 Linux，64 位都可以安装。   
优先支持 Ubuntu，Ubuntu 有官方编译好的安装包可以使用。   
其次是 CentOS 和 RedHat，有第三方组织 altinity 编译好的 rpm 包可以使用。   
如果是其他 Linux 系统，需要自己编译源码。 而且，机器的 CPU 必须支持 SSE 4.2 指令集。 

所以我们安装前先检查一下系统是否支持 SSE 4.2 指令集，我的系统还是一贯的 Centos7.6，我们打开 shell，输入

grep -q sse4_2 /proc/cpuinfo && echo “SSE 4.2 supported” || echo “SSE 4.2 not supported” 回车

![](https://mmbiz.qpic.cn/mmbiz_png/K57A68DuYL3UYXEo593ZIPAkZm8eOEacJEDuIY0CtxM4fLs4NjhLAh0WN46SxyxAb5gpqM6FNs4PC7RovB97Yw/640?wx_fmt=png)

可以看到是支持 SSE 4.2 的

接下来就正式开始安装

1、下载相关最新包, 下载地址：

https://packagecloud.io/altinity/clickhouse

我们下载 4 个文件

clickhouse-client-20.8.3.18-1.el7.x86_64.rpm

clickhouse-common-static-20.8.3.18-1.el7.x86_64.rpm

clickhouse-server-20.8.3.18-1.el7.x86_64.rpm

clickhouse-server-common-20.8.3.18-1.el7.x86_64.rpm

2. 将上述下载的包移至 Centos 服务器目录中并进入该目录

![](https://mmbiz.qpic.cn/mmbiz_png/K57A68DuYL3UYXEo593ZIPAkZm8eOEacaveneaQ15anZ9r8VZYYKvoe9CIAqsZKvhhViaG3KC7IiarjjLoTdgR7A/640?wx_fmt=png)

上传后

![](https://mmbiz.qpic.cn/mmbiz_png/K57A68DuYL3UYXEo593ZIPAkZm8eOEaciak09X8Sj3JBPnpHwQAx5dcdklnKvWCRl433kW3kLCicVSU8zNiaaTx6Q/640?wx_fmt=png)

3、我们按照下面的文件顺序执行安装

rpm -ivh clickhouse-common-static-20.8.3.18-1.el7.x86_64.rpm

rpm -ivh clickhouse-server-common-20.8.3.18-1.el7.x86_64.rpm

rpm -ivh clickhouse-server-20.8.3.18-1.el7.x86_64.rpm

rpm -ivh clickhouse-client-20.8.3.18-1.el7.x86_64.rpm

![](https://mmbiz.qpic.cn/mmbiz_png/K57A68DuYL3UYXEo593ZIPAkZm8eOEacajpJa6ePVaYLSXP0rPxJA78FYrHW4zwMJc9HEkTMiczEoKkKpJqN0gg/640?wx_fmt=png)

4、启动服务 执行 service clickhouse-server start

![](https://mmbiz.qpic.cn/mmbiz_png/K57A68DuYL3UYXEo593ZIPAkZm8eOEacUxwuQpWDfhFBKvmyzZJfWq3y9jHOLn4Z7vC3hfSLn0k4tXoqHdE1tQ/640?wx_fmt=png)

5、进入客户端

执行 clickhouse-client

![](https://mmbiz.qpic.cn/mmbiz_png/K57A68DuYL3UYXEo593ZIPAkZm8eOEac4cyvM6rwqdJxibdzft2OoDYicribzOG4WwpQ4IV3z7riciabMA9aJXmibtdw/640?wx_fmt=png)

可以看到我们已经可以使用 ck 了 打印出了版本号

三、使用 dbeaver 工具连接 ck，没使用过的请看我的使用教程 大数据分析学习第八课 数据仓库 Hive 配置客户端可视化管理工具 DBeaver

我们新建数据库连接，搜索选择 clickhouse

![](https://mmbiz.qpic.cn/mmbiz_png/K57A68DuYL3UYXEo593ZIPAkZm8eOEaco1RRRg1lLbhsyyFeejTC7PibLKxLPKb0BMyXicWKfKibiccTGzibWD9FvYg/640?wx_fmt=png)

下一步

![](https://mmbiz.qpic.cn/mmbiz_png/K57A68DuYL3UYXEo593ZIPAkZm8eOEacaNJ4upy8zTNgBV3iaDtwSDslkOCZ4LTgSoibKOZ4ib4ic5obbQlnyPbIhQ/640?wx_fmt=png)

我们填写主机地址 slave107，点确定后，dbeaver 会检查需要的驱动并开始下载驱动

![](https://mmbiz.qpic.cn/mmbiz_png/K57A68DuYL3UYXEo593ZIPAkZm8eOEacBdW3gDMibeooKojuGU7uXrq5jKy63ccQLuMcOXiaPGKUYJBMZcRiaxXqA/640?wx_fmt=png)

驱动安装后，我们遇到下面的报错

![](https://mmbiz.qpic.cn/mmbiz_png/K57A68DuYL3UYXEo593ZIPAkZm8eOEacQziaZKLt065CKib6uSeS9sTwt6xcYicBk4pwt4gNTibAibHkxXzJuTl51ZQ/640?wx_fmt=png)

应该是我们没有启用外网访问

我们进到 server 的配置文件 config.xml

![](https://mmbiz.qpic.cn/mmbiz_png/K57A68DuYL3UYXEo593ZIPAkZm8eOEacw5dHSQicUuKGMFOJv417BtQsQyBfFtOib612bGZmMOgEYNHkgCRicBJsQ/640?wx_fmt=png)

我们找到 <listen_host>::</listen_host > 这行  取消注释

![](https://mmbiz.qpic.cn/mmbiz_png/K57A68DuYL3UYXEo593ZIPAkZm8eOEac18WXxEgWW1WM7icGUKdvDic1hsibpZC9xwsuKMhXEGq51IC6CH4kZ3oDQ/640?wx_fmt=png)

接下来检查配置 users.xml

vi users.xml

主要检查下面节点如下正常即可

```
<networks>
    <ip>::/0</ip>
</networks>
```

接下来我们需要重启一下 ck 服务使配置生效

```
sudo service clickhouse-server stop #停止
sudo service clickhouse-server start #启动
```

![](https://mmbiz.qpic.cn/mmbiz_png/K57A68DuYL3UYXEo593ZIPAkZm8eOEacgq55AKEk2juSJLPM3zEIua3QjwqCvUeGtyHv8yIevxdt0ZaZibgKduA/640?wx_fmt=png)

重启后 我们先用 client 连接测试一下

执行 **clickhouse-client --port 9000 --host 127.0.0.1 --multiline**

![](https://mmbiz.qpic.cn/mmbiz_png/K57A68DuYL3UYXEo593ZIPAkZm8eOEach15JxLWMvlxAnsrpW1lPAYic45E6W9VkIiaia9x4wbBEJkd3GG1w1Kn1Q/640?wx_fmt=png)

**测试正常 接下来我们继续使用 dbeaver 连接，这里注意我们使用的端口不能是 9000 了，就用默认的 8123**

![](https://mmbiz.qpic.cn/mmbiz_png/K57A68DuYL3UYXEo593ZIPAkZm8eOEaclu3OBicHrDJ4he79icThAWt1Sdmvjib6AGSmWqopldgXdAEicM6pml2d1g/640?wx_fmt=png)

**可以看到连接正常了，左侧数据库连接展开，我们可以看到 ck 默认有 2 个库，一个 default，里面什么都没有，一个 system，是内置的一些系统表**

![](https://mmbiz.qpic.cn/mmbiz_png/K57A68DuYL3UYXEo593ZIPAkZm8eOEacvHfVNtrMiaIGSTLAmibev1aRW9ia73oP0Nw8CAmNico8iab0S8O3pZHnicnQ/640?wx_fmt=png)

**我们在 default 库新建一个 test 表**

![](https://mmbiz.qpic.cn/mmbiz_png/K57A68DuYL3UYXEo593ZIPAkZm8eOEacvVIDMRqkFjUt8ZmcU9W7c9Pnnr86RBkE07ef6rDhZHW7WiaUfe9xuWA/640?wx_fmt=png)

**保存会弹出一个界面，里面有建表 sql**

![](https://mmbiz.qpic.cn/mmbiz_png/K57A68DuYL3UYXEo593ZIPAkZm8eOEacCzEX1PwhuTYuaMz3397v0Ycy5K13NWSU8HibtGAKMp2pYQp4yeQAYPQ/640?wx_fmt=png)

**我们点击执行，表就建好了，我们插入 4 条测试数据**

![](https://mmbiz.qpic.cn/mmbiz_png/K57A68DuYL3UYXEo593ZIPAkZm8eOEacYexbSIIFvH4icxXic8Dicib3leegIp8LBBJ4oV6xo0HBuUmxhyArMwsibVw/640?wx_fmt=png)

接下来我们统计每个人名的个数，可以自己手工输入 sql，也可以右键表名 => 生成 sql

![](https://mmbiz.qpic.cn/mmbiz_png/K57A68DuYL3UYXEo593ZIPAkZm8eOEacIB0Hic2LxAGI2HqFmxcSfeZ0QEHV1IF2FApxGCVuXahicDGhkgH28Z7w/640?wx_fmt=png)
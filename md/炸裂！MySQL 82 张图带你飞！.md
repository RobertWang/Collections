> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NTY1MjY0MQ==&mid=2650786929&idx=5&sn=2aa9040f6e6772c53d5958b58dc08852&chksm=befe593f8989d029a3777726955c9636ecfae22934c2aa4a6cadeb9dec2ec470ec17b331e9f5&scene=21#wechat_redirect)

作者 | cxuan

来源 | Java 建设者（ID：javajianshe）

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVhcfHK3fA9lEU3e4M8wl0HjarHcv8JicmCDbZQtSZbCt49LQxyPCTA0w/640?wx_fmt=png)

之前两篇文章带你了解了 MySQL 的基础语法和 MySQL 的进阶内容，今天这篇文章我们来了解一下 MySQL 中的高级内容。

事务控制和锁定语句
---------

我们知道，MyISAM 和 MEMORY 存储引擎支持`表级锁定(table-level locking)`，InnoDB 存储引擎支持`行级锁定(row-level locking)`，BDB 存储引擎支持`页级锁定(page-level locking)`。各个锁定级别的特点如下

页级锁：销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般

表级锁：表级锁是对整张表进行加锁，MyISAM 和 MEMORY 主要支持表级锁，表级锁加锁快，不会出现死锁，锁的粒度比较粗，并发度最低

行级锁：行级锁可以说是 MySQL 中粒度最细的一种锁了，InnoDB 支持行级锁，行级锁容易发生死锁，并发度比较好，同时锁的开销也比较大。

MySQL 默认情况下支持表级锁定和行级锁定。但是在某些情况下需要手动控制事务以确保整个事务的完整性，下面我们就来探讨一下事务控制。但是在探讨事务控制之前我们先来认识一下两个锁定语句

### 锁定语句

MySQL 的锁定语句主要有两个 `Lock` 和 `unLock`，Lock Tables 可用于锁定当前线程的表，就跟 Java 语法中的 Lock 锁的用法是一样的，如果表锁定，意味着其他线程不能再操作表，直到锁定被释放为止。如下图所示

```
lock table cxuan005 read;
```

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVzxGyFFUCK2sXFfGicibVLq2dI384WicjFcClEOWfIyibF3DRAyjMdHrOJQ/640?wx_fmt=png)

我们锁定了 cxuan005 的 read 锁，然后这时我们再进行一次查询，看看是否能够执行这条语句

```
select * from cxuan005 where id = 111;
```

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVY7qMwN7AJwIJ2tQxR3uaVRj2tV50yZth9zhI5w4Zuswhlu4FRAB86Q/640?wx_fmt=png)

可以看到，在进行 read 锁定了，我们仍旧能够执行查询语句。

现在我们另外起一个窗口，相当于另起了一个线程来进行查询操作。

```
select * from cxuan005;
```

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVsxCwicibFC3eRqOLw67qp72icpF9Y3ic0al1ceBdjDzqvbcLjPZmSywdVA/640?wx_fmt=png)

这是第二个窗口执行查询的结果，可以看到，在一个线程执行 read 锁定后，其他线程仍然可以进行表的查询操作。

那么第二个线程能否执行更新操作呢？我们来看一下

```
update cxuan005 set info='cxuan' where id = 111;
```

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVq5QoL4MbnMFzMkZwNwuiakfHRQI5GoI61ZSV4ccN4cLomKtgb99AODA/640?wx_fmt=png)

发生了什么？怎么没有提示结果呢？其实这个情况下表示 cxuan005 已经被加上了 read 锁，由于当前线程不是持有锁的线程，所以当前线程无法执行更新。

### 解锁语句

现在我们把窗口切换成持有 read 锁的线程，来进行 read 锁的解锁

```
unlock tables;
```

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVEFrmribIiayD3Hgn5TibJP2ibiceXibIGdMGWKIlCx733eZCwqZmkUIAicaZw/640?wx_fmt=png)

在解锁完成前，进行更新的线程会一直等待，直到解锁完成后，才会进行更新。我们可以看一下更新线程的结果。

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVWic9z6xhyqHQ9RmAjxeXCE1693AGN8BAbJPV4xU5pykdMxEL3riaU7rw/640?wx_fmt=png)

可以看到，线程已经更新完毕，我们看一下更新的结果

```
select * from cxuan005 where id = 111;
```

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVtAibxibibGVb7a93pk1mHbuMcYduarIaQb2kjUs1TmE0cUBznX8XYCiaqw/640?wx_fmt=png)

如上图所示，id = 111 的值已经被更新成了 cxuan。

事务控制
----

`事务(Transaction)` 是访问和更新数据库的基本执行单元，一个事务中可能会包含多个 SQL 语句，事务中的这些 SQL 语句要么都执行，要么都不执行，而 MySQL 它是一个关系型数据库，它自然也是支持事务的。事务同时也是区分关系型数据库和非关系型数据库的一个重要的方面。

在 MySQL 事务中，主要涉及的语法包含 **SET AUTOCOMMIT、START TRANSACTION、COMMIT 和 ROLLBACK** 等。

### 自动提交

在 MySQL 中，事务默认是`自动提交(Autocommit)`的，如下所示

```
show variables like 'autocommit';
```

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVuQg5jcKBSF2ibsYvGlS4LuljeCib9VCVC6QVwJOfzpA1FvS0Lq3Iics2Q/640?wx_fmt=png)

在自动提交的模式下，每个 SQL 语句都会当作一个事务执行提交操作，例如我们上面使用的更新语句

```
update cxuan005 set info='cxuan' where id = 111;
```

> 如果想要关闭数据库的自动提交应该怎么做呢？

其实，MySQL 是可以关闭自动提交的，你可以执行

```
set autocommit = 0;
```

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVAhISFiab7t8GvpBLwvXkiaRb7vd46JEXY8X0LeUOciaiaph3ywCoBWTHKQ/640?wx_fmt=png)

然后我们再看一下自动提交是否关闭了，再次执行一下 show variables like 'autocommit' 语句

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVMJ2OMEN0BQYP46aCcVH3e6CvicUPx7akZf54aibPSvU2LRIrZkqDu4Eg/640?wx_fmt=png)

可以看到，自动提交已经关闭了，再次执行

```
set autocommit = 1;
```

会再次开启自动提交。

> 这里注意一下特殊操作。
> 
> 在 MySQL 中，存在一些特殊的命令，如果在事务中执行了这些命令，会马上强制执行 commit 提交事务；比如 DDL 语句 (create table/drop table/alter/table)、lock tables 语句等等。
> 
> 不过，常用的 select、insert、update 和 delete 命令，都不会强制提交事务。

### 手动提交

如果需要手动 commit 和 rollback 的话，就需要明确的事务控制语句了。

典型的 MySQL 事务操作如下

```
start transaction;
... # 一条或者多条语句
commit;
```

上面代码中的 start transaction 就是事务的开始语句，编写 SQL 后会调用 commit 提交事务，然后将事务统一执行，如果 SQL 语句出现错误会自动调用 Rollback 进行回滚。

下面我们就通过示例来演示一下 MySQL 的事务，同样的，我们需要启动两个窗口来演示，为了便于区分，我们使用 mysql01 和 mysql02 来命名。

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVGK3QNfYjyl52c5ficB8aSglPAicWyCgICXpibPd5zwawlicHbXWozjXJHA/640?wx_fmt=png)

我们用 `start transaction` 命令启动一个事务，然后在 cxuan005 表中插入一条数据，此时 mysql02 不做任何操作。涉及的 SQL 语句如下。

```
start transaction;
```

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVd0p33UicYIOJ6vRhFEp2iaMlbMQmYMmc7CuLyibibNy85yAiceiatMYqDd6Q/640?wx_fmt=png)

然后执行

```
select * from cxuan005;
```

查询一下 cxuan005 中的数据

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVNPVZYRNqAPQEtTqMo0WfP03LBgcEDaGSWSUR7KI5kFvfWz8zHDXuPg/640?wx_fmt=png)

嗯。。。很多长度太长了，现在我们把所有的 info 数据都更新为 cxuan 。

```
update cxuan005 set info='cxuan';
```

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVvJkwJmAXw7YH0AvWEnJQYbLZono4oRAmI1cYUe4OaEgIP8S9PtiadcQ/640?wx_fmt=png)

更新完毕后，我们先不提交事务，分别在 mysql01 和 mysql02 中进行查询，发现只有 mysql01 窗口中的查询已经生效，而 mysql02 中还是更新前的数据

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVa8WNaVezgpUFHHH06emzXrdActI7E63R81ZntWKnyqPTWoR0Oz4iaRA/640?wx_fmt=png)

现在我们在 mysql01 中 commit 当前事务，然后在 mysql02 中查询，发现数据已经被修改了。

除了 commit 之外，MySQL 中还有 `commit and chain` 命令，这个命令会提交当前事务并且重新开启一个新的事务。如下代码所示

```
start transaction; # 开启一个新的事务
insert into cxuan005(id,info) values (555,'cxuan005'); # 插入一条数据
commit and chain; # 提交当前事务并重新开启一个事务
```

上面是一个事务操作，在 commit and chain 键入后，我们可以再次执行 SQL 语句

```
update cxuan005 set info = 'cxuan' where id = 555;
commit;
```

然后再次查询

```
select * from cxuan005;
```

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVZBv5LFTJg07D0uqoJgyxosopglTgo38sd4nJbXagzVqSomXGAJh1CA/640?wx_fmt=png)

执行后，可以发现，我们仅仅使用了一个 start transaction 命令就执行了两次事务操作。

如果在手动提交的事务中，你发现有一条 SQL 语句写的不正确或者有其他原因需要回滚，那么此时你就会用到 `rollback` 语句，它会回滚当前事务，相当于什么也没发生。如下代码所示。

```
start transaction;
delete from cxuan005 where id = 555;
rollback;
```

> 这里`切忌`一点：delete 删除语句一定要加 where ，不加 where 语句的删除就是耍流氓。

在同一个事务操作中，最好使用相同存储引擎的表，如果使用不同存储引擎的表后，rollback 语句会对非事务类型的表进行特别处理，因此 commit 、rollback 只能对事务类型的表进行提交和回滚。

我们提交的事务一般都会被记录到二进制的日志中，但是如果一个事务中包含非事务类型的表，那么回滚操作也会被记录到二进制日志中，以确保非事务类型的表可以被复制到从数据库中。

这里解释一下什么是事务表和非事务表

#### 事务表和非事务表

事务表故名思义就是支持事务的表，支不支持事务和 MySQL 的存储类型有关，一般情况下，`InnoDB` 存储引擎的表是支持事务的，关于 InnoDB 的知识，我们会在后面详细介绍。

非事务表相应的就是不支持事务的表，在 MySQL 中，存储引擎 `MyISAM` 是不支持事务的，非事务表的特点是不支持回滚。

对于回滚的话，还要讲一点就是 `SAVEPOINT`，它能指定事务回滚的一部分，但是不能指定事务提交的一部分。SAVEPOINT 可以指定多个，在满足不同条件的同时，回滚不同的 SAVEPOINT。需要注意的是，如果定义了两个相同名称的 SAVEPOINT，则后面定义的 SAVEPOINT 会覆盖之前的定义。如果 SAVEPOINT 不再需要的话，可以通过 `RELEASE SAVEPOINT` 来进行删除。删除后的 SAVEPOINT 不能再执行 ROLLBACK TO SAVEPOINT 命令。

我们通过一个示例来进行模拟不同的 SAVEPOINT

首先先启动一个事务 ，向 cxuan005 中插入一条数据，然后进行查询，那么是可以查询到这条记录的

```
start transaction;
insert into cxuan005(id,info) values(666,'cxuan666');
select * from cxuan005 where id = 666;
```

查询之后的记录如下

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVuuxiaEHRAV7OM9ofDvlr6hcPaEDAehmtpJg3icVhDqRhKbO0ol05SAug/640?wx_fmt=png)

然后我们定义一个 SAVEPOINT，如下所示

```
savepoint test;
```

然后继续插入一条记录

```
insert into cxuan005(id,info) values(777,'cxuan777');
```

此时就可以查询到两条新增记录了，id 是 666 和 777 的记录。

```
select * from cxuan005 where id = 777;
```

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kV4eT00Fw6AU2G4oz4C955Foyg3pYv5MI63JgQ73ibhZc5LjoAOCHt3rQ/640?wx_fmt=png)

那么我们可以回滚到刚刚定义的 SAVEPOINT

```
rollback to savepoint test;
```

再次查询 cxuan005 这个表，可以看到，只有 id=666 的这条记录插入进来了，说明 id=777 这条记录已经被回滚了。

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVlswwwhowKGOiaXSw6nNBqpic1OsiaARY7BLnSVP6WomuPT5Spxh4yx0jA/640?wx_fmt=png)

此时我们看到的都是 mysql01 中事务还没有提交前的状态，所以这时候 mysql02 中执行查询操作是看不到 666 这条记录的。

然后我们在 mysql01 中执行 commit 操作，那么此时在 mysql02 中就可以查询到这条记录了。

SQL 安全问题
--------

SQL 安全问题应该是我们程序员比较忽视的一个地方了。日常开发中，我们一般只会关心 SQL 能不能解决我们的业务问题，能不能把数据查出来，而对于 SQL 问题，我们一般都认为这是 DBA 的活，其实我们 CRUD 程序员也应该了解一下 SQL 的安全问题。

### SQL 注入简介

SQL 注入就是利用某些数据库的外部接口将用户数据插入到实际的 SQL 中，从而达到入侵`数据库`的目的。SQL 注入是一种常见的网络攻击的方式，它不是利用操作系统的 BUG 来实现攻击的。SQL 主要是针对程序员编写时的疏忽来入侵的。

SQL 注入攻击有很大的危害，攻击者可以利用它读取、修改或者删除数据库内的数据，获取数据库中的用户名和密码，甚至获得数据库管理员的权限。并且 SQL 注入一般比较难以防范。

SQL Mode
--------

MySQL 可以运行在不同的 SQL Mode 模式下，不同的 SQL Mode 定义了不同的 SQL 语法，数据校验规则，这样就能够在不同的环境中使用 MySQL ，下面我们就来介绍一下 SQL Mode。

### SQL Mode 解决问题

SQL Mode 可以解决下面这几种问题

*   通过设置 SQL Mode，可以完成不同严格程度的数据校验，有效保障数据的准确性。
    
*   设置 SQL Mode 为 `ANSI` 模式，来保证大多数 SQL 符合标准的 SQL 语法，这样应用在不同数据库的迁移中，不需要对 SQL 进行较大的改变
    
*   数据在不同数据库的迁移中，通过改变 SQL Mode 能够更方便的进行迁移。
    

下面我们就通过示例来演示一下 SQL Mode 用法

我们可以通过

```
select @@sql_mode;
```

来查看默认的 SQL Mode，如下是我的数据库所支持的 SQL Mode

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVhp8DV55HuPvBXFYqL2HCdIeTmCzQsxdiaH5FOvUuFk7XnIatmXjoo8w/640?wx_fmt=png)

涉及到很多 SQL Mode，下面是这些 SQL Mode 的解释

`ONLY_FULL_GROUP_BY`：这个模式会对 GROUP BY 进行合法性检查，对于 GROUP BY 操作，如果在 SELECT 中的列，没有在 GROUP BY 中出现，那么将认为这个 SQL 是不合法的，因为列不在 GROUP BY 从句中

同样举个例子，我们现在查询一下 cxuan005 的 id 和 info 字段。

```
select id,info from cxuan005;
```

这样是可以运行的

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kV9ortS13hNXicMnO5o3fWDfrHHDRkU44PicpKlHPRLbNjUWdSicuzjp6aA/640?wx_fmt=png)

然后我们使用 GROUP BY 字句进行分组，这里只对 info 进行分组，我们看一下会出现什么情况

```
select id,info from cxuan005 group by info;
```

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kV7xHicHP8K2LicAI6p8FxzU5XJattnibCk0zv79nXuI2oIBiapn0rZEVldw/640?wx_fmt=png)

我们可以从错误原因中看到，这条 SQL 语句是不符合 ONLY_FULL_GROUP_BY 的这条 SQL Mode 的。因为我们只对 info 进行分组了，没有对 id 进行分组，我们把 SQL 语句改成如下形式

```
select id,info from cxuan005 group by id,info;
```

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVKRymLznQ4A3TemC9jUKmQZ75JtWia1gttx2yZ1Q0a6ENdjQfvvAVsPQ/640?wx_fmt=png)

这样 SQL 就能正确执行了。

当然，我们也可以删除 sql_mode = ONLY_FULL_GROUP_BY 的这条 Mode，可以使用

```
SET sql_mode=(SELECT REPLACE(@@sql_mode,'ONLY_FULL_GROUP_BY',''));
```

来进行删除，删除后我们使用分组语句就可以放飞自我了。

```
select id,info from cxuan005 group by info;
```

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVBF0icBPmQlqCz4MuBR8ics7EalEtgOXImgxlkndOHgtscRGULRdCJhWQ/640?wx_fmt=png)

但是这种做法只是暂时的修改，我们可以修改配置文件 my.ini 中的 sql_mode= STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

`STRICT_TRANS_TABLES`：这就是严格模式，在这个模式下会对数据进行严格的校验，错误数据不能插入，报 error 错误。如果不能将给定的值插入到事务表中，则放弃该语句。对于非事务表，如果值出现在单行语句或多行语句的第 1 行，则放弃该语句。

> 当使用 innodb 存储引擎表时，考虑使用 innodb_strict_mode 模式的 sql_mode，它能增量额外的错误检测功能。

`NO_ZERO_IN_DATE`：这个模式影响着日期中的月份和天数是否可以为 0（注意年份是非 0 的），这个模式也取决于严格模式是否被启用。如果这个模式未启用，那么日期中的零部分被允许并且插入没有警告。如果这个模式启用，那么日期中的零部分插入被作为 `0000-00-00` 并且产生一个警告。

这个模式需要注意下，如果启用的话，需要 `STRICT_TRANS_TABLES` 和 `NO_ZERO_IN_DATE` 同时启用，否则不起作用，也就是

```
set session sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE';
```

然后我们换表了，使用 cxuan003 这张表，表结构如下

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVPtR6LR1AjHB2RD0Q3iaQPAWWicEXib7GF1YA0DNPDsTbcjtLS8drz4UEw/640?wx_fmt=png)

我们主要测试日期的使用，在 cxuan003 中插入一条日期为 `0000-00-00` 的数据

```
insert into cxuan003 values(111,'study','0000-00-00');
```

发现能够执行成功，但是把年月日各自变为 0 之后再进行插入，则会插入失败。

```
insert into cxuan003 values(111,'study','2021-00-00');
```

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVapgdgic1sbtd4RmwEKR4GB4Lgu450PXqs4j3ZFyNSplmCWSj9ZqBCIw/640?wx_fmt=png)

```
insert into cxuan003 values(111,'study','2021-01-00');
```

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVO7bkS5QlgfHSqwjB8ibUZOUyN0SBgo6RduyzE6f9LGmA0hmwsCDKSqw/640?wx_fmt=png)

这些组合有很多，我这里就不再细致演示了，读者可以自行测试。

如果要插入 `0000-00-00` 这样的数据，必须设置 `NO_ZERO_IN_DATE` 和 `NO_ZERO_DATE`。

`ERROR_FOR_DIVISION_BY_ZERO`：如果这个模式未启用，那么零除操作将会插入空值并且不会产生警告；如果这个模式启用，零除操作插入空值并产生警告；如果这个模式和严格模式都启用，零除从操作将会产生一个错误。

`NO_AUTO_CREATE_USER`：禁止使用 grant 语句自动创建用户，除非认证信息被指定。

`NO_ENGINE_SUBSTITUTION`：此模式指定当执行 create 语句或者 alter 语句指定的存储引擎没有启用或者没有编译时，控制默认存储引擎的自动切换。默认是启用状态的。

### SQL Mode 三种作用域

SQL Mode 按作用区域和时间可分为 3。个级别，分别是**会话级别，全局级别，配置（永久生效）级别**。

我们上面使用的 SQL Mode 都是 `会话级别`，会话级别就是当前窗口域有效。它的设置方式是

```
set @@session.sql_mode='xx_mode'
set session sql_mode='xx_mode'
```

全局域就是当前会话关闭不失效，但是在 MySQL 重启后失效。它的设置方式是

```
set global sql_mode='xx_mode';
set @@global.sql_mode='xx_mode';
```

配置域就是在 `vi /etc/my.cnf` 里面添加

```
[mysqld]
sql-mode = "xx_mode"
```

配置域在保存退出后，重启服务器，即可永久生效。

SQL 正则表达式
---------

正则表达式相信大家应该都用过，不过你在 MySQL 中用过正则表达式吗？下面我们就来聊一聊 SQL 中的正则表达式。

`正则表达式(Regular Expression)` 是指一个用来描述或者匹配字符串的句法规则。正则表达式通常用来检索和替换某个文本中的文本内容。很多语言都支持正则表达式，MySQL 同样也不例外，MySQL 利用 `REGEXP` 命令提供给用户扩展的正则表达式功能。下面是 MySQL 中正则表达式的一些规则。

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVR9IGZZ38mr6aZCqdcW6zRV9qNnOv2Ksm4yREZuqUSXk2uDVjvTaW7g/640?wx_fmt=png)

下面来演示一下正则表达式的用法

*   `^` 在字符串的开始进行匹配，根据返回的结果来判断是否匹配，1 = 匹配，0 = 不匹配。下面尝试匹配字符串 `aaaabbbccc` 是否以字符串 `a` 为开始
    
    ```
    select 'aaaabbbccc' regexp '^a';
    ```
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVSZdkbpzbLiaa2h9XG24rrLDB76eCXm9nCDiangWBV4U0B4kaBKhYJGtw/640?wx_fmt=png)
    
*   同样的，`$` 会在末尾处进行匹配，如下所示
    
    ```
    select 'aaaabbbccc' regexp 'c$';
    ```
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVQT3TpibAjfj0zCLzScL1tzx2ib4tHHIriciasMnFsyuyvZswLZB7GqiaOOw/640?wx_fmt=png)

*   `.` 匹配单个任意字符
    
    ```
    select 'berska' regexp '.s', 'zara' regexp '.a';
    ```
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVOwz6QOSLsTvXA3VdsYbrZjMa3knUfp9SJib9bwc8W7JAh3BgxPJOe7w/640?wx_fmt=png)
    
*   `[...]` 表示匹配括号内的任意字符，示例如下
    
    ```
    select 'whosyourdaddy' regexp '[abc]';
    ```
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVjspFcicqN1LJwUhbW7nqr2wQkHeBHegT4iceZFXXGPWQ8DNmqtNFkVag/640?wx_fmt=png)
    
*   `[^...]` 匹配括号内不包含的任意字符，和 `[...]` 是相反的，如果有任何匹配不上，返回 0 ，全部匹配上返回 1。
    
    ```
    select 'x' regexp '[^xyz]';
    ```
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVk9iaNsFHpC4jIJ65xduwmxj82vYhG0FqvYE9Q5GY7DRQ94mmuSQOufA/640?wx_fmt=png)
    
*   `n*` 表示匹配零个或者多个 n 字符串，如下
    
    ```
    select 'aabbcc' regexp 'd*';
    ```
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVDRh0oug2suKbPia5zbRZOJtgtd5rhXr0fwWibwFjeyjbWqKdsxaMBoow/640?wx_fmt=png)  
    
    没有 d 出现也可以返回 1 ，因为 * 表示 0 或者多个。
    
*   `n+` 表示匹配 1 个或者 n 个字符串
    
    ```
    select 'aabbcc' regexp 'd+';
    ```
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVqoBictyfRxXQqV3zfgQPDQYOUcfd7iaSD6W2WIxP2pSaoutHNM3yexTg/640?wx_fmt=png)
    
*   `n?` 的用法和 n+ 类似，只不过 n? 可以匹配空串
    

常见 SQL 技巧
---------

### RAND() 函数

大多数数据库都会提供产生随机数的函数，通过这些函数可以产生随机数，也可以使用从数据库表中抽取随机产生的记录，这对统计分析来说很有用。

在 MySQL 中，通常使用 `RAND()` 函数来产生随机数。RAND() 和 ORDER BY 组合完成数据抽取功能，如下所示。

我们新建一张表用于数据检索。

```
CREATE TABLE `clerk_Info` (
  `id` int(11) NOT NULL,
  `name` varchar(255) DEFAULT NULL,
  `salary` decimal(10,2) DEFAULT NULL,
  `companyId` int(10) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1
```

然后插入一些数据，插入完成后的数据如下。

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVBChshE01QSWhR5eYWnNuNF6wfgaJGebicR4Lttiad5vMy53WGPpicapBQ/640?wx_fmt=png)

然后我们可以使用 RAND() 函数进行随机检索数据行

```
select * from clerk_info order by rand();
```

检索完成后的数据如下

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVauQgctz6T4KSCh8wfSiaJrrwkRPBYx6EcMuNZonmibjuONtIozXQteMg/640?wx_fmt=png)

多次查询后发现每次检索的数据顺序都是随机的。

这个函数多用于随机抽样，比如选取一定数量的样本在进行随机排序，需要用到 `limit` 关键字。

### GROUP BY + WITH ROLLUP

我们经常使用 GROUP BY 语句，但是你用过 `GROUP BY` 和 `WITH ROLLUP` 一起使用的吗？使用 GROUP BY 和 WITH ROLLUP 字句可以检索出更多的分组集合信息。

我们仍旧对 clerk_info 表进行操作，我们对 name 和 salary 进行分组统计工资总数。

```
select name,sum(salary) from clerk_info group by name with rollup;
```

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVTC9wMZlMZXTMshdQDlwr3iadpgmhlOSNicMJqTKYysqO5x5COticSPozw/640?wx_fmt=png)

可以看到上面的表按照 name 进行分组，然后再对 money 进行统计。

也就是说 GROUP BY 语句执行完成后可以满足用户想要的任何一个分组以及分组组合的聚合信息值。

> 这里需要注意一点，不能同时使用 ORDER BY 字句对结果进行排序，ROLLUP 和 ORDER BY 是互斥的。

### 数据库名、表名大小写问题

在 MySQL 中，**数据库中的每个表至少对应数据库目录中的一个文件，当然这取决于存储引擎的实现了**。不同的操作系统对大小写的敏感性决定了数据库和表名的大小写的敏感性。在 UNIX 操作系统中是对大小写敏感的，因此数据库名和表名也是具有敏感性的，而到了 Windows 则不存在敏感性问题，因为 Windows 操作系统本身对大小写不敏感。**列、索引、触发器**在任何平台上都对大小写不敏感。

在 MySQL 中，数据库名和表名是由 `lower_case_tables_name` 系统变量决定的。可以在启动 `mysqld` 时设置这个系统变量。下面是 `lower_case_tables_name` 的值。

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVb1ybGrYV0xMVQoRtwT5v1HT5cJ24ISenZYXM26tpBqjYu87oUmoc1w/640?wx_fmt=png)

如果只在一个平台上使用 MySQL 的话，通常不需要修改 `lower_case_tables_name` 变量。如果想要在不同系统系统之间迁移表就会涉及到大小写问题，因为 UNIX 中 clerk_info 和 CLERK_INFO 被认为是两个不同的表，而 Windows 中则认为是一个。在 UNIX 中使用 lower_case_tables_name=0， 而在 Windows 中使用 lower_case_tables_name=2，这样可以保留数据库名和表名的大小写，但是不能保证所有的 SQL 查询中使用的表名和数据库名的大小写相同。如果 SQL 语句中没有正确引用数据库名和表名的大小写，那么虽然在 Windows 中能正确执行，但是如果将查询转移到 UNIX 中，大小写不正确，将会导致查询失败。

### 外键问题

这里需要注意一个问题，`InnoDB` 存储引擎是支持外键的，而 `MyISAM` 存储引擎是不支持外键的，因此在 MyISAM 中设置外键会不起作用。

MySQL 常用函数
----------

下面我们来了解一下 MySQL 函数，MySQL 函数也是我们日常开发过程中经常使用的，选用合适的函数能够提高我们的开发效率，下面我们就来一起认识一下这些函数

### 字符串函数

字符串函数是最常用的一种函数了，MySQL 也是支持很多种字符串函数，下面是 MySQL 支持的字符串函数表

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVdejTAicba8MYfPcwjcOebnkzfpOMKicCzqvTicUbddS7Kf6jRbuJTBmlg/640?wx_fmt=png)

下面通过具体的示例演示一下每个函数的用法

*   LOWER(str) 和 UPPER(str) 函数：用于转换大小写
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVicWqOEpiaicJgiclnS3sdEQHB13cAzLPaa7BOoS6icccdDiaJjjH251ehUibw/640?wx_fmt=png)

*   CONCAT(s1,s2 ... sn) ：把传入的参数拼接成一个字符串
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kV3EGuc9YTSM1CHUhssHicicELZXYFIAFppNia8iao0IssicQ0YQSS9DIB5YQ/640?wx_fmt=png)

上面把 `c xu an` 拼接成为了一个字符串，另外需要注意一点，任何和 NULL 进行字符串拼接的结果都是 NULL。

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVnjVfBorzqelFM1XYcpaMc6wkde63nLYp4XL2lMpEiaadTZANzibC1t2g/640?wx_fmt=png)

*   LEFT(str,x) 和 RIGHT(str,x) 函数：分别返回字符串最左边的 x 个字符和最右边的 x 个字符。如果第二个参数是 NULL，那么将不会返回任何字符串
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVyz4BXMdPU0cUIYqVLibautsyJe6icI5y6qsP6RxCxq9O82sWtRFdiaSqA/640?wx_fmt=png)

*   INSERT(str,x,y,instr) ：将字符串 str 从指定 x 的位置开始， 取 y 个长度的字串替换为 instr。
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVJkY1XsHU9m11Jr8e46RoeiaqXDpDb355ic8ka5uiaaDLrlICnf1ibBPuvg/640?wx_fmt=png)

*   LTRIM(str) 和 RTRIM(str) 分别表示去掉字符串 str 左侧和右侧的空格
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVcgTweBjpLjSAUJfCkibqVnvwiaUcZd2A3SPT4RdrBQYfQ8xT0SSkPvSA/640?wx_fmt=png)

*   REPEAT(str,x) 函数：返回 str 重复 x 次的结果
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVPS3qtNV93FRw1GjIafuqBronSPJdovsU5SgfJc8ElY9JzibqVSIsf2w/640?wx_fmt=png)

*   TRIM(str) 函数：用于去掉目标字符串的空格
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVwMMXSJQJPia2cZHicCHK6fnYYvXXopBGpXNyddab5A0I3pv9F1z5cfVg/640?wx_fmt=png)

*   SUBSTRING(str,x,y) 函数：返回从字符串 str 中第 x 位置起 y 个字符长度的字符串
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVGThxtx7mXMGpNGuwsMEuvFSzd1LG301jfEicoRETlyxfQAxXgzO204g/640?wx_fmt=png)

*   LPAD(str,n,pad) 和 RPAD(str,n,pad) 函数：用字符串 pad 对 str 左边和右边进行填充，直到长度为 n 个字符长度
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVGwuBXhoPzqhHFY7CSScQ7Ekcpl5uzg0SvKrUPxxicsSmrx7zhnyPfmA/640?wx_fmt=png)

*   STRCMP(s1,s2) 用于比较字符串 s1 和 s2 的 ASCII 值大小。如果 s1 < s2，则返回 -1；如果 s1 = s2 ，返回 0 ；如果 s1 > s2 ，返回 1。
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVvOcqLORgr4ibL0AibKOzIZkwZjepdMUAsLicqTKBbMVzDLxl2CtibztGmw/640?wx_fmt=png)

*   REPLACE(str,a,b) : 用字符串 b 替换字符串 str 种所有出现的字符串 a
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVGCFibYEa74bmmTMCo4UqYJHRZNE5TFMsnRNKauicnALE9yljpmanN6JA/640?wx_fmt=png)

### 数值函数

MySQL 支持数值函数，这些函数能够处理很多数值运算。下面我们一起来学习一下 MySQL 中的数值函数，下面是所有的数值函数

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVMbkygsk5IlSJfd0IQmPdve4s5RicuDYicnoM0K45LhYtxB88WYIaia2Ew/640?wx_fmt=png)

下面我们还是以实践为主来聊一聊这些用法

*   ABS(x) 函数：返回 x 的绝对值
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVliclZx02bJ4QeRrGGsWyns8tpQYpI3LjsplZvxoD8udA3BfHbUkXUKQ/640?wx_fmt=png)

*   CEIL(x) 函数：返回大于 x 的整数
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVBqNxeeJVyMGHibEKicWYonA9AClvYQt645H8OFXjpz3zFOg2KRuWe1jQ/640?wx_fmt=png)

*   MOD(x,y)，对 x 和 y 进行取模操作
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kV4B11bUddaA8Bh50DBv4z3ibMN0ibdOvDUa4Rrd7tRLhicOtI0IbXlkiazA/640?wx_fmt=png)

*   ROUND(x,y) 返回 x 四舍五入后保留 y 位小数的值；如果是整数，那么 y 位就是 0 ；如果不指定 y ，那么 y 默认也是 0 。
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kV4yHVG19c6HiaFPSwwz4x30ccWCibbgY6nDbkUP6mPKpWIUMfsBPRghnA/640?wx_fmt=png)

*   FLOOR(x) : 返回小于 x 的最大整数，用法与 CEIL 相反
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVYkup4mJibVDSiaFc6aZuk2dE8OhfSOJZJb4rGEI8dK0sRicOaRrEaTBTw/640?wx_fmt=png)

*   TRUNCATE(x,y): 返回数字 x 截断为 y 位小数的结果， TRUNCATE 知识截断，并不是四舍五入。
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVnSuOMaauNTduM66TBuWu5woSxh46tZWwQBIWFpMbUC2qibFHn9sicXGw/640?wx_fmt=png)

*   RAND() ：返回 0 到 1 的随机值
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVC0ia4ZNmFNDDmeAw9rJy7U3qB4QWLzIYMLollAHDzVcsicHFo6F1DBuA/640?wx_fmt=png)

### 日期和时间函数

日期和时间函数也是 MySQL 中非常重要的一部分，下面我们就来一起认识一下这些函数

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVTb1emaHAPyMTPfrwbHd5WbJbggpVJIzic5BV6uHHbwiaeJYcicyrkY09w/640?wx_fmt=png)

 下面结合示例来讲解一下每个函数的使用

*   NOW(): 返回当前的日期和时间
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVmEOSib19LJyprkJlJuKF9NtXnxVXibXyvPd2jn2GOibvssTlVicqNA5fibQ/640?wx_fmt=png)

*   WEEK(DATE) 和 YEAR(DATE) ：前者返回的是一年中的第几周，后者返回的是给定日期的哪一年
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVRWj2FFJ1ibX1VzeO9768SnaGsr1icBotefI1V8c6apMv1bK5tJc3rVicA/640?wx_fmt=png)

    

*   HOUR(time) 和 MINUTE(time) : 返回给定时间的小时，后者返回给定时间的分钟
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kV5ibEy4UyZz8iciabhsmPb4h9IibMAVDkY2ypYn2GXHWQZEwHENGncyJNpw/640?wx_fmt=png)

*   MONTHNAME(date) 函数：返回 date 的英文月份
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVgDzMQvSVC1ib9nibaOTgibKf40lzvjBusoNibeKbkVQtdamus8mSOQcEIg/640?wx_fmt=png)

*   CURDATE() 函数：返回当前日期，只包含年月日
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVgMtAfJyzCrMsDV2KwpodGFUCWLEum6sedJw4eSwz3vyhQyOXra4UWw/640?wx_fmt=png)

*   CURTIME() 函数：返回当前时间，只包含时分秒
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVtOEh8gVLcKviawu6NFY3PEsL0N4fOOiaopgw454BjYlkBjwaNN4rLG4w/640?wx_fmt=png)

*   UNIX_TIMESTAMP(date) : 返回 UNIX 的时间戳
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVTUmia82ezIIzUQz8ptG7j5Fmv46TvpkWmwL34vaSUOxcU0mTrl178Hw/640?wx_fmt=png)

*   FROM_UNIXTIME(date) : 返回 UNIXTIME 时间戳的日期值，和 UNIX_TIMESTAMP 相反
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVDNTdBtZAfD71R0FiaAaAITKwrK6xN4dLpB2fq3dajdpnHTCh932KQcw/640?wx_fmt=png)

*   DATE_FORMAT(date,fmt) 函数：按照字符串 fmt 对 date 进行格式化，格式化后按照指定日期格式显示
    

具体的日期格式可以参考这篇文章 https://blog.csdn.net/weixin_38703170/article/details/82177837

我们演示一下将当前日期显示为**年月日**的这种形式，使用的日期格式是 **%M %D %Y**。

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVeo3yicFLljrCJibnuvCq9vLK96UbpeoNDooPkOXRCgvItF7tbH8Lzvbg/640?wx_fmt=png)

*   DATE_ADD(date, interval, expr type) 函数：返回与所给日期 date 相差 interval 时间段的日期
    

interval 表示间隔类型的关键字，expr 是表达式，这个表达式对应后面的类型，type 是间隔类型，MySQL 提供了 13 种时间间隔类型

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVdz2KvPiabVpP7ibEo6ibvVUg0nln2YdvxdR1H2JiajHbvsHbBZs9UxJ3fA/640?wx_fmt=png)

DATE_DIFF(date1, date2) 用来计算两个日期之间相差的天数

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVqibQZ7JzHjkibxXoib65gwpq7EsibLiaClyJj1Xn4GHR5w39iaHUu5PMuJoQ/640?wx_fmt=png)

查看离 2021 - 01 - 01 还有多少天

### 流程函数

流程函数也是很常用的一类函数，用户可以使用这类函数在 SQL 中实现条件选择。这样做能够提高查询效率。下表列出了这些流程函数

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kV8CczhiaIK2CAd2kNaTcwsqLM2ma6TpApxHjHxXKF3micfUlTnhoKb1KQ/640?wx_fmt=png)

其他函数

除了我们介绍过的字符串函数、日期和时间函数、流程函数，还有一些函数并不属于上面三类函数，它们是

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVVReBWwQ4n7BfrYIRI4wl7IPnu8VeAYcZTyx9TkAElOw2WXzNvfzjicg/640?wx_fmt=png)

下面来看一下具体的使用

*   VERSION: 返回当前数据库版本
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVwSo8ph6Do3251ZysYrlcYRuzPTHcxpNbZlCt2yyOSu6wZwvDMM31DA/640?wx_fmt=png)

*   DATABASE: 返回当前的数据库名
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVJVDKw9bsF40evJA20qYIw7TYoeTDW4iad239kug6DOz1vh9cS9OAMkw/640?wx_fmt=png)

*   USER : 返回当前登录用户名
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kViaonRWs3DNckicSViahIsvuQcKictrCTQNvYZgJw1vSAIgKDBJHzQVOceA/640?wx_fmt=png)

*   PASSWORD(str) : 返回字符串的加密版本，例如
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kV8Ah7qnk2ocFvLmpo0n0BeJ7e9zIh7iasRt6lJCV5DnGq3djATUxAm5Q/640?wx_fmt=png)

*   MD5(str) 函数：返回字符串 str 的 MD5 值
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVzRo79O2icMHkjK4qnNqvgHxGx0ZYYj8zLhupMP1Qqjsvbt1QQzdUyVg/640?wx_fmt=png)

*   INET_ATON(IP): 返回 IP 的网络字节序列
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVfnGZG3KxVubN8koT4msJ8XM09K96F3ECwNOUNP85ib5Am0M0bC3ZWdw/640?wx_fmt=png)

*   INET_NTOA(num) 函数：返回网络字节序列代表的 IP 地址，与 INET_ATON 相对
    

![](https://mmbiz.qpic.cn/mmbiz_png/wic3gC6OynIHmeNGTa1iaC8DFcMgQS36kVbv5CniaNsMQIncFb7hicZHibRt6m5wI0O9w5LNCYVOLZMicjGO9OH23xpg/640?wx_fmt=png)

总结
--

这篇文章我带你手把手撸了一波 MySQL 的高级内容，其实说高级也不一定真的高级或者说难，其实就是区分不同梯度的东西。
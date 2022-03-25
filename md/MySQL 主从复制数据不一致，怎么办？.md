> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/3XQWja5M-JiYtcfu5cmdfA)

1. 准备工作
-------

我这里有一张简单的图向大伙展示 MySQL 主从的工作方式：

![](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYnY9D7053UkCBvQvncCQ8vVoAIUkwktJGNcib72RN1yj7UjZibuOv9GVh3LLdMQomkIMeM4G9eBiaO5Q/640?wx_fmt=png)

*   主机：10.3.50.27:33061
    
*   从机：10.3.50.27:33062
    

### 1.1 主机配置

**1. 授权给从机服务器**

这里表示配置从机登录用户名为 rep1，密码为 123，并且必须从 `10.3.50.27` 这个地址登录，登录成功之后可以操作任意库中的任意表。其中，如果不需要限制登录地址，可以将 IP 地址更换为一个 `%`。

> ❝
> 
> 注意，在 MySQL8 里边，这块有一些变化。MySQL8 中用户创建和授权需要分开，不能像上面那样一步到位，具体方式如下：

**2. 修改主库配置文件**

开启 binlog ，并设置 server-id ，每次修改配置文件后都要重启 MySQL 服务才会生效

开启 binlog 主要是修改 MySQL 的配置文件 mysqld.cnf，该文件在容器的 `/etc/mysql/mysql.conf.d` 目录下。

![](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYnY9D7053UkCBvQvncCQ8vVeLhBQmVCjLJmicGN4vQrCEnFR52VRKGE8kgPRrXTeiaklGnt3F1tncdQ/640?wx_fmt=png)

针对该配置文件，我们做如下修改：

各项配置的含义松哥已经在注视中说明了。截图如下：

![](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYnY9D7053UkCBvQvncCQ8vVWea9IWyiaOPFFz9jL9DS0pkeYZkInibia11dSsUVRLTAfp0jLfEHjRXmA/640?wx_fmt=png)

*   log-bin：同步的日志路径及文件名，一定注意这个目录要是 MySQL 有权限写入的（我这里是偷懒了，直接放在了下面那个 datadir 下面）。
    
*   binlog-do-db：要同步的数据库名，当从机连上主机后，只有这里配置的数据库才会被同步，其他的不会被同步。
    
*   server-id: MySQL 在主从环境下的唯一标志符，给个任意数字，注意不能和从机重复。
    
*   **修改 binlog_format 的值为 STATEMENT，这一点很关键。**
    

这个操作的目的是为了在从数据库启动后，从这个点开始进行数据的恢复：

```
show master status;


```

![](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYnY9D7053UkCBvQvncCQ8vVTgbPQnwU4tlefYk6icdHXJw9HNZ9zE6jygg2SOxmkhnxHzJH4ibn4mlg/640?wx_fmt=png)

再看一眼 binlog_format 设置成功没：

![](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYnY9D7053UkCBvQvncCQ8vVV30dwoiaJ5YGGSuk0Yk91W0FRBZO9eDhyGbovBMrhEPUFM2smdpjYyw/640?wx_fmt=png)

### 1.2 从机配置

![](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYnY9D7053UkCBvQvncCQ8vVKJl0J5jeib4U1cbrA20kt52K8kQwKFiarhtiada8yibliamqv0vvRqr8BWA/640?wx_fmt=png)

注意从机这里只需要配置一下 server-id 即可。

**注意：如果从机是从主机复制来的，即我们通过复制 CentOS 虚拟机获取了 MySQL 实例 ，此时两个 MySQL 的 uuid 一样（正常安装是不会相同的），这时需要手动修改，修改位置在 `/var/lib/mysql/auto.cnf` ，注意随便修改这里几个字符即可，但也不可太过于随意，例如修改了 uuid 的长度。**

配置完成后，记得重启从机。

**2. 使用命令来配置从机**

```
change master to master_host='10.3.50.27',master_port=33061,master_user='rep1',master_password='123',master_log_file='javaboy_logbin.000001',master_log_pos=154;


```

这里配置了主机地址、端口以及从机登录主机的用户名和密码，注意最后两个参数要和 master 中的保持一致。

注意，由于 MySQL8 密码插件的问题，这个问题同样会给主从配置带来问题，所以在 MySQL8 配置主从上，上面这行命令需要添加 `get_master_public_key=1`，完整命令如下：

```
change master to master_host='10.3.50.27',master_port=33061,master_user='rep1',master_password='123',master_log_file='javaboy_logbin.000001',master_log_pos=154,get_master_public_key=1;


```

**3. 启动 slave 进程**

```
start slave;


```

启动之后查看从机状态：

```
show slave status\G;


```

![](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYnY9D7053UkCBvQvncCQ8vV3JRMJaILCpRCT5fkuOZCxq4lF8l3oxghwaUlBnsugl5IEGMJpmD0fw/640?wx_fmt=png)

**4. 查看 slave 的状态**

主要是下面两项值都要为为 YES，则表示配置正确：

```
Slave_IO_Running: Yes
Slave_SQL_Running: Yes


```

至此，配置完成，主机创建库，添加数据，从机会自动同步。

如果这两个有一个不为 YES ，表示主从环境搭建失败，此时可以阅读日志，查看出错的原因，再具体问题具体解决。

具体的同步过程如下：

1.  首先在从机 33062 上通过 change master 命令，设置主机 33061 的 IP、端口、用户名、密码，以及要从哪个位置开始请求 binlog（`master_log_pos`），这个位置包含文件名和日志偏移量。
    
2.  在从机 33061 上执行 `start slave` 命令，这时候从机会启动两个线程，分别是 `io_thread` 和 `sql_thread`。
    
3.  `io_thread` 负责与主机建立连接。
    
4.  主机 33061 校验完用户名、密码后，开始按照从机 33062 传过来的位置，从本地读取 binlog，发给 33062。
    
5.  从机 33062 拿到 binlog 后，写到本地文件，称为中转日志（relay log）。
    
6.  `sql_thread` 线程读取中转日志，解析出日志里的命令，并执行。
    

大致就是这样一个流程。

2. 数据不一致问题
----------

接下来我们创建一个 javaboy_db 的数据库，并在里边创建一个 user 表，user 表的定义如下：

```
CREATE TABLE `user` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `uuid` varchar(128) DEFAULT NULL,
  `name` varchar(64) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;


```

接下来我们在主机中向 user 表中插入一条记录，如下：

![](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYnY9D7053UkCBvQvncCQ8vVctG9fia354UFnvDezAAyiaGkaRBTHpQJkKo4HfdosEA1FdJ5iacz5XI9A/640?wx_fmt=png)

按道理，这条记录会同步到 33062 这台从机上：

![](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYnY9D7053UkCBvQvncCQ8vVDe28om9IcJBKe45prXibVqhghb8fibjaeQwc6ZUOU1Q6kxMRlmHfKR1Q/640?wx_fmt=png)

大家看到，**数据确实同步了，但是 uuid 却不一样。**

3. 原因分析
-------

我们知道，MySQL 主从同步最主要的依据就是 binlog，master 将自己的 binlog 发给 slave，slave 重放之后获取和 master 一致的数据。

那我们就来看看 master 生成的 binlog 是啥样子。

我们按照事件的方式来看一下 binlog，命令格式如下：

```
show binlog events [IN 'log_name'] [FROM pos] [LIMIT [offset,] row_count];


```

这个表示以事件的方式来查看 binlog，这里涉及到几个参数：

*   log_name：可以指定要查看的 binlog 日志文件名，如果不指定的话，表示查看最早的 binlog 文件。
    
*   pos：从哪个 pos 点开始查看，凡是 binlog 记录下来的操作都有一个 pos 点，这个其实就是相当于我们可以指定从哪个操作开始查看日志，如果不指定的话，就是从该 binlog 的开头开始查看。
    
*   offset：这是是偏移量，不指定默认就是 0。
    
*   row_count：查看多少行记录，不指定就是查看所有。
    

查看命令如下（我这里就从 pos 为 154 的位置开始）：

```
show binlog events IN 'javaboy_logbin.000001' FROM 154;


```

查看结果如下（部分）：

![](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYnY9D7053UkCBvQvncCQ8vVjgLKXzf6GNQwF30yMqHzUY1O9355E4cpL7flEUaR45w4Sjdh5D9gkQ/640?wx_fmt=png)

从图中可以看到，记录在 binlog 原文中的日志是：`use javaboy_db; insert into user(uuid,name) values(uuid(),'javaboy')`。

这句 SQL 将来同步到 slave 之后，slave 照着执行一下，那必然出现执行结果不一致的问题，因为 `uuid()` 函数每次执行结果都不一样。

现在小伙伴们看明白问题的原因了吧。

4. 问题解决
-------

问题倒也好解决，上篇文章我们说过，我们可以将 binlog_format 设置为 ROW 来解决这个问题。

具体操作步骤如下。

在主机中，修改 `/etc/mysql/mysql.conf.d/mysqld.cnf` 配置文件，将 binlog_format 改为 ROW，如下：

![](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYnY9D7053UkCBvQvncCQ8vVdv0mL8Y2cQjqeulgmSYdOpjEJNicGU6TEkDVB7jwS1CEfMY05zsibUrw/640?wx_fmt=png)

修改完成后，重启主机，主机重启之后，会产生新的 binlog 文件，所以我们需要重新查看主机的最新状态并重新配置从机，先来看主机，如下：

![](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYnY9D7053UkCBvQvncCQ8vVOgZbR22peaKXsrX25smXrLJBF4MmdjG10o3Q5rp1JyhQknJR1picQ4g/640?wx_fmt=png)

以此为依据，让从机重新连接主机，在从机上再进行如下操作：

```
stop slave;

change master to master_host='10.3.50.27',master_port=33061,master_user='rep1',master_password='123',master_log_file='javaboy_logbin.000002',master_log_pos=794;

start slave;


```

重新配置完从机之后，我们继续向 user 表插入一条数据，插入完成后，我们再去看从机的数据，发现此时的数据已经是一致的了。

解决这个问题，我们最主要的更改就是修改了 binlog_format 为 ROW，当我们把 binlog_format 改为 ROW 之后，我们来看看此时 binlog 中都记录了啥。

```
show binlog events IN 'javaboy_logbin.000002' FROM 794;


```

![](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYnY9D7053UkCBvQvncCQ8vV5iaZQY1nL6Fzx62pzIkECGPwtSheq21F9WoE24U0ZcJ8HF3GVZHo1Kw/640?wx_fmt=png)

大家看到，在 BEGIN 和 COMMIT 之间，就是我们的数据修改操作。

*   Table_map：这一行是说明了接下来要操作 javaboy_db.user 表。
    
*   Write_rows：这一行是说明了要写一行新的数据了。
    

不过这里看不出啥端倪来，我们借助 mysqlbinlog 工具来看看是否有新的发现。

为了查看 binlog，MySQL 为我们提供了两个官方工具，除了上面的 `show binlog events`，另一个就是 `mysqlbinlog` 命令，如下（注意在系统中执行该命令，不是在 MySQL 终端执行该命令）：

```
mysqlbinlog -vv /var/lib/mysql/javaboy_logbin.000002 --start-position=794;


```

*   -vv 表示显示详细信息，这样就会打印出 binlog 中二进制文件的内容。
    

![](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYnY9D7053UkCBvQvncCQ8vVWYw6PPjmHJHVdMnwutpMEtNTWibwzeFvt6Ria0dmRibEACWKmuw2diaicEQ/640?wx_fmt=png)

这里的内容比较多，我们来看几个比较关键的地方：

1.  Table_map: `javaboy_db`.`user` mapped to number 108：这表示接下来要操作编号为 108 的表，每张表都有一个自己的编号。
    
2.  Write_rows: table id 108 flags: STMT_END_F：这个就是具体的添加操作了，向编号为 108 的表中添加一条记录。
    

接下来那两行，大致上瞅一眼，像是 Base64 转码后的内容，大家感兴趣的可以自行解码看看，解码后有一些是乱码的，但是有一些字符串如 uuid 则没有乱码，我们也能大致猜出来这里存储的内容。

接下来我们看下面记录的 SQL，如下：

![](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYnY9D7053UkCBvQvncCQ8vVCjHapmxuLZRD5mc6wuKyBNhhTxz8Y6Uiaj2z24BUD2ztcfr57QahxyQ/640?wx_fmt=png)

这就是日志中记录的内容，可以看到，每个字段上具体的值是啥，都写下来了，这样当然就不会发生数据不一致的情况了。

好啦，今天通过一个简单的案例，跟小伙伴们分享了 binlog 两种不同的日志格式，另外还有一中 MIXED 格式现在很少用了，感兴趣的小伙伴可以结合上篇文章的内容，在本文案例的基础上继续测试 MIXED 模式，这里我就不赘述啦～
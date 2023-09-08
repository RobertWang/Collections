> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/pfdVXELt_SAvj8Ves9FKQA)

> 我们可以看到在应用程序中使用了多少未提交的事务吗？可以看到 MySQL 服务器当前正在运行多少未提交的事务。根据

我们可以看到在应用程序中使用了多少未提交的事务吗？

可以看到 MySQL 服务器当前正在运行多少未提交的事务。

根据 MySQL 的版本，有几种选择。这些在下面描述。

**The INFORMATION_SCHEMA INNODB_TRX Table In MySQL 5.5 And Later**  

查找活动 InnoDB 事务的最佳方式是使用 information_schema.INNODB_TRX 表:

```
mysql> SELECT trx_id, trx_mysql_thread_id, trx_state, trx_started, trx_isolation_level, trx_query FROM information_schema.INNODB_TRX;
+--------+---------------------+-----------+---------------------+---------------------+----------------------------------------------------------------------------------------------+
| trx_id | trx_mysql_thread_id | trx_state | trx_started         | trx_isolation_level | trx_query                                                                                    |
+--------+---------------------+-----------+---------------------+---------------------+----------------------------------------------------------------------------------------------+
|   7000 |                  46 | RUNNING   | 2016-07-26 09:23:14 | REPEATABLE READ     | LOAD DATA LOCAL INFILE 'CountryLanguage.txt' INTO TABLE 'CountryLanguage' CHARACTER SET utf8 |
|   6689 |                  43 | RUNNING   | 2016-07-26 09:17:12 | REPEATABLE READ     | NULL                                                                                         |
+--------+---------------------+-----------+---------------------+---------------------+----------------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)

```

**trx_query 为 NULL 时，表示事务空闲。例如，这可能发生在两次查询之间的短时间内，或者因为事务被遗忘。**

trx_mysql_thread_id 列可用于连接 performance_schema.threads、sys.processlist、sys.session 或 information_schema.PROCESSLIST 表以获取有关连接的更多信息，例如，它处于当前状态的时间。

### **The Sys Schema In MySQL 5.7 And Later**

在 MySQL 5.7 和更高版本中，在性能模式中启用了事务工具和使用者，sys.processlist 和 sys.session 视图也可用于查找活动事务:

```
mysql> UPDATE performance_schema.setup_consumers SET ENABLED = 'YES' WHERE NAME = 'events_transactions_current';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1 Changed: 1 Warnings: 0
mysql> UPDATE performance_schema.setup_instruments SET ENABLED = 'YES', TIMED = 'YES' WHERE NAME = 'transaction';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1 Changed: 1 Warnings: 0
...
mysql> SELECT thd_id, conn_id, user, db, command, trx_latency, trx_state, trx_autocommit, current_statement, last_statement FROM sys.session WHERE trx_state IS NOT NULL\G
*************************** 1. row ***************************
thd_id: 74
conn_id: 48
user: root@localhost
db: db1
command: Sleep
trx_latency: 18.59 m
trx_state: ACTIVE
trx_autocommit: NO
current_statement: NULL
last_statement: SELECT * FROM employees.employees WHERE emp_no = 12345
*************************** 2. row ***************************
thd_id: 68
conn_id: 42
user: root@localhost
db: db1
command: Query
trx_latency: 480.81 us
trx_state: ACTIVE
trx_autocommit: YES
current_statement: SELECT thd_id, conn_id, user, ... on WHERE trx_state IS NOT NULL
last_statement: NULL
*************************** 3. row ***************************
thd_id: 80
conn_id: 54
user: root@localhost
db: world
command: Query
trx_latency: 154.07 ms
trx_state: ACTIVE
trx_autocommit: YES
current_statement: LOAD DATA LOCAL INFILE 'City.t ... ABLE 'City' CHARACTER SET utf8
last_statement: NULL
3 rows in set (0.07 sec)

```

### SHOW ENGINE INNODB STATUS And The InnoDB Monitor

在 MySQL 的所有版本中，可以使用显示引擎 InnoDB 状态或 INNODB 监视器。输出包括一个包含所有事务列表的事务部分。例如:

```
mysql> SHOW ENGINE INNODB STATUS\G
*************************** 1. row ***************************
Type: InnoDB
Name:
Status:
=====================================
2016-07-26 09:18:12 0x7fdd88f3c700 INNODB MONITOR OUTPUT
=====================================
...
------------
TRANSACTIONS
------------
Trx id counter 7397
Purge done for trx's n:o < 7075 undo n:o = 7073, sees < 7073
...

```

说明：在 MySQL 的旧版本中，由于 SHOW ENGINE INNODB STATUS 输出的总大小限制，事务列表可能会被截断。

正在进行的交易是那些被列为活动的交易，例如:

```
---TRANSACTION 6996, ACTIVE 30 sec
MySQL thread id 45, OS thread handle 140584640485120, query id 2224 localhost 127.0.0.1 root cleaning up
Trx read view will not see trx with id >= 7073, sees < 7073

```

如果您想要自动收集 InnoDB 监视器输出，您可以启用监视器:

在 5.6.15 和更高版本以及 5.7 和更高版本中，通过将 innodb_status_output 选项设置为 on（设置为关闭来禁用）:

mysql> SET GLOBAL innodb_status_output = ON;  
Query OK, 0 rows affected (0.00 sec)
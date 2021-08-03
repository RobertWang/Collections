> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/a75c738d6b12)

_如果文章中存在某些名词不理解，不用太在意，可先放过，先部署好 环境体验一下 airflow 功能，在后续学习中再深入理解。_

**更多 airflow 资料，可查看：[airflow 从入门到精通学习笔记系列](https://www.jianshu.com/p/f318d0e83618)**

[如安装过程中遇到问题，可查看 airflow 安装及使用的常见问题](https://www.jianshu.com/p/59948794fb4d)

安装
--

实验环境：  
centos7  
python3.6  
安装脚本：

```
yum install -y python36
yum install -y python36-pip
yum install -y python36-devel
pip3 install apache-airflow
pip3 install pymysql


```

设置 airflow 的根目录：

```
echo "export AIRFLOW_HOME=/data/airflow" >> /root/.bashrc
source /root/.bashrc


```

初始化
---

初始化数据库表（默认使用本地的 sqlite 数据库）：

```
[root@VM_7_246_centos /data]# airflow initdb
[2019-08-05 19:23:29,691] {__init__.py:51} INFO - Using executor SequentialExecutor
DB: sqlite:////data/airflow/airflow.db
[2019-08-05 19:23:30,019] {db.py:350} INFO - Creating tables
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> e3a246e0dc1, current schema
INFO  [alembic.runtime.migration] Running upgrade e3a246e0dc1 -> 1507a7289a2f, create is_encrypted
/usr/local/lib/python3.6/site-packages/alembic/util/messaging.py:69: UserWarning: Skipping unsupported ALTER for creation of implicit constraint
  warnings.warn(msg)
INFO  [alembic.runtime.migration] Running upgrade 1507a7289a2f -> 13eb55f81627, maintain history for compatibility with earlier migrations
INFO  [alembic.runtime.migration] Running upgrade 13eb55f81627 -> 338e90f54d61, More logging into task_instance
INFO  [alembic.runtime.migration] Running upgrade 338e90f54d61 -> 52d714495f0, job_id indices
INFO  [alembic.runtime.migration] Running upgrade 52d714495f0 -> 502898887f84, Adding extra to Log
INFO  [alembic.runtime.migration] Running upgrade 502898887f84 -> 1b38cef5b76e, add dagrun
INFO  [alembic.runtime.migration] Running upgrade 1b38cef5b76e -> 2e541a1dcfed, task_duration
INFO  [alembic.runtime.migration] Running upgrade 2e541a1dcfed -> 40e67319e3a9, dagrun_config
INFO  [alembic.runtime.migration] Running upgrade 40e67319e3a9 -> 561833c1c74b, add password column to user
INFO  [alembic.runtime.migration] Running upgrade 561833c1c74b -> 4446e08588, dagrun start end
INFO  [alembic.runtime.migration] Running upgrade 4446e08588 -> bbc73705a13e, Add notification_sent column to sla_miss
INFO  [alembic.runtime.migration] Running upgrade bbc73705a13e -> bba5a7cfc896, Add a column to track the encryption state of the 'Extra' field in connection
INFO  [alembic.runtime.migration] Running upgrade bba5a7cfc896 -> 1968acfc09e3, add is_encrypted column to variable table
INFO  [alembic.runtime.migration] Running upgrade 1968acfc09e3 -> 2e82aab8ef20, rename user table
INFO  [alembic.runtime.migration] Running upgrade 2e82aab8ef20 -> 211e584da130, add TI state index
INFO  [alembic.runtime.migration] Running upgrade 211e584da130 -> 64de9cddf6c9, add task fails journal table
INFO  [alembic.runtime.migration] Running upgrade 64de9cddf6c9 -> f2ca10b85618, add dag_stats table
INFO  [alembic.runtime.migration] Running upgrade f2ca10b85618 -> 4addfa1236f1, Add fractional seconds to mysql tables
INFO  [alembic.runtime.migration] Running upgrade 4addfa1236f1 -> 8504051e801b, xcom dag task indices
INFO  [alembic.runtime.migration] Running upgrade 8504051e801b -> 5e7d17757c7a, add pid field to TaskInstance
INFO  [alembic.runtime.migration] Running upgrade 5e7d17757c7a -> 127d2bf2dfa7, Add dag_id/state index on dag_run table
INFO  [alembic.runtime.migration] Running upgrade 127d2bf2dfa7 -> cc1e65623dc7, add max tries column to task instance
INFO  [alembic.runtime.migration] Running upgrade cc1e65623dc7 -> bdaa763e6c56, Make xcom value column a large binary
INFO  [alembic.runtime.migration] Running upgrade bdaa763e6c56 -> 947454bf1dff, add ti job_id index
INFO  [alembic.runtime.migration] Running upgrade 947454bf1dff -> d2ae31099d61, Increase text size for MySQL (not relevant for other DBs' text types)
INFO  [alembic.runtime.migration] Running upgrade d2ae31099d61 -> 0e2a74e0fc9f, Add time zone awareness
INFO  [alembic.runtime.migration] Running upgrade d2ae31099d61 -> 33ae817a1ff4, kubernetes_resource_checkpointing
INFO  [alembic.runtime.migration] Running upgrade 33ae817a1ff4 -> 27c6a30d7c24, kubernetes_resource_checkpointing
INFO  [alembic.runtime.migration] Running upgrade 27c6a30d7c24 -> 86770d1215c0, add kubernetes scheduler uniqueness
INFO  [alembic.runtime.migration] Running upgrade 86770d1215c0, 0e2a74e0fc9f -> 05f30312d566, merge heads
INFO  [alembic.runtime.migration] Running upgrade 05f30312d566 -> f23433877c24, fix mysql not null constraint
INFO  [alembic.runtime.migration] Running upgrade f23433877c24 -> 856955da8476, fix sqlite foreign key
INFO  [alembic.runtime.migration] Running upgrade 856955da8476 -> 9635ae0956e7, index-faskfail
INFO  [alembic.runtime.migration] Running upgrade 9635ae0956e7 -> dd25f486b8ea
INFO  [alembic.runtime.migration] Running upgrade dd25f486b8ea -> bf00311e1990, add index to taskinstance
INFO  [alembic.runtime.migration] Running upgrade 9635ae0956e7 -> 0a2a5b66e19d, add task_reschedule table
INFO  [alembic.runtime.migration] Running upgrade 0a2a5b66e19d, bf00311e1990 -> 03bc53e68815, merge_heads_2
INFO  [alembic.runtime.migration] Running upgrade 03bc53e68815 -> 41f5f12752f8, add superuser field
INFO  [alembic.runtime.migration] Running upgrade 41f5f12752f8 -> c8ffec048a3b, add fields to dag
INFO  [alembic.runtime.migration] Running upgrade c8ffec048a3b -> a56c9515abdc, Remove dag_stat table
INFO  [alembic.runtime.migration] Running upgrade c8ffec048a3b -> dd4ecb8fbee3, Add schedule interval to dag
INFO  [alembic.runtime.migration] Running upgrade dd4ecb8fbee3 -> 939bb1e647c8, task reschedule fk on cascade delete
WARNI [airflow.utils.log.logging_mixin.LoggingMixin] cryptography not found - values will not be stored encrypted.
Done.


```

查看其生成文件：

```
[root@VM_7_246_centos /data]# ls $AIRFLOW_HOME
airflow.cfg  airflow.db  logs  unittests.cfg


```

配置
--

配置 MySQL 数据库（创建 airflow 数据库，并创建用户和授权，给 airflow 访问数据库使用）：

```
# 新建airflow的数据库
MariaDB [(none)]> CREATE DATABASE airflow;
# 授权从localhost登录的root账号对airflow的所有操作
MariaDB [(none)]> GRANT all privileges on root.* TO 'root'@'localhost'  IDENTIFIED BY '123456'; 
MariaDB [(none)]> FLUSH PRIVILEGES;


```

配置 airflow 使用 LocalExecutor 执行器，及使用 MySQL 数据库：

```
vim airflow/airflow.cfg
# The executor class that airflow should use. Choices include
# SequentialExecutor, LocalExecutor, CeleryExecutor, DaskExecutor, KubernetesExecutor
#executor = SequentialExecutor
executor = LocalExecutor

# The SqlAlchemy connection string to the metadata database.
# SqlAlchemy supports many different database engine, more information
# their website
#sql_alchemy_conn = sqlite:////data/airflow/airflow.db
sql_alchemy_conn = mysql+pymysql://root:123456@localhost:3306/airflow


```

再次初始化数据库表（在 MySQL 中创建）：

```
airflow initdb


```

查看创建的 airflow 数据表：

```
MariaDB [None] > use airflow;
MariaDB [airflow]> show tables;
+-------------------+
| Tables_in_airflow |
+-------------------+
| alembic_version  |
| chart            |
| connection        |
| dag              |
| dag_pickle        |
| dag_run          |
| dag_stats        |
| import_error      |
| job              |
| known_event      |
| known_event_type  |
| log              |
| sla_miss          |
| slot_pool        |
| task_fail        |
| task_instance    |
| users            |
| variable          |
| xcom              |
+-------------------+


```

服务启动
----

添加 airflow-scheduler 服务启动脚本：

```
/usr/lib/systemd/system/airflow-scheduler.service 
[Unit]
Description=Airflow Scheduler
Wants=network-online.target
After=network-online.target

[Service]
User=root
Group=root
Type=simple
Restart=on-failure
Environment=export AIRFLOW_HOME=/data/airflow
ExecStart=/usr/local/bin/airflow scheduler
LimitNOFILE=10000
TimeoutStopSec=20

[Install]
WantedBy=multi-user.target


```

启动 airflow-scheduler 服务：

```
systemctl start airflow-scheduler


```

添加 airflow-webserver 服务启动脚本：

```
/usr/lib/systemd/system/airflow-webserver.service 
[Unit]
Description=Airflow Scheduler
Wants=network-online.target
After=network-online.target

[Service]
User=root
Group=root
Type=simple
Restart=on-failure
Environment=export AIRFLOW_HOME=/data/airflow
ExecStart=/usr/local/bin/airflow webserver
LimitNOFILE=10000
TimeoutStopSec=20

[Install]
WantedBy=multi-user.target


```

启动 airflow-webserver 服务：

```
systemctl start airflow-webserver


```

使用
--

访问 [http://localhost:8080](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A8080) 地址，看到首页界面即成功；  

![](http://upload-images.jianshu.io/upload_images/19135173-3f5c43e27f8e4b72.PNG) 首页

**更多 airflow 资料，可查看：[airflow 从入门到精通学习笔记系列](https://www.jianshu.com/p/f318d0e83618)**

**如发现文中有错误，望留言指明，万分感激；  
如对此文章内容感兴趣，想进一步探讨，可以留言交流；  
如想转发此文章，请留言协商一下，切勿不指明出处的转发，尊重原创；  
如阅读过程中有收获，并想感谢一下，欢迎打赏；  
---- 小林帮**
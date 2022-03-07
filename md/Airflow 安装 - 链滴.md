> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [ld246.com](https://ld246.com/article/1556442877206)

> Airflow 安装部署 新闻信息是通过爬虫获取，使用 scrapy 框架进行爬虫任务；使用 airflow 工作流监控平台对爬虫任务进行管理、监控（可使用 CeleryExecutor 分布式，也可使用 LocalExecutor 多进程进行数据采集）。

Airflow 安装部署
------------

新闻信息是通过爬虫获取，使用 scrapy 框架进行爬虫任务；使用 airflow 工作流监控平台对爬虫任务进行管理、监控（可使用 CeleryExecutor 分布式，也可使用 LocalExecutor 多进程进行数据采集）。以下主要是对 airflow 的安装和配置。

1. 系统环境 y
---------

目前使用的系统环境为 `Centos Linux release 7.4.1708 (core)`,`linux` 版本的内核 `Linux version 3.10.0-693.2.2e17.x86_64`.

ip 地址：

*   外网：`47.104.191.52`
*   内网：`172.31.178.92`

2. 准备 python 环境，安装 Anaconda
---------------------------

### 2.1 下载安装文件

[下载地址 1(官方网站)](https://link.ld246.com/forward?goto=https%3A%2F%2Frepo.continuum.io%2Farchive%2Findex.html)

[下载地址 2(清华开源镜像)](https://link.ld246.com/forward?goto=https%3A%2F%2Fmirrors.tuna.tsinghua.edu.cn%2Fanaconda%2Farchive%2F)

下载对应版本安装文件

### 2.2 上传安装文件，开始安装

将下载的文件上传到 Linux 系统中 `/opt`

1、执行命令安装

`cd /opt`

`sh Anaconda3-5.2.0-Linux-x86_64.sh` (按回车键，直到出现>>> 输入 yes)

`/opt/anaconda3` (安装目录)

2、配置环境变量

`echo "export PATH=/opt/anaconda3/bin:$PATH" >> /etc/profile`

`source /etc/profile`

3. 安装 mysql （供 airflow 使用）、redis
--------------------------------

mysql 作为 airflow 数据库，主要是记录 airflow 信息；

redis 作为 celery 的 broker 和 backend（也可以用 RabbitMQ），如果不使用 CeleryExecutor 则不需要 redis 配置。

4. 安装配置 airflow
---------------

1.  通过 `anaconda` 安装虚拟环境 `news_push`
    
    `/opt/anaconda3/bin/conda create -y --name news_push python=3.6.5`
    
2.  airflow 安装、配置
    
    *   激活虚拟环境 `news_push`
        
        `source activate news_push`
        
    *   通过 pip 安装 airflow
        
        `pip install apache-airflow`
        
    *   配置 airflow 目录 (先创建 / opt/NewsPush 项目目录)
        
        `echo "export AIRFLOW_HOME=/opt/NewsPush/airflow >> /etc/profile"`
        
        `source /etc/profile`
        
    *   初始化数据库
        
        `airflow initdb`
        
    *   启动 airflow
        
        `airflow webserver -p 5556`
        
        可到浏览器查看 http://**ip**:5556/admin/
        
    *   配置 `airflow`- 更改数据库为 `mysql`
        
        *   修改 mysql 配置文件参数（/etc/my.cnf），并重启 mysql
            
            explicit_defaults_for_timestamp=true
            
        *   登录 mysql
            
            `mysql -uroot -p` 回车后输入**密码**
            
        *   新建用户 airflow
            
            `create user 'airflow'@'localhost' identified by 'airflow';`
            
        *   创建数据库 airflow
            
            `create database airflow;`
            
        *   赋予权限
            
            `grant all privileges on airflow.* to 'airflow'@'%' identified by 'airflow';`
            
            `flush privileges;`
            
    *   修改 airflow 配置文件
        
        `vim /opt/NewsPush/airflow/airflow.cfg`
        
        修改内容为：
        
        ```
        executor = CeleryExecutor
        sql_alchemy_conn=mysql://ariflow:airflow@localhost:3306/ariflow
        load_examples = False
        endpoint_url = http://localhost:5556
        base_url = http://localhost:5556
        web_server_port = 5556
        broker_url = redis://172.31.178.92:6379/3
        celery_result_backend = redis://172.31.178.92:6379/4
        flower_port = 5557
        
        
        ```
        
    *   安装 celery 支持及 celeryde redis 组件
        
        `pip install airflow[celery]`
        
        `pip install celery[redis]`
        
    *   安装 MySQL-python
        
        `yum install MySQL-python`
        
        `pip install PyMySQL==0.7.1`
        
        如果 PyMySQL 版本为 0.8.0 或以上则会有警告:
        
        ```
        /opt/anaconda3/envs/news_push/lib/python3.6/site-packages/pymysql/cursors.py:170: Warning: (1300, "Invalid utf8mb4 chara
          result = self._query(query)
        
        
        ```
        
    *   再次初始化
        
        `airflow initdb`
        
    *   错误解决
        
        *   错误信息
            
            ```
            Traceback (most recent call last):
              File "/opt/anaconda3/envs/news_push/bin/airflow", line 17, in <module>
                from airflow import configuration
              File "/opt/anaconda3/envs/news_push/lib/python3.6/site-packages/airflow/__init__.py", line 30, in <module>
                from airflow import settings
              File "/opt/anaconda3/envs/news_push/lib/python3.6/site-packages/airflow/settings.py", line 159, in <module>
                configure_orm()
              File "/opt/anaconda3/envs/news_push/lib/python3.6/site-packages/airflow/settings.py", line 147, in configure_orm
                engine = create_engine(SQL_ALCHEMY_CONN, **engine_args)
              File "/opt/anaconda3/envs/news_push/lib/python3.6/site-packages/sqlalchemy/engine/__init__.py", line 424, in create_engine
                return strategy.create(*args, **kwargs)
              File "/opt/anaconda3/envs/news_push/lib/python3.6/site-packages/sqlalchemy/engine/strategies.py", line 81, in create
                dbapi = dialect_cls.dbapi(**dbapi_args)
              File "/opt/anaconda3/envs/news_push/lib/python3.6/site-packages/sqlalchemy/dialects/mysql/mysqldb.py", line 102, in dbapi
                return __import__('MySQLdb')
            ModuleNotFoundError: No module named 'MySQLdb'
            
            
            
            ```
            
        *   解决 (MySQLdb 对 python3.* 支持)
            
            `vim /opt/anaconda3/envs/news_push/lib/python3.6/site-packages/sqlalchemy/dialects/mysql/mysqldb.py` （最后一行错误信息. py 文件路径）
            
            在代码开头增加
            
            ```
            import pymysql
            pymysql.install_as_MySQLdb()
            
            
            ```
            
    *   再次初始化
        
        `airflow initdb`
        
3.  airflow 启动及测试
    
    *   创建一个 dag(/opt/NewsPush/airflow/dags/hello_world.py)
        
        ```
        from airflow import DAG
        from airflow.utils.dates import days_ago
        from airflow.operators.bash_operator import BashOperator
        from airflow.operators.python_operator import PythonOperator
        
        default_args = {
            'owner': 'airflow',
            'start_date': days_ago(1) #必须设置，尽量用固定时间，如果使用动态的当前时间会有意想不到的问题。任务会先执行一次，再根据起始时间和schedule_interval设置开始执行
        }
        
        dag = DAG(
            'example_hello_world_dag',
            default_args=default_args,
            description='my first DAG',
            # schedule_interval=timedelta(days=1)
            schedule_interval='0 */1 * * *'	#每个小时执行一次
        )
        
        def print_hello():
            return 'Hello World!'
        
        hello_operator = PythonOperator(
            task_id='hello_task',
            python_callable=print_hello,
            dag=dag
        )
        
        
        ```
        
    *   airflow 启动
        
        以下命令都是单独开启一个窗口来启动，便于观察日志（也可以在后台启动）
        
        **注意：celery worker 启动尽量不要用 root 用户启动，如果要用 root 用户启动则添加环境变量。**
        
        **用其他用户启动则 airflow 启动命令也对应用用户启动，并更改项目目录权限属于此用户，否则日志记录时没有权限会影响 worker 运行。**
        
        ```
        echo export C_FORCE_ROOT= true >> /etc/profile
        source /etc/profile
        
        
        ```
        
        ```
        airflow webserver #启动airflow web页面
        airflow scheduler #启动调度器，执行任务调度，不过任务默认是关闭的，需要在页面手动开启
        airflow worker #启动celery workd
        airflow flower #启动flower监控页面
        
        
        ```
        
        linux 添加用户、用户组、密码
        
        ```
        groupadd airflow #添加用户组airflow
        useradd -g airflow airflow #添加用airflow到用户组airflow
        passwd airflow #设置密码
        
        
        ```
        
        更改项目目录权限为启动用户 (airflow) 权限
        
        `chowm -R airflow:airflow /opt/NewsPush/`
        
        airflow 浏览器访问地址：[http://47.104.191.52/admin](https://link.ld246.com/forward?goto=http%3A%2F%2F47.104.191.52%3A5556%2Fadmin)
        
        flower 浏览器访问地址：[http://47.104.191.52/](https://link.ld246.com/forward?goto=http%3A%2F%2F47.104.191.52%3A5557%2F)
        

参考资料
----

[Airflow 使用](https://link.ld246.com/forward?goto=https%3A%2F%2Fwww.cnblogs.com%2Ftestzcy%2Fp%2F8427141.html)

[Airflow 安装启动](https://link.ld246.com/forward?goto=https%3A%2F%2Fblog.csdn.net%2Fqazplm12_3%2Farticle%2Fdetails%2F53065654)

[Airflow 框架下支持 celery 的问题](https://link.ld246.com/forward?goto=http%3A%2F%2F13.59.15.96%2Fdoku.php%2Fpython%3Aframe%3Aairflow%25E6%25A1%2586%25E6%259E%25B6%25E4%25B8%258B%25E6%2594%25AF%25E6%258C%2581celery%25E7%259A%2584%25E9%2597%25AE%25E9%25A2%2598)

964 17 424 64 122 53 4 53 210
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/duanhaoxin/p/11211815.html)

> # Airflow 1.10 + 安装本次安装 Airflow 版本为 1.10+，其需要依赖 Python 和 DB，本次选择的 DB 为 Mysql。本次安装组件及版本如下：Airflow == 1.10.0Pytho

[Airflow 安装与使用](https://www.cnblogs.com/duanhaoxin/p/11211815.html)
===================================================================

```
# Airflow 1.10+安装

本次安装Airflow版本为1.10+，其需要依赖Python和DB，本次选择的DB为Mysql。
本次安装组件及版本如下：Airflow == 1.10.0
Python == 3.6.5
Mysql == 5.7

# 整体流程
1. 建表
2. 安装
3. 配置
4. 运行
5. 配置任务


```
启动schedule
airflow scheduler -D
启动webserver
airflow webserver -D

ps -ef|grep -Ei '(airflow-webserver)'| grep master | awk '{print $2}'|xargs -i kill {}
ps -ef | grep -Ei 'airflow' | grep -v 'grep' | awk '{print $2}' | xargs -i kill {}
## 建库、建用户

```
库名为airflow
'create database airflow;'
建用户

用户名为airflow，并且设置所有ip均可以访问。

create user 'airflow'@'%' identified by 'airflow';
create user 'airflow'@'localhost' identified by 'airflow';

用户授权
这里为新建的airflow用户授予airflow库的所有权限

grant all on airflow.* to 'airflow'@'%';
flush privileges
```

## Airflow安装
```
这里通过 virtualenv 进行安装。

----- 通过virtualenv安装

$ mkdir /usr/local/virtual_env && cd /usr/local/virtual_env # 创建目录
$ virtualenv --no-site-packages airflow --python=python  # 创建虚拟环境
$ source /usr/local/virtual_env/airflow/bin/activate # 激活虚拟环境

----- 安装指定版本或者默认
$ pip install apache-airflow -i https://pypi.douban.com/simple
在安装完一堆的依赖后，就需要配置 AIRFLOW_HOME 环境变量，后续的 DAG 和 Plugin 都将以该目录作为根目录查找，如上，可以直接设置为 /tmp/project 。

报错
ERROR: flask 1.1.1 has requirement Jinja2>=2.10.1, but you'll have jinja2 2.10 which is incompatible.
ERROR: flask 1.1.1 has requirement Werkzeug>=0.15, but you'll have werkzeug 0.14.1 which is incompatible.

执行：pip3 install -U Flask==1.0.4
执行：pip3 install -U pika==0.13.1


重新执行 ：pip install apache-airflow -i https://pypi.douban.com/simple


----- 设置环境变量
(airflow) $ export AIRFLOW_HOME=/tmp/airflow


----- 查看其版本信息
(airflow) $ airflow version
  ____________       _____________
 ____    |__( )_________  __/__  /________      __
____  /| |_  /__  ___/_  /_ __  /_  __ \_ | /| / /
___  ___ |  / _  /   _  __/ _  / / /_/ /_ |/ |/ /
 _/_/  |_/_/  /_/    /_/    /_/  \____/____/|__/
   v1.8.0
执行了上述的命令后，会生成 airflow.cfg 和 unittests.cfg 两个文件，其中前者是一个配置文件 。

## airflow 配置


----- 修改Airflow DB配置
### 1. 安装Mysql模块

pip install "apache-airflow[mysql]"
这里可以简单说下，airflow依赖的其他组件均可以此方式安装。在之后安装password组件同样是通过此方式。

修改Airflow DB配置
修改${AIRFLOW_HOME}/airflow.cfg

sql_alchemy_conn = mysql+mysqldb://airflow:airflow@localhost:3306/airflow
参数的格式为mysql://帐号:密码@ip:port/db

初始化db
新建airflow依赖的表。

airflow initdb

如报错 Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2)
需改sql_alchemy_conn = mysql+mysqldb://airflow:airflow@127.0.0.1:3306/airflow

```

### 2. 用户认证
```
本文采用的用户认证方式为password方式，其他方式如LDAP同样支持但是本文不会介绍。笔者在安装时实验过LDAP方式但是未成功过。

安装passsword组件
pip install "apache-airflow[password]"
2. 修改 airflow.cfg

[webserver]
authenticate = True
auth_backend = airflow.contrib.auth.backends.password_auth
3. 在python环境中执行如下代码以添加账户：

import airflow  
from airflow import models, settings  
from airflow.contrib.auth.backends.password_auth import PasswordUser  
user = PasswordUser(models.User())  
user.username = 'admin'  # 用户名
user.email = 'emailExample@163.com' # 用户邮箱  
user.password = 'password'   # 用户密码
session = settings.Session()  
session.add(user)  
session.commit()  
session.close()  
exit() 
```


### 3. 配置邮件服务
此配置设置的是dag的task失败或者重试时发送邮件的发送者。配置如下：
```
[smtp]
# If you want airflow to send emails on retries, failure, and you want to use
# the airflow.utils.email.send_email_smtp function, you have to configure an
smtp_host = smtp.163.com
smtp_starttls = True
smtp_ssl = False
# Uncomment and set the user/pass settings if you want to use SMTP AUTH
smtp_user = mailExample@163.com
smtp_password = password
smtp_port = 25
smtp_mail_from = mailExample@163.com
接下来简单把dag的Python代码列出来，以供参考：

default_args = {
  'owner': 'ownerExample',
  'start_date': datetime(2018, 9, 18),
  'email': ['mailReceiver@163.com'], # 出问题时，发送报警Email的地址，可以填多个，用逗号隔开。
  'email_on_failure': ['mailReceiver@163.com'], # 任务失败且重试次数用完时发送Email。
  'email_on_retry': True, # 任务重试时是否发送Email
  'depends_on_past': False, # 是否依赖于过去。如果为True，那么必须要昨天的DAG执行成功了，今天的DAG才能执行。
  'retries': 3,
  'retry_delay': timedelta(minutes=3),
}
```

### 4、配置Executor
```
设置Executor
修改：airflow.cfg

executor = LocalExecutor
本文中由于只有单节点所以使用的是LocalExecutor模式。
```

### 5. 修改log地址
```
[core]
base_log_folder = /servers/logs/airflow
[scheduler]
child_process_log_directory = servers/logs/airflow/scheduler
```

### 6. 修改webserver地址
```
修改webserver地址
[webserver]
base_url = http://host:port
可以通过上面配置的地址访问webserver。
```

### 7. 可选配置

```
（可选）修改Scheduler线程数
如果调度任务不多的话可以把线程数调小，默认为32。参数为：parallelism


（可选）不加载example dag
如果不想加载示例dag可以把load_examples配置改为False，默认为True。这个配置只有在第一次启动airflow之前设置才有效。


如果此方法不生效，可以删除${PYTHON_HOME}/site-packages/airflow/example_dags目录，也是同样的效果。

（可选）修改检测新dag间隔
修改min_file_process_interval参数为10，每10s识别一次新的dag。默认为0，没有时间间隔。

```

## 运行airflow
```
启动schedule
airflow scheduler 
启动webserver
airflow webserver 
```


## 安装问题汇总
```
1. Global variable explicit_defaults_for_timestamp needs to be on (1) for mysql


修改Mysql配置文件my.cnf，具体步骤如下：

查找my.cnf文件位置
mysql --help | grep my.cnf
下图红框处为my.cnf文件所在位置：


修改文件
explicit_defaults_for_timestamp=true
注意：必须写在【mysqld】下

重启Mysql
sudo systemctl restart mysqld.service
查看修改是否生效。执行如下SQL，如果值为1则为生效。


2. pip install "apache-airflow[mysql]"报错：

mysql_config not found

安装mysql-devel:

首先查看是否有mysql_config文件。
find / -name mysql_config

如果没有安装mysql-devel
yum install mysql-devel
安装之后再次查找，结果如图：

3. 其他问题找我
```

## 配置任务

在 AirFlow 中，每个节点都是一个任务，可以是一条命令行 (BashOperator)，可以是一段 Python 脚本 (PythonOperator) 等等，然后这些节点根据依赖关系构成了一条流程，一个图，称为一个 DAG 。

默认会到 ${AIRFLOW_HOME}/dags 目录下查找，可以直接在该目录下创建相应的文件。

如下是一个简单的示例。

```
import airflow
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from airflow.operators.python_operator import PythonOperator
from datetime import timedelta, datetime
import pytz

# -------------------------------------------------------------------------------
# these args will get passed on to each operator
# you can override them on a per-task basis during operator initialization

default_args = {
    'owner': 'qxy',
    'depends_on_past': False,
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

tz = pytz.timezone('Asia/Shanghai')
# naive = datetime.strptime("2018-06-13 17:40:00", "%Y-%m-%d %H:%M:%S")
# local_dt = tz.localize(naive, is_dst=None)
# utc_dt = local_dt.astimezone(pytz.utc).replace(tzinfo=None)

dt = datetime(2019, 7, 16, 16, 30, tzinfo=tz)
utc_dt = dt.astimezone(pytz.utc).replace(tzinfo=None)


dag = DAG(
    'airflow_interval_test',
    default_args=default_args,
    description='airflow_interval_test',
    schedule_interval='35 17 * * *',
    start_date=utc_dt

)

t1 = BashOperator(
    task_id='sleep',
    bash_command='sleep 5',
    dag=dag)

t2 = BashOperator(
    task_id='print_date',
    bash_command='date',
    dag=dag)

t1 >> t2

```
该文件创建一个简单的 DAG，只有三个运算符，两个 BaseOperator ，也就是执行 Bash 命令分别打印日期以及休眠 5 秒；另一个为 PythonOperator 在执行任务时调用 print_hello() 函数。
文件创建好后，放置到 ${AIRFLOW_HOME}/dags，airflow 自动读取该DAG。

----- 测试是否正常，如果无报错那么就说明正常
$ python /tmp/project/dags/hello_world.py

```
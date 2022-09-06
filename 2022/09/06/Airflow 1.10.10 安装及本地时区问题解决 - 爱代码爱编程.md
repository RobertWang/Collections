> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [icode.best](https://icode.best/i/61364443966858)

> airflow2.0 之后的版本更改了时区问题，更改 airflow.cfg 文件中的 default_timezone 和 default_ui_timezone 为 Asia/Shanghai 后，发现在 Airflow Web UI 上已经显示了北京时间，但是对 scheduler 并不起作用，需要对源码稍作修改才能使得 scheduler 安装 utc + 8 的时区来调度任务。

airflow2.0 之后的版本更改了时区问题，更改 airflow.cfg 文件中的 default_timezone 和 default_ui_timezone 为 Asia/Shanghai 后，发现在 Airflow Web UI 上已经显示了北京时间，但是对 scheduler 并不起作用，而且调度不稳定，经常出现不调度的情况，果断回退到 1.10.10 版本

### 1.0 安装 MySQL 数据库

省略

### 1.1 创建数据库

```
create database airflow default charset utf8 collate utf8_general_ci; 

create user 'airflow'@'%' identified by 'airflow';
create user 'airflow'@'localhost' identified by 'airflow';

grant all on airflow.* to 'airflow'@'%';
flush privileges;


```

### 1.2 配置 my.cnf

```
[client]
default-character-set=utf8mb4
[mysql]
default-character-set=utf8mb4
[mysqld]
collation-server = utf8mb4_unicode_ci
init-connect='SET NAMES utf8mb4'
character-set-server = utf8mb4
explicit_defaults_for_timestamp=1


```

### 2.0 安装环境依赖

```
yum install python-devel mysql-devel -y
yum install python-devel
yum install python3-devel
yum install mysql-devel
pip3 install apache-airflow==1.10.10


```

#### 2.1 修改 airflow 配置：

```
airflow initdb

具体要修改的内容如下
[core]
executor=LocalExecutor
sql_alchemy_conn = mysql://user:password@IP:3306/airflow
[smtp]
smtp_host = mail.ndpmedia.com
smtp_starttls = True
smtp_ssl = False
smtp_user = user
smtp_password = pass
smtp_port = 25
smtp_timeout = 30
smtp_mail_from =与user相同
smtp_retry_limit = 5

[webserver]
security = Flask AppBuilder
secure_mode = True
rbac=True


```

#### 2.2 启动

```
airflow users create --username admin --firstname admin --lastname admin --role Admin --email  example@XX.com
airflow webserver 启动web服务
airflow scheduler  启动调度程序


```

#### 2. 后台启动

```
编辑shell脚本启动
vim startAirflow.sh


ps -ef|grep airflow | grep -v grep | awk  '{print "kill -9 " $2}'|sh

nohup airflow webserver >/dev/null 2>&1 &

nohup airflow scheduler >/dev/null 2>&1 &


```

#### 3. 安装异常处理

1.`Exception: Global variable explicit_defaults_for_timestamp needs to be on (1) for mysql`  
解决方法：

```
进入mysql airflow 数据库，设置global explicit_defaults_for_timestamp
show global variables like '%timestamp%';
set global explicit_defaults_for_timestamp =1;


```

#### 更改时区步骤：

###### 1. 更改配置文件：

在 airflow 家目录下修改 airflow.cfg，设置

```
default_timezone = Asia/Shanghai


```

###### 2. 更改源码：

进入 airflow 包的安装位置，也就是 site-packages 中的 airflow，我这边的位置是 / usr/local/lib/python3/site-packages/airflow

`(1).修改utils/timezone.py`

在 utc = pendulum.timezone(‘UTC’)( 第 27 行 ) 代码下添加

```
from airflow.configuration import conf
try:
    tz = conf.get("core", "default_timezone")
    if tz == "system":
        utc = pendulum.local_timezone()
    else:
        utc = pendulum.timezone(tz)
except Exception:
    pass


```

修改 utcnow() 函数 (在第 69 行)

```
d = dt.datetime.now()


```

`(2).修改 utils/sqlalchemy.py`

在 utc = pendulum.timezone(‘UTC’) (第 37 行) 代码下添加

```
try:
    tz = conf.get("core", "default_timezone")
    if tz == "system":
        utc = pendulum.local_timezone()
    else:
        utc = pendulum.timezone(tz)
except Exception:
    pass


```

`(3)修改airflow/www/templates/admin/master.html(第31行)`

```
// var UTCseconds = (x.getTime() + x.getTimezoneOffset() * 60 * 1000);
var UTCseconds = x.getTime();
// "timeFormat":"H:i:s %UTC%",
"timeFormat":"H:i:s"


```

至此，airflow 1.10.10 时区问题就正常了
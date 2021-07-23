> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/cord/p/9450910.html)

#### 1, 简介

​ Airflow 是一个可编程，调度和监控的工作流平台，基于有向无环图 (DAG)，airflow 可以定义一组有依赖的任务，按照依赖依次执行。airflow 提供了丰富的命令行工具用于系统管控，而其 web 管理界面同样也可以方便的管控调度任务，并且对任务运行状态进行实时监控，方便了系统的运维和管理。

#### 2，执行器 (Executor)

​ Airflow 本身是一个综合平台，它兼容多种组件，所以在使用的时候有多种方案可以选择。比如最关键的执行器就有四种选择:

SequentialExecutor：单进程顺序执行任务，默认执行器，通常只用于测试

LocalExecutor：多进程本地执行任务

CeleryExecutor：分布式调度，生产常用

DaskExecutor ：动态任务调度，主要用于数据分析

在当前项目使用`CeleryExecutor`作为执行器。

celery 是一个分布式调度框架，其本身无队列功能，需要使用第三方组件，比如 redis 或者 rabbitmq，当前项目使用的是 rabbitmq，系统整体结构如下所示:

![](https://images2018.cnblogs.com/blog/564309/201808/564309-20180819194613780-1361764238.png)

其中:

turing 为外部系统

GDags 服务帮助拼接成 dag

master 节点 webui 管理 dags、日志等信息

scheduler 负责调度，只支持单节点

worker 负责执行具体 dag 中的 task, worker 支持多节点

在整个调度系统中，节点之间的传递介质是消息，而消息的本质内容是执行脚本的命令，也就是说，工作节点的 dag 文件必须和 master 节点的 dag 文件保持一致，不然任务的执行会出问题。

#### 3，任务处理器

airflow 内置了丰富的任务处理器，用于实现不同类型的任务:

BashOperator : 执行 bash 命令

PythonOperator : 调用 python 代码

EmailOperator : 发送邮件

HTTPOperator : 发送 HTTP 请求

SqlOperator : 执行 SQL 命令

除了这些基本的构建块之外，还有更多的特定处理器:`DockerOperator`，`HiveOperator`，`S3FileTransferOperator`，`PrestoToMysqlOperator`，`SlackOperator` ...

在当前项目使用了`HTTPOperator` 作为执行器，用于调用 JAVA 服务，整体结构图如下:

![](https://images2018.cnblogs.com/blog/564309/201808/564309-20180809182421066-723069403.jpg)

![](https://images2018.cnblogs.com/blog/564309/201807/564309-20180713094202245-308260969.jpg)

关于 airflow 的环境搭建可以参考另外一篇博客: [https://www.cnblogs.com/cord/p/9226608.html](https://www.cnblogs.com/cord/p/9226608.html)

#### 4，基本使用

##### 4.1，常用命令

```
$ airflow webserver -D		 守护进程运行webserver


```

```
$ airflow scheduler -D		 守护进程运行调度器


```

```
$ airflow worker -D		     守护进程运行调度器


```

```
$ airflow worker -c 1 -D 	 守护进程运行celery worker并指定任务并发数为1


```

```
$ airflow pause dag_id　 	暂停任务


```

```
$ airflow unpause dag_id 	 取消暂停，等同于在管理界面打开off按钮


```

```
$ airflow list_tasks dag_id  查看task列表


```

```
$ airflow clear dag_id       清空任务实例


```

```
$ airflow trigger_dag dag_id -r RUN_ID -e EXEC_DATE  运行整个dag文件


```

```
$ airflow run dag_id task_id execution_date       运行task


```

##### 4.2，web 管控界面的使用

启动 web 管控界面需要执行`airflow webserver -D`命令，默认访问端口是 8080

[http://110.55.63.51:8080/admin/](http://110.55.63.51:8080/admin/)

![](https://images2018.cnblogs.com/blog/564309/201808/564309-20180822220832467-1828073871.png)

(1) 任务启动暂停开关

(2) 任务运行状态

(3) 待执行，未分发的任务

(4) 手动触发执行任务

(5) 任务管控界面

选择对应 dag 栏目，点击 (5) Graph View 即可进入任务管控界面

![](https://images2018.cnblogs.com/blog/564309/201808/564309-20180822220854820-333369803.png)

点击对应的任务，会弹出一个任务管控台，主要几个功能如下:

View Log : 查看任务日志

Run : 运行选中任务

Clear：清空任务队列

Mark Success : 标记任务为成功状态

##### 4.3 通过定义 DAG 文件实现创建定时任务

###### 1) 普通任务

```
from datetime import timedelta, datetime
import airflow
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from airflow.operators.dummy_operator import DummyOperator

default_args = { #默认参数
    'owner': 'jifeng.si', #dag拥有者，用于权限管控
    'depends_on_past': False,  #是否依赖上游任务
    'start_date': datetime(2018, 5, 2), #任务开始时间，默认utc时间
    'email': ['123456789@qq.com'], #告警通知邮箱地址
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

dag = DAG(
    'example_hello_world_dag',  #dag的id
    default_args=default_args,
    description='my first DAG', #描述
    schedule_interval='*/25 * * * *', # crontab
    start_date=datetime(2018, 5, 28) #开始时间，覆盖默认参数
)

def print_hello():
    return 'Hello world!'

dummy_operator = DummyOperator(task_id='dummy_task', dag=dag)

hello_operator = BashOperator(   #通过BashOperator定义执行bash命令的任务
    task_id='sleep_task',
    depends_on_past=False,
    bash_command='echo `date` >> /home/py/test.txt',
    dag=dag
)

dummy_operator >> hello_operator #设置任务依赖关系
#dummy_operator.set_downstream(hello_operator)


```

###### 2) 定义 http 任务并使用本地时间

```
import os
from datetime import timedelta, datetime
import pytz
from airflow.operators.http_operator import SimpleHttpOperator
from airflow.models import DAG

default_args = {
    'owner': 'cord',
    # 'depends_on_past': False,
    'depends_on_past': True,
    'wait_for_downstream': True,
    'execution_timeout': timedelta(minutes=3),
    'email': ['123456789@qq.com'],
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

#将本地时间转换为utc时间，再设置为start_date
tz = pytz.timezone('Asia/Shanghai')
dt = datetime(2018, 7, 26, 12, 20, tzinfo=tz)
utc_dt = dt.astimezone(pytz.utc).replace(tzinfo=None)

os.environ['AIRFLOW_CONN_HTTP_TEST']='http://localhost:9090'

dag = DAG(
    'bm01',
    default_args=default_args,
    description='my DAG',
    schedule_interval='*/2 * * * *',
    start_date=utc_dt
)

#通过SimpleHttpOperator定义http任务
task1 = SimpleHttpOperator(
    task_id='get_op1',
    http_conn_id='http_test',
    method='GET',
    endpoint='test1',
    data={},
    headers={},
    dag=dag)

task2 = SimpleHttpOperator(
    task_id='get_op2',
    http_conn_id='http_test',
    method='GET',
    endpoint='test2',
    data={},
    headers={},
    dag=dag)

task1 >> task2



```

##### 4.4 crontab 语法

`crontab`格式如下所示:

```
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of week (0 - 6) (Sunday to Saturday;
# │ │ │ │ │                                       7 is also Sunday on some systems)
# │ │ │ │ │
# │ │ │ │ │
# * * * * *  command to execute


```

<table><thead><tr><th>域</th><th>是否必须</th><th>取值范围</th><th>可用特殊符号</th><th>备注</th></tr></thead><tbody><tr><td>Minutes</td><td>Yes</td><td>0–59</td><td><code>*</code> <code>,</code> <code>-</code></td><td></td></tr><tr><td>Hours</td><td>Yes</td><td>0–23</td><td><code>*</code> <code>,</code> <code>-</code></td><td></td></tr><tr><td>Day of month</td><td>Yes</td><td>1–31</td><td><code>*</code> <code>,</code> <code>-</code> <code>?</code> <code>L</code> <code>W</code></td><td><code>?</code> <code>L</code> <code>W</code>部分实现可用</td></tr><tr><td>Month</td><td>Yes</td><td>1–12 or JAN–DEC</td><td><code>*</code> <code>,</code> <code>-</code></td><td></td></tr><tr><td>Day of week</td><td>Yes</td><td>0–6 or SUN–SAT</td><td><code>*</code> <code>,</code> <code>-</code> <code>?</code> <code>L</code> <code>#</code></td><td><code>?</code> <code>L</code> <code>W</code> 部分实现可用</td></tr><tr><td>Year</td><td>No</td><td>1970–2099</td><td><code>*</code> <code>,</code> <code>-</code></td><td>标准实现里无这一项</td></tr></tbody></table>

特殊符号功能说明：

**逗号 (`,`)**  
​ 逗号用于分隔一个列表里的元素，比如 "MON,WED,FRI" 在第五域 (day of week) 表示 Mondays, Wednesdays and Fridays。

**连字符 (`-`)**  
​ 连字符用于表示范围，比如 2000–2010 表示 2000 到 2010 之间的每年，包括这两年 (闭区间)。

**百分号 (`%`)**  
​ 用于命令 (command) 中的格式化

**L**  
​ 表示`last`，最后一个，比如第五域，`5L`表示当月最后一个星期五

**W**  
​ `W`表示 weekday（Monday-Friday），指离指定日期附近的工作日，比如第三域设置为`15L` ，这表示临近当月 15 附近的工作日，假如 15 号是星期六，那么定时器会在 14 号执行，如果 15 号是星期天，那么定时器会在 16 号执行，也就是说只会在离指定日期最近的那天执行。

**井号`#`**  
​ `#`用于第五域（day of week），# 后面跟着一个 1~5 之间的数字，这个用于表示第几个星期，比如`5#3`表示第三个星期五

**?**  
​ 在有些实现里面，`？`与`*`的功能相同，还有一些实现里面`?`表示 cron 的启动时间，比如 当 cron 服务在 8:25am 启动，则`? ? * * * *`会更新为`25 8 * * * *`, 直到下一次 cron 服务重新启动，定时器会再次更新。

**/**  
​ `/`一般与`*`组合使用，后面跟着一个数字，表示频率，比如在第一域 (Minutes) 中`*/5`表示每 5 分钟，是普通列表表示 5,10,15,20,25,30,35,40,45,50,55,00 的缩写

参考链接：  
[https://segmentfault.com/a/1190000012803744?utm_source=tuicool&utm_medium=referral](https://segmentfault.com/a/1190000012803744?utm_source=tuicool&utm_medium=referral)

[https://en.wikipedia.org/wiki/Cron](https://en.wikipedia.org/wiki/Cron)
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/lyhtbc/p/airflow-spark-schedule.html)

之前试用了 azkaban 一小段时间，虽然上手快速方便，但是功能还是太简单，不够灵活。  
Airflow 使用代码来管理任务，这样应该是最灵活的，决定试一下。

我是 python 零基础，在使用 airflow 的过程中可谓吃尽了苦头。。好歹最后实现所有要求，两三周的时间没有白费  
![](https://img2020.cnblogs.com/blog/302941/202005/302941-20200524084234569-2023200232.jpg)  

### 看完这篇文章，可以达到如下目标：

1.  安装 airflow
2.  如何修改界面右上角显示的时间到当前时区
3.  如何添加任务
4.  调试任务 python 代码
5.  如何启动 spark 任务
6.  如何限定任务同时执行的个数
7.  如何手动触发任务时传入参数
8.  如何在 airflow 界面上重新运行任务
9.  如何查看任务 log 及所有任务的运行记录
10.  如何在任务失败时发邮件（腾讯企业邮箱）
11.  如何在任务失败时发消息到企业微信

以下过程已经过去了有一段时间，当时记录的也不一定很全面，如果有的不能执行，请留言告知。  

### 安装 airflow

系统：Ubuntu 16  
python: 3.7  
airflow 版本：1.10.10  

#### 保持 pip3 到最新版本

pip3 install --upgrade pip  

#### 安装使用 pip3

切换到 root 用户执行： `pip3 install apache-airflow`  
你以为敲完这条命令就可以去把个妹或者撩个汉再回来就装好了，请坐下。

我碰到的错误：  
`Python.h not found`  
运行  
`sudo apt-get install python3.7-dev`

某些依赖版本不对：  
`ERROR: pendulum 1.4.4 has requirement python-dateutil<3.0.0.0,>=2.6.0.0, but you'll have python-dateutil 2.4.2 which is incompatible.`  
`ERROR: pandas 0.25.3 has requirement python-dateutil>=2.6.1, but you'll have python-dateutil 2.4.2 which is incompatible.`  
运行  
`pip install python-dateutil --upgrade`  
哪个包版本不对，更新哪个  

#### 数据库使用 mysql

相信你看这个文章的时候应该不会还没有尝试装过 airflow，所以 airflow.cfg 这个文件已经有了，在哪也很清楚

修改 airflow.cfg：  
`sql_alchemy_conn = mysql://airflow:password@jjh1810:3306/airflow`

使用 root 用户连接到 mysql：

```
create user 'airflow'@'%' identified by '123';
grant all privileges on airflow.* to 'airflow'@'%';
flush privileges;
set explicit_defaults_for_timestamp = 1; --这一行至关重要


```

再使用 airflow 用户登录 mysql：  
`create database airflow CHARACTER SET = utf8mb4;`  

#### 初始化数据库

`airflow initdb`

这时候会报 mysql 依赖问题，如：  
`No module named '_mysql'`

安装 python 的 mysql 依赖：  
[No module named MySQLdb](https://stackoverflow.com/questions/454854/no-module-named-mysqldb "No module named MySQLdb")  
[python3： mysql 错误：ModuleNotFoundError: No module named 'ConfigParser'](https://blog.csdn.net/junyilao/article/details/81948039 "python3： mysql错误：ModuleNotFoundError: No module named 'ConfigParser'")

这个时候终于可以启动 airflow 了：  
**启的时候不要使用 root 用户，回到普通用户**

```
airflow webserver -p 8080
airflow scheduler


```

### 如何修改界面右上角显示的时间到当前时区

相信应该所有人都会干这个事情：  
哟？airflow 里有个时区的配置，改了应该就好了  
`default_timezone = Asia/Shanghai`

然后去刷一下页面  
![](https://img2020.cnblogs.com/blog/302941/202005/302941-20200524101416016-841427688.gif)

还是 UTC 嘛，这配置骗人的吗？  
那么看这一篇文章吧：  
[Airflow 修改时区](https://oldecho.github.io/airflow/2019/01/21/Airflow-%E4%BF%AE%E6%94%B9%E6%97%B6%E5%8C%BA/ "Airflow 修改时区")  
这个不仅改掉了界面上的时区显示，而且 schedule 里的时间一起改掉了，绝对是居家旅行，编码调试必备方案。

**改的时候注意：** python 的代码是根据缩进来区别代码块的，所以拷代码的时候一定要注意缩进没有问题  

### 如何添加任务

在~/airflow 下创建 dags 文件夹，把. py 文件放入即可  
airflow 启动了一个叫 **DagFileProcessorManager** 的进程来扫描 dags 目录，一但有文件个数变更，或者内容变更都会很快被检测到  
这个进程有相应的 log 文件，可以提供一些文件处理错误信息  

### 调试任务 python 代码

#### 关闭 schedule

这个时候已经开始写任务的 python 代码了，对于 python 小白与刚开始接触 airflow 的小哥或老哥来说，简直就是痛不欲生  
有一个配置在调试的时候比较实用，就是关掉任务的 schudle，只有手动触发才会执行。  
把 dag 的 schedule_interval 设置为 None  
`schedule_interval=None`  

#### python 小白实用技巧

还有 python 代码里单引号和双引号是等价的，如果有引号嵌套可以分开使用，避免麻烦的转义，如：  
`hour = '{{ dag_run.conf["hour"] }}'`  

#### Jinja template

反正我第一眼看到这个东西，特别是官方教程里那一大块的模板文本的时候，心里只有一个字: WTF?!

```
templated_command = """
{% for i in range(5) %}
    echo "{{ ds }}"
    echo "{{ macros.ds_add(ds, 7)}}"
    echo "{{ params.my_param }}"
{% endfor %}
"""


```

其实也不是很复杂，这个玩意理解了以后还是比较方便的。除了在代码中使用普通的 python 变量或者 airflow 内置变量之外，很多时候在字符串中也需要使用 airflow 的内置变量会比较灵活和方便，Jinja template 提供的就是这个功能。

内置变量说明见：[Macros reference](https://airflow.apache.org/docs/stable/macros-ref.html "Macros reference")  

### 如何启动 spark 任务

airflow 是很强大很灵活的，官方提供了功能丰富的各种插件，各种 JobOperator。来，简单好用童叟无欺的 SparkSubmitOperator 了解一下？

我的需求很简单，可以提交任务到不同的 spark 集群。这样就要求不能使用机器默认的 hadoop 和 spark 环境变量，必须为每个任务指定独立的配置文件。不知道是不是有大牛一次性成功的，反正我是试了无数次，一句话在心里不停的重复：“这什么吊东西!”  
![](https://img2020.cnblogs.com/blog/302941/202005/302941-20200524111103535-452466845.gif)

小可愚钝，google 能搜出来的都看过了，怎么都不行，死活都不行，主要是环境变量不对。  

#### 调用 linux 脚本执行 spark-submit 是最灵活方便的办法

转念一想，还是传统的 spark 提交方式最好用啊，就是执行 sh 脚本利用 spark-submit 进行提交，这样 spark 就与 airflow 无关了，而且不管是环境变量还是参数配置都最灵活，传入参数也很方便。  
这样只要使用普通的 BashOperator 就可以了，而且 airflow 应该专注如何调度任务，而不是还要兼顾任务的配置，就算 SparkSubmitOperator 可以工作，也是使用 sh 脚本的方式更好。  

### 如何限定任务同时执行的个数

像 spark 任务通常使用的资源都会比较多，如果 dag 执行开始时间到当前时间间隔很长，或是暂停很长时间再开启，那么一开启的时候 schedule 会瞬间创建大量任务，提交到默认的 pool，这个 pool 默认的大小是 128。这样肯定是大家不希望看到的。

一个解决办法，为每个 spark 任务创建单独的 pool，大小设置为 1，这样一个 spark 任务一次就只能有一个在运行状态，后面都排队。

界面上操作：[Admin] -> [Pools]，slots 设为 1。  
然后在 spark task 的 operator 里添加参数：`pool='PoolName'`  

### 如何手动触发任务时传入参数

假设任务是每小时触发一次，处理 24 小时前的数据，也就是今天 8 点处理昨天 8 点这一个小时的数据。除了 schedule 自动执行的逻辑，我们还希望可以手动触发任务，并且指定某个小时重新处理。

**注：** 这个功能只有 1.10.10 才支持，就是在界面上点击 [Trigger DAG] 的时候可以填入参数（固定为 Json 格式）。

先来看一下最终的结果  
`hour='{{ dag_run.conf["hour"] if dag_run.conf["hour"] else (execution_date - macros.timedelta(hours=24)).strftime("%Y%m%d%H") }}'`  
这里使用了 Jinja template，通过 dag_run 对象可以获取界面上传入的参数，如果没有这个参数（也就是通过 schedule 触发），那么执行默认的逻辑（这里是 24 之前），并且格式化时间与界面输入保持一致。  

### 如何在 airflow 界面上重新运行任务

这个功能默认的 Executor 是不支持的，需要安装 CeleryExecutor，而 CeleryExecutor 需要一个存放消息的框架，这里我们选择 rabbitmq。

假定 rabbitmq 已经装好。  
安装请看官方文档：[Celery Executor](https://airflow.apache.org/docs/stable/executor/celery.html "Celery Executor")

配置

```
executor = CeleryExecutor
borker_url = amqp://user:password@host:port


```

**注：** 如果 rabbitmq 是集群模式，这里也是挑一台出来使用。指定所有节点我还没有配置成功，如果有会配置的，请留言告知。

如何在界面上重跑任务呢？  
界面上点击 dag 进入 dag 管理界面，点击 [Tree View]。  
Task 每次运行都会用一个小方块来表示，点击小方块，再点击 [Run] 按钮就可以了。

**注：** Tree View 这里最多只显示固定数量的历史记录，如果再早的时间只能通过点击 [Trigger DAG] 再指定参数运行。  

#### 任务运行时间的问题

这里有一个关键的问题，在界面上点击 8 个小时以前任务执行，那么任务触发的时候，运行的是 8 个小时之前的时间，还是当前时间呢？

如果我们是通过之前的 hour 变量的来指定时间的，那任务运行的时间就是 8 个小时之前，任务当时触发的时间。为什么呢？  
我们在 Jinja template 里使用的变量 dag_run, execute_date 这个并不是运行时变量，每次 task 触发，相关的上下文信息都会存到数据库里。所以 8 个小时之后我们再重新运行 task 的时候，是从数据库中读取当时的上下文信息，而不是现在的信息。  

### 如何查看任务 log 及所有任务的运行记录

#### 查看所有任务的运行记录

1.  DAG 界面里的 [Graph View] -> 点击任务 -> [Task Instances]
2.  主菜单里的 [Browser] -> [Task Instances]  
    

#### 查看 log

这就比较简单了

1.  点击 [Tree View] 里的小方块，可以查看 log
2.  Task Intances 列表最后一列，也可以查看 log  
    

### 如何在任务失败时发邮件（腾讯企业邮箱）

首先 DAG 的 default_args 需要配置

```
'email':['name@mail.com'],
'email_on_failure': True


```

修改 airflow.cfg

```
smtp_host = smtp.exmail.qq.com
smtp_starttls = False
smtp_ssl = True
smtp_port = 465
smtp_mail_from = name@mail.com
smtp_user = name@mail.com
smtp_password = password


```

首先 smtp_ssl = True， smtp_port = 465 是一个重点。再次 smtp_mail_from 和 smtp_user 都使用同一个有效的邮箱地址。  

### 如何在任务失败时发消息到企业微信

有时候觉得发邮件可能还不够，想把失败消息发到企业微信，这样更能及时的发现问题。  

#### 添加企业微信依赖

airflow 官方支持钉钉的通知，企业微信通知就是根据这个改动而来，代码有两个 py 文件：[airflow 企业微信支持](https://github.com/furyegg/airflow/tree/master/dags "airflow企业微信支持")  
把这两个 py 文件放到 dags 目录，也就是和 dag 的 py 文件放在一起。

在企业微信群中创建机器人

1.  右键点击群
    
2.  选择 [Add Group Robot]，并创建
    
3.  获取机器人的 key：右键 [View Information]，可以得到一个 URL  
    `https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=xxxxxx-xx-xx`  
    这个 key 的值就是机器人的 ID
    
4.  在 airflow 中创建企业微信的连接：[主菜单] -> [Admin] -> [Connections]，配置填写：
    

```
Conn Id: wechat_robot
Conn Type: HTTP
Host: https://qyapi.weixin.qq.com
Password: 前面得到的key值，也就是机器人的ID


```

#### 在代码中使用

1.  代码中 import WechatOperator  
    `from wechat_operator import WechatOperator`
    
2.  创建 failure call 方法：
    

```
def failure_callback(context):
    dagConf = context['dag_run'].conf
    taskInst = context['task_instance']

    hour = None
    if 'hour' in dagConf:
        hour = dagConf['hour']
    else:
        hour = (taskInst.execution_date - timedelta(hours=24)).strftime('%Y%m%d%H')

    message = 'Airflow任务失败:\n' \
              'DAG: {}\n' \
              'Task: {}\n' \
              '时间: {}\n' \
        .format(taskInst.dag_id,
                taskInst.task_id,
                hour)
    return WechatOperator(
        task_id='wechat_task',
        wechat_conn_id='wechat_robot',
        message_type='text',
        message=message,
        at_all=True,
    ).execute(context)


```

这个代码应该还是很好懂的，主要是为了创建 WechatOperator 对象。  
有个逻辑来重新获取执行时间（这里必须使用代码，而不能直接使用 Jinja template），为的是在通知里面可以直接看到是哪个时间出错了。

3.  default_args 添加 failure callback 配置  
    `'on_failure_callbak': failure_callback`  
    

### 结束语

到这里，总算是搭建好一个可以正式投入生产使用的环境了。

Airflow 虽然很灵活，但是想真正满足生产需求，还是经历了不少痛苦。特别是要求会使用 python，加上 airflow 官方文档也不是很详细，这两点导致入门曲线太陡峭了。
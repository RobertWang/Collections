> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/cord/p/9397584.html)

airflow 常见问题的排查记录如下:
--------------------

#### 1，airflow 怎么批量`unpause`大量的 dag 任务

​ 普通少量任务可以通过命令`airflow unpause dag_id`命令来启动，或者在 web 界面点击启动按钮实现，但是当任务过多的时候，一个个任务去启动就比较麻烦。其实 dag 信息是存储在数据库中的，可以通过批量修改数据库信息来达到批量启动 dag 任务的效果。假如是用 mysql 作为`sql_alchemy_conn`，那么只需要登录 airflow 数据库，然后更新表 dag 的 is_paused 字段为 0 即可启动 dag 任务。

示例: `update dag set is_paused = 0 where dag_id like "benchmark%";`

#### 2，airflow 的 scheduler 进程在执行一个任务后就挂起进入假死状态

出现这个情况的一般原因是 scheduler 调度器生成了任务，但是无法发布出去。而日志中又没有什么错误信息。

可能原因是 Borker 连接依赖库没安装：  
如果是 redis 作为 broker 则执行`pip install apache‐airflow[redis]`  
如果是 rabbitmq 作为 broker 则执行`pip install apache-airflow[rabbitmq]`  
还有要排查 scheduler 节点是否能正常访问 rabbitmq。

#### 3，当定义的`dag`文件过多的时候，airflow 的 scheduler 节点运行效率缓慢

airflow 的 scheduler 默认是起两个线程，可以通过修改配置文件`airflow.cfg`改进:

```
[scheduler]
# The scheduler can run multiple threads in parallel to schedule dags.
# This defines how many threads will run.
#默认是2这里改为100
max_threads = 100


```

#### 4，airflow 日志级别更改

```
$ vi airflow.cfg


```

```
[core]
#logging_level = INFO
logging_level = WARNING


```

**NOTSET < DEBUG < INFO < WARNING < ERROR < CRITICAL**

如果把 log 的级别设置为 INFO， 那么小于 INFO 级别的日志都不输出， 大于等于 INFO 级别的日志都输出。也就是说，日志级别越高，打印的日志越不详细。默认日志级别为 WARNING。

**注意: 如果将`logging_level`改为`WARNING`或以上级别，则不仅仅是日志，命令行输出明细也会同样受到影响，也只会输出大于等于指定级别的信息，所以如果命令行输出信息不全且系统无错误日志输出，那么说明是日志级别过高导致的。**

#### 5，AirFlow: jinja2.exceptions.TemplateNotFound

​ 这是由于 airflow 使用了 jinja2 作为模板引擎导致的一个陷阱，当使用 bash 命令的时候，尾部必须加一个空格:

> *   Described [here](https://github.com/airbnb/airflow/issues/1017) : see below. You need to add a space after the script name in cases where you are directly calling a bash scripts in the `bash_command` attribute of `BashOperator` - this is because the Airflow tries to apply a Jinja template to it, which will fail.

```
t2 = BashOperator(
task_id='sleep',
bash_command="/home/batcher/test.sh", // This fails with `Jinja template not found` error
#bash_command="/home/batcher/test.sh ", // This works (has a space after)
dag=dag)


```

参考链接:

[https://stackoverflow.com/questions/42147514/templatenotfound-error-when-running-simple-airflow-bashoperator](https://stackoverflow.com/questions/42147514/templatenotfound-error-when-running-simple-airflow-bashoperator)

[https://cwiki.apache.org/confluence/display/AIRFLOW/Common+Pitfalls](https://cwiki.apache.org/confluence/display/AIRFLOW/Common+Pitfalls)

#### 6，AirFlow: Task is not able to be run

任务执行一段时间后突然无法执行，后台 worker 日志显示如下提示:

```
[2018-05-25 17:22:05,068] {jobs.py:2508} INFO - Task is not able to be run


```

查看任务对应的执行日志:

```
cat /home/py/airflow-home/logs/testBashOperator/print_date/2018-05-25T00:00:00/6.log
...
[2018-05-25 17:22:05,067] {models.py:1190} INFO - Dependencies not met for <TaskInstance: testBashOperator.print_date 2018-05-25 00:00:00 [success]>, 
dependency 'Task Instance State' FAILED: Task is in the 'success' state which is not a valid state for execution. The task must be cleared in order to be run.


```

根据错误提示，说明依赖任务状态失败，针对这种情况有两种解决办法:

*   使用 airflow run 运行 task 的时候指定忽略依赖 task:
    
    ```
    $ airflow run -A dag_id task_id execution_date
    
    
    ```
    
*   使用命令 airflow clear dag_id 进行任务清理:
    
    ```
    $ airflow clear -u testBashOperator
    
    
    ```
    

#### 7，CELERY: PRECONDITION_FAILED - inequivalent arg 'x-expires' for queue 'celery@xxxx.celery.pidbox' in vhost ''

在升级 celery 4.x 以后使用 rabbitmq 为 broker 运行任务抛出如下异常:

```
[2018-06-29 09:32:14,622: CRITICAL/MainProcess] Unrecoverable error: PreconditionFailed(406, "PRECONDITION_FAILED - inequivalent arg 'x-expires' for queue 'celery@PQ
SZ-L01395.celery.pidbox' in vhost '/': received the value '10000' of type 'signedint' but current is none", (50, 10), 'Queue.declare')
Traceback (most recent call last):
  File "c:\programdata\anaconda3\lib\site-packages\celery\worker\worker.py", line 205, in start
    self.blueprint.start(self)
.......
  File "c:\programdata\anaconda3\lib\site-packages\amqp\channel.py", line 277, in _on_close
    reply_code, reply_text, (class_id, method_id), ChannelError,
amqp.exceptions.PreconditionFailed: Queue.declare: (406) PRECONDITION_FAILED - inequivalent arg 'x-expires' for queue 'celery@PQSZ-L01395.celery.pidbox' in vhost '/'
: received the value '10000' of type 'signedint' but current is none


```

出现该错误的原因一般是因为 rabbitmq 的客户端和服务端参数不一致导致的，将其参数保持一致即可。

​ **比如这里提示是 x-expires 对应的 celery 中的配置是 control_queue_expires。因此只需要在配置文件中加上 control_queue_expires = None 即可**。

​ 在 celery 3.x 中是没有这两项配置的，在 4.x 中必须保证这两项配置的一致性，不然就会抛出如上的异常。

我这里遇到的了两个 rabbitmq 的配置与 celery 配置的映射关系如下表：

<table><thead><tr><th>rabbitmq</th><th>celery4.x</th></tr></thead><tbody><tr><td>x-expires</td><td>control_queue_expires</td></tr><tr><td>x-message-ttl</td><td>control_queue_ttl</td></tr></tbody></table>

#### 8，CELERY: The AMQP result backend is scheduled for deprecation in version 4.0 and removal in version v5.0.Please use RPC backend or a persistent backend

celery 升级到 4.x 之后运行抛出如下异常:

```
/anaconda/anaconda3/lib/python3.6/site-packages/celery/backends/amqp.py:67: CPendingDeprecationWarning: 
    The AMQP result backend is scheduled for deprecation in     version 4.0 and removal in version v5.0.     Please use RPC backend or a persistent backend.
  alternative='Please use RPC backend or a persistent backend.')


```

原因解析:  
在 celery 4.0 中 rabbitmq 配置 result_backbend 方式变了:  
以前是跟 broker 一样:  
result_backend = 'amqp://guest:guest@localhost:5672//'  
现在对应的是 rpc 配置:  
result_backend = 'rpc://'

参考链接:  
[http://docs.celeryproject.org/en/latest/userguide/configuration.html#std:setting-event_queue_prefix](http://docs.celeryproject.org/en/latest/userguide/configuration.html#std:setting-event_queue_prefix)

#### 9，CELERY: ValueError('not enough values to unpack (expected 3, got 0)',)

windows 上运行 celery 4.x 抛出以下错误:

```
[2018-07-02 10:54:17,516: ERROR/MainProcess] Task handler raised error: ValueError('not enough values to unpack (expected 3, got 0)',)
Traceback (most recent call last):
	......
    tasks, accept, hostname = _loc
ValueError: not enough values to unpack (expected 3, got 0)



```

celery 4.x 暂时不支持 windows 平台，如果为了调试目的的话，可以通过替换 celery 的线程池实现以达到在 windows 平台上运行的目的:

```
pip install eventlet


```

```
celery -A <module> worker -l info -P eventlet


```

参考链接:

[https://stackoverflow.com/questions/45744992/celery-raises-valueerror-not-enough-values-to-unpack](https://stackoverflow.com/questions/45744992/celery-raises-valueerror-not-enough-values-to-unpack)

[https://blog.csdn.net/qq_30242609/article/details/79047660](https://blog.csdn.net/qq_30242609/article/details/79047660)

#### 10，Airflow: ERROR - 'DisabledBackend' object has no attribute '_get_task_meta_for'

airflow 运行中抛出以下异常:

```
Traceback (most recent call last):
  File "/anaconda/anaconda3/lib/python3.6/site-packages/airflow/executors/celery_executor.py", line 83, in sync
......
    return self._maybe_set_cache(self.backend.get_task_meta(self.id))
  File "/anaconda/anaconda3/lib/python3.6/site-packages/celery/backends/base.py", line 307, in get_task_meta
    meta = self._get_task_meta_for(task_id)
AttributeError: 'DisabledBackend' object has no attribute '_get_task_meta_for'
[2018-07-04 10:52:14,746] {celery_executor.py:101} ERROR - Error syncing the celery executor, ignoring it:
[2018-07-04 10:52:14,746] {celery_executor.py:102} ERROR - 'DisabledBackend' object has no attribute '_get_task_meta_for'


```

这种错误有两种可能原因:

1.  CELERY_RESULT_BACKEND 属性没有配置或者配置错误;
2.  celery 版本太低，比如 airflow 1.9.0 要使用 celery4.x，所以检查 celery 版本，保持版本兼容;

#### 11，airflow.exceptions.AirflowException dag_id could not be found xxxx. Either the dag did not exist or it failed to parse

查看 worker 日志 `airflow-worker.err`

```
airflow.exceptions.AirflowException: dag_id could not be found: bmhttp. Either the dag did not exist or it failed to parse.
[2018-07-31 17:37:34,191: ERROR/ForkPoolWorker-6] Task airflow.executors.celery_executor.execute_command[181c78d0-242c-4265-aabe-11d04887f44a] raised unexpected: AirflowException('Celery command failed',)
Traceback (most recent call last):
  File "/anaconda/anaconda3/lib/python3.6/site-packages/airflow/executors/celery_executor.py", line 52, in execute_command
    subprocess.check_call(command, shell=True)
  File "/anaconda/anaconda3/lib/python3.6/subprocess.py", line 291, in check_call
    raise CalledProcessError(retcode, cmd)
subprocess.CalledProcessError: Command 'airflow run bmhttp get_op1 2018-07-26T06:28:00 --local -sd /home/ignite/airflow/dags/BenchMark01.py' returned non-zero exit status 1.


```

​ 通过异常日志中的`Command`信息得知, 调度节点在生成任务消息的时候同时也指定了要执行的脚本的路径 (通过 ds 参数指定)，也就是说调度节点(scheduler) 和工作节点 (worker) 相应的 dag 脚本文件必须置于相同的路径下面，不然就会出现以上错误。

参考链接:

[https://stackoverflow.com/questions/43235130/airflow-dag-id-could-not-be-found](https://stackoverflow.com/questions/43235130/airflow-dag-id-could-not-be-found)

#### 12，airlfow 的 REST API 调用返回 `Airflow 404 = lots of circles`

​ 出现这个错误的原因是因为 URL 中未提供`origin`参数，这个参数用于重定向，例如调用 airflow 的`/run`接口，可用示例如下所示:

[http://localhost:8080/admin/airflow/run?dag_id=example_hello_world_dag&task_id=sleep_task&execution_date=20180807&ignore_all_deps=true&origin=/admin](http://localhost:8080/admin/airflow/run?dag_id=example_hello_world_dag&task_id=sleep_task&execution_date=20180807&ignore_all_deps=true&origin=/admin)
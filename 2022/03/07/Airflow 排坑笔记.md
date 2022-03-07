> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/336458279)

此前提过要分享一些 数据科学领域 好用好玩的框架与工具，现在正式来填坑。

第一期的主角——Airflow。

![](https://pic4.zhimg.com/v2-7b0f3d1067ab4067c08633aa280d24ff_b.jpg)

相信做过数据开发或者运维的人，对 crontab 这个 linux 系统定时调度功能都应该是又爱又恨，爱它简洁明了，恨它功能单一：

1.  缺少任务依赖
2.  无任务执行状态
3.  需要手工进行日志管理
4.  无历史执行情况记录：包括状态、时间等
5.  纯命令行，无可视化交互界面
6.  单机执行

于是也诞生了诸多开源调度系统来补足这方面缺憾，airflow 就是其中之一。

Airflow 是什么：

```
Airflow是一个可编程的调度和监控的工作流平台，基于有向无环图(DAG)，能灵活地定义有依赖的任务组。
Airflow同时提供了丰富的命令行工具用于系统管控、友好的web管理界面提供调度任务管控，方便对整个系统的监控、运维和管理。
Airflow已于2019进入了Apache顶级项目。

```

简而言之，Airflow 是一款强大灵活可扩展的调度系统。

本文作为一个排坑笔记，将不涉及安装、入门教学的介绍（这些内容强烈建议读官方文档），主要笔墨会围绕以下内容进行展开：

1.  **常见问题排坑（持续更新）**
2.  **源码阅读**
3.  **相关资源链接**

【一】、常见问题排坑
----------

### 一、任务对象概念梳理

使用 airflow 前，4 个最基本的 airflow 任务对象概念不能混淆：

1.  DAG：有向无环图 (Directed Acyclic Graph)，是将所有需要运行的 Task 按照依赖组合起来，描述的是所有 Task 执行的顺序和依赖关系。  
    
2.  Operator：可以用于定义具体任务的算子，如 BashOperator 对应 bash 命令执行，PythonOperator 对应 Python 函数执行，EmailOperator 对应发送邮件等。  
    
3.  Task：具体某一时刻要执行的任务实例，是 Operator 的实例化  
    
4.  Task Instance：可以理解为是 Task 再进一步的实例化。因为单次的 Task 有失败的可能，因而会被重复调度，每次 Task 的运行就是不同的 task instance，并有不同的执行状态，包括 "running", “success”, “failed”, “skipped”, "up for retry" 等。  
    

### 二、UTC 时间问题

前端页面、日志、以及数据库中的任务元数据，默认都是 UTC 时间，即使修改了系统配置项，也还是有部分显示依然是 UTC 时间。官方也阐述了其[原因](https://link.zhihu.com/?target=https%3A//airflow.apache.org/docs/apache-airflow/stable/timezone.html)，主要考虑点是本地时间机制无法解决夏令时切换问题。

至于具体的解决方案

1.  首先可以明确的是：绝不建议改源代码。源码中有 200 多处 datetime.dastetime.utcnow()，且修改后不能快速地随版本更新
2.  按官方的建议，可以在配置文件或者系统变量修改 timezone，当然其最终效果仅仅是 scheduler 在检查 dag 是否该执行时使用本地时间，其余的时间标记依旧为 utc 时间。比如喂给任务的时间宏变量 execution_date、ts 等，均维持为 UTC 时间。
3.  如果想要完全转化为本地时间，建议使用 python wrapper，可以选择将时间宏变量转化为本地时间（wrapper 代码如下），也可以给 context 额外添加几个变量如 local_start_time, local_end_time  
    

```
import pendulum
from functools import wraps

_local_timezone = pendulum.timezone('Asia/Shanghai')

def tz_func_wrapper(func):
    tz_keys = ['execution_date', ]

    @wraps(func)
    def wrapper(*args, **kwargs):
        for tz_key in tz_keys:
            tz_value = kwargs.get(tz_key)
            if tz_value is not None:
                if isinstance(tz_value, pendulum.pendulum.Pendulum):
                    native_value = tz_value.astimezone(_local_timezone)._datetime.replace(tzinfo=None)
                    kwargs[tz_key] = native_value
        result = func(*args, **kwargs)
        return result

    return wrapper

```

### 三、逻辑执行时间

1.  一个任务被执行后，会被标记上一个 execution_date，即上文代码中看到的这个变量。然而，要注意的是，这个 execution_date 只是逻辑上的执行时间。对于一个日级别 dag，假设它在 2020 年 1 月 2 日凌晨执行，但是 airflow 系统里，却会把它的 execution_date 标记为 2020 年 1 月 1 日凌晨，即实际上的执行时间是 execution_date + schedule_interval。同样的情况也发生在 start_date 这个参数上，即实际情况下，第一个执行的 dag run 的实际执行日期，并不是 start_date， 而是 start_date + schedule_interval  
    
2.  为什么要这么处理呢？  
    
3.  这算是历史习惯积累下的产物，在批处理时代还是非常适用的。比如说，要在每天结束的时候，回看这一整天的交易数据，那么就相当于在次日 00:00 时刻，去回望昨天的数据，这个时候，dt 标记成昨天显然是更适合的，因为这些数据的确是在昨天产生的。  
    
4.  但是在一些准实时的环境中，这样的逻辑执行时间就不是那么适用了。例如我们要每个一分钟去保存市场的一个截面数据，如果这个时候，明显使用实际执行时间作为时间戳才更加合适。如此情景下，补充两个时间宏变量能更好的方便程序编写：local_star_time 对应的 airflow 概念下的 execution_date，而 local_end_time 对应实际执行时间  
    （更新：2.2 版本后，airflow 已经正式添加了 data_interval_start 和 data_interval_end 两个宏变量来标记实际执行时间)  
    

分享一个简单的任务脚本，通过观察它的执行日志，应该能对 airflow 的时间宏变量机制有更直观的了解。

```
# tag: airflow, DAG, dag
import logging
import os
from airflow.models import DAG
import pendulum
from airflow.operators.python_operator import PythonOperator


def print_args(**kwargs):
    for k, v in kwargs.items():
        logging.info(f'{k}({type(v)}): {v}')


def print_hello(**kwargs):
    import sys
    logging.info('*' * 40)
    logging.info('hello world')
    print('interpreter python version: {}'.format(sys.version))
    os.system('echo "env python: $(which python)"')
    os.system('echo "home: ${HOME}"')
    os.system('echo "user: $(whoami)"')
    logging.info('*' * 40)


def operator_wrapper(func):
    return PythonOperator(python_callable=func, provide_context=True, task_id=func.__name__, )


default_args = {
    'owner': 'jesse',
    'depend_on_past': False,
    'start_date': pendulum.datetime(2020, 1, 1, tzinfo=pendulum.timezone('Asia/Shanghai')),
}

with DAG(
        dag_id='test_hello_world',
        schedule_interval='*/5 11-12 * * *',
        default_args=default_args,
        catchup=False,
) as dag:
    task1 = operator_wrapper(print_args)
    task2 = operator_wrapper(print_hello)
    task1 >> task2

```

![](https://pic3.zhimg.com/v2-dd190d7c47fb86c750ce8224ec159032_b.jpg)

### 四、Dag 撰写

（1）任务流是否需要是否补齐历史任务，有两个可选项

*   catchup，DAG 级别。首次提交 dag 时，会从设置的 start_date 补齐到现在，如果 catchup=False，则只会执行最新的一次任务
*   LastOnlyOperator，为 Task 级别。要注意，lastest_only 只是对这条分支上的任务群有效，对于整个 dag，run 还是会从 start_date 每隔 schedule_interval 时间运行

（2）是否依赖历史执行任务

1.  depend_on_past=True 时, 只有上一个 DagRun 被成功执行后，下一个 DagRun 才能被执行。也就是说，depend_on_past 关注于整个 DAG 过去执行的状况，一旦某一次失败了，后面就不再执行了（避免有一些业务脚本，会有严格时间顺序的要求）
2.  trigger_rule=all_success 时，只有前置的任务实例 tis 全面被执行成功的时候，下一个 task instance 才会被送到 worker 中进行运行。
3.  简而言之 depends_on_past {True, False} 是 dag 级别的触发规则；而 trigger_rule {all_success, all_failed, all_done, one_failed, one_success, dummy} 是 operator 级别的触发规则

（3）任务手动触发：

任务总有失败的时候，通常我们可以在交互式的 webserver 界面进行操作，但是该界面操作时，execution_date 是无法指定的。这对于依赖时间宏变量的任务来说，会直接导致任务不可控。而这个时候，就要使用命令行 trigger_dag、backfill 这两个工具

1.  trigger_dag，在固定的某个时间点执行 dag。特别的，如果历史上已经在该时间点成功执行过，则不会再次重复执行，这个特性能很好地避免重复执行带来的数据错误。
2.  backfill 则是批量版 trigger_dag，在某个固定时间段内执行。
3.  另外，提一下 test 这个命令。test 也是能执行指定的任务，不同的是，trigger_dag、backfill 将会把任务执行的元数据写到底层数据库中，而 test 不会。一个简单的例子就是，你可以使用 test 在相同的时间内重复执行，但是其他两个命令会直接跳过。

（4）脚本化：

**撰写 DAG 的 python 脚本时，不要在 global 层面导入其他较重的模块，如 pandas、sklearn 等，否则在 schedule 时非常耗时，如果必须要导入，强烈建议放在子函数下。**

1.  因为默认情况下，airflow 每个几秒会扫描所有的 dag 文件，对每个 dag 文件会专门 spawn 一个新的子进程，去执行一遍 dag 脚本，从而解析出其中的 dag 实例。
2.  由于 spawn 的新进程此前并未载入过 pandas、sklearn 这样的第三方库，于是在扫描的时候将不得不花时间和资源去载入这些第三方库到内存。
3.  由于一些重型库导入可能需要若干秒，一旦库增多、或者 dag 脚本增多、加上每隔几秒就要做一次扫描工作，服务器的大量资源将被挤占掉。

（更新：2.0 版本以后，扫描机制已优化，将会根据文件的最新修改时间进行解析，以上困扰不再存在）

### 五、运维

修改默认配置：某些默认配置非常耗费 CPU 资源，可以改以下几点

1.  减少 dags 文件夹轮询次数（很重要） ，参见 [stackoverflow 的这个问题提问](https://link.zhihu.com/?target=https%3A//stackoverflow.com/questions/42419834/airbnb-airflow-using-all-system-resources)

*   AIRFLOW__SCHEDULER__MIN_FILE_PROCESS_INTERVAL=20

2. 减少 worker 更新频率

*   AIRFLOW__WEBSERVER__WORKERS=2 # 2 * NUM_CPU_CORES + 1
*   AIRFLOW__WEBSERVER__WORKER_REFRESH_INTERVAL=1800 # Restart workers every 30min instead of 30seconds
*   AIRFLOW__WEBSERVER__WEB_SERVER_WORKER_TIMEOUT=300 #Kill workers if they don't start within 5min instead of 2min

3. 其他：

*   AIRFLOW__CORE__DEFAULT_TIMEZONE=Asia/Shanghai # Change timezone to UTC+8

### 六、分布式方案

生产系统中，我们通常使用分布式部署的方式，实现系统的高可用，并充分利用集群的计算资源。

分布式部署时，通常会对接上 CI、CD 系统，实现任务的持续交付：

1.  任务提交至代码仓库，触发 CI/CD
2.  完成项目测试、打包后，将包、镜像等制品推到目标仓库
3.  将包含任务编排信息的 Dag 提交到集群配置文件的挂载目录
4.  airflow 在下一次轮训中感应到 Dag 脚本变动，进而更新数据库中的任务信息
5.  到任务触发时间点时，scheduler 节点将任务标记为待运行，交由 Executor 安排给空闲 Worker 执行
6.  如果任务是本地类型的，则在 Worker 节点直接运行（需要注意 worker 节点的环境配置）
7.  大部分任务将是远程类型的，任务将交由集群进行执行如 Docker Swarm/K8s
8.  集群将拉取镜像，生成容器，并执行给定的任务操作，完成后删除容器，结束本次任务运行。

![](https://pic3.zhimg.com/v2-2361cc34ee080b53ad8e7a9335365686_b.jpg)

【二】、源码阅读
--------

第二部分是针对 airflow 源码阅读的分享。

这部分其实是当年笔者踩坑、google 无果后，才强迫自己去看的。看完之后，确实是加深了对调度系统机制的理解，后续再写 dag 的时候，也都能避开潜在的各种坑。

当然也说明一下：

1.  如果只是要使用现有的功能，其实不需要如此深入，可以跳过以下内容，多花点时间看官方文档更合适。
2.  如果要对系统做一定的定制化，那么笔者的这些笔记应该会对你有所帮助

![](https://pic1.zhimg.com/v2-b6236b74c539f6dae10644143207af14_b.jpg)

### （1）airflow 核心组件

先从系统的组件介绍开始

在 airflow 中，最核心的是两个组件，`DagFileProcessorAgent` & `Executor`

一、DagFileProcessorAgent（图上半部分）

```
"""
Agent for DAG file processing. It is responsible for all DAG parsing
related jobs in scheduler process. Mainly it can spin up DagFileProcessorManager
in a subprocess, collect DAG parsing results from it and communicate
signal/DAG parsing stat with it.

This class runs in the main `airflow scheduler` process.
"""

```

简单翻译来说，这个组件的主要负责就是解析工作目录下的所有 dag 文件。其工作原理大概是（看图理解会更易消化）：

1.  在 Agent 对象中，包含这一个主动类 DagFileProcessorManager 的实例，这个实例将会定时 spawn 出多个 DagFileProcess 进程
2.  DagFileProcess 进程中的主要工作是，生成一个 DagFileProcessor 对象，该对象负责_单个_具体的 dag 解析工作，解析工作过程中，如果发现任务的下次执行时间正好触达了，那么任务实例就会被推回到 Agent 的队列之中等待执行。

二、 Executor（图下半部分）

Executor，即任务执行器，顾名思义，其职责就是将被触发的任务实际执行掉。Executor 有多种类型可供使用 LocalExecutor, CeleryExecutor, MesosExecutor, YarnExecutor，我们这里重点用 LocalExecutor 来说明其大致的工作原理：

1.  Executor 解析被触发的任务，并发送到中间件上 Parallelism 上。
2.  主动类 Worker 将从中间件的队列中拉取任务
3.  任务将在新的子进程 SubProcess 执行，保证异常的任务不会影响到整个系统。
4.  任务执行完之后，通过进程间的通信机制，会将执行结果和状态返回给 Executor 的 result_queue 中，如果没有执行成功并且有失败重启机制，那么 Executor 就会安排再次执行任务。

### （2）DAG 生命周期

以上是通过 DagFileProcessorAgent 和 Executor 大体了解了调度执行的核心机制，下面我们将按照 Dag 对象的生命周期顺序，来展示系统实际的工作流情况

（按图从上至下，蓝线标识）

1.  DagFileProcessor(DFP)，解析包含 dag 的 py 文件，并转换成内存中 dag 对象，主要步骤为

1.  解析 py 文件，收集其中包含的符合规范的 Dag 对象，主要由 Dag.collect_dags 完成
2.  处理 Dag 对象，有 processor.process_dags 完成

1.  查看 dag 是否符合触发条件，是的话，生成 DagRun 对象
2.  从 DagRun 对象中抽取出其包含的 TaskInstance 实例
3.  检查 TaskInstance 依赖条件是否满足，若是，则标记为 SCHEDULED、同步到数据库（DummyOperator 除外，直接标记为 SUCCESS）

3.  DagFileProcessorManager(DFPM)，顾名思义管理着若干 DFP，当然实际上并不是直接管理 DFP，而是管理着若干个 DFPP 进程，每个进程中管理一个 DFP 对象，一个 DFP 对象处理一个文件，相当于说，每个 dag 文件都是用一个独立的 DFPP 进程中进行处理的。  
    
4.  DFPP 处理完成后，DFPM 就会将需要执行的 DAG 对象，通过进程间的通信管道，从 DFPP 发回到 Agent  
    
5.  Agent 根据 dag 的 id 号从数据库找到可执行的任务，即之前已经被标记为 scheduled 的 TaskIntance，并且将这些任务送到 Executor 执行  
    
6.  Executor 也是一个主动类，他会定时轮训被缓存的任务字典 queue_tasks，一旦有任务，就会将其解析成 SimpleTaskInstance，并发送到中间件 Parallelism 的队列之中  
    
7.  订阅者中间件的若干个 Worker 就会从中间件中取出任务，并新起一个子进程执行命令。执行完的状态，将会返回到 Executor 的 result_queue，等待下一次心跳时被确认完后，就正式完成 DAG 的一轮处理  
    

### （3）潜在瓶颈

阅读完代码以后，除了感叹到 airflow 架构的微妙细节，也能感受到 airflow 的一大瓶颈：起了大量进程进行工作处理，即便是 dag 文件解析，也使用了单独的进程进行处理，而且仅仅解析一次后就将进程回收了。解析工作、调度的工作全部都是 master 单机完成，这种不断起进程、回收进程的操作也是非常吃资源的，因此可以预想到，当大量任务运行时，调度的性能将不可避免地下降。

另外提一句，如果对性能有较高的要求，很推荐去了解另一个由国人贡献的开源调度系统 [dolphin-scheduler](https://link.zhihu.com/?target=https%3A//dolphinscheduler.apache.org/zh-cn/)，性能很强大，也是 apache 孵化项目。

【三】、相关资源链接
----------

最后，是笔者最使用 airflow 过程中主要参考的一些资源，独乐乐不如众乐乐，也欢迎小伙伴分享相关资源给大伙一起用用

1.  **官网文档，必读，至少要通读一遍！：[https://airflow.apache.org/docs](https://link.zhihu.com/?target=https%3A//airflow.apache.org/docs)**
2.  **Github 的 Awesome 项目集合库，包含了部署方案、教程、书籍、最佳实践等等，内容全面，笔者强推其中的最佳实践部分，干货很多：[awesome-apache-airflow](https://link.zhihu.com/?target=https%3A//github.com/jghoman/awesome-apache-airflow%23best-practices-lessons-learned-and-cool-use-cases)**
3.  使用 airflow 做 ETL 的范例代码：[gtoonstra/etl-with-airflow](https://link.zhihu.com/?target=https%3A//github.com/gtoonstra/etl-with-airflow)
4.  官网文档的中文翻译，版本有点旧，实在阅读不了英文文档再选择这个：[https://airflow.apachecn.org/](https://link.zhihu.com/?target=https%3A//airflow.apachecn.org/)
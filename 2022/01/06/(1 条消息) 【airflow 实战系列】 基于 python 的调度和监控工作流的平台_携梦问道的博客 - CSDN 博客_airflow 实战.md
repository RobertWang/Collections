> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_41797451/article/details/89091161)

简介
==

airflow 是一个使用 [python](https://so.csdn.net/so/search?q=python) 语言编写的 data pipeline 调度和监控工作流的平台。Airflow 被 Airbnb 内部用来创建、监控和调整数据管道。任何工作流都可以在这个使用 Python 来编写的平台上运行。

这个平台拥有和 Hive、Presto、MySQL、HDFS、Postgres 和 S3 交互的能力，并且提供了钩子使得系统拥有很好地扩展性。除了一个命令行界面，该工具还提供了一个基于 Web 的用户界面让您可以可视化管道的依赖关系、监控进度、触发任务等。

传统 Workflow 通常使用 TextFiles(json,xml/etc) 来定义 DAG, 然后 Scheduler 解析这些 DAG 文件形成具体的 TaskObject 执行；Airflow 没这么干，它直接用 Python 写 DAGdefinition, 一下子突破了文本文件表达能力的局限，定义 DAG 变得简单。

Airflow 的架构
-----------

在一个可扩展的生产环境中，Airflow 含有以下组件：

*   一个元数据库（MySQL 或 Postgres）
    
*   一组 Airflow 工作节点
    
*   一个调节器（Redis 或 RabbitMQ）
    
*   一个 Airflow Web 服务器
    

所有这些组件可以在一个机器上随意扩展运行。如果使用 LocalExcuter 来适度的安装则可以获得相当多的额外性能。

![](https://blog-10039692.file.myqcloud.com/1493196246754_6904_1493196247073.jpg)

![](https://blog-10039692.file.myqcloud.com/1493196268009_4301_1493196268290.jpg)

优点
--

*   python 脚本实现 DAG，非常容易扩展
    
*   工作流依赖可视化
    
*   no XML
    
*   可测试
    
*   可作为 crontab 的替代
    
*   可实现复杂的依赖规则
    
*   Pools
    
*   CLI 和 Web UI
    

功能简介
----

![](https://blog-10039692.file.myqcloud.com/1493196328613_4508_1493196328721.jpg)

常见命令
----

*   initdb，初始化元数据 DB，元数据包括了 DAG 本身的信息、运行信息等；
    
*   resetdb，清空元数据 DB；
    
*   `list_dags`，列出所有 DAG；
    
*   `list_tasks`，列出某 DAG 的所有 task；
    
*   test，测试某 task 的运行状况；
    
*   backfill，测试某 DAG 在设定的日期区间的运行状况；
    
*   webserver，开启 webserver 服务；
    
*   scheduler，用于监控与触发 DAG。
    

ETL
---

ETL，是英文 Extract-Transform-Load 的缩写，用来描述将数据从来源端经过抽取（extract）、转换（transform）、加载（load）至目的端的过程。ETL 一词较常用在[数据仓库](http://baike.baidu.com/item/%E6%95%B0%E6%8D%AE%E4%BB%93%E5%BA%93)，但其对象并不限于数据仓库。

Airflow 设计时，只是为了很好的处理 ETL 任务而已，但是其精良的设计，正好可以用来解决任务的各种依赖问题。

任务依赖
----

通常，在一个运维系统，数据分析系统，或测试系统等大型系统中，我们会有各种各样的依赖需求。比如：

*   时间依赖：任务需要等待某一个时间点触发
    
*   外部系统依赖：任务依赖 Mysql 中的数据，HDFS 中的数据等等，这些不同的外部系统需要调用接口去访问
    
*   机器依赖：任务的执行只能在特定的某一台机器的环境中，可能这台机器内存比较大，也可能只有那台机器上有特殊的库文件
    
*   任务间依赖：任务 A 需要在任务 B 完成后启动，两个任务互相间会产生影响
    
*   资源依赖：任务消耗资源非常多，使用同一个资源的任务需要被限制，比如跑个数据转换任务要 10 个 G，机器一共就 30 个 G，最多只能跑两个，我希望类似的任务排个队
    
*   权限依赖：某种任务只能由某个权限的用户启动
    

也许大家会觉得这些是在任务程序中的逻辑需要处理的部分，但是我认为，这些逻辑可以抽象为任务控制逻辑的部分，和实际任务执行逻辑解耦合。

如何理解 Crontab
------------

现在让我们来看下最常用的依赖管理系统，Crontab

在各种系统中，总有些定时任务需要处理，每当在这个时候，我们第一个想到的总是 crontab。

确实，crontab 可以很好的处理定时执行任务的需求，但是对于 crontab 来说，执行任务，只是调用一个程序如此简单，而程序中的各种逻辑都不属于 crontab 的管辖范围（很好的遵循了 KISS）

所以我们可以抽象的认为：

crontab 是一种依赖管理系统，而且只管理时间上的依赖。

Airflow 的处理依赖的方式
----------------

Airflow 的核心概念，是 DAG(有向无环图)，DAG 由一个或多个 TASK 组成，而这个 DAG 正是解决了上文所说的任务间依赖。Task A 执行完成后才能执行 Task B，多个 Task 之间的依赖关系可以很好的用 DAG 表示完善

Airflow 完整的支持 crontab 表达式，也支持直接使用 python 的 datatime 表述时间，还可以用 datatime 的 delta 表述时间差。这样可以解决任务的时间依赖问题。

Airflow 在 CeleryExecuter 下可以使用不同的用户启动 Worker，不同的 Worker 监听不同的 Queue，这样可以解决用户权限依赖问题。Worker 也可以启动在多个不同的机器上，解决机器依赖的问题。

Airflow 可以为任意一个 Task 指定一个抽象的 Pool，每个 Pool 可以指定一个 Slot 数。每当一个 Task 启动时，就占用一个 Slot，当 Slot 数占满时，其余的任务就处于等待状态。这样就解决了资源依赖问题。

Airflow 中有 Hook 机制 (其实我觉得不应该叫 Hook)，作用时建立一个与外部数据系统之间的连接，比如 Mysql，HDFS，本地文件系统(文件系统也被认为是外部系统) 等，通过拓展 Hook 能够接入任意的外部系统的接口进行连接，这样就解决的外部系统依赖问题。

参考
--

[http://wingerted.com/2017/02/20/introduce-to-airflow/](http://wingerted.com/2017/02/20/introduce-to-airflow/)

[https://www.youtube.com/watch?v=cHATHSB_450](https://www.youtube.com/watch?v=cHATHSB_450)

[https://www.youtube.com/watch?v=Pr0FrvIIfTU](https://www.youtube.com/watch?v=Pr0FrvIIfTU)

文章来源于 [https://www.cnblogs.com/qyun/p/6791857.html](https://www.cnblogs.com/qyun/p/6791857.html)
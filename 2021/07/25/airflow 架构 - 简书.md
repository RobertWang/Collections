> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/8c358be41ff0)

架构
--

airflow 的架构如下图所示：  

![](http://upload-images.jianshu.io/upload_images/10848392-5faa5f1edfadc358.png) airflow 架构

**airflow 组成部分：**

*   **Workers**: 执行指定的任务（tasks）。
*   **Scheduler（调度器）**：负责添加必要的任务（tasks）到任务队列。
*   **Web server（web 服务端）**：HTTP 服务器提供 DAG/task 的状态信息。
*   **数据库**：存储 任务（tasks）、DAGs、 变量、连接等信息。
*   **Celery**：队列机制。Celery 由 Broker（代理）和 Result backend（结果后端）两个组件组成。Broker 负责存储执行的命令。Result backend 存储完成执行命令的状态信息。

**各种组件之间的通信**  
[1] **Web server –> Workers** ： 获取任务执行日志。  
[2] **Web server –> DAG files** ： 展示 DAG 结构。  
[3] **Web server –> Database** ：获取任务状态。  
[4] **Workers –> DAG files** ：展示 DAG 结构和执行任务。  
[5] **Workers –> Database** ： 获取和存储连接配置信息、变量 XCOM。  
[6] **Workers –> Celery’s result backend** ： 存储任务执行信息。  
[7] **Workers –> Celery’s broker** ：存储执行的命令。  
[8] **Scheduler –> Database** ： 存储 DAG 运行信息和相关的任务。  
[9] **Scheduler –> DAG files** ： 展示 DAG 的结构和执行任务。  
[10] **Scheduler –> Celery’s result backend** ： 获取已经执行完的任务信息。  
[11] **Scheduler –> Celery’s broker** ： 把执行的命令发送给 Celery’s broker。

**参考：**  
_[https://airflow.apache.org/docs/stable/howto/executor/use-celery.html](https://links.jianshu.com/go?to=https%3A%2F%2Fairflow.apache.org%2Fdocs%2Fstable%2Fhowto%2Fexecutor%2Fuse-celery.html)_
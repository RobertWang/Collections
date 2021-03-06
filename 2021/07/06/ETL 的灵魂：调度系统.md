> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA5ODM5MDU3MA==&mid=2650873251&idx=2&sn=d8e974ca6bb62740b8b9426387e15f78&chksm=8b67fae6bc1073f0e27bf6a18b7f3be00ab6bdf551c01c25e15549752558643c610e5c7ad508&mpshare=1&scene=1&srcid=0705N5J6CgQm1XgMNg0PcSAj&sharer_sharetime=1625463514697&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> 最近有小伙伴聊说他们的调度系统经常出问题，领导要求大家人在哪电脑背到哪，家庭生活一地鸡毛……，其实我也有类似的经历，今天给大家分享一下做调度系统的一些经验！

最近有小伙伴聊说他们的调度系统经常出问题，领导要求大家人在哪电脑背到哪，家庭生活一地鸡毛……，其实我也有类似的经历，今天给大家分享一下做调度系统的一些经验！  

目前大数据平台经常会用来跑一些批任务，跑批处理当然就离不开定时任务。比如定时抽取业务数据库的数据，定时跑 hive/spark 任务，定时推送日报、月报指标数据。任务调度系统已经俨然成为了大数据处理平台不可或缺的一部分，**可以说是 ETL 任务的灵****魂**。

01

**原始任务调度**  

![](https://mmbiz.qpic.cn/mmbiz_png/z2DApiaibzMicibdsicgeyk3shhFGQnoog4OWqKdc72nmTUFEW4YgWlOFf0kaoK32iaCAibGIqT1ytxhjGzY1no0V0xDQ/640?wx_fmt=png)

记得第一次参与大数据平台从无到有的搭建，最开始任务调度就是用的 Crontab，分时日月周，各种任务脚本配置在一台主机上。Crontab 使用非常方便，配置也很简单。刚开始任务很少，用着还可以，每天起床巡检一下日志。随着任务越来越多，出现了任务不能在原来计划的时间完成，出现了上级任务跑完前，后面依赖的任务已经起来了，这时候没有数据，任务就会报错，或者两个任务并行跑了，出现了错误的结果。排查任务错误原因越来麻烦，各种任务的依赖关系越来越复杂，最后排查任务问题就行从一团乱麻中，一根一根梳理出每天麻绳。crontab 虽然简单，稳定，但是随着任务的增加和依赖关系越来越复杂，已经完全不能满足我们的需求了，这时候就需要建设自己的调度系统了。

02

**调度系统**  

调度系统，关注的首要重点是在正确的时间点启动正确的作业，确保作业按照正确的依赖关系及时准确的执行。资源的利用率通常不是第一关注要点，业务流程的正确性才是最重要的。（但是到随着业务的发展，ETL 任务越来越多，你会发现经常有任务因为资源问题没有按时启动！）

实际调度中，多个任务单元之间往往有着强依赖关系，上游任务执行并成功，下游任务才可以执行。比如上游任务 1 结束后拿到结果，下游任务 2、任务 3 需结合任务 1 的结果才能执行，因此下游任务的开始一定是在上游任务成功运行拿到结果之后才可以开始。而为了保证数据处理结果的准确性，就必须要求这些任务按照上下游依赖关系有序、高效的执行，最终确保能按时正常生成业务指标。  

![](https://mmbiz.qpic.cn/mmbiz_png/z2DApiaibzMicibdsicgeyk3shhFGQnoog4OWDgzfmNGUhoe4OyrXZG2vYrxxFNQhyHCWOQ1USQTdKxC1QvL67dfLTg/640?wx_fmt=png)

一款成熟易用，便于管理和维护的作业调度系统，需要和大量的周边组件对接，要处理或使用到包括：**血缘管理，权限控制，负载流控，监控报警，质量分析**等各种服务或事务。

03

**调度系统分类**

调度系统一般分为两类：定时分片类作业调度系统和 DAG 工作流类作业调度系统

**定时分片类作业调度系统**

这种功能定位的作业调度系统，其最早的需要来源和出发点往往是做一个分布式的 Crontab。

**核心：**

*   将一个大的任务拆成多个小任务分配到不同的服务器上执行， 难点在于要做到不漏，不重，保证负载平衡，节点崩溃时自动进行任务迁移等。
    
*   保证任务触发的强实时和可靠性
    

所以，负载均衡，弹性扩容，状态同步和失效转移通常是这类调度系统在架构设计时重点考虑的特性。

**DGA 工作流调度系统**

这一类系统的方向，重点定位于任务的调度依赖关系的正确处理，分片执行的逻辑通常不是系统关注的核心，或者不是系统核心流程的关键组成部分。

**核心：**

*   足够丰富和灵活的依赖触发机制：比如时间触发任务，依赖触发任务，混合触发任务
    
*   作业的计划，变更和执行流水的管理和同步
    
*   任务的优先级管理，业务隔离，权限管理等
    
*   各种特殊流程的处理，比如暂停任务，重刷历史数据，人工标注失败／成功，临时任务和周期任务的协同等
    
*   完备的监控报警通知机制
    

04

**几个调度系统**

### **Airflow**  

Apache Airflow 是一种功能强大的工具，可作为任务的有向无环图（DAG）编排、任务调度和任务监控的工作流工具。Airflow 在 DAG 中管理作业之间的执行依赖，并可以处理作业失败，重试和警报。开发人员可以编写 Python 代码以将数据转换为工作流中的操作。

![](https://mmbiz.qpic.cn/mmbiz_png/z2DApiaibzMic9kdGSPGiaia5azCULYFLVt2bLobEotr4dERmyO0jCL5iag9H7xStYgeHPIzwMWXmr8U7EHoa3aacqGw/640?wx_fmt=png)

主要有如下几种组件构成：

*   web server: 主要包括工作流配置，监控，管理等操作
    
*   scheduler: 工作流调度进程，触发工作流执行，状态更新等操作
    
*   消息队列：存放任务执行命令和任务执行状态报告
    
*   worker: 执行任务和汇报状态
    
*   mysql: 存放工作流，任务元数据信息
    

具体执行流程：

1.  scheduler 扫描 dag 文件存入数据库，判断是否触发执行
    
2.  到达触发执行时间的 dag, 生成 dag_run，task_instance 存入数据库
    
3.  发送执行任务命令到消息队列
    
4.  worker 从队列获取任务执行命令执行任务
    
5.  worker 汇报任务执行状态到消息队列
    
6.  schduler 获取任务执行状态，并做下一步操作
    
7.  schduler 根据状态更新数据库
    

### **Kettle**

将各个任务操作组件拖放到工作区，kettle 支持各种常见的数据转换。此外，用户可以将 Python，Java，JavaScript 和 SQL 中的自定义脚本拖放到画布上。kettle 可以接受许多文件类型作为输入，还可以通过 JDBC，ODBC 连接到 40 多个数据库，作为源或目标。社区版本是免费的，但提供的功能比付费版本少。

![](https://mmbiz.qpic.cn/mmbiz_jpg/z2DApiaibzMic9kdGSPGiaia5azCULYFLVt2bzRM60VquLHoNmYACt1vKHibkHfd326mHDBBjIxTDeOiaAoP8HyN9AZWQ/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz/z2DApiaibzMic9kdGSPGiaia5azCULYFLVt2bb9oXtesDLsHlqxsaKuMf2A2JwXiblnLpRmhz8d6cIhnA38GiaTtpLCIg/640?wx_fmt=jpeg)

### **XXL-JOB**

XXL-JOB 是一个分布式任务调度平台，其核心设计目标是开发迅速、学习简单、轻量级、易扩展。将调度行为抽象形成 “调度中心” 公共平台，而平台自身并不承担业务逻辑，“调度中心”负责发起调度请求；将任务抽象成分散的 JobHandler，交由 “执行器” 统一管理，“执行器”负责接收调度请求并执行对应的 JobHandler 中业务逻辑；因此，“调度”和 “任务” 两部分可以相互解耦，提高系统整体稳定性和扩展性。（后来才知道 XXL 是作者名字拼音首字母缩写）

![](https://mmbiz.qpic.cn/mmbiz_png/z2DApiaibzMic9kdGSPGiaia5azCULYFLVt2baJxps4eiaR1bGNN7aIjLF0ibZpoF71EK1bTeC1JxI8bZXOviasIHVdMUA/640?wx_fmt=png)

调度系统开源工具有很多，可以结合自己公司人员的熟悉程度和需求选择合适的进行改进。  

05

**如何自己开发一个调度系统**  

调度平台其实需要解决三个问题：**任务编排、任务执行和任务监控**。  

-------------------------------------

![](https://mmbiz.qpic.cn/mmbiz_png/z2DApiaibzMic9kdGSPGiaia5azCULYFLVt2bKmRh73Zu5I5GNGplsoLRxObKh7TicnW2SKiaFf7HCXh8rnJZ7Y9EE0DQ/640?wx_fmt=png)

*   **任务编排，**采用调用外部编排服务的方式，主要考虑的是编排需要根据业务的一些属性进行实现，所以将易变的业务部分从作业调度平台分离出去。如果后续有对编排逻辑进行调整和修改，都无需操作业务作业调度平台。
    
*   **任务排队，**支持多队列排队配置，后期根据不同类型的开发人员可以配置不同的队列和资源，比如面向不同的开发人员需要有不同的服务队列，面向不同的任务也需要有不同的队列优先级支持。通过队列来隔离调度，能够更好地满足具有不同需求的用户。不同队列的资源不同，合理的利用资源，达到业务价值最大化。
    
*   **任务****调度，**是对任务、以及属于该任务的一组子任务进行调度，为了简单可控起见，每个任务经过编排后会得到一组有序的任务列表，然后对每个任务进行调度。这里面，稍有点复杂的是，任务里还有子任务，子任务是一些处理组件，比如字段转换、数据抽取，子任务需要在上层任务中引用实现调度。任务是调度运行的基本单位。被调度运行的任务会发送到消息队列中，然后等待任务协调计算平台消费并运行任务，这时调度平台只需要等待任务运行完成的结果消息到达，然后对作业和任务的状态进行更新，根据实际状态确定下一次调度的任务。
    

调度平台设计中还需要注意以下几项：

1.  **调度运行的任务需要进行超时处理**，比如某个任务由于开发人员设计不合理导致运行时间过长，可以设置任务最大的执行时长，超过最大时长的任务需要及时 kill 掉，以免占用大量资源，影响正常的任务运行。
    
2.  **控制同时能够被调度的作业的数量**，集群资源是有限的，我们需要控制任务的并发量，后期任务上千上万后我们要及时调整任务的启动时间，避免同时启动大量的任务，减少调度资源和计算资源压力；
    
3.  **作业优先级控制**，每个业务都有一定的重要级别，我们要有限保障最重要的业务优先执行，优先给与调度资源分配。在任务积压时候，先执行优先级高的任务，保障业务影响最小化。
    

06

**总结与展望**  

ETL 开发是数据工程师必备的技能之一，在数据仓库、BI 等场景中起到重要的作用。但很多从业者连 ETL 对应的英文是什么都不了解，更不要谈对 ETL 的深入解析，这无疑是非常不称职的。做 ETL 你可以用任何的编程语言来完成开发，无论是 shell、python、java 甚至数据库的存储过程，只要它最终是让数据完成抽取（E）、转化（T）、加载（L）的效果即可。由于 ETL 是极为复杂的过程，而手写程序不易管理，所以越来越多的可视化调度编排工具出现了。

调度系统作为大数据平台的核心部分之一，牵扯的业务逻辑比较复杂，场景不同，也许需求就会差别很多，所以，有自研能力的公司都会选择市面上开源系统二次开发或者完全自研一套调度系统，已满足自身 ETL 任务调度需求。

不管是哪种工具，只要具备**高效运行、稳定可靠、易于维护**特点，都是一款好工具。

> 参考：
> 
> https://www.cnblogs.com/muzhongjiang/p/12641027.html https://www.kettle.net.cn/ 
> 
> https://www.xuxueli.com/xxl-job/ 
> 
> https://airflow.apache.org/

- EOF -
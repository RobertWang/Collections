> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/bigberg/p/8761047.html)

### 一、Swarm 介绍

　　Swarm 是 Docker 公司在 2014 年 12 月初发布的一套较为简单的工具，用来管理 Docker 集群，它将一群 Docker 宿主机变成一个单一的，虚拟的主机。Swarm 使用标准的 Docker API 接口作为其前端访问入口，换言之，各种形式的 Docker Client(docker client in Go, docker_py, docker 等) 均可以直接与 Swarm 通信。Swarm 几乎全部用 Go 语言来完成开发，Swarm0.2 版本增加了一个新的策略来调度集群中的容器，使得在可用的节点上传播它们，以及支持更多的 Docker 命令以及集群驱动。Swarm deamon 只是一个调度器（Scheduler）加路由器 (router)，Swarm 自己不运行容器，它只是接受 docker 客户端发送过来的请求，调度适合的节点来运行容器，这意味着，即使 Swarm 由于某些原因挂掉了，集群中的节点也会照常运行，当 Swarm 重新恢复运行之后，它会收集重建集群信息。

　　Docker 的 Swarm(集群) 模式，集成很多工具和特性，比如：跨主机上快速部署服务，服务的快速扩展，集群的管理整合到 docker 引擎，这意味着可以不可以不使用第三方管理工具。分散设计，声明式的服务模型，可扩展，状态协调处理，多主机网络，分布式的服务发现，负载均衡，滚动更新，安全（通信的加密）等等。

### 二、Swarm 架构

　　Swarm 作为一个管理 Docker 集群的工具，首先需要将其部署起来，可以单独将 Swarm 部署于一个节点。另外，自然需要一个 Docker 集群，集群上每一个节点均安装有 Docker。具体的 Swarm 架构图可以参照下图：

　　　![](https://images2018.cnblogs.com/blog/821560/201804/821560-20180409163634016-626148217.png)

　　Swarm 架构中最主要的处理部分自然是 Swarm 节点，Swarm 管理的对象自然是 Docker Cluster，Docker Cluster 由多个 Docker Node 组成，而负责给 Swarm 发送请求的是 Docker Client。

### 三、Swarm 关键概念　　

　　1）Swarm 集群的管理和编排是使用嵌入到 docker 引擎的 SwarmKit，可以在 docker 初始化时启动 swarm 模式或者加入已存在的 swarm

　　2）Node 一个节点 (node) 是已加入到 swarm 的 Docker 引擎的实例 当部署应用到集群，你将会提交服务定义到管理节点，接着 Manager 管理节点调度任务到 worker 节点，manager 节点还执行维护集群的状态的编排和群集管理功能，worker 节点接收并执行来自 manager 节点的任务。通常，manager 节点也可以是 worker 节点，worker 节点会报告当前状态给 manager 节点

　　3）服务（Service） 服务是要在 worker 节点上要执行任务的定义，它在工作者节点上执行，当你创建服务的时，你需要指定容器镜像

　　4）任务（Task） 任务是在 docekr 容器中执行的命令，Manager 节点根据指定数量的任务副本分配任务给 worker 节点

```
docker swarm：集群管理，子命令有init, join, leave, update。（docker swarm --help查看帮助）
docker service：服务创建，子命令有create, inspect, update, remove, tasks。（docker service--help查看帮助）
docker node：节点管理，子命令有accept, promote, demote, inspect, update, tasks, ls, rm。（docker node --help查看帮助）
  
node是加入到swarm集群中的一个docker引擎实体，可以在一台物理机上运行多个node，node分为：
manager nodes，也就是管理节点
worker nodes，也就是工作节点
  
1）manager node管理节点：执行集群的管理功能，维护集群的状态，选举一个leader节点去执行调度任务。
2）worker node工作节点：接收和执行任务。参与容器集群负载调度，仅用于承载task。
3）service服务：一个服务是工作节点上执行任务的定义。创建一个服务，指定了容器所使用的镜像和容器运行的命令。
   service是运行在worker nodes上的task的描述，service的描述包括使用哪个docker 镜像，以及在使用该镜像的容器中执行什么命令。
4）task任务：一个任务包含了一个容器及其运行的命令。task是service的执行实体，task启动docker容器并在容器中执行任务。　

```

### 四、主要功能　

*   **集群管理与 Docker Engine 集成:** 使用 Docker Engine CLI 来创建一个你能部署应用服务到 Docker Engine 的 swarm。你不需要其他编排软件来创建或管理 swarm。
*   **分散式设计：**Docker Engine 不是在部署时处理节点角色之间的差异，而是在运行时扮演自己角色。你可以使用 Docker Engine 部署两种类型的节点，管理器和 worker。这意味着你可以从单个磁盘映像构建整个 swarm。
*   **声明性服务模型：** Docker Engine 使用声明性方法来定义应用程序堆栈中各种服务的所需状态。例如，你可以描述由消息队列服务和数据库后端的 Web 前端服务组成的应用程序。
*   **伸缩性：**对于每个服务，你可以声明要运行的任务数。当你向上或向下缩放时，swarm 管理器通过添加或删除任务来自动适应，以保持所需状态。
*   **期望的状态协调：**swarm 管理器节点持续监控群集状态，并调整你描述的期望状态与实际状态之间的任何差异。 例如，如果设置运行一个 10 个副本容器的服务，这时 worker 机器托管其中的两个副本崩溃，管理器则将创建两个新副本以替换已崩溃的副本。 swarm 管理器将新副本分配给正在运行和可用的 worker。
*   **多主机网络：**你可以为服务指定覆盖网络 ([overlay](https://www.centos.bz/tag/overlay/) network)。 当 swarm 管理器初始化或更新应用程序时，它会自动为容器在覆盖网络 (overlay network) 上分配地址。
*   **服务发现：**Swarm 管理器节点为 swarm 中的每个服务分配唯一的 DNS 名称，并负载平衡运行中的容器。 你可以通过嵌入在 swarm 中的 DNS 服务器查询在 swarm 中运行中的每个容器。
*   **负载平衡：**你可以将服务的端口暴露给外部的负载均衡器。 在内部，swarm 允许你指定如何在节点之间分发服务容器。
*   **安全通信：**swarm 中的每个节点强制执行 TLS 相互验证和加密，以保护其自身与所有其他节点之间的通信。 你可以选择使用自签名根证书或来自自定义根 CA 的证书。
*   **滚动更新：**在上线新功能期间，你可以增量地应用服务更新到节点。 swarm 管理器允许你控制将服务部署到不同节点集之间的延迟。 如果出现任何问题，你可以将任务回滚到服务的先前版本。

### 五、Swarm 工作方式

　　1）Node

　　![](https://images2018.cnblogs.com/blog/821560/201804/821560-20180409164150891-1148217638.png)

 　　2）Service（服务、任务、容器）

 　　![](https://images2018.cnblogs.com/blog/821560/201804/821560-20180409164251732-1692641033.png)

　　3）任务和调度

　　![](https://images2018.cnblogs.com/blog/821560/201804/821560-20180409164329275-1462999764.png)

　　4）服务副本与全局服务

　　![](https://images2018.cnblogs.com/blog/821560/201804/821560-20180409164436035-2062044686.png)

### 六、Swarm 调度策略

　　Swarm 在 scheduler 节点（leader 节点）运行容器的时候，会根据指定的策略来计算最适合运行容器的节点，目前支持的策略有：spread, binpack, random。

　　1）Random 顾名思义，就是随机选择一个 Node 来运行容器，一般用作调试用，spread 和 binpack 策略会根据各个节点的可用的 CPU, RAM 以及正在运 行的容器的数量来计算应该运行容器的节点。

　　2）Spread 在同等条件下，Spread 策略会选择运行容器最少的那台节点来运行新的容器，binpack 策略会选择运行容器最集中的那台机器来运行新的节点。 使用 Spread 策略会使得容器会均衡的分布在集群中的各个节点上运行，一旦一个节点挂掉了只会损失少部分的容器。

　　3）Binpack Binpack 策略最大化的避免容器碎片化，就是说 binpack 策略尽可能的把还未使用的节点留给需要更大空间的容器运行，尽可能的把容器运行在 一个节点上面。

### 七、Swarm Cluster 模式的特性　　

```
1）批量创建服务
建立容器之前先创建一个overlay的网络，用来保证在不同主机上的容器网络互通的网络模式
  
2）强大的集群的容错性
当容器副本中的其中某一个或某几个节点宕机后，cluster会根据自己的服务注册发现机制，以及之前设定的值--replicas n，
在集群中剩余的空闲节点上，重新拉起容器副本。整个副本迁移的过程无需人工干预，迁移后原本的集群的load balance依旧好使！
不难看出，docker service其实不仅仅是批量启动服务这么简单，而是在集群中定义了一种状态。Cluster会持续检测服务的健康状态
并维护集群的高可用性。
  
3）服务节点的可扩展性
Swarm Cluster不光只是提供了优秀的高可用性，同时也提供了节点弹性扩展或缩减的功能。当容器组想动态扩展时，只需通过scale
参数即可复制出新的副本出来。
  
仔细观察的话，可以发现所有扩展出来的容器副本都run在原先的节点下面，如果有需求想在每台节点上都run一个相同的副本，方法
其实很简单，只需要在命令中将"--replicas n"更换成"--mode=global"即可！
 
复制服务（--replicas n）
将一系列复制任务分发至各节点当中，具体取决于您所需要的设置状态，例如“--replicas 3”。
 
全局服务（--mode=global）
适用于集群内全部可用节点上的服务任务，例如“--mode global”。如果大家在 Swarm 集群中设有 7 台 Docker 节点，则全部节点之上都将存在对应容器。
  
4. 调度机制
所谓的调度其主要功能是cluster的server端去选择在哪个服务器节点上创建并启动一个容器实例的动作。它是由一个装箱算法和过滤器
组合而成。每次通过过滤器（constraint）启动容器的时候，swarm cluster 都会调用调度机制筛选出匹配约束条件的服务器，并在这上面运行容器。

```
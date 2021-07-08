> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIzOTU0NTQ0MA==&mid=2247504070&idx=1&sn=5729e5f9bbcd0922dd27637322d56194&chksm=e92aedc9de5d64df0b4ac6f4ef537023cb3c2ca7041ae4bd9f442fe7d649a243b63271af75a1&mpshare=1&scene=1&srcid=0708y76RgztbZxHKZ2s1vMr0&sharer_sharetime=1625740463878&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

**引言**

EPaxos（Egalitarian Paxos）作为工业界备受瞩目的下一代分布式一致性算法，具有广阔的应用前景。但纵观业内，至今仍未出现一个 EPaxos 的工程实现，甚至都没看到一篇能把 EPaxos 讲得通俗一点的文章。EPaxos 算法理论虽好，但由于其实在晦涩难懂，工程实现上也有很多挑战，实际应用落地尚未成熟。

本文旨在通俗易懂地介绍 EPaxos 算法，由浅入深、一步一步的让只有 Paxos 或 Raft 等分布式一致性算法基础的同学都能轻易看懂 EPaxos，真正将晦涩难懂的 EPaxos，变的平易近人，带入千万家。

在[《一文了解分布式一致性算法 EPaxos》](http://mp.weixin.qq.com/s?__biz=MzIzOTU0NTQ0MA==&mid=2247500380&idx=1&sn=025c1c9fcdba8e72a5eb2e8c8aad98fd&chksm=e92aff53de5d76454f580f3adebda282f31d3f97a3d5c6f8cdb9c2d5d8283b1719263368753b&scene=21#wechat_redirect)中，从 Paxos 的问题引出 EPaxos，介绍了 EPaxos 的基本概念与直观理解，相信读者已经对 EPaxos 有了整体的印象。

本文将从 Paxos 与 EPaxos 对比的角度，介绍 EPaxos 核心协议流程。上一篇文章最后留下的思考题，相信在阅读完本文后，都能找到答案。阅读本文需要一些 Paxos 或 Raft 等分布式一致性算法背景。

**一  EPaxos 基本思想**

EPaxos 是一个 Leaderless 的一致性算法，无需选举 Leader，任意副本均可发起提议。

Leaderless 也可以看作每个副本都是 Leader，从 Multi-Paxos 或 Raft 的角度看，如果使用多 Group，将每个 Leader 划分到不同的 Group，每个副本担任一个 Group 的 Leader，同时担任其它所有 Group 的 Follower，好像也可以做到类似 Leaderless 的效果。

使用多 Group 实现的 Leaderless，每个 Group 独立的对一系列 Instance 达成一致，每个 Group 产生一条 Instance 序列，不同 Group 产生的 Instance 彼此独立，不能确定先后顺序。因此跨 Group 的一致性是一大问题，在一致性层面无法解决，往往需要在上层使用分布式事务来解决。

EPaxos 解决了这个问题，实现了真正的 Leaderless。EPaxos 通过跟踪 Instance 之间的依赖关系，确定不同 Group 产生的 Instance 的相对顺序，然后通过排序将多 Group 产生的多条 Instance 序列合并成一条全局的 Instance 序列，实现了跨 Group 的一致性，也即实现了真正的 Leaderless。

EPaxos 先运行共识协议，使各副本对 Instance 的值和 Instance 依赖的相对顺序达成一致，随后运行排序算法，基于前面已经达成共识的 Instance 的相对顺序，对 Instance 进行全局排序，最终得到一致的全局 Instance 序列。

以上是站在 Multi-Paxos 或 Raft 使用多 Group 实现 Leaderless 的角度引出 EPaxos 的基本思想，实际 Group 是一致性算法之外的概念，这里引入 Group 只是为了方便介绍，实际 EPaxos 中并没有 Group 的概念，但与 Paxos 或 Raft 类似，可以在 EPaxos 之上实现多 Group。

**二  EPaxos 的 Instance**

EPaxos 的 Instance 与 Paxos 有所不同，Paxos 的 Instance 事先分配序号，但 EPaxos 的 Instance 不事先分配序号，各副本可以并发的乱序提交，但跟踪记录 Instance 之间的依赖关系，最后根据依赖关系排序。这里先总结一下不同点：

EPaxos 的 Instance 与 Paxos 的不同点

Paxos 的 Instance 由全局连续递增的 InstanceID 标识，InstanceID 也是 Instance 的序号，全局唯一，连续递增。

EPaxos 的 Instance 空间是二维的，每个副本拥有其中一行，因此由二维的 R.i 标识，其中 R 为副本标识，i 为副本内连续递增的整数，每开始一个新的 Instance 就加一。副本 R 拥有的 Instance 序列为 R.1，R.2，R.3，......  R.i，......

EPaxos 的 Instance 相对 Paxos 还有一些额外的属性：

*   state 标识 Instance 当前的状态，可取值 pre-accepted、accepted、committed。因为 EPaxos 的 Instance 的状态比较多，所以需要专门的 state 字段标识。
    

*   deps 为依赖的 Instance 集合，存放所有依赖的 Instance 的标识，即要在前面执行的 Instance。deps 保存了 Instance 之间的相对顺序，后续将基于 deps 对 Instance 进行排序。
    

*   seq 为 Instance 的序列号，其值为 deps 中所有 Instance 的 seq 的最大值加一，反映 Instance 提议的顺序，后续排序的时候使用。
    

EPaxos 的 Instance 的 deps 和 seq 属性与 Instance 的值一样，也需要在各副本之间达成一致，后续各副本将独立的基于 deps 和 seq 对 Instance 进行排序。因为 EPaxos 的排序算法是确定性的，各副本基于相同的 deps 和 seq 对 Instance 进行排序，最终会得到一致的全局 Instance 序列。

将 Instance 看作图的顶点，deps 就是顶点的出边，图的顶点和边都确定并在各副本之间达成一致之后，各副本对图进行确定性的排序，最终得到一致的 Instance 序列。

**三  EPaxos 共识协议**

Paxos 提交一个值需要两阶段，而 EPaxos 的 Instance 多了依赖集合 deps 和序列号 seq，为了确定这些属性的值，EPaxos 比 Paxos 多了一个阶段。完整的 EPaxos 有三阶段，但并非每个阶段都是必须的，下表将 Paxos 与 EPaxos 的协议流程进行对比：

Paxos 与 EPaxos 的协议流程对比

EPaxos 与 Paxos 相比，Prepare 阶段和 Accept 阶段类似，但 EPaxos 多了 PreAccept 阶段，也是 EPaxos 最关键的一个阶段。正因为 EPaxos 多了 PreAccept 阶段，Instance 的状态更多了，所以引入专门的 state 属性来标识 Instance 当前所处的状态（pre-accepted、accepted、committed）。没有引入 Prepare 阶段的状态，是因为 Prepare 阶段没有提议值，可以通过本地有无提议值来与其它状态区分。通常情况下，EPaxos 只运行 PreAccept 阶段，即可 Commit，Prepare 阶段和 Accept 阶段都能跳过。

EPaxos 与 Paxos 类似，在任意阶段如果发现 Instance 被抢占，都需要回退到 Prepare 阶段重新开始。

1  Prepare 阶段

Prepare 阶段的作用，EPaxos 与 Paxos 类似，都是为了争取提议权，同时学习之前已经提议的最新值。在 EPaxos 中，因为每个副本都拥有各自独立的 Instance 空间，在自己的 Instance 空间上提议，相当于 Multi-Paxos 的 Leader，所以与 Multi-Paxos 类似，通常情况下可直接跳过 Prepare 阶段，直接从下一阶段开始。

2  PreAccept 阶段

PreAccept 阶段是 EPaxos 特有的，其作用是确定 Instance 的依赖集合 deps 和序列号 seq，同时尽量让提议值、deps 和 seq 在各副本间达成一致。若 PreAccept 阶段已经达成一致，则直接到 Commit 阶段（Fast Path），否则需要运行 Accept 阶段，然后才到 Commit 阶段（Slow Path）。

PreAccept 阶段是如何确定 Instance 的依赖集合 deps 和序列号 seq 的呢？其实比较简单，从副本本地已存在的 Instance 中，收集需要在前面执行的 Instance，即本地 deps 集合，本地 seq 即本地 deps 中的所有 Instance 的 seq 的最大值加一。最终的依赖集合 deps 取大多数副本的本地 deps 集合的并集，最终的序列号 seq 则取他们的本地 seq 的最大值。

各副本并发提议的时候，不同副本的本地 deps 和本地 seq 都可能不一样，那么 PreAccept 阶段如何才能达成一致呢？如果有足够多的副本（Fast Quorum）的本地 deps 和本地 seq 都相同，则已经达成了一致。否则，最终的依赖集合 deps 取大多数（Slow Quorum）副本的本地 deps 的并集，最终的序列号 seq 取它们的本地 seq 的最大值，然后运行 Accept 阶段达成一致。

PreAccept 阶段的 Fast Quorum 始终包含提议者，会在后面讨论原因。Fast Quorum 的值不小于 Slow Quorum。假设副本总数为 N，可容忍 F 个副本同时失效，N = 2F + 1，则 Fast Quorum = 2F，优化后的 EPaxos 可优化至 F + [(F + 1) / 2 ]，Slow Quorum = F + 1。Fast Quorum 取值的推导这里先不做介绍，会在后续文章中详细讨论，Slow Quorum 即大多数副本，与 Paxos 的 Accept 阶段相同。

3  Accept 阶段

Accept 阶段的作用，EPaxos 与 Paxos 类似，但 Paxos 只将提议值同步到大多数副本，EPaxos 需要将提议值、deps 和 seq 都同步到大多数副本，一旦形成多数派则决议达成。若在 PreAccept 阶段已经达成决议，则可跳过 Accept 阶段，直接进入 Commit 阶段。

4  Commit 阶段

Commit 阶段的作用，EPaxos 与 Paxos 类似，都是将达成的决议异步发送给其它副本，让其它副本学习到决议，不同的是 EPaxos 的决议不仅包括决议值，还包括 deps 和 seq。

**四  EPaxos 排序算法**

与 Paxos 不同，EPaxos 的 Instance 提交后，它们的顺序还没有确定，因此 EPaxos 需要一个额外的排序过程，对已提交的 Instance 进行排序。当 Instance 被提交，并且其依赖集合 deps 中的所有 Instance 也都提交后，就可以开始一次排序过程。

将 EPaxos 的 Instance 看作图的顶点，Instance 的依赖集合 deps 看作顶点的出边，Instance 的值和依赖集合 deps 达成一致后，图的顶点和边就在各副本之间达成一致，因此各副本会看到相同的依赖图。

EPaxos 对 Instance 排序的过程，类似于对图进行确定性的拓扑排序。但需要注意的是 EPaxos 的 Instance 之间的依赖可能形成环，即图中可能会有环路，因此不完全是拓扑排序。

为了处理循环依赖，EPaxos 对 Instance 排序的算法需要先寻找图的强连通分量，环路都包含在了强连通分量中，如果把一个强连通分量整体看作图的一个顶点，则所有强连通分量构成一个有向无环图（DAG），然后对所有的强连通分量构成的有向无环图进行拓扑排序。

EPaxos 排序算法的流程如图 1 所示，其中由背景色圈起来的部分是强连通分量：

EPaxos 排序算法

寻找图的强连通分量一般使用 Tarjan 算法，它是一个递归算法，实测发现递归实现很容易爆栈，也给工程应用带来了一定的挑战。

不同强连通分量中的 Instance 按照确定性的拓扑顺序排序，同一强连通分量中的 Instance 是并发提议的，理论上可以按任意确定性规则排序。EPaxos 为每个 Instance 计算了一个 seq 序列号，seq 的大小反映了 Instance 提议的顺序，同一强连通分量里面的 Instance 按照 seq 大小排序。实际 seq 可能重复，并不能保证全局唯一递增，EPaxos 论文并未考虑到这一点，实际可以使用 seq 加副本标识排序。

事实上，随着新的 Instance 不断的运行，旧的 Instance 可能依赖新的 Instance，新的 Instance 又可能依赖更新的 Instance，如此下去可能导致依赖链不断延伸，没有终结，排序过程一直无法进行，形成活锁。这也是 EPaxos 工程应用的一大挑战。

因为 Instance 排序算法是确定性的，各副本基于一致的依赖图对 Instance 排序后，将得到一致的 Instance 序列。

**五  EPaxos 案例**

下面使用一个具体的案例，介绍 EPaxos 的核心协议流程，如下图所示，系统由 R1、R2、R3、R4、R5，5 个副本组成，水平方向代表时间，值 A、B、C 的提议流程如图所示。

EPaxos 共识协议

案例中各 Instance 的属性取值如下表所示：

EPaxos 核心协议流程案例中 Instance 的属性

1  提议值 A

首先 R1 运行 PreAccept 阶段发起提议值 A，它首先获取自己的本地 deps 和本地 seq，此时它本地还没有任何 Instance，所以本地 deps 为空集，本地 seq 为初始值 1，它持久化提议值 A、本地 deps 和本地 seq。

然后 R1 向 R2、R3 广播 PreAccept(A) 消息，消息携带提议值 A、本地 deps、以及本地 seq（图中未标示），此时 R1、R2、R3 组成 Fast Quorum。PreAccept 消息可以只广播给 Fast Quorum 中的副本，EPaxos 论文中将这种优化称为 Thrifty 优化。

R2、R3 收到 PreAccept(A) 消息后，分别获取自己的本地 deps 和本地 seq，与 R1 类似，本地 deps 为空集，本地 seq 为 1，持久化后回复 R1。

R1 收到 Fast Quorum 中的副本的本地 deps 和本地 seq 都相同，决议达成，最终的 deps 为空集，seq 为 1，运行 Commit 阶段提交。

2  提议值 B

接下来 R5 运行 PreAccept 阶段发起提议值 B，此时它本地也还没有任何 Instance，所以本地 deps 为空集，本地 seq 为初始值 1。R5 本地持久化后，向 R3、R4 广播 PreAccept(B) 消息。

R4 回复的本地 deps 为空集，本地 seq 为 1，R3 此时本地已经有了值 A 的 Instance，因此 R3 回复的本地 deps 为 {1.1}，也即图上标示的 {A}，本地 seq 为 2，即 A 的 Instance 的 seq 加一。

Fast Quorum 中的副本的本地 deps 和 seq 不都相同，因此需要运行 Accept 阶段，最终的 deps 取大多数副本的本地 deps 的并集为 {1.1}，也即图上标的 {A}，最终的 seq 取大多数副本的本地 seq 的最大值为 2，通过 Accept 阶段，将提议值 B、最终的 deps、最终的 seq 达成多数派。最后运行 Commit 阶段提交。

3  提议值 C

最后 R1 运行 PreAccept 阶段发起提议值 C，此时 R1 本地已经有了值 A 的 Instance，因此本地 deps 为 {1.1}，也即图上标示的{A}，本地 seq 为 3。R1 本地持久化后，向 R2、R3 广播 PreAccept(C) 消息。

R2 和 R3 此时本地已经有了值 A 和值 B 的 Instance，因此 R2、R3 回复的本地 deps 为 1.1，5.1}，也即图上标示的 {A，B}，本地 seq 为 3，即 B 的 Instance 的 seq 加一。

Fast Quorum 中除了提议者 R1 以外的其它副本的本地 deps 和 seq 都相同，因此决议已经达成，最终的 deps 为 {1.1，5.1}，也即 {A，B}，seq 为 3，运行 Commit 阶段提交。

4  排序

提议值 A、B、C 的 Instance 按照它们的依赖集合 deps 画出的依赖关系如下图（左）所示：

提议值 A、B、C 的 Instance 之间的依赖关系（左），排序之后的顺序（右）

A 的 Instance 的 deps 为空集，因此没有出边；B 的 Instance 的 deps 为 {A}，因此有一条出边指向 A；C 的 Instance 的 deps 为 {A，B}，因此有两条出边，分别指向 A 和 B。

依赖关系图中没有循环依赖，已经是一个有向无环图（DAG）。因此顶点 A、B、C 各自是一个强连通分量，进行确定性的拓扑排序后得到它们的顺序：A <— B <— C，如图如图（右）所示。

**六  EPaxos 讨论**

1  Instance 冲突

EPaxos 引入 Instance 冲突（Interfere）的概念（与 Parallel Raft 类似，与并发冲突不是一个概念），若两个 Instance 的值之间没有冲突（例如访问不同的 key），则它们的先后顺序无关紧要，甚至可以并行处理，因此 EPaxos 只处理有冲突的日志之间的依赖关系。

EPaxos 的 Instance 的依赖集合 deps，存放的是需要在该 Instance 之前执行的 Instance。引入冲突之后，deps 中存放的是与该 Instance 冲突的 Instance。

冲突是一个与具体应用相关的概念，引入冲突之后，所有 Instance 不再是全序的，变成了偏序的，减少了依赖，降低了 Slow Path 的概率，提高了效率。

2  Fast Quorum

关于 Fast Quorum 取值的推导，留待后续文章介绍，这里先讨论下，为什么 PreAccept 阶段的 Fast Quorum 始终包含提议者。

EPaxos 在每个阶段，提议者总是本地先持久化成功之后，再广播消息给其它副本。也就是说提议者总是在 Quorum 中，因此判断是否达成 Quorum 时，提议者总是可以算一票。

在 PreAccept 阶段，提议者将其本地 deps 和 seq 包含在了 PreAccept 消息中，作为其它副本计算它们本地 deps 和 seq 的基础，使得提议者的本地 deps 和 seq 总是包含在最终的 deps 和 seq 中，因此 PreAccept 阶段的 Fast Quorum 始终包含提议者。

EPaxos 总是先本地持久化成功之后再广播给其它副本，这样可以减小 Fast Quorum，但也导致本地持久化与网络消息收发不能并行进行，降低了一些效率，同时也使得提议者不能容忍本地磁盘损坏的情况，这些都是 EPaxos 工程应用必须面对的问题。

Fast Quorum 的值为什么不会小于 Slow Quorum 呢？这里无需推导 Fast Quorum 的取值，从直观上就可以得出这个结论。在 Paxos 中一个副本提议一个值，所有副本只会有两种结果，接受或者不接受这个值。而在 EPaxos 中每个副本都可能给出不同的 deps 和 seq，因此需要更多的副本给出相同的结果才能保证在有副本失效后能恢复出正确的结果。

**七  EPaxos 伪代码**

到这里，相信读者已经可以看懂 EPaxos 核心协议流程的伪代码了。EPaxos 核心协议流程伪代码如下图所示，为了简单，省略了提议 ID（Proposal ID，或者叫 Ballot Number，投票编号）相关的部分，这部分与 Paxos 相同。

伪代码中将日志视为命令（Command），每个 Instance 对一条 Command 达成一致，每个副本使用一个二维数组 cmds 保存收到的 Command。

EPaxos 核心协议流程伪代码

**八  总结**

EPaxos 通过显示维护 Instance 之间的依赖关系，不仅去除了对 Leader 的依赖，还使得 Instance 可以并发的乱序提交，可以更好的进行 Pipelining 优化，同时显式维护依赖也使得乱序执行成为可能。EPaxos 支持乱序确认、乱序提交、乱序执行，理论上可以有更高的吞吐量。同时也可以看到一些 EPaxos 工程应用的挑战，这些都是 EPaxos 工程落地要解决的问题。

本文从 Paxos 与 EPaxos 对比的角度，介绍了 EPaxos 核心协议流程，但 EPaxos 的内容决不仅仅只有这些，特别是 Failover 场景下，如何保证日志序列的顺序性。

**思考**

最后留下几个思考题，感兴趣的同学可以先思考思考：

1.  Instance 的 seq 为什么会重复，什么情况下会重复？
    
2.  Fast Quorum 的取值如何推导？
    
3.  如果一个 Instance 的共识协议流程还未完成，其提议者就宕机了，其它副本该怎么处理这个 Instance？
    

**招聘**

阿里云存储 - 基础系统 & 协同服务团队，负责自研的飞天分布式协同基础服务 - 女娲，产品支持着阿里云的计算、存储、网络等几乎所有云产品，支持从单地域到全球化各个规模下的数据协同，持续保持业界先进水平。团队致力于分布式系统中最核心的分布式一致性技术的研发，在这里，你会获得实现超大规模分布式系统核心技术的经验，在个人专业上成长的同时在中国云计算的发展史上留下自己的印记。感兴趣的童鞋可联系 xiangguang.yxg@alibaba-inc.com

《分布式文件存储系统技术及实现》

本课程针对分步式文件存储系统的实现进行讲解，首先分析为什么要使用这种分步式存储系统，以及这种系统在设计时需要注意的问题，并通过比较常见的分步式存储系统（HDFS、Ceph 等），分享阿里 Pangu 系统针对其中问题的解决方法，结合 Pangu 系统说明分步式存储系统的设计要点。

点击 “阅读原文”，开始学习吧~
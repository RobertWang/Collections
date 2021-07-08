> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIzOTU0NTQ0MA==&mid=2247500380&idx=1&sn=025c1c9fcdba8e72a5eb2e8c8aad98fd&chksm=e92aff53de5d76454f580f3adebda282f31d3f97a3d5c6f8cdb9c2d5d8283b1719263368753b&scene=21#wechat_redirect)

**引言**

EPaxos（Egalitarian Paxos）作为工业界备受瞩目的下一代分布式一致性算法，具有广阔的应用前景。但纵观业内，至今仍未出现一个 EPaxos 的工程实现，甚至都没看到一篇能把 EPaxos 讲的通俗一点的文章。EPaxos 算法理论虽好，但由于其实在晦涩难懂，工程实现上也有很多挑战，实际应用落地尚未成熟。

本文旨在通俗易懂地介绍 EPaxos 算法，由浅入深、一步一步的让只有 Paxos 或 Raft 等分布式一致性算法基础的同学都能轻易看懂 EPaxos，真正将晦涩难懂的 EPaxos，变的平易近人，带入千万家。

**Paxos 的问题**

一切还要从 Paxos 说起。Paxos 是分布式一致性算法的鼻祖，在 2F+1 个副本中可以容忍 F 个副本同时失效。

Paxos 正常情况下达成一次决议需要两个阶段：Prepare 阶段和 Accept 阶段。

Prepare 阶段各副本竞争提议权，Accept 阶段竞争到提议权的副本发起提议，让议案在各副本间达成一致。

使用 Paxos 对一系列值达成一致的流程如图 1 所示。三个副本，以不同颜色标识，A、B、C、D 是它们提议的值。它们竞争每个 Instance，提议自己的值：

图 1 使用 Paxos 对一系列值达成一致

Paxos 独立的决定每个 Instance 的值。针对每个 Instance，运行完整的 Paxos 两阶段流程，决定该 Instance 的值。

Paxos 达成一次决议至少需要两次网络来回，在并发情况下可能需要更多的网络来回，极端情况下甚至可能形成活锁，效率低下。为了解决这些问题，Multi-Paxos 应运而生。

Multi-Paxos 在各副本中选举一个 Leader，提议由 Leader 发起，没有竞争，解决了活锁问题。提议都由 Leader 发起的情况下，Prepare 阶段可以跳过，将两阶段变为一阶段，提高效率。

使用 Multi-Paxos 对一系列值达成一致的流程如图 2 所示。三个副本，以不同颜色标识，首先进行 Leader 选举，绿色副本被选为 Leader，然后连续提议 A、B、C、D 四个值：

图 2 使用 Multi-Paxos 对一系列值达成一致

Multi-Paxos 首先选举 Leader，Leader 选出来后 Instance 的提议权都归 Leader，无需竞争 Instance 的提议权，因此可以省略 Prepare 阶段，只需要一阶段。Leader 的存在提高了达成决议的效率，但同时也成为了性能和可用性的瓶颈。

Leader 需要处理比其它副本更多的消息，各副本负载不均衡，资源利用率不高。Leader 宕机后系统不可用，直到新 Leader 被选举出来，才能恢复服务，降低了可用性。

Basic Paxos 每个副本都能提议，可用性高，但因为竞争冲突导致效率低下；Multi-Paxos 选举 Leader 避免冲突，提高效率，但同时又引入了 Leader 瓶颈，降低了可用性。效率和可用性能否兼顾？EPaxos 正是为了解决此问题而提出。不同于 Multi-Paxos 引入 Leader 来避免冲突，EPaxos 采用另一种思路，它直面冲突，试图解决冲突问题。

**EPaxos 的解决方案**

EPaxos 是一个 Leaderless 的一致性算法，任意副本均可发起提议，通常情况下，达成一次决议需要一次或两次网络来回。

EPaxos 无 Leader 选举开销，一个副本不可用可立即访问其他副本，具有更高的可用性。各副本负载均衡，无 Leader 瓶颈，具有更高的吞吐量。客户端可选择最近的副本提供服务，在跨 AZ 跨地域场景下具有更小的延迟。

不同于 Paxos，事先对所有 Instance 编号，然后再独立对每个 Instance 的值一一达成一致。EPaxos 可并发的处理多个 Instance，不事先对 Instance 编号，而是在运行时动态决定各 Instance 之间的顺序。

EPaxos 不仅对每个 Instance 的值达成一致，还对 Instance 之间的相对顺序达成一致。EPaxos 将不同 Instance 之间的相对顺序也作为一致性问题，在各个副本之间达成一致，因此各个副本可并发地在不同的 Instance 中发起提议，在这些 Instance 的值和相对顺序都达成一致后，各副本再对它们按照相对顺序重新排序，形成一致的顺序。

使用 EPaxos 对一系列值达成一致的流程如图 3 所示：三个副本，以不同颜色标识，各副本有自己的 Instance 空间，在各自的 Instance 中提议自己的值，A、B、C、D 是它们提议的值。每个 Instance 不仅对值达成一致，还对与其它 Instance 之间的相对顺序达成一致。

图 3 使用 EPaxos 对一系列值达成一致

EPaxos 的 Instance 空间是二维的，每个副本拥有二维 Instance 空间中的一行，无需竞争 Instance 的提议权，各副本可并发的在各自的 Instance 空间中发起提议，同时维护 Instance 之间的相对顺序，对 Instance 的值和相对顺序都达成一致。最后各副本各自按照相对顺序对 Instance 进行确定性的重新排序，即对一系列值达成一致。

EPaxos 引入依赖（deps）的概念，作为 Instance 的一个属性，以表示 Instance 之间的相对顺序。A ← B 即 B 依赖 A，表示 A 在 B 之前。每个 Instance 都有自己的依赖集合，EPaxos 维护 Instance 之间的依赖，并让依赖集合与值一起在各副本之间达成一致，最后各副本按照依赖对 Instance 重新排序，得到一致的值序列。图 3 中的案例，最后各副本达成一致的一系列值为：A ← B ← C ← D。

将 EPaxos 的 Instance 看作图上的结点，Instance 的依赖集合看作结点的出边，Instance 的值和依赖集合达成决议后，图的结点和边就在各副本之间达成一致，因此各副本会看到到相同的图。

EPaxos 对 Instance 重新排序的过程，类似于对图进行确定性的拓扑排序。但需要注意的是 EPaxos 的 Instance 之间的依赖可能形成环，即图中可能有环路，因此不完全是拓扑排序。

为了处理循环依赖，EPaxos 对 Instance 重排序的算法需要先寻找图的强连通分量，环路都包含在了强连通分量中，所有强连通分量构成一个有向无环图（DAG），然后对强连通分量进行确定性的拓扑排序。

EPaxos 对 Instance 重新排序的流程如图 4 所示，其中由背景色圈起来的是强连通分量：

图 4 EPaxos 对 Instance 重新排序流程

寻找图的强连通分量一般使用 Tarjan 算法，它是一个递归算法，实际压测发现递归实现很容易爆栈，也给工程应用带来了一定的挑战。

不同强连通分量中的 Instance 按照确定性的拓扑顺序排序，同一强连通分量中的 Instance 是并发提议的，理论上可按任意确定性规则排序。EPaxos 给出了一种方案，为每个 Instance 维护了一个 seq 序列号，seq 的大小近似反映了 Instance 提议的顺序，期望全局唯一递增，同一强连通分量里面的 Instance 按照 seq 大小排序。实现的时候测试发现 seq 可能重复，EPaxos 论文并未考虑到这一点，后续文章会更详细的介绍此问题与解决方案。

当有 Instance 达成决议，并且其依赖的所有 Instance 也都达成决议后，就可以开始一次排序过程。实际上，随着新的 Instance 不断的运行，旧的 Instance 可能依赖新的 Instance，新的 Instance 又可能依赖更新的 Instance，导致依赖链不断延伸，没有终结，排序过程一直无法进行，形成活锁。这也是 EPaxos 工程应用的一大挑战。

因为 Instance 排序算法是确定性的，各副本基于一致的依赖关系图对 Instance 重新排序后，得到一致的 Instance 序列，即对一系列值达成一致。

**总结**

EPaxos 通过引入动态顺序，同时兼顾了效率和可用性，融合了 Basic Paxos 和 Multi-Paxos 的优点，具有广阔的应用前景。本文粗浅的介绍了 EPaxos 的基本概念和直观理解，希望能让读者对 EPaxos 有个整体印象。

**思考**

最后留下几个思考题，感兴趣的同学可以思考思考：

1.  EPaxos 的 Instance 没有事先编号，那 Instance 如何标识？
    
2.  EPaxos 如何确定 Instance 的依赖集合，又如何让依赖集合达成一致？
    
3.  EPaxos 的 Instance 之间的依赖为什么会形成环，什么情况下会形成循环依赖？
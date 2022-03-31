> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/tM0RHCnCGbctXl3EDSaSNQ)

背景

基于丢包的拥塞控制算法的缺陷

*   丢包≠拥塞
    

在 tcp 拥塞控制算法提出的 20 世纪 80 年代，直接把丢包等价于发⽣了拥塞，需要降低发送 速率。受限于硬件的发展⽔平和⽹络普及程度，这在过去的很⻓⼀段时间都是⼗分适⽤的。然⽽ 30 多年 过去了，⽹络的带宽从⼏ Mbps 已经发展到了⼏ Gbps，⽹络上的⽹卡、交换机和路由器的内存也从⼏ KB 发展到了⼏ GB。并且⽹络的复杂程度也原超当年，各种跨洋跨⼤洲的通信链路以及⽆线⽹络使得⽹ 络丢包和⽹络拥塞的关系越来越稀薄。在实际情况中，⾼速⼴域⽹内的传输错误丢包是不能忽略的，特 别是在⽆线⽹络的情况下，整个链路的不稳定性导致的丢包更是⾮常常⻅。如果简单的认为丢包就是拥 塞所引起，从⽽降低⼀半的速率，将会导致⽹络吞吐变低。 

*   缓冲区膨胀 (bufferbloat)
    

⽹络中的各种设备基本都有⾃⼰的缓存，基于丢包的拥塞控制算法⾸先会填 满⽹络中的缓存，当缓存填满以后出现了丢包才开始降低发送速率。如果⽹络瓶颈的缓存⽐较⼤时，这 278 就可能造成极⼤的延时。要解决上⾯的问题，就需要另外⼀种思路，不是基于丢包来估计⽹络发⽣了拥堵。

原理

tcp 拥塞控制的初衷是为了在保证通信质量的前提下尽可能⾼的利⽤通信链路的带宽。⼀个 tcp 链接就如同 ⼀条条管道，评价⼀条 tcp 链路的传输性能主要依靠 2 个参数: RTprop (round-trip propagation time) 往 返传播时间和 BtlBw (bottleneck bandwidth) 瓶颈带宽，就如同现实管道中的管道⻓度和最⼩直径。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EVNE5LhswBCBGNBiaNTttkGLXpic6SqLnNIG28125zWerU1ts1ZXo41Aib6L2MonGROlbvPsme032P6gNNnh2NkUA/640?wx_fmt=png)

当⽹络中数据包不多，还没有填满瓶颈链路的管道时，RTprop 决定着链路的性能。随着投递率的增加，往 返时延不发⽣变化。在 1979 Leonard Kleinrock 证明了，当数据包刚好填满管道时，即满⾜最⼤带宽 BtlBw 和最⼩时延 RTprop，⽆论是单独链接还是整个⽹络来说是最优⼯作点。定义带宽时延积 BDP=BtlBw × RTprop，则在最优点⽹络中的数据包数量 = BDP。继续增加⽹络中的数据包，超出 BDP 的 数据包会占⽤ buffer，达到瓶颈带宽的⽹络的投递率不再发射变化，RTT 会增加。继续增加数据包， buffer 会被填满从⽽发⽣丢包。故在 BDP 线的右侧，⽹络拥塞持续作⽤。过去基于丢包的拥塞控制算法⼯ 作在 bandwidth-limited 区域的右侧边界，将瓶颈链路管道填满后继续填充 buffer，直到 buffer 填满发⽣ 丢包，拥塞控制算法发现丢包，将发送窗⼝减半后再线性增加。过去存储器昂贵，buffer 的容量只⽐ BDP 稍⼤，增加的时延不明显，随着内存价格的下降导致 buffer 容量远⼤于 BDP，增加的时延就很显著了。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EVNE5LhswBCBGNBiaNTttkGLXpic6SqLnNRvnGnYlwZn5ELZibUYbhJI0l6WiaOFEnnCfutqNdj2OAUXZmFuwErxTA/640?wx_fmt=png)

针对上述的问题，BBR 采⽤了完全不同的⽅案：既然丢包不能等同于拥塞，那就忽略丢包，通过观测和量 化链路的瓶颈带宽和往返时延来让拥塞控制算法处理真正的⽹络拥塞，从⽽尽可能的让⽹络传输达到 Kleinrock 所提出来的最优⼯作点。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EVNE5LhswBCBGNBiaNTttkGLXpic6SqLnNpmrDH530KLfFuTH5GYWeTFHdNdr7SA5LsUznUibiae8P3yeNlbvribqRA/640?wx_fmt=png)

然⽽另外⼀个问题随之⽽来了，同样在 20 世纪 80 年代，Jeffrey M. Jaffe 就证明了没有什么算法能同时计 算出这个最佳⼯作点。要测量最低延迟 (min RTT)，就必须要排空缓存，⽹络中的数据包越少越好，⽽此时 带宽是较低的。要测量最⼤带宽 (max BW)，就要把链路瓶颈填满，在 buffer 中有⼀定的数据包，⽽此时延 时⼜是较⾼的。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EVNE5LhswBCBGNBiaNTttkGLXpic6SqLnNdVMgN4NcFvctttwro2wRG3TaRic21qiad874tH8Fnpxun3BQ2GaPnJXA/640?wx_fmt=png)

针对测不准的问题，BBR 算法采⽤的⽅案是，交替测量带宽和延迟，⽤⼀段时间内的带宽极⼤值和延迟极 ⼩值作为估计值，动态更新测量值，最终控制发送速率，避免⽹络拥塞。

实现

BBR 启动以后主要分为四个阶段：

*   启动阶段
    

BBR 采⽤类似标准 TCP 的 slow start，指数的⽅式来探测⽹络的带宽，⽬的也 是尽可能快的占满管道，经过三次发现投递率不再增⻓，说明管道被填满，开始占⽤ buffer。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EVNE5LhswBCBGNBiaNTttkGLXpic6SqLnNMaQJmh3dHxzNDfl6twHAiaic5H0bX1Hwh3E5hNmZU04CwtpCe54RnwUw/640?wx_fmt=png)

*   排空阶段
    

指数降低发送速率，相当于是 startup 的逆过程，将多占的 buffer 慢慢排空。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EVNE5LhswBCBGNBiaNTttkGLXpic6SqLnNmHLhk6pVtXXVWQCEN7reb4aY85pW436IqbciclQYsRazVXalvl2k2oA/640?wx_fmt=png)

*   PROBE_BW:
    

在之后，BBR 进⼊稳定⼯作状态。BBR 会不断的通过改变发送速率进⾏带宽探测，先在 ⼀个 RTT 时间内增加发送速率探测最⼤带宽，如果 RTT 没有变化，后减⼩发送速率排空前⼀个 RTT 多发 出来地包，后⾯ 6 个周期使⽤更新后的估计带宽发包  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EVNE5LhswBCBGNBiaNTttkGLXpic6SqLnN4MnahRzN2fHeUDvl24b0TyzfyNPIFNKdibodYD5icMSSqBpMjjdW8Ogg/640?wx_fmt=png)

*   PROBE_RTT：
    

延迟探测阶段，BBR 设定了 min RTT 的超时时间为 10 秒，每过 10 秒，会进⼊⾮常短暂延 迟探测阶段，为了探测最⼩延迟，BBR 在这段时间内仅发送最少量的包。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EVNE5LhswBCBGNBiaNTttkGLXpic6SqLnNvvNFqSzyfYdlk5V6XoXkdfuPiaPjymIrRoNu8ibEpM1AhagUicJg6npIw/640?wx_fmt=png)

在稳定⼯作的情况下，BBR 会交替的进⾏带宽探测和延时探测，并且越 98% 时间都处于带宽探测阶段，带 宽探测阶段时定期尝试增加发包速率，如果收到确认的速率也增加了，就进⼀步增加发包速率。下图展示了在第 20s 时⽹络带宽增加⼀倍时⽹络的⾏为。向上的尖峰表明它增加发送速率来探测带宽变化， 向下的尖峰表明它降低发送速率排空数据。如果带宽不变，增加发送速率肯定会使 RTT 增加。如果带宽增 加，则增加发包速率时 RTT 不变。这样经过三个周期之后，估计的带宽增加了 1.95 倍。284 ⽽在第 40s 时⽹络带宽减半，因为发送速率不变，inflight(在途数据包) 增加，多出来的数据包占⽤了 buffer，RTT 会显著增加，带宽的估计是滑动窗⼝时间内的极⼤值，当之前的估计值超时之后，降低⼀半 后的有效带宽就会变成新的估计带宽从⽽减⼩发送速率，经过 4 秒后，BBR 逐渐收敛。

我们回过头再来看 BBR 要解决的 2 个问题

*   优化在⾼丢包率下⽹络的吞吐率：
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EVNE5LhswBCBGNBiaNTttkGLXpic6SqLnN5icR86ib0Iy0XichC9MooPxcfRV5Hnc7SlYKGibsiaAw34EkBPUxH7NLFGQ/640?wx_fmt=png)

上图可以看到，传统的 TCP CUBIC 在随着丢包率的增加性能急剧下降，在千分之⼀的丢包率情况下，系统 吞吐量就下降到⽹络的百分之⼗，⽽ BBR 则在百分之五以下的丢包率的情况下，基本没有性能损失。  

*   减少缓冲区膨胀，降低⾼缓存导致的延迟
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EVNE5LhswBCBGNBiaNTttkGLXpic6SqLnNLHKib5BicMeia7nZ7b5nnAVQPmLMS4y2vKkmPZiczCviaLJjp3ZNcW8LELg/640?wx_fmt=png)

上图展示了在不同 buffer ⼤⼩的情况下，BBR 和 CUBIC 的延时表现。红线为传统 TCP CUBIC，CUBIC 倾 向于填满 buffer，这也导致延迟基本随着 buffer 的增⼤线性增⻓，⽽延时过⻓可能直接导致系统建⽴连接 超时。⽽ BBR 基本保持发送队列为最⼩，随着 buffer 的增加⽹络延迟基本不变。所以 BBR 能在存在⼀定丢包率的的⽹络环境中充分利⽤带宽，⼗分的适合在跨区域的⾼延迟⾼速⽹络。当 然 BBR 也不是万能的，许多测试和实际应⽤也发现了 BBR 存在在⽹络状况变化较⼤的情况下测不准导致⼤ 量丢包，交替测量导致波动较⼤，收敛较慢导致的公平问题等。Google 也在持续优化 BBR 算法，在 2018 年 公布的 BBR 2.0 研究进展中就包含了： 

*   提⾼与基于丢包的拥塞算法 (Reno/Cubic) 的共存能⼒。降低 BBR 算法的抢占性，提⾼不同算法之间的 公平性。 
    
*   减⼩排队丢包和排队时延的情况。这⾥主要根据丢包率和标记 ECN ⽐例来设置 inflight 的两个⻔限值， inflight_hi 和 inflight_lo。 
    
*   加快 min_rtt 的收敛性，增加是将进⼊ Probe_RTT 模式的频率由 10s 设置到 2.5s。 
    
*   减⼩ Probe_RTT 模式带来的带宽的波动。
    

参考⽂献 

[1] Neal Cardwell. "TCP BBR congestion control comes to GCP – your Internet just got faster" 

[2] Neal Cardwell, et al. "BBR: Congestion-Based Congestion Control." 

[3] Yuchung Cheng, Neal Cardwell. "Making Linux TCP Fast" 

[4] Neal Cardwell. "BBR Congestion Control Work at Google IETF 102 Update"
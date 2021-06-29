> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666555238&idx=2&sn=ef390174b38f5f7b1e1dd9c64349b890&chksm=80dca3cdb7ab2adb887447bb4abf015246695ef27b0ba430635d527e3b5f3fabb70a8c250829&scene=21#wechat_redirect)

> 来自 Google 的 TCP BBR 拥塞控制算法解析，硬核推荐

今天推荐一篇在 TCP BBR 技术里面分析非常透彻的文章，希望大家可以学习到一些真正的知识，理解其背后的设计原理，才能应对各种面试和工作挑战！  

**宏观背景下的 BBR**

1980 年代的拥塞崩溃导致了 1980 年代的拥塞控制机制的出炉，某种意义上这属于见招拆招的策略，针对 1980 年代的拥塞，提出了 1980 年代的**拥塞控制算法**，即 ss，ssthresh，congestion avoid 这些。

说实话，这些机制完美适应了 1980 年代的网络特征，低带宽，浅缓存队列，美好持续到了 2000 年代。

随后互联网大爆发，多媒体应用特别是图片，音视频类的应用促使带宽必须猛增，而摩尔定律促使存储设施趋于廉价而路由器队列缓存猛增，这便是 BBR 诞生的背景。换句话说，1980 年代的 CC 已经不适用了，2010 年代需要另外的一次见招拆招。

如果说上一次 1980 年代的 CC 旨在**收敛**，那么这一次 BBR 则旨在 **效能最大化**，E，至少我个人是这么认为的，这也和 BBR 的初衷 **提高带宽利用率** 相一致！

插个形象的 gif：

![](https://mmbiz.qpic.cn/mmbiz_gif/cYSwmJQric6lj9IPp3VK8ACM3uUNVqtv8tRrbibxjy95hr20w58kTGfcAKibVkVjpdFmoglIF2c0ofGEBju5Lp7Sg/640?wx_fmt=gif)

https://cloud.google.com/blog/products/networking/tcp-bbr-congestion-control-comes-to-gcp-your-internet-just-got-faster

**正文开始**

国庆节前，我看到了 bbr 算法，发现它就是那个唯一正确的做法 (可能有点夸张，但起码它是一个通往正确道路的起点！)，所以花了点时间研究了一下它，包括其 patch 的注释，patch 代码，并亲自移植了 bbr patch 到更低版本的内核，在这个过程中，我也产生了一些想法，作为备忘，整理了一篇文章，记如下，多年以后，再看 TCP bbr 算法的资料时，我的记录也算是中文社区少有的第一个吃螃蟹记录了，也算够了！  

正文之前，给出本文的图例：

![](https://mmbiz.qpic.cn/mmbiz_jpg/cYSwmJQric6lj9IPp3VK8ACM3uUNVqtv8fNcNT7icSeOqS7kt91ia0xg8msqiaeYHnXPC9A6dTZxrXzjxAU91fvsuA/640?wx_fmt=jpeg)

**BBR 的组成**

bbr 算法实际上非常简单，在实现上它由 5 部分组成：

![](https://mmbiz.qpic.cn/mmbiz_png/cYSwmJQric6lj9IPp3VK8ACM3uUNVqtv8KwZLQN6d3oVT1GzSfkhpDZMibFAGL5f0GtyiaEWEu9CmROwojicgyn5JQ/640?wx_fmt=png)

**BBR 的组成**

**1. 即时速率的计算**

计算一个即时的带宽 bw，该带宽是 bbr 一切计算的基准，bbr 将会根据当前的即时带宽以及其所处的 pipe 状态来计算 pacing rate 以及 cwnd(见下文)，后面我们会看到，这个即时带宽计算方法的突破式改进是 bbr 之所以简单且高效的根源。计算方案按照标量计算，不再关注数据的含义。在 bbr 运行过程中，系统会跟踪当前为止最大的即时带宽。

**2.RTT 的跟踪**

bbr 之所以可以获取非常高的带宽利用率，是因为它可以非常安全且豪放地探测到带宽的最大值以及 rtt 的最小值，这样计算出来的 BDP 就是目前为止 TCP 管道的最大容量。bbr 的目标就是达到这个最大的容量！这个目标最终驱动了 cwnd 的计算。在 bbr 运行过程中，系统会跟踪当前为止最小 RTT。

**3.BBR 状态机的维持**

bbr 算法根据互联网的拥塞行为有针对性地定义了 4 中状态，即 STARTUP，DRAIN，PROBE_BW，PROBE_RTT。bbr 通过对上述计算的即时带宽 bw 以及 rtt 的持续观察，在这 4 个状态之间自由切换，相比之前的所有拥塞控制算法，其革命性的改进在于 bbr 拥塞算法不再跟踪系统的 TCP 拥塞状态机，而旨在用统一的方式来应对 pacing rate 和 cwnd 的计算，不管当前 TCP 是处在 Open 状态还是处在 Disorder 状态，抑或已经在 Recovery 状态，换句话说，bbr 算法感觉不到丢包，它能看到的就是 bw 和 rtt！

**4. 结果输出 - pacing rate 和 cwnd**

首先必须要说一下，bbr 的输出并不仅仅是一个 cwnd，更重要的是 pacing rate。在传统意义上，cwnd 是 TCP 拥塞控制算法的唯一输出，但是它仅仅规定了当前的 TCP 最多可以发送多少数据，它并没有规定怎么把这么多数据发出去，在 Linux 的实现中，如果发出去这么多数据呢？简单而粗暴，突发！忽略接收端通告窗口的前提下，Linux 会把 cwnd 一窗数据全部突发出去，而这往往会造成路由器的排队，在深队列的情况下，会测量出 rtt 剧烈地抖动。

bbr 在计算 cwnd 的同时，还计算了一个与之适配的 pacing rate，该 pacing rate 规定 cwnd 指示的一窗数据的数据包之间，以多大的时间间隔发送出去。

**5. 其它外部机制的利用 - fq，rack 等**

bbr 之所以可以高效地运行且如此简单，是因为很多机制并不是它本身实现的，而是利用了外部的已有机制，比如下一节中将要阐述的它为什么在计算带宽 bw 时能如此放心地将重传数据也计算在内...

**带宽计算细节以及状态机**

**1. 即时带宽的计算**

bbr 作为一个纯粹的拥塞控制算法，完全忽略了系统层面的 TCP 状态，计算带宽时它仅仅需要两个值就够了：

**1). 应答了多少数据，记为 delivered；**

**2). 应答 1) 中的 delivered 这么多数据所用的时间，记为 interval_us。**

**将上述二者相除，就能得到带宽：**

**bw = delivered/interval_us**

非常简单！以上的计算完全是标量计算，只关注数据的大小，不关注数据的含义，比如 delivered 的采集中，bbr 根本不管某一个应答是重传后的 ACK 确认的，正常 ACK 确认的，还是说 SACK 确认的。bbr 只关心被应答了多少！

这和 TCP/IP 网络模型是一致的，因为在中间链路上，路由器交换机们也不会去管这些数据包是重传的还是乱序的，然而拥塞也是在这些地方发生的，既然拥塞点都不关心数据的意义，TCP 为什么要关注呢？反过来，我们看一下拥塞发生的原因，即数据量超过了路由器的带宽限制，利用这一点，只需要精心地控制发送的数据量就好了，完全不用管什么乱序，重传之类的。当然我的意思是说，拥塞控制算法中不用管这些，但这并不意味着它们是被放弃的，其它的机制会关注的，比如 SACK 机制，RACK 机制，RTO 机制等。

接下来我们看一下这个 delivered 以及 interval_us 的采集是如何实现的。还是像往常一样，我不准备分析源码，因为如果分析源码的话，往往难以抓住重点，过一段时间自己也看不懂了，相反，画图的话，就可以过滤掉很多诸如 unlikely 等异常流或者当前无需关注的东西：

![](https://mmbiz.qpic.cn/mmbiz_jpg/cYSwmJQric6lj9IPp3VK8ACM3uUNVqtv8uicUcrlSiaqdYnzOuB8LfjXmiajdKVeIcMyDBbURqJXvHyD7b7TibrGMlg/640?wx_fmt=jpeg)

上图中，我故意用了一个极端点的例子，在该例子中，我几乎都是使用的 SACK，当 X 被 SACK 时，我们可以根据图示很容易算出从 Delivered 为 7 时的数据包被确认到 X 被确认为止，一共有 12-7=5 个数据包被确认，即这段时间网络上清空了 5 个数据包！我们便很容易算出带宽值了。我的这个图示在解释带宽计算方法之外，还有一个目的，即说明 bbr 在计算带宽时是不关注数据包是否按序确认的，它只关注数量，即数据包被网络清空的数量。实实在在的计算，不猜 Lost，不猜乱序，这些东西，你再怎么猜也猜不准！

计算所得的 bw 就是 bbr 此后一切计算的基准。

**2. 状态机**

bbr 的状态机转换图以及注释如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_jpg/cYSwmJQric6lj9IPp3VK8ACM3uUNVqtv8BA0DcNt4jKpYN6mZaEhc845fnCa7bJd6MzymZp8KVjaOh1ich2OZl8g/640?wx_fmt=jpeg)

通过上述的状态机以及上一节的带宽计算方式，我们知道了 bbr 的工作方式：不断地基于当前带宽以及当前的增益系数计算 pacing rate 以及 cwnd，以此 2 个结果作为拥塞控制算法的输出，在 TCP 连接的持续过程中，每收到一个 ACK，都会计算即时的带宽，然后将结果反馈给 bbr 的 pipe 状态机，不断地调节增益系数，这就是 bbr 的全部，我们发现它是一个典型的封闭反馈系统，与 TCP 当前处于什么拥塞状态完全无关，其简图如下：

![](https://mmbiz.qpic.cn/mmbiz_jpg/cYSwmJQric6lj9IPp3VK8ACM3uUNVqtv8LtO51WZ2rP773o5AwMQydgrR0oyRhsRDgiapT1bvWrYBia9gHibF59FUA/640?wx_fmt=jpeg)

这非常不同于之前的所有拥塞控制算法，在之前的算法中，我们发现拥塞算法内部是受外部的拥塞状态影响的，比如说在 Recovery 状态下，甚至都不会进入拥塞控制算法，在 bbr 进入内核之前，Linux 使用 PRR 算法控制了 Recovery 状态的窗口调整，即便说这个时候网络已经恢复，TCP 也无法发现，因为 TCP 的 Recovery 状态还未恢复到 Open，这就是根源！

**pacing rate 以及 cwnd 的计算**

这一节好像是重点中的重点，但是我觉得如果理解了 bbr 的带宽计算，状态机以及其增益系数的概念，这里就不是重点了，这里只是一个公式化的结论。

pacing rate 怎么计算？很简单，就是是使用时间窗口内 (默认 10 轮采样) 最大 BW。上一次采样的即时 BW，用它来在可能的情况下更新时间窗口内的 BW 采样值集合。这次能否按照这个时间窗口内最大 BW 发送数据呢？这样看当前的增益系数的值，设为 G，那么 BW*G 就是 pacing rate 的值，是不是很简单呢？！

至于说 cwnd 的计算可能要稍微复杂一点，但是也是可以理解的，我们知道，cwnd 其实描述了一条网络管道 (rwnd 描述了接收端缓冲区)，因此 cwnd 其实就是这个管道的容量，也就是 BDP！

BW 我们已经有了，缺少的是 D，也就是 RTT，不过别忘了，bbr 一直在持续搜集最小的 RTT 值，注意，bbr 并没有采用什么移动指数平均算法来 “猜测”RTT(我用猜测而不是预测的原因是，猜测的结果往往更加不可信！)，而是直接冒泡采集最小的 RTT(注意这个 RTT 是 TCP 系统层面移动指数平均的结果，即 SRTT，但 brr 并不会对此结果再次做平均！)。我们用这个最小 RTT 干什么呢？

当前是计算 BDP 了！这里 bbr 取的 RTT 就是这个最小 RTT。最小 RTT 表示一个曾经达到的最佳 RTT，既然曾经达到过，说明这是客观的可以再次达到的 RTT，这样有益于网络管道利用率最大化！

我们采用 BDP*G'就算出了 cwnd，这里的 G'是 cwnd 的增益系数，与带宽增益系数含义一样，根据 bbr 的状态机来获取！

**BBR 的细节浅述**

该节的题目比较怪异，既然是细节为什么又要浅述？？

这是我的风格，一方面，说是细节是因为这些东西还真的很少有人注意到，另一方面，说是浅述，是因为我一般都不会去分析代码以及代码里每一个异常流，我认为那些对于理解原理帮助不大，那些东西只是在研发和优化时才是有用的，所以说，像往常一样，我这里的这个小节还是一如既往地去谈及一些 “细节”。

**1. 豪放且大胆的安全探测**

在看到 bbr 之后，我觉得之前的 TCP 拥塞控制算法都错了，并不是思想错了，而是实现的问题。

bbr 之所以敢大胆的去探测预估带宽是因为 TCP 把更多的权力交给了它！在 bbr 之前，很多本应该由拥塞控制算法去处理的细节并不归拥塞控制算法管。在详述之前，我们必须分清两件事：

1). 传输多少数据？

2). 传输哪些数据？

按照 “上帝的事情上帝管，凯撒的事情凯撒管” 的原则，这两件事本来就该由不同的机制来完成，不考虑对端接收窗口的情况下，拥塞窗口是唯一的主导因素，“传输多少数据”这件事应该由拥塞算法来回答，而 “传输哪些数据” 这个问题应该由 TCP 拥塞状态机以及 SACK 分布来决定，诚然这两个问题是不同的问题，不应该杂糅在一起。

然而，在 bbr 进入内核之前的 Linux TCP 实现中，以上两个问题并不是分得特别清。TCP 的拥塞状态只有在 Open 时才是上述的职责分离的完美样子，一旦进入 Lost 或者 Recovery，那么拥塞控制算法即便对 “问题 1)：传输多少数据” 都无能为力，在 Linux 的现有实现中，PRR 算法将接管一切，一直把窗口下降到 ssthresh，在 Lost 状态则反应更加激烈，直接 cwnd 硬着陆！随后等丢失数据传输成功后再执行慢启动.... 在重新进入 Open 状态之前，拥塞控制算法几乎不会起作用，这并不是一种高速公路上的模式(小碰擦，拍照后停靠路边，自行解决)，更像是闹市区的交通事故处理方式(无论怎样，保持现场，直到交警和保险公司的人来现场处置)。

bbr 算法逃离了这一切错误的做法，在 bbr 的 patch 中，并非只是完成了一个 tcp_bbr.c，而是对整个 TCP 拥塞状态控制框架进行了大手术，我们可以从以下的拥塞控制核心函数中可见一斑：

```
static void tcp_cong_control(struct sock *sk, u32 ack, u32 acked_sacked,
                 int flag, const struct rate_sample *rs)
{
    const struct inet_connection_sock *icsk = inet_csk(sk);
    if (icsk->icsk_ca_ops->cong_control) {
        icsk->icsk_ca_ops->cong_control(sk, rs);
        return;
    }
    if (tcp_in_cwnd_reduction(sk)) {
        tcp_cwnd_reduction(sk, acked_sacked, flag);
    } else if (tcp_may_raise_cwnd(sk, flag)) {
        tcp_cong_avoid(sk, ack, acked_sacked);
    }
    tcp_update_pacing_rate(sk);
}

```

在这个框架下，无论处在哪个状态 (Open，Disorder，Recovery，Lost...)，如果拥塞控制算法自己声明有这个能力，那么具体可以传输多少数据，完全由拥塞控制算法自行决定，TCP 拥塞状态控制机制不再干预！

**2. 为什么 bbr 可以忽略 Recovery 和 Lost 状态**

看懂了以上第 1 点，这一点就很容易理解了。

在第 1 点中，我描述了 bbr 确实忽略了 Recovery 等非 Open 的拥塞状态，但是为什么可以忽略呢？一般而言，很多人都会质疑，会说 bbr 采用这么鲁莽的方式，最终一定会让窗口卡住不再滑动，但是我要反驳，你难道不知道 cwnd 只是个标量吗？我画一个图来分析：

![](https://mmbiz.qpic.cn/mmbiz_jpg/cYSwmJQric6lj9IPp3VK8ACM3uUNVqtv8mJeicAzUEq4BnAncRjmEDZdDCHQ4TMYxV3cgmfcicKMM67W97h450H3g/640?wx_fmt=jpeg)

看懂了吗？不存在任何问题！基本上，我们在讨论拥塞控制算法的时候，会忽略流量控制，因为不想让 rwnd 和 cwnd 杂糅起来，但是在这里，它们相遇了，幸运的是，并没有引发冲突！

然而，这并不是全部，本节旨在 “浅析”，因此就不会关注代码处理的细节。在 bbr 的实现中，如果算法外部的 TCP 拥塞状态已经进入了 Lost，那么 cwnd 该是多少呢？在 bbr 之前的拥塞算法中，包括 cubic 在内的所有算法中，当 TCP 核心实现从将 cwnd 调整到 1 或者 prr 到 ssthresh 一直到恢复到 Open 状态，拥塞算法无权干预流程，然而 bbr 不。虽然说进入 Lost 状态后，cwnd 会硬着陆到 1，然而由于 bbr 的接管，在 Lost 期间，cwnd 还是可以根据即时带宽调整的！

这意味着什么？

这意味着 bbr 可以区别噪声丢包和拥塞丢包了！

**a). 噪声丢包**

如果是噪声丢包，在收到 reordering 个重复 ACK 后，由于 bbr 并不区分一个确认是 ACK 还是 SACK 引起的，所以在 bbr 看来，即时带宽并没有降低，可能还有所增加，所以一个数据包的丢失并不会引发什么，bbr 依旧会给出一个比较大的 cwnd 配额，此时虽然 TCP 可能已经进入了 Recovery 状态，但 bbr 依旧按照自己的 bw 以及调整后的增益系数来计算 cwnd 的新值，过程中并不会受到任何 TCP 拥塞状态的影响。

如此一来，所有的噪声丢包就被区别开来了！bbr 的宗旨是：“首先，在我的 bw 计算指示我发生拥塞之前，任何传统的 TCP 拥塞判断 - 丢包 / 时延增加，均全部失效，我并不 care 丢包和 RTT 增加”，随后 brr 又会说：“但是我比较 care 的是，RTT 在一段时间内 (随你怎么配，但我个人倾向于自学习) 都没有达到我所采集到的最小值或者更小的值！这也许意味着着链路真的发生拥塞了！”...

**b). 拥塞丢包**

将 a)的论述反过来，我们就会得到奇妙的封闭性结论。这样，bbr 不光是消除了吞吐曲线的锯齿 (ssthresh 所致，bbr 并不使用 ssthresh！)，而且还消除了传统拥塞控制算法(指 bbr 以及封闭的傻逼 Appex 之前) 的判断滞后性问题。在 cubic 发现丢包进而判断为拥塞时，拥塞可能已经缓解了，但是 cubic 无法发现这一点。为什么？原因在于 cubic 在计算新的 cwnd 的时候，并没有把当前的网络状态 (比如 bw) 当作参数，而只是一味的按照数学意义上的三次方程去计算，这是错误的，这不是一个正确的反馈系统的做法！

基于 a) 和 b)，看到了吧，这就是新的拥塞判断机制！综合考虑丢包和 RTT 的增加：

**b-1). 如果丢包时真的发生了拥塞，那么测量的即时带宽肯定会减少，否则，丢包即拥塞就是谎言。**

**b-2). 如果 RTT 增加时真的发生了拥塞，那么测量的即时带宽肯定会减少，否则，时延增加即拥塞就是谎言。**

bbr 测量了即时带宽，这个统一 cwnd 和 rtt 的计量，完全忽略了丢包，因此 bbr 的算法思想是 TCP 拥塞控制的正轨！事实上，丢包本就不应该作为一种拥塞的标志，它只是拥塞的表现。

**3. 状态机的点点滴滴**

我在上文已经呈现了关于 STARTUP，DRAIN，PROBE_BW，PROBE_RTT 的状态图以及些许细节，当时我指出这个状态图的目标是为了完成 bbr 的目标，即填满整个网络！在这个状态图看来，所有已知的东西就是当前的即时带宽，所有可以计算的东西就是增益系数，然后根据这两个元素就可以轻易计算出 pacing rate 和 cwnd，是不是很简单呢？整体看来就是就是这么简单，但是从细节上看，不同的 pipe 状态中的增益系数的计算却是值得推敲的，以下是 bbr 处在各个状态时的增益系数：

**STARTUP：2~3**

**DRAIN：pacing rate 的增益系数为 1000/2885，cwnd 的增益系数为 1000/2005+1。**

**PROBE_BW：5/4，1，3/4，bbr 在 PROBE_BW 期间会随机在这些增益系数之间选择当前的增益系数。**

**PROBE_RTT：1。但是在探测 RTT 期间，为了防止丢包，cwnd 会强制 cut 到最小值，即 4 个 MSS。**

我们可以看到，bbr 并没有明确的所谓 “降窗时刻”，一切都是按照状态机来的，期间丝毫不会理会 TCP 是否处在 Open，Recovery 等状态。在此前的拥塞控制算法中，除了 Vegas 等基于延时的算法会在计算得到的 target cwnd 小于当前 cwnd 时视为拥塞而在算法中降窗外，其它的所有基于丢包的算法中均是检测到丢包(RTO 或者 reordering 个重复 ACK) 时降窗的，可悲的是，这个降窗过程并不受拥塞算法的控制，拥塞算法只能消极地给出一个 ssthresh 值，即降窗的目标，这显然是令人无助的！

bbr 不再关注丢包事件，它并不把丢包当成很严重的事，这事也不归它管，只要 TCP 拥塞状态机控制机制可以合理地将一些包标记为 LOST，然后重传它们便是了，bbr 能做的仅仅是告诉 TCP 一共可以发出去多少数据，仅此而已！然而，如果 TCP 并没有把 LOST 数据包合理标记好，bbr 并不 care，它只是根据当前的 bw 和增益系数给出下一个 pacing rate 以及 cwnd 而已！

**4. 关于 Sched FQ**

这里涉及的是 bbr 之外的东西，Fair queue！在 bbr 的 patch 最后，会发现几行注释：

**NOTE: BBR *must* be used with the fq qdisc ("man tc-fq") with pacing enabled, since pacing is integral to the BBR design andimplementation. BBR without pacing would not function properly, and may incur unnecessary high packet loss rates.**

记住这几行文字并理解它们。

这是 bbr 最为重要的一方面。虽然说 Linux 的 TCP 实现早就支持的 pacing rate，但直到 4.8 版本都没有在 TCP 层面支持它，很大的一部分原因是因为借助已有的 FQ 可以很完美地实现 pacing rate！TCP 可以借助 FQ 来实现平缓而非突发的数据发送！

关于 FQ 的详细内容可以去看相关的 manual 和源码，这里要说的仅仅是，FQ 可以根据 bbr 设置的 pacing rate 将一个 cwnd 内的数据的发送从 “突发到网络” 这种行为变换到 “平缓发送到网路” 的行为，所谓的平缓发送指的就是数据包是按照带宽速率计算的间隔一个个发送到网络的，而不是突发进网络的！

这样一来，就给了网络缓存以缓解的机会！记住，关键问题是 bbr 会在每收到 ACK/SACK 时计算 bw，这个精确的测量不会漏掉任何可乘之机，即便当前网络拥塞了，它只要能在下一时刻恢复，bbr 就可以发现，因此即时带宽通常可以表现这一点！

**5. 其它**

还有关于令牌桶监管发现 (lt policed) 的主题，long term 采样的主题，留到后面的文章具体阐述吧，本文已经足够长了。

**6.bufferbloat 问题**

关于深队列，数据包如何如何长时间排队但不丢包却引发 RTO，对于浅队列，数据包如何如何频繁丢包... 谈起这个话题我一开始想滔滔不绝，后来想骂人，现在我三缄其口！任何人都知道端到端的 QoS 是一个典型的反馈系统，但是任何人都只是夸夸其谈，我选择的是闭口不说，如果非要我说，我的回答就是：不知道！

这是一个怎么说都能对又怎么说都能错的话题，就像股票预测那样，所以我选择闭嘴。

bbr 算法到来后，单单从公共测试结果上看，貌似解决了 bufferbloat 问题，也许吧，也许。bbr 好像真的开始在高速公路上飚车了... 最后给出一个测试图，来自《A quick look at TCP BBR》：

![](https://mmbiz.qpic.cn/mmbiz_png/cYSwmJQric6lj9IPp3VK8ACM3uUNVqtv8VBO3uS73iaaOicEKxlLt62PcOOCbQD1Znlu4zhJX4U6rTa949icoMcS8A/640?wx_fmt=png)

**bbr 代码的简单性和复杂性**

我一向觉得 TCP 拥塞控制算法太过复杂，而复杂的东西基本上就是用来装逼的垃圾，直到遇到了 bbr。

Neal Cardwell 提供的 patch 简单而又直接，大家可以从该 bbr 的 patch 上一看究竟！在 bbr 模块之外，Neal Cardwell 主要更改了 tcp_ack 函数里面关于 delivered 计数的部分以及拥塞控制主函数，这一切都十分显然，只要 patch 代码就可以一目了然。在数据包被发送的时候 - 不管是初次发送还是重传，均会被当前 TCP 的连接状况记录在该数据包的 tcp_skb_cb 中，在数据包被应答的时候 - 不管是被 ACK 还是被 SACK，均会根据当前的状态和其 tcp_skb_cb 中状态计算出一个带宽，这些显而易见的逻辑相比任何人都应该知道哪里的代码被修改了！

**patch 链接：**

**https://patchwork.ozlabs.org/project/netdev/list/?submitter=10078&state=***

然而，这种查找和确认的工作太令人感到悲哀，读懂代码是容易的，移植代码是无聊的，因为时间卡的太紧！我必须要说的是，如果一件感兴趣的事情变成了必须要完成的工作，那么做它的激情起码减少了 1/4，OK，还不算太坏，然而如果这个必须完成的工作有了 deadline，那么激情就会再减少 1/4，最后，如果有人在背后一直催，那么完蛋，这件事可以瞬间完成，但是我可以郑重说明这是凑合的结果！但是实际上，这件事本应该可以立即快速有高质量的完成并验收！

**写在最后**

我比较喜欢工匠精神，一种时间打磨精品的精神，一种自由引导创造的精神。  

> 转自：极客重生
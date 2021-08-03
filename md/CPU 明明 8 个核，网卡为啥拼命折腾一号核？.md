> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIyNjMxOTY0NA==&mid=2247484717&idx=1&sn=2c1dd6c389c8476eb4fd178c714eaafc&scene=21#wechat_redirect)

中断机制
----

我是 CPU 一号车间的阿 Q，我又来了！

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYUPRKNunqktZtZ4Ir8H7NibwfO6Rrwvdu3XpoiaaNxTSW5I4xroZfmpwdbl0XW68sfwBIf9v6hq6Z5A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYUPRKNunqktZtZ4Ir8H7Nibw6icZEmTn3ibOtiaM9ZZq2jWvk1LfrUicyibC9EW8FOeDBmdWGLNJgzU9h1w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYUPRKNunqktZtZ4Ir8H7NibwR3QD5eZutE2RMArmLhmiaExCo3QOcYfAhPbeVIGO7Y1bmkaXemgKjnQ/640?wx_fmt=png)

8259A PIC
---------

为此，厂里单独组建了一个全资的子公司来负责这事儿，他就是`可编程中断控制器PIC`，外号 8259A，其他单位想联系我们都得通过这个 PIC，我们只需要和 PIC 进行对接就可以了。

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYUPRKNunqktZtZ4Ir8H7Nibw7VkH3jNoCXR8JO7pHs9YIzbFySuU5GNgmv3SGEzHf1aI507kt5x0bw/640?wx_fmt=png)

我们给办事单位都分配了一个编号，叫做`中断向量`。我们还准备了一个表格叫`中断描述符表IDT`，表格里记录了很多信息，其中就有处理这个中断号对应的函数地址。我们找 PIC 拿到编号后就执行处理函数就 OK 了。

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYUPRKNunqktZtZ4Ir8H7Nibw03rkGpTClLEy3KlWeJ5V9O7ycL9kqGDxNmwCIlERsz0H3RBVJI39tQ/640?wx_fmt=png)

这个表格有点大，足足有 256 项，咱 CPU 车间空间有限，放不下，就把它放在内存那家伙那里了，为了能快速找到这个表，专门添置了一个叫`idtr`的寄存器指向这个表格。

其实除了中断，我们在执行指令的时候如果遇到了`异常`情况，也会去这个表里执行异常处理函数，最常见的比如遇到了除数是 0，内存地址错误等等情况。

这种情况下，我们必须主动放下手里的活，去处理异常，所以我们也说`异常`是同步的，而中断不知道什么时候发生，所以是异步的。

APIC
----

8259A 干的挺不错的，不过后来咱们厂扩大规模，从单核 CPU 变成了多核，他就有点应付不过来了。

终于有一天，厂里召开会议，把 8259A 给撤了，成立了一个新的全资子公司叫`高级可编程中断控制器APIC`，名字就多了个高级两个字，干的活还是一样的。

不过你还别说，这两个字还真不是吹嘘，比 8259A 不知道高到哪里去了。

这个 APIC 的新公司一上台，就成立了两个部门，一个叫`I/O APIC`，负责接待那些要找我们办事儿的单位，一个叫`Local APIC`，以外包的形式入驻到我 CPU 的各个车间工作，因为就挨着我们办公，所以取名叫 Local。

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYUPRKNunqktZtZ4Ir8H7Nibw2mOuOPs39B09iaZtlKGcrzp2Eia03OqdXusHibnWXxxBiaCCYJ9VdfM2icg/640?wx_fmt=png)

`I/O APIC`收到中断信号以后，根据自己的策略就分发到对应的`Local APIC`，咱们八个车间就可以专心处理了，为我们省了不少事儿。

不仅如此，通过这个外包团队，我们八个车间还能向彼此发起中断请求，我们把这个叫做处理器间中断`Inter-Processor Interrupt`，简称 **IPI**。

中断亲和性
-----

每当网络中有数据包到来，网卡那家伙就发送一个中断消息过来，告诉我们去处理。

不过最近不知道怎么回事，网络数据量激增。咱们厂里明明有 8 个车间，他非得一个劲的只给我们发消息，搞得我们手头的工作老是被打断，忙得不可开交。

终于，我忍不住了，去找网卡那家伙理论了一番。不过他告诉我，这也不能怪他，分发给谁处理，那是 APIC 在负责。

想想也是，回头我就去了 APIC 那里，要求他们分摊一点给别的车间处理。

APIC 表示这他们做不了主，得让厂里来决定。

没过几天，厂里开了个会，参会的有各车间代表、APIC 负责人，还请了操作系统那边的相关代表过来。

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYUPRKNunqktZtZ4Ir8H7NibwC9ZXqI1ibicRGNNicOW1YydzRkfXDJIcVlJNbY6SWOpiaiaPCgIaLpJYhdA/640?wx_fmt=png)

会上，大家为了此事争执不休。

二号车间虎子：“阿 Q，谁叫你们一号车间是`Bootstrap Processor`，你们就多辛苦一点嘛”

三号车间代表：“你这话说的不合适，大家是一个 Team，要互相帮助！要不这样，既然有这么多单位要联系我们，咱就分下工，比如一号车间负责网卡，二号负责磁盘，我们三号负责键盘，以此类推”

五号车间代表：“你想的倒是挺美哦，键盘一天能发多少中断，网卡一天要发多少中断，你净挑轻松的干。这样吧，咱就用随机分发进行负载均衡你们觉得怎么样？”

八号车间代表：“随机个啥啊，多麻烦，依我看呐咱 8 个车间就轮流来呗”

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYUPRKNunqktZtZ4Ir8H7NibwH4BCTJYzkLZjJw1vnxIPCmFto2GaGSAn8MVP1dQo83FQfoA26A2RGg/640?wx_fmt=png)

这时，领导问操作系统代表有没有什么建议。

这代表站起身来，推了推眼镜说到：“几位有没有听过线程的`CPU亲和性`？”

大家都摇了摇头，问到：“这是个什么意思？”

“就是有些线程想绑定在你们之中的某一个核上面执行，不希望一会儿在这个核执行，一会儿在那个核执行”

我接过他的话：“好像是有这么回事儿，之前有遇到过，有个线程一直被分配到我们一号车间，不过我们对这个不用关心吧，执行谁不是干活啊，对我们都一个样”

代表摇了摇头，“唉，这可不一样！你们每个核的一二级缓存都是自己在管理，要是换到别的核，这缓存多半就没用了，又得重新来建立，这换来换去的岂不是瞎耽误功夫嘛！对于一般的线程他们倒是不关心，但是有些线程执行大量的内存访问和运算处理，又对性能要求很高的话，那就很在意这个问题了”

我们几个都恍然大悟，纷纷点头。

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYUPRKNunqktZtZ4Ir8H7NibwbibSz0rb86LNLkuW9qffib6sgakjlns5drkl1Upqb6eSNcg3AP8Hj1eg/640?wx_fmt=png)

虎子起身问到：“那你们是如何实现这个亲和性的呢？这跟我们今天的会议又有什么关系呢？”

代表继续回答说到：“我先回答你的第一个问题。线程调度是我们操作系统完成的工作，我们提供了 API 接口，线程通过调用这些接口表明自己的亲和性意愿，我们在调度的时候就能按照他们的意愿把线程分配给你们来执行。”

代表喝了一口水接着说到：“我再回答你的第二个问题。既然线程可以有亲和性，那中断也可以按照这个思路来分发啊！APIC 默认有一套分发策略，但是也提供亲和性的设置，可以指定谁哪些核来处理，这样不用把规矩定死，灵活可变，岂不更好？”

刚说完，会议室门口突然出现一年轻少年，挥手将操作系统代表唤了出去。

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYUPRKNunqktZtZ4Ir8H7NibwEuC2R5KL6dPXG52efiagVZumfhmME7QWfp5PQofWGOsz6XDN9nryib8w/640?wx_fmt=png)

接下来，我们详细讨论了这种方案的可行性，最后大家一致决定，就照这么办，我们一起提出了一个叫`中断亲和性`的东西，操作系统那边提供一个可配置的入口 **smp_affinity**，可以通过设置各处理器核的掩码来决定中断交由谁来处理，APIC 回去负责落地支持。

有了这套方案，再遇到网络高峰期，咱们一号车间的压力就有办法缓解了。

我们刚刚达成一致，操作系统代表返回会议室，神色凝重的说到：“不好意思各位，操作系统那边有点事情需要赶回去处理一下，先走一步了”

**未完待续 ······**

彩蛋
--

> 随着网卡的一声中断，一个新的数据包来到了这片土地。
> 
> 帝国网络部新来的年轻人显然没有意识到危险的到来 ······
> 
> _预知后事如何，请关注后续精彩 ······_

往期 TOP5 文章
----------

[真惨！连各大编程语言都摆起地摊了！](https://mp.weixin.qq.com/s?__biz=MzIyNjMxOTY0NA==&mid=2247484366&idx=1&sn=f8b0f10fc7c0d9b987271cee499f9092&scene=21#wechat_redirect)

[因为一个跨域请求，我差点丢了饭碗](https://mp.weixin.qq.com/s?__biz=MzIyNjMxOTY0NA==&mid=2247484178&idx=1&sn=7d8e5efe7cba41122a6d978a08058627&scene=21#wechat_redirect)

[完了！CPU 一味求快出事儿了！](https://mp.weixin.qq.com/s?__biz=MzIyNjMxOTY0NA==&mid=2247484072&idx=1&sn=ad1de598214dbb4eec652789d500d3a6&scene=21#wechat_redirect)

[哈希表哪家强？几大编程语言吵起来了！](https://mp.weixin.qq.com/s?__biz=MzIyNjMxOTY0NA==&mid=2247484076&idx=1&sn=890977e58f86a4fb3b6a26b487697bf8&scene=21#wechat_redirect)

[一个 HTTP 数据包的奇幻之旅](https://mp.weixin.qq.com/s?__biz=MzIyNjMxOTY0NA==&mid=2247484098&idx=1&sn=1c6a80bd949f875fa361f7b47654e5bd&scene=21#wechat_redirect)
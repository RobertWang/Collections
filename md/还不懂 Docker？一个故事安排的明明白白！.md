> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIyNjMxOTY0NA==&mid=2247487647&idx=1&sn=87275d2f356de79391afa13c137e9a86&scene=21#wechat_redirect)

程序员受苦久矣
-------

多年前的一个夜晚，风雨大作，一个名叫 **Docker** 的年轻人来到 Linux 帝国拜见帝国的长老。

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYWNJ9SbOfoM2MEt69FRJ6sTKMpp7BePvStB0Ulpl5MKqH3Rm8uvqIgTlXECBicEwwm6ehdCGaVah8A/640?wx_fmt=png)

“Linux 长老，天下程序员苦于应用部署久矣，我要改变这一现状，希望长老你能帮帮我”

长老回答：“哦，小小年纪，口气不小，先请入座，你有何所求，愿闻其详”

Docker 坐下后开始侃侃而谈：“当今天下，应用开发、测试、部署，各种库的依赖纷繁复杂，再加上版本之间的差异，经常出现在开发环境运行正常，而到测试环境和线上环境就出问题的现象，程序员们饱受此苦，是时候改变这一状况了。”

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYWNJ9SbOfoM2MEt69FRJ6sTb9SzZ5rwqX1VZoBFCI8sZ4Rh59ibFBwe2o92Uib99N7yvWOyGQtsLHHA/640?wx_fmt=png)

Docker 回头看了一眼长老接着说到：“我想做一个虚拟的**容器**，让应用程序们运行其中，将它们需要的依赖环境整体打包，以便在不同机器上移植后，仍然能提供一致的运行环境，彻底将程序员们解放出来！”

Linux 长老听闻，微微点头：“年轻人想法不错，不过听你的描述，好像**虚拟机**就能解决这个问题。将应用和所依赖的环境部署到虚拟机中，然后做个快照，直接部署虚拟机不就可以了吗？”

Docker 连连摇头说到：“长老有所不知，虚拟机这家伙笨重如牛，体积又大，动不动就是以 G 为单位的大小，因为它里面要运行一个完整的操作系统，所以跑起来格外费劲，慢就不说了，还非常占资源，一台机器上跑不了几台虚拟机就把性能拖垮了！而我想要做一个**轻量级的虚拟容器**，**只提供一个运行环境，不用运行一个操作系统，所有容器中的系统内核还是和外面的宿主机共用的，这样就可以批量复制很多个容器，轻便又快捷**”

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYWNJ9SbOfoM2MEt69FRJ6sTVlCDXG7qvdAvxTklpWKqLFg2JW4emG4rCiaX4libthnnCmCI0iaoicoG1g/640?wx_fmt=png)

Linux 长老站了起来，来回踱步了几圈，思考片刻之后，忽然拍桌子大声说到：“真是个好想法，这个项目我投了！”

Docker 眼里见光，喜上眉梢，“这事还真离不开长老的帮助，要实现我说的目标，对进程的管理隔离都至关重要，还望长老助我一臂之力！”

“你稍等”，Linux 长老转身回到内屋。没多久就出来了，手里拿了些什么东西。

“年轻人，回去之后，尽管放手大干，我赐你三个锦囊，若遇难题，可依次拆开，必有大用”

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYWNJ9SbOfoM2MEt69FRJ6sTkJww55F1gQECDqmkk4mpOoqrxicYMewN0WnkFanFe6aicDZMLYMEqfcA/640?wx_fmt=png)

Docker 开心的收下了三个锦囊，拜别 Linux 长老后，冒雨而归。

锦囊 1：chroot & pivot_root
------------------------

受到长老的鼓励，Docker 充满了干劲，很快就准备启动他的项目。

作为一个容器，首要任务就是限制容器中进程的活动范围——能访问的文件系统目录。决不能让容器中的进程去肆意访问真实的系统目录，得将他们的活动范围划定到一个指定的区域，不得越雷池半步！

到底该如何限制这些进程的活动区域呢？Docker 遇到了第一个难题。

苦思良久未果，Docker 终于忍不住拆开了 Linux 长老送给自己的第一个锦囊，只见上面写了两个函数的名字：**chroot & pivot_root**。

Docker 从未使用过这两个函数，于是在 Linux 帝国四处打听它们的作用。后来得知，通过这两个函数，可以修改进程和系统的根目录到一个新的位置。Docker 大喜，长老真是诚不欺我！

有了这两个函数，Docker 开始想办法怎么来 “伪造” 一个文件系统来`欺骗`容器中的进程。

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYWNJ9SbOfoM2MEt69FRJ6sTY1acvkU7O6ogvSNCdZ5UAUrUghMuxm3uutZWqCtibsIfhtY4FP5OMWg/640?wx_fmt=png)

为了不露出破绽，Docker 很聪明，用操作系统镜像文件挂载到容器进程的**根目录**下，变成容器的 **rootfs**，和真实系统目录一模一样，足可以以假乱真：

```
$ ls /
bin dev etc home lib lib64 mnt opt proc root run sbin sys tmp usr var


```

锦囊 2：namespace
--------------

文件系统的问题总算解决了，但是 Docker 不敢懈怠，因为在他心里，还有一个大问题一直困扰着他，那就是如何把真实系统所在的世界隐藏起来，别让容器中的进程看到。

比如进程列表、网络设备、用户列表这些，是决不能让容器中的进程知道的，得让他们看到的世界是一个干净如新的系统。

Docker 心里清楚，**自己虽然叫容器，但这只是表面现象，容器内的进程其实和自己一样，都是运行在宿主操作系统上面的一个个进程**，想要遮住这些进程的眼睛，瞒天过海，实在不是什么容易的事情。

Docker 想过用 HOOK 的方式，欺骗进程，但实施起来工作太过复杂，兼容性差，稳定性也得不到保障，思来想去也没想到什么好的主意。

正在一筹莫展之际，Docker 又想起了 Linux 长老送给自己的锦囊，他赶紧拿了出来，打开了第二个锦囊，只见上面写着：**namespace**。

Docker 还是不解其中之意，于是又在 Linux 帝国到处打听什么是 namespace。

经过一阵琢磨，Docker 总算是明白了，原来这个 namespace 是帝国提供的一种机制，通过它可以划定一个个的**命名空间**，然后把进程划分到这些命名空间中。

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYWNJ9SbOfoM2MEt69FRJ6sTLQJChFLZOafHprI3AHwtZ3lFExxGED79DrTPplyhlH8TibEUlEIKYUw/640?wx_fmt=png)

而每个命名空间都是独立存在的，命名空间里面的进程都无法看到空间之外的进程、用户、网络等等信息。

这不正是 Docker 想要的吗？真是踏破铁鞋无觅处，得来全不费功夫！

Docker 赶紧加班加点，用上了这个 namespace，将进程的 “视野” 锁定在容器规定的范围内，如此一来，容器内的进程彷佛被施上了**障眼法**，再也看不到外面的世界。

锦囊 3：CGroup
-----------

文件系统和进程隔离的问题都解决了，Docker 心里的石头总算是放下了。心里着急着想测试自己的容器，可又好奇这最后一个锦囊写的是什么，于是打开了第三个锦囊，只见上面写着：**CGroup**。

这又是什么东西？Docker 仍然看不懂，不过这一次管不了那么许多了，先运行起来再说。

试着运行了一段时间，一切都在 Docker 的计划之中，容器中的进程都能正常的运行，都被他构建的虚拟文件系统和隔离出来的系统环境给欺骗了，Docker 高兴坏了！

很快，Docker 就开始在 Linux 帝国推广自己的容器技术，结果大受欢迎，收获了无数粉丝，连 **nginx**、**redis** 等一众大佬都纷纷入驻。

然而，鲜花与掌声的背后，Docker 却不知道自己即将大难临头。

这天，Linux 帝国内存管理部的人扣下了 Docker 准备 “处决” 掉他，Docker 一脸诧异的问到，“到底发生了什么事，为什么要对我下手？”

管理人员厉声说到：“帝国管理的内存快被一个叫 Redis 的家伙用光了，现在要挑选一些进程来杀掉，不好意思，你中奖了”

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYWNJ9SbOfoM2MEt69FRJ6sTjjAQ6OGS7NKPI1HD1fHkJafgyZOyvga7nKSkQacypHPTCmUKsh0TiaQ/640?wx_fmt=png)

Redis？这家伙不是我容器里的进程吗？Docker 心中一惊！

“两位大人，我认识帝国的长老，麻烦通融通融，找别人去吧，Redis 那家伙，我有办法收拾他”

没想到他还认识帝国长老，管理人员犹豫了一下，就放了 Docker 到别处去了。

惊魂未定的 Docker，思来想去，如果不对容器中的进程加以管束，那简直太危险了！除了内存，还有 CPU、硬盘、网络等等资源，如果某个容器进程霸占着 CPU 不放手，又或者某个容器进程疯狂写硬盘，那迟早得连累到自己身上。看来必须得对这些进程进行管控，防止他们干出出格的事来。

这时候，他想起了 Linux 长老的第三个锦囊：**CGroup**！说不定能解这燃眉之急。

经过一番研究，Docker 如获至宝，原来这 CGroup 和 namespace 类似，也是 Linux 帝国的一套机制，通过它可以划定一个个的分组，然后限制每个分组能够使用的资源，比如内存的上限值、CPU 的使用率、硬盘空间总量等等。系统内核会自动检查和限制这些分组中的进程资源使用量。

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYWNJ9SbOfoM2MEt69FRJ6sTeUeOo1rJzGyhJ4x9FjCKCJrbUOvIzjibsO8avQPHMDRiaF7tzYUiapY1w/640?wx_fmt=png)

Linux 长老这三个锦囊简直太贴心了，一个比一个有用，Docker 内心充满了感激。

随后，Docker 加上了 CGroup 技术，加强了对容器中的进程管控，这才松了一口气。

在 Linux 长老三个锦囊妙计的加持下，Docker 可谓风光一时，成为了 Linux 帝国的大名人。

然而，能力越大，责任越大，让 Docker 没想到的是，新的挑战还在后面。

往期 TOP5 文章
----------

[那天，我被拉入一个 Redis 群聊 ···](https://mp.weixin.qq.com/s?__biz=MzIyNjMxOTY0NA==&mid=2247487533&idx=1&sn=49b600ef7eac342dad1f5a8048361099&scene=21#wechat_redirect)

[CPU 明明 8 个核，网卡为啥拼命折腾一号核？](https://mp.weixin.qq.com/s?__biz=MzIyNjMxOTY0NA==&mid=2247484717&idx=1&sn=2c1dd6c389c8476eb4fd178c714eaafc&scene=21#wechat_redirect)

[因为一个跨域请求，我差点丢了饭碗](https://mp.weixin.qq.com/s?__biz=MzIyNjMxOTY0NA==&mid=2247484178&idx=1&sn=7d8e5efe7cba41122a6d978a08058627&scene=21#wechat_redirect)

[完了！CPU 一味求快出事儿了！](https://mp.weixin.qq.com/s?__biz=MzIyNjMxOTY0NA==&mid=2247484072&idx=1&sn=ad1de598214dbb4eec652789d500d3a6&scene=21#wechat_redirect)

[哈希表哪家强？几大编程语言吵起来了！](https://mp.weixin.qq.com/s?__biz=MzIyNjMxOTY0NA==&mid=2247484076&idx=1&sn=890977e58f86a4fb3b6a26b487697bf8&scene=21#wechat_redirect)
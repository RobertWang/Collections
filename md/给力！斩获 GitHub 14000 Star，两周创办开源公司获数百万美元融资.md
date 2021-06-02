> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg4NDQwNTI0OQ==&mid=2247538296&idx=1&sn=30a931d3b51093c53b8b1b9aeb69ddd2&chksm=cfbab316f8cd3a00d3d01152831281ae1d4cf7a0b81ba8a45813d7c513fdfb8b746796bc6296&mpshare=1&scene=1&srcid=0602XLPBPHNgK8quPtxPQSnJ&sharer_sharetime=1622630585950&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

![](https://mmbiz.qpic.cn/mmbiz_jpg/Pn4Sm0RsAuhlURG4sTVgEtib8RZpmwiaQ9VVW6u20xqsnqRG43qQWpvvB7N31qbjibKtZcu5qP881qVKnqcgJ4eGQ/640?wx_fmt=jpeg)

作者 | 伍杏玲

出品 | AI 科技大本营（ID:rgznai100）

上世纪 90 年代初，21 岁大学生 Linus Torvalds 开源 Linux 操作系统，自此掀起全球开源浪潮。随后 “中国 Linux 第一人” 宫敏博士用手提肩背的方式将 20 盒磁带背回中国，磁带里装着 80G 容量的自由软件，组建起中国第一个自由软件库，点燃中国开源之火。

20 年时间滑过，中国开源力量在全球舞台上表现亮眼，喜讯不断，捷报连连：今年 3 月，开源被列入 “十四五” 规划；2020 年成立中国首个开源基金会开放原子开源基金会，推出中国首个国际通用开源协议木兰。在融资资本上，PingCAP 完成 D 轮 2.7 亿美元融资；TDengine 背后的涛思数据拿下 4700 万美元 B 轮融资；如今深受开发者喜爱的分布式数据库中间件 ShardingSphere，由其核心团队组建的商业公司—— SphereEx 完成了数百万美元天使轮融资。

在这背后，离不开两位开源 “赶集人”——Apache ShardingSphere 及 ElasticJob 创始人 & PMC Chair 张亮、Apache ShardingSphere PMC 潘娟，多年来孜孜不倦地投入。CSDN 专访两位 SphereEx 创始人，聊聊这一路走来，从普通从事开源的工程师到成功创办开源企业获得融资的心路历程。

左：张亮，右：潘娟  

时间拨回最初的启程，2015 年是张亮从事开发的第 10 年。“十”似乎是个有魔法的数字，每个人在 “十” 的节点上，或多或少遇见难忘的人和事。

这一年，张亮开源了第一个项目分布式调度框架 ElasticJob，当前 Star 数为 7K+。随后于 2016 年 1 月，开源数据库分库分表中间件 Sharding-JDBC（ShardingSphere 的前身），Sharding-JDBC 迅速引发开发者的关注度。

“早在 2008 年，我就想从事开源工作，当时自己开源了一个小项目，不过并不成功，star 数量寥寥可数。从 2008 年到 2016 年，这八年来我一直没有放弃做开源基础设施相关的项目，直到 2014 年起，我开始思考下一代基础设施将会如何发展。我预感分布式应是关键的发展技术，于是我逐渐找到适合发展的开源路径，找到毕生发挥余热的事业。” 张亮回忆往事。

谈及 ShardingSphere 五年的发展历程，每一步似乎走得稳妥与顺利：

一开始，Sharding-JDBC 是一套扩展于 Java JDBC 层的分库分表中间件，最初目标是透明化分库分表所带来的复杂度，包括数据源的管理、根据业务进行的 SQL 改写等。

随着张亮将它带进京东数科，在大公司里有了更大的落地应用场景。Sharding-JDBC 逐渐定位为面向云化的数据库中间件，从分库分表的 Java 开发框架演化成如今的分布式数据库生态体系。

2018 年 11 月，ShardingSphere 进入 Apache 基金会孵化器，仅用时 17 个月，便于 2020 年正式从 Apache 基金会毕业，成为 Apache 顶级项目。至此 Sharding-JDBC 也完成到 ShardingSphere，再到 Apache ShardingSphere 的转变。

当前的 ApacheShardingSphere 是一套开源的分布式数据库中间件解决方案组成的生态圈，它由 3 款相互独立，却又能够混合部署配合使用的产品组成。它们均提供标准化的数据分片、分布式事务和数据库治理功能，可适用于如 Java 同构、异构语言、云原生等各种多样化的应用场景。该项目在 GitHub 上收获 14000+ Star 数，近 170 家公司落地应用，拥有超过 200 位贡献者。

**![](https://mmbiz.qpic.cn/mmbiz_jpg/BnSNEaficFAYgTGBXg0ZvckYHA2iaBOsqO3ToytjEfeNjJlGPcZqK98xO4MJ2zC1xj07PfS6kAic0LAlMKFBLvWPQ/640?wx_fmt=jpeg)**

**五年野蛮成长，两周创立公司**

**他说 “尚未成功，还在逐梦路上”**

从 2016 年起，ShardingSphere 有了 5 年的技术和生态积累，且随着近几年国家大力支持基础设施建设，国内开源发展红火，让开源项目增添更多商业化可能性，张亮和潘娟意识到是时候集合更多资源共同将 ShardingSphere 做大做强。于是 SphereEx 应运而生。

SphereEx 的创办非常迅速。今年春节前，张亮和潘娟开始紧锣密鼓筹备公司事宜，春节后便拿到首轮融资。“如果不算春节休假的话，差不多两周左右的时间。” 张亮说。

公司名称 “SphereEx” 的含义深远：“Sphere”是 “ShardingSphere” 的延续，代表 “生态”，“Ex” 是扩展和探索，“SphereEx”代表“生态 + 扩展”，表示大家对 ShardingSphere 未来有更多的拓展和延伸，因为从技术产品规划上，除了 ShardingSphere 外，公司还推出其他产品，如数据库网格治理产品 Database Mesh，与 ShardingSphere 完全独立的分布式调度平台 ElasticJob 等，未来公司还有更多的开源项目和商业化项目。

从诞生到成立公司、获得融资，ShardingSphere 花了 5 年时间，似乎在技术、生态、资本各方面走得顺风顺水。“五年成长时间正是一个标准的开源项目开始向商业化转型所需时间”。

实际上，这一路走来遇到不少困难：当初 ShardingSphere  进入 Apache 基金会中间就历经波折。由于最初 ShardingSphere 是张亮一人完成项目的 80% 代码，Apache 基金会认为，这个项目属于一个人掌控的项目，暂时不适合进入。随后张亮积极在京东数科落地 ShardingSphere，让越来越多的人参与进开源项目，并逐渐组成团队来运营社区，让 ShardingSphere 变成一个友好、分散、生态建设良好的项目，才顺利从 Apache 基金会毕业。

又如  ShardingSphere 中间进行了很多次转型，从上文提到 ShardingSphere 经历几次更名就可以看出来，它可不是什么没有故事的开源项目。其中关键的两次重大变革就有：

一是从 Sharding-JDBC 演变成 ShardingSphere ，因为 Sharding-JDBC 一开始是 JDBC 的驱动端，而后研发了 Sharding-Proxy 工具，由于它已不再是单纯的 JDBC 启动端，以前的名称不合适，于是更名为 “ShardingSphere”，Sharding-JDBC 和 ShardingProxy 均为其子项目。

这中间的曲折演进过程不是张亮一个人掌握的，由于社区不断成长、开发者和企业的使用优化，自然而然进行迭代的。

“**ShardingSphere 像一棵拥有生命力的植物，它不是被某个开发者人工雕琢而成，而是吸取了众多开发者的营养贡献，自然成长发展。**” 张亮感慨道。

潘娟分享了一路见证 ShardingSphere 发展，总结了 ShardingSphere 深受开发者喜爱的原因：一是 ShardingSphere 给开发者和企业提供价值；二是赶上国内开源迅速成长的大环境；三是在大厂生根发芽，得到广泛传播；四是优秀团队支持。

张亮理智地表示：“**我们成立 SphereEx 公司后，并不代表成功，我们还在路上。只是以后大家能全情投入，共同朝着一个目标前进。现在远没有到论功行赏、评头论足的时候，正在快速发展中。我们始终坚持以开放的心态欢迎更多的人参与进来，一个人走得会很快，很多人会走得很远。**”

**![](https://mmbiz.qpic.cn/mmbiz_jpg/BnSNEaficFAYgTGBXg0ZvckYHA2iaBOsqOPyiaaIVCePZnia426uNibf0PLTPfS7VRaWql6vFWCwb2Cqgjp06UvU39Q/640?wx_fmt=jpeg)**

笔者在红杉成长中心办公室见到两位创始人，公司刚刚成立不久，两位创始人现在忙于各项公司运行事项：研发产品、员工沟通、布道宣传、各岗位招聘面试等。

张亮坦言四字箴言，**被推着转**。在创办公司过程中，他尝试了很多以往没有做过的事情，均是新鲜的体验，例如对接投资、组建公司细节等。这段时间张亮的眼界和格局开放不少，更期待和更多人完成更多有趣的事情。

如今张亮每天要求自己把写代码的时间降低到 40-50% 左右。其他时间是跟各种人沟通协调，“公司现在刚搭建，人力招聘、办公室等琐碎事情都需要花一些心思。”

毕竟开源不像在原来 “个人英雄主义” 的时代，现在做开源一定是“团体作战”，要想清楚该做什么事、往哪个方向发展，所以我们正组建团队，让更多人加入开源生态，减低个人瓶颈。

谈及今年的发展规划，张亮表示：

第一个目标是和团队做好磨合。目前虽求贤若渴，但分别设置团队 30、50、80 人的上限，假如团队到 30 人时，先观察团队的磨合度，如果团队磨合得还不错，那么继续招聘到 50 人；如果团队还有些摩擦力，就先放缓招聘速度，如此类推。目前办公室的规模是按照 50 人来设计的，因此今年的招聘目标为 50 人。

第二个目标是市场运营。今年将在社区运营下更多功夫，正确传达 ShardingSphere 的理念和技术。另外预计在海外开展社区运营工作，甚至成立办公室，让 ShardingSphere 走向国际。

第三个目标是产品研发，将投入约 20 人在开发垂直领域功能，让产品更加稳定。

同时张亮坦言，今年还没有商业计划，目前将全部精力用在产品研发、开源社区的建立上，让 ShardingSphere 为开发者和企业发挥其价值，才有更多商业化机会。

**![](https://mmbiz.qpic.cn/mmbiz_jpg/BnSNEaficFAYgTGBXg0ZvckYHA2iaBOsqOUuicXVGTwzDI9d46uDtYVV6QiaicNDpwPS4L8YeTopM3y06A6v6pamjoA/640?wx_fmt=jpeg)**

**与开源同成长**

尽管上文谈到国内开源发展的利好，可同时面临窘境：据 CSDN 《2020-2021 中国开发者调查报告》显示，82% 的开发者在开源上每周投入时间不超过 5 小时，每周在开源项目上投入时长超过 30 小时的仅占比 2%。77% 的开发者表示，不曾在开源上获得收入。

对此，潘娟分享了她与开源结缘的故事：

2018 年，张亮将 ShardingSphere 在京东数科做落地推广时，时任 DBA 的潘娟惊喜地发现 ShardingSphere 与自己一直想要的诉求完全相符，正是自己一直想做却没做的项目，于是转到张亮麾下，专门从事 ShardingSphere 的开源工作。

她坦言，当时初出茅庐，对开源一窍不通。但正是出于维护好、运营好项目的心态，驱使自己多吸收开源的知识来快速成长，同时积极参加开源 Meetup 活动、多与志同道合的开源人员沟通，心态慢慢开放和热衷开源文化，顺理成章地成为其中的 “赶集人”。

潘娟在开源分享会上，遇到不少想从事开源却没有时间的开发者，由于国内外开源环境不同，譬如初入职场的新人，工作繁忙着急上手，要是让他抽出时间来参加开源，确实较困难。

对此，潘娟建议道：

一是将开源组件引入公司项目，如果能将一个比较成熟和活跃的项目引入公司，自然而然可以在工作时间里参与到开源中。

二、假如无法在公司做开源，需要在业务时间参与开源的话，千万不要认为这是一件添加更多压力的工作，希望大家出于兴趣爱好而参与开源，享受这个过程，从而获得成长。并不一定规定大家在一周内必须参与多少时间，可以把时间拉长些，持续参加，慢慢积累。

三是加入 SphereEx 这类开源公司。潘娟进一步介绍参与开源的方式不一定就是贡献代码，还可以是回答问题，参与 MeetUp 活动等。譬如她刚开始参与 ShardingSphere 便是通过文档翻译形式参加，将项目的中文文档翻译成英文，提供给国外开发者阅读，而后才逐渐成长为 SphereEx 创始人。

**![](https://mmbiz.qpic.cn/mmbiz_jpg/BnSNEaficFAYgTGBXg0ZvckYHA2iaBOsqOHoPVFuCP8IMZsIKcMhGRrqicdwJpzvnXBwaI6fC2obKw3eiangqibyE5A/640?wx_fmt=jpeg)**

**开源不是一蹴而就，脚踏实地方能仰望星空**

对于开源生态建设，无疑是开源开发者的头疼难题，ShardingSphere 获得 14K+ Star，两位创始人根据过往在生态建设的经验，给出自己的思考：

张亮恳切表示，希望大家能融入开源的心态，做开源不是重复做一个轮子出来，就可以颠覆掉现有的开源生态。我们只有融入这个生态，共同玩转开源。建设开源很关键的一点是，看清自己该做什么、不该做什么，该配合别人做什么、该让别人配合自己做什么。

潘娟表示，一是只有做对他人有帮忙的开源产品，别人才愿意参与贡献。二是保持开放的心态和开源的理念，参与开源社区和基金会，共建生态。三是多考虑一线用户需求，从实际应用场景出发。四是核心团队人员开拓自身视野，多关注国际化、生态行业发展，带领大家走向更好的高度。

最后，谈及最近几年国内开源火热发展现状，张亮理性地提醒道，尽管当前国内开源发展现状看起来很乐观，但很多项目和国际顶级开源项目相比，差距较大。我们能否推出那些顶级开源项目还是未知数，未来还有很长的路要走。

另外，建设开源生态是需要积淀，不是简单促成的。大家不要以为，现在大环境看起来不错，便能一蹴而就获得成功。我们需理性观察，尽管当下开源火热，但其中成功的开源项目和开源公司不是现在出现的，而是历经三五年的静默期和积累期，才顺势发力，获得当前成就，所以希望大家脚踏实地，共建开源生态。

SphereEx 官网：http://www.sphere-ex.com/

ShardingSphere 官网：

https://shardingsphere.apache.org/

Apache ShardingSphere GitHub 地址：

https://github.com/apache/shardingsphere

张亮，ApacheMember，Apache ShardingSphere 及 ElasticJob 创始人 & PMC Chair，前京东科技架构专家、当当架构部总监。他擅长分布式架构、推崇优雅代码、热爱开源和技术分享。曾多次在大型技术峰会中担任出品人和分享嘉宾。出版书籍《未来架构——从服务化到云原生》。

潘娟，ApacheMember 及 Apache ShardingSphere PMC，前京东科技高级 DBA，曾负责京东数科数据库智能平台的设计与研发，现专注于分布式数据库 & 中间件生态及开源领域。被评为《2020 中国开源先锋人物》，多次受邀参加数据库 & 架构领域的相关会议分享。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Pn4Sm0RsAuglg1MhbQbn6UsPApBL7zZv3Oz3XqcvJJiaAFxrduwnH8WmBNNpCnPziaL1AH3IZP80HWca497RZrPQ/640?wx_fmt=jpeg)

```
2001 年创刊，20 年技术见证 
《新程序员001：开发者黄金十年》 
重磅来袭
扫描下方二维码，添加小助手
即刻加入 AI 科技大本营「读者群」
群内将不定期放送福利
快快加入吧！

```
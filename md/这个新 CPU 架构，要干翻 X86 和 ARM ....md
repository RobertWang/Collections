> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/TDUYN6tdf---hAFZiTl_nw)

如果你的手机正在使用骁龙 888，骁龙 778G 这样早两年发布的处理器，一定对 CPU 上那颗既能当主力大核，又能当中核的 Cortex-A78 移动旗舰核心不陌生。

最近，以 RISC-V 架构诞生的 SiFive P670 核心，性能比 A78 低 5%，但核心面积只用到后者的一半，简直离谱，成本、能效优势突出。

![](https://mmbiz.qpic.cn/mmbiz_png/wlQRw6PDv3oibmUSnrQO3GmWtaHK5IiaIGRMLZHUHYEJWJEuRBHJ2BDAY32FaiaB39wHxzPj8ianbiaLiaxWcDfPAAUg/640?wx_fmt=png)

谷歌官方也宣布安卓 15 上会正式支持 RISC-V 架构的微处理器。

这事可能会令许多普通消费者诧异，怎么就突然冒出个 RISC-V？芯片面积小的同时性能还这么强？

实际上，RISC-V 已经是继 x86、Arm 之后的第三大 CPU 架构。目前至少有 300 多家单位加入，其中有我们熟悉的阿里、谷歌、华为、三星、英伟达、高通、联发科、中科院计算所、麻省理工学院等。

![](https://mmbiz.qpic.cn/mmbiz_png/wlQRw6PDv3oibmUSnrQO3GmWtaHK5IiaIGsCZicOQiaVqOadmKArCTNvMuJdb0Xic25JDBLAeJGBibq1mWabeXhKMa8Q/640?wx_fmt=png)

前段时间，倪光南院士也在 RISC-V 中国峰会上表示：“**RISC-V 的未来在中国，而中国半导体芯片产业也需要 RISC-V，开源的 RISC-V 已成为中国业界最受欢迎的芯片架构**。”

![](https://mmbiz.qpic.cn/mmbiz_png/wlQRw6PDv3oD7ia3us4eYZoK3MbW8OLlGtEARESDicBSicT259uOcohs5ZzaLGdF1NCNSuvOuAZRKcqOtJcHE7cdw/640?wx_fmt=png)

凭借开源、免费的优势，RISC-V 未来很有可能真正做到三分天下，甚至在移动端直接接替同为精简指令集的 ARM 的霸主地位。

![](https://mmbiz.qpic.cn/mmbiz_png/wlQRw6PDv3q9ntr3HboavIZcAkjHw7Pw7icWIOBXet3ibCibzcGnXbECo2USqtxwxuxHWFXRNaJ84a2lhVeBLzuTw/640?wx_fmt=png)

_**RISC-V 的发展**_

RISC 的全程是 reduced instruction set computing，既精简指令集，V 是第五代技术。

![](https://mmbiz.qpic.cn/mmbiz_png/wlQRw6PDv3oibmUSnrQO3GmWtaHK5IiaIGibljWdIYqrjkicMYYpXfHRL9fkiag8watEyw9zZoamnFRu13OvF0ofKJg/640?wx_fmt=png)

RISC 源于 1980 年由图灵奖得主大卫・帕特森在加州大学柏克莱分校主持的 Berkeley RISC 项目。

其设计出的第一代 RISC-I 处理器，只有 32 条指令，并且晶体管只有同期 CISC（复杂指令集）设计芯片的一半，但性能却超过同期其他任何一种芯片，非常强大。

![](https://mmbiz.qpic.cn/mmbiz_png/wlQRw6PDv3oibmUSnrQO3GmWtaHK5IiaIG1RAB5ZvZzupLCXtlJm0A8IX3qzos8cyb9P5DklbgYhfx9DUnJBXofA/640?wx_fmt=png)

80 年代中后期，RISC 芯片迎来蓬勃发展，HP 和 SUN 开始推出搭载 RISC 芯片的计算机，包括工作站和服务器。

SUN 公司 SPARC 系统的成功，让更多人看到 RISC 的潜力，比如 IBM 公司也参与到了 RISC 的项目，其后 30 年间推出的 Power 系列 CPU，采用的就是 RISC 架构。

![](https://mmbiz.qpic.cn/mmbiz_png/wlQRw6PDv3oibmUSnrQO3GmWtaHK5IiaIGIO7z2iaiaia4QicTqc6iavJQ9MqPHhAukIcYTr5WjseDjKE5pOAGjNCVFCg/640?wx_fmt=png)

2010 年，加州大学伯克利分校的研究团队，起初只是单纯的想要一个能够更好为教学服务的指令集，他们当时考虑过 ARM、MIPS、SPARC 和 X86，但发现这些指令集不仅复杂，还有 IP 法律，授权难，授权费贵等问题。

于是决定自己操刀，全新的开源指令集架构项目 RISC-V 因此诞生。

![](https://mmbiz.qpic.cn/mmbiz_jpg/wlQRw6PDv3oibmUSnrQO3GmWtaHK5IiaIGPrYfZAFT4KermrsPn3tl9ugfnOvl7lUKX6iaw4qQ8Gd8CK6H0WuaHzw/640?wx_fmt=jpeg)

然而历史的巧妙之处在于，许多重大事件的诞生往往是无心插柳柳成荫。

**RISC-V 由于其精简、高效、低能耗、模块化、可拓展、免费开放、无历史负累低效指令等优势，很快出圈。RISC-V 团队意识到这份价值后，决定推出市场。**

![](https://mmbiz.qpic.cn/mmbiz_jpg/wlQRw6PDv3oibmUSnrQO3GmWtaHK5IiaIGeQOU9FthkHzwNj32jGxw1ftiaeJs7rgqeD0Do7LVXIsCxRYPH00NvMg/640?wx_fmt=jpeg)

2015 年，RISC-V 基金会成立，主打完全开源、无专利掣肘，有 BSD 许可证加持，这是商业公司最喜欢的开源许可证之一。

2019 年 6 月，RISC-V 基金会创始人之一大卫 · 帕特森宣布，将依托清华 - 伯克利深圳学院内部建设 RISC-V 国际开源实验室。

2020 年 3 月，RISC-V 基金会为规避美国对中国的贸易限制而将总部搬迁至瑞士，并更名为 RISC-V 国际。

![](https://mmbiz.qpic.cn/mmbiz_jpg/wlQRw6PDv3oibmUSnrQO3GmWtaHK5IiaIG0cRzav9R5ekh2iaw7AkoTogEiaXvNMuledIMnH7SnDV4kdJaKrozlung/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/wlQRw6PDv3q9ntr3HboavIZcAkjHw7Pwwb1ctHyia4ic8FqwF35pD4RFmH2h3VDoqBKDfvkN6J4YTibpBVlZhHO0g/640?wx_fmt=png)

_**指令集**_

CPU 发展至今，有非常多的架构，比如 X86，ARM、RISC-V、MIPS 等。虽然多， 但从逻辑角度上可以分为两种——“复杂指令集” 与 “精简指令集” 。

复杂指令集的代表是 X86，由英特尔、AMD 主导，我们电脑上的 CPU 几乎都是这一类。

![](https://mmbiz.qpic.cn/mmbiz_jpg/wlQRw6PDv3oibmUSnrQO3GmWtaHK5IiaIG6Jxrs1bwBjdbtymZm5cLsciaLV0Wj5S4XeXWtcmOicA9W8jYbdX99HjA/640?wx_fmt=jpeg)

“精简指令集” 的代表是 ARM，而 ARM 在移动端的普及苹果也功不可没。

早期苹果找上英特尔希望能研发出一款专为 iPhone 打造的高能效移动端芯片，但英特尔没有抓住这个机会，苹果就把目光瞄准了 ARM 指令集。

得益于精简指令集优秀的能耗比，现在苹果已彻底甩掉 X86 这辆大车，自研 ARM 指令集的芯片全面反哺到 Mac 端。

![](https://mmbiz.qpic.cn/mmbiz_jpg/wlQRw6PDv3oibmUSnrQO3GmWtaHK5IiaIGgrw0dg1m7Zzq3KoHDtUu3HXNcT5LIpw6KokbRChUvRNrGo3ib2go84w/640?wx_fmt=jpeg)

指令集是什么？苹果都嫌弃的复杂指令集，难道就一定比精简差吗？

![](https://mmbiz.qpic.cn/mmbiz_png/wlQRw6PDv3oibmUSnrQO3GmWtaHK5IiaIGfuqnRCvgrbEdnzssliaRGrmq37UQH7sXniaudH3lqpJyzxPANHPTazYg/640?wx_fmt=png)

首先，我们可以把硬件（CPU）想象成新建小区，软件想象成小区边的配套设施，比如医院，超市等。

而这个时候，小区和周边设施中间有一条河，想要互通无阻，就需要指令集充当桥的角色。

复杂指令集和精简指令集都能做到连接两地，但从一地到达另一地的出发目的却是可以不同的。

比如，小明只有一辆摩托车，他想要从小区过桥到超市买水，骑摩托车很快就能搞定。但如果他要从超市运输大量物资回家，就需要多走几趟。

![](https://mmbiz.qpic.cn/mmbiz_png/wlQRw6PDv3oibmUSnrQO3GmWtaHK5IiaIGIsyAqoia7ZLoprTSI0KgqyNdzKSOSf1SoPzpudXtu1lQrGuX5AENspw/640?wx_fmt=png)

而另一个人小虎，有一辆私家车，载货量足，运货的话，比摩托车强很多，但是只能走特定的大桥，如果只是买水，开车成本增高，虽然小虎也可以选择走路，但效率显然会变的更慢。

![](https://mmbiz.qpic.cn/mmbiz_png/wlQRw6PDv3oibmUSnrQO3GmWtaHK5IiaIGC3FYPJhZIRgMKuR1E3Ye9NN7d9RUxCXxtbvuckBtNj0fnKPOCoFmnA/640?wx_fmt=png)

所以就是，精简指令集和复杂指令集的优劣，要根据场景和需求来决定。

**总的来说，精简指令集适合移动端，能耗低，省电，性能会差点。复杂指令集适合 PC，处理速度快，能力更强，但比较臃肿，有时候不够高效。**

当然，这也并非绝对，实际上两者正在取长补短，互相学习。

![](https://mmbiz.qpic.cn/mmbiz_jpg/wlQRw6PDv3oibmUSnrQO3GmWtaHK5IiaIGoEfsiah95E5eyRkgz2XibDnstZAOelBMln194J3iaM3OUsJm11V2AIMuw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/wlQRw6PDv3q9ntr3HboavIZcAkjHw7Pw2vbOZLe86n36bY3nZRVPJC2BicBDcj1dowPia2ekQJXTic7MbsVxIPSeA/640?wx_fmt=png)

_**RISC-V 的特点**_

错位竞争并不可怕，可怕的是两个相似的对手在同一个赛道卷。RISC-V 和 ARM，同宗同源。既然 ARM 生态已经如此强大，为何还有许多开发者选择 RISC-V 使其成为第三大 CPU 架构？

原因，主要有以下几种。

**1. 更简洁**

在 19 年的一份各大指令集手册对比中，RISC-V 的页数只有 236 页，字数 76702 字，而 ARM-32 和 X86-32 的页数来到了 2000 多页，X86-32 的字数更是超过 200 万。简洁通常意味高效、可靠。

![](https://mmbiz.qpic.cn/mmbiz_png/wlQRw6PDv3oibmUSnrQO3GmWtaHK5IiaIGn1f3nYQ3ZToiaC6jcZ1m75L7srEh7icemMTfGQ5Yk077Zzz8CqYzndlQ/640?wx_fmt=png)

**2. 入门门槛低、设计简单**

ARM 和 X86 在定义新的架构时必须考虑兼容已有的技术，所以它们的规范文档多、指令数目多，晦涩难懂，初学者门槛高，不过在生态方面目前是占据绝对优势的。

反之， RISC-V 作为一个新生架构，不用考虑那么多，对设计者更加友好，也因其更简洁的设计，可以实现前面提到的同为精简指令集，但却可以以一半面积达到相同性能的实力。

![](https://mmbiz.qpic.cn/mmbiz_png/wlQRw6PDv3oibmUSnrQO3GmWtaHK5IiaIG1rQr1GMHZY12yahQKZ7DRbEHGBaKmqoICticZianIicuOz6kicOFKAribFw/640?wx_fmt=png)

**3. 易于移植，通用性强**

RISC 的设计们认为一个设计良好的指令集应该是开源且可以被任何人使用的，所以 RISC-V 架构的 CPU，是可以根据具体场景选择适合指令集的开源架构。

比如它可以用作服务器 CPU、物联网设备的 CPU、移动穿戴设备的 CPU、工控 CPU 、甚至是用在比指头还小的传感器中当 CPU。

![](https://mmbiz.qpic.cn/mmbiz_jpg/wlQRw6PDv3oibmUSnrQO3GmWtaHK5IiaIGy7ZeM0k1Yeu09nwIZNibg5aNPK2UXGD9hRwhqqsibLTj27IXZ2bRtJeA/640?wx_fmt=jpeg)

通用，意味资源可反复利用，软件的开发成本从而有效降低，因此开发商也能将精力更加集中在硬件设计上。

换言之，哪怕你一开始造计算机，也可以入局智能手表；一开始只做屏幕图显，未来也有机会上太空造航天器。

而这些正是得益于算法的强通用性。但如果你用的是 ARM 架构，由于 ARM 专利技术的私密性，移植就不会那么顺利了。

**4. 成本因素**

这是最最重要的原因。

许多人对 ARM 不了解，认为这个和苹果、联发科、高通、华为麒麟都有关系的架构是开源的，但实际上并不是。

它是商业授权指令集，IP 是需要收费的，ARM 的收费项目包括前期授权费（License）、版税（Royalty）和技术咨询服务费。

值得一提的是，在 23 年年初时，ARM 还曾有过更改授权模式的想法，它想根据销售设备的价格而抽取资金，这个想法当然遭到手机制造商们的集体抗议，因为这会让成本显著增加。

![](https://mmbiz.qpic.cn/mmbiz_png/wlQRw6PDv3oibmUSnrQO3GmWtaHK5IiaIGAA1dIekPFwWMlufaufyDyosIyAvmcz58M7Dw56iciaUDnWKtYJaaXE2A/640?wx_fmt=png)

此外，客户在使用 ARM 相关技术前，还需签署保密协议，因此也有人认为这类专有技术阻碍了公共教育用途和安全审核，还有软件和操作系统的自由发展。

![](https://mmbiz.qpic.cn/mmbiz_jpg/wlQRw6PDv3oibmUSnrQO3GmWtaHK5IiaIGHfXVrrVO3vsQ3nicCzaNOfBFOksvNn9W0onXx0QjOLu4VoSUPCBm4sg/640?wx_fmt=jpeg)

（RISC 还有高度规则的指令流水线、每条指令 (CPI) 的时钟周期数较低等显著等特点。）

可能迫于 RISC-V 的压力，ARM 曾偷偷上线一个网站：其中列举 RISC-V 的几大 “罪证”，包括：成本、生态系统、碎片化风险、安全性问题、设计验证。最后迫于舆论，ARM 关闭了该网站...

![](https://mmbiz.qpic.cn/mmbiz_jpg/wlQRw6PDv3oibmUSnrQO3GmWtaHK5IiaIGJfJufKLDicnfXLcGLT9bRkWE4JGCoQ2wibnIVCaPgzxVv0BZlg4kea2Q/640?wx_fmt=jpeg)

做个总结， RISC-V 作为后发技术，吸取前者经验，能做到相对简洁高效，指令集开放、开源，对小公司友好，生态虽目前仍不成熟，但发展潜力巨大。

![](https://mmbiz.qpic.cn/mmbiz_png/wlQRw6PDv3q9ntr3HboavIZcAkjHw7PwhGrhPGZQ73ZvnU7zjyfFku1b4RR0icHlnblaxq9mY5caR62CEfq2yQQ/640?wx_fmt=png)

_**前景和担忧**_

目前在嵌入式、物联网这块这块，RISC-V 凭其功耗优势，已是稳步前进。

![](https://mmbiz.qpic.cn/mmbiz_png/wlQRw6PDv3oibmUSnrQO3GmWtaHK5IiaIGXnFmiaKpYBY43UlibADM40l0YgxWpS6hvtQaNic6EdaHArm6BtpicbLg4A/640?wx_fmt=png)

开源的 RISC-V，毫无疑问能吸引更多玩家入局。但玩家一多，过度自由化的风险就会越高，存在这些玩家在已有基础上制定自有标准，研发核心各自为政互不兼容等情况。

所以，RISC-V 需要做的是避免 IP 过度碎片化，不然 MIPS 在多个方面就是它的前车之鉴。

![](https://mmbiz.qpic.cn/mmbiz_jpg/wlQRw6PDv3oibmUSnrQO3GmWtaHK5IiaIGJpBzXW1DWuqGsRqibYXiciaFZFffG2v5UxJicy670yuvTffFIpLUPNicPnQ/640?wx_fmt=jpeg)

另外，RISC-V 一旦大规模普及，调用的接口也会越来越多，指令集也不排除被一些厂商进一步定制扩展，而旧的底层指令集因为兼容性也不能丢，所以后续是否还能做到如此高效开放，是不是又会走上和 ARM 一样的老路，还有待商榷。

![](https://mmbiz.qpic.cn/mmbiz_jpg/wlQRw6PDv3oibmUSnrQO3GmWtaHK5IiaIGoKtRGArHjzQekmj4sVfia4r9G3WNNVPChrRicR7tA6icAP4QlhDMPywsw/640?wx_fmt=jpeg)

当然，解决办法也不是没有，那就是 RISC-V 需要一些高端玩家来共同制定一个通用标准。

![](https://mmbiz.qpic.cn/mmbiz_jpg/wlQRw6PDv3oibmUSnrQO3GmWtaHK5IiaIGJH5FyBOg3McLPC8uG10GuYYeFAxPu1qB3HqicB4xFVy9pX0BJLMhRmg/640?wx_fmt=jpeg)

比如，把指令集标准化得让其他人心服口服，这样就能够避免过渡碎片化。

同时，高端玩家还能利用自身影响力和财力，带动软件生态建设，开发大量软件工具，甚至直接设计相关的高端硬件，拿下以 CPU 为核心技术的市场。

RISC-V，目前最缺的就是这么一款王炸产品。

![](https://mmbiz.qpic.cn/mmbiz_jpg/wlQRw6PDv3oibmUSnrQO3GmWtaHK5IiaIGoKwDaR1G6RdqCpETdun6xk8n3KsfG067TgeTIBZbEe72qmk7dQkSJw/640?wx_fmt=jpeg)

但新的问题随之而来，巨头纷纷入局，虹吸效应显现，而且人人都不是慈善家，人多意见多，所以原本 RISC-V 的建立者话语权就会被进一步稀释。

入局的人会想办法建立一个长期稳定的商业模式。而建立者们可能只是想研发一个完全开源、无专利掣肘的指令集新架构服务于各行各业。

那么在多方利益对抗权衡后的 RISC-V，还能做到不忘初心吗？

当然，这些都是遥远的后话。RISC-V 目前的首要目标是活下去，先活着，再谈赚钱。它首先威胁的是 ARM 的市场。

![](https://mmbiz.qpic.cn/mmbiz_png/wlQRw6PDv3oibmUSnrQO3GmWtaHK5IiaIGVfJm1k6MGoPXnhFue33XOADUDsJSpNWRI7KLtjicGJIgiaASQibZCtORQ/640?wx_fmt=png)

可能不久后，我们就有机会用上另一种全新架构的 CPU，作为消费者而言，多一个选择，总比现在老气横秋进步缓慢的市场更有趣，不是吗。

_**参考资料**_：

维基百科：RISC

快科技：安卓 15 率先支持 第三大 CPU 架构 RISC-V 手机版性能逼近 A78

CSDN 博客：RISC-V 的前世今生

知乎：为什么 RISC-V 将超越 ARM？

图源网络

_**编辑**_：辉仔
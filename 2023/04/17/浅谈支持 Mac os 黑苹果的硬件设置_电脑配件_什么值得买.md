> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [post.smzdm.com](https://post.smzdm.com/p/ao9dpwzn/?send_by=3547620770&invite_code=zdm4ptnapkinv&zdm_ss=iOS_3547620770_&from=singlemessage)

浅谈支持 Mac os 黑苹果的硬件设置
====================

2022-09-27 20:47:40 87 点赞 581 收藏 50 评论

前言
--

自 2020 年 11 月，苹果在新品发布会正式发布了首款针对 Mac 电脑的自研芯片 M1 以来，苹果已经逐渐抛弃了合作了多年英特尔 CPU 平台，开始在自研芯片的道路上渐行渐远，随着苹果 mac 端的软硬件一体化，mac 黑苹果的前景顿时黯淡了下来。鉴于苹果系统每次发布新系统，都会兼容大约四五年的产品，所以，理论上黑苹果还能持续个四五年是没问题的。但在此之后，如果新的系统不再支持 X86 平台，那么黑苹果是否还能继续，目前还不明朗，所以现在折腾黑苹果，且行且珍惜吧。

前面一篇文章里推荐了黑苹果经常要用的一些软件，这里说说支持 Mac os 的硬件。本文仅限于讨论[台式机](https://www.smzdm.com/fenlei/taishiji/)，笔记本由于配置上相对固定的原因，要完美黑苹果挺难，不在本文讨论之列。

硬件配置
----

黑苹果的硬件和白苹果越接近约好，这样黑起来才比较顺利，最后的完成度也比较高，能够最大程度获得和白苹果相近的实用体验。

### 1.CPU  

理论上来说，只要是 intel 的 CPU，或者部分 AMD 型号 CPU，都可以运行黑苹果，关键在于核显能否顺利驱动。大部分黑苹果都选择了 intel 的，因为好驱动。不过有些较老的型号就不要考虑了，性能跟不上，也不支持最新的 mac os 系统。

目前黑苹果最容易安装，功能最全，运行最稳定的平台，是第七至第十代酷睿平台（2022 年新版本 macOS Ventura 官方直接砍到七代以前）。这几代的 CPU 核显，Mac 下均能顺利驱动，可以不需要配置独显就能正常实用。

2022 年以来，十二代酷睿大量上市，相比十代和十一代，十二代酷睿无论是单核还是多核性能提升都非常大，至少桌面端目前在纸面上轻松超越了苹果 M1 和 AMD 锐龙 5000 系列这两个对手，举个例子，中端型号 i5-12400 在仅有 6 个大核心和频率更低的情况下，理论性能测试和前两代酷睿高端型号持平（默频），甚至个别项目小幅领先（主要是单核），因此对于黑苹果性能要要求的用户，还是推荐考虑下十二代酷睿桌面 CPU，但核显无法驱动会有一定功能残缺，需配置免驱独显。

这里 12 代 CPU 首推 i5-12400F（不带 F 的型号也可以，但是核显也没法用，反而贵了 100），因为这个型号的 CPU 为纯大核，相对好驱动一些。

[百科](https://wiki.smzdm.com/)[![](https://qny.smzdm.com/202204/06/624d48e9d623e567.jpg_a200.jpg)](https://wiki.smzdm.com/p/prme687/)[intel 英特尔 酷睿 i5-12400F CPU 2.5GHz 6 核 12 线程](https://wiki.smzdm.com/p/prme687/)这款处理器采用 i5-12400F 盒装，CPU6 核心 12 线程，睿频频率至高可达 4.4GHz，一芯多用，状态稳定，高效运行、流畅稳定，强大的多任务处理能力，效率大幅提升。 _值_0 _点评_16 _原创_113 _好价_153 [去购买](https://go.smzdm.com/b63539d4c0bd1e54/ca_aa_yc_163_ao9dpwzn_16292_1687_1519_0)[看百科](https://wiki.smzdm.com/p/prme687/) ¥535 起

### 2. 主板  

黑苹果可考虑微星、技嘉、华硕、华擎等，近两年技嘉的 Vision D/G 系列以及微星的 MORTAR 迫击炮系列在黑苹果世界势头比较猛，安装案例也很多（意味着获得高质量的 EFI 文件比较容易），其它方面主要就是避免选择 BIOS 太烂的品牌，且 BIOS 设置里拥有 CFG Lock 选项的最佳，此项关联主板 MSR 0xE2 寄存器是否可读写（也就是 NVRAM 读写支持），对于逐渐成为主流的新一代引导工具 OpenCore 来说，解锁此项将带来更好的体验，BIOS 中没有该选项会导致额外的折腾流程；

如果选择的是 12 代酷睿，那么 B660 主板是比较有性价比的选择，这里我用的是技嘉（GIGABYTE）小雕 B660M AORUS ELITE DDR4，网上有比较多的 EFI 文件。

[百科](https://wiki.smzdm.com/)[![](https://y.zdmimg.com/202208/09/62f222714167f7613.jpg_a200.jpg)](https://wiki.smzdm.com/p/x181vq4/)[GIGABYTE 技嘉 B660M AORUS ELITE DDR4 小雕 MATX 主板（ Intel LGA1700、B660）](https://wiki.smzdm.com/p/x181vq4/) _值_0 _点评_2 _原创_5 _好价_15 [去购买](https://go.smzdm.com/c6dac40d6da4de52/ca_aa_yc_163_ao9dpwzn_16292_1687_1519_0)[看百科](https://wiki.smzdm.com/p/x181vq4/) ¥1029 起

### 3. 内存  

内存是黑苹果最没要求的硬件之一了。除了远古时期的 AMD 专用内存条可能存在问题之外，其它任意品牌、任意颗粒、任意频率均可；保证流畅运行的话，8G 起步吧。如果使用核显的话，尽量选择高频内存，性能会好一点。

### 4. [显卡](https://www.smzdm.com/fenlei/xianka/)

显卡值得说道的比较多，独显 N 卡及 A 卡及核显都有可以顺利驱动的型号，下面分开说说。

**【独显】**

**N 卡：**N 卡在 mac os 的受支持情况远分为以下几种情形。

1. 6-10 系（Pascal）部分显卡可以在 mac 下通过 WebDriver 驱动，但是最高只能安装 macOS 10.13.6 High Sierra，以后的 mac os 版本没有驱动了；

2. 20 系（Turing，包括 16 系）以及 30 系（Ampere）不被任何 macOS 版本支持，所以 RTX20-30 一系列显卡不要想黑苹果了；

3.Kelper 核心的一些 N 卡（部分 GT600 和 GT700 系列）一直免驱到 macOS Big Sur，但官方不再支持 macOS Monterey，需要使用第三方驱动补丁才能运行。但是否能顺利使用也得看人品，我曾折腾过一块 gt720 来安装 big sur，但是就是无法免驱，开机黑屏，最后只好放弃。  

有网友总结过可以吃上黑苹果的 N 卡型号如下：  

GTX Titan (GK 110 核心) 、GTX Titan Black(GK 110 核心) 、GTX 780 Ti 、GTX 780 、GTX 770 、GTX 760 Ti 、GTX 760 、GT 740 (GK107 核心)、GT 730 (GK208 核心)、GT 720 、GT 710 (GK208 核心)、GTX 680 、GTX 670 、GTX 660 Ti 、GTX 660 (GK 104 核心) 、GTX 650 (GK 107 核心) 、GT 640 (GK 107/208 核心) 、GT 635 、GT 630 (GK 107/208 核心)、Quadro K6000 、Quadro K5200 、Quadro K5000 、Quadro K4200 、Quadro K2000D 、Quadro K2000 、Quadro K600 、Quadro K420 、Quadro 410 、NVS 510。

不同的型号可能可以支持到不同的版本，但大部分最多到 10.13.6，所以如果要体验最新版 mac os 的，不要选择 N 卡。

**A 卡：**由于早些年苹果在自家电脑上的独显大部分以 A 卡为主，所以 mac os 支持情况比 N 卡好很多，很多型号可以直接免驱，是组建最新版高性能黑苹果的首选。

1.2019 年之前有一些型号是可以直接免驱的，性能虽然不行，但是胜在免驱便宜，现在矿难两三百可以入手，如果你的 CPU 核显不支持，但又想体验黑苹果且对显卡性能要求不高的话，弄一块这样的免驱老卡是比较合适的选择吗，我就用了一年多的 RX560。  

具体型号如下：,Rx 460/ 560 560d，Rx 470 470d / 570，Rx 480/580（580 需 2304sp 版本），Rx 590(不能是 GME 版)，Vega 56 ，Vega 64 ，Vega VII ，RX 560 XT。

2.2019 年推出的仙后座 RX5000 系列，在 macOS Catalina、macOS Big Sur、macOS Monterey 上是免驱的，该系目前性能最高 RX 5700XT（性能接近 RTX2070，Navi 10 核心）；

3. 目前性能最好的 A 卡是 RX6000 系列，2021 年 4 月 苹果从 macOS 11.4 起添加了对 Navi21 系列芯片的原生支持，对应 RX6800/RX6800XT/RX6900XT/W6800/W6900X，2021 年 10 苹果发布的 macOS 12.1 起提供了对 RX 6600XT 的支持，虽然官方没有写明，但同为 Navi23 芯片的 RX 6600 以及专业卡 W6600 也能免驱。

所以鉴于上述情况，如果对显卡性能有要求，性能最高的黑苹果 A 卡是 RX6900XT。不过一般情况下选择 RX6600XT 就够用了，现在矿潮已过，不到 2000 可以入手。不过买的时候注意鉴别，此型号矿卡较多。

[百科](https://wiki.smzdm.com/)[![](https://y.zdmimg.com/202108/26/6127392f632637330.jpg_a200.jpg)](https://wiki.smzdm.com/p/15e01zy/)[SAPPHIRE 蓝宝石 RX 6600 XT 8G D6 超白金 OC 显卡 8GB 银黑色](https://wiki.smzdm.com/p/15e01zy/)这款显卡核心频率可加速至 2607MHz，拥有 2048 个流处理器，拥有 8GB GDDR6 显存，显存速率 16Gbps，拥有 ARGB 炫彩灯效，使用超白金散热系统。接口：DP 1.4 * 3，HDMI 2.1 * 1。 _值_0 _点评_6 _原创_7 _好价_78 [看百科](https://wiki.smzdm.com/p/15e01zy/)

如果怕买到矿卡，也可以考虑今年新出的 RX6×50 系列，新型号杜绝了矿的风险，只是要使用仿冒 ID，不能直接免驱。我自己用的是下面这款，暗黑犬 RX6650XT。

[百科](https://wiki.smzdm.com/)[![](https://qny.smzdm.com/202208/09/62f1c43b2b0af8397.jpg_a200.jpg)](https://wiki.smzdm.com/p/60w0w0o/)[POWERCOLOR 撼讯 RX 6650XT 暗黑犬 显卡 8GB 白色](https://wiki.smzdm.com/p/60w0w0o/) RX6650XT 显卡，采用 2 个 100mm 独特造型的静压风扇，配备双滚珠轴承技术，配备 3*6mm 镀镍铜热管，横向铝质鳍片设计，支持风扇智能启停技术。采用 Navi23 核心，8G GDDR6 显存，游戏频率 2486MHz（OC）、2410MHz（Silent），Boost 频率 2689MHz（OC）、2635MHz（Slient）。1 个 HDMI 2.1 接口，3 个 DP 1.4 接口，建议电源 600W。 _值_0 _点评_6 _原创_10 _好价_65 [去购买](https://go.smzdm.com/492217643b76571b/ca_aa_yc_163_ao9dpwzn_16292_1687_1519_0)[看百科](https://wiki.smzdm.com/p/60w0w0o/)

此卡在黑苹果下 metal 模式跑分还可以。  

[![](https://qnam.smzdm.com/202209/27/6332837c739bc3745.png_e1080.jpg)](https://post.smzdm.com/p/ao9dpwzn/pic_2/)

这里特别提醒下：RX6700XT 系列，包括新出的 RX6750XT 等，由于核心不同，不能免驱，请大家注意不要选择。  

按照苹果公布的信息，所有 Polaris（对应 RX4x0 和 RX5x0）及更早的 AMD 显卡即将被 2022 年下半年发布的 Metal 3 API 放弃支持，Metal 3 将仅支持以下显卡芯片。所以如果想让黑苹果系统保持更新，就需要配备更新一点的显卡：Radeon RX Vega（56、64、VII）、Radeon RX5000（5300、5500XT、5600、5600XT、5700、5700XT、W5500、W5700）、Radeon RX6000（6600、6600XT、6650XT、6800XT、6900XT、6950XT、W6600、W6800，所有 6×50 显卡都需要仿冒设备 ID），所以 RX460-590 的用户，可以考虑升级显卡了。

**【核显】**

核显一般酷睿 10 代及以前的型号可以找到驱动，但是奔腾的赛扬处理器的核显目前无法驱动，AMDcpu 的核显一般无法驱动；核显中的 Intel UHD Graphics 630、Intel Iris Pro Graphics 支持最新的 metal3。

### 4. 蓝牙 & WiFi

蓝牙和 wifi 是一个模块，放在一起说，一般通过购置 pci 接口的无线网卡来解决。

在 2020 年以前，大家一般通过购置与白苹果一样的网卡型号来解决。后来 OpenIntelWireless 项目开发了驱动，英特尔网卡也可以在 macOS 上使用了，但还需完善。

**博通：**台式机推荐直接买 PCI 接口的二合一免驱款，免折腾完整支持苹果服务，隔空投送、接力、随航、通用控制等白苹果的功能都能实现，使用上最接近白苹果，缺点是 WiFi5 + 蓝牙 4.x，落后整一代；具体的型号如：BCM943602CDP、BCM943602CD、BCM94360CD、BCM943602CS、BCM94360CSAX、BCM94360CS、BCM94352Z 等等，我用的是这款：BCM94360CD CS2 台式 PCIE 千兆双频无线网卡。

京东 [![](http://a.zdmimg.com/202209/27/63328ac45fc3c5944.png_a200.jpg) ](https://go.smzdm.com/b0f85722edb5cbd3/ca_aa_yc_163_ao9dpwzn_16292_1687_1519_0) [适用黑适用苹果免驱 BCM94360CD/2CS 台式机双频千兆隔空接力 PCIe 无线网卡 BCM94360CS2](https://go.smzdm.com/b0f85722edb5cbd3/ca_aa_yc_163_ao9dpwzn_16292_1687_1519_0) 357 元 [去购买](https://go.smzdm.com/b0f85722edb5cbd3/ca_aa_yc_163_ao9dpwzn_16292_1687_1519_0) 

**英特尔：**AX200、AX201，这两款支持 Wifi6 + 蓝牙 5.0，目前解决了能无线上网的基本问题，但是空投什么的不支持，对苹果服务没有需求的可以先凑合，等待驱动完善；

### 5. [硬盘](https://www.smzdm.com/fenlei/yingpan/)

算是黑苹果下比较好驱动的硬件了，除了 eMMC 的硬盘无法驱动（常见于某些平板或者低端笔记本），基本上所有的 SATA 硬盘都支持，少部分 NVME 协议硬盘无法完美工作，甚至安装的时候还会造成内核恐慌，如三星的部分型号不支持 trim，大家在选购时尽量避开：三星 Samsung 950 Pro，三星 Samsung 960 Evo/Pro，三星 Samsung 970 Evo/Pro（玄学问题非常多 出现玄学问题建议屏蔽掉），还有镁光 2200S 据说也不是太稳定。具体大家可以先上网搜一下，排除掉一些。

我自己用的是海盗船 MP600，整体不错，mac 下可以跑满速度。

[![](https://qnam.smzdm.com/202209/27/63328c8a3b6b28735.png_e1080.jpg)](https://post.smzdm.com/p/ao9dpwzn/pic_3/)

[百科](https://wiki.smzdm.com/)[![](https://y.zdmimg.com/202105/31/60b4ba63e0c001161.jpg_a200.jpg)](https://wiki.smzdm.com/p/qxg978j/)[USCORSAIR 美商海盗船 MP600 NVMe M.2 固态硬盘 500GB（PCI-E4.0）](https://wiki.smzdm.com/p/qxg978j/)海盗船 MP600 是标准的 M.2 2280 外形，基于群联 PS5016-E16 主控方案，3D&nbsp;TLC 闪存，容量 500GB、1TB、2TB，标称持续读写速度最高可达 4950MB/s、4250MB/s。此外，MP600 系列散热控制到位，并没有使用太夸张的散热片，有的品牌铜块、热管都用上了。使用寿命方面，2TB 可达 3600TBW，1TB 版也有 1800TBW，500GB 版则是 850TBW。 _值_0 _点评_45 _原创_28 _好价_409 [看百科](https://wiki.smzdm.com/p/qxg978j/)

### 6. 其他硬件及外设  

电源、散热、[机箱](https://www.smzdm.com/fenlei/jixiang/)等均可根据个人喜好选择，只有 ITX 机箱需要特别额外注意各配件尺寸。

声卡一般主板上 efi 驱动可以解决，实在难以搞定的，可以考虑购置免驱型号的外置声卡。  

[摄像头](https://www.smzdm.com/fenlei/shexiangtou/)一般免驱，部分 windows hello 摄像头在 mac 下虽不能用作人脸识别登录，但是可以当普通摄像头使用。

指纹识别在 mac 下是用不了的，不用尝试弄出 touch id。

总结
--

台式机安装 mac 是比较简单的，根据网上的成熟方案，选择相应的硬件，抄作业配好 EFI，基本上都可以有不错的使用体验。核心是搞定显卡、声卡、网卡，其他硬件基本上问题都不大。

作者声明本文无利益相关，欢迎值友理性交流，和谐讨论～

![](https://res.smzdm.com/pc/pc_shequ/dist/img/the-end.png)
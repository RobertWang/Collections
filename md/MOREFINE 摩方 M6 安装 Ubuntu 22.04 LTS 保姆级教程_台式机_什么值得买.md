> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [post.smzdm.com](https://post.smzdm.com/p/a4peokqk/?send_by=3547620770&invite_code=zdm4ptnapkinv&zdm_ss=iOS_3547620770_&from=singlemessage)

MOREFINE 摩方 M6 安装 Ubuntu 22.04 LTS 保姆级教程
========================================

2022-05-23 13:30:42 49 点赞 368 收藏 49 评论

MOREFINE 摩方 M6，一款非常小巧机身的迷你电脑主机，搭载了 intel 赛扬 N5105 和奔腾 N6000 两种处理器，性能上已经比之前的赛扬以及奔腾有了长足的进步。相对与上一代的赛扬和奔腾的性能提升比较大，尤其是 GPU 的性能，使得 M6 在播放 4K 视频的时候流畅度有更大的进步。

[![](https://am.zdmimg.com/202205/23/628af9bc3ffa9629.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_2/)

M6 的采用的赛扬 N5105 和奔腾 N6000 处理器，能够流畅运行 Windows10 以及 Windows11，作为办公、播放 4K 高清视频等已经非常轻松，甚至用来做轻度的图形设计也能胜任。由于 GPU 的性能提升，在主流的中型游戏也能有不错的帧率，譬如 LOL 默认画质能到 60-80 帧、魔兽在中低画质也能获得 50 多帧，虽然看起来机器小，但跑起来还是会让人刮目相看。

[![](https://am.zdmimg.com/202205/23/628af9bc5a30f4417.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_3/)

麻雀虽小，五脏俱全，速度不慢，用来形容 MOREFINE 摩方 M6 再合适不过。不过，有不少用户，还是把 M6 用来当做生产力工具，安装 Linux 系统用来跑一些服务应用，比较常见的是 Ubuntu 系统。由于 Ubuntu 的稳定性和运行效率都有非常优秀的表现，以及免费和开源等优势，应用越来越广。下面，我们把 MOREFINE 摩方 M6 安装 Ubuntu 22.04 LTS 的步骤，逐一展示，过程尽量详细明了，希望更多的用户能够尝试 Ubuntu 系统。

[![](https://am.zdmimg.com/202205/23/628af9bc3c0472904.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_4/)

**准备工作**

需要用到的工具：

MOREFINE 摩方 M6 主机及[显示器](https://www.smzdm.com/fenlei/xianshiqi/)、键盘[鼠标](https://www.smzdm.com/fenlei/shubiao/)；

能上网能插 U 盘的 Windows 系统；

一个 8GB 以上容量的 U 盘。

**安装步骤：**  

一、下载 Ubuntu 和写入系统镜像

[下载 ubuntu.com 22.04 LTS](https://ubuntu.com/#download) 安装系统（[https://ubuntu.com/#download](https://ubuntu.com/#download)）

[![](https://am.zdmimg.com/202205/23/628af9bc6a8612252.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_5/)

[![](https://am.zdmimg.com/202205/23/628af9bc604da6598.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_6/)

[下载系统镜像制作安装工具 balenaEtcher-Portable（https://www.balena.io/etcher/）](https://www.balena.io/etcher)

[![](https://am.zdmimg.com/202205/23/628af9bc419101961.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_7/)

下载完毕后，插入 U 盘，打开刚下载好的 balenaEtcher

[![](https://am.zdmimg.com/202205/23/628af9be383ba2645.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_8/)

选中下载好的 Ubuntu 镜像：

[![](https://am.zdmimg.com/202205/23/628af9be731854947.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_9/)

选择作为安装盘的 U 盘，这一步将会抹掉 U 盘数据，谨慎操作

[![](https://am.zdmimg.com/202205/23/628af9be5fd798174.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_10/)

选择 U 盘。我们这里用的是 32GB 的 SanDisk U 盘，大[家安](https://pinpai.smzdm.com/551/)

[](https://pinpai.smzdm.com/551/)

装的是根据自己的 U 盘型号和容量来选择，别选错了。

[![](https://am.zdmimg.com/202205/23/628af9be68de02671.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_11/)

选择好之后，按 “Flash！”。

[![](https://am.zdmimg.com/202205/23/628af9be56a5f211.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_12/)

等待 U 盘制作

[![](https://am.zdmimg.com/202205/23/628af9be6d4f11318.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_13/)

[![](https://am.zdmimg.com/202205/23/628af9c04d24f8899.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_14/)

完成。完成后，系统会提示格式化 U 盘，不能选择格式化磁盘！！否则白做。

[![](https://am.zdmimg.com/202205/23/628af9c04e13b1790.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_15/)

**二、Ubuntu 22.04 LTS 安装**

插好 M6 的电源、键盘鼠标、显示器、制作系统用的 U 盘，如有连接[网线](https://www.smzdm.com/fenlei/wangxian/)，需要先拔出来，开机；

不停点按 F7，将会进入快速选择启动项[界面](https://pinpai.smzdm.com/288313/)

[](https://pinpai.smzdm.com/288313/)[关注](https://pinpai.smzdm.com/288313/) [](javascript:;)品牌 ![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)   粉丝：

*   商品百科
    
*   好价
    
*   社区文章
    

，选中 U 盘的 UEFI 引导，Enter 键进入安装

[![](https://am.zdmimg.com/202205/23/628af9c04f8c84502.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_16/)

等待加载完成，按 Enter 键进入安装程序

[![](https://am.zdmimg.com/202205/23/628af9c0913225015.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_17/)

[![](https://am.zdmimg.com/202205/23/628af9c0836553472.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_18/)

启动完毕，选择语言点击安装

[![](https://am.zdmimg.com/202205/23/628af9c0a1fce8075.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_19/)

[![](https://am.zdmimg.com/202205/23/628af9c224f835437.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_20/)

选择正常安装，安装第三方软件，继续

[![](https://am.zdmimg.com/202205/23/628af9c2505fa3039.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_21/)

选择清空磁盘安装

[![](https://am.zdmimg.com/202205/23/628af9c2568137232.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_22/)

[![](https://am.zdmimg.com/202205/23/628af9c2abb697176.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_23/)

6. 选择地区安装

[![](https://am.zdmimg.com/202205/23/628af9c2997796835.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_24/)

设置本地账户

[![](https://am.zdmimg.com/202205/23/628af9c2bc23c5216.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_25/)

等待安装

[![](https://am.zdmimg.com/202205/23/628af9c4145254984.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_26/)

[![](https://am.zdmimg.com/202205/23/628af9c465f297441.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_27/)

安装完毕，重启

[![](https://am.zdmimg.com/202205/23/628af9c4310719108.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_28/)

等 log 刷屏完，根据提示拔出 U 盘，按 Enter 键进入系统

[![](https://am.zdmimg.com/202205/23/628af9c49168a9076.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_29/)

[![](https://am.zdmimg.com/202205/23/628af9c4a2a222540.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_30/)

选择账号，输入密码，进入 Ubuntu 操作系统

[![](https://am.zdmimg.com/202205/23/628af9c4e4ca16468.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_31/)

[![](https://am.zdmimg.com/202205/23/628af9c615ac66695.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_32/)

设置权限

[![](https://am.zdmimg.com/202205/23/628af9c63b5cc9678.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_33/)

[![](https://am.zdmimg.com/202205/23/628af9c64ff752429.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_34/)

[![](https://am.zdmimg.com/202205/23/628af9c6a27d59024.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_35/)

安装完成

[![](https://am.zdmimg.com/202205/23/628af9c68acee2804.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_36/)

**三、关于驱动程序**

执行完上述的步骤后，就已经是安装 Ubuntu 系统了。由于 Ubuntu 系统已经自带了许多现有硬件的驱动程序，所以，MOREFINE 摩方 M6 在安装好 Ubuntu 22.04 LTS 系统后，可以直接正常连接上网使用了。如果有需要添加其他驱动，可以通过终端命令形式，单独安装，这里我们就不再详述。

在 MOREFINE 摩方 M6 安装了 Ubuntu 系统后，由于 Linux 核心的效率优势，在运行大部分程序时，比在 Windows10 或者 Windows11 系统下要更有效率。不过 Ubuntu 的界面和使用习惯，以及终端命令的输入等方式，需要有一定的时间适应和学习，才能用得更顺手。

[![](https://am.zdmimg.com/202205/23/628af9c6dffde1723.jpg_e1080.jpg)](https://post.smzdm.com/p/a4peokqk/pic_37/)

安装 Ubuntu 系统，相比 Windows 系统的安装要更简单，速度也快了许多。总结一下主要步骤：

第一步：下载 Ubuntu 系统镜像和安装工具 balenaEtcher；

第二部：用 balenaEtcher 写入镜像到 U 盘；

第三部：安装 Ubuntu 系统，这部分安装耗时 10 分钟左右。

有时间，我们再给大家来一篇安装 Ubuntu 和 Windows10 或者 Windows11 双系统的详细教程。

天猫精选 [![](http://img.alicdn.com/imgextra/i4/765053738/O1CN01xCgKc41dU3G2B7z55_!!765053738.jpg) ](https://go.smzdm.com/8acd12db28d61895/ca_bb_yc_163_a4peokqk_15681_301270_179_0) [MOREFINE 摩方 M6 N5105 迷你主机（N5105、8GB）](https://go.smzdm.com/8acd12db28d61895/ca_bb_yc_163_a4peokqk_15681_301270_179_0) 979 元 实时价格 9 小时前已更新 [去购买](https://go.smzdm.com/8acd12db28d61895/ca_bb_yc_163_a4peokqk_15681_301270_179_0) 

![](https://res.smzdm.com/pc/pc_shequ/dist/img/the-end.png)
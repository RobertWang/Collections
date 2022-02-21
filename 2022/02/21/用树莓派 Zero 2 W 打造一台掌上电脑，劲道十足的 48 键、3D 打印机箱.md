> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA4MDExMDEyMw==&mid=2247496417&idx=2&sn=583e94cfb755238a16b4bca7ec129491&chksm=9fab85a5a8dc0cb3a371dcfb3ee335ebb2047264c125b51c355029e6b4390d90e037e6236d9e&mpshare=1&scene=1&srcid=0220iAgHzvEqMlAxdxzRk7NB&sharer_sharetime=1645323632791&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> 一台可以放在手掌中的 PC。

如果现在有人提到掌上电脑，那几乎可以肯定他们所指的是智能手机。但是树莓派发烧友总有办法展示其复古的一面。无需多大的技术改进，加上一些陈旧的控制台部件和真正的机械键盘，你就可以拥有一台可以放在手掌中的 DIY PC。这就是本文将要介绍的 Penkesu 项目所实现的功能。

项目作者是 Penk Chen ，该项目介绍了如何通过 Raspberry Pi Zero 2 W（树莓派 Zero 2 W）打造一台复古风格的掌上电脑，它的分辨率为 400 x 1280 、触摸屏为 7.9 英寸。此外，它还包括其他电子部件：3.7 V Li-Po 电池和用于供电的 Adafruit PowerBoost 1000C。

成品如下图所示，看起来真的很复古。

这台树莓派打造的电脑使用起来也非常流畅，没有卡顿的感觉：

‍该项目上线短短三天就揽获了 600 + 星：

项目地址：https://github.com/penk/penkesu  

**这台掌上电脑是怎么造的？**

电脑机箱整体以显示器和键盘为中心进行设计，以实现（相对）紧凑的物理尺寸，机械键盘是正交的，有 48 个键，所有键在相同的行和列没有错位，就像一个网格，这与我们常用的键盘不同。 

![](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gW9wicib7EZNOsNO81mViciaAMjlwljaanVru6VHibxmgv9UpOlLRIHWPRP2D8W4JKK63nCeicBVf3ZeiabdA/640?wx_fmt=png)

*   显示：Waveshare 7.9 英寸电容式触摸屏；HDMI 带状电缆；
    
*   机箱：Game boy Advance SP 铰链，以允许 PC 折叠关闭，3D 打印部分 (STL 文件和 STEP 文件)；
    
*   电子器件：树莓派 Zero 2 W，3.7V 606090 (或相似规格) Li-Po 电池，Adafruit PowerBoost 1000C；
    
*   键盘：Kailh Low Profile Choc v1 Switches x 48、MBK Choc Low Profile Keycaps x 48、1N4148 Diode x 48、Arduino Pro Micro x 1、PCB x 1 。
    

![](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gW9wicib7EZNOsNO81mViciaAMjlcJumiaez4e3ic4aVyDpFKRwX1Wicrg00kDPsFPP0tR5TmtzbRIgJ8C4Lg/640?wx_fmt=png)

第二步在屏幕边框的 4 个角上添加热固螺纹插件 (M2x6)，同时在铰链盖上添加 2 个 M2x6。

第三步将带状电缆缠绕两次，然后通过铰链盖将其拉出。

![](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gW9wicib7EZNOsNO81mViciaAMjlWtWZXpjyUa7NyOaaaDyPmiclUhybnibHQFEPqKxbtRoaWCt1EPWSOiajQ/640?wx_fmt=png)

第四步接线：  

![](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gW9wicib7EZNOsNO81mViciaAMjllQK3zsBql01IzKrtg5YX8XzSQusLricWZJjibeufbx30hGWQshdGyUUQ/640?wx_fmt=png)

第五步将键盘的 micro USB 和显示器的 mini HDMI 端口连接到 Pi Zero 2 W；将 micro SD 卡插入 Pi Zero 2 W。

第六步用 M2x6 螺钉固定所有组件。

一台树莓派掌上电脑就这么完成啦。

_参考链接：https://arstechnica.com/gadgets/2022/02/diy-handheld-pc-uses-mechanical-keyboard-game-boy-pieces-raspberry-pi/_
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI4MjI1MTI0Mw==&mid=2247495304&idx=1&sn=ab61c35ad240c2f47aae1d7922cd1046&chksm=eb9e740cdce9fd1afd5e9588fc3288c1d570b8d3292d806529762a76b6341c707d0d0aba1831&scene=21#wechat_redirect)

来自机器之心

![](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWibaTB98qG1LnVibhtQiaUUet3wHZsg1jO7Oo48bYh09sxRbR4cxJSggDeZP8esVVZ5XZOiaXNOf29trA/640?wx_fmt=png)

_微软已经把从 Windows 3.0 以来万年不变的蓝屏死机画面在 Win 11 上改成了黑屏。_

对于大多数普通用户来说，Win 11 的正式版还要等待今年底或明年初才能上线，目前出现的预览版虽然是微软主动开放的，并不像之前的「泄露版」那样有版权问题，但还有不少 bug：现在连全局搜索都无法完成，回滚 Windows 10 也只能用镜像覆盖安装。

这些小困难并不能阻止一些人去做脑洞大开的事。既然 Win 11 计划要支持安卓应用了，那么安卓手机能不能装 Win 11 呢？最近就有国人团队进行尝试并获得了成功。

在 2015 年 Windows Phone 终结以后，微软就基本放弃了移动端操作系统的尝试。虽然我们可能永远无法看到微软真正重回手机市场，但已有开发者向我们展示了在安卓手机上运行 Win11 的方法。

这个项目名叫 Renegade Project，GitHub 链接：

https://github.com/edk2-porting/edk2-sdm845

虽然目前还处在早期阶段，但作者之一的 Xilin Wu 表示，Windows 11 ARM64 版可以安装在使用骁龙 845 处理器的手机上，而骁龙 855 的手机则部分支持 edk2-sm8150 端口。

在开发者的努力下，目前人们已成功在诸如一加 6T 和小米 8 这样的手机上启动了 Windows 11 系统。不过相应地也有问题，很多手机原本的功能就无法使用了，比如打电话。

![](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWibaTB98qG1LnVibhtQiaUUet3ibSe7BKousQs7YYuoIMia5SNn6HNJRtOBIfibdq9e3NlW7s2yaAuXg8YQ/640?wx_fmt=png)

_在一加 6T 上运行 Windows 11。_  

在 Windows 11 发布时，微软宣布该系统也支持 Arm 架构的平板和笔记本，不过只有高通骁龙处理器在官方支持之列。为了在安卓手机上运行桌面操作系统，开发人员创建了自己的工具 / 驱动程序，并为骁龙 845 启用了自定义 UEFI 环境。

![](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWibaTB98qG1LnVibhtQiaUUet30unibnuFOW7xvbSLhEJ3VoMPzL61aK7HDa35uBY79AN4iccaphyCrvOw/640?wx_fmt=png)

_在小米 8 上运行 Windows 11。_  

据开发者们介绍，这是因为高通的 xbl（UEFI 固件）和 abl（应用引导程序，用于加载 Linux 内核）是在零售设备上签名的，不能修改。因此，我们不能使用库引导加载程序来引导 Windows。

![](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWibaTB98qG1LnVibhtQiaUUet3r9nxLiaGslnmyVLwA8nO1fLiaRvdlnNXEzOoqX3RlWQYyiaAQSibnicRZcQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gWibaTB98qG1LnVibhtQiaUUet33CQbEcNt5d0YwyyNYyfySSOLFz4icicq7CxTQYjfFjdTdviay8VnH7n7w/640?wx_fmt=png)

看起来目前这些开发者破解的进展很顺利，手机上的 Win 11 可以完成一些基本的操作系统功能，包括蓝牙和 USB 输入。桌面尺寸很小但显示还算完整。但请记住让 Windows 11 上手机并不是微软的本意，想要真正用得上，设置手机通讯录、移动信号和电话等功能仍是挑战。

除了安卓手机，其他一些开发者最近也展示了在树莓派 4 和 Lumia 950 XL（最后一款 Windows Mobile 手机）上运行 Win 11 的结果，其中后者还支持打电话。

对于树莓派的开发者来说，和 IoT Core 版本系统功能缺失的体验不同，Arm 上的 Win 11 和 Win 10 在 Raspberry Pi 上提供了完整的桌面体验。新版本的 Windows 提供了比上代更好的性能，运行起来更流畅。

![](https://mmbiz.qpic.cn/mmbiz_jpg/KmXPKA19gWibaTB98qG1LnVibhtQiaUUet30hsnwMsq4S7gnzsiaGyyCH7ekTGAZdzh3xficibeBVZQVrrCOlHOPfc4g/640?wx_fmt=jpeg)

Windows 11 on ARM on Raspberry 的安装过程与 Windows 10 相同，需要具有 4GB 内存的 Raspberry Pi 4 和 Windows 11 ARM64 镜像。

这次 Win 11 的更新带来了更现代化的界面，丰富的软件生态支持，以及开放的应用商店，虽然它很少被人认为是最好的操作系统，但看来可以成为兼容性最好的那一个。

参考内容：

https://enter21st.com/devs-begin-bringing-windows-11-to-android-telephones-from-oneplus-xiaomi/

https://www.windowslatest.com/2021/06/30/windows-11-is-already-running-on-raspberry-pi-4/

公众号

**推荐阅读**

[装 X 神器！NuShell](http://mp.weixin.qq.com/s?__biz=MzI4MjI1MTI0Mw==&mid=2247495298&idx=1&sn=bfeb0c43ea3e9344d2d76daf4002db31&chksm=eb9e7406dce9fd105d361e7e85fde5f4810be155af54262d103771add54bba7fc471158a9e98&scene=21#wechat_redirect)

[红旗 Linux 操作系统 v11（献礼版）发布，个人永久免费，附镜像及安装教程](http://mp.weixin.qq.com/s?__biz=MzI4MjI1MTI0Mw==&mid=2247495294&idx=1&sn=154fe110421cd34176f89ec010b2cbf0&chksm=eb9e74fadce9fdece246e23ab6623832890859f4c75e1f92550d652a2f47660163e2d39e622e&scene=21#wechat_redirect)  

[VSCode 花式玩法（摸鱼）收藏一下 ！](http://mp.weixin.qq.com/s?__biz=MzI4MjI1MTI0Mw==&mid=2247495287&idx=1&sn=e3b258797b1b5e8833c638c8a9e43a22&chksm=eb9e74f3dce9fde55752979f9504136cd2f05d4914f3ae18bd2505b31b23bb834b1c2977d707&scene=21#wechat_redirect)

[程序员搞了一个离线地图 APP，还挺好用](http://mp.weixin.qq.com/s?__biz=MzI4MjI1MTI0Mw==&mid=2247495097&idx=1&sn=41f40cd51c0cf08012dae3b1e32bdfa5&chksm=eb9e773ddce9fe2bf4becf76d07b2b1ed4dee74e3f4f1b0641aa84842d30754fb1898b23463e&scene=21#wechat_redirect)
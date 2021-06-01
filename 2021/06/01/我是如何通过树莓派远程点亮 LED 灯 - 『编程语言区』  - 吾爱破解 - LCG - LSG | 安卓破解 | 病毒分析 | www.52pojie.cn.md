> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1451367&extra=page%3D1%26filter%3Dauthor%26orderby%3Ddateline) ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)alinwan0922 

_本文图片来自网络，侵删_

前言
--

本文聊的是我是如何实现远程开启 LED 灯

#### 呓语

不知不觉已经到了不上不下的年纪了，唯一不变的还没有找到所谓的**喜欢**，小时候长辈经常问我的问题 **你喜欢做什么？** 如今变成了我自己问自己**我到底喜欢做什么？** 20 多年的生存经验告诉我，坐以待毙可能永远也无法找到，所以我要尝试主动出击。

今天要分享我做了一个远程控制 LED 灯的小玩具

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/edd0b05a7f0d4569b1bfe60764f51de4~tplv-k3u1fbpfcp-zoom-1.image)

来吧，展示
-----

展示玩具

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/666f405a526b437c9e4ff372f8519909~tplv-k3u1fbpfcp-watermark.image)

Web 地址：[http://348r02z653.qicp.vip/](http://348r02z653.qicp.vip/)#/ （效果你们也看不到）

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d3af6a65dd5f471788ae2bbe92afd284~tplv-k3u1fbpfcp-watermark.image)

需求
--

如果我想远程控制家里的设备，我能想到的方案如下

1.  雇一个保姆每天到点在家里打开相应的设备
    
2.  家里装一套智能家居系统
    
3.  买一个树莓派，然后远程控制树莓派
    

认识我的人都知道我的经济情况，不能说特别有钱吧，起码也是**富可敌国**，所以我选择第一个方案。（那是不可能的不然也不会有这篇文章），所以我选择第三个方案，不要问我为啥不选 2，我觉得 2 这个数字我不喜欢不符合我的气质。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9ca1fd6622e44db09f3cd7d47c48e076~tplv-k3u1fbpfcp-zoom-1.image)

流程图
---

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dfd63c3f45fc4d89bc81e95f57324309~tplv-k3u1fbpfcp-zoom-1.image)

我用到的东西
------

#### 硬件

1.  树莓派 4B 及相关配件
    
    > 电源线（树莓派供电）、内存卡（提前烧录系统进去）、风扇（散热使用）彩虹线、面包板
    

> > 下载树莓派系统推荐国内镜像站

[国内镜像站](https://gitee.com/2lin/chinese-opensource-mirror-site) 、

[如何烧录系统](https://gitee.com/null_695_7527/raspberry_school_learning/tree/master/01.%E6%A0%91%E8%8E%93%E6%B4%BE%E5%BC%80%E6%9C%BA%E5%87%86%E5%A4%87)

2.  LED 3 色灯模块

#### 软件

1.  **Web** 使用 vue3 + ant-design-vue 等
    
2.  **API** 使用 .NET5
    

树莓派是什么？
-------

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f3aa2f445f4543bfb099b08c885ef49c~tplv-k3u1fbpfcp-zoom-1.image)

这个是树莓派，这篇文章聊的是另一个树莓派

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6dc8de8a23b84b98a8e26aea04942dc1~tplv-k3u1fbpfcp-zoom-1.image)

LED 3 色灯 长这个样子
--------------

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e2cacacd8a7c4ea4aadf448b39b95ac3~tplv-k3u1fbpfcp-zoom-1.image)

连接线路
----

我的接入方式为 GPIO 引脚 8（红灯）、10（绿灯）、12（蓝灯）、14（地线）

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0826469fe6924572896265154ccf2562~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a0e2f4c1a0741ceb6d5e33b63db1346~tplv-k3u1fbpfcp-zoom-1.image)

启动服务
----

#### API（不可以在 docker 中运行（目前没找到方案），因为要控制树莓派针脚）

直接独立部署，树莓派上不需要安装运行环境，环境都在程序包里面

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ec354573093343458e2819541562c2bb~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4a10799ff72e4392af435ed37183be91~tplv-k3u1fbpfcp-zoom-1.image)

#### Web （可以在 docker 中运行）

配置好 nginx 启动就可以

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/85821d0d62b4474a8b73b16676557ba0~tplv-k3u1fbpfcp-zoom-1.image)

代码
--

代码我都放在了 GitHub

[Web](https://github.com/lierlin/Raspberry_Pi_Web)

[API](https://github.com/lierlin/Raspberry_Pi_Api)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1ae20007af12473f8bfb2d6a54fc7c24~tplv-k3u1fbpfcp-zoom-1.image)

后记
--

传感设备在持续接入，远程红外控制空调、通过 Home Assistant 接入苹果的 HomeKit 等等

，还有一些零零散散的有趣思路

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ab8adbe0f48042baab1dbf189e082a28~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/faac56250d3f48bb8c359937e107184b~tplv-k3u1fbpfcp-zoom-1.image)

参考、来源
-----

[智能家居 Home Assistant 搭建从入门到入坑](https://files.mdnice.com/user/838/57f76b61-18d5-4c9b-81c3-33347fe5c9f2.png)

[画图工具](https://board.oktangle.com/)

[表情包来源](https://fabiaoqing.com/)

[视频压缩](https://fabiaoqing.com/)

[视频转 GIF](https://www.aconvert.com/cn/video/mp4-to-gif/)

[/md]![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)alinwan0922 

> [轻描淡写 9714 发表于 2021-6-1 15:56](https://www.52pojie.cn/forum.php?mod=redirect&goto=findpost&pid=38768622&ptid=1451367)  
> 树莓派有了，其他的家居哪里领？

哈哈哈，小米智能设备 也可对接、或者买 几毛钱的传感器玩都可以![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)翡冷翠的城 

> [alinwan0922 发表于 2021-6-1 17:38](https://www.52pojie.cn/forum.php?mod=redirect&goto=findpost&pid=38770329&ptid=1451367)  
> 可以一起玩一下项目呀

有啥能玩的，感觉我哪儿都搞不懂，就刷了一个 op 一个 win10 玩了一下 ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) jimoguying2020 卧槽，卧槽槽槽槽槽槽槽槽槽槽槽![](https://avatar.52pojie.cn/data/avatar/001/32/19/53_avatar_middle.jpg)星辰公子  楼主讲的和细致  
可惜我不用 ![](https://avatar.52pojie.cn/data/avatar/000/65/61/79_avatar_middle.jpg) pojie1nd2nd3nd 不善言辞，只能卧槽了![](https://static.52pojie.cn/static/image/smiley/default/42.gif) ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) Heya 哇哦，是大佬![](https://static.52pojie.cn/static/image/smiley/default/biggrin.gif)![](https://avatar.52pojie.cn/data/avatar/000/54/07/66_avatar_middle.jpg)轻描淡写 9714  树莓派有了，其他的家居哪里领？![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)zhujunliang955  请问用什么画的与原理图呢。![](https://static.52pojie.cn/static/image/smiley/default/40.gif)![](https://avatar.52pojie.cn/data/avatar/001/17/14/31_avatar_middle.jpg)andyle  很有趣的样子，但是因为懒不想动 ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) alinwan0922 

> [zhujunliang955 发表于 2021-6-1 16:00](https://www.52pojie.cn/forum.php?mod=redirect&goto=findpost&pid=38768693&ptid=1451367)  
> 请问用什么画的与原理图呢。

Fritzing, 去官网 http://fritzing.com 下载 ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) alinwan0922 

> [zhujunliang955 发表于 2021-6-1 16:00](https://www.52pojie.cn/forum.php?mod=redirect&goto=findpost&pid=38768693&ptid=1451367)  
> 请问用什么画的与原理图呢。

Fritzing, 去官网 http://fritzing.com 下载
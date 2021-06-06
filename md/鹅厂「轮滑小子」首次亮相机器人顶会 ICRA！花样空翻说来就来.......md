> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIwOTc2MTUyMg==&mid=2247520278&idx=4&sn=eefac5ca704ebe53be8f129276c19d02&chksm=976c358ba01bbc9d587a51c58128e364b74db891b6eb1cc418187cd38e1bd3ed0044c7111502&mpshare=1&scene=1&srcid=0606FsxjDPp1ALhizrYsVsFN&sharer_sharetime=1622991512962&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

![](https://mmbiz.qpic.cn/mmbiz_gif/cNFA8C0uVPvMuNa3wmD4jXPT4e1ko4YEQdgYl9lTyKtBRmqYMzl203Yh3Y5lCm5pticLfwicaLsB7hvA4eUsDX4Q/640?wx_fmt=gif)

转自 | AI 科技评论  

编辑 | 苦寒来

**机器人现在都能玩出什么花样了？**

直接跳上 40 厘米的台阶！（竖直起跳高度最高可达 60 厘米）：

![](https://mmbiz.qpic.cn/mmbiz_gif/KmXPKA19gWib8wzsbkqJxibnwD7xFgbHyDgSpGMhJ9h7KKgvuRicVImggopE53iactA6lNOnCJCNpuHicUjQ7o37wiaw/640?wx_fmt=gif)

360 度空翻挑战说来就来！

![](https://mmbiz.qpic.cn/mmbiz_gif/KmXPKA19gWib8wzsbkqJxibnwD7xFgbHyDTy3iaDWSzF3UIqicejWnBZNU20AZLwp6P8VEosPy7Yia3RWkMoCiaPUrZQ/640?wx_fmt=gif)

像不倒翁一样能灵活抗住各种「突发」状况，抗干扰能力极强！

![](https://mmbiz.qpic.cn/mmbiz_gif/KmXPKA19gWib8wzsbkqJxibnwD7xFgbHyDXGGLqgFzObGz8fVVRT5EYJnsqNsgqPH2p0UtkLg4JXPGmGJ0Bcys2A/640?wx_fmt=gif)

没错，这正是今日（6 月 3 日）正式亮相的腾讯轮腿式机器人 Ollie（奥利）！

它像一个灵活的 **“轮滑小子”**，能完成上面跳跃、360 度空翻等高难度动作。

简直让人忍不住大喊一声奥利给有没有

轮腿式机器人（wheel-legged robot）是近年来机器人研究的前沿领域。Ollie 兼具轮式结构和腿部能力，轮式结构移动快、效率高；腿部能力让 Ollie 适应不平地面、完成跳跃台阶等动作，达到了行业领先水平。

这是腾讯 Robotics X 实验室新型机器人。在自平衡自行车、机器狗 Jamoca 和 Max 之后，Ollie 积累了实验室的移动控制技术，并在运动规划、平衡与稳定性上重点突破，成为实验室又一大创新成果。

在日前举办的 ICRA 2021，**腾讯 AI Lab 及 Robotics X 实验室主任张正友博士受邀作大会报告，**介绍了 Robotics X 实验室在机器人移动研究领域的布局与进展，并分享了 Ollie 的技术细节。

### 

**1**

  

**Ollie 的机械设计大有玄机**

它的单腿采用并联机构，与身体形成五连杆结构，使整体具有结构简单、动态性能高、爆发力强的特点；“尾巴”的独特设计一方面为 Ollie 提供额外角动量，助其完成更高动态运动，如空翻。同时 “尾巴” 可充当第三条腿，增加稳定性，为搭载机械臂完成更多任务提供可能。

出色的运动能力，高动态高难度的空翻动作背后，源于腾讯 Robotics X 实验室的最新研究进展：非线性控制技术、全身动力学控制和轨迹规划。

非线性控制技术让机器人具备良好的平衡能力，此前实验室研发的自平衡自行车已应用同类技术，在静止及行进状态下均保持平衡不倒。

针对轮腿式机器人的形态和特点，实验室研发团队适应性地应用非线性控制方法，控制器不再受限于模型的可线性化区间内，使机器人 Ollie 在大角度倾斜时也具有良好的平衡能力和鲁棒性（Robust 音译，指在异常和危险情况下系统生存的能力）。

在双轮模式下，机器人与地面只有两个接触点，对平衡能力要求更高。在变换身高过不平整地面、甚至单腿过障碍时，Ollie 都能完美保持平衡。

![](https://mmbiz.qpic.cn/mmbiz_gif/cNFA8C0uVPvMuNa3wmD4jXPT4e1ko4YEaOmFVuywXU9NQXXNaoGicZqe0cf1LStZE8ibwTQdy2IQFqecR1a0kGgA/640?wx_fmt=gif)

还能轻松过个波浪坡：  

![](https://mmbiz.qpic.cn/mmbiz_gif/cNFA8C0uVPvMuNa3wmD4jXPT4e1ko4YEQdgYl9lTyKtBRmqYMzl203Yh3Y5lCm5pticLfwicaLsB7hvA4eUsDX4Q/640?wx_fmt=gif)

全身动力学控制像给 Ollie 装上了发达的 “小脑”，其采用最优化方法求得各关节力矩来实现全身姿态调整，不仅能让它实现更有挑战的运动，并在面对突如其来的巨大冲击如在空翻落地和遇到碰撞时，Ollie 能 “以柔克刚”，顺利抵抗外界干扰，保持平衡。

除了平衡能力之外，Ollie 还拥有轨迹规划能力。在完成这些动作时，Ollie 要动用自己的 “大脑” 提前 “想好” 运动轨迹，即如何应用自身的形态和结构特点，最大程度地发挥关节电机性能来实现目标运动。

Ollie 以全身动力学模型为基础，将整个跳跃或空翻过程分解为起跳、飞行、落地三个阶段，通过优化手段得到完成整个运动的关节电机位置、速度和关节力矩的参考值序列。

### 

**2**

  

**Robotics X 移动能力再突破 论文入选机器人顶会 ICRA**  

腾讯 Robotics X 实验室主攻移动、灵巧操作和智能体三大机器人核心通用技术的研究与应用。其中，移动能力被认为是机器人最核心、也是最基本的能力之一。  

实验室移动技术框架包含机械设计、感知、运动规划及控制，以及融合这三者的整机系统设计与搭建等四大模块，他们分别可理解为机器人的躯干、眼睛、大脑，以及各 “器官” 协调的能力。

![](https://mmbiz.qpic.cn/mmbiz_png/cNFA8C0uVPvMuNa3wmD4jXPT4e1ko4YEibaIPCUcE2RLQMyHV2B9Xs2T3hGJKcRDrjCRvqPWAV4kOicl1ibl23Tzg/640?wx_fmt=png)

Ollie 超强的 “轮滑”、“空翻” 能力就来源于这些器官协调后的结果。在机械设计、整机系统与控制软件上集成迭代了实验室技术积累，并重点在运动规划与控制上突破创新。新增的全身动力学控制与整机参数辨识提升了机器人运动的精准度、灵活度以及柔顺性，拓展了实验室的移动技术布局。

Ollie 相关研究论文已被 ICRA 2021 收录，论文介绍了轮腿式机器人平衡控制器的设计思路与实验结果。

![](https://mmbiz.qpic.cn/mmbiz_png/cNFA8C0uVPvMuNa3wmD4jXPT4e1ko4YEkLnTHBTl843P7ALSHJD2dfufxIpurOk3wqmfng4aUQdhdU7xbwSdTw/640?wx_fmt=png)

**ICRA 全称国际机器人与自动化会议（IEEE International Conference on Robotics and Automation），是机器人领域最有影响力的国际学术会议之一。**

![](https://mmbiz.qpic.cn/mmbiz_jpg/cNFA8C0uVPvMuNa3wmD4jXPT4e1ko4YEJzbsAKPOSeltBEQibl42vJicDjUhiaEqic2rianHfmccBIyTyJiciaicv7ko4A/640?wx_fmt=jpeg)

目前 Ollie 还处于研发阶段，实验室将基于轮腿式机器人平台的机动性特点，拓展平台上感知、负载等各功能模块搭建，让机器人具备更成熟、更丰富的能力。

比如，本次腾讯团队尝试探索了 Ollie 的负载能力，两轮、三轮可以随意切换，再加上一个机械臂，让它能够平稳地端起一杯咖啡，通过轮式移动递给远处的客人。

![](https://mmbiz.qpic.cn/mmbiz_gif/KmXPKA19gWib8wzsbkqJxibnwD7xFgbHyDXkickeuDpYkw09P77V9wFp2etDfNVG15uIK7ZXicqKJXYgdgYsM54fJw/640?wx_fmt=gif)

腾讯方面表示，未来，腾讯 Robotics X 实验室还将在机器人行业做全方位、多领域的探索，向人机共存、共创、共赢的未来不断迈进。

* * *

  

**推荐阅读**

（点击标题可跳转阅读）

[干货 | 公众号历史文章精选](https://mp.weixin.qq.com/s?__biz=MzIwOTc2MTUyMg==&mid=2247489073&idx=2&sn=01856c4f799c8bb2c72dc3e04ed6fa03&chksm=976fb3aca0183aba785ab0ef547c646a33a55c3e4c1c3689a246f902b47264b4dbff3a68bcc8&token=98277343&lang=zh_CN&scene=21#wechat_redirect)

[我的深度学习入门路线](https://mp.weixin.qq.com/s?__biz=MzIwOTc2MTUyMg==&mid=2247485239&idx=1&sn=c3e1eef018f1a90f221cca1b74fc361d&chksm=976fa2aaa0182bbc383a48a33623631e289483c4a0df56cddac52d22127babb66bdec65a6522&token=919391882&lang=zh_CN&scene=21#wechat_redirect)

[我的机器学习入门路线图](https://mp.weixin.qq.com/s?__biz=MzIwOTc2MTUyMg==&mid=2247484874&idx=1&sn=e4a54e67d7da788c5b75acfa96763e1c&chksm=976fa057a0182941e8dda24a469338100d81de848a6aad467f77c6950202ab02562739aa1c8c&scene=21&token=98277343&lang=zh_CN#wechat_redirect)
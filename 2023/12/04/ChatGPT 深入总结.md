> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.warmplace.cn](https://blog.warmplace.cn/post/chatgpt#ChatGPT%E5%8F%91%E5%B1%95%E5%8E%86%E7%A8%8B)

> 前端知识系统梳理，技术分享

前两天在`OpenAI`开发者大会上看到 ChatGPT 的能力进一步提升，API 接口费用降低，又一次震撼了我。从今年 2 月 ChatGPT 大火，到现在 11 月份已有 9 个月了，结合当下行情，不然不让人焦虑，如果越来越多的岗位都被 AI 抢走了，那么平民百姓如何体面的生活？技术的车轮不是滚滚向前，而是还有加速向前的趋势，没有人能阻止它前进，或许我们唯一能做的是学习它，使用它，先跑赢同伴吧，这样说或许很残忍，人类的进化史不正是这个过程吗？

今天打算系统梳理一下 ChatGPT 相关知识，围绕以下问题深入研究

1.  ChatGPT 的注册、GPT4 开通。
2.  ChatGPT 的使用技巧。
3.  浅谈 ChatGPT 原理。

*   2023 年 1 月份用户量超过 1 个亿
*   2 月 10 日推出会员计划 20 美刀一个月，能够更快响应和体验新功能
*   3 月 15 日推出 GPT4。Plus 会员才能用，3 小时只能用 25 次，由于只能使用海外信用卡，国内用户大都用不了
*   3 月 23 支持小范围的插件 Alpha 功能
*   4 月份 活跃用户达 1.73 亿
*   5 月 13 日 向所有的 plus 会员提供联网功能 + 插件功能，直接补足了被诟病的数学计算能力差及不能联网更新问题
*   5 月 19 日上线 IOS APP
*   7 月 14 日联网模式下先
*   7 月 15 日推出 Code Interpreter 模式
*   7 月 20 日 3 小时只能 25 次，增加到 50 次
*   7 月 21 日 上新了个人定制指令 Custom Instructions
*   7 月 25 日 androidApp 上线
*   8 月 4 日 小功能更新， 如文件上传、快捷键
*   11 月 7 日 GPT-4 Turbo 发布会， 支持 128K 上下文、训练数据更新至 2023 年 4 月、支持图像输入总结图像内容、GPT 商店、API 降价

目前的整体数据是

*   每月访问量 15 亿次
*   活跃用户 2 亿多
*   运营成本 2000 多万美金一个月
*   年底预计营收 2 个亿美金
*   明年预计营收 10 个亿美金
*   微软追加投资 100 个亿，很火很烧钱不差钱

[chatGPT 官网](https://chat.openai.com/)

**chatGPT 的注册**

关于 chatGPT 的注册使用，我在今年 2 月份的时候参考这篇文章 [国内用户如何使用 chatGPT](https://www.163.com/dy/article/HSTKB2GE05561MLH.html) 注册了账号，现在链接失效了。大致过程是，注册需要一个国外手机号，不支持国内用户， 没有国外手机号怎么办？ 这里借助一个 [接码平台](https://sms-activate.org/) 弄个虚拟的手机号，短信会发送到这个平台。 访问这个网站需要科学上网，梯子自备。选择 chatGPT 服务，当时选择的哪个国家或地区的手机号也不记得了，往这个站点充了几块还是几十块钱也不记得了，充值很简单，支付宝扫码支付即可，这个号码是按小时租的，下次要想用，好像也能租到，价格会高一点，但是注册 chatGPT 时我绑定了 goole 账号，虽然没有手机号了，我每次都是用 google 账号登陆。

想要开通 GPT4 需要美国的信用卡，到国外办卡肯定不现实， 就需要使用虚拟信用卡，充值信用卡又需要加密货币的稳定币，整个过程还是比较麻烦的，该方式主要有两种方案

1.  `Dupay` + `Okx`
2.  `OneKey` + `币安`

我使用的是方案一， `Dupay`的前身是`Depay`，它和`OneKey`的作用就是创建虚拟信用卡。

第一步：肯定是到 [Dupay 官网](https://www.dupay.one/)注册账号，你可以用[我这个邀请码](https://dupay.one/web-app/register-h5?invitCode=Z7TkDo&lang=zh-cn)注册，具体过程略

第二步：申请 VISA 类型的卡（Dupay 可以申请万事达和 VISA 两种类型的卡， 由于 VISA 标明了支持 chatGPT，不选万事达是怕万一不支持 chatGPT 就完蛋了）  
这里有个 KYC 认证，可理解为国内的实名认证，有三种类型的卡可选择

![](https://blog.warmplace.cn/static/img/d5d55b2434164c2ae38ec9bb5c00d071.image.webp) ![](https://blog.warmplace.cn/static/img/19797e4c260ebe41a214de201a361e3c.image.webp) ![](https://blog.warmplace.cn/static/img/a20851c3384ea76131c549aeaec99568.image.webp)

一般建议选高级卡，此外每个月要交`0.5$`月费，办卡手续费确实高！KYC 认证完了要缴费，没有美刀，只能暂停一下，，，

第三步：注册欧易账号，购买`USDT`转入`Dupay`账户

这里解释下[欧易](https://www.okx.com/cn)是区块链的交易所，它有很多个功能（购买各种虚拟币、虚拟币金融衍生品）这里我们只用到它购买`USDT`。`USDT`是虚拟稳定币，1:1 对标美元。[具体下载注册过程可以参考这个视频](https://www.youtube.com/watch?v=_yT8hmDaGY0)。另外提示 3 点

*   建议通过网页注册 (手机上 ios 需要北美 appid 才能下载， android 在你使用支付宝或微信的时候提示风险)
*   关键信息账号密码、助记词保存好
*   这里有个[邀请码`59255404`](https://www.ouxyi.clinic/join/59255404)，据说有新手福利

第四步：欧易提现到`Dupay`账户 具体参考[这个视频](https://www.youtube.com/watch?v=yA6jVNNwIyk)吧。

第五步：上一步转账大约 3-5 分钟到账，然后将 USDT 兑换成美刀然后继续开卡

第六部： 开发完成取 chatGPT 官网升级会员。注意 Dupay 的 VISA 卡的 cvv 安全码对应 chatGPT 官网上填写银行卡信息中的 CVC。

该方案费率很高

*   交易所 快捷买币 USDT -4%
*   交易所 USDT 提币到 虚拟卡 APP -3.82USDT
*   虚拟卡 APP USDT 转换为 USD -1%
*   虚拟卡 APP USD 提现到虚拟卡 -1.4%

如果你是 IOS 用户，那么恭喜你，注册 ChatGPT 账号不要花钱（在 app 上注册不需要手机号，但是使用 API Key 时必须绑定手机号）。可以在`App Store`下载 chatGPT app 并注册。另外可以通过礼品卡的方式购买 PLUS 会员，费率损失 < 1%

在`App Store`下载 ChatGPT 需要准备一个美国 APPID，可以参考[油管这个视频](https://www.youtube.com/watch?v=9dp48HTaZns) 注册美国区 APPID，设置付款方式无，支付宝购买 AppStore 礼品卡，充值 AppStore，然后进入 ChatGPT 开通，最后记得前往系统设置里取消订阅。

注册 APPID 需要一个干净的国外邮箱，推荐用`account.proton.me`这个网站注册，因为不需要手机号。

苹果的设计有个 Bug（或者说叫缺陷吧，无解），支付与 APPID 关联， 举个例子：

比如某个 APP1 安装需要支付 USD，你使用 APPID 为 A 的账号在 APPStore 下载安装了 APP1，你的朋友也想下载 APP1，那么可以让你朋友的手机登录你的账号下载 APPI，然后再切回他的账号，这是他不用支付费用就可以使用 APPI。

另外我觉得 android 用户也不用走虚拟币的方式充值 GPT 会员了，买个礼品卡，借朋友的 ios 手机用一下就行了。

以下是我的一些测试用例

*   一个蜗牛掉到一个井里，井深 5 米，它每天能爬 3 米，往下掉 2 米，请问它几天能从井里爬出来。 4 能答对，3.5 不行
*   websocket 为什么没有跨域问题 回答的结果不是我想要的
*   TS 类型体操方面问题 印象中，3.5 回答的不好有些乱扯
*   我问了一些电视剧人物相关的问题，感觉回答的不行，不知道是不是与训练数据有关
    *   ![](https://blog.warmplace.cn/static/img/1cb737028e26b540a1e0831ab5f56814.image.webp)
*   GPT4 的推导逻辑确实比较强

能画一些图片，还是挺美的，但是让他修改，不太尽如人意，偶尔也会出错，比如他给我生成了一副夏天海滩的照片，尽然有无头人。

后来按照网上 UP 主的指示，如果想要人物不变，可以让它返回 seed 值，我生成了几张图, 挺美的，给大伙看看。

Q: 给我生成一副画，一个中国乡村姑娘在田园中散步, 并返回 seed 值

![](https://blog.warmplace.cn/static/img/ba78a8945e215179818404f2c6b95d70.11.webp)

Q: 这位姑娘手里拿着一朵花， seed 值：2011363862

![](https://blog.warmplace.cn/static/img/e075893af2cf7ea5be93051a914ae1b6.2.webp)

Q: 她微笑着向我招手 seed 值：4056027149

![](https://blog.warmplace.cn/static/img/ace6eefb085e809b50645f09fbaae55b.3.webp)

Q: 我想看一下它的背影， 请给我一张长图， seed 值：4056027149

Q: 这个脖子太长了，能短一点吗， 另外图片给我宽幅， seed 值：4056027149

![](https://blog.warmplace.cn/static/img/1817be94e0b4a266e27ba851ba462e31.4.webp)

绝了，怪不得日本学艺术的小姑娘要自杀，哎！

我再让它生成走路的照片，她不会制作 GIF 图，经过我几轮调教还是有些问题

![](https://blog.warmplace.cn/static/img/7826b4b49dea11ba0083666420c50efa.5.webp)

也许模型算法再进化一些，训练数据再大一点，它真的能创作出能够直接用于游戏、视频创作的素材了，希望这一天晚一点到来吧！

过了两天，我把前面的三张图上传上去，让它再给我画几幅画，，

![](https://blog.warmplace.cn/static/img/aefd0f6fb006d116b7fed8d9495702d5.6.webp)

![](https://blog.warmplace.cn/static/img/95b0888c9922638d1e53b9ca5462d2c2.7.webp)

脸部还原的还行，但是我又测出来一些问题

1.  它好像对阳光的角度不太理解
2.  它不会微调或修改图片中的错误，新生成的图人物和背景改动都大
3.  能识别和对比 png 图片，但 webp 格式的图片不能识别和对比

前几天的 Turbe 发布会，奥特曼说 ChatGPT 支持定制个人专属聊天机器人了。

![](https://blog.warmplace.cn/static/img/fc93de539530eb58d12b64a4ce52b42a.image.webp)

![](https://blog.warmplace.cn/static/img/3cd539f5e5535bcd1f8ef44773b1474b.image.webp)

推荐使用 Configure 的形式创建机器人， 填一些内容就可以提问了。

![](https://blog.warmplace.cn/static/img/9b1e54f0113e13cbbb25a8470d265dbb.image.webp)

我们在发散想一下，如果关于这个人的资料很多，把它们上传到 AI，这部就克隆出一个人型机器人了吗？

*   Github 有个大佬写了一个项目`Pandora`现在 github 上不到了，不过参考[这篇文章](https://www.freedidi.com/9544.html)，通过 docker 的方式安装，安装后进入 docker 容器可以看到源码，用 python 写的，通过 Cloudflare 跨过长城，封装了 auth 登录，实现国内访问 ChatGPT。不过这版 UI 有点老，dockerhub 上看到作者推出了 [pandora-next](https://github.com/pandora-next/deploy)。这个镜像是最新的 UI。
*   油管和 github 很多好东西， 关于 AI 方面，如 AI 绘图、[制作有意境的图片](https://www.freedidi.com/8474.html)、 AI 去码去水印、AI 换脸等，自己搭建 AI 应用，重点是开源免费，还有就是依赖你显卡性能。
*   AI 绘图还有一些软件 `Midjourney`、`Canvas AI`
*   VScode 有一些 AI 相关的插件，TODO

尽管 ChatGPT 很强大，但还是存在回答不对或不准确的情况，搜索引擎是一个补充方案，很多场景也离不开搜索引擎。这里记录一下我在油管看到的一些搜索技巧。 [油管视频](https://www.youtube.com/watch?v=tiN6T1LewmQ)

*   `""`    限定关键字
*   `intile`  限定标题
*   `allintitle` 限定标题包含多个关键词
*   `intext`  限定内容关键字
*   `inurl`   限定网址关键字
*   `imagesize` 限定图片尺寸
*   `filetype`  限定文件格式

*   chat: 聊天
*   G: `Generative` 生成式
*   P: `Pre-trained` 预训练
*   T: `Transformer` 神经网络

简单说 chatGPT 真正做的事情是文字接龙的大语言模型。

文字接龙以 token 为单位，根据前面的 token 预测下一个 token 出现个概率分布，根据摇骰子给出答案继续计算下一个 token。

一般平均一个单词 0.75token，一个汉字可能 1～3 个 token，token 根模型有关，具体可以在[这个网站](https://platform.openai.com/tokenizer)查

给两个案例证明它是玩文字接龙的游戏

案例 1： ![](https://blog.warmplace.cn/static/img/ea90d0c32fc5b88d5d587ba722f096b6.image.webp)

其实并没有上海亚运会，是杭州亚运会，下面的网址也是 AI 臆想的。

案例 2： 让它写一个 1000 字某主题的作文，它会在半句中中断。

那么它如何记得你过去说过的话呢？ 其实很简单，每次提问就会把前面的话带过去，比如 chatGPT3.5 支持 4K token，如果超过 4K 的内容就采用 FIFO 算法，所以如果聊天超过一定字数它会忘记你之前说的话。

如何计算下一个 token 的分布几率呢？先来看一张图 ![](https://blog.warmplace.cn/static/img/672d57a564ea84adae8f957a0d3e8428.image.webp) 这个语言模型的背后是类神经网络，类神经网络你可以理解为从一堆函式中选出的一部分函式， 每个函式输入是不完整的句子，输出是下一个 token 的概率分布，每个函式有上亿个参数，函式里面是线性代数矩阵运算。

按照这个模型，如果训练的数据增加，回答的准确了也确实会增加，但正确率无法超过 55%，而人类有 90% 以上的正确率。

**督导式学习起到画龙点睛的作用**

如果人为的给一些数据打上标注，喂食 AI，即使最小的模型也比最大的无训练的机器学习模型要更准确。如下图。 ![](https://blog.warmplace.cn/static/img/59e71595ca8bd36ecd1034e500955d4b.image.webp)

预训练： 在机器自我学习前的人工督导式学习叫做**预训练**， 预训练的模型叫做基石模型， 在此额基础上的继续学习叫做**微调**

此外研究发现，在多重语言上做预训练后，只教某一语言的某一任务，机器也能自动学会其他语言的同类任务，如下图 ![](https://blog.warmplace.cn/static/img/f6c545d6173c2d3cc5b7fa797c36c42b.image.webp)

此外 chatGPT 还会增强式学习，你每次使用 chatGPT 时，对回答点赞或者点 bad 赞，都是对 ChatGPT 的增强式学习。

GPT3 经过督导式学习和增强式学习后的模型就叫 GPT3.5

虽然我对 AI 领域只是了解一些皮毛，对 chatGPT 的使用技巧也有待提升，但总结必须有，这样才能熟练运用。

ChatGPT3.5 功能总结

*   仅支持纯文本输入，支持 4K 上下文
*   响应速度比较快，支持翻译，自然语言与机器语言转换，内容相对搜索引擎靠谱。
*   但对于有技术深度的问题，或强逻辑、脑筋急转弯问题回答的不好

ChatGPT4 功能总结

*   支持文本输入，文件输入，支持 128K 上下文，支持联网
*   输出更准确，逻辑能力也有提升，但响应速度较慢
*   提供了插件系统，通过插件弥补了数学逻辑方面的不足等问题，通过插件还可以语音输入语音输出

优缺点和应用场景

*   国内用户每月主要支付 > 140RMB，每日 5 元的费用，对平民百姓来说还是比较贵的
*   chatGPT 冲击会很大，教育、培训机构、编程、创作等，介于目前 chatGPT 的逻辑推理、作图等方面有待提升，chatGPT 提供的还只是半成品，属于半自动驾驶，目前阶段需要快速掌握 chatGPT 的使用技巧

一些使用技巧

1.  把需求讲清楚
2.  提供范例， 比如让它写晶晶体（夹杂英文）的文章， 要先教会他晶晶体是什么。
3.  鼓励 chatGPT 想一想
4.  使用咒语
5.  拆解任务
6.  自主进行规划
7.  让 GPT 反思
8.  JoHari 沟通视窗，检测，反向提问

关于新技术最后在引用油管 UP 主的几句话，个人感觉很受用

> 不管什么新技术出来，最先嗅到商机的总是那些卖课的机构。 追热点很正常也很合理，  
> 不过呢，有的是真的在给你介绍前沿科技 介绍背后的技术原理，给你介绍技术的应用场景和局限，帮你拓展认知。  
> 有的却是在跟你忽悠，你不学 GPT 你就会被 AI 淘汰，现在直播价 199 最后 5 分钟下播就是 899，而且每天循环播放，这些呢是哎，不知道怎么说，当然很多人就是会被割韭菜，不是因为他没有判断能力，只是因为他身在别人的私域，说实话绝大部分的课程内容，还不如油管上的免费分享

也许有人会说 ChatGPT 只是一个学习了大量知识，经过人类老师调教，精通文字接龙游戏的机器，并非真的有逻辑推理能力。个人认为是不是真懂这个问题没那么重要了，这就好比人类发明了钻木取火不知道氧化反应，爱迪生发明了电，不知道电子是什么，未来世界如何很难想象。洪水猛兽既然放出来了，只能接受它、学习它、使用它、拥抱它、与它共存！
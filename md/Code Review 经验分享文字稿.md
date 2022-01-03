> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/0JJP1vMPgZmEOFSIG3Qsrw)

  

团队内部的关于 Code Review 的一些经验分享，看起来比较虚，但是也是用心准备。可能各个公司各个平台不一样，不过整体思路逻辑都可以借鉴参考，兴许能有所收益启发。以下主要为文字稿，看起来如有有些奇怪的地方稍微能理解即可。图文较多，阅读时长约 10 min 。  

  

简单的自我介绍下：鳗鱼，前端开发一枚，毕业后就一直在百度，现在 base 深圳，想要了解更多的可以关注或者看看之前的文章。有个免责声明，本次分享内容仅为基于当前经验的个人感想，如果后续有什么升级的想法，后面再说。有什么问题，也可以随时交流。  

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVsicFNexLexmy3of6oDS4iaYk5Oq5Q7VVAGPkSxAW2A5Lt3qNQDgd5GJA/640?wx_fmt=png)

本次分享会包含三个部分：什么是 CR、曾经有过的尝试、遇到的困难 & 解决思路等（厂内员工还有个跟平台相关的实用小技巧可，不在此处列举）。  

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVBcfMyJJJawibMrun5sRykNzbDTsKZcbE3Y8naspiaUbekSZicLEoTkCTw/640?wx_fmt=png)

**PART 1 · 什么是 CR**  

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVBcfMyJJJawibMrun5sRykNzbDTsKZcbE3Y8naspiaUbekSZicLEoTkCTw/640?wx_fmt=png)

这里是来自维基百科的官方解释，CR，全称 Code Review，中文名 - 代码审查，其目的是在找出及修正在软件开发初期未发现的错误，提升软件质量及开发者的技术。代码审查常以不同的形式进行，例如结对编程、非正式的过整个代码，或是正式的软件检查。

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVBf1hWTF9ibD9RUTIbGeIuNyBvldiboMWfia1tq42DVcBouZde3A2X6YIg/640?wx_fmt=png)

这么说下来，是不是有种，看起来很厉害但是好想又什么都没说的样子？帮忙翻译一下，用程序员能理解的话来说，其实，就是 git commit => git merge 的过程。

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVyCucNrMH6G4CeC5eHT7s2I2onFxOgupZ3MEy5hZbiamXTCC87oiaSpsQ/640?wx_fmt=png)

当然，这个过程，需要我们做的事情，可不止提交上去那么简单，通常来说：

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVySA7TtROt0SnYdXibWT7ZuicBLUmdVFqcwg5LejibE8H5vhQv6Le8jDIw/640?wx_fmt=png)

所以，我们这里要讲 CR，并不只是代码提上去，然后由点合并的一个操作按钮的过程。  
而需要评审人从 Code 的变动作为切入点，review 可能延伸到研发的全流程，比如需求的梳理对不对，代码的设计情况，真的的 coding 风格，以及延伸到相关的其他上下文等。  

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVkr1k8KFCia5WpGhuibpykLZkNWb33QyGDLia9jY1MRJsHePiavB3g2Q2xQ/640?wx_fmt=png)

这么听起来好想还是有点抽象，我们看几个现实发生的例子。

第一个，一个看似简单的 API 变更。

前段时间 CR 的时候，看到新增了一个很诡异的 API，叫 forceUpdate，还很认真的写了注释，跟强制更新有关。但是作为一个好奇的 reviewer，需要确认的一个点，为什么这么做呢？  

于是有了一系列对话：**为什么要加这东西？这里这么复杂会不会有性能问题？有没有调研过别的实现方式？如果有问题？影响面会有大？需求上有没有可以商量的可能？**  
  
经过一系列断断续续的连环问答，发现其实是一个省略号的需求，逻辑做得特别复杂，然后页面上出了一些小 bug，加个强制更新可以解决这个 bug。再反推回去，其实整个需求都有一定的优化空间，所以在 cr 的时候，可以根据代码的 diff 去多问问为什么，然后你就会发现：

*   可能一个小细节造成了很大的实现成本，实际上是有讨论空间的
    
*   可能设计上的不大不合理，其实有更成熟的方案，但是他就自己非常努力的去各种实现
    
*   可能代码改动不多，实际上是接口设计不合理某一层偷懒，最后所有的负担都是在这一层去承担
    

这种时候，作为一个评审人，可以拿出勇气，跟你的提交者一起，把需求反推回去，寻找一个比较好的平衡点，而不是疯狂做兼容。**这是一个通过流程延伸，反观需求合理性的例子。**

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVe6J5eMV3ts7Mj5BBvkBbcx04KesjXIDpTuYdFWcrDb2E4Z721OE8BQ/640?wx_fmt=png)

再看一个例子，有一个根据 repeat 字符串方法，然后他加了段兼容的逻辑，判断如果不是字符串，就抛出一个必须为字符串的异常。看起来是不是没什么毛病，甚至想夸一下这同学不错，知道处理异常了。

  
但是，这种非对外透明的调整，对调用方是否有影响呢？比如原来他返回的是 string，现在返回类型是 string 或者异常。如果调用方没有做好异常处理，整体页面是不是就可能崩掉了，然后就会出现一个线上 bug：偶现白屏 ，但是又觉得没有做什么改动，如果没有异常日志上报，问题查起来可能也需要费点功夫。  
  
这个故事告诉我们，对于代码，特别是方法的改动，有时候瞻前顾后是有需要的。以及用好工具，比如我可以通过编辑器看看引用方等。**这其实是一个通过代码 diff 去延伸上下文看的例子。**

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVocZic0mWIPlpajQCdMTH7YvHQ6WPIyn4LTTToK9tzx8woDwaWRDGwEA/640?wx_fmt=png)

以上，是两个很简单的例子，但是可以看出来，通常一段随便代码的提交，他不仅仅是几行字符 diff 这么简单。你看到的变更可能是几个小点稍微调整了一下，但是实际上可能他是在一棵庞大复杂的树下面，正在疯狂努力。

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVvypn8koknVRiaDTgRcfz4NDgicicicIc6hpVB6TkTmPxHyxVOTTXyVakuQ/640?wx_fmt=png)

如果我们只是片面的看 Review，只看 CR 过程中产生的 diff。那么，我们的 CR 就不能走得很远，很有可能，某天，代码就再也维护不动，某天，他就塌了。

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVTqGtfhETRibGAkx3sEicXAVaQbQWDMPCmMORYTMGQTTwyibOiaP6XR5YUg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVBcfMyJJJawibMrun5sRykNzbDTsKZcbE3Y8naspiaUbekSZicLEoTkCTw/640?wx_fmt=png)

**PART 2 · 曾经有过的尝试**  

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVBcfMyJJJawibMrun5sRykNzbDTsKZcbE3Y8naspiaUbekSZicLEoTkCTw/640?wx_fmt=png)

基本概念的大的理念解释完毕之后，接下来看看我们 FE 方向做的一些尝试和努力。  

我是 17 年到百度的，那时候应该是刚从 Gitlab 迁移到内部平台，基本没人管，也没有意识到有 CR 这个过程。当时大多时候是一个人一个项目，以及平台也没有特别严格的手段去要求一定要有人帮你 cr 才能合入，所以很容易沉浸在自己的世界里放飞自我。感觉，CR 就是个摆设。  
  
后面到 18 年，周会没有话题的时候就会去做 CR，主持人轮流主持，自己挑选代码仓库评审并做一些分享。通常单独拉分支，然后通过注释的形式写评论。优点是总算会有人看看别人的代码，但也存在一些问题：

1.  代码库就那么几个，会有一种觉得看过了就不看了的想法，撑不了几回合  
    
2.  容易形式化，经常周会就不小心陷入某个细节
    
3.  代码已经 merge 进去了再做 CR，指出了问题也会由于各种大家都知道的原因，并不会修改  
    

  
接着，19 年尝试制定基本的 CR 规范，然后专门拉了个 CR 群，提交完在群里甩链接 at 人帮忙过。优点：CR 的氛围基本形成，有问题能大家一起讨论，但是可能大多还停留在扣细节阶段。存在问题：

1.  群消息太多，会主动认真 CR 的人太少，项目紧的情况下，经常被无脑合入  
    
2.  新同学一般不好意思，还是容易直接发给自己熟悉的人，并没有什么有效阻止的手段
    
3.  相似问题，缺乏沉淀
    

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVZKSTocZYXZ5dfTE0SSvKvoUAzLkte3UVSQeqRv4H5yegrW8xGxEKEQ/640?wx_fmt=png)  

到了 20 年的时候，整个部门都开始参照此思路，各个方向都拉了一个 CR 群，由各方向 leader 带头每天 CR，而且不同团队的核心成员基本对于各方向的群都在，所以时不时就会遇到，你写的代码被某个 “大佬” 给 CR 了。不过这么来，消息过于爆炸，而且会发现不同群都有类似的问题，一而再的去做讨论修改，比较浪费时间。  
  
所以在去年下半年的时候，针对所有有评论的 CR，都要求去做一定的汇总，然后针对类似的问题会以类似话题的形式沉淀下来，比如 if else 嵌套太多应该怎么办等。周会也加入了一个每周速报的形式，对一些有意思的 CR 由主持人做一定讲解，另外也有团队研发了各种效率机器人方便 CR。  
  
这一套下来，去年应该是前端方向，至少是各负责人都意识到了 CR 的重要性，但是也带来了一定问题。人太多，而且大家分布在不同的项目中，假设两个人一个项目，就算有群，他们也会很愉快的相互 +2 不怎么具体关注细节，跟人强相关。因此有团队正在尝试一种小组 CR 的形式，所有代码都先直接 +2，然后每周结束 reset 回没有 review 的节点，重新提个 cr，大家对此做一定的讨论评审（类似 github 的 pr）。这样对在时间精力上相对消耗不会太多，但是小组需要一个有经验丰富的同学带头，而且要强运营才能推进下去。  

（背景补充：当前厂内的 CR，不同于 github 的 pr，主要是基于 commit 去进行评审的，即每次代码提交后，会单独生成一个评审链接，当此评审大家觉得没问题之后，会有一个打分环节，即 +2 后才允许合入。）  

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZV1pan2bsyx6bYp7gFiaiamzkSLVzvuWVEF2X7ZibUs0H5zaZrY4AA1Y2sg/640?wx_fmt=png)  

以上就是目前的一个情况，截图是我们做的一些事情，以及沉淀的文档。比如刚刚说的 CR 机器人，一些针对知识点的沉淀，还有像是有评论的 cr 问题汇总等等（脱敏需要码比较厚）

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVQ9bFib65QOibky9ZIkaE1jAAvXZbuhuv60jfHbSK9MwRTAHJG0dR10cQ/640?wx_fmt=png)

这么一路走过来呢，个人感觉现状其实不能完全说成功，但能看到大家都在不断持续努力，都在慢慢的进步。

这里主要是想表达一个观点，有些事，道理我们都懂。但是，种种原因，并不是那么容易做。如果有在尝试去推进一些看起来有那么点不一样的东西，可能都会有体会，这里送大家一句话，改了下罗曼罗兰的名言：  

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVDRzAW5yES91xEw90371NwGKPYbp1g4snxqXEgibYqCXdicnT0WzKk8VQ/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVBcfMyJJJawibMrun5sRykNzbDTsKZcbE3Y8naspiaUbekSZicLEoTkCTw/640?wx_fmt=png)

**PART 3 · 遇到问题 & 解决思路**  

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVBcfMyJJJawibMrun5sRykNzbDTsKZcbE3Y8naspiaUbekSZicLEoTkCTw/640?wx_fmt=png)

  
3.1 业务压力

接下来分享一些在这个过程之中，遇到了一些困难，以及觉得算是比较靠谱的解决思路。

首先呢，**最大的一个问题，业务压力**。想象下，每天起床到公司打开电脑，感叹下又是美好的一天，准备好好的帮忙做一下 cr。然后看了看产品的各种需求邮件，PMO 在群里各种 at 你问排期，什么时候上线，我们就像是待宰的小羊羔。

  
这时候，大家通常就会默默的减少 cr 的时间，于是就有了以下真实对话：来来来赶紧过一把，部署下 QA 等着验证; 时间这么紧，就不要掰扯命名这些规范问题了吧，后面慢慢整; 我回头改，先过了吧，提交东西太多了怕有冲突... 等等。这时候，最为一个想认真 cr 的人，我也很无奈啊。

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVBBPPPbiaHoa7ASA1icE4WL87QNarDql1BG8RUBwtDlGYuDwicA00LndDw/640?wx_fmt=png)

这种通常来说有几个问题和解决思路。首先，对于被项目排期各种催的场景，想一下，现在我们的排期，大多是，开发 + 视觉交互还原 + 测试 时间，有时还会留点 buffer。但是 CR 其实也算是编码的一个过程，作为一个过程，他必然是会消耗时间的，那么我们不能当一个规范流程，算在开发时间里面？  
  
前段时间发生的实际场景：项目时间比较紧，尽管提出了问题，但是没有细究，CR 直接快速过了。后面果然有问题，花费了两三天去排查解决，整个模块几乎重写。  
  
通常来说：开发 + CR 讨论时间 < 不 CR，遇到问题后面修改，设计不合理的维护成本。所以这里的一个点，前期就接受他的存在，并将它计算在排期中。  

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZV5wibEyGyICxaaP6fKEAFspIPC8zRBprB8aLZicUbpib7lVHuJkE5OdIMw/640?wx_fmt=png)

第二个问题，也是很容易出现的。业务时间比较紧张的情况下，我们看到一些不大合理但是实际上并不影响大功能的东西，比如代码规范，再比如命名  
如果开发的同学辛辛苦苦的赶工熬夜写代码，然后被测试拿着刀说让你部署一把。然后呢，你在悠闲的嗑着瓜子，跟他说，这里的命名应该语意化一点，具体你想想，那很可能大家谁都想砍你。  
  
这种情况其实是一个团队磨合的问题，没有比较清晰的 重要程度分级概念，主次拿捏不准，所以可能在细节上耗费太多精力。建议就是遵循 CR 中的小细节小分歧不阻塞业务的原则，收录下来讨论即可，除非设计思路上有大问题。CR 的生命周期不在 +2 合入时终止。

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVFyMbXRtBQHicJI8ykw1e17c4OaoCLatWuV1PNzTdkNjfkGib8icNlKzfg/640?wx_fmt=png)

第三个点，可以先看下右边的一个漫画。

你在认真的写代码，然后旁边的小明开心的喊你帮我看个 CR。作为一个认真负责的你，当然是满口答应，准备点进去看看，过了一秒钟打开网页，发现 326 个文件变更，此时求你的心里阴影面积。  
  
这种场景，其实是一次性囤的代码太多导致的，很多同学觉得，我应该把一个功能完整的开发完毕，再提上去，大家才知道我在写什么，但是实际上，coding 是个循序渐进的过程，建议日常开发的同学控制 commit 粒度，尽可能保证每天提交，以及尽可能写好 commit message，还有就是 commit 跟卡片做好关联，也有助于 reviewer 能更好的理解你的意图。  

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVxTLX86OhbcDJ3vdN373QQ7tkTbXibm9aicShwZuq5aIhwWqoOKaVVuicQ/640?wx_fmt=png)

3.2 心态意识  

可以直接看图，一些常见的心理活动：

这些问题说好解还挺好解决的，说不好解毕竟这种是心理问题，有时候也没那么容易。主要产生的原因可能是：

*   对于被评审人，他可能会把技术中的 CR 跟人类天生的 “怕被拒绝” 联系起来
    
*   对于评审人：反过来，他也可能会想着照顾别人感受，不敢给负分，有问题也只是 +1 带着一些评论
    

本质上，**只要理解了理解技术上的「打回」和「不认可」是两回事**就好办了，另外也建议规范不同分数的标准，大家认知统一。例如我们团队内会有一个参考分数（跟公司平台对应）  

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZV9ZvK4PY40RFan7fLruGofbFGu3fH24OxoMwFAClN5d5gDxLuPiawDHQ/640?wx_fmt=png)

这里是，我们定了一些 CR 相关的礼仪，针对做 CR 的同学和提交代码的同学都有一定要求。  
对于评审人：

1.  希望他对每行代码都认真负责，除了代码规范，更需要看业务本身的逻辑  
    
2.  希望多评论，评论不光是说觉得有什么做的不好，也可以是有什么做得好的夸奖
    
3.  时间紧的情况下可以先过，但是一定要做好回顾
    
4.  沉淀通用问题，定期分享总结  
    

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVEBg2ric9iaaMQvE56BPDKTIEpfRtUWfWu1LibrwunoHmbNpvEmibPiavURg/640?wx_fmt=png)

对于代码提交者，也有几个期望：比如一次提交尽可能一个主题，小粒度提交，提交的时候先看看避免有冲突的情况。尽可能每个评审都有回复，具体回复内容可以为，nice, will fix it / feature & 解释 / not a bug 解释 / thanks… 再有经常说的，文档沉淀。

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVM332uEfvRibeyNeJbpsPqSXc9fA2g79B9hOmVp4N4vbfHnAp6RcHQTQ/640?wx_fmt=png)

另外还有一个点，感悟性的评论，曾经我们这边有过一次较为激烈的讨论，简单说一下结论。首先建议尽量减少感悟性的评论，比如：感觉这里不好可以优化一下，不要嵌套这么多层，可以精简，这里太乱了改下吧...  
  
取而代之的应该是：这块不行，要改，这块不行，要咋改，这块不行，要咋改，为什么。  
  
其实这个过程对评审人的要求并不低，也是一个相互学习的过程。现实中由于大家的精力有限，每条都事无巨细的写好可能比较费时间，而且对于提交代码者可能也容易失去思考问题的能力。可以相对权衡一下，团队内有一定的约定，针对不同类型的问题，cr 格式也可以不一样，具体问题具体分析。  

3.3 项目不熟悉  

第三点，项目不熟悉，特别是在一个人跟进一个项目的情况下。我有心，但是东西实在太多了，面对各种什么的业务逻辑，感觉就是魔法攻击。会觉得奇怪，但是又说不出有什么不对，只能当人肉规范检查器。  
  
这里可以插播一个小故事，如果有喜欢玩游戏的同学可能知道，GTA5 联机版本加载速度超级慢，通常要 10 多分钟。然后呢，就有牛逼哄哄的黑客老哥去反编译文件，发现开机会逐个以字符的形式，读取一个 10M 左右六万多行的 json 文件，执行了 19 亿次 if 命令，现在还是如此 :)  
  
想一下，如果要你去做这种代码的 review，是不是肯定会一脸懵，虽然他是能正常运行的。  

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVRPllqMucDwCzaR80td5b7slFibt90DZAIdAJdC3A2fRyywBMMVykx0A/640?wx_fmt=png)

其实，很多时候，我们觉得看起来很复杂的东西，很有可能就是一些代码设计的问题，比如逻辑不清晰抽象程度不够，命名混乱，重复代码过多，代码文件过于庞大等。  
最直观的做法，也不用说我要仔细的去揣测你在干嘛，同作为一个 team 的同学，直接把他拉过来线下做下讲解，再去评论，会节省很多时间（如果远程，电话会议即可）  
  
这里再借用我们厂内一个大佬的一句话：五分钟内无法让我看懂的，这个代码写得一定有问题。  

所以，在 CR 过程中，评审人，可以大胆一点，不用想着说这代码我看不懂是不是我技术不行，看不懂就直接大胆的问，不要被表面各种奇怪的逻辑所迷惑，建议深入到问题具体研究，尽可能避免一些拍脑袋的决策。另外对于我们平时在写代码的时候，也尽量想一想，你写的代码是要让人看的，而不只是给机器看的。

一个比较有效的做法：当你想放飞自我，随便苟且一下的时候。  
**想想准备接手你的代码的可怜兮兮的同事**，比如招了个新同学让你带着一起做项目。或者，再**想想半年后，产品让你在这个模块加点东西，可怜兮兮的自己**。  

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVOCX8P1n6G4lLaNibONJfFpEBAXc7NPWoHh4HlLqNDPtNbGK3ic9MPVWw/640?wx_fmt=png)

有一句话送给大家：**SOMETIMES, It’s “not the code is RIGHT” that matters. INSTEAD, It’s “the code is written RIGHT” is matters.**  

有时候，并不是代码正确，能运行就够了，更多，他应该是被写得是正确的即更应该在写的过程中，需要考虑其可读性可维护性。

3.4 重复低质 CR  

举个例子，我们经常说，一个函数传参不要太多，如果多了尽量写成对象。但是对于新同学，或者对业务不熟的同学，他可能真的只是加个东西，然后加着加着突然就变多了。作为评审人，刚开始可能你会很认真的提醒他，还找到了一些理论对应的支撑文档，然后碰到第二个同学，你指出了问题，给出了建议怎么写的方案，自认为做得很好。  
  
但是，随着 cr 的次数越来越多，你可能发现，这问题还有第三次，第四次，第五次。你也开始慢慢的没有了耐心，可能 cr 就会变成一句话评论，这里参数简洁点，那新同学他就一脸懵。于是，你又重新费劲心思解释了一番，然后叹了口气，**感觉自己每天都在做类似的事情，生命在毫无意义的燃烧，想要有意念脑电波传输功能**。  

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZV34n1gnOy7Ns7Xa8FlYsOKicXYwQEPvTxgbQoQiao5eC4V260c0PWBJTA/640?wx_fmt=png)

那么解决方案呢，其实倒是不难。首先要知道一个道理，人生一大黄金法则：我们曾经踩过的坑，我们会一个个再踩一遍。我也不知道谁说的。  
  
那么怎么尽量减少踩坑呢，一种方法让所有人都踩一遍加深印象，但是随着人员流动团队人数增加显然不靠谱。再有就是需要积极做好文档的沉淀，总结，分享之类的。  
  
比如我们这么两年下来，我们可能会把觉得有价值的 CR 进行收录，上面也有对应的每周有负责的同学去 review 看看，再有可能一个项目中遇到的问题，跟另外的一个项目差不多的，就可以把相关同学拉起来一起做一个讨论，或者可以有一些奖惩措施不过目前只是想法也还没验证过。

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVrQCwCHpKW0ARib0QIBheUD7KMm7iavknL7MFZoeCUIoUx4jdKXWWXKcw/640?wx_fmt=png)

3.5 意见分歧  

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZV1icZYictDULrTpkqVLgpWZBkbed5gD3XzltDerVibzHI06j7RCEeBjrJQ/640?wx_fmt=png)

很多时候，代码的写法并没有绝对的对错，而是在不同的场景下一的一定权衡。但经常，由于大家的视角不一样习惯不同，就会出现我觉得我的做法好，你觉得你的做法好。萝卜青菜各有所爱，一不小心争执不下。  
比如历史上的缩进圣战，还有 php 是不是最好的语言各种梗，持续了一两个年代。

首先声明一个点，能碰到这问题，是个好事。至少说明，大家看问题，有自己的思考在内。那么，碰到这种问题怎么办呢，团队成员之间只要能拥抱事情不可能非黑即白，兼听则明即可。

  
通常建议的做法，以不阻塞业务原则，持续讨论。比如可以闲暇的时候，发起辩论赛，团队内部做一些投票决策或者找技术大牛来协助抉择。当然最后还需要有文档沉淀，不然可能后人又会再发起个讨论，你又要重新说一遍，回到了重复问题的场景。  

  
--- 以上，还有很多，简而言之 ---  
  
假设 Code 件艺术品，那么 CR 就是在 Review 创作的过程，这里有个比较形象的例子  

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVS9rnRB5WcKDFzQrKuwfMia035efLg9hdbibjmHwD54mYC0LnlhVlGViaA/640?wx_fmt=png)

这是一副毕加索的画，叫公牛，具体现在市场价多少没有仔细追究过，不过按照他平均的画作，最少几千万美元是少不了的。很多人看的时候，就会觉得，这不就是儿童的简笔画，我也可以。  

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVOVefKFdJ4Roh0rmm1MlsMng4guGfAmPiah49twMMeqVaozMbr4n0G1g/640?wx_fmt=png)  

但是实际上，当你看到过程图，其实是一步步的抽象过程，跟我们写代码类似。有时候，真的不能只看结果而不看过程，当然，也不能过于注重过程而不看结果，需要大家自己去寻找一个平衡。  

归根结底：要做好 CR，在意识上，心态上，规范上，经验磨合上，沉淀上都需要花一定的功夫，而不单单只是停留在 +1 +2 的按钮上。**他是一个团队相互协作成长的过程**，例如大家对 CR 的一个认同程度，是否都很好的规范支撑你去做 CR，大家是否愿意一起建立规范，文档沉淀去哪里，什么时候去做分享，有没有好用的工具等等。  
  
所以想要做好 CR，不是说我听了这次分享看了这篇文章觉得启发很大，决定要好好做就能做好了，这些都是需要大家一起去探索的过程，当然，听了可能可以少走一些弯路。

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVBcfMyJJJawibMrun5sRykNzbDTsKZcbE3Y8naspiaUbekSZicLEoTkCTw/640?wx_fmt=png)

**PART 4 · THE END**  

![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVBcfMyJJJawibMrun5sRykNzbDTsKZcbE3Y8naspiaUbekSZicLEoTkCTw/640?wx_fmt=png)

其实这里应该有一些小 tips，但是由于跟平台关系比较密切，就先省略。  

除了上面的各种经验，还有很重要的一个点，要想好好做好 CR，Leader 需要身先士卒，主动参与。也许有一天，可能团队就开始慢慢的变好，会开始自我成长。毕竟，梦想还是要有的，万一呢？  

附录，推荐一些本次分享用到的小工具，有需要的可自行百度搜索：  

*   代码高亮美化 Carbon
    
*   手绘风格流程图工具 excalidraw
    
*   有逼格的插画网站 dribbble  
    
*   个人手绘工具 iPad + Apple Pencil
    

最后的最后，感谢大家滑倒了底部，后续可能可以有一些视频可以分享出来，也可能没有。整体稍微零散，也可能没有说期待的技术干货，后续会继续加油。大家有各种想要了解的话题也可以留言点赞给点鼓励（例如请鳗鱼一杯奶茶 ![](https://mmbiz.qpic.cn/mmbiz_png/yrowK5gVe27EjVo2vAvtoCDic2JiaSPmZVklyq3wcsfymbViaicbCzcR5Fer0MI614MtxFURSUC6fVKVL2MyQs1Glw/640?wx_fmt=png)），有空慢慢写系列~~
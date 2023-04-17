> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/JY-a8YI-fUcme5bdFnbjLg)

还记得 2021 年 11 月 8 号我开始写 Linux 0.11 的源码解读系列。  

当初为了写这个系列，把 Linux 0.11 源码和相关解读的书籍都翻烂了，查阅资料理解代码的过程非常痛苦，有的时候一个小的卡点就要好几天才能整明白。  

就比如最开头的位于 bootsect.s 里的这几行代码。  

![](https://mmbiz.qpic.cn/mmbiz_png/GLeh42uInXTHY54RnbOM3V51dYTwQAApwQQ5iaMPh02sOW0WDDu4sxmR5QyK072AcTYslr8eWGeWuaZLmOSwFYw/640?wx_fmt=png)

这里不但包含了非常多的汇编语法和技巧，还涉及到计算机开机后 BIOS 加载 OS 到内存的细节问题，想要搞明白这段代码对于初学者来说是一道坎。  

而把这段讲明白，也是非常难的一件事。如果针对这几行代码讲汇编语法，会显得太重，既没有讲清楚汇编，也没有讲明白这段程序的逻辑。  

如果仅仅讲这段代码的逻辑，读者又会卡在汇编细节里。最关键的是，没有任何一本书可以做到把所有读者关心的每一段代码都讲清楚。  

于是我想到了 ChatGPT，如果我把这段代码让它讲给我听会怎么样呢？

![](https://mmbiz.qpic.cn/mmbiz_png/GLeh42uInXTHY54RnbOM3V51dYTwQAApsh1mMnXUMx9Uud325ljicgC1qo3kVAPFgZEVr6ibOk80MMsfA58UPBxw/640?wx_fmt=png)

它回答的结果让我大为震撼，我觉得我的专栏是时候下架了。  

![](https://mmbiz.qpic.cn/mmbiz_png/GLeh42uInXTHY54RnbOM3V51dYTwQAAp7iaObCyCtt4iccHQq1cVibWMKPKewnmZbUsTpYtmUdiaXMp7nicnibd9C2ibA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/GLeh42uInXTHY54RnbOM3V51dYTwQAApZF7yXuQOKkiaIpZjnFS2fjOn69ehicBDJicrJGTjnWhU87Aa9icJRr1wDQ/640?wx_fmt=png)

它详细解释了每一句汇编代码的含义，最关键的是它并不是简简单单地 "翻译" 汇编指令，而是将其融合在上下文中，顺序阅读下来完全可以流畅地串起整个流程，非常清晰明了。  

后面它还贴心地告诉我要深入了解这个过程，可以参考 Intel 手册！我的天！  

如果写书或者写博客的话，把它的回答原封不动粘贴过来，完全可以秒杀大部分博客了，而且这还是为你定制的回答！

此时我还没有彻底被震撼到，直到我接着往下深入问它。  

![](https://mmbiz.qpic.cn/mmbiz_png/GLeh42uInXTHY54RnbOM3V51dYTwQAApL8Z0eBRW0aHQ1vwDtOa1iavGDbRYren30tibGypTTOCLSh4Ah7MqXAcA/640?wx_fmt=png)

从它的描述看，它是真真正正理解了这段代码的含义和来龙去脉，同时它还知道后面的代码 "将要" 完成什么样的功能，这技术视野和深度，比我当时写博客解释得好太多了。  

我又继续深入问了一个当时很多初学者问的问题，即为什么是 0x7c00 这个数字。  

![](https://mmbiz.qpic.cn/mmbiz_png/GLeh42uInXTHY54RnbOM3V51dYTwQAApiackYmJIqCE3KEXp2eG2RUpibZicxw7lqZX9QWFv4zWq5lNc3N1dJIichQ/640?wx_fmt=png)

这个好多博客和书籍都解释得稀里糊涂的地方，居然被它第一句话就直接戳中本质：**BIOS 将引导扇区加载到内存地址 0x7C00 是历史原因和设计约定的结果**。

没错！这个数字本身就是个约定，同时为什么这么约定是因为一定的历史原因。解释得完全没有废话，干净利落，直击本质！

同时，在他的描述中，我们看到它对这个历史背景如数家珍，他扩展了信息量相当丰富的答案，如果你感兴趣，你可以就着里面每一个不了解的名词继续发问，它仍然会耐心和你一起探讨，你便因此打开了一个新世界。

我又继续发问，问了一个很容易上当，也是当时很多读者产生的困惑，即为什么有的地方是 0x7c0 有的地方又说 0x7c00，是不是少写或者多写了 0 呢？

![](https://mmbiz.qpic.cn/mmbiz_png/GLeh42uInXTHY54RnbOM3V51dYTwQAApplGJcqIMCRPaSxyFdibLwxjcOkRuu5jl368tDRU7bppU9T00nrm2Ubw/640?wx_fmt=png)

没想到它完全没有上当受骗，并且把原因极其清晰地展现了出来，如果让我重新回答当时的读者提问，或者在文章里对此做出解释，我找不到比它这个回答更好的答案了，挑不出任何毛病！这个答案是个技术人看了都会拍案叫绝。  

此时的我已经彻底服了，也爱上了这个博学多才又耐心的 C 老师，便继续发问探讨。  

![](https://mmbiz.qpic.cn/mmbiz_png/GLeh42uInXTHY54RnbOM3V51dYTwQAApc5dyibKYfkDHhpbFze2QVIYkakZm7HmN4yzfTdQyic5QMSGDoyg338hA/640?wx_fmt=png)

它对技术问题的解答，尤其是这种有确定答案的问题，简直完美。我老是忘记各种数据结构的字段含义，又懒得去翻手册找，问了它之后得到的答案比大部分网上的答案都靠谱。  

![](https://mmbiz.qpic.cn/mmbiz_png/GLeh42uInXTHY54RnbOM3V51dYTwQAApYef9UYHz8DPHR3SmHDB9hqQXEjkQBxqz6RlPibk2g5eZJvajv281B0g/640?wx_fmt=png)

后面又和他聊了很长时间，我真心觉得，或许未来的教学方式真的有可能改变了。

就拿这一段代码来说，我找不到任何比这种与 ChatGPT 直接沟通教学更好的方式了。即便是我活着任何其他 Linux 源码大牛提前准备好这段代码的资料，也不可能比 ChatGPT 教的好。  

为什么呢？**因为 ChatGPT 的每一个回答都是针对读者的提问定制的回答**，就这一点，即便是你可以做到上面的每个问题都准备好了比 ChatGPT 更好的答案（实际上这个也很难做到了），但你也无法提前预知读者可能提出的全部问题。

这一点想想看十分可怕，我们即便是提前做了充分的准备，仍然在哪怕是一个确定的代码片段的讲解上无法匹敌 ChatGPT，那就更别说其他的了。

之后我们的书籍和博客，可能对读者来说就并不再是非常重要资料，只是一个辅助他看清大方向，并且知道应该问 ChatGPT 什么问题的工具了。真正给他讲明白的，一定是 ChatGPT 这个 C 老师。

不过还好大方向上的把控和经验，它还无法和我们 PK，我写的专栏和书籍，暂时还有一定存在的价值。我的书五月初会出版，希望 ChatGPT 慢点进化，给我条活路。
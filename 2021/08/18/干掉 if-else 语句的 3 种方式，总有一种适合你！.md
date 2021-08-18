> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIwNDY1NTU1OA==&mid=2247494906&idx=1&sn=5ea2bed5eb83e525c579a4fd8244084a&chksm=973e7253a049fb45afe818e527213ce316f01f6a665849126fbaf9c7af0d3fb2bccebf93a0ba&scene=21#wechat_redirect)

来源丨 love1024.blog.csdn.net/article/details/104955363

场景
==

日常开发，if-else 语句写的不少吧？？当逻辑分支非常多的时候，if-else 套了一层又一层，虽然业务功能倒是实现了，但是看起来是真的很不优雅，尤其是对于我这种有强迫症的程序 "猿"，看到这么多 if-else，脑袋瓜子就嗡嗡的，总想着解锁新姿势：干掉过多的 if-else！！！

**本文将介绍三板斧手段：**

*   [优先判断条件，条件不满足的，逻辑及时中断返回；](http://mp.weixin.qq.com/s?__biz=MzA3MjMwMzg2Nw==&mid=2247498404&idx=2&sn=d501b955843386d69b41aef6684d5a1a&chksm=9f22ef30a8556626207b2710e8b57761d725ea054871cad589e945d5359623600c923e9378af&scene=21#wechat_redirect)
    
*   [融入策略模式；](http://mp.weixin.qq.com/s?__biz=MzA3MjMwMzg2Nw==&mid=2247498404&idx=2&sn=d501b955843386d69b41aef6684d5a1a&chksm=9f22ef30a8556626207b2710e8b57761d725ea054871cad589e945d5359623600c923e9378af&scene=21#wechat_redirect)
    
*   [策略模式 + 工厂 + 单例模式，锦上添花；](http://mp.weixin.qq.com/s?__biz=MzA3MjMwMzg2Nw==&mid=2247498404&idx=2&sn=d501b955843386d69b41aef6684d5a1a&chksm=9f22ef30a8556626207b2710e8b57761d725ea054871cad589e945d5359623600c923e9378af&scene=21#wechat_redirect)
    

接下来先附上一段很久以前自己写的业务代码，核心逻辑就是在支付回调中根据用户购买的价格包赋予用户对应的权益 (VIP 视频会员天数 + 抽奖机会次数)。

我的天，太多 if-else 了……(看不清楚可以点击图片放大)

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufDY3Ia1LwTraFGkEHZOEmhreNTaep8nptJcpzSDoTC1oWAQ0yYjhgFQcntlIDicZzWPoDh3YZqNHg/640?wx_fmt=png)

### 1. 优先判断条件，不满足及时中断

这点非常容易理解，就是说在业务逻辑里面，先把不符合条件的给先过滤掉，而不是层层嵌套 if-else 判断，结合代码图看一下：

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufDY3Ia1LwTraFGkEHZOEmhtvaMlYo1Q10B1IHf9vibZtoWtbffJo3TKkSyqyKaMMFUxoJmia0b6bCg/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufDY3Ia1LwTraFGkEHZOEmh45MSxSuSLXuoGBzflXkDLu54sJzSu7eF4KE2SVJobPyLzCPtGzTnxg/640?wx_fmt=png)

### 2. 策略模式改造

先用策略模式替换掉文章开头讲到的，用户充值后根据价格包 (付的多少钱) 给用户增加 VIP 天数及抽检机会次数的逻辑，我这里就简化成 "根据 - 价格包区分给用户增加不同的体育会员视频 VIP 天数" 这个动作来讲解：

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufDY3Ia1LwTraFGkEHZOEmhSoZXrETAdr9x7LoF5DLicAQREp4OqUbsTV1MN0zaDbCoWS5DWNtBSKg/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufDY3Ia1LwTraFGkEHZOEmh8Bic1wMWGPKgy3ZeEmXOJ8Veuh2GluSxJCfKYWK4aU6VTZ1u2TccAKA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufDY3Ia1LwTraFGkEHZOEmhnRqAlUEzwoRqv2VAhXhoWKw3jNGvZT1eZPic70x6WkVhllG7gLibLUnA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufDY3Ia1LwTraFGkEHZOEmhF4kYHGafyIqwSibttSIfqX3g38gLz9ZFmx2YItLQNibkicXlOicga16eibQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufDY3Ia1LwTraFGkEHZOEmhZVnAurkyXMJyhp5AL7RDCU6MLCLe1c2ehQvPWXX8W58SWdVmibQRLmg/640?wx_fmt=png)[最新 Java 面试题出炉！（带全部答案）](http://mp.weixin.qq.com/s?__biz=MzA3MjMwMzg2Nw==&mid=2247498404&idx=2&sn=d501b955843386d69b41aef6684d5a1a&chksm=9f22ef30a8556626207b2710e8b57761d725ea054871cad589e945d5359623600c923e9378af&scene=21#wechat_redirect)

表面上看，代码稍微优雅了点，但是还是没和 if-else 彻底说拜拜，且`recharge()`充值方法可单独拎出来，只需要根据 priceCode 实例化不同的策略对象即可：

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufDY3Ia1LwTraFGkEHZOEmhIFCLc2X5xeutIT4cyfHWDeI1L3KjMibIpRBf1RESX6cWWWibbaBnBh4Q/640?wx_fmt=png)

### 3. 策略模式 + 工厂 + 单例模式，锦上添花

接下来使用 "工厂类 + 单例" 来给代码加点料:

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufDY3Ia1LwTraFGkEHZOEmhVaiccMUGuEhVgGHTZKicfw0Wrs1XSM6iceiaCcD6Qr5gez890Cu6EoE8Qg/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufDY3Ia1LwTraFGkEHZOEmhtO8QxoJoDzDlyVduWiaic6zvfmJvVr1t36lGRNkz3Wx8d6nZXny4C5dA/640?wx_fmt=png)

* * *
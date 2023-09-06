> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247525213&idx=2&sn=ba7775b4aff1e4f47809d9c15dd30577&chksm=e9693d50de1eb4467cc05214b8fb10dc2b99644a084849fb06850d9d276c7856ddc2a4e160d0&scene=21#wechat_redirect)

鹅妹子嘤！[斯坦福用 ChatGPT 打造的「AI 小镇」](https://mp.weixin.qq.com/s?__biz=MzIzNjc1NzUzMw==&mid=2247672915&idx=3&sn=9aa7a4ac24ea054d1580078f1cb5db05&scene=21#wechat_redirect)，可以在自己的电脑上跑了，还是能定制的那种！

这是一个由 25 个 ChatGPT 组成的虚拟世界，完完全全地模拟了真实的人类生活。

就在刚刚，作者在 GitHub 上公布了这个项目的源代码，还贴心地附上了部署教程。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBvdh1pHxVXIfX1RoHVqJNpgicjFUUnAibJiaLyagj9RcVIia2D8Lok7yAtnfKnYOIQ5mjNkTOA9peNSw/640?wx_fmt=png&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

目前获得的星标已经超过了 1.3K，而且还在上涨！

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBvdh1pHxVXIfX1RoHVqJNpcibP8ZKWcqrjibAmwlWLE30rTpRn9TwaI5z3FFfIUv2o1ic9r0iaAhWY5w/640?wx_fmt=png&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

英伟达的 Jim Fan 博士第一时间点赞转发，直言「前方有无限多新的可能」。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBvdh1pHxVXIfX1RoHVqJNpbVARAv2S6S1JuicNCvKXPuL4asANkoOJ2WVwMexMIEgxBLoXost6WFw/640?wx_fmt=png&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

网友们则开始了许愿，期待着推出一个宝可梦版本：

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBvdh1pHxVXIfX1RoHVqJNpobZcBrNluteF5Enu04FhWdRZmbzE4m7MO8chN0zVyDvhvV0vz1LLgw/640?wx_fmt=png&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

创建自己的小镇
-------

这个小镇中，可以添加最多 25 个由 ChatGPT 扮演的角色。

他们有着不同的身份、性格和年龄，共同生活在这个小镇里。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBvdh1pHxVXIfX1RoHVqJNp49IlkojxP1Awlc6cGRC0zMlAiamUZhgTOE7UIHW0fmO6AYUnDIXVvBw/640?wx_fmt=png&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

他们像人类一样进行着自己的活动，也像人类一样彼此交流。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBvdh1pHxVXIfX1RoHVqJNpFh6zhYmWbyYJhRMPbJBBHBU6fjs8k2yJ9G1NC1vvf0b5X7PibNaPsMw/640?wx_fmt=png&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

交流时所用的，**也是人类的语言**。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBvdh1pHxVXIfX1RoHVqJNpG2I4s1srgMqIgefnGZEWjIaRkzC4B31qiaSzdjsvU7VoaSvGQw4IQrw/640?wx_fmt=png&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

那么，该如何打造一个这样的「AI 小镇」呢？

在开始之前，先要检查 GitHub 文档里的 requirements.txt 列出的依赖是否都安装好了。

然后把项目文件克隆到本地，并创建配置文件。

配置文件要放在 reverie 目录下的 backend_server 中，并命名为 utils.py。

其中填入如下的内容，注意替换 OpenAI 的 API。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBvdh1pHxVXIfX1RoHVqJNpQiawocibNgwZDcYuQSmX2dtRkMYib68wMOicDvicEYaMYl9jgGMQDMcjvFQ/640?wx_fmt=png&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

配置好这些之后，就可以开始运行啦！

![](https://mmbiz.qpic.cn/mmbiz_gif/YicUhk5aAGtBvdh1pHxVXIfX1RoHVqJNpBmc6E8Tp1ekj1IrjibKxpRGCadFpfOKC8lIISkpz16RYD2mdrkXskSA/640?wx_fmt=gif&tp=wxpic&wxfrom=5&wx_lazy=1)

首先转到 environment 目录下的 frontend_server，开启运行环境。

> python manage.py runserver

启动完成后，可以通过浏览器访问 localhost:8000 检查是否成功。

如果成功的话，会看到下面的内容：

> Your environment server is up and running

环境启动完毕，就可以开启主程序了，还是要回到 reverie 目录下的 backend_server 当中。

也就是之前存放配置文件的目录。

> python reverie.py

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBvdh1pHxVXIfX1RoHVqJNpeYbZdlnA29XbnVw9pSNhZDiaiaTsxPTeOZicHm2K5HvKT85biaHrcHAqaw/640?wx_fmt=png&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

输入启动指令之后，程序会让我们输入人物的名字。

这里人物的名字要在程序预设的列表里有才行，而且要按照下面的格式：

> base_the_ville_人物名 1_人物名 2……

假如我们选择的是 Isabella Rodriguez、Maria Lopez 和 Klaus Mueller 输入的内容就是：

> base_the_ville_isabella_maria_klaus

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBvdh1pHxVXIfX1RoHVqJNpP6PhnVcb9r0b7nmwelR7FC1icq8PtZOuOY17RKvfvN75VAhyUbSLK8Q/640?wx_fmt=png&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

###### **△**程序内置的人物名

然后也给这个小镇起个名，并记住这个名字。

用浏览器打开 localhost:8000/simulator_home，这个「AI 小镇」就呈现在我们眼前了。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBvdh1pHxVXIfX1RoHVqJNpcydf58NSTPPpLXO2U27xSTKROwzN2CiczhT4dsYp0cjYdNv6asgAZicg/640?wx_fmt=png&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

回到命令行界面当中，会看到系统要求我们输入选项。

此时输入 run + 空格 + 次数，回车之后小镇里的人们就动起来了。

此外，模拟的整个过程都可以回放，只需要用浏览器访问这个地址：

> localhost:8000/replay / 小镇名称 / 想看的步骤

注意替换里面的名字和步骤序号。

历史记录文件也可以在这个目录里找到：

> environment/frontend_server/static_dirs/assets/the_ville

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBvdh1pHxVXIfX1RoHVqJNpNIlBtIljNAjFSTiaLtRzgiagibn9vpHZWhK9O7B3yiaod4LpKUqXAFuibdg/640?wx_fmt=png&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

此外，还可以对小镇中上演的故事开头进行「定制」。

还是按照前面的步骤，在给小镇命名之后，系统要求输入选项时使用下面的命令：

> call -- load history the_ville / 历史记录文件名. csv

此时程序会把读取到的记录作为故事的开始。

由于历史记录是**明文可编辑**的，所以通过**修改这些记录**也就实现了内容的定制。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBvdh1pHxVXIfX1RoHVqJNpHRfj5u4Kiaia2iam6OZ9GMoHN1suhjnEBuudJiaZGIMrzH5xAmS2icUHCQg/640?wx_fmt=png&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

除了将代码进行开源，团队也对此前发布的论文进行了一些修改。

新版本中，作者替换了一些用词，格式也更加完善，不过核心内容上改变不大。

相比于旧版，新版本还增加了 Acknowledgement 部分，对为本项目做出贡献的人和机构表达了感谢。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBvdh1pHxVXIfX1RoHVqJNpsnPhhyVyEd5bC87YOiaVl0jeGd4kL4vETXP2ibqLKqFYb5otzteDxR4Q/640?wx_fmt=png&tp=wxpic&wxfrom=5&wx_lazy=1&wx_co=1)

此外，作者还对参考文献进行了调整，数量上少了一篇，顺序也有所改动。

快来打造一个属于你的「AI 小镇」吧！

GitHub 页面：  
https://github.com/joonspk-research/generative_agents  
参考链接：  
https://twitter.com/drjimfan/status/1689315683958652928  
论文地址：  
https://arxiv.org/pdf/2304.03442.pdf

--END--
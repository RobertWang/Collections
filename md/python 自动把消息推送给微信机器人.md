> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/yIR-owCu_8Ly8NZS5kObqg)

> 很多时候，我们希望监控一些最新信息，能够第一时间在微信上看到。

背景
==

很多时候，我们希望监控一些最新信息，能够第一时间在微信上看到。现在有很多这方面的消息推送工具，比如 wxpusher、Pushplus、server 饭、server 酱等：

注册平台
====

下面以 wxpusher 为例，说明如何做一个信息推送的微信机器人。在 wxpusher 网站（https://wxpusher.zjiecode.com）注册一个账号

![](https://mmbiz.qpic.cn/mmbiz_png/euGjuBN81k4Nibb9g3AaH7tLiaibNDS3qGwvBeHBq3q1x8ibLdjWR8xWe3IODK5RyZhG1CibWicjjPqSxDwO58YWjkibw/640?wx_fmt=png)image-20230903230148865

然后创建一个应用

![](https://mmbiz.qpic.cn/mmbiz_png/euGjuBN81k4Nibb9g3AaH7tLiaibNDS3qGwEprtf4VMOs1jL0HuliauVpKxZWdgrtHKgTWv5ZKsyLHGCVSgQq9VbDw/640?wx_fmt=png)image-20230903230212592

自己关注一下应用

![](https://mmbiz.qpic.cn/mmbiz_png/euGjuBN81k4Nibb9g3AaH7tLiaibNDS3qGwosbnbb8v9FLTrnTBQpsy3SkIfIMuSp9nUurqbKiboPnWia6LtrxTN3Vg/640?wx_fmt=png)image-20230903230308948

然后在用户列表，找到自己的 uid

![](https://mmbiz.qpic.cn/mmbiz_png/euGjuBN81k4Nibb9g3AaH7tLiaibNDS3qGwd57dquuZQRZNrrFYMlndpjKibwQFtw3aRTibAmrGbXSgUQCEUWuqjd5A/640?wx_fmt=png)image-20230903230419758

功能实现
====

您需要安装 WxPusher 的 Python SDK。您可以使用 pip 命令来安装它

```
pip install wxpusher -i https://pypi.org/simple

```

下边是代码

```
from wxpusher import WxPusher

app_token = "AT_QHEZNz5ZetcUVPRjvpknxBNlJjyxxxxx"
uid = "UID_1hE13Mx7QO2d7zEXwn5oHUMxxxxx"
content = "又出BUG了，赶紧来处理！"

WxPusher.send_message(content, token=app_token, uids=[uid])


```

你的微信就收到消息了

![](https://mmbiz.qpic.cn/mmbiz_png/euGjuBN81k4Nibb9g3AaH7tLiaibNDS3qGwKDDYkhiaufSHjTB8ZYNT9kic46hYhfCuDY9mTbpbpsmhxB51ODM4ialeQ/640?wx_fmt=png)image-20230903231026999

> WxPusher 推送的是实时消息，时效性比较强，过期以后消息也就没有价值了，目前 WxPusher 会为你保留 7 天的数据 ，7 天以后不再提供可靠性保证，会不定时清理历史消息；
> 
> 单条消息的数据长度 (字符数) 限制是：content<40000;summary<100;url<400;
> 
> 单条消息最大发送 UID 的数量 < 2000，单条消息最大发送 topicIds 的数量 < 5;
> 
> 单个微信用户，也就是单个 UID，每天最多接收 500 条消息 。

我有个大胆的想法
--------

小伙伴在平常有没有遇到以下这种情况：遇到技术难题时，网上教程一堆堆，优秀的很多，但也有很多是过时的，或者是 copy 来 copy 去，甚至错别字都没改。

我公众号的技术文章，都是亲自校验过的。至少可以保证在发文的一段时间，不会过时。如果你在实操过程中，有遇到问题，可以在同名公众号留言，免费解答，相互学习，相互成长 ^v^
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/2HLA4YlLdLmdFKD5xAbLSg)

**业务背景  
**

假设你现在正在一个创业公司担任 CTO，因为微信工作生活娱乐不区分，已经发生了很多次将敏感信息发错人甚至发错群的尴尬事件了！你司 CEO 决定做一款 IM 工具，为了区别微信和 QQ 大众化的 IM 需求，你们公司主打安全 IM，这款产品的竞争力如下：主打私密聊天，严格控制私密好友的数量，而不是像微信一样，买个菜都可能要加个微信。

【公司背景】

1.  技术团队大约 10 个人，后端 6 个，前端 2 个，  Android 2 个，  iOS 还没有；

2. 后端 Java 为主，大部分是 3-5 年左右的；

3.  后端具备 MySQL、微服务、  Redis 等开发使用经验；

4.  后端没有大数据和推荐相关经验。

**业务基本场景**

![](https://mmbiz.qpic.cn/mmbiz_png/BQNLpqZPuV7GXzGmZu5BiaMuSicQsj07GeicPMnicDE1gaMG6e7tfIWuwuqEmy9PqeN9HWM7lJwQ5ol3hXeR5zXhYQ/640?wx_fmt=png)

1. 每个用户都会通过算法生成非对称的公钥和私钥；

2. 用户发送的消息会通过公钥加密，接收用户的消息使用自己的私钥解密；

3.  只能创建一对一聊天；

4. 聊天消息 “阅后即焚”，最多只保留 60 分钟；

5. 无需使用手机号注册；

6.  每个用户最多 20 个好友。

总体架构思路  

老板说我们 3 年内要做到 1 千万注册用户，作为 CTO 的你应该如何做架构设计？

十万：落地快，但是如果业务发展很快， 架构很快不适应了怎么办？

百万：落地慢一些，但同样面临业务发展过 快的风险。

千万：落地时间可能要 6 个月以上，但基本上 3 年内无需再动架构。

**超前设计，架构真的不用动么？**

![](https://mmbiz.qpic.cn/mmbiz_png/BQNLpqZPuV7GXzGmZu5BiaMuSicQsj07GeGdofpb7Gl48nak6U7a6MA26aSM7t6utYOXqYLVibia5lDfpNSYtP83BQ/640?wx_fmt=png)

1. 业务规模变化

可以超前化设计应对。

2. 业务多样性

无法预测会做什么功能，  业务多样 性会导致团队人数增多到多少更加 无法预测。

3. 技术发展

无法预测，尤其是和法律政策相关的，例如区块链、国产化。

十万用户规模存储性能估算

![](https://mmbiz.qpic.cn/mmbiz_png/BQNLpqZPuV7GXzGmZu5BiaMuSicQsj07GeymFY5MmBmH2ibCuH8iantWLAebLgW2JTAdwx8BGxicwLYiaeCtsjQ7atYQ/640?wx_fmt=png)

【注册】

十万用户注册信息。

【登录】

虽然 IM 是比较活跃的产品，但由于是全新的产品，我们假设十万注册用户，每天活跃用户有 40%，登录每天 4 万。

【加好友】

每个活跃用户最多 20 个好友，好友关系数 4 万 * 20 = 80 万 关系数据。

【聊天】

假设每个活跃用户每天向 5 位好友发送 100 条消息，则消息数量为：4 万 * 5 * 100 = 2000 万，且数据当天基本都被删除了， 所以写入、读取、删除次数都可以估算为 2000 万。

**存储架构设计**

![](https://mmbiz.qpic.cn/mmbiz_png/BQNLpqZPuV7GXzGmZu5BiaMuSicQsj07GetCM2rpOmAhgtAd3CQeeeB9vhJb26IwSjxNwVbaGQEhBjkPlVRzaTicA/640?wx_fmt=png)

10 万用户注册信息， 4 万登录请求 ，80 万关系数据。

![](https://mmbiz.qpic.cn/mmbiz_png/BQNLpqZPuV7GXzGmZu5BiaMuSicQsj07GeCic8E4gno9SoI8oR0mRmKJhJ0aFdtJH21cOMX9ruWsVStIkBgzmyWdg/640?wx_fmt=png)2000 万写入，  2000 万读取，  2000 万删除。

**十万用户规模计算性能估算**

![](https://mmbiz.qpic.cn/mmbiz_png/BQNLpqZPuV7GXzGmZu5BiaMuSicQsj07GeymFY5MmBmH2ibCuH8iantWLAebLgW2JTAdwx8BGxicwLYiaeCtsjQ7atYQ/640?wx_fmt=png)

【注册】

1 年达到十万用户注册，注册 TPS 很低。

【登录】

虽然 IM 是比较活跃的产品，但由于是全新的产品，我们假设十万注册用户，每天活跃用户有 40%，假设登录时间集中在早晚 4 小时，登录 TPS 均值：4 万 / 14400 = 3。

【加好友】

每个活跃用户最多 20 个好友，好友关系数 4 万 * 20 = 80 万数据，按照 1 年内来计算，  TPS 可以忽略不计。

【聊天】

假设每个活跃用户每天向 5 位好友发送 100 条消息，则消息数量为：4 万 * 5 * 100 = 2000 万;

假设每天集中在早中晚 3 个时间段 6 小时内 (早上 1 小时中午 1 小时晚上 4 小时)；

发送消息 TPS ：2000 万 / (3600*6)  ≈ 1000；

       读取消息 QPS = 发送消息 TPS，删除消息 TPS ≈发送消息 TPS。

**计算架构之负载均衡**

![](https://mmbiz.qpic.cn/mmbiz_png/BQNLpqZPuV7GXzGmZu5BiaMuSicQsj07Ge4Kn1oDh7MTnPaLKnoefULCWeG0kicxwPaDNjFXsYib89cXEmY3KdGocQ/640?wx_fmt=png)

**计算架构之缓存架构**

![](https://mmbiz.qpic.cn/mmbiz_png/BQNLpqZPuV7GXzGmZu5BiaMuSicQsj07Gegvt95yQz0VfSopC5MicURUBPNwEKRrvPrdg4lr492qCicxWic2QqAkViaQ/640?wx_fmt=png)

**可扩展架构设计**

![](https://mmbiz.qpic.cn/mmbiz_png/BQNLpqZPuV7GXzGmZu5BiaMuSicQsj07GeENiaxT72qZbaHJptsWEUht9M0gXic3fOV1pxuf9YLVsWicOS2MVXAY3Vg/640?wx_fmt=png)

**高可用架构设计 - 同城数据灾备**

![](https://mmbiz.qpic.cn/mmbiz_png/BQNLpqZPuV7GXzGmZu5BiaMuSicQsj07GeTARJgX92oulL4TEQz6FHuAN6RF2bxEuibwkZk5K1v37LKd2Oct2OGMA/640?wx_fmt=png)
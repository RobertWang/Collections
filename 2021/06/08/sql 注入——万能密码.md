> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1623068181&ver=3116&signature=rfqrxYTBf51H3KIV6wuTLgQ73Qs26unJpxdSrZTcw8hDF*-rrASkQjg59HLu8HIlIJrCIvU2eRiPUM7L49JtqFAS73bQTKNH6sZPRuoVWW2IoJ1cs*sKrl31RYSUa334&new=1)

  

  

通过两个 ctf 题来做讲解，第一个很简单，第二个多了些限制

http://lab1.xseclab.com/sqli2_3265b4852c13383560327d1c31550b60/index.php

http://ctf5.shiyanbar.com/web/wonderkun/web/index.html

  

万能密码原理

万能密码漏洞都是因为提交字符未加过滤或过滤不严而导致的。

```
$sql = "SELECT * FROM user WHERE username = '{$username}' AND password = '{$password}'";
$res = $dbConnect->query($sql);

```

后台数据库中会执行  

```
select * from user where username = ' ' and password = ''

```

两种情况：

1. 已知账户名

既然知道了 username，就可以将后面的注释使逻辑为真

常用方法：

![](https://mmbiz.qpic.cn/mmbiz_png/Kbb3MRHusib26JqM4bDTD5sXmQgDJia6hncAu1RdJuH1JYgT69pwawh80Hc4eLCGStXHd6xXiaJ2MADlAj2T6eBHA/640?wx_fmt=png)

2. 未知账户名  

需要通过构造 sql 语句使 where 为真

常用方法：

![](https://mmbiz.qpic.cn/mmbiz_png/Kbb3MRHusib26JqM4bDTD5sXmQgDJia6hnich5TzjqGTJpChjiaF0UYGyCVicvxXfN9Qef7303lHj8C58j3szibZSdog/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Kbb3MRHusib26JqM4bDTD5sXmQgDJia6hnlrNYOsHCZW4hzus5H7PAleJFoXSSnvybJ8FliaHTh2YRs15nBmuSiarg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Kbb3MRHusib26JqM4bDTD5sXmQgDJia6hn7PH7EOUkEbnPZibOlPveZer5HYxewicSGZxmnkaSBjMUzF1Wyw2ib173Q/640?wx_fmt=png)

''+' ' 相当于 false（0）  

![](https://mmbiz.qpic.cn/mmbiz_png/Kbb3MRHusib26JqM4bDTD5sXmQgDJia6hnyibkcwTxE9qucTg3iaoVZkmTib7ib4yo76W15htmeNZicWc2TkeVibvutnzQ/640?wx_fmt=png)

'' =' ' 为 Ture（1）

''+' '和' '=' ' 和 0 = 0 都是为了使  

where username = '' and password =' '成立

where 0 and 0 或 where 1 and 1

  

结合例子讲解

第一题：  

源码中告诉我们有个用户名是 admin

![](https://mmbiz.qpic.cn/mmbiz_png/Kbb3MRHusib26JqM4bDTD5sXmQgDJia6hnQic58lTv4AiagbkiaM4IK2RZiaGTliafR7BZSPHNm0ow04IaxPx6aOEPicVA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Kbb3MRHusib26JqM4bDTD5sXmQgDJia6hnUuaicyVsHygEMyacRAeHfBZia8XplcOg9VTibNWetohibxMlmebd6HVfeQ/640?wx_fmt=png)

假如我们并不知道 admin 这个用户

![](https://mmbiz.qpic.cn/mmbiz_png/Kbb3MRHusib26JqM4bDTD5sXmQgDJia6hn9qDOZ2LG1MI77LeLrPd6PelRLpdNnPtKBvQcian2JZbkmB18ABiaZoxw/640?wx_fmt=png)

发现也可以成功绕过  
  

第二题：

我们不知道任何用户名的情况下  

![](https://mmbiz.qpic.cn/mmbiz_png/Kbb3MRHusib26JqM4bDTD5sXmQgDJia6hnGHQXjLjlE3ibRnzRP5AXO5NibmM8riaibmk9ATrQB8zRnicf8c0g4sKh33Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Kbb3MRHusib26JqM4bDTD5sXmQgDJia6hnsDIgKrQurBZ4oEx80ygDicQTEwb1o8RfyrVRa9h9XyUt4ia0XiavTKFmg/640?wx_fmt=png)

我们发现貌似不行了，or 被过滤了。看看都过滤了哪些字符

![](https://mmbiz.qpic.cn/mmbiz_png/Kbb3MRHusib26JqM4bDTD5sXmQgDJia6hnbflNLDKEfrezYIu4FBNiafeiaicrdNaSE28XqGbWFqe6qH1mNbWlEp9pg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Kbb3MRHusib26JqM4bDTD5sXmQgDJia6hnBy8Gbke8UXBibs9eRDDISliadYQLWURWg98xA5apg3KggElMIRuvCZhg/640?wx_fmt=png)

可以看到 # and  or 都被过滤了，+ 和 = 没有被过滤

![](https://mmbiz.qpic.cn/mmbiz_png/Kbb3MRHusib26JqM4bDTD5sXmQgDJia6hnibT18m3Huknoc4RibCrDxXVTrNbjicib83MESKEH6R3zlUdGeKUGzHrXUQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Kbb3MRHusib26JqM4bDTD5sXmQgDJia6hnCZNPwB86hVeMJHqaqxYKLwicCB6vcogCySqmibqib3l3ERUKUKX5gh8gg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Kbb3MRHusib26JqM4bDTD5sXmQgDJia6hnAooKtyNjrEmibOwhggg8pL6Yl2SibibaKUGRhrlX6ict3pUtjh9Nms3NEQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Kbb3MRHusib26JqM4bDTD5sXmQgDJia6hnEgvSCETYPIygnQPmHTX91ZUegcB7ypL5ygx3OeQwXKpykeLymnC9HA/640?wx_fmt=png)

最终通过'' =' ' 获得了 flag  

对于 select * from user where username = '' and password =''来说

只要我们能够构造出合适的语句使 where 后面的为真就可以了

本质上就是使 where 1 and 1 或 where 0 and 0

  

* * *

内 容 来 源 | 五 号 黯 区

 官 网 地 址 | www.dark5.net
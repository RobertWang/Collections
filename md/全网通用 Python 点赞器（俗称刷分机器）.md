> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/d0Mxo4a_MPYUnTJSM8JyXg)

> 全网通用 Python 点赞器（俗称刷分机器）

> 在今天，任何一个社区类平台，都具备点赞功能，应运而生的就是自动点赞器，俗称刷分机 / 刷赞器。

本文将为你介绍一款点赞机器人，最简单易理解的核心逻辑。

全文涉及的伪代码，使用 Python 编写，由于是伪代码的原因，不懂 Python，你也能看懂。

本篇博客试用场景
--------

本次点赞机器人，主要面向电脑上的 Web 站点，不涉及 APP 端。

点赞机器人核心逻辑
---------

模拟点击操作，触发点赞，喜欢等操作。

实现点赞操作前，还有一步重要的代码实现，模拟登录。

因此，点赞机器人的基本需求如下：

1.  模拟登录；
    
2.  进行点赞；
    

对该需求进行扩展后，存在两个常见的业务场景。

1.  通过模拟登录大量账号，实现针对 “一人 / 一物 / 一文 / 一视频” 的大量点赞，即刷别人的分；
    
2.  通过登录一账号，实现针对 “多人” 的批量点赞，即刷自己的分。
    

代码层级的实现
-------

基本逻辑梳理清楚之后，就可以进入实际的编码环节。

### 模拟登录

在登录实现上，存在两种思路：

1.  大量注册（也可购买）账号，通过 Python 程序切换账号，每次登录点赞之后，切换下一账号；
    
2.  提前通过技术或人工手段，模拟登录，记录账号登录后产生的 Cookie，后续维护 Cookie 池实现操作逻辑。
    

思路二存在的问题为 Cookie 有效期问题，如网站无此限制，建议采用该方式，效率更高。

伪代码实现

```
# 思路一
with open("users.txt","r") as f:
 user_pass = f.readline()
 # 模拟登录
 login(user_pass)
 # 完成登录后操作
 do_someting()

# 思路二
with open("cookies.txt","r") as f:
 one_cookie = f.readline()
 # 通过携带 cookie 参数访问接口
 get_detail(one_cookie)

with open("users.txt","r") as f:
 user_pass = f.readline()
 # 模拟登录
 login(user_pass)
 # 完成登录后操作
 do_someting()

# 思路二
with open("cookies.txt","r") as f:
 one_cookie = f.readline()
 # 通过携带 cookie 参数访问接口
 get_detail(one_cookie)

with open("users.txt","r") as f:
 user_pass = f.readline()
 # 模拟登录
 login(user_pass)
 # 完成登录后操作
 do_someting()

# 思路二
with open("cookies.txt","r") as f:
 one_cookie = f.readline()
 # 通过携带 cookie 参数访问接口
 get_detail(one_cookie)
# 思路一
with open("users.txt","r") as f:
 user_pass = f.readline()
 # 模拟登录
 login(user_pass)
 # 完成登录后操作
 do_someting()

# 思路二
with open("cookies.txt","r") as f:
 one_cookie = f.readline()
 # 通过携带 cookie 参数访问接口
 get_detail(one_cookie)

```

其中思路二的 Cookie 池，可以人工或者使用程序创建。

**在模拟登录部分，你将碰到两个学习难点**

1.  验证码识别问题；
    
2.  IP 反爬限制。
    

*   难点一最易上手的解决方案，对接打码平台。
    
*   难点二解决方案，购买 IP 代理池，也可自建代理池，重点看项目成本与对稳定性的要求。
    

### 点赞器

在很多项目中，当你完成了模拟登录操作，已经表示该网站对你 完全开放了。

接下来你要做的就是寻找点赞接口，例如下面的案例（只做参考使用）：

CSDN 点赞接口如下：

```
# POST 传递用户标识与文章 ID
Request URL: https://blog.csdn.net//phoenix/web/v1/article/like
Request Method: POST
# POST 参数如下
articleId=118558076


```

知乎点赞接口如下：

```
# 直接 POST 传递，用户标识在 Cookie 中
Request URL: https://www.zhihu.com/api/v4/zvideos/1391420717800554497/likers
Request Method: POST


```

bilibili 点赞接口如下：

```
# 传递用户标识的同时，传递相应的参数
Request URL: https://api.bilibili.com/x/web-interface/archive/like
Request Method: POST
# POST 参数如下
aid: 631588341
like: 1
csrf: b39b26b6b8071e2f908de715c266cb59


```

通过上述几个案例，你会发现，点赞操作接口格式基本类似，都是通过 POST 传递 Cookie 与特定参数到服务器中。

其中 B 站的特殊一些，携带了一个 csrf 参数，该参数可以从 Cookie 中直接提取。

伪代码实现

```
import requests

def like(params):
 # 请求头中获取 Cookie 由模拟登录获取
 cookie = get_cookie()
 # cookie = login()
 headers = {
  "其它属性":"属性值",
  "Cookie":cookie # 重点包含用户标识 Cookie
 }
 res = requests.post("地址","参数","请求头")

```

**在调用点赞接口部分，你将碰到一个学习难点**

*   接口中包含位置参数，例如上述的 B 站点赞链接中的 csrf，碰到未知参数的解决思路参考下述描述。
    

继续拿 B 站举例，打开浏览器开发者工具，切换到 network 选项卡，当点击点赞的时候，会出现点赞的数据请求，如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_png/ULibHgXIt3jx6gnJENYZfFTWXWfg8ibTOxgKELkiaElH2mDh79zD0lRg0NFqGibUWGNAiaPV0nmNMCorKtAEugpmvHw/640?wx_fmt=png)

该请求同时出现了 POST 的相关参数，接下来，你只需要按下键盘的 Ctrl+F，打开搜索窗口（就是在当前开发者工具的 network 选项卡中打开），在搜索框中，输入要检索的值，即可找到该值所出现的所有请求位置，然后再进行后续分析即可。重点要找到该参数值产生的位置与原理。

![](https://mmbiz.qpic.cn/mmbiz_png/ULibHgXIt3jx6gnJENYZfFTWXWfg8ibTOxKuI1YGVP2shfjtnGtSqUjibP3MwkicFvs8zb486dge3KaCZGnicfTuA9w/640?wx_fmt=png)

点赞机器人总结
-------

自动点赞机器人存在多样的应用场景，准确的说，该操作会造成某些平台的失衡，也会影响平台数据的公平性，但正是因为有需求，所以市场上现在存在大量的点赞器，刷分器，评论器，甚至存在大量的公司去经营此类业务。

我们不支持该类业务，但可以学习它的实现原理。毕竟使用 Python 实现一款自动化工具，了解原理之后，将变得非常简单。

希望本文能让你，实现一款属于你自己的小众刷分机器。
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIzMzMzOTI3Nw==&mid=2247500574&idx=1&sn=728e24719ac43007af568514113f688c&chksm=e885a3fcdff22aeadb6a06d0846547d7bf14a17ec6097f98e4052ea105c07d87119b36a621a5&mpshare=1&scene=1&srcid=0507jHZ2Xtzbver29jpQ9CVb&sharer_sharetime=1620325337077&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

公众号

Shodan 在百度百科里被给出了这么一句话介绍：Shodan 是互联网上最可怕的搜索引擎。  

为什么呢？与谷歌、百度等搜索引擎爬取网页信息不同，Shodan 爬取的是互联网上所有设备的 IP 地址及其端口号。

而随着智能家电的普及，家家户户都有许多电器连接到互联网，这些设备存在被入侵的可能性，这是十分危险的。

  
你会搜素到这个品牌的摄像头设备遍及全球的 IP 及其暴露的端口号：  

可以看到，这台机器暴露了 17、80、111、995、3128、5000、6000、20547 端口，黑客可以根据这些端口进行针对性的攻击。

不过也不需要过于担心，如果你的服务不存在漏洞，一般是无法攻入的。但有些端口号会暴露摄像头的 web 管理端，如下：

![](https://mmbiz.qpic.cn/mmbiz_png/h6NqozYcCQ6pGb1DIl1Kn2ZS0rEribiboRJf1T0cyaeJefRVKcFD8ZEmXZNMXNB5Itd2Nj2jAGqAU6LWarhk7D4Q/640?wx_fmt=png)  

那么黑客可能可以用暴力破解的方式，强行进入摄像头后台管理端，获取到实时的录像。

谨记这会侵犯别人的隐私权，是违法的行为，我们是遵纪守法的好公民所以知道它的原理和危害就足够。我们的目的是运用技术保护好个人隐私，如非必要不将摄像头接入互联网，一定要接入的话，不能使用容易被破解的弱口令。

Shodan Web 端非常好用，但如果我们有从 Python 搜索的需求怎么办？

没关系，Shodan 官方也提供了 Python SDK 包，下面就来讲讲这个 SDK 包的使用。

_**1. 准备**_

开始之前，你要确保 Python 和 pip 已经成功安装在电脑上，如果没有，可以访问这篇文章：[超详细 Python 安装指南](http://mp.weixin.qq.com/s?__biz=MzI3MzM0ODU4Mg==&mid=2247485004&idx=1&sn=6f89120cf926e71c7eb4788744ff625f&chksm=eb25e4c5dc526dd31f216f56b963179a0bc301a5654644ef98f436aa4740caa6f2774046296f&scene=21#wechat_redirect) 进行安装。

(可选 1) 如果你用 Python 的目的是数据分析，可以直接安装 Anaconda：[Python 数据分析与挖掘好帮手—Anaconda](http://mp.weixin.qq.com/s?__biz=MzI3MzM0ODU4Mg==&mid=2247486014&idx=1&sn=4519422fbd83b5feffcbb21552226bc3&chksm=eb25e8b7dc5261a1aef2fa400ca7bcaa8c06394ea1f9a5860ab02bcf95d4664f41903b12bbd8&scene=21#wechat_redirect)，它内置了 Python 和 pip .

(可选 2) 此外，推荐大家用 VSCode 编辑器，它有许多的优点：[Python 编程的最好搭档—VSCode 详细指南](http://mp.weixin.qq.com/s?__biz=MzI3MzM0ODU4Mg==&mid=2247485849&idx=1&sn=ec098cf67a55bd1d61d4513397434c94&chksm=eb25eb10dc52620682db716d206c18b00bd53c01743729a9dea381e1791566a04a06f1fabca5&scene=21#wechat_redirect)。

**请选择以下任一种方式输入命令安装依赖**：  
1. Windows 环境 打开 Cmd (开始 - 运行 - CMD)。  
2. MacOS 环境 打开 Terminal (command + 空格输入 Terminal)。  
3. 如果你用的是 VSCode 编辑器 或 Pycharm，可以直接使用界面下方的 Terminal.

```
pip install shodan

```

使用 Shodan 必须注册账号，注册网址：https://account.shodan.io/register

![](https://mmbiz.qpic.cn/mmbiz_png/h6NqozYcCQ6pGb1DIl1Kn2ZS0rEribiboRzD22Dx80M6VHUT8hFtdJtIeaLhqibicfGQ26DO4cTIWpBGLHslhGNnFg/640?wx_fmt=png)

  
输入完相关信息，点击 CREATE 会跳转到个人账户页：  

![](https://mmbiz.qpic.cn/mmbiz_png/h6NqozYcCQ6pGb1DIl1Kn2ZS0rEribiboRCd6t0tVsBNXY8U3EF754QKGXMIBdkSKY35ZlOtF1CDXV6norABkrIg/640?wx_fmt=png)

此时 API Key 会显示你的 API 秘钥，请记录这个秘钥，后续会使用到这个秘钥去请求接口。

_**3.Shodan 基本调用**_

Shodan 本质上就是一个搜索引擎，你只需要输入搜索的关键词：  

```
# 公众号：Python 实用宝典
# 2021-05-04
from shodan import Shodan

api = Shodan('你的API KEY')

def search_shodan(keyword):
    # 调用搜索接口
    result = api.search(keyword)

    # 显示所有IP
    for service in result['matches']:
            print(service['ip_str'])

search_shodan("Hikvision-Webs")

```

结果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/h6NqozYcCQ6pGb1DIl1Kn2ZS0rEribiboRuSNw4LCQqoRt8icpgBChXrBQTXNEUxwtPsyF36FKETfzr5hJ5F3eo4A/640?wx_fmt=png)

可惜的是，普通 API 只能像这样搜索关键字，无法使用过滤条件如： **`Hikvision-Webs country:"US"`** 搜索美国境内的所有 Hikvision 网站管理端。

如果你想要使用过滤条件，Shodan 需要你升级 API 权限：

![](https://mmbiz.qpic.cn/mmbiz_png/h6NqozYcCQ6pGb1DIl1Kn2ZS0rEribiboRjTm7EVsjHPJuL23ffNImO7Y0vcSEicIAj4jkvEnNqfIPwlMLLJJzM7w/640?wx_fmt=png)

挺贵的，不过还好是一次性支付，永久使用。

_**4. Shodan 高级使用**_

Shodan 的用处当然不仅仅是在黑客攻防中，它还能用于统计。如果你想要了解哪些国家的使用这款摄像头的数量最多，可以使用 Facets 特性。

```
# 公众号：Python 实用宝典
# 2021-05-04
from shodan import Shodan

api = Shodan('你的API KEY')
def try_facets(query):
    FACETS = [
        'org',
        'domain',
        'port',
        'asn',
        ('country', 3),
    ]

    FACET_TITLES = {
        'org': 'Top 5 Organizations',
        'domain': 'Top 5 Domains',
        'port': 'Top 5 Ports',
        'asn': 'Top 5 Autonomous Systems',
        'country': 'Top 3 Countries',
    }

    try:
        # 使用 count() 方法可以不需要升级API，且比 search 方法更快。
        result = api.count(query, facets=FACETS)

        print('Shodan Summary Information')
        print('Query: %s' % query)
        print('Total Results: %s\n' % result['total'])

        # 显示每个要素的摘要
        for facet in result['facets']:
            print(FACET_TITLES[facet])

            for term in result['facets'][facet]:
                print('%s: %s' % (term['value'], term['count']))

    except Exception as e:
        print('Error: %s' % e)

try_facets("Hikvision-Webs")

```

得到结果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/h6NqozYcCQ6pGb1DIl1Kn2ZS0rEribiboRey52bgcldgGQXQoAVictUp0dJiaFpdic6wsXwGkFZJia73Mp0eibV3WRKeA/640?wx_fmt=png)

从 Top 3 Countries 中可以看到，这款摄像头使用数量排名前三的国家分别是：美国、日本和德国。

没想到吧，Shodan 居然还能用于产品分析。同样地原理，如果你把关键词改为 **`apache`** ，你可以知道目前哪些国家使用 apache 服务器数量最多，最普遍被使用的版本号是什么。

简而言之，Shodan 是一个非常强大的搜索引擎，它在好人手里，能被发挥出巨大的潜能。如果 Shodan 落入坏人之手的话，那真是一个可怕的东西。

为了避免受到不必要的攻击，请大家及时检查所有联网设备的管理端的密码，如果有使用默认密码及弱口令，立即进行密码的更改，以保证服务的安全。

本文所有源代码可在公众号后台回复：**shodan** 下载。

![](https://mmbiz.qpic.cn/mmbiz_jpg/WLXa0NsmXuiap5yprf7DJXhpdhC0XIBAopbpFTUe1eSSuGbT5Kg63CPBicfpxwLAFIm2wPkicB5NWdSicbzziaibPXSA/640?wx_fmt=jpeg)

推荐阅读  点击标题可跳转

[14 个 Python"冷兵器" 合集，让你的终端一秒开挂](http://mp.weixin.qq.com/s?__biz=MzIzMzMzOTI3Nw==&mid=2247500488&idx=1&sn=88f3f3c0ddf0fdc0f1c80449e2ab9cab&chksm=e885a22adff22b3c1c7486ec177b72be487e149ade57839877b445043d453a90a8a7f8fe9fed&scene=21#wechat_redirect)  

[4000 字，一篇数据可视化 "保姆级" 攻略](http://mp.weixin.qq.com/s?__biz=MzIzMzMzOTI3Nw==&mid=2247500475&idx=1&sn=0e1f66538313c5f090e59149ed3309a6&chksm=e885a259dff22b4fffab3ea92a2156aa33bb543a502ee854eed9604d7a98f5ae62528d53937c&scene=21#wechat_redirect)  

[使用 MitmProxy 玩爬虫的，这篇文章别错过了](http://mp.weixin.qq.com/s?__biz=MzIzMzMzOTI3Nw==&mid=2247500430&idx=1&sn=c545d4643e298ddc254aac6e5994dd76&chksm=e885a26cdff22b7a1eb45955f6b4b4925759e088bbec84db5f17c18d4be733701c47e982ed26&scene=21#wechat_redirect)  

[一个好用的 Python 自带模块：优先级调度器](http://mp.weixin.qq.com/s?__biz=MzIzMzMzOTI3Nw==&mid=2247500406&idx=2&sn=944f82553901a4239d107679cd176733&chksm=e885a294dff22b821889605e3258acc71d6b882d0af18a7c3580175075af4d576724f4a7ad04&scene=21#wechat_redirect)  

[谷歌发布新编程语言，专治 SQL 各种 “不服”](http://mp.weixin.qq.com/s?__biz=MzIzMzMzOTI3Nw==&mid=2247500394&idx=1&sn=1df92b231b3ba1829cae1fb87478b900&chksm=e885a288dff22b9e5bd0387e961375882c61fd32e84adad2cb242c8e4d670fc68c2124d01cc4&scene=21#wechat_redirect)  

**如果对你有帮助。**

**请不吝****点赞****，点****在看，****谢谢**
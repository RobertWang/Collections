> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU3NDgxMzI0Mw==&mid=2247498166&idx=2&sn=29b2f1f863d3543ee711a935c130c8ad&chksm=fd2e1ce2ca5995f4dcb81937f0ca3f90b3a52d60ed6a29c353e1476c61aa29a5d99295d95934&mpshare=1&scene=1&srcid=0625aFUbfsycwFADnPqgrjbW&sharer_sharetime=1624562291945&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

```
点击上方“程序员大白”，选择“星标”公众号

重磅干货，第一时间送达

```

##### 晓查 发自 凹非寺   
量子位 报道 | 公众号 QbitAI

当输入这样一句命令后：

```
pip install openvc

```

最近，安全公司 Sonatype 发现，很多恶意软件都伪装成常见的 PyPI 包，往往只差几个字母。

随着加密货币的火爆，黑客们开始把挖矿软件植入其中。如果用户手打 pip 安装命令手滑一下，自己的电脑就可能变成 “矿机”。

PyPI 里的挖矿软件
-----------

常用的绘图工具包 **matplotlib** 首当其冲。PyPI 今年有多个与之类似的恶意软件包，如 **mplatlib**、**maratlib**（记住这个软件包名称）等等。

这类 “李鬼” 共有 7 种之多，都是一个叫做 **nedog123** 的用户上传到 PyPI。其中像 maratlib 还是今年 4 月份发布的。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBicribDoRYoF2DVhqjlfxQa4zBkUjA9mWtGcMHHWyYL9G5ia61heqWRrwPiaAG58UDfdYWzeS3y8O5SQ/640?wx_fmt=png)

这一组恶意软件以 “maratlib” 为核心，其他软件都是把它作为依赖项，比如 “learninglib” 就是这种情况：

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBicribDoRYoF2DVhqjlfxQa4x8X3Pia2tAdvEPmxX6tXMuqDD6QkIVOE0EwPQLm8jv9dR1ibDMmSBsOw/640?wx_fmt=png)

这代码还算是比较 “直白” 的，有些恶意软件将依赖项稍微隐藏了一下，比如“mplatlib”：

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBicribDoRYoF2DVhqjlfxQa4InPjjHUg8WLVGPSmgnQGBxwZkoOTGtLCO7XT2BygSWibeOAE6kByM3w/640?wx_fmt=png)

它把依赖项伪装成 “LKEK”，从第 47 行代码可以看出 LKEK 就是 maratlib。

接着 Sonatype 的安全工程师又对 maratlib 1.0 安装包进行了分析，发现它已经伪装得很深了，使用一般工具已经很难分析这些代码里到底藏了什么。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBicribDoRYoF2DVhqjlfxQa4TgSYuQzdDnEFpyjU8KlZx7QjEmJ2odNLIGXspY6XGQANRjH1KTLgNw/640?wx_fmt=png)

他只好把版本倒回 0.6，这个版本的 maratlib 没有对代码做伪装，它会从 GitHub 下载和运行 Bash 脚本代码：

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBicribDoRYoF2DVhqjlfxQa43UIgSUFdTPuPp2iaSaCyibw9c52IiaaiaepglHcaZEBqHwzYTfibb7hRqIg/640?wx_fmt=png)

但服务 bash 脚本的网址抛出 404 错误，说明这个地址已经被 GitHub 删除, 或者被黑客 nedog123 废弃不用。

经过更深的挖掘，这名安全工程师发现，黑客将代码迁移到了 “Marat Nedogimov” 和“maratoff”用户名下。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBicribDoRYoF2DVhqjlfxQa4f78byQXEm6R09hypbvLX32V8ZdHlT7BYbG5oVDa3PxwF3ibdLKTIavA/640?wx_fmt=png)

这个 aza2.sh 脚本会下载一个名为 “Ubqminer” 的挖矿软件，而上图中那一长串字符就是黑客的数字钱包地址。

至此，案件已经告破。好消息是，PyPI 已经删除了这些恶意软件包。

但是，据 Sonatype 公司统计，这 7 个李鬼软件已经总共被下载超过 **5000 次**。

恶意 PyPI 包防不胜防
-------------

这次发现的 maratlib，可能只是 PyPI 恶意软件包的冰山一角。PyPI 包管理工具的问题一直为用户所诟病。

今年 2 月，[有人将 CUDA 加速包 CuPy 换成了恶意软件](http://mp.weixin.qq.com/s?__biz=MzIzNjc1NzUzMw==&mid=2247570215&idx=5&sn=07f34beb81c7ae6680f30775bd773cd7&chksm=e8d16455dfa6ed43872e1ced1697adfade87720638d557a158b7cc29fbd467fdc28b362e01ad&scene=21#wechat_redirect)。还有一位白帽黑客发现，[只要向公共库上传 PyPI 软件包，就能轻易替换掉私有化的同名软件包](http://mp.weixin.qq.com/s?__biz=MzIzNjc1NzUzMw==&mid=2247568147&idx=1&sn=8a24265aa48b743ae81ab58b8dfab3d7&chksm=e8d17c61dfa6f577c8eea271c0efb4b9747e866e98fb7a8d092e04143f09e11cb8b517d7786f&scene=21#wechat_redirect)，大大增加了科技公司中毒风险。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBicribDoRYoF2DVhqjlfxQa4aBHricFic67IHTrAGdSsLExVxcEjmcMCpNYWzlBrb2YoYWAqbmLRFk1A/640?wx_fmt=png)

###### **△** 每月 PyPI 恶意软件包数量

早在 2016 年，就有人用相似名称的方法发布 PyPI 恶意软件包，骗过了 1.7 万名程序员，导致这个恶意程序被运行了 4.5 万次，甚至连美国军方都中招了。

使用 pip 请谨慎
----------

那么，我们如何预防被安装恶意的 PyPI 软件包？

你以为只要认真检查安装命令就行了？No！

由于 PyPI 绝大部分软件包都是第三方编写和维护的，这体现了开源的优势，但也埋下了审核不严的危险种子。

如今，很多软件都需要安装依赖项，个人不可能一一检查，甚至大公司也做不到。有时候一个软件里写了上百个依赖项，根本没法审查代码。

最好的办法就是监控 setup.py 的行为，在安装不太放心的软件包时，可以在容器中通过 pip 安装包，同时收集系统调用和网络流量，来分析其是否有恶意行为。

最后再提醒一下大家，不仅 pip 命令有风险，使用 npm、gem 等软件包安装命令也可能中毒，一定要对来源不明的软件包仔细核查。

参考链接：  
[1]https://blog.sonatype.com/sonatype-catches-new-pypi-cryptomining-malware-via-automated-detection  
[2]https://arstechnica.com/gadgets/2021/06/counterfeit-pypi-packages-with-5000-downloads-installed-cryptominers/  
[3]https://www.freebuf.com/articles/web/254820.html  
[4]https://github.com/rsc-dev/pypi_malware
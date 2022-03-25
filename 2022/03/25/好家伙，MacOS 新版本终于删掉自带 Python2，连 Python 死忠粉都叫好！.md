> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/hfXiMYtTR3H4eINqDUDcdg)

  

喜大普奔，苹果现已正式推出的 macOS 12.3 版本，终于把自带的 Python 2 给删了！  

此前测试版推出时，就有网友激动地表示：

> “
> 
> 终于！虽然我是 Python 的死忠粉，但我真的希望操作系统们不要再内置Python了！！！
> 
> ”

![图片](https://mmbiz.qpic.cn/mmbiz_png/DAE6TYB3GW9H68nOU0YhdwudCVIUrPxGjHAM7HDr8OZnnpvs5sngibG8eKic3IfyBHS1TLp3AibcJcGTVhcziaOIiaw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

> “
> 
> 操作系统捆绑编程语言是缺点而非优点。
> 
> ”

![图片](https://mmbiz.qpic.cn/mmbiz_png/DAE6TYB3GW9H68nOU0YhdwudCVIUrPxGnjS0IbibVMo3Wciaa8Xa09XvSsl6PuZIM709L1vPASOf9Osjlf5A5oiaQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

毕竟，一顿操作猛如虎，最后因为 Python 版本混乱代码跑不起来的，应该不止我一个……

![图片](https://mmbiz.qpic.cn/mmbiz_png/DAE6TYB3GW9H68nOU0YhdwudCVIUrPxGov7gZsHOFSbPMlZ1eM8miaibtyVGTNcd6Ilx4j7bTCibWnicI9FssYtqcQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

具体的更新是酱婶的：

![图片](https://mmbiz.qpic.cn/mmbiz_png/DAE6TYB3GW9H68nOU0YhdwudCVIUrPxGop7HresAy0icicwDtWP2YicdPN9s98OJTW0A0Pfr7TqJa5UMvAgatXjYA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

苹果表示，在此次更新中，原本内置安装的 Python 2.7 会被移除，并建议开发者们使用 Python 3 或者其他编程语言。

此前，苹果曾解释称，在系统内保留 2020 年官方就已停止更新维护的 Python 2，是为了保证旧版软件的兼容性。

需要注意的是，macOS Monterey 12.3 并没有预装 Python 3。

天下苦 Python 环境混乱久矣
-----------------

> “
> 
> 人生苦短，我用Python。
> 
> ”

![图片](https://mmbiz.qpic.cn/mmbiz_png/DAE6TYB3GW9H68nOU0YhdwudCVIUrPxGEz8V1EYhMQeibTwAAiaN2lctFdOGB6gWTsaFUZ17j2n7eHbFyA4lrthg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

Python 因其简单易用、学习成本低而风靡全世界。

但优点突出，槽点也着实不少。

除了执行速度慢、Python 2 和 3 不兼容这样的问题，其开发环境之混乱也常常为人所诟病。

看另一张著名的梗图就大概能感受到开发者们的痛苦了……

![图片](https://mmbiz.qpic.cn/mmbiz_png/DAE6TYB3GW9H68nOU0YhdwudCVIUrPxGogib3KKPdzJxWmKvBAKgH8jTxnicpeR5ZkYZMSo3AQq1pbGhRCE0wCQQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

事实上，为了解决这个问题，程序员们也没少努力。

比如 pipenv，就是专门用来简化 Python 开发环境设置的工具。

具体而言，pipenv具有以下特性：

*   集成pip和virtualenv两者的功能；
    
*   使用Pipfile和Pipfile.lock来替代requirement.txt，更容易搞清依赖关系；
    
*   可以在开发环境中使用多个Python版本；
    
*   广泛使用哈希校验，能自动暴露安全漏洞；
    
*   可通过自动加载.env读取环境变量，简化开发流程。
    

virtualenv、venv、poetry、conda……这些 Python 环境管理工具也都在程序员群体中流行。

不过，也有程序员吐槽，一次又一次的重复造轮子本身也是一种灾难……

> “
> 
> Python社区一次一次又一次地重复造轮子，distutils、setuptools、pip、pipenv、tox、flit、conda、poetry、virtualenv、requirements.txt、setup.py、setup.cfg、pyproject.toml……需要处理的麻烦事儿简直列不完。这是一场灾难。
> 
> ”

![图片](https://mmbiz.qpic.cn/mmbiz_png/DAE6TYB3GW9H68nOU0YhdwudCVIUrPxGxKDoqNxXFE6T7UtrPeeELRFpXCWthqgTd9umkFwEhymyd1Via3kXSAQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

那么，你被 Python 的“混乱”困扰过吗？

参考链接：

[1]https://developer.apple.com/documentation/macos-release-notes/macos-12_3-release-notes#Python

[2]https://news.ycombinator.com/item?id=30115214

> 来源：量子位
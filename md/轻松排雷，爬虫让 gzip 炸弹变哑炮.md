> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/WmnZlkawRSpMvChFFcCC0A)

在之前的文章《[反爬虫的极致手段，几行代码直接炸了爬虫服务器](http://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652591583&idx=1&sn=36fe4fc6ef6de94f8ae655be675bcdc9&chksm=84657d95b312f483f531db86e0633ae08a6fc35c5547f18f51aa5c212044b5dec9eb6b648336&scene=21#wechat_redirect)》中，讲到了后端如何使用 gzip 返回极高压缩率的文件，从而瞬间卡死爬虫。

![](https://mmbiz.qpic.cn/mmbiz_png/ohoo1dCmvqcmDWLgF2nUel83T2uHj5tcCSIkKxSx43ZDjyKuPXppeOWV5ibdccO6MLqYkx53YVicLYxwIE0d5ibgQ/640?wx_fmt=png)

你只需要判断`resp.headers`中，是否有一个名为`content-encoding`，值包含`gzip`或`deflate`的字段。如果没有这个字段，或者值不含`gzip`、`deflate`那么你就可以放心，它大概率不是炸弹。

值得一提的是，当你不读取`resp.content`、`resp.text`的时候，Requests 是不会擅自给你解压缩的，如下图所示。因此你可以放心查看 Headers。：

![](https://mmbiz.qpic.cn/mmbiz_png/ohoo1dCmvqcmDWLgF2nUel83T2uHj5tcqw8qaAgee6MQfD2R964H3IROKINWDic1mY22wZCtEQiajfu5DuPQkwrA/640?wx_fmt=png)

那么，如果你发现网站返回的内容确实是`gzip`压缩后的内容了怎么办呢？这个时候，我们如何做到既不解压缩，又能获取到解压以后的大小？

如果你本地检查一个`.gz`文件，那么你可以使用命令`gzip -l xxx.gz`来查看它的头信息：

![](https://mmbiz.qpic.cn/mmbiz_png/ohoo1dCmvqcmDWLgF2nUel83T2uHj5tc8Hmwicle0OjwMMialvAcRFqOJRGcPJWjibdSvpFVDu9icylfjSd4HtibD7g/640?wx_fmt=png)

打印出来的数据中，第一个数字是压缩后的大小，第二个数字是解压以后的大小，第三个百分比是压缩率。这些信息是储存在压缩文件的头部信息中的，不用解压就能获取到。

那么当我使用 Requests 的时候，如何获得压缩后的二进制数据，防止它擅自解压缩？方法其实非常简单：

运行效果如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/ohoo1dCmvqcmDWLgF2nUel83T2uHj5tc684U0vdWv83FDc5xk9EjZ8QRynyMjcC5zZsjNHsyTPmtBu9RZ2PJmA/640?wx_fmt=png)

此时可以看到，这个大小是压缩后的二进制数据的大小。现在，我们可以使用如下代码，在不解压的情况下，查询到解压缩后的文件大小：

```
import gzip
import io
import requests
resp = requests.get(url, stream=True)

decompressed = resp.raw.read()
with gzip.open(io.BytesIO(decompressed), 'rb') as g:
    g.seek(0, 2)
    origin_size = g.tell()
    print(origin_size)


```

运行效果如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/ohoo1dCmvqcmDWLgF2nUel83T2uHj5tcQ6x2mza9AI8pA54DhgnP8NkpXdrPiaMMwIiaGLJ0xQ776l84WRnmnrmA/640?wx_fmt=png)

打印出来的数字转成 MB 就是 10MB，也就是我们昨天测试的解压后的文件大小。

- EOF -
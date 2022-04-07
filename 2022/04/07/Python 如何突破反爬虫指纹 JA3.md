> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652579773&idx=2&sn=4e516f1ab29eba941139035eeed2fa2b&chksm=84650ff7b31286e1104dcc550b1733151c5da119d6a79fc279d35b3fe952d14af2e37833c7ad&scene=21#wechat_redirect)

在前几天的文章[《为什么随机 IP、随机 UA 也逃不掉被反爬虫的命运》](http://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652579729&idx=1&sn=9795d274d17f96faa3f45ce4a70742da&chksm=84650fdbb31286cd8614fd4e63639205d92dab3786990bdec2ed08120650cb95f2c274758046&scene=21#wechat_redirect)里面，我介绍了 JA3 指纹算法。这个算法可以在你改掉 IP 和 UA 的情况下依然识别到你。

今天，我们来介绍如何在 Python 里面，使用 requests 请求网站的时候，修改 JA3 指纹。

requests 是基于 urllib3 实现的。要修改 JA3 相关的底层参数，所以我们今天要修改 urllib3 里面的东西。

我们知道 JA3 指纹里面，很大的一块就是 Cipher Suits，也就是加密算法。而 requests 里面默认的加密算法如下：

`ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+HIGH:DH+HIGH:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+HIGH:RSA+3DES:!aNULL:!eNULL:!MD5`

冒号分割了不同的加密算法。这些加密算法每一种顺序其实就对应了一个 JA3 指纹字符串，只要我们修改这个顺序，就能得到不同的 JA3 字符串。

在 requests 里面，要修改 Cipher Suits 中的加密算法，需要修改 urllib3 里面的 ssl  上下文，并实现一个新的 HTTP 适配器 (HTTPAdapter)。在这个适配器里面，我们在每次请求的时候，随机更换加密算法。但需要注意的是`!aNULL:!eNULL:!MD5`就不用修改了，让他们保持在最后。

涉及到的代码如下：

```
from requests.adapters import HTTPAdapter
from requests.packages.urllib3.util.ssl_ import create_urllib3_context

ORIGIN_CIPHERS = ('ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+HIGH:'
'DH+HIGH:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+HIGH:RSA+3DES')


class DESAdapter(HTTPAdapter):
    def __init__(self, *args, **kwargs):
        """
        A TransportAdapter that re-enables 3DES support in Requests.
        """
        CIPHERS = ORIGIN_CIPHERS.split(':')
        random.shuffle(CIPHERS)
        CIPHERS = ':'.join(CIPHERS)
        self.CIPHERS = CIPHERS + ':!aNULL:!eNULL:!MD5'
        super().__init__(*args, **kwargs)
        
        
    def init_poolmanager(self, *args, **kwargs):
        context = create_urllib3_context(ciphers=self.CIPHERS)
        kwargs['ssl_context'] = context
        return super(DESAdapter, self).init_poolmanager(*args, **kwargs)

    def proxy_manager_for(self, *args, **kwargs):
        context = create_urllib3_context(ciphers=self.CIPHERS)
        kwargs['ssl_context'] = context
        return super(DESAdapter, self).proxy_manager_for(*args, **kwargs)


```

在一般情况下，当我们实现一个子类的时候，`__init__`的第一行应该是`super().__init__(*args, **kwargs)`，但是由于`init_poolmanager`和`proxy_manager_for`是复写了父类的两个方法，这两个方法是在执行`super().__init__(*args, **kwargs)`的时候就执行的。所以，我们随机设置 Cipher Suits 的时候，需要放在`super().__init__(*args, **kwargs)`的前面。

有了适配器以后，我们使用 requests 时，初始化一个 Session，然后把这个适配器绑定到特定的网站上：

```
import requests
headers = {'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.131 Safari/537.36 Edg/92.0.902.67'}
s = requests.Session()
s.headers.update(headers)

for _ in range(5):
    s.mount('https://ja3er.com', DESAdapter())
    resp = s.get('https://ja3er.com/json').json()
    print(resp)


```

其中，`s.mount`的第一个参数表示这个适配器只在`https://ja3er.com`开头的网址中生效。

运行效果如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/ohoo1dCmvqcIfY8w5O3ia6dDxYX5MbianbtonWMHjqnPkM8VVORvQCFl4b5O95KXPFUw6KlibFSM4ZpXYIKyDNQwQ/640?wx_fmt=png)

可以看到，`ja3_hash`已经改变了，说明我们请求时的 JA3 指纹已经发生了改变。

- EOF -

推荐阅读  点击标题可跳转

1、[什么神器，竟然能自动检索、修复 Python 代码 bug？](http://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652579700&idx=2&sn=f4fc0323bc90840d09fc295706bd699d&chksm=84650f3eb3128628fd935f22585a2849ba46b826eff1ed68d9ca9e646d7f40b1b743c2f550d0&scene=21#wechat_redirect)

2、[悄悄学习 Doris，偷偷惊艳所有人 ：Apache Doris 四万字小总结](http://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652579693&idx=2&sn=c11e2fd170600d9255026682d3e73704&chksm=84650f27b31286312fd6219a2fd6ca0861efb5f38b0a7b64143fac8c24755783f9d2712c3e05&scene=21#wechat_redirect)

3、[数据分析利器之 Excel 函数篇](http://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652579682&idx=2&sn=b20c5fa6a17bb09d4f42a2874408c239&chksm=84650f28b312863ea3cfeebbebbee03372136d10183e2f178e42fc4b4df12d2a806ed76d53ff&scene=21#wechat_redirect)
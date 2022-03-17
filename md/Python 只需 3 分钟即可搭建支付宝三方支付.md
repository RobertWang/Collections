> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247512998&idx=2&sn=d82f383ff6a62f6ad0f14e67c4c2ddf7&chksm=e969edabde1e64bddd6857f271e81ba2a7abfedece9d5ab15a72263b6571f2a7080f7808e584&mpshare=1&scene=1&srcid=0315d7BHD6k5qqYTP3OTJxbg&sharer_sharetime=1647323373407&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> 编辑：乐乐 | 来自：blog.csdn.net/weixin_45394086/article/details/121843483

[Pythn 人工智能技术 (ID:coder_experience) 第 767 期推文](http://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247489525&idx=1&sn=26e9ce9d935a845d03ce4a3d30ebc975&chksm=e96a49f8de1dc0eeda78a0f1fa1ab74bfa13222fecbb924f52e59b59cd8b431d6e0fd00cc1cb&scene=21#wechat_redirect)

**正文**

**大家好，我是 Python 人工智能技术**

```
为什么使用三方支付？
```

再没有三方支付平台之前，用户发起支付请求的时候，用户要去和银行签约 (转账)，特别的不方便，为了解决这些问题，就有了三方支付，三方平台去完成签约，给用户节省时间。

**支付宝支付的流程**

商户拿到支付宝的公钥、自己的私钥（私钥加密、公钥解密），用私钥请求支付宝，支付宝解密、验签、进行支付处理，支付宝将处理的返回值传给商户，当支付成功后，返还给商户订单号、金额、时间戳等消息，支付失败后同样给商户反馈结果。

**配置流程**

**1、获取 APPID**

> 支付宝开放平台: https://open.alipay.com/

*   登录支付宝开放平台–> 点击控制台
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/PvP6qjUpvIpYI9w8ic5LgaJia11bNKiaPQAQ6UF9ZsZIfGG7p3bkqD0pJ9J8RhcST5S1YnfXgfoib30UdgicrmBXCibw/640?wx_fmt=png)

*   点击沙箱 (复制 APPID)
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/PvP6qjUpvIpYI9w8ic5LgaJia11bNKiaPQAQ7w3ey8xsQ7taQXQsavts1feW3PC7qhZ2YKVvpRna32HOsrpJITNHA/640?wx_fmt=png)

**2、在线生成密钥**

*   点击文档，找到开发助手，点击在线加密。
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/PvP6qjUpvIpYI9w8ic5LgaJia11bNKiaPQAPJtg6xszOuCo1uUeL9EKuM01P7SstzArbLzOmuQXPWNI3xxXQrWpNA/640?wx_fmt=png)

*   获取私钥
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/PvP6qjUpvIpYI9w8ic5LgaJia11bNKiaPQAaXTed1AyIv2NbdSrWeISaribtv1DeA27ER3XhvzQsmiabnlUHHv1X8yQ/640?wx_fmt=png)

**3、获取公钥**

*   点击应用公钥
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/PvP6qjUpvIpYI9w8ic5LgaJia11bNKiaPQAaoribgTv4wSicTZcEVztxic0fBE6ibvXEiaa3giat57lK21Evc6ia8IqXbiaOA/640?wx_fmt=png)

*   生成公钥
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/PvP6qjUpvIpYI9w8ic5LgaJia11bNKiaPQAocMuVBgO5209FjnIVJcoIJfCKAn4FxsLyeQMnT6XqbIpPvztBZJ8qg/640?wx_fmt=png)

现在已经拿到了需要的公钥。另外，搜索公众号顶级架构师后台回复 “面试”，获取一份惊喜礼包。

**python 项目中集成支付宝**

*   构建支付类
    

```
from datetime import datetime
from Crypto.PublicKey import RSA
from Crypto.Signature import PKCS1_v1_5
from Crypto.Hash import SHA256
from urllib.parse import quote_plus
from base64 import decodebytes, encodebytes
import json
class AliPay:
    """
    支付宝支付接口(PC端支付接口)
    """
    def __init__(self, appid, app_notify_url, app_private_key_path,
                 alipay_public_key_path, return_url, debug=False):
        self.appid = appid
        self.app_notify_url = app_notify_url
        self.app_private_key_path = app_private_key_path
        self.app_private_key = None
        self.return_url = return_url
        with open(self.app_private_key_path) as fp:
            self.app_private_key = RSA.importKey(fp.read())
        self.alipay_public_key_path = alipay_public_key_path
        with open(self.alipay_public_key_path) as fp:
            self.alipay_public_key = RSA.importKey(fp.read())
        if debug is True:
            self.__gateway = "https://openapi.alipaydev.com/gateway.do"
        else:
            self.__gateway = "https://openapi.alipay.com/gateway.do"
    def direct_pay(self, subject, out_trade_no, total_amount, return_url=None, **kwargs):
        biz_content = {
            "subject": subject,
            "out_trade_no": out_trade_no,
            "total_amount": total_amount,
            "product_code": "FAST_INSTANT_TRADE_PAY",
        }
        biz_content.update(kwargs)
        data = self.build_body("alipay.trade.page.pay", biz_content, self.return_url)
        return self.sign_data(data)
    def build_body(self, method, biz_content, return_url=None):
        data = {
            "app_id": self.appid,
            "method": method,
            "charset": "utf-8",
            "sign_type": "RSA2",
            "timestamp": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
            "version": "1.0",
            "biz_content": biz_content
        }
        if return_url is not None:
            data["notify_url"] = self.app_notify_url
            data["return_url"] = self.return_url
        return data
    def sign_data(self, data):
        data.pop("sign", None)
        unsigned_items = self.ordered_data(data)
        unsigned_string = "&".join("{0}={1}".format(k, v) for k, v in unsigned_items)
        sign = self.sign(unsigned_string.encode("utf-8"))
        quoted_string = "&".join("{0}={1}".format(k, quote_plus(v)) for k, v in unsigned_items)
        signed_string = quoted_string + "&sign=" + quote_plus(sign)
        return signed_string
    def ordered_data(self, data):
        complex_keys = []
        for key, value in data.items():
            if isinstance(value, dict):
                complex_keys.append(key)
        for key in complex_keys:
            data[key] = json.dumps(data[key], separators=(',', ':'))
        return sorted([(k, v) for k, v in data.items()])
    def sign(self, unsigned_string):
        key = self.app_private_key
        signer = PKCS1_v1_5.new(key)
        signature = signer.sign(SHA256.new(unsigned_string))
        sign = encodebytes(signature).decode("utf8").replace("\n", "")
        return sign
    def _verify(self, raw_content, signature):
        key = self.alipay_public_key
        signer = PKCS1_v1_5.new(key)
        digest = SHA256.new()
        digest.update(raw_content.encode("utf8"))
        if signer.verify(digest, decodebytes(signature.encode("utf8"))):
            return True
        return False
    def verify(self, data, signature):
        if "sign_type" in data:
            data.pop("sign_type")
        unsigned_items = self.ordered_data(data)
        message = "&".join(u"{}={}".format(k, v) for k, v in unsigned_items)
        return self._verify(message, signature)
```

*   实例化类
    

```
def init_alipay():
    # 初始化Alipay
    alipay = AliPay(
        appid="appid",
        app_notify_url="回调地址",
        return_url="回调地址",
        app_private_key_path="私钥相对路径",
        alipay_public_key_path="公钥相对路径",
        debug=True  # 支付环境
    )
    return alipay
```

*   API
    

```
async def get(self):
    alipay = init_alipay()
    # 传一个标题  订单号  订单价格
    params = alipay.direct_pay("三方广告平台", order_no, money)
    url = f"https://openapi.alipaydev.com/gateway.do?{params}"
    return self.write(ret_json(url))
# 构建一个回调地址,用于支付成功后回调,在回调地址中可以获取订单号(out_trade_no)、金额(total_amount)、时间戳(timestamp),然后进行处理业务逻辑。
```

### **总结**

支付包有自己的接口文档，以上是我在 Python 环境下配置的，可以直接使用。

****你还有什****么想要补充的吗？****  

> 免责声明：本文内容来源于网络，文章版权归原作者所有，意在传播相关技术知识 & 行业趋势，供大家学习交流，若涉及作品版权问题，请联系删除或授权事宜。  

--END--

[保姆级别！带你搭建一台服务器！](http://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247510687&idx=2&sn=aad0db260df36d4bfb94b0a28ec92c47&chksm=e969f492de1e7d84dab0d87f6ba9d491a1992c09b91da3e9b41f04aa9e19e6760b50ea7e231a&scene=21#wechat_redirect)

[一个很酷的博客系统](http://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247511873&idx=1&sn=e47d03bd0d618f26876f5e06e625e58a&chksm=e969f14cde1e785af0b15c025418dfc1e7d4d6d09753c859d4a80499122558e054ddce860619&scene=21#wechat_redirect)  

[68 个 Python 内置函数详解](http://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247512721&idx=2&sn=fc70dc0ed8be50d197aa253341a312a8&chksm=e969ec9cde1e658a9e1942d25954882cf7dad2c4377e0e1453711ec94b44ce09f8fbd37ddc2c&scene=21#wechat_redirect)  

[一个悄然成为世界最流行的操作系统](http://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247512900&idx=2&sn=c6e45d76002a3331dac48dc276f41250&chksm=e969ed49de1e645ff38d87a03eefc8c19c0fc826234993aa7c6b3cfe9695ef314ac5e3d2727c&scene=21#wechat_redirect)  

[Python | AioHttp 异步抓取火星图片](http://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247512917&idx=2&sn=2fab61a05e1f77f37e91d169834a5277&chksm=e969ed58de1e644ee363af63f109c298318bfb7851c8658410ee4241e880719c604807b16ea5&scene=21#wechat_redirect)  

[Python 的打包神器——Nuitka](http://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247512942&idx=2&sn=9d6d97430cacddb30431e333171d44bf&chksm=e969ed63de1e64752544fdc41a2458fad4a6ba9fc7e0611bfcfb0c0611f2ed1fa5d24fe94977&scene=21#wechat_redirect)
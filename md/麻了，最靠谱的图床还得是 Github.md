> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU2NTczODU3NA==&mid=2247498945&idx=1&sn=4ae9c23648a0a61641e54f8d18519005&chksm=fcb591e0cbc218f68bd9c75a0dff525412271c463e359761f9488a5bad7006058125ca815d2a&mpshare=1&scene=1&srcid=10267wmDKjFdKueQ5zr6MyUm&sharer_sharetime=1666769558031&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

有段时间没写文，其中一部分原因也是因为图床这件事情很困扰人。

随着 gitee 外链被封了，我之前写的很多文章里的图片都失效了。

后面有空也一直在找合适的图床，中间试了一下七牛云，但是还需要自己整一个域名。

所以就直接放弃了。

之后也看了下 Github，因为我之前一直都是用 Github。

为啥后面没用了呢？

因为 Github 图片显示真的令人头疼。

raw.githubusercontent.com 无法访问，导致所有图片都无法正常显示。

中间也试过网上各种改 host 的方法，不过实在是太低效了。

不过最近忽然想起以前使用的 devSideCar 工具，应该是可以解决这个问题的。

![](https://mmbiz.qpic.cn/mmbiz_png/MsfpRUYbUlrApQsZWn5pW9FIFTic0NA98U3KHpicGb9sdst5KoDYuenJuuXXVams8aDs1DP4lJO6kxkHBrquFWGw/640?wx_fmt=png)

果不其然，上面这张图就是存放在 Github 仓库中的。

所以，目前我使用的最合适的写文解决方案就是 Markdown+devsideCar+Github

其中 Markdown 是我自己搭建的一个编写工具，十分的好用。

https://goodmd.netlify.app/

上面就是我部署的网址，大家可以收藏使用。但是不保证能够一直运行下去，大家可以使用这个开源项目，自己搭建一套。

devsideCar 大家可以直接百度进行下载，关于 devsideCar 就是你的证书一定要按照正确的姿势进行安装。不然就没啥用了。

![](https://mmbiz.qpic.cn/mmbiz_png/MsfpRUYbUlrApQsZWn5pW9FIFTic0NA98PaWiagNib5J0c12od3jDFPf41QLZmY80DUrL8eanEOQIMXqibWGLiaVRxg/640?wx_fmt=png)

想要使用 github 的图床需要一个 Github Token，下面我给大家简单说下具体的步骤。

第一步，你需要创建一个新仓库，注意权限是 public。

第二步，点击右上角头像中的 settings

![](https://mmbiz.qpic.cn/mmbiz_png/MsfpRUYbUlrApQsZWn5pW9FIFTic0NA98BUBIvk1vP1lr99xYvUX48iaPLcKZkXXBeQtK8GmssiaKicdZhPqXIUrhQ/640?wx_fmt=png)

第三步，找到左侧菜单栏，Developer settings

![](https://mmbiz.qpic.cn/mmbiz_png/MsfpRUYbUlrApQsZWn5pW9FIFTic0NA98vLDmYibicwlCqLNlM9py7iaTvQg2mSl8LKnm7MyeJyhSY4Ts90vznSRXQ/640?wx_fmt=png)

第四步，点击 Personal access tokens

![](https://mmbiz.qpic.cn/mmbiz_png/MsfpRUYbUlrApQsZWn5pW9FIFTic0NA98KBEbyhzibudBmeVdVvLGjkDuRAaGSfJmWMCEPiaC9ic99QhxqRI0kNI3w/640?wx_fmt=png)

第五步，点击 Generate new token

![](https://mmbiz.qpic.cn/mmbiz_png/MsfpRUYbUlrApQsZWn5pW9FIFTic0NA98PMdfRtPkJj6kQhJTfIoO3wcHJtQmUCMUqPj3LKIphz2qr84ibA1B88Q/640?wx_fmt=png)

第六步，把图片几个信息填好，然后直接创建即可。

![](https://mmbiz.qpic.cn/mmbiz_png/MsfpRUYbUlrApQsZWn5pW9FIFTic0NA98Y0T99XKiciateXF3ibweVxuPSnPaXrbmMALC2ibVCiaxpIicQI14rAOXRgHg/640?wx_fmt=png)

最后，你就可以拿着这个 token 去玩耍了 。

![](https://mmbiz.qpic.cn/mmbiz_png/MsfpRUYbUlrApQsZWn5pW9FIFTic0NA980WMbD9EGwFTbMHicyvUSDNsic7Oqb6NdhBz2dZ1nyE5DI4mkxnI9aCsg/640?wx_fmt=png)

很多小伙伴可能很多都是在本地来写文章，其中会用到 Typora。

那么怎么给 Typora 配图床呢？

很早之前，我就有写过

[Typora 如何配置 gitee 图床？超详细教程](https://mp.weixin.qq.com/s?__biz=MzU2NTczODU3NA==&mid=2247492884&idx=1&sn=545347c366562e5c2f85e8e16c6080a7&scene=21#wechat_redirect "Typora如何配置gitee图床？超详细教程")

其中解决方案是 Typora+PicGo+Gitee。

这里大家只需要把 Gitee 换成 Github 就行了。

OK，今天的文章就写到这了。

小小的提示：后续公众号可能会改名字
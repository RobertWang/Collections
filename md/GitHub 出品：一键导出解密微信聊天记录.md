> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxOTcxNTIwNQ==&mid=2457957489&idx=2&sn=c12252c40e8e67aa871e1fdb8ed43813&chksm=8cb7119bbbc0988d98c504ec88ea18ff855eef59b06509f9c8c40095c5a7dbc060b4582a6965&mpshare=1&scene=1&srcid=0605hjcmAcNmiORSy3K5ZiZN&sharer_sharetime=1622869901111&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

公众号关注 “GitHubDaily”

设为 “星标”，每天带你逛 GitHub！

![](https://mmbiz.qpic.cn/mmbiz_jpg/uDRkMWLia28iaHLvG19uUuDib1qOzRRUjrxjBlm6IluY9SKZdiaLkuyGEGeBRQwjJR9oATOTz95jVqpvCuhju3gE3w/640?wx_fmt=jpeg)

国内有一款 App 拥有 12.25 亿的月活，你觉得它是什么，可能这个问题还没过脑子呢，咱就能给出答案，微信。

不是说我们太敏感，而是微信一方面在积极努力的完善自己的生态圈，搞了很多看着花里胡哨的新功能上线。

另一方面却又有很多被吐槽了很多年的小功能迟迟没有完善。

所以也难怪会有 5 亿人吐槽，会有 1 亿人想教张小龙做产品。

![](https://mmbiz.qpic.cn/mmbiz_png/ncicWtGoBHtLvUwibse91Mbwa4GwoS3LKiauMxmRlpm8NJuAg19zsD7GzKCNqBuxa5NcNuegfrHJo0bKGVjvopxeQ/640?wx_fmt=png)

不管怎么样吧，有些问题还是要解决的，比如今天就要和大家聊的微信聊天记录的备份。

要知道，油盐不进的微信老早就宣称自己是不存储用户任何聊天记录的。

也就是所有的数据都存储在终端，我们放心，它也安心。

但事实却是，存于终端的聊天记录占据了很大的存储空间不说，稍有不慎误删或碰到其他意外，一切都会成为过往云烟。

把聊天记录备份下来，或许是你某一时刻的刚需（真到后悔那一刻就晚了）。

不过这个问题微信并不是没有注意到，电脑和手机两个终端的联动就可以把聊天记录备份到本地：

![](https://mmbiz.qpic.cn/mmbiz_jpg/ncicWtGoBHtLvUwibse91Mbwa4GwoS3LKiaib8oQGWB4Ij0nZQSPrZub6lE2Q81uZ6GThodErsDehx921SF3WVtItQ/640?wx_fmt=jpeg)

上面两个功能一个是把聊天记录备份到电脑，一个是把聊天记录恢复到手机。

除此之外，你要是想随意访问这个备份，对不起，加密招待。

![](https://mmbiz.qpic.cn/mmbiz_png/ncicWtGoBHtLvUwibse91Mbwa4GwoS3LKiarFfic3ib02RAAA3fodfHCoV7nJEWFpnBFNSQ4FJtCCrhrtep8uwhkwLA/640?wx_fmt=png)

上面那三个加密过数据文件就存储了我们备份的聊天记录。

这样一来二去之间并没有解决占用大量手机空间的问题，反而在查看信息方面带来了不少限制。

其实网上有很多解密的教程，数据库较为固定、加密方式比较简单的安卓端成了技术党的突破口。

不过因为微信对安卓上存储的数据资料限制了权限，你要是非腾讯开发用户，就只能想办法搞到 root 权限。

没有 root 权限的可以使用迂回战术用安卓模拟器达成目标，不过真正的大头还在后面的数据库解密。

我研究半天发现门槛太高，想让小白也能随意备份，随意访问，小丑只会是我自己。

当然，最直接的办法还是去某宝、某鱼搜一下，会发现有钱确实可以为所欲为。

怎么把这个问题完美（白嫖）解决，我发现了一个叫 WechatExporter 的工具。

不过 WechatExporter 这个工具需要一个苹果设备（iPhone 或 iPad）作为前置条件，用安卓的小伙伴先别急，耐心看下去，也会有新发现。

**iTunes 备份**

为啥要搞到一台苹果设备，因为我们需要用到 iTunes 作为桥梁，把聊天记录备份到电脑上。

我猜会有小伙伴会问直接用电脑端的备份行不行？

这个坑我已经帮大家踩过了，WechatExporter 只支持 iTunes 备份的文件格式。

如果大家没有苹果设备，就得先想办法搞上一台，然后在自己手机的微信上通过「我 - 设置 - 通用」找到「聊天记录迁移」的选项。

![](https://mmbiz.qpic.cn/mmbiz_jpg/ncicWtGoBHtLvUwibse91Mbwa4GwoS3LKiaAhWXKiavhQcrAKDSDRUr4HePr8bF4Hpa9jXkyaibY6lbRRKYoLUvS61g/640?wx_fmt=jpeg)

这些准备工作做好了，把苹果设备连上数据线、打开 iTunes，少不了一个短暂的授权。

然后打开设备页，选择备份到「本电脑」，再选择「立即备份」。

![](https://mmbiz.qpic.cn/mmbiz_jpg/ncicWtGoBHtLvUwibse91Mbwa4GwoS3LKiaaJe7mPBOCcXjkMYbWqtL1CfY2m70oChGqI5Egx73xibM7wZoaEeSpiaw/640?wx_fmt=jpeg)

注意，千万不要给备份文件再加密了，这都是血的教训。

等待 iTunes 最上方的备份进度条跑完，就完成了前置的备份工作。

WechatExporter 真正完成的是解密和导出。

**导出聊天记录**

解压后的 WechatExporter 只有 10M 大小，我们要做的是打开文件夹内的可执行文件 WechatExporter.exe。

![](https://mmbiz.qpic.cn/mmbiz_jpg/ncicWtGoBHtLvUwibse91Mbwa4GwoS3LKiagrlIPQicfF0EDFL7W7OgNMI2Ra748pQGYvW7jq6XhZkSd0wEHJMWQYA/640?wx_fmt=jpeg)

你就能看到 WechatExporter 的真面目，整个界面分成了目录区和待导出区两部分。

![](https://mmbiz.qpic.cn/mmbiz_jpg/ncicWtGoBHtLvUwibse91Mbwa4GwoS3LKiaAVLMhetXJkFBicG4yD4H0RPpzht0VQXk9IKS4fDBzLPhdFkbUxg4Rhw/640?wx_fmt=jpeg)

WechatExporter 会自动帮你寻找 iTunes 的备份目录，如果你后面移动了备份文件，也可以手动选择，至于导到哪里，这个自己决定就好。

待导出区显示三列分别是聊天记录微信昵称、聊天数、以及你自己的微信昵称，除此之外还有你使用的 iTunes、iOS 和微信版本信息。

你只要给每一行前的框框内打个勾，再选择「导出」就能实现备份导出了，批量导出什么的是基操。

不单单是个人聊天记录，群聊的记录也会显示在其中。

而导出的聊天记录有两种格式，一种是 HTML、一种是 txt 文本。

![](https://mmbiz.qpic.cn/mmbiz_png/ncicWtGoBHtLvUwibse91Mbwa4GwoS3LKiazg1NN90cgJ752wJt3HIGlGeD8OJv29eDPNgicuL0wJ8nkcV266iawZag/640?wx_fmt=png)

你可以选择左下角的「显示日志」，查看导出进度，经过我的测试哈，1 万条聊天记录导出只需要 20s 左右（HTML）。

![](https://mmbiz.qpic.cn/mmbiz_jpg/ncicWtGoBHtLvUwibse91Mbwa4GwoS3LKiazseAoZCPbUvE1ZN17CLiamNuStMQ4OezoiaQsbQkWcictDC1iaOcv0Avyw/640?wx_fmt=jpeg)

当然了，导出内容包括你日常聊天的所有文字、表情包、图片、语音、视频文件、文档文件等等。

只要手机备份时没有失效，WechatExporter 都能给你保存到本地。

其实 WechatExporter 不是单单导出你用 iTunes 备份时的微信账号的聊天记录，只要是在手机上登录过，还没删除的微信账号，都可以导出。

![](https://mmbiz.qpic.cn/mmbiz_jpg/ncicWtGoBHtLvUwibse91Mbwa4GwoS3LKia10X9YNklst2fXyIQH3icY7Qau3DwFKYgJ6fxjxq8JRoZ3OQfxBl8D7A/640?wx_fmt=jpeg)

所以如果你是借别人的手机搞备份操作，千万别不小心把朋友的聊天记录给 copy 了，万一看到啥不该看的，那多尴尬。

最后看看效果如何，打开导出的目录，前两个是存放不同账号聊天记录的文件夹：

![](https://mmbiz.qpic.cn/mmbiz_jpg/ncicWtGoBHtLvUwibse91Mbwa4GwoS3LKiaY8QBPULBKmicYlYqOGaichzWeyYiaFy96Es2iccNuUTiatUckUdY8MXYFog/640?wx_fmt=jpeg)

选择一个进去，能看到导出的各路文件，而以 .html 为后缀的页面，就是对应的聊天记录：

![](https://mmbiz.qpic.cn/mmbiz_jpg/ncicWtGoBHtLvUwibse91Mbwa4GwoS3LKianATD5dP1MMoFycm9Dnmc4KrQUbEcA5Py2IIcUtCiaPqCiaTiafsVbaezQ/640?wx_fmt=jpeg)

网页打开是这样的：

![](https://mmbiz.qpic.cn/mmbiz_jpg/ncicWtGoBHtLvUwibse91Mbwa4GwoS3LKiakiahSx9Y3hWsSjOFErhJ5V3ENODsILkMy1xwdnuwF4A9ze63NIzVyLQ/640?wx_fmt=jpeg)

打开一个语音试试效果，没问题。

![](https://mmbiz.qpic.cn/mmbiz_gif/ncicWtGoBHtLvUwibse91Mbwa4GwoS3LKiaYPeQYzh2upKk4owAnnEv6tpGSABRibPFM9QB0glFNWmA9l2guc9oKtg/640?wx_fmt=gif)

打开一个图片，试试效果，也没问题。

![](https://mmbiz.qpic.cn/mmbiz_gif/ncicWtGoBHtLvUwibse91Mbwa4GwoS3LKiapTqAWzkjhN1kYAQcDuzefSy7ricBFGSPRqbO236O3P2ianwicWk2jklcQ/640?wx_fmt=gif)

这样的备份是不是很 nice，无论是记录回忆还是保存重要信息，轻轻松松保存到本地，想访问就访问，还不占用手机空间。

特别是如果你有特别在意的人，就可以打开你们的聊天记录浏览回味。

不过，如果你还想让自己浏览聊天记录时更顺心一点，比如让网页提供过滤功能，你还需要进行一些细节上的小设置。

**其他**

所有的细节设置都在顶部菜单栏的「选项」里设置。

![](https://mmbiz.qpic.cn/mmbiz_png/ncicWtGoBHtLvUwibse91Mbwa4GwoS3LKian5CA65QaQqO55Rib3UjmUGVDicRrUSopB6z6ZHEIwa0M8GREibY51sTpg/640?wx_fmt=png)

看看你就知道了，如果你想要网页上方出现过滤器，就把最后一个「显示消息过滤」给勾上。

至于消息按时间倒序排列、头像和表情的保存目录、异步加载或上滑加载都需要你自己选择了。

WechatExporter 是绿色开源的一款小工具，我在其 GitHub 的「issue」上看到了一个希望作者参考 wxbackup 样式的工具，我也顺着这个链接试了试。

![](https://mmbiz.qpic.cn/mmbiz_jpg/ncicWtGoBHtLvUwibse91Mbwa4GwoS3LKiaJ5zpP06w0Vsszlxcl0Nth0qzEAAvCOaq0QUZzOHWta8hfwFOg0hFqQ/640?wx_fmt=jpeg)

和 WechatExporter 本质上一样，都是通过 iTunes 备份实现导出。

我这里把 wxbackup 优缺点总结一下好了：

优点：可按月份筛选查看聊天记录、可增量导出

![](https://mmbiz.qpic.cn/mmbiz_png/ncicWtGoBHtLvUwibse91Mbwa4GwoS3LKiaJm4CDzhOcs4zwMQW4zEibT3XOH4ib44t1qkNnpT6YqCorflhmeBtOB0g/640?wx_fmt=png)

缺点：wxbackup 体量更大、iTunes 备份目录需要手动添加、微信昵称有瑕疵、音频文件无法正常播放、不可批量导出、导出速度较慢

我去具体的了解了一下 wxbackup，作者本人是个 Mac 用户，Windows 版是后来出的，使用起来远没有 Mac 版的舒服。

不过 wxbackup 官网上的常见问题解答倒是句句真切，对于微信聊天记录的备份还有疑问的，看看这个就明白了：

![](https://mmbiz.qpic.cn/mmbiz_png/ncicWtGoBHtLvUwibse91Mbwa4GwoS3LKiamaTia92LoQ0mmpya1tj2k6tqryBkRaibh8GUhXZpQtTt7uFjwNvsS4Vw/640?wx_fmt=png)

**结语**

保存聊天记录这事，我的想法是如果你真有备份导出的想法，放在本地肯定是要比存在云端安全。

某宝、某鱼上也有不少提供这个服务的商家，说是一对一辅助保存，但你看了这篇文章，不也能很好的把聊天记录给保存下来。

微信里积累了无数的聊天记录，把重要的聊天记录导出来，然后在电脑上好好收藏和品味，真的舒服。

看到这的小伙伴别忘了顺手点个在看和赞![](https://mmbiz.qpic.cn/mmbiz_png/ncicWtGoBHtLvUwibse91Mbwa4GwoS3LKiaSP4rVhsk5HV6Xq7895FsUlrYfEU20fcm4ccSO1kn77VHCw2UVQbm9A/640?wx_fmt=png)，不过说到底 WechatExporter 把聊天记录导成 HTML 文件，也就是对微信备份功能的补充。

微信还有哪需要补充的，大家一起来聊聊呗。

GitHub：https://github.com/tsycnh/WeChatExporter

公众号
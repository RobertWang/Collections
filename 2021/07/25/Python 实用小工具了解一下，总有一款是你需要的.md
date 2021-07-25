> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247503669&idx=2&sn=e7bd850a7b515b3820edd0fad12f9e17&chksm=e9699138de1e182e3b2c0f8171e72b7a80344caf5c67c9048eb1417a99791b88cc593c2d321e&mpshare=1&scene=1&srcid=0724mgWVqX1LzfU4o6lVQSWa&sharer_sharetime=1627116432376&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

1. 实现简单探测

使用 socket 模块，connect() 方法建立与指定 IP 和端口的网络连接；revc(1024) 方法将读取套接字中接下来的 1024B 数据

![图片](https://mmbiz.qpic.cn/mmbiz_png/Kad3LZzM7n5ts2Y6Iwpzxy9XqiahiaxLvkYpzcAYGiaVwdofu9YdkkXM8gTkIxw1Va7gt3zuOGQmwYYATdaSGkF3A/640?wx_fmt=png)

通过函数实现

通过 def() 关键字定义，示例中定义扫描 FTP banner 信息的函数：

![图片](https://mmbiz.qpic.cn/mmbiz_png/Kad3LZzM7n5ts2Y6Iwpzxy9XqiahiaxLvk7NnwqBx9vwgSkmcTlc4VYKGA9Dz18dVurJa8HfzyWrNhefEcM9b4tw/640?wx_fmt=jpeg)

迭代实现

![图片](https://mmbiz.qpic.cn/mmbiz_png/Kad3LZzM7n5ts2Y6Iwpzxy9XqiahiaxLvkcCO3lkqeW2ic4DEjy25jRSEHXcocial0M1AZ5iceMB2LtcPCwrdeRrX0g/640?wx_fmt=jpeg)

OS 模块

os.path.isfile() 检查该文件是否存在

os.access() 判断当前用户是否有权限读取该文件

搜索公众号 GitHub 猿后台回复 “打飞机”，获取一份惊喜礼包。

![图片](https://mmbiz.qpic.cn/mmbiz_png/Kad3LZzM7n5ts2Y6Iwpzxy9XqiahiaxLvkuCjquHZb8bm0LpIyRsHw0xWmgzDJ6JAjlWOnVZaMBQxSPhUTbV6rTA/640?wx_fmt=png)

整合

将上述各个模块整合起来，实现对目标主机的端口及其 banner 信息的扫描：

![图片](https://mmbiz.qpic.cn/mmbiz_png/Kad3LZzM7n5ts2Y6Iwpzxy9XqiahiaxLvkx7HbOG6s4BqoycKf5KejN8Ch9fVscL6Ns40VCWh7MZ9DX6DQj3F2Cg/640?wx_fmt=jpeg)

运行结果：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Kad3LZzM7n5ts2Y6Iwpzxy9XqiahiaxLvkYMNMRWZ9OR9wyhW0yvuhGPGlZAGIz8YFaUm6L2WAZtWG0ib8HfrMvlA/640?wx_fmt=jpeg)

第一个程序：Unix 口令破解机

这段代码通过分别读取两个文件，一个为加密口令文件，另一个为用于猜测的字典文件。在 testPass() 函数中读取字典文件，并通过 crypt.crypt() 进行加密，其中需要一个明文密码以及两个字节的盐，然后再用加密后的信息和加密口令进行比较查看是否相等即可。

![图片](https://mmbiz.qpic.cn/mmbiz_png/Kad3LZzM7n5ts2Y6Iwpzxy9XqiahiaxLvkFQkoWsRBGLGicDmNzex6voMRIY1jPw97PXeCF1BRoJIbEMj8ibUibUT2g/640?wx_fmt=jpeg)

第二个程序：一个 Zip 文件口令破解机  

主要使用 zipfile 库的 extractall() 方法，其中 pwd 参数指定密码

![图片](https://mmbiz.qpic.cn/mmbiz_png/Kad3LZzM7n5ts2Y6Iwpzxy9XqiahiaxLvkt3h4clytw0hbWTmnkyPVuiatjUk6CBNTgYibuATnqI3P0BA1Aj36KlgA/640?wx_fmt=jpeg)

代码中导入了 optparse 库解析命令行参数，调用 OptionParser() 生成一个参数解析器类的示例，parser.add_option() 指定具体解析哪些命令行参数  

usage 输出的是参数的帮助信息；同时也采用了多线程的方式提高破解速率。

运行结果：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Kad3LZzM7n5ts2Y6Iwpzxy9XqiahiaxLvkdTxgmJsvexo3e4eDRNiah0RG4bXPIwlTeVeR6WTslz8Jia9kHXjKOAdQ/640?wx_fmt=jpeg)

  

**你还有什么想要补充的吗？**  

> 免责声明：本文内容来源于网络，文章版权归原作者所有，意在传播相关技术知识 & 行业趋势，供大家学习交流，若涉及作品版权问题，请联系删除或授权事宜。  

[Python 如何调用远程接口](http://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247503589&idx=2&sn=65f630c4b6c4be0f719be21e86b01693&chksm=e96990e8de1e19fe7ad8edb0f1eb991431c64fde5f85dedfa6ce8ef25fe141f974a302b0d0a8&scene=21#wechat_redirect)  

[十五种 Python IDE 使用对比？这里有一份优缺点列表](http://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247503334&idx=2&sn=4b916e3b9bd77b3a633009fbe228f457&chksm=e96993ebde1e1afdb4ea8c35ebdd838bc4131940a85d33ce98a639f3abc49b2522f3d326593f&scene=21#wechat_redirect)
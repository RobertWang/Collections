> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI2MDY2NzQwMQ==&mid=2247495690&idx=1&sn=cffd01036592826116f7436c517f9994&chksm=ea64916cdd13187ab6d02c2bed30ef67628d78271e255b2db1a157ba1a648f4f52a4fe9524c7&mpshare=1&scene=1&srcid=040395aPaPIeMj7vC1h0j7gp&sharer_sharetime=1649001483284&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

人生苦短，快学 Python！

编码问题一直是 Python 学习者一个头疼的问题，经常看到的 **gbk**、**utf-8**，这都是啥玩意儿？因此，今天我正好出一期教程，好好讲述一下编码的起源和发展。

![](https://mmbiz.qpic.cn/mmbiz_png/dScwkmMJhccShVwx5IRSibia2hd7QJ5xdibHvZA9EsrODQBRHAM6Inb3JP02gwC6EibWzj9gjz3FLiaLNP375uazgcw/640?wx_fmt=png)

问题起源
----

我们在学习 Python 的过程中，可能会经常遇到下方这样的编码问题。有时候我们需要选择 **gbk**，有时候需要选择 **utf-8**。你以为这样就完了吗？我们碰到的还有 **gb2312**，**gb18030** 等各种奇奇怪怪的编码。那么，编码的起源究竟是怎样的呢？我们今天就用 **“讲故事”** 的方式，带你认识一下它。

![](https://mmbiz.qpic.cn/mmbiz_png/hRU7GdO3rMVNZ3b3mfoBLFHiaTGClXCoAWpVuBiaadugFdeNOAZvzg2EFxTslIjwxkVoAHTjFNR3KxcdAXsMm9sA/640?wx_fmt=png)

黄同学给你讲故事  

-----------

#### 1）烽火士兵的故事

在正式讲故事之前，我们先来看一下下方这张图，我们暂且称其为**《烽火士兵》**的故事，那么这个故事究竟是怎么与编码问题扯上联系的呢？接着听我讲故事。

![](https://mmbiz.qpic.cn/mmbiz_png/hRU7GdO3rMVNZ3b3mfoBLFHiaTGClXCoAVHckDnaVIFLL7kkD0E7G77tuj5YuIjXhuAia9GVbkDbqr8A57cYOgAA/640?wx_fmt=png)

**这一串数字，从右朝左看**

点燃第 1 根，代表有一个士兵，点燃第 2 根，代表有二个士兵。那么也就是说，点燃 2 个烽火，最多可以表示三个士兵。**梳理一下逻辑**，1 个烽火都不点，表示有零个士兵；只点燃第 1 个烽火，就表示有一个士兵；在点燃第 2 个烽火的时候，熄灭第 1 个烽火，就表示有二个士兵；同时点燃 2 个烽火，就表示有三个士兵。

**综上所述：**2 根烽火，可以表示：0、1、2、3 个士兵，即 1+2。3 根烽火，可以表示：0、1、2、3、4、5、6、7 个士兵，即 1+2+4。依此类推下去…

![](https://mmbiz.qpic.cn/mmbiz_gif/hRU7GdO3rMVNZ3b3mfoBLFHiaTGClXCoAPpH5Xqaz80HlqqpEZwzv3CUTyMhWoFEN7X1V4x050xOAgtiavrCRiaQQ/640?wx_fmt=gif)

通过上面的叙述，你可能已经发现了，这不就是类似计算机里面的**二进制数字**吗？只有 0 和 1，0 表示熄灭烽火，1 表示点燃烽火。对应到计算机中就是，0 表示关，1 表示开。下面黄同学就带着大家说一下 **“计算机中的 0 和 1”**。

计算机的底层是电路，只认识 0 和 1，就是你初中物理中所谓的 **“电路”**，0 表示关，1 表示开，没有别的玩意儿。但是你想呀，一个电路只有 0 和 1 的话，怎么展示出这绚丽多彩的世界呢？因此，聪明的老外，把日常所用的文字和符号，编码成 0101010… 类型，这样电脑就能够表示文字了。所以，**先记住一个关键语：“用什么编码，就用什么解码”**。

由于，计算机是由美国人发明的。因此，最早的计算机编码：**ASCII 码** (也是服务于美国人的)，里面只有美国人日常所用的 26 个英文字母、数字、标点等常用字符，所以，最早的计算机也只有英文、数字、标点等特殊字符。不要惊叹为啥只有美国人常用的英文字母和符号，因为老美当时就没有想过计算机会迅速在全世界普及开来，谁也不能提前预知未来。

**接着我们来说说最早的计算机编码：**ASCII 码。ASCII 码占 8 个比特位，也就是一个字节，其中最前面一个位是扩展位，都是 0，为了日后扩展所用，其余位置不是 0 就是 1。这是由于计算机对数字 7 不敏感，熟悉 2、4、8、16、32... 等数字，所以扩展了一位，成了 8 位。那么根据**排列组合**的知识，ASCII 码可以表示 2^7=128 个码位，即可以表示 128 种不同的符号，其实这些符号已经够美国人使用了。这就是当时最早的计算机编码 (ASCII 码)，这就是当时老美的打算。

![](https://mmbiz.qpic.cn/mmbiz_png/hRU7GdO3rMVNZ3b3mfoBLFHiaTGClXCoAWwFZa87ml030rq6PwtYgQoUgrGZxK5NJq5n10xfES0IjvWOTjE0taw/640?wx_fmt=png)

#### 2）计算机在中国的发展

随着计算机在世界各地的发展，我们发现原有的码位已经不够存放多国的文字和符号了，为了讲清楚这件事儿，我们以计算机在中国的发展为例，进行说明。

通过前面的叙述，我们已经知道最早的字符编码 ASCII 码中，并没有中文，但是随着计算机在中国的普及，我们必要要让计算机中能够表示中文呀，怎么办呢？基于此：**中国北大方正团队，发明了 gbk 编码**。但是这些字符肯定不能直接往 ASCII 码里面放，因为 ASCII 只有 8 位，最多才有 **2^8=256** 个空位，存放九万多汉字，显然不可能 (就连中文中常用的 3000 汉字，也存放不了)。所以在 gbk 中，汉字用 2 个字节表示，变成了 ASCII 码中字节长度的 2 倍，即 gbk 占 16 位，共 **2^16=65536** 个空位，这个对于存放常用汉字，多得多，但是，仍然不能将所有汉字存放进去，谁让中华文化源远流长，博大精深呢。

![](https://mmbiz.qpic.cn/mmbiz_png/hRU7GdO3rMVNZ3b3mfoBLFHiaTGClXCoANdh631PBRSibD0iaQiaiahbsxAmV1iaJ9TFib7rDs2COSce4OxCDm3jVXUgA/640?wx_fmt=png)

说到 gbk，就不得不说它的兄弟姐妹了 (如图所示)，其实它们是一个系列，都是由于当时的需要，逐步衍生出来的，**这三种不同的编码都是向上兼容的**。可以看出：GB18030 表示的字符数最多，这也就是为什么有时候使用 Python 读取 Excel 表时，使用 GB2312 和 GBK 都不行，而必须使用 GB18030 的原因了。

![](https://mmbiz.qpic.cn/mmbiz_png/hRU7GdO3rMVNZ3b3mfoBLFHiaTGClXCoAJQc9A2ukp8LISoia9hAVJNGFdDsbqBf0zRItW85sZFianQWibt8KQbJicA/640?wx_fmt=png)

#### 3）计算机如何兼容多国语言

计算机不仅在中国发展开来，其实计算机是在全世界迅速发展开来。如果说中国有自己独有的 GBK 编码，那么韩国、日本肯定也有它们自己独有的编码。但是当今是 **“经济全球化”** 的时代，任何一个国家都不可能的单独发展，假如你有一个国际合作的业务，我们在中国写的代码，要是想拿到国外去用，出现乱码，这样多尴尬？**那么这个问题最终是怎么解决的呢？**

![](https://mmbiz.qpic.cn/mmbiz_png/hRU7GdO3rMVNZ3b3mfoBLFHiaTGClXCoAibYqJic9eyB9Y2bMzAYoibiaXZWRl9WPofX1nJIIcOnoqfyA4sbAKZgcUw/640?wx_fmt=png)

为此，美国人又发明了一个叫做 **“Unicode”** 的东西，又叫做 **“万国码”**。其实完全可以见名知意，万国码万国码，肯定是为了包含全世界的字符编码！那么什么是万国码呢？接着听黄同学给你讲。

![](https://mmbiz.qpic.cn/mmbiz_png/hRU7GdO3rMVNZ3b3mfoBLFHiaTGClXCoAOP6mW5ThhVoXALtiakP4DI9glFuZgsRYcHbhJ6IZscPv8KrIylgFibZg/640?wx_fmt=png)

计算机扩展一般是成倍增加的，要么是 1 个字节、2 个字节、4 个字节......。最开始的 Unicode，又叫 **ucs-2**，ASCII 存储采用 1 个字节，因此 ucs-2 采用 2 个字节进行存储，最多有 **2^16=65536** 个空位，这样仍然无法兼容全世界的字符。于是 **ucs-4** 产生了，存储采用 4 个字节，共 **2^32=4 亿**多个空位。但是据统计，全世界文字、数字、符号信息加起来也就 23 万，对于 4 亿多空间来说，ucs-4 简直太**浪费空间**了，这个对于文件传输来说，及其浪费流量。

![](https://mmbiz.qpic.cn/mmbiz_png/hRU7GdO3rMVNZ3b3mfoBLFHiaTGClXCoAAAeQ7NPj9sicm3QKoOc7dObunNd3Dib6nV4icjOPhr80SibyMfD1L5FemA/640?wx_fmt=png)

考虑到节省空间，在 Unicode 基础上，我们又发明了 **utf-8，****一种可变长的 Unicode 字符编码**。Utf-8，对于英文来说，采用 ASCII 码占位方式，占 8 位，即 1 个字节；存放欧洲文字时，占 16 位，即 2 个字节；存放中文时，占 24 位，即 3 个字节。虽然**对于中文来说很浪费空间**，但是为了能把全世界文字都统一起来，又为了节省空间，采用这种方式，已经很好了 (因为毕竟不可能做到面面俱到，谁让中国字符最多，会吃亏一点)。

编码知识总结
------

#### 1）字符编码发展史

![](https://mmbiz.qpic.cn/mmbiz_png/hRU7GdO3rMVNZ3b3mfoBLFHiaTGClXCoAPJnk2UGM6OzaGq7rPFsywnBIPcomRxpaa4G7ziaJtVXIjFQaOibgSicQg/640?wx_fmt=png)

#### 2）以小写字母 a 为例，说明字符编码

![](https://mmbiz.qpic.cn/mmbiz_png/hRU7GdO3rMVNZ3b3mfoBLFHiaTGClXCoAh4J3ibNznRxeDqZ8yiaBDW0hStjetZWaQRUYCxbcPh5ea5yT36icDqNgw/640?wx_fmt=png)

#### 3）带着大家写写代码，认识一下字符编码

###### ① 关于 Python2 和 Python3 的区别

**在 Python2 中**，默认字符编码是 ASCII 码，因此在 Python2 中写中文，首行一般都会加上 -_- coding:utf-8 -_-，看了这篇文章，我想你对这个东西已经有了一个清楚的认识。但是 Python2 现在已经停止更新了，我们了解即可，不用太关注。

**对于 Python3.x 来说**，默认字符编码是 utf-8，而 utf-8 是 Unicode 的扩展集。即 Python3.x 中默认所有的字符都是 Unicode。说白点，我们在 Python3.x 中随便写点啥，编码就是 Unicode 编码。

**对比 Python2 和 Python3：**

```
# 在Python2中如果要表示Unicode编码，应该这样写。
my_name = u"黄伟"
# 在Python3中如果要表示Unicode编码，应该这样写。
my_name = "黄伟"
```

说到这里，我们可以下一个结论：**不同编码之间的转换，都要经过一个 Unicode**。

② encode 编码和 decode 解码

```
>>> name1 = "我是你们的teacher老师"
>>> name2 = "你们是我的student学生"
>>> # 将name1编码为“utf-8”
>>> name1_encode = name1.encode("utf-8")
>>> name1_encode
b'\xe6\x88\x91\xe6\x98\xaf\xe4\xbd\xa0\xe4\xbb\xac\xe7\x9a\x84teacher\xe8\x80\x81\xe5\xb8\x88'
>>> # 将name1_encode解码还原
>>> name1_encode.decode("utf-8")
'我是你们的teacher老师'
---------------------------------------------------------
>>> # 将name2编码为“gbk”
>>> name2_encode = name2.encode("gbk")
>>> name2_encode
b'\xc4\xe3\xc3\xc7\xca\xc7\xce\xd2\xb5\xc4student\xd1\xa7\xc9\xfa'
>>> # 将name2_encode解码还原
>>> name2_encode.decode("gbk")
'你们是我的student学生'
-------------------------------------------------
>>> # name1_encode此时是“utf-8”编码，如果用“gbk”解码，会出现什么？
>>> name1_encode.decode("gbk")
'鎴戞槸浣犱滑鐨則eacher鑰佸笀'
# 上面就是我们常说的乱码、乱码、乱码！
```

**代码分析：**从代码中可以看出，如果是 utf-8 编码，每个中文字符就是 3 个字节存储。如果是 gbk 编码，每个中文字符就是 2 个字节存储。
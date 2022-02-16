> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIzMzMzOTI3Nw==&mid=2247505468&idx=1&sn=bb90eb254b8d86a339cba57cb79942fa&chksm=e885b6dedff23fc8a714bd262b45e927e7975a2ce6e2f6dc8296d3d3c75fac607e8b4cc02c4f&mpshare=1&scene=1&srcid=0207D5pjg4hLVUhxvEF4DdI0&sharer_sharetime=1644210930842&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

我们今天来看一段炫技代码。它可以把任何能接收两个参数的函数定义成一个特殊的运算符。

![](https://mmbiz.qpic.cn/mmbiz_png/ohoo1dCmvqckGUn0fIooL54TT6OTHc61oKicVa3AIiahRzsc0ic76AmgfhtnCibRofbrFLGqFxWuwIBgpgAl6wSGeg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ohoo1dCmvqckGUn0fIooL54TT6OTHc61XR881lFuj6BlibxFIB7LbpCeb9GQoutQniaGuhXJibiaanPJgdMGrAXK8w/640?wx_fmt=png)

这种炫技有余，实用不足的功能是怎么实现的呢？其实原理非常简单，只有 8 行代码：

```
from functools import partial

class Change(object):
    def __init__(self, func):
        self.func = func
    def __or__(self, other):
        return self.func(other)
    def __ror__(self, other):
        self.func = partial(self.func, other)
        return self
```

这里就涉及到一个盲点和两个真正的知识点。这个盲点就是，你可能以为 `|到|`是一个字符，但是它是 3 个字符；你可能会把`|拼接|`看做一个整体，但是它实际上是 3 个部分：左边的`|`、`拼接`和右边的`|`。

我们把空格加上，就很明显了：

![](https://mmbiz.qpic.cn/mmbiz_png/ohoo1dCmvqckGUn0fIooL54TT6OTHc61zRMrDsH27RqqL4r5K6txMhgErrekegJXM0Y8oicncc1KMzSFicX1VVqQ/640?wx_fmt=png)

两个真正的知识点，就是`__or__`和`__ror__`这两个魔术方法和偏函数`partial`。而`Change`本身就是一个普通的类而已，`__or__`和`__ror__`定义了这个类的实例在左侧遇到`|`时，右侧遇到`|`时的具体行为。

我们一个一个来讲。首先是`__or__`。它定义了实例的右侧遇到`|`时的具体行为。例如，我们用一个简单的代码来进行测试：

```
class Test:
    def __init__(self, num):
        self.num = num
    def __or__(self, other):
        print(f'我右边有一个东西，它是：{other}')

x = Test(100)
x | 55
```

运行效果如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/ohoo1dCmvqckGUn0fIooL54TT6OTHc61B7080HSKpro2B8T6ibFNpJfKsiaH0NNQStrUH5GhyU3ITwhicmccAmQbQ/640?wx_fmt=png)

但如果你把竖线放在左边，他就会报错，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/ohoo1dCmvqckGUn0fIooL54TT6OTHc61TWOhiaqJ4sy0Mc1QOiabc4KcnbqgXg8cDluGpCDQHDPD8dgLAeVvDlpw/640?wx_fmt=png)

而`__ror__`就是用来定义`|`在实例左边的时候，它的行为：

![](https://mmbiz.qpic.cn/mmbiz_png/ohoo1dCmvqckGUn0fIooL54TT6OTHc61zQEblNru1WH9Dn9GtVaVniaV8Qyy0wWlVqkM8ooicFwAVwLVtp0VIeZw/640?wx_fmt=png)

所以，我们最开始的例子中，`2 |到| 10`，实际上应该理解为：

1.  `到`是`Change(range)`返回的实例
    
2.  `2 | 到` 生成一个中间对象，我们假设它是`x`
    
3.  `x | 10` 生成结果
    

在我们演示的例子中，`2 | 到`首先进入了`Change`类的`__ror__`方法中：

```
    def __ror__(self, other):
        self.func = partial(self.func, other)
        return self
```

其中，一开始的`self.func`就是我们在初始化实例`Change(range)`时传入的参数`range`。所以`partial(self.func, other)`等价于`partial(range, 2)`。关于偏函数`partial`，大家可以看我这篇文章：[偏函数：在 Python 中设定默认参数的另一种办法](https://mp.weixin.qq.com/s?__biz=MzI2MzEwNTY3OQ==&mid=2648980044&idx=1&sn=6c53bcd5b744970962f975acd96a7b18&scene=21#wechat_redirect)。简单来说，使用偏函数，可以给一个真正的函数传一部分参数，过一会再补剩下的参数。

可能大家在日常的开发者，很少会让一个实例方法返回`self`。关于这个写法，大家可以看我的这一篇文章：[一日一技：在 Python 里面实现链式调用](https://mp.weixin.qq.com/s?__biz=MzI2MzEwNTY3OQ==&mid=2648982442&idx=1&sn=93799b5c7b5a5e08ce70aa48ffb17ed2&scene=21#wechat_redirect)。也就是说，`1 | 到`返回的，依然是`Change`类的一个实例，我们简称它为`x`。这个实例的属性`self.func`的值是`partial(range, 2)`。

接下来，`x | 10`，调用的是`__or__`方法，于是，此时执行的是`partial(range, 2)(10)`。偏函数的参数补全了，于是它里面的`range`真正运行了起来，成为了`range(2, 10)`。

至此，这个`Change`类我们就解析透了。大家知道，在 Python 里面，魔术方法是有很多的，如果你不想用`|`，你还可以用其它的，例如：

![](https://mmbiz.qpic.cn/mmbiz_png/ohoo1dCmvqckGUn0fIooL54TT6OTHc6189NBhlyvib481mJ7ELNtA31kSETCkrIicxsFWIhXHQtAZZFEYia8XqP7w/640?wx_fmt=png)

或者：

![](https://mmbiz.qpic.cn/mmbiz_png/ohoo1dCmvqckGUn0fIooL54TT6OTHc61mgyJ8DFHGf8lb2zgvxAjibJmEzanicOY2eFgWHicVY5SV4l0FDYS9ibia4Q/640?wx_fmt=png)

或者：

![](https://mmbiz.qpic.cn/mmbiz_png/ohoo1dCmvqckGUn0fIooL54TT6OTHc61YQhk3ic9iazZQOlBWvoGGTmp1hzQnpRiaykX2DJRhu2ADWhjFdqXMInQQ/640?wx_fmt=png)

同时，这个`Change`类，你甚至可以直接当做装饰器来使用。任何能够接收两个参数的函数，都能使用这个装饰器。例如：

![](https://mmbiz.qpic.cn/mmbiz_png/ohoo1dCmvqckGUn0fIooL54TT6OTHc61kNZeK2WLPElmPBU32iarI3IeMTnYLBmbTZLI1LOYicicXQ6gIHbR00xRw/640?wx_fmt=png)

最后总结一下。大家都知道，我是非常反对在工作代码中炫技的，因为炫技的写法很难读，很难维护。今天这个炫技的方法，虽然我也不推荐大家用在工作中，但是它短短 8 行代码里面，包含了很多个知识点，这就值得大家玩一玩了。
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU2NTczODU3NA==&mid=2247497367&idx=2&sn=f641e157dab6e234685faca4ade5fc1d&chksm=fcb59bb6cbc212a0499d4c6c6b8a3e36329075ce4492e124e7891247ce2f39a72046702b176f&mpshare=1&scene=1&srcid=1206wvwOjs0NE7NZvoHmuqWB&sharer_sharetime=1638763766002&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

公众号

大家好，我是 Kuls。

今天在 Github 阅读 EdgeDB[1] 的代码，发现它在处理大量`if...elif...else`判断的时候，使用了一个非常巧妙的装饰器。我们来看看这个方法具体是什么样的。

正好今天是双十一，假设我们要做一个功能，根据用户的等级判断他可以获得的折扣。常规的`if ... elif...`写法是这样的：

```
def get_discount(level):
    if level == 1:
        "大量计算代码"
        discount = 0.1
    elif level == 2:
        "大量计算代码"
        discount = 0.2
    elif level == 3:
        discount = 0.3
    elif level == 4:
        discount = 0.4
    elif level == 5:
        discount = 0.5
    elif level == 6:
        discount = 3 + 2 - 5 * 0.1
    else:
         return '等级错误'
    return discount
```

大家都知道，这样大量的`if ... elif...`代码非常难看，也很难维护。并且每个 if 的内部有很多代码。这个函数就会被拉得非常长。

有一些同学知道，可以使用字典来改写这个太长的 if 判断：

```
def parse_level_1():
    "大量计算代码"
    discount = 0.1
    return discount

def parse_level_2():
    "大量计算代码"
    discount = 0.2
    return discount

def parse_level_3():
    "大量计算代码"
    discount = 0.3
    return discount

def parse_level_4():
    "大量计算代码"
    discount = 0.4
    return discount

def parse_level_5():
    "大量计算代码"
    discount = 0.5
    return discount

def parse_level_6():
    "大量计算代码"
    discount = 3 + 2 - 5 * 0.1
    return discount

discount_map = {
 1: parse_level_1,
  2: parse_level_2,
  3: parse_level_3,
  4: parse_level_4,
  5: parse_level_5,
  6: parse_level_6,
}

discount = discount_map.get(level, '等级错误')
```

但今天我学到的这个方法，比用字典更简单。我们先来看它的效果：

```
@value_dispatch
def get_discount(level):
    return '等级错误'

@get_discount.register(1)
def parse_level_1(level):
    "大量计算代码"
    discount = 0.1
    return discount

@get_discount.register(2)
def parse_level_2(level):
    "大量计算代码"
    discount = 0.2
    return discount

@get_discount.register(3)
def parse_level_3(level):
    "大量计算代码"
    discount = 0.3
    return discount

@get_discount.register(4)
def parse_level_4(level):
    "大量计算代码"
    discount = 0.4
    return discount

@get_discount.register(5)
def parse_level_5(level):
    "大量计算代码"
    discount = 0.5
    return discount

@get_discount.register(6)
def parse_level_1(level):
    "大量计算代码"
    discount = 3 + 2 - 5 * 0.1
    return discount


discount = get_discount(3)
print(f'等级3的用户，获得的折扣是：{discount}')
```

运行效果如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/ohoo1dCmvqcXJy9a8t6YG6WdGufTwJe7KK3EHt3XlIMMAhCOZreG8rVIALXrZsjg20Bp7fnDH1SYGylEvyHT7Q/640?wx_fmt=png)

这样写，比用字典的方式更直观，比直接用`if ... elif...`更简洁。

那么，这个装饰器`value_dispatch`是怎么实现的呢？密码就藏在这个开源项目`EdgeDB`的源代码 [2] 中，核心代码只有 20 多行：

![](https://mmbiz.qpic.cn/mmbiz_png/ohoo1dCmvqcXJy9a8t6YG6WdGufTwJe7Y1QtgSz9sQXJ0yIQ5ibl2lkUABFa9Fl0T73L1icdGOTicD7UOm1RsQGGg/640?wx_fmt=png)

并且，还能够实现或查询。例如用户等级为 2 或者 3 的时候，折扣都是 0.2，那么代码可以写成：

```
@get_discount.register(2)
@get_discount.register(3)
def parse_level_2(level):
    "大量计算代码"
    discount = 0.2
    return discount
```

运行效果如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/ohoo1dCmvqcXJy9a8t6YG6WdGufTwJe7GFnavfrjF5G9N1ianxLjIKiaDTkU2FqZvw5GiaBhLjsiagQ6ib8P61icCP2A/640?wx_fmt=png)

它这个代码目前只能实现相等的查询。但其实只要对这个代码稍作修改，我们就能实现大于、小于、大于等于、小于等于、不等于、`in`等等判断。如果大家有兴趣的话，请在文章下面留言，我们明天就来说说怎么对这个代码进行改造，实现更多的逻辑判断。

**参考文献**

[1] EdgeDB: https://github.com/edgedb/edgedb  

[2] 源代码: https://github.com/edgedb/edgedb/blob/master/edb/common/value_dispatch.py

END
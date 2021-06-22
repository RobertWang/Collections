> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxMjUyNDQ5OA==&mid=2653576455&idx=2&sn=050d80e7efead672c1d27f37d1d15206&chksm=806e73bab719faac3047be2dbca11b25cb9dc6503046f4ff537bad1c89e8144217b8ecbcc6f8&mpshare=1&scene=1&srcid=0622vDWcCtc4q6Cg2vDLPmSq&sharer_sharetime=1624360616361&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

Python 3.10 的开发已经稳定下来，我们终于可以测试最终版本中将包含的所有新功能。下面我们将介绍 Python 3.10 中最有趣的一些新增功能——结构模式匹配、带括号的上下文管理器、 更多类型以及新的报错消息。

**结构模式匹配**

结构模式匹配是要添加到 Python 中的一个很棒的功能。想象一个如下所示的 if-else 语句（Python 3.9）：

```
http_code = "418"
if http_code == "200":
print("OK")
elif http_code == "404":
print("Not Found")
elif http_code == "418":
print("I'm a teapot")
else:
print("Code not found")

```

输出：

```
I
'm a teapot
```

Python 3.10 中可以这样写：

```
http_code = "418"
match http_code:
case"200":
print("OK")
case"404":
print("Not Found")
case"418":
print("I'm a teapot")
case _:
print("Code not found")

```

这就是新的 `match-case`语句——很酷，但目前还没有什么特别之处。使 `match-case`语句如此有趣的原因是一种称为结构模式匹配的东西。结构模式匹配允许我们执行相同的 `match-case` 逻辑，但基于我们的比较对象的结构是否与给定的模式匹配。

因此，让我们定义两个字典，它们都具有不同的结构。

```
dict_a = {
'id': 1,
'meta': {
'source': 'abc',
'location': 'west'
}
}

```

```
dict_b = {
'id': 2,
'source': 'def',
'location': 'west'
}

```

现在，我们可以编写一个模式来匹配 `dict_a`，如下所示：

```
{
'id': int,
'meta': {'source': str,
'location': str}
}

```

还有一个匹配 `dict_b`的模式：

```
{
'id': int,
'source': str,
'location': str
}

```

如果我们将这两个放在一个 `match-case`语句中，以及有效的 `else/`和包罗万象的 `case_` - 我们得到：

```
# loop through both dictionaries and a 'test'
for d in[dict_a, dict_b, 'test']:
    match d:
case{'id': ident,
'meta': {'source': source,
'location': loc}}:
print(ident, source, loc)
case{'id': ident,
'source': source,
'location': loc}:
print(ident, source, loc)
case _:
print('no match')

```

输出结果：

```
1 abc west
2def west
no match

```

是不是很酷？我已经发现这对数据处理非常有用。

**带括号的上下文管理器**

一个较小的变化是新的基于 PEG 的解析器。以前的 Python 解释器有很多限制，这限制了 Python 开发人员可以使用的语法。

Python 3.9 的基于 PEG 的解析器消除了这些障碍，从长远来看，这可能会导致更优雅的语法——这种变化的第一个例子是新的带括号的上下文管理器。在 Python 3.9 之前，我们可以写这样的东西来打开两个（或更多）文件 I/O 流：

```
with open('file1.txt', 'r') as fin, open('file2.txt', 'w') as fout:
    fout.write(fin.read())

```

第一行很长。但是由于解析器的限制，我们可以将此行拆分为多行的唯一方法是使用 `\` 行继续符：

```
with open('file1.txt', 'r') as fin, \
     open('file2.txt', 'w') as fout:
    fout.write(fin.read())

```

它是有效的，但不是很 Pythonic。使用新的解析器，我们现在可以将括号将这一行拆分为多行，如下所示：

```
with(open('file1.txt', 'r') as fin,
      open('file2.txt', 'w') as fout):
    fout.write(fin.read())

```

这种写法很 Pythonic。现在，在我们继续，如果我们写：

```
with(open('file1.txt', 'r') as fin,
      open('file2.txt', 'w') as fout):
    fout.write(fin.read())

```

在 Python 3.9 中也可以这样写。这是因为新的解析器启用了这种语法，尽管直到 Python 3.10 才被正式支持。

**`Typing`功能**

Python 的输入功能也有更多更新。这里最有趣的添加是包含了一个新的运算符，它的行为类似于类型的 `OR` 逻辑，我们之前使用 `Union` 方法来实现：

```
from typing importUnion
def add(x: Union[int, float], y: Union[int, float]):
return x + y

```

现在，我们不需要写成 `fromtypingimportUnion`，并且 `Union[int,float]` 已经简化为 `int|float`，看起来更简洁：

```
def add(x: int| float, y: int| float):
return x + y

```

**更加完善的报错信息**

相信你第一次看到时都会去百度或者 Google 搜索：

```
SyntaxError
: unexpected EOF 
while
 parsing
```

输入 SyntaxError 时，Google 中排名第一的结果表明我们中的许多人确实在某个时候做过。

![](https://mmbiz.qpic.cn/mmbiz_png/5mt0ewv9OS2g6SjuHAO6RSh0wiaIsFtpgMxpMibcaeVou33X8CbQc0YJPkaBsHA0EZajpGw2rMcVgOPcWGup4N4Q/640?wx_fmt=png)

这不是一条明确的报错消息，Python 中充满了不太明确的报错消息。幸运的是，有人注意到了它们——其中许多消息都得到了显著改善。

![](https://mmbiz.qpic.cn/mmbiz_png/5mt0ewv9OS2g6SjuHAO6RSh0wiaIsFtpgqV6PGecxWGxZ5RgMZ0KYbc3FL30HLI9gHiaHyMQHnnynsIhmn1TL5Kw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/5mt0ewv9OS2g6SjuHAO6RSh0wiaIsFtpgFfibCUaVJsd4pTt82B0W2xRiakBKgNJVH7UdZd70BTcn3OjwObKB3p7w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/5mt0ewv9OS2g6SjuHAO6RSh0wiaIsFtpg4y1yJ2D3DTj564y0urQZZpH5ianpkEkglOgqWGgTtib2bJ0JgTZNr2WQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/5mt0ewv9OS2g6SjuHAO6RSh0wiaIsFtpgFu8UrpDttWEoLgMc9HIqd48QlyJzJ99rS16SsjwFHdtSAglCXxcKMA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/5mt0ewv9OS2g6SjuHAO6RSh0wiaIsFtpgzI3e2y6iabs5B2cyvtKYHpsDebiaNrrc84Z6k8icQjz8RZhla72VPj5iag/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/5mt0ewv9OS2g6SjuHAO6RSh0wiaIsFtpg8fa7pXvQPkMPIZGFVLue1Kg7D1KzGcaHZciaqdBmqc7vNdpJJicMZSxg/640?wx_fmt=png)

官方更改列表中提到了更多改动 - 但在测试期间似乎没有显示，包括：

```
from collections import namedtoplo
> AttributeError: module'collections' has no attribute 'namedtoplo'. Did you mean: namedtuple?

```

在这里， `AttributeError` 与之前相同，但增加了一个建议的属性名称—— `namedtoplo` 被标识为属性 `namedtuple`的潜在拼写错误。

同样，我们看到 `NameError`消息也有相同的改进：

```
new_var = 5
print(new_vr)
> NameError: name 'new_vr'isnotdefined. Did you mean: new_var?

```

**总结**

以上是 Python 3.10 引入的一些关键新功能！

完整版本预计于 2021 年 10 月 4 日发布，从现在开始，Python 开发人员将致力于改进已经添加的内容——但不会引入新功能。如果您想自己检查一下，可以从这里下载 3.10.0b1。

https://www.python.org/downloads/release/python-3100b1/
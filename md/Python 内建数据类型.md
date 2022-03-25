> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [paul.pub](https://paul.pub/python-built-in-types/)

> Python 内建数据类型, Python, types,numeric,dictionary,tuple,set, 本文是一篇关于 Python 基础知识的文章，介绍 Python 中最常用的内建数据类型。

本文是一篇关于 Python 基础知识的文章，介绍 Python 中最常用的内建数据类型。

*   [前言](#id-前言)
*   [数据类型概述](#id-数据类型概述)
*   [数值类型](#id-数值类型)
    *   [数值字面值](#id-数值字面值)
    *   [进制与转换](#id-进制与转换)
    *   [运算](#id-运算)
    *   [链式比较](#id-链式比较)
    *   [除法](#id-除法)
*   [字符串](#id-字符串)
    *   [字符串字面值](#id-字符串字面值)
    *   [字符串的方法](#id-字符串的方法)
    *   [格式化](#id-格式化)
*   [集合](#id-集合)
*   [列表](#id-列表)
*   [字典](#id-字典)
*   [元组](#id-元组)
*   [成员关系与迭代](#id-成员关系与迭代)
*   [索引和分片](#id-索引和分片)
*   [变量与对象](#id-变量与对象)
    *   [== 和 is](#id-和is)
    *   [赋值与复制](#id-赋值与复制)
*   [参考资料与推荐读物](#id-参考资料与推荐读物)

![](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-04-python-built-in-types/bg.jpeg)

越来越多的人使用 Python 语言，其实很多 Python 程序员都有 C/C++ 或者 Java 的编程背景。

不同的编程语言会有很多相似的概念，例如：分支语句，变量，函数，类等。另一方面，不同的编程语言之间也会有很多微妙的差别。例如：是强类型还是弱类型，是否支持垃圾回收等。

除此之外，每种编程语言自带的数据类型也是一种很强烈的语言特征。

为了和大家一起尽可能少的被语言的底层细节干扰，因此接下来会有一些关于 Python 基础知识的文章。

本文作为开始，会讲解 Python 中内建数据类型的知识。当然，本文不打算取代 Python 官方文档或者是其他更加详尽的技术书籍。本文的目的是帮忙读者快速梳理这方面知识。并罗列出核心内容。

从 2020 年开始，Python 2 将逐渐退出历史舞台。因此**我们的系列文章仅仅提及 Python 3，不会提及 Python 2 中的语法**。

从分解的角度来看，Python 的程序由模块构成，模块由语句构成，语句由表达式构成，表达式创建并处理对象。由此可以看到，对象是程序的最基本组成单元，而对象是类型的实例。

Python 中包含了大量的数据类型（其他编程语言，例如 Java，C++ 也是一样）。但其中有一些最为大家所频繁使用的就是内建数据类型。正如其名称所示，这些类型是内置在语言中的，不需要额外的库或者依赖。在满足要求的情况下，使用内建类型是比较好的选择。

本文涉及的类型如下表所示：

<table><thead><tr><th>类型名称</th><th>英文</th></tr></thead><tbody><tr><td>数字</td><td>Number</td></tr><tr><td>字符串</td><td>String</td></tr><tr><td>列表</td><td>List</td></tr><tr><td>字典</td><td>Dictionary</td></tr><tr><td>元祖</td><td>Tuple</td></tr><tr><td>集合</td><td>Set</td></tr></tbody></table>

每种类型都有特定适用的场景。根据是否可变，可以将这些类型分为下面两类：

*   可变类型：列表，字典，集合
*   不可变类型：数字，字符串，元祖

对于可变类型，你可以原地修改它们。但对于不可变类型来说，如果你想要修改它们，只能创建它们的拷贝。

除了通过类名的方式来构建对象，下面几种括号是创建这些对象的更简洁的方法：

*   `(...)`：元组
*   `[...]`：列表
*   `{...}`：字典，集合

Python 中的数值类型并非一种类型，而是一系列类型，它们如下表所示：

<table><thead><tr><th>类型</th><th>类名</th><th>说明</th></tr></thead><tbody><tr><td>整数类型</td><td>int</td><td>包括正整数，负整数和零</td></tr><tr><td>浮点类型</td><td>float</td><td>带有小数部分的数字</td></tr><tr><td>布尔值类型</td><td>bool</td><td>包含<code>True</code>和<code>False</code>两个状态</td></tr><tr><td>复数类型</td><td>complex</td><td>通常在工程和科学应用中使用，包含实部和虚部</td></tr><tr><td>小数类型</td><td>decimal.Decimal</td><td>精度固定的浮点数</td></tr><tr><td>分数类型</td><td>fractions.Fraction</td><td>保持了分子和分母两个部分，避免了浮点运算的不精确</td></tr></tbody></table>

整数，浮点数和布尔值类型是大家都比较熟悉的，自然不用多说。而后面三种类型大家可能用的不多。这里稍加介绍：

*   复数带有实部和虚部两个部分，虚部的表示是数字和`j`或者`J`作为后缀。例如：`2 + 1j`。
*   小数是精度固定的浮点数，小数通过字符串来初始化，例如：`Decimal('0.00001')`。
*   分数的实现保留了分子和分母两个部分。这样做的好处是避免了浮点运算的不精确性，坏处是性能较差（相对于浮点数）。

数值字面值
-----

我们通常通过数值字面值来初始化数值类型。下面是一些示例：

```
from decimal import Decimal
from fractions import Fraction

int_number = 1234
float_number = 0.1234
bool_number = True
complex_number = 3 + 4j
decimal_number = Decimal('0.1') + Decimal('0.2')
fraction_numer = Fraction(1, 3)

print(int_number, type(int_number))
print(float_number, type(float_number))
print(bool_number, type(bool_number))
print(complex_number, type(complex_number))
print(decimal_number, type(decimal_number))
print(fraction_numer, type(fraction_numer))


```

该代码输出如下：

```
(1234, <type 'int'>)
(0.1234, <type 'float'>)
(True, <type 'bool'>)
((3+4j), <type 'complex'>)
(Decimal('0.3'), <class 'decimal.Decimal'>)
(Fraction(1, 3), <class 'fractions.Fraction'>)


```

进制与转换
-----

对于拥有十个手指的人类来说，10 进制是最自然的。但对于计算机来说，二进制，八进制和十六进制也非常常用。

在 Python 中：

*   `0b`或者`0B`作为前缀表示二进制，后面允许的数字是：0，1
*   `0o`或者`0O`作为前缀表示八进制，后面允许的数字是：0-7
*   `0x`或者`0X`作为前缀表示十六进制，后面允许的数字是：0-9 和 a-f 或 A-F

需要注意的是，不同进制仅仅是书写的不同，作为数值时，用什么进制来书写并没有什么区别。在输出的时候，Python 默认是使用十进制实现。

不过 Python 提供了`bin`、`oct`和`hex`三个函数将数值转换成二进制，八进制和十六进制的字符串。

```
a = 0b111
b = 0o127
c = 0xAb

print('a =', a, 'bin:', bin(a), 'oct:', oct(a), 'hex:', hex(a))
print('b =', b, 'bin:', bin(b), 'oct:', oct(b), 'hex:', hex(b))
print('c =', c, 'bin:', bin(c), 'oct:', oct(c), 'hex:', hex(c))


```

其输出如下：

```
a = 7 bin: 0b111 oct: 0o7 hex: 0x7
b = 87 bin: 0b1010111 oct: 0o127 hex: 0x57
c = 171 bin: 0b10101011 oct: 0o253 hex: 0xab


```

运算
--

与数值类型相关的是一系列的运算。运算涉及函数和运算符。函数包含`pow`，`abs`等，这类函数非常的多，甚至还有很多第三方的开源科学计算库，本文不打算在这方面过多说明，大家可以在网上自行搜索文档。

这里仅列出数值类型常见的运算符，它们如下表所示：

<table><thead><tr><th>运算符</th><th>说明</th></tr></thead><tbody><tr><td>x if y else z</td><td>三元选择表达式</td></tr><tr><td>x or y</td><td>逻辑或</td></tr><tr><td>x and y</td><td>逻辑与</td></tr><tr><td>not x</td><td>逻辑非</td></tr><tr><td>x in y, x not in y</td><td>成员关系，是否存在</td></tr><tr><td>x is y, x is not y</td><td>同一性测试，是否相同</td></tr><tr><td>x &lt;y, x &lt;= y, x&gt; y, x &gt;= y</td><td>大小比较</td></tr><tr><td>x == y, x != y</td><td>值等价性判断</td></tr><tr><td>x | y</td><td>按位或，集合并集</td></tr><tr><td>x ^ y</td><td>按位异或，集合对称差集</td></tr><tr><td>x &amp; y</td><td>按位与，集合交集</td></tr><tr><td>~x</td><td>按位非</td></tr><tr><td>x «&nbsp;y, x&nbsp;» y</td><td>左移和右移操作</td></tr><tr><td>x + y, x - y, x * y</td><td>加法，减法，乘法</td></tr><tr><td>x % y</td><td>求余数</td></tr><tr><td>x / y, x // y</td><td>真除法和向下取整除法</td></tr><tr><td>-x, +x</td><td>取负，取正</td></tr><tr><td>x ** y</td><td>幂运算</td></tr></tbody></table>

链式比较
----

判断一个数字是否在某个区间内是一个非常常见的操作。在其他语言中，这通常需要两个比较式并求与来完成。但在 Python 中，只需要一个表达式即可以完成。因为它支持链式比较。

例如，判断数字是否在`[1, 100]`区间内：

```
a = 55
b = 101

print('1 <= a <= 100: ', 1 <= a <= 100)
print('1 <= b <= 100: ', 1 <= b <= 100)


```

这段代码输出如下：

```
1 <= a <= 100:  True
1 <= b <= 100:  False


```

链式比较可以很长，例如：`5 > 4 > 3 > 2 > 1`。很显然，链式比较要比多个比较式求与要简洁很多。

除法
--

如果你有 C/C++ 或者 Java 语言的编程经验，你会知道，在这些语言中，两个整数相除，其结果仍然是整数，小数部分会被丢弃。这称之为向下取整除法。

在 Python 中，向下取整除法并非`/`符号，而是`//`符号。在 Python 3 中，`/`除法会保留小数部分，这称之为：“真除法”。

```
a = 100
b = 3

print(a, '/', b, '=', a / b)
print(a, '//', b, '=', a // b)


```

其输出如下：

```
100 / 3 = 33.333333333333336
100 // 3 = 33


```

对于从其他语言转向 Python 语言的一定要注意这些细节差异，因为一个数值的差别可能导致整个程序运转出错。

要说编程中最常用的数据类型，除了数字类型之外就一定是字符串类型了。

Python 没有单个字符的数据类型，只有包含一系列字符的字符串类型。

希望读者知道，单就字符串本身其实就是一个非常大的话题，由于篇幅所限，本文仅能提及最基本的内容。

字符串字面值
------

大部分时候我们都会通过字面值来初始化字符串。Python 提供了多种字面值的表达方法。

*   单引号：`'simple string'`
*   双引号：`"It's a tiger"`
*   三引号：`'''...content...'''`，`"""...content..."""`
*   原始字符串格式：`r"C:\folder\file.txt"`
*   字节字面值
*   Unicode 字面值

在 Python 中，单引号和双引号来表达字符串是一样的含义，因此`'string content'`和`"string content"`的值是相等的。另外，在单引号中可以使用双引号，双引号中也可以使用单引号。

```
a = 'string content'
b = "string content"
c = 'He says "hello"'
d = "It's a nice day."

print(a == b)
print(c)
print(d)


```

输出：

```
True
He says "hello"
It's a nice day.


```

三引号用来包含多行文本。因此其中可以包含换行符。例如：

```
'''
STAR
WARS

A long time ago in a galaxy far,
far away...
'''


```

这种写法在想要注释掉多行代码时可能很有用。

最后，原始字符串格式，使得我们不用再对特殊字符转义。这在字符串中包含很多反斜杠时（例如：Windows 文件路径时）会很有用。

字符串的方法
------

如果你查阅 Python 的文档：[String Methods](https://docs.python.org/3/library/stdtypes.html#string-methods) ，就会发现字符串类型包含了非常多的方法。

这里仅列出这些方法，对于它们的详细说明请参阅官方文档。其实很多时候，通过方法名称我们已经可以想到方法的作用。

> 请注意，字符阿是不可变类型，那些试图 “修改” 字符串内容的方法其实是返回了一个新的拷贝。

```
str.capitalize()
str.casefold()
str.center(width[, fillchar])
str.count(sub[, start[, end]])
str.encode(encoding="utf-8", errors="strict")
str.endswith(suffix[, start[, end]])
str.expandtabs(tabsize=8)
str.find(sub[, start[, end]])
str.format(*args, **kwargs)
str.format_map(mapping)
str.index(sub[, start[, end]])
str.isalnum()
str.isalpha()
str.isascii()
str.isdecimal()
str.isdigit()
str.isidentifier()
str.islower()
str.isnumeric()
str.isprintable()
str.isspace()
str.istitle()
str.isupper()
str.join(iterable)
str.ljust(width[, fillchar])
str.lower()
str.lstrip([chars])
static str.maketrans(x[, y[, z]])
str.partition(sep)
str.replace(old, new[, count])
str.rfind(sub[, start[, end]])
str.rindex(sub[, start[, end]])
str.rjust(width[, fillchar])
str.rpartition(sep)
str.rsplit(sep=None, maxsplit=-1)
str.rstrip([chars])
str.split(sep=None, maxsplit=-1)
str.splitlines([keepends])
str.startswith(prefix[, start[, end]])
str.strip([chars])
str.swapcase()
str.title()
str.translate(table)
str.upper()
str.zfill(width)


```

格式化
---

我们常常需要按照特定的格式拼接字符串，这就是字符串格式化的用武之地。

Python 中下面两种格式化写法都很常用：

*   % 形式：`'...%s... % (values)`
*   format 方法：`'...{}...'.format(values)`

第一种写法是将格式化模板与待替换的字符串通过`%`连接。例如：

```
template = 'Star.Wars.Episode.%d.%s.%d'
year = 1999
print(template % (1, 'The.Phantom.Menace', year))


```

它将输出：

```
Star.Wars.Episode.1.The.Phantom.Menace.1999


```

如果你有 C/C++ 编程背景，你可以意识到，这个格式与`printf`非常的类似。确实没错，每个待替换的字符串通过`%`开始，后面在字母表示待替换的字符串格式，例如：`%d`代表整数，`%s`代表字符串，`%f`代表浮点数。详细的格式请参见这里：[String Formatting Operations](https://docs.python.org/2/library/stdtypes.html#string-formatting)。

除了上面这种指定类型的写法，还可以对占位符命名，然后通过一个字典类型来替换：

```
template2 = 'Star.Wars.Episode.%(number)d.%(title)s.%(year)d'
content = {'number': 2, 'title': 'Attack.of.the.Clones', 'year': 2002}
print(template2 % content)


```

它将输出：

```
Star.Wars.Episode.2.Attack.of.the.Clones.2002


```

第二种常用的格式化方法是字符串的`format`函数。例如：

```
template3 = 'Star.Wars.Episode.{}.{}.{}'
print(template3.format(3, 'Revenge.of.the.Sith', 2005))


```

它将输出：

```
Star.Wars.Episode.3.Revenge.of.the.Sith.2005


```

我们看到，格式化模板中的占位符是通过大括号的形式表达的。在`format`函数中，需要按顺序填入待替换的内容。

你也可以在大括号中指定待替换字符串的下标：

```
template4 = 'Star.Wars.Episode.{2}.{1}.{0}'
print(template4.format(1977, 'A.New.Hope', 4))


```

输出：

```
Star.Wars.Episode.4.A.New.Hope.1977


```

另外，你也可以通过命名的方式来指定

```
template5 = 'Star.Wars.Episode.{number}.{title}.{year}'
print(template5.format(number=5, title='The.Empire.Strikes.Back', year=1980))


```

它将输出：

```
Star.Wars.Episode.5.The.Empire.Strikes.Back.1980


```

Python 中的`set`类实现了数学中集合的概念。

集合中的一个元素只会出现一次，无论它被添加了多少次。集合中的元素是无序的。另外还有一个需要注意的是，集合中只能包含不可变对象（例如：数值和字符串），但是不能包含列表和字典，因为它们是可变的。

下面两种初始化方法效果是一样的：

```
set_a = set([1, 2, 3, 4, 5])
set_b = {3, 4, 5, 6, 7}


```

集合可以进行一系列的运算：求并集，求交集，求对称差集等。这些运算可以以方法形式调用，具体请查阅这里：[Python Set Types](https://docs.python.org/3/library/stdtypes.html#set)。

除了方法调用，使用运算符的方式看起来会更简洁：

```
set_a = set([1, 2, 3, 4, 5])
set_b = {3, 4, 5, 6, 7}

print('set a:', set_a)
print('set b:', set_b)
print('----------------------')
print('a - b:', set_a - set_b)
print('a ^ b:', set_a ^ set_b)
print('a & b:', set_a & set_b)
print('a | b:', set_a | set_b)


```

这段代码输出如下：

```
set a: {1, 2, 3, 4, 5}
set b: {3, 4, 5, 6, 7}
----------------------
a - b: {1, 2}
a ^ b: {1, 2, 6, 7}
a & b: {3, 4, 5}
a | b: {1, 2, 3, 4, 5, 6, 7}


```

列表是多个对象的集合。它可以包含不同类型的对象，并且可以嵌套。与字符串不同的是，列表是可变对象，你可以直接在原位置修改它。

列表通过`[]`或者`list()`初始化：

```
list_a = [1, 2, 3, 4, 5]
list_b = list('a', 'b', 'c', 'd', 'e')


```

<table><thead><tr><th>操作</th><th>说明</th></tr></thead><tbody><tr><td>L1 + L2</td><td>拼接</td></tr><tr><td>L * 3</td><td>重复</td></tr><tr><td><code>L.append(1)</code></td><td>尾部添加</td></tr><tr><td><code>L.extend([2, 3, 4])</code></td><td>尾部扩展</td></tr><tr><td><code>L.insert(i, X)</code></td><td>为位置 i 插入值 X</td></tr><tr><td><code>L.index(X)</code></td><td>返回 X 的索引位置</td></tr><tr><td><code>L.count(X)</code></td><td>统计 X 出现的次数</td></tr><tr><td><code>L.sort()</code></td><td>排序</td></tr><tr><td><code>L.reverse()</code></td><td>反转</td></tr><tr><td><code>L.copy()</code></td><td>复制</td></tr><tr><td><code>L.clear()</code></td><td>清除所有元素</td></tr><tr><td><code>L.pop(i)</code></td><td>删除 i 处元素，并返回</td></tr><tr><td><code>L.remove(X)</code></td><td>删除元素 X</td></tr><tr><td><code>del L[i]</code></td><td>删除 i 处元素</td></tr><tr><td><code>del L[i:j]</code></td><td>删除 i 到 j 处元素</td></tr><tr><td><code>L[i:j] = []</code></td><td>删除 i 到 j 处元素</td></tr><tr><td><code>L[i] = 3</code></td><td>索引赋值</td></tr></tbody></table>

下面是一段代码示例：

```
list_a = [1, 2, 3, 4, 5]
print('a:', list_a)

list_a.append(1);
print('after append 1, a:', list_a)

list_a.extend([1,2,3])
print('after extend [1, 2, 3], a:', list_a)

print("index of '1' in a:", list_a.index(1))

list_a.sort()
print('after sort, a: ', list_a)

list_a.reverse()
print('after reverse, a:', list_a)

list_a.remove(1)
print('after remove(1), a:', list_a)

del list_a[1:3]
print('after del a[1:3], a:', list_a)

list_b = list('abcde')
print('b:', list_b)
list_c = list_a + list_b
print('a + b:', list_c)


```

它的输出如下：

```
a: [1, 2, 3, 4, 5]
after append 1, a: [1, 2, 3, 4, 5, 1]
after extend [1, 2, 3], a: [1, 2, 3, 4, 5, 1, 1, 2, 3]
index of '1' in a: 0
after sort, a:  [1, 1, 1, 2, 2, 3, 3, 4, 5]
after reverse, a: [5, 4, 3, 3, 2, 2, 1, 1, 1]
after remove(1), a: [5, 4, 3, 3, 2, 2, 1, 1]
after del a[1:3], a: [5, 3, 2, 2, 1, 1]
b: ['a', 'b', 'c', 'd', 'e']
a + b: [5, 3, 2, 2, 1, 1, 'a', 'b', 'c', 'd', 'e']


```

字典（Dictionary）也是软件开发中非常常用的类型。这种类型在其他编程语言中通常叫做 Map。

无论是 Dictionary 还是 Map，它们都是 “键：值” 对的方式存储数据。

与集合还有列表类似，字典也有两种初始化方式：`{}`或者`dict()`。如果你有两个一一对应的数组，你可以使用`zip`函数将它们映射成字典。

字典中的键和值可以任意添加，对同样的键赋值将覆盖原先的值。但需要注意的是，字典中的键并不存在什么特定的顺序，如果有需要，你要另外对键进行排序。字典的值可以是其他任何数据类型，甚至是另一个字典或者列表，这意味着字典类型支持嵌套。

字典的常见操作如下：

<table><thead><tr><th>操作</th><th>说明</th></tr></thead><tbody><tr><td><code>dic.keys()</code></td><td>返回所有键</td></tr><tr><td><code>dic.values()</code></td><td>返回所有值</td></tr><tr><td><code>dic.items()</code></td><td>所有 “键 + 值” 元组</td></tr><tr><td><code>dic.copy()</code></td><td>复制</td></tr><tr><td><code>dic.clear()</code></td><td>清除所有条目</td></tr><tr><td><code>dic.get(key, default?)</code></td><td>通过键获取，如果不存在返回 default 值</td></tr><tr><td><code>dic.pop(key, default?)</code></td><td>通过键删除，如果不存在返回 default 值</td></tr><tr><td><code>dic.popitem()</code></td><td>删除并返回所有键值对</td></tr><tr><td><code>dic[key] = 42</code></td><td>通过键赋值</td></tr><tr><td><code>del D[key]</code></td><td>根据键删除条目</td></tr></tbody></table>

下面是一段代码示例：

```
paul = dict(Name='Paul', Hobby=['Music', 'Photograph'], Job='Developer')
moira = dict(zip(['Name', 'Hobby', 'Job'], ['Moira', 'Movie', 'Student']))

family = {'People': [paul, moira], 'City': 'Nanjing'}


print('paul.keys:', paul.keys())
print('paul.values:', paul.values())
print('paul.City:', paul.get('City', 'Nanjing'))

print('Family, city:', family['City'])

for p in family['People']:
	print('\t', p)

print("Moira's hobby is:", moira['Hobby'])

paul['Hobby'] = ['Rock', 'Computer']
print('paul:', paul)
del paul['Hobby']
print('paul:', paul)


```

这段代码输出如下：

```
paul.keys: dict_keys(['Name', 'Hobby', 'Job'])
paul.values: dict_values(['Paul', ['Music', 'Photograph'], 'Developer'])
paul.City: Nanjing
Family, city: Nanjing
   {'Name': 'Paul', 'Hobby': ['Music', 'Photograph'], 'Job': 'Developer'}
   {'Name': 'Moira', 'Hobby': 'Movie', 'Job': 'Student'}
Moira's hobby is: Movie
paul: {'Name': 'Paul', 'Hobby': ['Rock', 'Computer'], 'Job': 'Developer'}
paul: {'Name': 'Paul', 'Job': 'Developer'}


```

最后，我们还要学习的一种类型叫做元组（Tuple）。

元组也是对象的组合。它其中包含的元素是有序的，通过下标访问其中的元素。它可以包含不同类型的元素。

元组与列表有些相似，但最大的不同在于元组是不可变对象，这意味着你不能原地修改它。

元组可以通过`()`或者`tuple`初始化。

元组常见的操作如下：

<table><thead><tr><th>操作</th><th>说明</th></tr></thead><tbody><tr><td>t1 + t2</td><td>拼接</td></tr><tr><td>t * 3</td><td>重复</td></tr><tr><td>t.count(‘a’)</td><td>返回元素出现次数</td></tr><tr><td>t.index(‘a’)</td><td>返回第一次找到的元素下标</td></tr></tbody></table>

下面是一个代码示例：

```
t1 = (1, '2', (3, 4))
t2 = tuple([1, '2', [3,4]])
t3 = t1 + t2

print('len(t3): ', len(t3))
print('t3.count(1): ', t3.count(1))
print("t3.index('2'): ", t3.index('2'))


```

结果如下：

```
len(t3):  6
t3.count(1):  2
t3.index('2'):  1


```

对于字符串，集合，列表，字典和元组几种包含了多个元素的数据类型来说，它们有一些共同的操作：

*   通过`in`确认成员关系。注意：对于字典来说，`in`是指键所组成的集合。
*   通过`for x in y`来遍历其中的元素。
*   来遍历的时候，你可以使用表达式创建新的数据。

```
t = (1, '2', 3.0)
l = [1, 2, 3.0]
s = {1, '2', 3.0}
d = {'Name': 'Paul', 'Hobby': ['Music', 'Photograph']}
str = 'Attack.of.the.Clones'

print('1 in', t, ':', 1 in t)
print('1 in', l, ':', 1 in l)
print('1 in', s, ':', 1 in s)
print('"Name" in', d, ':', 'Name' in d)
print('"Attack" in', str, ':', "Attack" in str)

print('\nelements in list:')
for x in l:
	print('\t', x)
print('\n')

new_l = [x ** 2 for x in l]
print('new_l:', new_l)


```

这段代码输出如下：

```
1 in (1, '2', 3.0) : True
1 in [1, 2, 3.0] : True
1 in {1, 3.0, '2'} : True
"Name" in {'Name': 'Paul', 'Hobby': ['Music', 'Photograph']} : True
"Attack" in Attack.of.the.Clones : True

elements in list:
	 1
	 2
	 3.0


new_l: [1, 4, 9.0]


```

对于有序集合（字符串，列表，元组）来说，它们都支持被称之为索引和分片的操作方式。

> 对于字典来说，它仅支持通过键值索引操作，但不支持分片。

索引是通过下标来访问指定位置的元素。这种操作在其他语言中也很常见，例如 Java 和 C++。但 Python 的索引不仅支持 0 和正整数，它还支持负整数，例如：`str[-2]`，这意味着访问倒数第二个元素。一般的，对于一个有序集合 l 来说，`l[-n]`意味着倒数第 n 个元素，即：`l[len(l)-n]`。

另外，分片（slice）就是 Python 所特有的操作了。分片支持下面这样的语法：

*   `l[:]`是指获取集合 l 中的所有元素
*   `l[n:m]`是指获取`[n, m)`范围内的元素
*   `l[:9]`是指获取`[0,9)`范围内的元素
*   `l[n:]`是指获取第 n 个开始到末尾的元素
*   `l[:-n]`是指获取`[0, len(l)-n)`范围内的元素
*   `l[n:m:d]`是指获取`[n, m)`范围内的元素，但是每隔 d 次取一个元素

这样说起来很费劲，但是如果直接看代码可能就很好理解了：

```
str = "Star.Wars.Episode.4.A.New.Hope.1977"

print('str[:]:', str[:])
print('str[5:9]:', str[5:9])
print('str[:9]:', str[:9])
print('str[5:]:', str[5:])
print('str[:-8]:', str[:-16])
print('str[::2]', str[::2])


```

这段代码输出如下：

```
str[:]: Star.Wars.Episode.4.A.New.Hope.1977
str[5:9]: Wars
str[:9]: Star.Wars
str[5:]: Wars.Episode.4.A.New.Hope.1977
str[:-8]: Star.Wars.Episode.4
str[::2] Sa.asEioe4ANwHp.97


```

尽管在很多时候我们不用区分变量（variable）与对象（object）的区别，但如果搞懂它们将对你分析一些微妙问题有很大的帮助。

Python 与其他语言有一个很大的区别是：Python 中的变量不需要声明类型。实际上，变量本身并不存在类型，类型是对象才有的属性。

简单来说，变量就像是一个手柄，它是一个有实际名称的实体，但是它并不存储值。对象才是实际存储值的实体，你需要通过变量来使用对象。

当你定义一个变量并指定值时，Python 实际上是创建了一个对象来存储值，然后将变量指向它。如下图所示：

![](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-04-python-built-in-types/obj_var_1.png)

如果接下来你又创建了另外一个变量并通过原先的变量赋值，那么此时两个变量将指向同一个对象：

![](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-04-python-built-in-types/obj_var_2.png)

接下来，如果你又对 b 进行了一次新的赋值，那么它们将指向不同的对象：

![](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-04-python-built-in-types/obj_var_3.png)

上面我们看到的是不可变的数字对象。因此我们只能对变量重新赋值，而不能修改它。

当如果两个变量指向同一个可变对象，情况就会不一样：

![](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-04-python-built-in-types/obj_var_4.png)

此时，如果通过任何一个变量修改对象的值，因为这个对象在两个变量之间共享，因此通过另外一个变量读取值时也会反映出来。

![](https://paul-pub.oss-cn-beijing.aliyuncs.com/2020/2020-04-python-built-in-types/obj_var_5.png)

> 如果你熟悉 C/C++ 中的指针，你会发现这和指针非常的相似，只不过我们没有使用`*`符号。

== 和 is
-------

有了上面的铺垫，理解`==`和`is`就很容易了：

*   `==`是判断两个对象的值是否相等
*   `is`是判断两个变量是否指向同一个对象

因此，下面这段代码，两个判断结果都是`True`。

```
a = 100
b = a;
print('a==b:', a==b)
print('a is b:', a is b)


```

如果接着将 b 赋值 100 会怎么样？它会指向一个新的对象吗？

```
b = 100
print('a==b:', a==b)
print('a is b:', a is b)


```

答案是否定的，这里两个判断仍然都将是`True`。因为同一个数字对象在运行时会被共享。

如果是可变对象呢：

```
a = [1, 2, 3]
b = a
print('a==b:', a==b)
print('a is b:', a is b)

a[2] = 100
print('a:', a)
print('b:', b)

b = [1, 2, 100]
print('a==b:', a==b)
print('a is b:', a is b)


```

毫无疑问，这里的第一处两个判断都将是`True`。因此当进行`a[2] = 100`操作时，对`b`也会有影响。

不过，如果重新对 b 进行赋值：`b = [1, 2, 3]`，它将指向一个新的对象，因此最后的判断`a==b`将一次返回`True`，`a is b`将返回`False`。

上面这段代码将输出如下：

```
a==b: True
a is b: True
a: [1, 2, 100]
b: [1, 2, 100]
a==b: True
a is b: False


```

赋值与复制
-----

*   [Python Built-in Types](https://docs.python.org/3/library/stdtypes.html)
*   [Learning Python Fifth Edition](https://www.amazon.com/-/zh/dp/1449355730/)
*   [Fluent Python](https://www.amazon.com/-/zh/dp/1491946008/)
*   [High Performance Python: Practical Performant Programming for Humans](https://www.amazon.com/-/zh/dp/1449361595/)
*   [Think Python: How to Think Like a Computer Scientist](https://www.amazon.com/-/zh/dp/1491939362/)
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.zhihu.com](https://www.zhihu.com/question/19698598) ![](https://pic2.zhimg.com/ca6bf4bb82d9cf152b618edcbda01606_xs.jpg?source=1940ef5c)刘志军

**print**
---------

在进行程序调试时用得最多的语句可能就是 print，在 Python 2 中，print 是一条语句，而 Python3 中作为函数存在。有人可能就有疑问了，我在 Python2 中明明也看到当函数使用：

```
# py2
print("hello")  # 等价 print  ("hello")

#py3
print("hello")


```

然而，你看到的只是表象，那么上面两个表达式有什么区别？从输出结果来看是一样的，但本质上，前者是把 ("hello") 当作一个整体，而后者 print() 是个函数，接收字符串作为参数。

```
# py2
>>> print("hello", "world")
('hello', 'world')

# py3
>>> print("hello", "world")
hello world


```

这个例子更明显了，在 py2 中，print 语句后面接的是一个元组对象，而在 py3 中，print 函数可以接收多个位置参数。如果希望在 Python2 中 把 print 当函数使用，那么可以导入 future 模块 中的 print_function

```
# py2
>>> print("hello", "world")
('hello', 'world')
>>> 
>>> from __future__ import print_function
>>> print("hello", "world")
hello world


```

**编码**
------

Python2 的默认编码是 asscii，这也是导致 Python2 中经常遇到编码问题的原因之一，至于是为什么会使用 asscii 作为默认编码，原因在于 Python 这门语言诞生的时候还没出现 Unicode。Python 3 默认采用了 UTF-8 作为默认编码，因此你不再需要在文件顶部写 # coding=utf-8 了。

```
# py2
>>> sys.getdefaultencoding()
'ascii'

# py3
>>> sys.getdefaultencoding()
'utf-8'


```

网上不少文章说通过修改默认编码格式来解决 Python2 的编码问题，其实这是个大坑，不要这么干。

**字符串**
-------

字符串是最大的变化之一，这个变化使得编码问题降到了最低可能。在 Python2 中，字符串有两个类型，一个是 unicode，一个是 str，前者表示文本字符串，后者表示字节序列，不过两者并没有明显的界限，开发者也感觉很混乱，不明白编码错误的原因，不过在 Python3 中两者做了严格区分，分别用 str 表示字符串，byte 表示字节序列，任何需要写入文本或者网络传输的数据都只接收字节序列，这就从源头上阻止了编码错误的问题。

![](https://pic3.zhimg.com/50/v2-522b62a41136e87f742cc66fa7d5ca8e_720w.jpg?source=1940ef5c)

**True 和 False**

True 和 False 在 Python2 中是两个全局变量（名字），在数值上分别对应 1 和 0，既然是变量，那么他们就可以指向其它对象，例如：

```
# py2
>>> True = False
>>> True
False
>>> True is False
True
>>> False = "x"
>>> False
'x'
>>> if False:
...     print("?")
... 
?


```

显然，上面的代码违背了 Python 的设计哲学 _Explicit is better than implicit._。而 Python3 修正了这个缺陷，True 和 False 变为两个关键字，永远指向两个固定的对象，不允许再被重新赋值。

```
# py3
>>> True = 1
  File "<stdin>", line 1
SyntaxError: can't assign to keyword


```

**迭代器**
-------

在 Python2 中很多返回列表对象的内置函数和方法在 Python 3 都改成了返回类似于迭代器的对象，因为迭代器的惰性加载特性使得操作大数据更有效率。Python2 中的 range 和 xrange 函数合并成了 range，如果同时兼容 2 和 3，可以这样：

```
try:
    range = xrange
except:
    pass


```

另外，字典对象的 dict.keys()、dict.values() 方法都不再返回列表，而是以一个类似迭代器的 "view" 对象返回。高阶函数 map、filter、zip 返回的也都不是列表对象了。Python2 的迭代器必须实现 next 方法，而 Python3 改成了 __next__

**nonlocal**
------------

我们都知道在 Python2 中可以在函数里面可以用关键字 global 声明某个变量为全局变量，但是在嵌套函数中，想要给一个变量声明为非局部变量是没法实现的，在 Pyhon3，新增了关键字 nonlcoal，使得非局部变量成为可能。

```
def func():
    c = 1
    def foo():
        c = 12
    foo()
    print(c)
func()    #1


```

可以对比上面两段代码的输出结果

```
def func():
    c = 1
    def foo():
        nonlocal c
        c = 12
    foo()
    print(c)
func()   # 12


```

其实很多内建模块也做了大量调整，Python3 中的模块组织更加清晰，类更加先进，还引入了异步 IO，先写这么多

------- 更新 -------

多谢知友

@YFdyh 指出，py2 出现的时候其实已经有了 unicode 统一编码了，只不过 py2 为了向后兼容还是沿用了 py1.x 的设计逻辑

![](https://pic2.zhimg.com/5c772dde6_xs.jpg?source=1940ef5c)王猫猫我来更正及评论下.  
> 1. print 不再是语句，而是函数，比如原来是 print 'abc' 现在是 print('abc')  
但是 python2.6+ 可以使用 from __future__ import print_function 来实现相同功能  
> 2. 在 Python 3 中，没有旧式类，只有新式类，也就是说不用再像这样 class Foobar(object): pass 显式地子类化 object  
但是最好还是加上. 主要区别在于 old-style 是 classtype 类型而 new-style 是 type 类型  
> 3. 原来 1/2（两个整数相除）结果是 0，现在是 0.5 了  
python 2.2+ 以上都可以使用 from __future__ import division 实现改特性, 同时注意 // 取代了之前的 / 运算  
> 4. 新的字符串格式化方法 format 取代 %  
错误, 从 python2.6+ 开始已经在 str 和 unicode 中有该方法, 同时 python3 依然支持 % 算符  
> 6. xrange 重命名为 range  
同时更改的还有一系列内置函数及方法, 都返回迭代器对象, 而不是列表或者 元组, 比如 filter, map, dict.items 等  
> 7. != 取代 < >  
python2 也很少有人用 < > 所以不算什么修改  
> 8. long 重命名为 int  
不完全对, python3 彻底废弃了 long+int 双整数实现的方法, 统一为 int , 支持高精度整数运算.  
> 9. except Exception, e 变成 except (Exception) as e  
只有 python2.5 及以下版本不支持该语法. python2.6 是支持的. 不算新东西  
> 10. exec 变成函数  
类似 print() 的变化, 之前是语句.

简单补充下  
* 主要是类库的变化, 组织结构变了些. 但功能没变. urlparse - > urllib.parse 这样的变化  
* 最核心的变化它没有说, 对 bytes 和 原生 UNICODE 字符串的支持, 删除了 unicode 对象, str 为原生 unicode 字符串, bytes 替代了之前的 str 这个是最核心的.  
* 其它... 貌似不怎么重要了. ![](https://pic2.zhimg.com/v2-2943114321273ccfe238e005608b252a_xs.jpg?source=1940ef5c)hresh

**前言**
------

Python 的 3.0 版本，常被称为 Python 3000，或简称 Py3k。相对于 Python 的早期版本，这是一个较大的升级。为了不带入过多的累赘，Python3 在设计的时候没有考虑向下相容。许多针对早期 Python 版本设计的程式都无法在 Python3 上正常执行。

为了照顾现有程式，Python 2.6 作为一个过渡版本，基本使用了 Python2 的语法和库，同时考虑了向 Python3 的迁移，允许使用部分 Python3 的语法与函数。

随着 Python 语言市场份额的增长，第三方库都正在努力地相容 Python 3.x 版本，所以如果想要学习 Python 的朋友们，最好从 Python3 开始学起，虽然 Python2 慢慢将淡出市场，但是我们还是有必要了解两者之间的区别。

**区别**
------

**1.print 函数**

print 语句没有了，取而代之的是 print() 函数。

```
Old: print "The answer is", 2*2
New: print("The answer is", 2*2)
​
Old: print x,           # Trailing comma suppresses newline
New: print(x, end=" ")  # Appends a space instead of a newline
​
Old: print              # Prints a newline
New: print()            # You must call the function!
​
Old: print >>sys.stderr, "fatal error"
New: print("fatal error", file=sys.stderr)
​
Old: print (x, y)       # prints repr((x, y))
New: print((x, y))      # Not the same as print(x, y)!

```

**2. 整除**

Python2 中 / 的结果是整型，Python3 中是浮点类型。

Python2:

```
>>> 1 / 2
0
>>> 1.0 / 2.0
0.5

```

Python3:

```
>>> 1/2
0.5

```

**3.Unicode**

Python2 默认编码是 ASCII，在使用 Python2 的过程中经常会遇到编码问题，当时因为 Python 语言还没使用 Unicode，所以使用 ASCII 作为默认编码。Python3 默认编码是 Unicode(utf-8)，也就不需要在文件头部写 # coding=utf-8。

Python2：

```
>>> str = "我爱北京天安门"
>>> str
'\xe6\x88\x91\xe7\x88\xb1\xe5\x8c\x97\xe4\xba\xac\xe5\xa4\xa9\xe5\xae\x89\xe9\x97\xa8'
>>> sys.getdefaultencoding()
'ascii'

```

Python3：

```
>>> str = "我爱北京天安门"
>>> str
'我爱北京天安门'
>>> sys.getdefaultencoding()
'utf-8'

```

**4. 迭代器**

在 Python2 中很多返回列表对象的内置函数和方法在 Python3 都改成了返回类似于迭代器的对象，因为迭代器的惰性加载特性使得操作大数据更有效率。Python2 中的 xrange 在 Python3 中重命名为 range。

Python2：

```
>>> [x for x in xrange(10)]
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

```

Python3:

```
>>> [x for x in range(10)]
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> [x for x in xrange(10)]
​
NameError                                 Traceback (most recent call last)
<ipython-input-2-4ecef59b8e55> in <module>()
----> 1 [x for x in xrange(10)]
​
NameError: name 'xrange' is not defined
​

```

另外，字典对象的 dict.keys()、dict.values() 方法都不再返回列表，而是以一个类似迭代器的 "view" 对象返回。 Python2：

```
>>>dic1={'a':1,'b':2,'c':3,'d':4,'e':5}
>>>dic1.keys()
>>>dic1.values()
['a', 'c', 'b', 'e', 'd']
[1, 3, 2, 5, 4]

```

Python3:

```
>>>dic1={'a':1,'b':2,'c':3,'d':4,'e':5}
>>>dic1.keys()
>>>dic1.values()
dict_keys(['a', 'b', 'c', 'd', 'e'])
dict_values([1, 2, 3, 4, 5])

```

高阶函数 map、filter、zip 返回的也都不是列表对象了，对于比较高端的 reduce 函数，它在 Python 3.x 中已经不属于 built-in 了，被挪到 functools 模块当中。

Python2：

```
#类型是 built-in function(内置函数)
>>> map
<built-in function map>
>>> filter
<built-in function filter>
#输出的结果类型都是列表
>>> map(lambda x:x *2, [1,2,3])
[2, 4, 6]
>>> filter(lambda x:x %2 ==0,range(10))
[0, 2, 4, 6, 8]

```

Python3：

```
>>> map
<class 'map'>
>>> map(print,[1,2,3])
<map object at 0x10d8bd400>
>>> filter
<class 'filter'>
>>> filter(lambda x:x % 2 == 0, range(10))
<filter object at 0x10d8bd3c8>
​

```

map、filter 从函数变成了类，其次，它们的返回结果也从当初的列表成了一个可迭代的对象, 我们尝试用 next 函数来进行手工迭代。

```
>>> f =filter(lambda x:x %2 ==0, range(10))
>>> next(f)
0
>>> next(f)
2

```

**5. 不等运算符**

Python2 中不等于有两种写法 != 和 <>；Python3 中去掉了 <>, 只有 != 一种写法，我并未使用过 <>。

**6. 数据类型**

Python3 去除了 long 类型，现在只有一种整型——int，但它的行为就像 Python2 版本的 long；

新增了 bytes 类型，对应于 Python2 版本的八位串

**7.True 和 False**

True 和 False 在 Python2 中是两个全局变量（名字），在数值上分别对应 1 和 0，既然是变量，那么就可以重新指向其它对象。

```
>>> False = '1'
>>> False
'1'
>>> if False:
...     print("?")
... 
?

```

Python3 修正了这个缺陷，True 和 False 变为两个关键字，永远指向两个固定的对象，不允许再被重新赋值。

```
>>> True = 1
  File "<stdin>", line 1
  SyntaxError: can't assign to keyword

```

**7.nonlocal**

global 适用于函数内部修改全局变量的值，但是在嵌套函数中，想要给一个变量声明为非局部变量是没法实现的，在 Python3 中，新增了关键字 nonlcoal，使得非局部变量成为可能。

Python2：

```
def func():
    c = 1
    def foo():
        nonlocal c
        c = 12
    foo()
    print(c)
func() 
 File "<ipython-input-10-441055e26d24>", line 4
    nonlocal c
             ^
SyntaxError: invalid syntax

```

Python3：

```
def func():
    c = 1
    def foo():
        nonlocal c
        c = 12
    foo()
    print(c)
func() #12

```

**8. 通过 input() 解析用户的输入**

在 python2 中 raw_input() 和 input()，两个函数都存在，其中区别为：raw_input()--- 将所有输入作为字符串看待，返回字符串类型； input()----- 只能接收 "数字" 的输入，在对待纯数字输入时具有自己的特性，它返回所输入的数字的类型（int, float ）。

在 python3 中 raw_input() 和 input() 进行了整合，去除了 raw_input()，仅保留了 input() 函数，其接收任意任性输入，将所有输入默认为字符串处理，并返回字符串类型。

**9. 异常**

except 关键字的使用在 Python 3 中处理异常也轻微的改变了，在 Python 3 中我们现在使用 as 作为关键词。

Python2：

```
print 'Python', python_version()
try:
    let_us_cause_a_NameError
except NameError, err:
    print err, '--> our error message'
​
​
Python 2.7.6
name 'let_us_cause_a_NameError' is not defined --> our error message
​

```

Python3：

```
print('Python', python_version())
try:
    let_us_cause_a_NameError
except NameError as err:
    print(err, '--> our error message')
​
​
Python 3.4.1
name 'let_us_cause_a_NameError' is not defined --> our error message
​

```

raise 语句使用逗号将抛出对象类型和参数分开，3.x 取消了这种奇葩的写法，直接调用构造函数抛出对象即可。

**9.For 循环变量和全局命名空间泄漏**

在 Python3 中 for 循环变量不会再导致命名空间泄漏。

```
"列表推导不再支持 [... for var in item1, item2, ...] 这样的语法。使用 [... for var in (item1, item2, ...)] 代替。也需要提醒的是列表推导有不同的语义：  他们关闭了在 `list()` 构造器中的生成器表达式的语法糖, 并且特别是循环控制变量不再泄漏进周围的作用范围域."

```

Python2：

```
print 'Python', python_version()
​
i = 1
print 'before: i =', i
​
print 'comprehension: ', [i for i in range(5)]
​
print 'after: i =', i
​
​
Python 2.7.6
before: i = 1
comprehension:  [0, 1, 2, 3, 4]
after: i = 4

```

Python3：

```
print('Python', python_version())
​
i = 1
print('before: i =', i)
​
print('comprehension:', [i for i in range(5)])
​
print('after: i =', i)
​
​
Python 3.4.1
before: i = 1
comprehension: [0, 1, 2, 3, 4]
after: i = 1

```

**10. 继承**

```
class A:
     def __init__(self):
         print("A")
​
class B(A):
     pass
​
​
class C(A):
    def __init__(self):
        print("C")
​
class D(B,C):
    pass
​
d1 = D()

```

Python2 结果为 A，Python3 结果为 C。

python2 的继承顺序是 D -> B -> A -> C 深度优先 python3 的继承顺序是 D -> B -> C -> A 广度优先

![](https://pic2.zhimg.com/v2-1fc6909076f644cfa47b9fb4df6442f0_xs.jpg?source=1940ef5c)景略集智

大约 20 年前，2000 年 10 月，Python 2.0 终于看到了曙光。此后 Python 取得了很多重要进展，Python 2 的几个版本陆续发布。经过长时间测试后，2008 年 12 月，Python 3 面世，而它和 Python 2 并不兼容。

正因为这个原因，开发者们不得不做出一个艰难的决定：

1.  要么继续用 Python 2 编写当前项目，不使用新的 Python 3；
2.  要么开始用新的 Python 3，但要将整个项目重新梳理，确保一切正常运行。

![](https://pic1.zhimg.com/50/v2-c20b4b24f3a5d2d25afc913dc1d5f274_720w.jpg?source=1940ef5c)

大部分网站和项目最终选择转向 Python 3，整体上这也和 Python 的开发趋势一致，因为 Python 官方也宣布 2020 年后不再继续支持 Python 2.7。

然而，有些项目仍然在用 Python 2，所以对它们而言，如何将代码从 Python 2 转为 Python 3 是个很重要的议题。本文我们会谈谈 Python 2 和 Python 3 的一些区别，**更重要的是，向大家介绍一些将 Python 2 代码转到 Python 3 兼容代码的方法**。

**Python 2 和 Python 3 的区别**

首先，我们说说 Python 2 和 Python 3 之间的一些区别，瞅瞅哪些方面会给我们带来问题。

在 Python 3 中，print 是个函数，而在 Python 2 中它是个语句，这样两者就有了不同的编写风格和工作原理。比如：

```
Python2: print "The answer is", 2*2
Python3: print("The answer is", 2*2)

```

```
Python2: print x, 
Python3: print(x, end=" ") 

```

```
Python2: print 
Python3: print() 

```

```
Python2: print >>sys.stderr, "fatal error"
Python3: print("fatal error", file=sys.stderr)

```

一些常见的方法在 Python 3 中不再返回列表，比如字典方法 dict.keys()，dict.items() 和 dict.values()，而是返回‘views’。例如，k = d.keys()；k.sort() 已经行不通了。如果我们还是这么做，就会得到报错：

```
AttributeError: 'dict_keys' object has no attribute 'sort'

```

相反，我们需要使用：k = sorted(d)。

之前经常使用的 dict.iterkeys()，dict.iteritems() 和 dict.itervalues() 方法不再支持。如果我们看看 Python 2 和 3 中的字典方法，会发现差异巨大。

Python 2.7 中的字典方法：

```
In [19]: d = {1:100, 2:200, 3:300}

In [20]: d.
d.clear d.get d.iteritems d.keys d.setdefault
d.viewitems d.copy d.has_key d.iterkeys d.pop
d.update d.viewkeys d.fromkeys d.items d.itervalues
d.popitem d.values d.viewvalues

```

Python 3 中的字典方法：

```
In [21]: d = {1:100, 2:200, 3:300}

In [22]: d.
 clear() get() pop() update()
 copy() items() popitem() values()
 fromkeys() keys() setdefault()

```

在 Python 3 中，map() 和 filter() 返回迭代器而不是列表。如果你真的需要列表，可以用 list(map(...))，但通常最好的解决办法是用列表生成器（特别是原始代码使用拉姆达表达式时），或者也可以重写代码，也就不需要这样一个列表。

在 Python 3 中，不再使用 xrange()，代之以 range()，但能处理任何大小的值。

zip() 会返回一个迭代器。

Python 3 也简化了比较运算符的规则。

如果运算对象的数据类型不相同，比较运算符 (<, <=,> =,>) 会导致 TypeError 异常，这样 1 < '', 0 > None 或 len <= len 等类型的表达式不再有效，比如 None < None 会调用 TypeError 而不返回 False。因而在 Python 3 中只有同一数据类型的对象可以比较。注意，这一规则并不适用于比较运算符 === 和 !=，不能比较的类型的对象始终不相等。

Bulltin.sorted() 和 list.sort() 不再接受 cmp 参数，而是提供了一个比较函数。相反我们使用关键字参数。

同样，一些内置函数（有些被移除）、多函数参数传递、类的创建、使用正则表达式等等也做了改动。

那么对于这些新变动，我们该怎么应对呢？下面分享几种方法，帮大家把 Python 2 写的代码转为 Python 3 能兼容的类型。

**将 Python 2 代码转为 Python 3 兼容代码**

**1. __future__ imports**

**from __future__ import division**

在 Python 2 中，除法的工作规则如下：如果你用一个整数除以另一个整数，结果也会是一个整数。比如，3/2=1，而不是我们通常认为的 1.5。这是因为在 Python 2 中，要想得到一个分数结果，你必须明确将数字类型指定为浮点数：3/2.0 = 1.5.

这就为我们编程带来了不便，而在 Python 3 中除法规则和我们通常认为的保持一致——不管除数和被除数是整数也好，小数也罢，结果不再四舍五入为整数，而是 3/2=1.5。另一方面，不管是什么样的除法，结果都会是浮点类型，即便并不需要。例如：4/2 = 2.0。

可以在 Python 官网的 PEP 238 部分查看该导入模块的用法：

[https://www.python.org/dev/peps/pep-0238/](https://link.zhihu.com/?target=https%3A//www.python.org/dev/peps/pep-0238/)

**from __future__ import print_function**

我们前面说过了 print 函数 / 运算符在 Python 2 和 3 中的区别。该导入（在 Python 2 脚本中写在开头部分）会将 print 导入为函数，让你像 Python 3 写的正常程序中以同样的方式使用它。

更多用法详情可查看 Python 官网的 PEP 3105 部分：

[https://www.python.org/dev/peps/pep-3105/](https://link.zhihu.com/?target=https%3A//www.python.org/dev/peps/pep-3105/)

如果想让 Python 2 编写的程序像 Python 3 写的程序那样导入模块 / 包 / 库，可以使用上面这行代码。

可以参看 Python 官网的 PEP 328 部分了解它的更多功能：

[https://www.python.org/dev/peps/pep-0328/](https://link.zhihu.com/?target=https%3A//www.python.org/dev/peps/pep-0328/)

2. **Unicode literals 模块**

**from __future__ import unicode_literals**

和 Python 3 中字符串总为字符串不同，Python 2 中有两种类型的字符串—— str 和 uncode：

In [1]: line = 'test'

In [2]: line2 = u'тест'

Python 3 只使用 ASCII 字符，而 Python 2 使用所有可能的字符，比如西里尔字母和象形文字等等。

要想能像 Python 3 中一样使用单一字符串类型，我们可以导入 unicode_literals 模块，其具体的功能可查看 Python 官网 PEP 3112 部分：

[https://www.python.org/dev/peps/pep-3112/](https://link.zhihu.com/?target=https%3A//www.python.org/dev/peps/pep-3112/)

如果需要确保后端也兼容 Python 3 编写的程序，可以使用 u’some text’格式，第一个引号前加上前缀'u'。

可以阅读官方文档查看 _future_library 的更多信息：

[https://docs.python.org/3/library/__future__.html](https://link.zhihu.com/?target=https%3A//docs.python.org/3/library/__future__.html)

3. **Min/max 函数**

在 Python 3 中，如果想对列表排序或找到最大 / 最小值，所有的元素必须可以比较。如果你原来的代码是 Python 2 写的，里面有包含 None 元素的列表，那么换到 Python 3 时就可能会出现一些问题。那么可以用 min/max 函数来解决这种冲突。

```
def listmin(L):
 '''
 Returns min of an iterable L,
 Ignoring null (None) values.
 If all values are null, returns None
 '''
 values = [v for v in L if v is not None]
return min(values) if values else None

```

也可以写一个相似的函数来确定最大元素。

4. **虚拟变量**

在使用由不同版本 Python 编写的代码时，还有一个很有意思的地方。

```
from __future__ import print_function
a = [i for i in range(10)]
print(i)

```

如果我们在 Python 2 解释器上运行这段代码，我们会得到结果 9，因为用于列表推导式的 i 变量留在了内存中。如果你在下部分代码中忘了这回事，再使用 i 变量的话，会导致不可见的错误。

在 Python 3 中一切更为简单，在这个例子中的变量只在列表的创建期间使用，之后不再保存。这样当我们运行代码时，就会看到如下结果：

```
NameError: name 'i' is not defined

```

**结语**

本文我们谈论了 Python 2 和 Python 3 的一些区别，也看到虽然 Python 2 代码和 Python 3 代码不兼容，但我们可以通过一些方法抚平二者的差异，更简单地将 Python 2 编写的项目转为 Python 3 兼容项目。

* * *

> **_参考资料：_**  
> **_[https://py.checkio.org/blog/how-to-translate-code-from-python-2-to-python-3/](https://link.zhihu.com/?target=https%3A//py.checkio.org/blog/how-to-translate-code-from-python-2-to-python-3/)_**

[http://weixin.qq.com/r/80QiOi3EDnBxrWlZ9xHh](https://link.zhihu.com/?target=http%3A//weixin.qq.com/r/80QiOi3EDnBxrWlZ9xHh) (二维码自动识别)

 ![](https://pic3.zhimg.com/258463ec3_xs.jpg?source=1940ef5c) 小明​​按照当前时间点（Python 2.7 和 Python3.6），从宏观上介绍下 Python 3 和 Python 2 的区别，并举一些对应常见的例子：

1. **统一了字符编码支持。**我特意把它拿出来放在第一条...

2. **增加了新的语法**。print/exec 等成为了函数，[格式化字符串变量](https://link.zhihu.com/?target=https%3A//www.python.org/dev/peps/pep-0498/)，[类型标注](https://link.zhihu.com/?target=https%3A//www.python.org/dev/peps/pep-0484/)，添加了 nonlocal、yield from、async/await、yield for 关键词和__annotations__、__context__、__traceback__、__qualname__等 dunder 方法。

3. **修改了一些语法**。metaclass，raise、[map](https://link.zhihu.com/?target=https%3A//docs.python.org/3/library/functions.html%23map)、[filter](https://link.zhihu.com/?target=https%3A//docs.python.org/3/library/functions.html%23filter) 以及 dict 的 items/keys/values 方法返回迭代对象而不是列表，描述符协议，[保存类属性定义顺序](https://link.zhihu.com/?target=https%3A//www.python.org/dev/peps/pep-0520)，[保存关键字参数顺序](https://link.zhihu.com/?target=https%3A//www.python.org/dev/peps/pep-0468)

4. **去掉了一些语法**。cmp、<>(也就是!=)、xrange（其实就是 range）、不再有经典类

5. **增加一些新的模块**。concurrent.futures、venv、[unittest.mock](https://link.zhihu.com/?target=https%3A//docs.python.org/3/library/unittest.mock.html%23module-unittest.mock)、[asyncio](https://link.zhihu.com/?target=https%3A//docs.python.org/3/library/asyncio.html%23module-asyncio)、[selectors](https://link.zhihu.com/?target=https%3A//docs.python.org/3/library/selectors.html%23module-selectors)、[typing](https://link.zhihu.com/?target=https%3A//docs.python.org/3/library/typing.html%23module-typing) 等

6. **修改了一些模块**。主要是对模块添加函数 / 类 / 方法（如 functools.lru_cache、threading.Barrier）或者参数。

7. **模块改名**。把一些相关的模块放进同一个包里面（如 httplib, BaseHTTPServer, CGIHTTPServer, SimpleHTTPServer, Cookie, cookielib 放进了 http 里面，[urllib](https://link.zhihu.com/?target=https%3A//docs.python.org/3/library/urllib.html%23module-urllib), urllib2, urlparse, robotparse 放进了 [urllib](https://link.zhihu.com/?target=https%3A//docs.python.org/3/library/urllib.html%23module-urllib) 里面），个例如 SocketServer 改成了 socketserver，Queue 改成 queue 等

8. **去掉了一些模块或者函数**。gopherlib、md5、contextlib.nested、inspect.getmoduleinfo 等。去掉的内容的原因主要是 2 点：1. 过时的技术产物，已经没什么人在用了；2. 出现了新的替代产物后者被证明存在意义不大。理论上对于开发者影响很小。

9. **优化**。重新实现了 dict 可以减少 20%-25% 的内存使用；提升 [pickle](https://link.zhihu.com/?target=https%3A//docs.python.org/3/library/pickle.html%23module-pickle) 序列化和反序列化的效率；collections.OrderedDict 改用 C 实现；通过 os.scandir 对 glob 模块中的 glob() 及 iglob() 进行优化，使得它们现在大概快了 3-6 倍等.. 这些都是喜大普奔的好消息，同样开发者不需要感知，默默的就会让结果变得更好。

10. **其他**。构建过程、C 的 API、安全性等方面的修改，通常对于开发者不需要关心。

最后，重提一下我经常说的那句话：

```
Python 2/3的思想基本是共通的，只有少量的语法有差别甚至不兼容。当对Python熟悉到
一定程度时， 即使只会Python 2也可以在很短的时间就能写Python 3的代码。

```
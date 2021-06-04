> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NTY1MjY0MQ==&mid=2650800750&idx=5&sn=509997b6b83bf08418049cfd23b26a44&chksm=bd0197208a761e36513f0fdef2ca64decf58b13438bd43187a40e76ff4c21358c487099e5e4e&mpshare=1&scene=1&srcid=0511U42pWQ0QUBvFCvZLo8HF&sharer_sharetime=1620738323561&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

文 | 巩庆奎

出品 | 简说 Python（ID：xksnh888）

要说 Python 里使用频率最高的符号，我想下划线应该排第一吧？

在不同场合下，下划线有不同含义：比如`_var`表示内部变量；`__var`表示私有属性；`__var__`表示魔术方法；这些含义有的是程序员群体的约定，如`_var`；有的是 Python 解释器规定的形式，如`__var`。

*   `_`用于临时变量
    
*   `var_`用于解决命名冲突问题
    
*   `_var`用于保护变量
    
*   `__var`用于私有变量
    
*   `__var__`用于魔术方法
    

一、`_`用于临时变量
===========

单下划线一般用于表示临时变量，在 REPL、for 循环和元组拆包等场景中比较常见。

1.1 REPL
--------

```
>>> 1+1
2
>>> _
2
>>> a=2+2
>>> _
2


```

`1+1`，结果为 2，赋值给`_`；而赋值表达式`a=2+2`a 为 4，但整个表达式结果为`None`，故不会关联到`_`。这有点类似日常大家使用的计算器中的`ANS`按键，直接保存了上次的计算结果。

1.2 for 循环中的_
-------------

for 循环中`_`作为临时变量用。下划线来指代没什么意义的变量。例如在如下函数中，当我们只关心函数执行次数，而不关心具体次序的情况下，可以使用`_`作为参数。

```
nums = 13
for _ in range(nums):
    fun_oper()


```

1.3 元组拆包中的_
-----------

第三个用法是元组拆包，赋值的时候可以用`_`来表示略过的内容。如下代码忽略北京市人口数，只取得名字和区号。

```
>>> city,_,code = ('Beijing',21536000,'010')
>>> print(city,code)
Beijing 010


```

如果需要略过的内容多于一个的话，可以使用`*`开头的参数，表示忽略多个内容。如下代码忽略面积和人口数，只取得名字和区号

```
city,*_,code = ('Beijing',21536000,16410.54,'010')


```

1.4 国际化函数
---------

在一些国际化编程中，`_`常用来表示翻译函数名。例如 gettext 包使用时：

```
import gettext
zh = gettext.tranlation('dict','locale',languages=['zh_CN'])
zh.install()
_('hello world')


```

依据设定的字典文件，其返回相应的汉字 “你好世界”。

1.5 大数字表示形式
-----------

`_`也可用于数字的分割，这在数字比较长的时候常用。

```
>>> a = 9_999_999_999
>>> a
9999999999


```

a 的值自动忽略了下划线。这样用_分割数字，有利于便捷读取比较大的数。

二、`var_`用于解决命名冲突问题
==================

变量后面加一个下划线。主要用于解决命名冲突问题，元编程中遇时 Python 保留的关键字时，需要临时创建一个变量的副本时，都可以使用这种机制。

```
def type_obj_class(name,class_):
    pass

def tag(name,*content,class_):
    pass


```

以上代码中出现的`class`是 Python 的保留关键字，直接使用会报错，使用下划线后缀的方式解决了这个问题。

三、`_var`用于保护变量
==============

前面一个下划线，后面加上变量，这是仅供内部使用的 “保护变量”。比如函数、方法或者属性。

这种保护不是强制规定，而是一种程序员的约定，解释器不做访问控制。一般来讲这些属性都作为实现细节而不需要调用者关心，随时都可能改变，我们编程时虽然能访问，但是不建议访问。

这种属性，只有在导入时，才能发挥保护作用。而且必须是`from XXX import *`这种导入形式才能发挥保护作用。

> 使用`from XXX import *`是一种通配导入（wildcard import），这是 Python 社区不推荐的方式，因为你根本搞不清你到底导入了什么属性、方法，很可能搞乱你自己的命名空间。PEP8 推荐的导入方式是`from XXX import aVar , b_func , c_func`这种形式。

比如在下例汽车库函数 tools.py 里定义的 “保护属性”：发动机型号和轮胎型号，这属于实现细节，没必要暴露给用户。当我们使用 from tools import * 语句调用时，其实际并没有导入所有`_`开头的属性，只导入了普通 drive 方法。

```
_moto_type = 'L15b2'
_wheel_type = 'michelin'

def drive():
    _start_engine()
    _drive_wheel()

def _start_engine():
    print('start engine %s'%_moto_type)
    
def _drive_wheel():
    print('drive wheel %s'%_wheel_type)


```

查看命令空间`print(vars())`可见，只有 drive 函数被导入进来，其他下划线开头的 “私有属性” 都没有导入进来。

```
{'__name__': '__main__', '__doc__': None, '__package__': None, '__loader__': <_frozen_importlib_external.SourceFileLoader object at 0x005CF868>, '__spec__': None, '__annotations__':{}, '__builtins__': <module 'builtins' (built-in)>, '__file__': '.\\xiahuaxian.py', '__cached__': None, 'walk': <function walk at 0x01DA8C40>, 'root': '.\\__pycache__', '_': [21536000, 16410.54], 'dirs': ['tools.cpython-38.pyc'], 'city': 'Beijing', 'code': '010', 'drive': <function drive at 0x01DBC4A8>}


```

3.1 突破保护属性
----------

之所以说是 “保护” 并不是“私有”，是因为 Python 没有提供解释器机制来控制访问权限。我们依然可以访问这些属性：

```
import tools
tools._moto_type = 'EA211'
tools.drive()


```

以上代码，以越过 “保护属性”。此外，还有两种方法能突破这个限制，一种是将“私有属性” 添加到 tool.py 文件的__all__列表里，使`from tools import *`也导入这些本该隐藏的属性。

```
__all__ = ['drive','_moto_type','_wheel_type']


```

另一种是导入时指定 “受保护属性” 名。

```
from tools import drive,_start_engine
_start_engine()


```

甚至是，使用`import tools`也可以轻易突破保护限制。所以可见，“保护属性” 是一种简单的隐藏机制，只有在`from tools import *`时，由解释器提供简单的保护，但是可以轻易突破。这种保护更多地依赖程序员的共识：不访问、修改 “保护属性”。除此之外，有没有更安全的保护机制呢？有，就是下一部分讨论的私有变量。

四、`__var`用于私有变量
===============

私有属性解决的之前的保护属性保护力度不够的问题。变量前面加上两个下划线，类里面作为属性名和方法都可以。两个下划线属性由 Python 的改写机制来实现对这个属性的保护。

看下面汽车例子中，品牌为普通属性，发动机为 “保护属性”，车轮品牌为 “私有属性”。

```
class Car:
    def __init__(self):
        self.brand = 'Honda'
        self._moto_type = 'L15B2'
        self.__wheel_type = 'michelin'

    def drive(self):
        print('Start the engine %s,drive the wheel %s,I get a running %s car'%
        (self._moto_type,
        self.__wheel_type,
        self.brand))


```

我们用`var(car1)`查看下具体属性值，

```
['_Car__wheel_type', '__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', '_moto_type', 'brand', 'drive']


```

可见，实例化 car1 中，普通属性 self.brand 和保护属性 self._moto_type 都得以保存，两个下划线的私有属性__wheel_type 没有了。取而代之的是`_Car_wheel_type`这个属性。这就是改写机制（Name mangling）。两个下划线的属性，被改写成带有类名前缀的变量，这样子类很难明明一个和如此复杂名字重名的属性。保证了属性不被重载，保证了其的私有性。

4.1 突破私有属性
----------

这里 “私有变量” 的实现，是从解释器层面给与的改写，保护了私有变量。但是这个机制并非绝对安全，因为我们依然可以通过 obj._ClasssName__private 来访问__private 私有属性。

```
car1.brand = 'Toyota'
car1._moto_type = '6AR-FSE'
car1._Car__wheel_type = 'BRIDGESTONE'
car1.drive()


```

结果

```
Start the engine 6AR-FSE,\
drive the wheel BRIDGESTONE,\
I get a running Toyota car


```

可见，对改写机制改写的私有变量，虽然保护性加强了，但依然可以访问并修改。只是这种修改，只是一种杂耍般的操作，并不可取。

五、`__var__`用于魔术方法
=================

变量前面两个下划线，后面两个下划线。这是 Python 当中的魔术方法，一般是给系统程序调用的。例如上例中的__init__就是类的初始化魔术方法，还有支持 len 函数的__len__方法，支持上下文管理器协议的__enter__和__exit__方法，支持迭代器协议的__iter__方法，支持格式化显示的__repr__和__str__方法等等。这里我们为上例的 Car 类添加魔术方法__repr__来支持格式化显示。

```
    def __repr__(self):
        return '***Car %s:with %s Engine,%sWheel***'%
        (self.brand,self._moto_type,self.__wheel_type)


```

5.1 Python 魔术方法分类
-----------------

> 以下所有魔术方法均需要在前后加上__，这里省略了这些双下划线。

*   一元运算符 neg pos abs invert
    
*   转换 complex int float round inex
    
*   算术运算 add sub mul truediv floordiv mod divmod pow lshift rshift and xor or
    

> 算术运算除 and 之外，前面再加上 r，表示反运算。除 dimod 外，前面加上 i，表示就地运算。

*   比较 lt le eq ne gt ge
    
*   类属性 getattr getattribute setattr delattr dir get set delete
    
*   格式化 bytes hash bool format
    
*   类相关 init del new
    
*   列表 getitem
    
*   迭代器 iter next
    
*   上下文管理器 enter exit
    

六、总结
====

总之，下划线在 Python 当中应用还是很广泛的，甚至可以说 Python 对下划线有所偏爱。可以看到 `_`常用于临时变量，在 REPL，for 循环，元组拆包和国际化中得到了广泛应用。`var_`用于解决命名冲突问题，使用时比较简单易懂的。`_var`对变量的保护，只是一种脆弱的保护，更多依靠程序员的约定。`__var`用于私有变量，借助改写机制支持，已经支持了私有变量，但是仍然存在漏洞。对`__var__`用于魔术方法，进行了一个简单的介绍，魔术方法较多，但是理解并不复杂。希望以后可以进一步介绍这些魔术方法。
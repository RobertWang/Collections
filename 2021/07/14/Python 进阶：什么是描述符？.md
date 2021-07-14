> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5OTA1MDUyMA==&mid=2655464567&idx=4&sn=e3dfd0063c8ca2093f47aedd2c4829ac&chksm=bd72e0008a056916ae6db9c9ccd68b22c9ac8b60ebc1dadf36fcd2a25f22ff1e8bed24d73bb4&mpshare=1&scene=1&srcid=0713vgIQINb5WsYT6GiNTegS&sharer_sharetime=1626143703824&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

*   `function`、`bound method`、`unbound method`
    
*   装饰器`property`、`staticmethod`、`classmethod`
    

什么是描述符？
=======

```
class Ten:
    def __get__(self, obj, objtype=None):
        return 10

class A:
    x = Ten()   # 属性换成了一个类
    
print(A.x) # 10


```

```
class Age:
    def __get__(self, obj, objtype=None):
        if obj.name == 'zhangsan':
            return 20
        elif obj.name == 'lisi':
            return 25
        else:
            return ValueError("unknow")

class Person:

    age = Age()

    def __init__(self, name):
        self.name = name

p1 = Person('zhangsan')
print(p1.age)   # 20

p2 = Person('lisi')
print(p2.age)   # 25

p3 = Person('wangwu')
print(p3.age)   # unknow


```

这只是一个非常简单的例子，我们可以看到，通过描述符的使用，我们可以轻易地改变一个类属性的定义方式。

描述符协议
=====

其实，一个类属性想要托管给一个类，这个类内部实现的方法不能是随便定义的，它必须遵守「描述符协议」，也就是要实现以下几个方法：

*   `__get__(self, obj, type=None)`
    
*   `__set__(self, obj, value)`
    
*   `__delete__(self, obj)`
    

另外，描述符又可以分为「数据描述符」和「非数据描述符」：

*   只定义了 `__get___`，叫做非数据描述符
    
*   除了定义 `__get__` 之外，还定义了 `__set__` 或 `__delete__`，叫做数据描述符
    

现在我们来看一个包含 `__get__` 和 `__set__` 方法的描述符例子：

```
# coding: utf8

class Age:

    def __init__(self, value=20):
        self.value = value

    def __get__(self, obj, type=None):
        print('call __get__: obj: %s type: %s' % (obj, type))
        return self.value

    def __set__(self, obj, value):
        if value <= 0:
            raise ValueError("age must be greater than 0")
        print('call __set__: obj: %s value: %s' % (obj, value))
        self.value = value

class Person:

    age = Age()

    def __init__(self, name):
        self.name = name

p1 = Person('zhangsan')
print(p1.age)
# call __get__: obj: <__main__.Person object at 0x1055509e8> type: <class '__main__.Person'>
# 20

print(Person.age)
# call __get__: obj: None type: <class '__main__.Person'>
# 20

p1.age = 25
# call __set__: obj: <__main__.Person object at 0x1055509e8> value: 25

print(p1.age)
# call __get__: obj: <__main__.Person object at 0x1055509e8> type: <class '__main__.Person'>
# 25

p1.age = -1
# ValueError: age must be greater than 0


```

从输出结果来看，当我们获取或修改 `age` 属性时，调用了 `Age` 的 `__get__` 和 `__set__` 方法：

*   当调用 `p1.age` 时，`__get__` 被调用，参数 `obj` 是 `Person` 实例，`type` 是 `type(Person)`
    
*   当调用 `Person.age` 时，`__get__` 被调用，参数 `obj` 是 `None`，`type` 是 `type(Person)`
    
*   当调用 `p1.age = 25`时，`__set__` 被调用，参数 `obj` 是 `Person` 实例，`value` 是 25
    
*   当调用 `p1.age = -1`时，`__set__` 没有通过校验，抛出 `ValueError`
    

这就需要我们了解一下描述符的工作原理。

描述符的工作原理
========

在开发时，不知道你有没有想过这样一个问题：通常我们写这样的代码 `a.b`，其背后到底发生了什么？

1.  `a` 可能是一个类，也可能是一个实例，我们这里统称为对象
    
2.  `b` 可能是一个属性，也可能是一个方法，方法其实也可以看做是类的属性
    

其实，无论是以上哪种情况，在 Python 中，都有一个统一的调用逻辑：

1.  先调用 `__getattribute__` 尝试获得结果
    
2.  如果没有结果，调用 `__getattr__`
    

```
def getattr_hook(obj, name):
    try:
        return obj.__getattribute__(name)
    except AttributeError:
        if not hasattr(type(obj), '__getattr__'):
            raise
    return type(obj).__getattr__(obj, name) 


```

我们这里需要重点关注一下 `__getattribute__`，因为它是所有属性查找的入口，它内部实现的属性查找顺序是这样的：

1.  要查找的属性，在类中是否是一个描述符
    
2.  如果是描述符，再检查它是否是一个数据描述符
    
3.  如果是数据描述符，则调用数据描述符的 `__get__`
    
4.  如果不是数据描述符，则从 `__dict__` 中查找
    
5.  如果 `__dict__` 中查找不到，再看它是否是一个非数据描述符
    
6.  如果是非数据描述符，则调用非数据描述符的 `__get__`
    
7.  如果也不是一个非数据描述符，则从类属性中查找
    
8.  如果类中也没有这个属性，抛出 `AttributeError` 异常
    

```
# 获取一个对象的属性
def __getattribute__(obj, name):
    null = object()
    # 对象的类型 也就是实例的类
    objtype = type(obj)
    # 从这个类中获取指定属性
    cls_var = getattr(objtype, name, null)
    # 如果这个类实现了描述符协议
    descr_get = getattr(type(cls_var), '__get__', null)
    if descr_get is not null:
        if (hasattr(type(cls_var), '__set__')
            or hasattr(type(cls_var), '__delete__')):
            # 优先从数据描述符中获取属性
            return descr_get(cls_var, obj, objtype)
    # 从实例中获取属性
    if hasattr(obj, '__dict__') and name in vars(obj):
        return vars(obj)[name]
    # 从非数据描述符获取属性
    if descr_get is not null:
        return descr_get(cls_var, obj, objtype)
    # 从类中获取属性
    if cls_var is not null:
        return cls_var
    # 抛出 AttributeError 会触发调用 __getattr__
    raise AttributeError(name)


```

在 `__getattribute__` 中，它会检查这个类属性是否是一个描述符，如果是一个描述符，那么就会调用它的 `__get__` 方法。但具体的调用细节和传入的参数是下面这样的：

*   如果 `a` 是一个**实例**，调用细节为：
    

```
type(a).__dict__['b'].__get__(a, type(a))


```

*   如果 `a` 是一个**类**，调用细节为：
    

```
a.__dict__['b'].__get__(None, a)


```

数据描述符和非数据描述符
============

了解了描述符的工作原理，我们继续来看数据描述符和非数据描述符的区别。

*   只定义了 `__get___`，叫做非数据描述符
    
*   除了定义 `__get__` 之外，还定义了 `__set__` 或 `__delete__`，叫做数据描述符
    

此外，我们从上面描述符调用的顺序可以看到，在对象中查找属性时，数据描述符要优先于非数据描述符调用。

我们再来看一个**非数据描述符**的例子：

```
class A:

    def __init__(self):
        self.foo = 'abc'

    def foo(self):
        return 'xyz'

print(A().foo)  # 输出什么？


```

答案是 `abc`。

这就和非数据描述符有关系了。

```
print(dir(A.foo))
# [... '__get__', '__getattribute__', ...]


```

看到了吗？`A` 的 `foo` 方法其实实现了 `__get__`，我们在上面的分析已经得知：只定义 `__get__` 方法的对象，它其实是一个非数据描述符，也就是说，**我们在类中定义的方法，其实本身就是一个非数据描述符。**

到这里我们可以总结一下关于描述符的相关知识点：

*   描述符必须是一个类属性
    
*   `__getattribute__` 是查找一个属性（方法）的入口
    
*   `__getattribute__` 定义了一个属性（方法）的查找顺序：数据描述符、实例属性、非数据描述符、类属性
    
*   如果我们重写了 `__getattribute__` 方法，会阻止描述符的调用
    
*   所有方法其实都是一个非数据描述符，因为它定义了 `__get__`
    

描述符的使用场景
========

在这里我用描述符实现了一个属性校验器，你可以参考这个例子，在类似的场景中去使用它。

```
class Validator:

    def __init__(self):
        self.data = {}

    def __get__(self, obj, objtype=None):
        return self.data[obj]

    def __set__(self, obj, value):
        # 校验通过后再赋值
        self.validate(value)
        self.data[obj] = value

    def validate(self, value):
        pass    


```

接下来，我们定义两个校验类，继承 `Validator`，然后实现自己的校验逻辑。

```
class Number(Validator):

    def __init__(self, minvalue=None, maxvalue=None):
        super(Number, self).__init__()
        self.minvalue = minvalue
        self.maxvalue = maxvalue

    def validate(self, value):
        if not isinstance(value, (int, float)):
            raise TypeError(f'Expected {value!r} to be an int or float')
        if self.minvalue is not None and value < self.minvalue:
            raise ValueError(
                f'Expected {value!r} to be at least {self.minvalue!r}'
            )
        if self.maxvalue is not None and value > self.maxvalue:
            raise ValueError(
                f'Expected {value!r} to be no more than {self.maxvalue!r}'
            )

class String(Validator):

    def __init__(self, minsize=None, maxsize=None):
        super(String, self).__init__()
        self.minsize = minsize
        self.maxsize = maxsize

    def validate(self, value):
        if not isinstance(value, str):
            raise TypeError(f'Expected {value!r} to be an str')
        if self.minsize is not None and len(value) < self.minsize:
            raise ValueError(
                f'Expected {value!r} to be no smaller than {self.minsize!r}'
            )
        if self.maxsize is not None and len(value) > self.maxsize:
            raise ValueError(
                f'Expected {value!r} to be no bigger than {self.maxsize!r}'
            )


```

```
class Person:

    # 定义属性的校验规则 内部用描述符实现
    name = String(minsize=3, maxsize=10)
    age = Number(minvalue=1, maxvalue=120)

    def __init__(self, name, age):
        self.name = name
        self.age = age

# 属性符合规则
p1 = Person('zhangsan', 20)
print(p1.name, p1.age)

# 属性不符合规则
p2 = person('a', 20)
# ValueError: Expected 'a' to be no smaller than 3
p3 = Person('zhangsan', -1)
# ValueError: Expected -1 to be at least 1


```

现在，当我们对 `Person` 实例进行初始化时，就可以校验这些属性是否符合预定义的规则了。

function 与 method
=================

来看下面这段代码：

```
class A:

    def foo(self):
        return 'xyz'

print(A.__dict__['foo']) # <function foo at 0x10a790d70>
print(A.foo)     # <unbound method A.foo>
print(A().foo)   # <bound method A.foo of <__main__.A object at 0x10a793050>>


```

*   `function` 准确来说就是一个函数，并且它实现了 `__get__` 方法，因此每一个 `function` 都是一个非数据描述符，而在类中会把 `function` 放到 `__dict__` 中存储
    
*   当 `function` 被实例调用时，它是一个 `bound method`
    
*   当 `function` 被类调用时， 它是一个 `unbound method`
    

`function` 是一个非数据描述符，我们之前已经讲到了。

property/staticmethod/classmethod
=================================

我们再来看 `property`、`staticmethod`、`classmethod`。

其实，我们也可以直接利用 Python 描述符的特性来实现这些装饰器，

```
class property:

    def __init__(self, fget=None, fset=None, fdel=None, doc=None):
        self.fget = fget
        self.fset = fset
        self.fdel = fdel
        self.__doc__ = doc

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self.fget
        if self.fget is None:
            raise AttributeError(), "unreadable attribute"
        return self.fget(obj)

    def __set__(self, obj, value):
        if self.fset is None:
            raise AttributeError, "can't set attribute"
        return self.fset(obj, value)

    def __delete__(self, obj):
        if self.fdel is None:
            raise AttributeError, "can't delete attribute"
        return self.fdel(obj)

    def getter(self, fget):
        return type(self)(fget, self.fset, self.fdel, self.__doc__)

    def setter(self, fset):
        return type(self)(self.fget, fset, self.fdel, self.__doc__)

    def deleter(self, fdel):
        return type(self)(self.fget, self.fset, fdel, self.__doc__)


```

`staticmethod` 的 Python 版实现：

```
class staticmethod:

    def __init__(self, func):
        self.func = func

    def __get__(self, obj, objtype=None):
        return self.func


```

```
class classmethod:

    def __init__(self, func):
        self.func = func

    def __get__(self, obj, klass=None):
        if klass is None:
            klass = type(obj)
        def newfunc(*args):
            return self.func(klass, *args)
        return newfunc


```

除此之外，你还可以实现其他功能强大的装饰器。

总结
==

这篇文章我们主要讲了 Python 描述符的工作原理。

之后我们又分析了获取一个属性的过程，一切的入口都在 `__getattribute__` 中，这个方法定义了寻找属性的顺序，其中实例属性优先于数据描述符调用，数据描述符要优先于非数据描述符调用。

最后我们分析了 `function` 和 `method` 的区别，以及使用 Python 描述符也可以实现 `property`、`staticmethod`、`classmethod` 装饰器。

> Tips：关注 “Python 开发者” 公众号，并在后台发送「**进阶**」可获取该系列文章。
> 
>   

点赞和在看就是最大的支持❤️
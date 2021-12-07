> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI2NjY5NzI0NA==&mid=2247505099&idx=2&sn=aea585ea0f2ee7482f7ca965a04b7eea&chksm=ea88bbb8ddff32ae540cb7415fb7237d4d6c3120addc843b2077de06308d30d204ae7a46a092&mpshare=1&scene=1&srcid=1206UZqvkAG3pF2kvjTrpHwZ&sharer_sharetime=1638764192281&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

写出能完成功能的程序每个程序员都可以搞定，但能写出优雅的程序的程序员却寥寥无几，因此程序写的优雅与否则是区分顶级程序员与一般程序员的终极指标所在。  

那身为一名 Pythoner，有哪些技巧能让我们写出优雅的 Python 代码呢，今天派森酱就给大家介绍七个能快速提升代码逼格的重要技巧。

0x00 规范命名
---------

没有哪个程序员会抗拒一段命名规范的代码！

命名作为编程界的一大难题，实属难倒了很多人。不知道你是否还记得自己那些曾经很沙雕的命名呢。

```
a,b,c  x,y,z a1,a2 4_s,4s... 
def do_something():
def fun():
...
```

相信你看到上面的命名也是一头雾水，好的命名不一定要写的多优雅，最起码要做到见名识意。统一的命名风格可以让代码看起来更简洁，风格更统一，这样阅读者一看就知道这个变量或者函数是用来干嘛的，不至于猜半天浪费过多的精力在不必要的事情上。

0x01 面向对象
---------

Python 是一门面向对象语言，因此我们有必要熟悉面向对象的一些设计原则。

单一职责原则是指一个函数只做一件事，不要将多个功能集中在同一个函数中，不要大而全，要小而精。这样，当有需求变化时，我们只需要修改对应的部分即可，程序应对变化的能力明显提升。

开放封闭原则是指对扩展开放，对修改关闭。

写程序的都知道，甲方是善变的，今天说用这种方式实现，明天可能就变卦了，这太正常了。所以我们写程序时一定要注意程序的可扩展性，当甲方改动需求时，我们尽可能的少改动或者不改动原有代码，而是通过添加新的实现类来扩展功能，这意味着你系统的原有功能是不会遭到破坏的，则稳定性有极大提升。

接口隔离原则是指调用方不应该依赖其不需要的接口，接口间的依赖关系应当建立在最小功能接口原则之上。

单一职责和接口隔离都是为了提高类的内聚性，降低他们之间的耦合性。这是面向对象封装思想的完美体现。

0x02 使用 with
------------

平时写代码难免会遇到操作文件的需求，一般都是用 `open()` 函数来打开一个文件，最后等操作完成之后通过 `close()` 函数来关闭文件，但有时候写多了难免会觉得很麻烦，难道不可以在我操作完自动关闭文件么，可以的。使用 with 来操作文件无需考虑关闭问题，我们只需要关心核心的业务逻辑即可。

```
with open('tmp.txt', 'w') as f:
    f.write('xxx')
    ...
```

0x03 使用 get
-----------

![](https://mmbiz.qpic.cn/mmbiz_png/pbRNVEA1d2xkPZnNp5FtRu4oAhTWQoBB4129zyBDuRn3Gic8CXJicjGBVJoYJ7Ydubg9vBuia9alM7fXUEuFV4u4A/640?wx_fmt=png)

当我们从字典中获取一个不存在的 key 时，如果是用中括号的方式来获取的话程序会返回 `KeyError`。这时候建议通过 `get()` 函数来获取。  

同时通过 `get()` 函数来获取 value 时还可以设置默认值 default_value，当 key 不存在时则会返回 default_value。

0x04 提前返回
---------

平时写的代码中少不了 if else 等控制语句，但有时候有的小伙伴喜欢将 if else 嵌套好多层，过几个月之后自己都看不明白当时写的啥。

比如下面这个程序，根据考试成绩来做评级。

```
score = 100
if score >= 60: # 及格
    if score >= 70: # 中等
        if score >= 80: # 良好 
            if score >= 90: # 优秀
                if score >= 100: # 满分
                    print("满分")
                else:
                    print("优秀")
            else:
                print("良好")
        else:
            print("中等")
    else:
        print("及格")
else:
    print("不及格")
print("程序结束")
```

这种代码一看就想打人有木有，可读性极差。

代码的逻辑就是判断分数是否在一个区间，然后给出与之相匹配的评级，既然如此，则可以改写如下：

```
def get_score_level(score):
    if score >= 100: # 满分
        print("满分")
        return

    if score >= 90: # 优秀
        print("优秀")
        return

    if score >= 80: # 良好
        print("良好")
        return    

    if score >= 70: # 中等
        print("中等")
        return

    if score >= 60: # 及格
        print("及格")
        return

    print("不及格")
    print("程序结束")
```

这种处理方式是极其优雅的，从上往下清晰明了，大大增加了代码的可读性和可维护性。

0x05 生成器
--------

我们都知道通过列表生成式可以直接创建一个新的列表，但受机器内存限制，列表的容量肯定是有限的。如果列表里面的数据是通过某种规律推导计算出来的，那是否可以在迭代过程中不断的推算出后面的元素呢，这样就不必一次性创建完整个列表，按需使用即可，这时候生成器就派上用场了。

![](https://mmbiz.qpic.cn/mmbiz_png/pbRNVEA1d2xkPZnNp5FtRu4oAhTWQoBBQQzEwdUCbPfWQWMeTv9hdF56pDiaIEb67EFbkas4tfkLRrCQoNeNK9A/640?wx_fmt=png)

0x06 装饰器
--------

试想一下如下的场景，当后端接收到用户请求后，需要对用户进行鉴权，总不能将鉴权的代码复制来复制去吧；还有我们的项目都是需要记录日志的，这两种情况最适合使用装饰器。事实上 Flask 框架中就大量使用装饰器来进行鉴权操作。

一切皆对象！

在 Python 中我们可以在函数中定义函数，也可以从函数中返回函数，还可以将函数作为参数传给另一个函数。

```
def hi():
    print("now you are inside the hi() function")
 
    def greet():
        return "now you are in the greet() function"
 
    def welcome():
        return "now you are in the welcome() function"
 
    print(greet())
    print(welcome())
    print("now you are back in the hi() function")

hi()
# output
# now you are inside the hi() function
# now you are in the greet() function
# now you are in the welcome() function
# now you are back in the hi() function
```

在上面的代码中，我们在 `hi()` 函数内部定义了两个新的函数，无论何时调用 `hi()` 其内部的函数都将会被调用。

```
def hi():
    def greet():
        return "now you are in the greet() function"
 
    def welcome():
        return "now you are in the welcome() function"
 
    if name == "yasoob":
        return greet
    else:
        return welcome
 
a = hi()
print(a)
print(a())

# output
# <function hi.<locals>.greet at 0x7fe3e547a0e0>
# now you are in the greet() function
```

在这个例子中，由于默认参数 `name = yasoob` 因此 `a = hi()` 返回的是 `greet` 函数。a 也就指向了 `hi()` 函数内部的 `greet()` 函数。

```
def hi():
    return "hi yasoob!"
 
def doSomethingBeforeHi(func):
    print("I am doing some boring work before executing hi()")
    print(func())
 
doSomethingBeforeHi(hi)

# output
# I am doing some boring work before executing hi()
# hi yasoob!
```

在最后这个例子中，我们将 `hi()` 函数传递给了另外一个函数，并且他们还很愉快的执行了。

现在，让我们来看看 Python 中的装饰器吧。

```
def a_new_decorator(a_func):
 
    def wrapTheFunction():
        print("I am doing some boring work before executing a_func()")
 
        a_func()
 
        print("I am doing some boring work after executing a_func()")
 
    return wrapTheFunction

def a_function_requiring_decoration():
    print("I am the function which needs some decoration to remove my foul smell")

 
a_new_function_requiring_decoration = a_new_decorator(a_function_requiring_decoration)

a_new_function_requiring_decoration()

# output
# I am doing some boring work before executing a_func()
# I am the function which needs some decoration to remove my foul smell
# I am doing some boring work after executing a_func()
```

看懂了没，就是上面我们介绍的基础操作的组合。事实上这就是 python 中的装饰器所做的事，通过这种方式来修改一个函数的行为。

但如果每次都这么写的话未免也太麻烦了吧，因此 python 为我们提供了一个便捷操作 `@`。

```
def a_new_decorator(a_func):
  ...

@a_new_decorator
def a_function_requiring_decoration():
    print("I am the function which needs some decoration to remove my foul smell")

a_function_requiring_decoration()

# output
# I am doing some boring work before executing a_func()
# I am the function which needs some decoration to remove my foul smell
# I am doing some boring work after executing a_func()
```

总结
--

今天派森酱给大家介绍了几个重要的提升代码逼格的技巧，小伙伴们还有什么独家技巧可以在评论区交流哦～
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIzMzMzOTI3Nw==&mid=2247502388&idx=1&sn=f7098d4f0108c651c9d0c6a0f7335d21&chksm=e885aad6dff223c0a9bacbed137f3f793e9dd244810c87f004a68dc52a946f5186df485e2dfc&mpshare=1&scene=1&srcid=0629pLbTKYklG2n3L2XmL9qi&sharer_sharetime=1624940218482&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

大家好，我是明哥。

1. 参数分类
-------

函数，在定义的时候，可以有参数的，也可以没有参数。

从函数定义的角度来看，参数可以分为两种：

1.  `必选参数`：调用函数时必须要指定的参数，在定义时没有等号
    
2.  `可选参数`：也叫`默认参数`，调用函数时可以指定也可以不指定，不指定就默认的参数值来。
    

例如下面的代码中，a 和 b 属于必选参数， c 和 d 属于可选参数

```
def func(a,b,c=0, d=1):
    pass


```

从函数调用的角度来看，参数可以分为两种：

1.  `关键字参数`：调用时，使用 key=value 形式传参的，这样传递参数就可以不按定义顺序来。
    
2.  `位置参数`：调用时，不使用关键字参数的 key-value 形式传参，这样传参要注意按照函数定义时参数的顺序来。
    

```
def func(a,b,c=0, d=1):
    pass

  # 关键字参数传参方法
func(a=10, c=30, b=20, d=40)

  # 位置参数传参方法
func(10, 20, 30, 40)


```

最后还有一种非常特殊的参数，叫做`可变参数`。

意思是参数个数可变，可以是 0 个或者任意个，但是传参时不能指定参数名，通常使用 `*args` 和 `**kw` 来表示：

*   `*args`：接收到的所有按照位置参数方式传递进来的参数，是一个元组类型
    
*   `**kw` ：接收到的所有按照关键字参数方式传递进来的参数，是一个字典类型
    

```
def func(*args, **kw):
    print(args)
    print(kw)

func(10, 20, c=20, d=40)


```

输出如下

```
(10, 20)
{'c': 20, 'd': 40}


```

2. 十一个案例
--------

**案例一**：在下面这个函数中， a 是必选参数，是必须要指定的

```
>>> def demo_func(a):
...     print(a)
... 
>>> demo_func(10) 
10
>>> demo_func()  # 不指定会报错
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: demo_func() missing 1 required positional argument: 'a'


```

**案例二**：在下面这个函数中，b 是可选参数（默认参数），可以指定也可以不指定，不指定的话，默认为 10

```
>>> def demo_func(b=10):
...     print(b)
... 
>>> demo_func(20)
20
>>> demo_func()
10


```

**案例三**：在下面这个函数中， name 和 age 都是必选参数，在调用指定参数时，如果不使用关键字参数方式传参，需要注意顺序

```
>>> def print_profile(name, age):
...     return f"我的名字叫{name}，今年{age}岁了"
...
>>> print_profile("iswbm", 27)
'我的名字叫iswbm，今年27岁了'


```

如果参数太多，你不想太花精力去注意顺序，可以使用关键字参数方式传参，在指定参数时附上参数名，比如这样：

```
>>> print_profile(age=27, )
'我的名字叫iswbm，今年27岁了'


```

**案例四**：在下面这个函数中，`args` 参数和上面的参数名不太一样，在它前面有一个 `*`，这就表明了它是一个可变参数，可以接收任意个数的不指定参数名的参数。

```
>>> def demo_func(*args):
...     print(args)
... 
>>> 
>>> demo_func(10, 20, 30)
(10, 20, 30)


```

**案例五**：在下面这个函数中，`kw` 参数和上面的 `*args` 还多了一个 `*` ，总共两个 `**` ，这个意思是 `kw` 是一个可变关键字参数，可以接收任意个数的带参数名的参数。

```
>>> def demo_func(**kw):
...     print(kw)
... 
>>> demo_func(a=10, b=20, c=30)
{'a': 10, 'b': 20, 'c': 30}


```

**案例六**：在定义时，必选参数一定要在可选参数的前面，不然运行时会报错

```
>>> def demo_func(a=1, b):
...     print(a, b)
... 
  File "<stdin>", line 1
SyntaxError: non-default argument follows default argument
>>>
>>> def demo_func(a, b=1):
...     print(a, b)
... 
>>>


```

**案例七**：在定义时，可变位置参数一定要在可变关键字参数前面，不然运行时也会报错

```
>>> def demo_func(**kw, *args):
  File "<stdin>", line 1
    def demo_func(**kw, *args):
                        ^
SyntaxError: invalid syntax
>>> 
>>> def demo_func(*args, **kw):
...     print(args, kw)
... 
>>> 


```

**案例八**：可变位置参数可以放在必选参数前面，但是在调用时，必选参数必须要指定参数名来传入，否则会报错

```
>>> def demo_func(*args, b):
...     print(args)
...     print(b)
... 
>>> demo_func(1, 2, 100)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: demo_func() missing 1 required keyword-only argument: 'b'
>>> 
>>> demo_func(1, 2, b=100)
(1, 2)
100


```

**案例九**：可变关键字参数则不一样，可变关键字参数一定得放在最后，下面三个示例中，不管关键字参数后面接位置参数，还是默认参数，还是可变参数，都会报错。

```
>>> def demo_func(**kw, a):
  File "<stdin>", line 1
    def demo_func(**kw, a):
                        ^
SyntaxError: invalid syntax
>>> 
>>> def demo_func(**kw, a=1):
  File "<stdin>", line 1
    def demo_func(**kw, a=1):
                        ^
SyntaxError: invalid syntax
>>> 
>>> def demo_func(**kw, *args):
  File "<stdin>", line 1
    def demo_func(**kw, *args):
                        ^
SyntaxError: invalid syntax


```

**案例十**：将上面的知识点串起来，四种参数类型可以在一个函数中出现，但一定要注意顺序

```
def demo_func(arg1, arg2=10, *args, **kw):
    print("arg1: ", arg1)
    print("arg2: ", arg2)
    print("args: ", args)
    print("kw: ", kw)


```

试着调用这个函数，输出如下：

```
>>> demo_func(1,12, 100, 200, d=1000, e=2000)
arg1:  1
arg2:  12
args:  (100, 200)
kw:  {'d': 1000, 'e': 2000}


```

**案例十一**：使用单独的 `*`，当你在给后面的位置参数传递时，对你传参的方式有严格要求，你在传参时必须要以关键字参数的方式传参数，要写参数名，不然会报错。

```
>>> def demo_func(a, b, *, c):
...     print(a)
...     print(b)
...     print(c)
... 
>>> 
>>> demo_func(1, 2, 3)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: demo_func() takes 2 positional arguments but 3 were given
>>> 
>>> demo_func(1, 2, c=3)
1
2
3


```

3. 传参的坑
-------

函数参数传递的是实际对象的内存地址。如果参数是引用类型的数据类型（列表、字典等），在函数内部修改后，就算没有把修改后的值返回回去，外面的值其实也已经发生了变化。

```
>>> def add_item(item, source_list):
...     source_list.append(item)
...
>>> alist = [0,1]
>>> add_item(2, alist)
>>> alist
[0, 1, 2]


```
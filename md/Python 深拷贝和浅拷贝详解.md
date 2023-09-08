> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/7xH_DlQibjhcg_QY-YpSrg)

先说下结论
-----

> ❝
> 
> *   深拷贝 (deepcopy) 拷贝的多一些 (外层和内层都会拷贝)
>     
> *   浅拷贝 (copy) 拷贝的少一些 (只拷贝最外层)
>     
> *   最外层是可变类型:
>     
> 
> *   **「[不可变, 不可变], copy(浅拷贝), deepcopy(深拷贝) 都会生成新的对象, 没有任何区别」**
>     
> *   [[],[]] ,  copy(浅拷贝) 只会拷贝最外层, 生成新的对象, 内层的对象都是引用, deepcopy(深拷贝) 内层和外层都会拷贝, **「内层和外层都是全新的对象」**
>     
> 
> *   最外层是不可变类型
>     
> 
> *   ([],[]),  copy(浅拷贝) 仅仅只是引用, 浅拷贝只拷贝最外层, 内层就是引用关系, 最外层不可变的, 拷贝生成的新对象不能够做任何的修改, 所以就直接引用, deepcopy(深拷贝) 会拷贝得到**「全新的对象, 深拷贝会从内到外都拷贝,」**  ([],[]) 内层是可变的有可能需要对内层的对象做修改, 所以依然会得到新的对象
>     
> *   **「((),()) 完全的不可变类型, 深拷贝和浅拷贝都是指向, 不会生成新的对象」**
>     
> *   ((),[]) 浅拷贝是指向, 不会生成新的对象,  深拷贝: 如果内层是不可变就是引用, 内层是可变就会**「拷贝得到全新的对象」**
>     
> 
> ❞

下面看演示
-----

### 最外层是可变类型

> ❝
> 
> 最外层是可变：[不可变, 不可变] copy(浅拷贝), deepcopy(深拷贝) 都会生成新的对象, 没有任何区别
> 
> ❞

```
# 最外层是可变：[不可变, 不可变]
# copy(浅拷贝), deepcopy(深拷贝)都会生成新的对象,没有任何区别
import copy

a1 = [1, 2, 3, 4]
a2 = ["ab", "cd"]

b1 = copy.copy(a1)
b2 = copy.copy(a2)

c1 = copy.deepcopy(a1)
c2 = copy.deepcopy(a2)

print(id(a1), id(a2), id(c1), id(c2))

print(a1, a2, b1, b2, c1, c2)
print("********************************")
a1 = [5, 6]
a2 = ["56"]

b11 = copy.copy(a1)
b22 = copy.copy(a2)


a1.append("哈哈")
a2.append("哈哈")
print("********************************")

print(id(a1), id(a2))

print(a1, a2, b11, b22)
# 140429459489728 140429190309504 140429190303616 140429459542272
# [1, 2, 3, 4] ['ab', 'cd'] [1, 2, 3, 4] ['ab', 'cd'] [1, 2, 3, 4] ['ab', 'cd']
# ********************************
# ********************************
# 140429190308288 140429459489728
# [5, 6, '哈哈'] ['56', '哈哈'] [5, 6] ['56']




```

> ❝
> 
> 最外层是可变：[[],[]]
> 
> copy(浅拷贝) 只会拷贝最外层, 生成新的对象, 内层的对象都是引用, deepcopy(深拷贝) 内层和外层都会拷贝, **「内层和外层都是全新的对象」**
> 
> ❞

### 最外层是不可变类型

> ❝
> 
> 最外层是不可变
> 
> ([],[]),  copy(浅拷贝) 仅仅只是引用, 浅拷贝只拷贝最外层, 内层就是引用关系, 最外层不可变的, 拷贝生成的新对象不能够做任何的修改, 所以就直接引用, deepcopy(深拷贝) 会拷贝得到全新的对象, 深拷贝会从内到外都拷贝,  ([],[]) 内层是可变的有可能需要对内层的对象做修改, 所以依然会得到新的对象
> 
> ❞

```
# 2023/5/13 14:43 , @File: demo2.py, @Comment : python3.9.7
# 最外层是不可变类型
# - ([],[]),  copy(浅拷贝) 仅仅只是引用, 浅拷贝只拷贝最外层,内层就是引用关系, 最外层不可变的,拷贝生成的新对象不能够做任何的修改, 所以就直接引用, deepcopy(深拷贝) 会拷贝得到全新的对象,深拷贝会从内到外都拷贝,  ([],[])内层是可变的有可能需要对内层的对象做修改, 所以依然会得到新的对象
# - ((),()) 完全的不可变类型, 深拷贝和浅拷贝都是指向, 不会生成新的对象
# - ((),[]) 浅拷贝是指向, 不会生成新的对象,  深拷贝: 如果内层是不可变就是引用,内层是可变就会拷贝得到全新的对象

import copy

a = ([1, 2, 3, 4],["ab", "cd"])

b = copy.copy(a)

c = copy.deepcopy(a)


print("a的地址是%s:,值是：%s" %(id(a),a))
print("b的地址是%s:,值是：%s" %(id(b),b))
print("c的地址是%s:,值是：%s" %(id(c),c))

print("********************************")
a[0].append(1)

print("a的地址是%s:,值是：%s" %(id(a),a))
print("b的地址是%s:,值是：%s" %(id(b),b))
print("c的地址是%s:,值是：%s" %(id(c),c))

# a的地址是140518445333120:,值是：[[1, 2, 3, 4], ['ab', 'cd']]
# b的地址是140518445249152:,值是：[[1, 2, 3, 4], ['ab', 'cd']]
# c的地址是140518445333824:,值是：[[1, 2, 3, 4], ['ab', 'cd']]
# ********************************
# a的地址是140518445333120:,值是：[[1, 2, 3, 4, 1], ['ab', 'cd']]
# b的地址是140518445249152:,值是：[[1, 2, 3, 4, 1], ['ab', 'cd']]
# c的地址是140518445333824:,值是：[[1, 2, 3, 4], ['ab', 'cd']]


```

> ❝
> 
> 最外层是不可变
> 
> ((),()) 完全的不可变类型, 深拷贝和浅拷贝都是指向, 不会生成新的对象
> 
> ❞

```
# 2023/5/13 23:01 , @File: demo4.py, @Comment : python3.9.7
# 最外层是不可变类型
# - ([],[]),  copy(浅拷贝) 仅仅只是引用, 浅拷贝只拷贝最外层,内层就是引用关系, 最外层不可变的,拷贝生成的新对象不能够做任何的修改, 所以就直接引用, deepcopy(深拷贝) 会拷贝得到全新的对象,深拷贝会从内到外都拷贝,  ([],[])内层是可变的有可能需要对内层的对象做修改, 所以依然会得到新的对象
# - ((),()) 完全的不可变类型, 深拷贝和浅拷贝都是指向, 不会生成新的对象
# - ((),[]) 浅拷贝是指向, 不会生成新的对象,  深拷贝: 如果内层是不可变就是引用,内层是可变就会拷贝得到全新的对象

import copy

a = ((1, 2, 3, 4),("ab", "cd"))

b = copy.copy(a)

c = copy.deepcopy(a)


print("a的地址是%s:,值是：%s" %(id(a),a))
print("b的地址是%s:,值是：%s" %(id(b),b))
print("c的地址是%s:,值是：%s" %(id(c),c))

print("********************************")


# a,b,c 的地址都一样，完全不可变，都是引用
# a的地址是140197396272640:,值是：((1, 2, 3, 4), ('ab', 'cd'))
# b的地址是140197396272640:,值是：((1, 2, 3, 4), ('ab', 'cd'))
# c的地址是140197396272640:,值是：((1, 2, 3, 4), ('ab', 'cd'))
# ********************************


```

> ❝
> 
> 最外层是不可变
> 
> ((),[]) 浅拷贝是指向, 不会生成新的对象,  深拷贝: 如果内层是不可变就是引用, 内层是可变就会拷贝得到全新的对象
> 
> ❞

```
# 2023/5/13 23:09 , @File: demo5.py, @Comment : python3.9.7
# 最外层是不可变类型
# - ([],[]),  copy(浅拷贝) 仅仅只是引用, 浅拷贝只拷贝最外层,内层就是引用关系, 最外层不可变的,拷贝生成的新对象不能够做任何的修改, 所以就直接引用, deepcopy(深拷贝) 会拷贝得到全新的对象,深拷贝会从内到外都拷贝,  ([],[])内层是可变的有可能需要对内层的对象做修改, 所以依然会得到新的对象
# - ((),()) 完全的不可变类型, 深拷贝和浅拷贝都是指向, 不会生成新的对象
# - ((),[]) 浅拷贝是指向, 不会生成新的对象,  深拷贝: 如果内层是不可变就是引用,内层是可变就会拷贝得到全新的对象

import copy

a1 = [1, 23, 4]
a2 = ("ab", "cd")
a = (a1, a2)

b = copy.copy(a)

c = copy.deepcopy(a)

a1.append(5)

print("a的地址是%s:,值是：%s" % (id(a), a))
print("b的地址是%s:,值是：%s" % (id(b), b))
print("c的地址是%s:,值是：%s" % (id(c), c))

print("********************************")
# a的地址是140693331230976:,值是：([1, 23, 4, 5], ('ab', 'cd'))
# b的地址是140693331230976:,值是：([1, 23, 4, 5], ('ab', 'cd'))
# c的地址是140694136733056:,值是：([1, 23, 4], ('ab', 'cd'))
# ********************************



```

补充资料
----

看到一个解释，供大家参考：

> ❝
> 
> 首先深拷贝和浅拷贝都是对原对象的拷贝，都会生成一个看起来相同的对象，本质区别就是拷贝出来的对象的「地址」是否与原对象一样，即就是对原对象的地址的拷贝，还是值的拷贝
> 
> **「深拷贝：」**对原对象的地址的拷贝，新拷贝了一份与原对象不同的地址的对象，修改对象中的任何值，都不会改变深拷贝的对象的值。
> 
> **「浅拷贝：」**对原对象的值的拷贝，地址仍是一个指针指向原对象的地址，浅拷贝或者原对象的值发生变化，那原对象和浅拷贝对象的值都会随着被改变。
> 
> 浅拷贝（影子克隆）：只复制对象的基本类型，对象类型，仍属于原来的引用
> 
> 深拷贝（深度克隆）：不仅复制对象的基本类，同时也复制原对象的对象，完全是新对象产生的
> 
> 1.copy.copy 浅拷贝——只拷贝对象，不会拷贝对象的引用对象，不会拷贝原始对象的内部的
> 
> 2.copy.deepcopy  深拷贝——拷贝对象的值类型，还拷贝了原始对象，而产生了一个新的对象，不仅仅只拷贝了原始对象的引用
> 
> **「二、补充概念：可变对象，不可变对象」**
> 
> 可变对象：一个对象在不改变其所指向的地址前提下，可以修改其所执行的地址中的值
> 
> 不可变对象：一个对象所指向的地址上的值是不能修改的，如果修改了就是它执行的地址就改变了，相当于将这个对象指向的值复制出来一份，然后做了修改后存到另一个地址上了。
> 
> 区别：可变对象就不会这样会修改值后另存到一个新的地址上，而是直接再原对象的地址上把值给改变了，这个对象依然执行这个地址
> 
> 本质区别：可变对象修改了值，不会新建一个内存地址的对象，不可变对象如果修改了值，及时复制了一份新的内存地址，原始地址的值不会被改变。
> 
> ❞

参考资料
----

<table><thead><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;">序号</th><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;" class="">参考链接</th><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;">备注</th></tr></thead><tbody><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;">1</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;" class="">某培训机构的课堂笔记</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;"><br></td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;">2</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;">https://blog.csdn.net/Mielt/article/details/119141586</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;"><br></td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;">3</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;">https://zhuanlan.zhihu.com/p/57893374</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;">5 张图彻底理解 Python 中的浅拷贝与深拷贝</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;">4</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;">https://blog.csdn.net/lovedingd/article/details/128257172</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;"><br></td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;">5</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;">https://docs.python.org/3/library/copy.html</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;">官方文档自然少不了的</td></tr></tbody></table>
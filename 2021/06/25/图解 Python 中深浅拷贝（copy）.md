> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg4NzYyNjU2NQ==&mid=2247492561&idx=3&sn=119ccc55c09e5ba266844395630fcb9d&chksm=cf852dfbf8f2a4edd4bd3cb78ae98ffbf10040f54201251eab1de610af5fe27a299c6a44bce4&mpshare=1&scene=1&srcid=0623FJbAxKjZa7kLYFZwiS1A&sharer_sharetime=1624421944965&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> https://blog.csdn.net/mall_lucy/article/details/104531218

在工作中，常涉及到数据的传递，在数据传递使用过程中，可能会发生数据被修改的问题。为了防止数据被修改，就需要在传递一个副本，即使副本被修改，也不会影响原数据的使用。为了生成这个副本，就产生了拷贝。今天就说一下 Python 中的深浅拷贝问题。

#### **一、深浅 copy**

1.  **赋值运算**
    

```
l1 = [1, 2, 3, [22, 33]]
l2 = l1
l1.append(666)
print(l1)  # [1, 2, 3, [22, 33], 666]
print(l2)  # [1, 2, 3, [22, 33], 666]

```

注意：l2 = l1 是一个指向，是赋值，和深浅 copy 无关。

2.  **浅 copy**
    

其实列表是一个一个的槽位，每个槽位存储的是该对象的内存地址

```
#例1. 给大列表添加元素
l1 = [1, 2, 3, [22, 33]]
l2 = l1.copy()
# 或者下面这种方式，也是浅copy
# import copy
# l2 = copy.copy(l1)
l1.append(666)
print(l1)  # [1, 2, 3, [22, 33], 666]
print(l2)  # [1, 2, 3, [22, 33]]
#例2. 给小列表添加元素
l1 = [1, 2, 3, [22, 33]]
l2 = l1.copy()
l1[-1].append(666)
print(l1)  # [1, 2, 3, [22, 33, 666]]
print(l2)  # [1, 2, 3, [22, 33, 666]]、
例3. 将l1列表中第一个元素改为6
l1 = [1, 2, 3, [22, 33]]
l2 = l1.copy()
l1[0] = 6
print(l1)  # [6, 2, 3, [22, 33]]
print(l2)  # [1, 2, 3, [22, 33]]

```

图解：

例 1

![](https://mmbiz.qpic.cn/mmbiz_png/fhujzoQe7TrGa4hyhWRYBIYJImp81lDuYCscqWWe9bmA5qTddKaTLjDWZWIvzvqxQaz925FDyS5Sbq7aMLMKJA/640?wx_fmt=png)

例 2  

![](https://mmbiz.qpic.cn/mmbiz_png/fhujzoQe7TrGa4hyhWRYBIYJImp81lDuaYQNIb7icoqbV5qzqRrAIMGibNeWicsSBib8Lp5Sd7JCEiap1QMcKz8f07A/640?wx_fmt=png)

例 3

![](https://mmbiz.qpic.cn/mmbiz_png/fhujzoQe7TrGa4hyhWRYBIYJImp81lDu4oHzsVPTIO9wHiaVvtp4PbGKRW0Nic04DF5GiaUgU1vEO7SsosGgmyf4w/640?wx_fmt=png)

**小结：**

浅 copy：会在内存中新开辟一个空间，存放这个 copy 的列表，但是列表里面的内容还是沿用之前对象的内存地址。

3.  **深 copy**
    

```
import copy
l1 = [1, 2, 3, [22, 33]]
l2 = copy.deepcopy(l1)
l1.append(666)
print(l1)  # [1, 2, 3, [22, 33], 666]
print(l2)  # [1, 2, 3, [22, 33]]

```

图解：

本质如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/fhujzoQe7TrGa4hyhWRYBIYJImp81lDuwSY78wUgLnSJSl2LV3P3CAXxDVdYRfT3h3ibIF3uXDg5SNq6FPBAvnQ/640?wx_fmt=png)

但是 python 对深 copy 做了一个优化，将可变的数据类型在内存中重新创建一份，而不可变的数据类型则沿用之前的，所以内存中是下面这样的：

![](https://mmbiz.qpic.cn/mmbiz_png/fhujzoQe7TrGa4hyhWRYBIYJImp81lDuXN5hiav6PepmCeGPASD1m9WS4FYgUjq5mj9tnSkCanfymavZZJZsb0w/640?wx_fmt=png)

**小结：**

深 copy：会在内存中开辟新空间，将原列表以及列表里面的可变数据类型重新创建一份，不可变数据类型则沿用之前的。

**为什么 Python 默认的拷贝方式是浅拷贝？**

*   时间角度：浅拷贝花费时间更少。
    
*   空间角度：浅拷贝花费内存更少。
    
*   效率角度：浅拷贝只拷贝顶层数据，一般情况下比深拷贝效率高。
    
      
    

**总结：**

*   不可变对象在赋值时会开辟新空间。
    
*   可变对象在赋值时，修改一个的值，另一个也会发生改变。
    
*   深、浅拷贝对不可变对象拷贝时，不开辟新空间，相当于赋值操作。
    
*   浅拷贝在拷贝时，只拷贝第一层中的引用，如果元素是可变对象，并且被修改，那么拷贝的对象也会发生变化。
    
*   深拷贝在拷贝时，会逐层进行拷贝，直到所有的引用都是不可变对象为止。
    
*   Python 有多种方式实现浅拷贝，copy 模块的 copy 函数 ，对象的 copy 函数 ，工厂方法，切片等。
    
*   大多数情况下，编写程序时，都是使用浅拷贝，除非有特定的需求。
    
*   浅拷贝的优点：拷贝速度快，占用空间少，拷贝效率高。
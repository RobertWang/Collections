> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NzU0MzU0Nw==&mid=2651398413&idx=3&sn=c4919baa506bf245b62168dc506077f8&chksm=bd25ea198a52630f08fc173565e6980622541ed604881b180a2fc55ab741bbc0bc4f039ead23&mpshare=1&scene=1&srcid=0713GoEaoJl7V9yDeKONd1D3&sharer_sharetime=1626142110515&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> 公众号：机器之心
> 
> 作者：George Seif

> Python 多好用不用多说，大家看看自己用的语言就知道了。但是 Python 隐藏的高级功能你都 get 了吗？本文中，作者列举了 Python 中五种略高级的特征以及它们的使用方法，快来一探究竟吧！

```
def square_it_func(a):
    return a * a

x = map(square_it_func, [1, 4, 7])
print(x) # prints '[1, 16, 49]'

def multiplier_func(a, b):
    return a * b

x = map(multiplier_func, [1, 4, 7], [2, 5, 8])
print(x) # prints '[2, 20, 56]'看看上面的示例！我们可以将函数应用于单个或多个列表。实际上，你可以使用任何 Python 函数作为 map 函数的输入，只要它与你正在操作的序列元素是兼容的。

```

```
# Our numbers
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15]

# Function that filters out all numbers which are odd
def filter_odd_numbers(num):

    if num % 2 == 0:
        return True
    else:
        return False

filtered_numbers = filter(filter_odd_numbers, numbers)

print(filtered_numbers)
# filtered_numbers = [2, 4, 6, 8, 10, 12, 14]

```

我们不仅评估了每个列表元素的 True 或 False，filter() 函数还确保只返回匹配为 True 的元素。非常便于处理检查表达式和构建返回列表这两步。

Python 的 Itertools 模块是处理迭代器的工具集合。迭代器是一种可以在 for 循环语句（包括列表、元组和字典）中使用的数据类型。

```
from itertools import *

# Easy joining of two lists into a list of tuples
for i in izip([1, 2, 3], ['a', 'b', 'c']):
    print i
# ('a', 1)
# ('b', 2)
# ('c', 3)

# The count() function returns an interator that 
# produces consecutive integers, forever. This 
# one is great for adding indices next to your list 
# elements for readability and convenience
for i in izip(count(1), ['Bob', 'Emily', 'Joe']):
    print i
# (1, 'Bob')
# (2, 'Emily')
# (3, 'Joe')    

# The dropwhile() function returns an iterator that returns 
# all the elements of the input which come after a certain 
# condition becomes false for the first time. 
def check_for_drop(x):
    print 'Checking: ', x
    return (x > 5)

for i in dropwhile(should_drop, [2, 4, 6, 8, 10, 12]):
    print 'Result: ', i

# Checking: 2
# Checking: 4
# Result: 6
# Result: 8
# Result: 10
# Result: 12


# The groupby() function is great for retrieving bunches
# of iterator elements which are the same or have similar 
# properties

a = sorted([1, 2, 1, 3, 2, 1, 2, 3, 4, 5])
for key, value in groupby(a):
    print(key, value), end=' ')

# (1, [1, 1, 1])
# (2, [2, 2, 2]) 
# (3, [3, 3]) 
# (4, [4]) 
# (5, [5]) 

```

**Generator 函数**

比如，我们想把 1 到 1000 的所有数字相加，以下代码块的第一部分向你展示了如何使用 for 循环来进行这一计算。

代码中第二部分展示了使用 Python generator 函数对数字列表求和。generator 函数创建元素，并只在必要时将其存储在内存中，即一次一个。这意味着，如果你要创建十亿浮点数，你只能一次一个地把它们存储在内存中！Python 2.x 中的 xrange() 函数就是使用 generator 来构建列表。

也就是说，如果你想对列表进行多次迭代，并且它足够小，可以放进内存，那最好使用 for 循环或 Python 2.x 中的 range 函数。因为 generator 函数和 xrange 函数将会在你每次访问它们时生成新的列表值，而 Python 2.x range 函数是静态的列表，而且整数已经置于内存中，以便快速访问。

```
# (1) Using a for loopv
numbers = list()

for i in range(1000):
    numbers.append(i+1)

total = sum(numbers)

# (2) Using a generator
 def generate_numbers(n):
     num, numbers = 1, []
     while num < n:
           numbers.append(num)
     num += 1
     return numbers
 total = sum(generate_numbers(1000))

 # (3) range() vs xrange()
 total = sum(range(1000 + 1))
 total = sum(xrange(1000 + 1))


```

介绍一本非常经典的入门 PDF，它讲解的是程序员必知的硬核基础知识，看完能让你对计算机有一个基础的了解和入门，是培养你 `内核` 的基础，我们看下目录大纲
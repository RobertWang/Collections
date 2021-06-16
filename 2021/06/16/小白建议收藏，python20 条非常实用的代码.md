> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/380939584)

Python 一直以来被诟病速度慢，影响开发效率，希望这次 Guido 老爷子能帮 python 打一场漂亮的翻身仗。

这篇文章不准备介绍 Python 速度如何，而是给大家带来一些常用且实用的 Python 代码实例，几乎是开发者必备的知识点。

**1、合并两个字典**
------------

Python3.5 之后，合并字典变得容易起来。我们可以通过`**`符号解压字典，并将多个字典传入`{}`中，实现合并。

```
def Merge(dict1, dict2): 
    res = {**dict1, **dict2} 
    return res 
      
# 两个字典
dict1 = {"name": "Joy", "age": 25}
dict2 = {"name": "Joy", "city": "New York"}
dict3 = Merge(dict1, dict2) 
print(dict3)
```

输出：

```
{'name': 'Joy', 'age': 25, 'city': 'New York'}
```

**2、链式比较**
----------

python 有链式比较的机制，在一行里支持多种运算符比较。相当于拆分多个逻辑表达式，再进行逻辑与操作。

```
a = 5

print(2 8)
print(1 == a 3)
```

输出：

```
True
False
```

**3、重复打印字符串**
-------------

将一个字符串重复打印多次，一般使用循环实现，但有更简易的方式可以实现。

```
n = 5
string = "Hello!"

print(string * n)
```

输出：

```
Hello!Hello!Hello!Hello!Hello!
```

**4、检查文件是否存在**
--------------

我们知道 Python 有专门处理系统交互的模块 - os，它可以处理文件的各种增删改查操作。

那如何检查一个文件是否存在呢？os 模块可以轻松实现。

```
from os import path

def check_for_file():
    print("Does file exist:", path.exists("data.csv"))

if __name__=="__main__":
    check_for_file()
```

输出：

```
Does file exist: False
```

**5、检索列表最后一个元素**
----------------

在使用列表的时候，有时会需要取最后一个元素，有下面几种方式可以实现。

```
my_list = ['banana', 'apple', 'orange', 'pineapple']

#索引方法
last_element = my_list[-1]

#pop方法
last_element = my_list.pop()
```

输出：

```
'pineapple'
```

**6、列表推导式**
-----------

列表推导式是 for 循环的简易形式，可以在一行代码里创建一个新列表，同时能通过 if 语句进行判断筛选

```
def get_vowels(string):
    return [vowel for vowel in string if vowel in 'aeiou'] 

print("Vowels are:", get_vowels('This is some random string'))
```

输出：

```
Vowels are:  ['i', 'i', 'o', 'e', 'a', 'o', 'i']
```

**7、计算代码执行时间**
--------------

python 中 time 模块提供了时间处理相关的各种函数方法，我们可以使用它来计算代码执行的时间。

```
import time

start_time = time.time()

total = 0
for i in range(10):
  total += i
print("Sum:", total)

end_time = time.time()
time_taken = end_time - start_time
print("Time: ", time_taken)
```

输出：

```
Sum: 45
Time:  0.0009975433349609375
```

**8、查找出现次数最多的元素**
-----------------

使用 max 方法找出列表中出现次数最多的元素。

```
def most_frequent(list):
  return max(set(list), key=list.count)

mylist = [1,1,2,3,4,5,6,6,2,2]
print("出现次数最多的元素是:", most_frequent(mylist))
```

输出：

```
出现次数最多的元素是: 2
```

**9、将两个列表转换为字典**
----------------

有两个列表，将列表 A 里的元素作为键，将列表 B 里的对应元素作为值，组成一个字典。

```
def list_to_dictionary(keys, values):
  return dict(zip(keys, values))

list1 = [1, 2, 3]
list2 = ['one', 'two', 'three']

print(list_to_dictionary(list1, list2))
```

输出：

```
{1: 'one', 2: 'two', 3: 'three'}
```

**10、异常处理**
-----------

Python 提供了 try...except...finally 的方式来处理代码异常，当然还有其他组合的方式。

```
a, b = 1,0

try:
    print(a/b)
except ZeroDivisionError:
    print("Can not divide by zero")
finally:
    print("Executing finally block")
```

输出：

```
Can not divide by zero
Executing finally block
```

**11、反转字符串**
------------

使用切片操作对字符串进行反转，这是比较直接有效的方式。这也可以用来检测回文数。

```
str = "Hello World"

print("反转后字符串是:", str[::-1])
```

输出：

```
反转后字符串是: dlroW olleH
```

**12、字符串列表组成单个字符串**
-------------------

使用 join 方法将字符串列表组成单个字符串。

```
list = ["Hello", "world", "Ok", "Bye!"]
combined_string = " ".join(list)

print(combined_string)
```

输出：

```
Hello world Ok Bye!
```

**13、返回字典缺失键的默认值**
------------------

字典中的 get 方法用于返回指定键的值，如果键不在字典中返回默认值 None 或者设置的默认值。

```
dict = {1:'one', 2:'two', 4:'four'}

#returning three as default value
print(dict.get(3, 'three'))

print("原始字典:", dict)
```

输出：

```
three
原始字典: {1: 'one', 2: 'two', 4: 'four'}
```

**14、交换两个变量的值**
---------------

在不使用临时变量的前提下，交换两个变量的值。

```
a, b = 5, 10

# 方法1
a, b = b, a

# 方法2
def swap(a,b):
  return b,a
swap(a,b)
```

**15、正则表达式**
------------

正则表达式用来匹配处理字符串，python 中的 re 模块提供了全部的正则功能。

```
import re

text = "The rain in spain"
result = re.search("rain", text)

print(True if result else False)
```

输出：

```
True
```

**16、筛选值**
----------

python 中的 filter 方法可以用来进行值的筛选。

```
my_list = [0,1,2,3,6,7,9,11]

result = filter(lambda x: x % 2!=0, my_list)

print(list(result))
```

输出：

```
[1, 3, 7, 9, 11]
```

**17、统计字频**
-----------

判断字符串每个元素出现的次数，可以用 collections 模块中的 Counter 方法来实现，非常简洁。

```
from collections import Counter
result = Counter('banana')
print(result)
```

输出：

```
Counter({'a': 3, 'n': 2, 'b': 1})
```

**18、变量的内存占用**
--------------

如何输出 python 中变量的内存占用大小，可以通过 sys 模块来实现。

```
import sys

var1 = 15
list1 = [1,2,3,4,5]

print(sys.getsizeof(var1))
print(sys.getsizeof(list1))
```

输出：

**19、链式函数调用**
-------------

在一行代码中调用多个函数。

```
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

a, b = 5, 10

print((add if b > a else subtract)(a,b))
```

输出：

**20、从列表中删除重复项**
----------------

删除列表中重复项一般可以通过遍历来筛选去重，或者直接使用集合方法。

```
list1 = [1,2,3,3,4,'John', 'Ana', 'Mark', 'John']

# 方法1
def remove_duplicate(list_value):
    return list(set(list_value))
print(remove_duplicate(list1))

# 方法2
result = []
[result.append(x) for x in list1 if x not in result]
print(result)
```

输出：

```
[1, 2, 3, 4, 'Ana', 'John', 'Mark']
[1, 2, 3, 4, 'John', 'Ana', 'Mark']
```
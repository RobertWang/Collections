> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU3NjE4NjQ4MA==&mid=2247516903&idx=2&sn=c2d339460f4fec14c73e4c74f8cc5be0&chksm=fd1573f8ca62faeea801da2b30b28ed068014b8a876ec0055840d217ee2e8136960316a48748&mpshare=1&scene=1&srcid=0727jAUTQSyOQLNetQQpmsBQ&sharer_sharetime=1627375096527&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

今天和大家来分享几个关于 Python 的小技巧，都是非常简单易懂的内容，希望大家看了之后能够有所收获。  

```
my_string = "my name is xiao ming"
# 通过title()来实现首字母大写
new_string = my_string.title()
print(new_string)
-------------------------------------
# output
# My Name Is Xiao Ming
```

**0****3**

  

**给字符串去重**

**0****4**

  

**拆分字符串**

Python split() 通过指定分隔符对字符串进行切片，默认的分隔符是 " "

```
string_1 = "My name is xiao ming"
string_2 = "sample, string 1, string 2"
# 默认的分隔符是空格，来进行拆分
print(string_1.split())
# 根据分隔符"，"来进行拆分
print(string_2.split(','))
------------------------------------
# output
# ['My', 'name', 'is', 'xiao', 'ming']
# ['sample', ' string 1', ' string 2']
```

**0****5**

  

**将字典中的字符串连词成串**

```
list_of_strings = ['My', 'name', 'is', 'Xiao', 'Ming']
# 通过空格和join来连词成句
print(' '.join(list_of_strings))
-----------------------------------------
# output
# My name is Xiao Ming
```

**0****6**

  

**查看列表中各元素出现的个数**

```
from collections import Counter
my_list = ['a','a','b','b','b','c','d','d','d','d','d']
count = Counter(my_list) 
print(count) 
# Counter({'d': 5, 'b': 3, 'a': 2, 'c': 1})
print(count['b']) # 单独的“b”元素出现的次数
# 3
print(count.most_common(1)) # 出现频率最多的元素
# [('d', 5)]
```

**0****7**

  

**合并两字典**

```
dict_1 = {'apple': 9, 'banana': 6}
dict_2 = {'grape': 4, 'orange': 8}
# 方法一
combined_dict = {**dict_1, **dict_2}
print(combined_dict)
# 方法二
dict_1.update(dict_2)
print(dict_1)
# 方法三
print(dict(dict_1.items() | dict_2.items()))
---------------------------------------
# output 
# {'apple': 9, 'banana': 6, 'grape': 4, 'orange': 8}
# {'apple': 9, 'banana': 6, 'grape': 4, 'orange': 8}
# {'apple': 9, 'banana': 6, 'grape': 4, 'orange': 8}
```

```
import time
start_time = time.time()
########################
# 具体的程序..........
########################
end_time = time.time()
time_taken_in_micro = (end_time- start_time) * (10 ** 6)
print(time_taken_in_micro)
```

有时候会存在列表当中还嵌套着列表的情况，

```
from iteration_utilities import deepflatten
l = [[1,2,3],[4,[5],[6,7]],[8,[9,[10]]]]
print(list(deepflatten(l, depth=3)))
-----------------------------------------
# output
# [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

**10**

  

**查看列表当中是否存在重复值**

```
def unique(l):
    if len(l)==len(set(l)):
        print("不存在重复值")
    else:
        print("存在重复值")
unique([1,2,3,4])
# 不存在重复值
unique([1,1,2,3])
# 存在重复值
```

**11**

  

**数组的转置**

```
array = [['a', 'b'], ['c', 'd'], ['e', 'f']]
transposed = zip(*array)
print(list(transposed)) 
------------------------------------------
# output
# [('a', 'c', 'e'), ('b', 'd', 'f')]
```

**12**

  

**找出两列表当中的不同元素**

```
def difference(a, b):
    set_a = set(a)
    set_b = set(b)
    comparison = set_a.difference(set_b)
    return list(comparison)
# 返回第一个列表的不同的元素
difference([1,2,6], [1,2,5])
# [6]
```

**13**

  

**将两列表变成键值对**

将两个列表合并成一个键值对的字典

```
def to_dictionary(keys, values):
    return dict(zip(keys, values))
keys = ["a", "b", "c"]    
values = [2, 3, 4]
print(to_dictionary(keys, values))
-------------------------------------------
# output
# {'a': 2, 'b': 3, 'c': 4}
```

**14**

  

**对字典进行排序**

根据字典当中的值对字典进行排序

```
d = {'apple': 9, 'grape': 4, 'banana': 6, 'orange': 8}
# 方法一
sorted(d.items(), key = lambda x: x[1]) # 从小到大排序
# [('grape', 4), ('banana', 6), ('orange', 8), ('apple', 9)]
sorted(d.items(), key = lambda x: x[1], reverse = True) # 从大到小排序
# [('apple', 9), ('orange', 8), ('banana', 6), ('grape', 4)]
# 方法二
from operator import itemgetter
print(sorted(d.items(), key = itemgetter(1)))
# [('grape', 4), ('banana', 6), ('orange', 8), ('apple', 9)]
```

**15**

  

**列表中最大 / 最小值的索引**

```
list1 = [20, 30, 50, 70, 90]
def max_index(list_test):
    return max(range(len(list_test)), key = list_test.__getitem__)
def min_index(list_test):
    return min(range(len(list_test)), key = list_test.__getitem__)
max_index(list1)
# 4
min_index(list1)
# 0
```
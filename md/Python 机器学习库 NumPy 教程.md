> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [paul.pub](https://paul.pub/numpy-tutorial/)

> Python 机器学习库 NumPy 教程, AI, Python,MachineLearning,NumPy, NumPy 是一个 Python 语言的软件包，它非常适合于科学计算。

NumPy 是一个 Python 语言的软件包，它非常适合于科学计算。在我们使用 Python 语言进行机器学习编程的时候，这是一个非常常用的基础库。

本文是对它的一个入门教程。

*   [介绍](#id-介绍)
*   [获取 NumPy](#id-获取numpy)
*   [基础属性与数组创建](#id-基础属性与数组创建)
    *   [特定 array 的创建](#id-特定array的创建)
*   [Shape 与操作](#id-shape与操作)
*   [索引](#id-索引)
*   [数学运算](#id-数学运算)
*   [矩阵](#id-矩阵)
*   [随机数](#id-随机数)
*   [参考资料与推荐读物](#id-参考资料与推荐读物)

NumPy 是一个用于科技计算的基础软件包，它是 Python 语言实现的。它包含了：

*   强大的 N 维数组结构
*   精密复杂的函数
*   可集成到 C/C++ 和 Fortran 代码的工具
*   线性代数，傅里叶变换以及随机数能力

除了科学计算的用途以外，NumPy 也可被用作高效的通用数据的多维容器。由于它适用于任意类型的数据，这使得 NumPy 可以无缝和高效的集成到多种类型的数据库中。

由于这是一个 Python 语言的软件包，因此需要你的机器上首先需要具备 Python 语言的环境。关于这一点，请自行在网络上搜索获取方法。

关于如何获取 NumPy 也请参阅`scipy.org`官网上的 [Installing packages](https://scipy.org/install.html)。本文不再赘述。

笔者推荐使用`pip`的方式安装 Python 包，命令如下：

本文的代码在如下的环境中验证和测试：

*   硬件：MacBook Pro 2015
*   OS：macOS High Sierra
*   语言环境：Python 3.6.2
*   软件包：numpy 1.13.3

可以在这里获取到本文的所有源码：[https://github.com/paulQuei/numpy_tutorial](https://github.com/paulQuei/numpy_tutorial)

另外，

*   为了简单起见，本文我们会通过 Python 的`print`函数来进行结果的验证
*   为了拼写方便，我们会默认`import numpy as np`

NumPy 的基础是一个同构的多维数据，数组中的元素可以通过下标来索引。在 NumPy 中，维度称之为`axis`（复数是`axes`），维度的数量称之为`rank`。

例如：

下面是一个具有 rank 1 的数组，axis 的长度为 3：

下面是一个具有 rank 2 的数组，axis 的长度也是 3：

我们可以通过`array`函数来创建 NumPy 的数组，例如这样：

```
a = np.array([1, 2, 3])
b = np.array([(1,2,3), (4,5,6)])


```

请注意，这里方括号是必须的，下面这种写法是错误的：

```
a = np.array(1,2,3,4) # WRONG!!!


```

NumPy 的数组类是`ndarray`，它有一个别名是 `numpy.array`，但这与 Python 标准库的`array.array`并不一样。后者仅仅是一个一维数组。而`ndarray`具有以下的属性：

*   `ndarray.ndim`：数组的维数。在 Python 世界中，维数称之为`rank`
*   `ndarray.shape`：数组的维度。这是一系列数字，长度由数组的维度（`ndim`）决定。例如：长度为 n 的一维数组的`shape`是 (n)。一个 n 行 m 列的矩阵的`shape`是 (n,m)
*   `ndarray.size`：数组中所有元素的数量
*   `ndarray.dtype`：数组中元素的类型，例如`numpy.int32`, `numpy.int16`或者`numpy.float64`
*   `ndarray.itemsize`：数组中每个元素的大小，单位为字节
*   `ndarray.data`：存储数组元素的缓冲。通常我们只需要通过下标来访问元素，而不需要访问缓冲

下面我们来看一下代码示例：

```
# create_array.py

import numpy as np

a = np.array([1, 2, 3])
b = np.array([(1,2,3), (4,5,6)])

print('a=')
print(a)
print("a's ndim {}".format(a.ndim))
print("a's shape {}".format(a.shape))
print("a's size {}".format(a.size))
print("a's dtype {}".format(a.dtype))
print("a's itemsize {}".format(a.itemsize))

print('')

print('b=')
print(b)
print("b's ndim {}".format(b.ndim))
print("b's shape {}".format(b.shape))
print("b's size {}".format(b.size))
print("b's dtype {}".format(b.dtype))
print("b's itemsize {}".format(b.itemsize))


```

下面是这段代码的输出：

```
a=
[1 2 3]
a's ndim 1
a's shape (3,)
a's size 3
a's dtype int64
a's itemsize 8

b=
[[1 2 3]
 [4 5 6]]
b's ndim 2
b's shape (2, 3)
b's size 6
b's dtype int64
b's itemsize 8


```

我们也可以在创建数组的时候，指定元素的类型，例如这样：

```
c = np.array( [ [1,2], [3,4] ], dtype=complex )


```

关于 array 函数的更多参数说明，请参见这里：[numpy.array](https://docs.scipy.org/doc/numpy/reference/generated/numpy.array.html)

> 注：NumPy 本身支持多维数组，也支持各种类型元素的数据。但考虑到，三维及以上的数组结构并不容易理解，而且我们在进行机器学习编程的时候，用的最多的是矩阵运算。因此，本文接下来的例子主要以一维和二维数字型数组来进行示例说明。

特定 array 的创建
------------

在实际上的项目工程中，我们常常会需要一些特定的数据，NumPy 中提供了这么一些辅助函数:

*   `zeros`：用来创建元素全部是 0 的数组
*   `ones`：用来创建元素全部是 1 的数组
*   `empty`：用来创建未初始化的数据，因此是内容是不确定的
*   `arange`：通过指定范围和步长来创建数组
*   `linespace`：通过指定范围和元素数量来创建数组
*   `random`：用来生成随机数

```
# create_specific_array.py

import numpy as np

a = np.zeros((2,3))
print('np.zeros((2,3)= \n{}\n'.format(a))

b = np.ones((2,3))
print('np.ones((2,3))= \n{}\n'.format(b))

c = np.empty((2,3))
print('np.empty((2,3))= \n{}\n'.format(c))

d = np.arange(1, 2, 0.3)
print('np.arange(1, 2, 0.3)= \n{}\n'.format(d))

e = np.linspace(1, 2, 7)
print('np.linspace(1, 2, 7)= \n{}\n'.format(e))

f = np.random.random((2,3))
print('np.random.random((2,3))= \n{}\n'.format(f))


```

这段代码的输出如下

```
np.zeros((2,3)= 
[[ 0.  0.  0.]
 [ 0.  0.  0.]]

np.ones((2,3))= 
[[ 1.  1.  1.]
 [ 1.  1.  1.]]

np.empty((2,3))= 
[[ 1.  1.  1.]
 [ 1.  1.  1.]]

np.arange(1, 2, 0.3)= 
[ 1.   1.3  1.6  1.9]

np.linspace(1, 2, 7)= 
[ 1.          1.16666667  1.33333333  1.5         1.66666667  1.83333333
  2.        ]

np.random.random((2,3))= 
[[ 0.5744616   0.58700653  0.59609648]
 [ 0.0417809   0.23810732  0.38372978]]


```

除了生成数组之外，当我们已经持有某个数据之后，我们可能会需要根据已有数组来产生一些新的数据结构，这时候我们可以使用下面这些函数：

*   `reshape`：根据已有数组和指定的 shape，生成一个新的数组
*   `vstack`：用来将多个数组在垂直（v 代表 vertical）方向拼接（数组的维度必须匹配）
*   `hstack`：用来将多个数组在水平（h 代表 horizontal）方向拼接（数组的维度必须匹配）
*   `hsplit`：用来将数组在水平方向拆分
*   `vsplit`：用来将数组在垂直方向拆分

下面我们通过一些例子来进行说明。

为了便于测试，我们先创建几个数据。这里我们创建了：

*   `zero_line`：一行包含 3 个 0 的数组
*   `one_column`：一列包含 3 个 1 的数组
*   `a`：一个 2 行 3 列的矩阵
*   `b`：[11, 20) 区间的整数数组

```
# shape_manipulation.py

zero_line = np.zeros((1,3))
one_column = np.ones((3,1))
print("zero_line = \n{}\n".format(zero_line))
print("one_column = \n{}\n".format(one_column))

a = np.array([(1,2,3), (4,5,6)])
b = np.arange(11, 20)
print("a = \n{}\n".format(a))
print("b = \n{}\n".format(b))


```

通过输出我们可以看到它们的结构：

```
zero_line = 
[[ 0.  0.  0.]]

one_column = 
[[ 1.]
 [ 1.]
 [ 1.]]

a = 
[[1 2 3]
 [4 5 6]]

b = 
[11 12 13 14 15 16 17 18 19]


```

数组 b 原先是一个一维数组，现在我们通过 reshape 方法将其调整成为一个 3 行 3 列的矩阵：

```
# shape_manipulation.py

b = b.reshape(3, -1)
print("b.reshape(3, -1) = \n{}\n".format(b))


```

这里的第二参数设为 - 1，表示根据实际情况自动决定。由于原先是 9 个元素的数组，因此调整后刚好是 3X3 的矩阵。这段代码输出如下：

```
b.reshape(3, -1) = 
[[11 12 13]
 [14 15 16]
 [17 18 19]]


```

接着，我们通过 vstack 函数，将三个数组在垂直方向拼接：

```
# shape_manipulation.py

c = np.vstack((a, b, zero_line))
print("c = np.vstack((a,b, zero_line)) = \n{}\n".format(c))


```

这段代码输出如下，请读者仔细观察一下拼接前后的数据结构：

```
c = np.vstack((a,b, zero_line)) = 
[[  1.   2.   3.]
 [  4.   5.   6.]
 [ 11.  12.  13.]
 [ 14.  15.  16.]
 [ 17.  18.  19.]
 [  0.   0.   0.]]


```

同样的，我们也可以通过 hstack 进行水平方向的拼接。为了可以拼接我们需要先将数组 a 调整一下结构：

```
# shape_manipulation.py

a = a.reshape(3, 2)
print("a.reshape(3, 2) = \n{}\n".format(a))

d = np.hstack((a, b, one_column))
print("d = np.hstack((a,b, one_column)) = \n{}\n".format(d))


```

这段代码输出如下，请再次仔细观察拼接前后的数据结构：

```
a.reshape(3, 2) = 
[[1 2]
 [3 4]
 [5 6]]

d = np.hstack((a,b, one_column)) = 
[[  1.   2.  11.  12.  13.   1.]
 [  3.   4.  14.  15.  16.   1.]
 [  5.   6.  17.  18.  19.   1.]]


```

请注意，如果两个数组的结构是不兼容的，拼接将无法完成。例如下面这行代码，它将无法执行：

```
# shape_manipulation.py

# np.vstack((a,b)) # ValueError: dimensions not match


```

这是因为数组 a 具有两列，而数组 b 具有 3 列，所以它们无法拼接。

接下来我们再看一下拆分。首先，我们将数组 d 在水平方向拆分成 3 个数组。然后我们将中间一个（下标是 1）数组打印出来：

```
# shape_manipulation.py

e = np.hsplit(d, 3) # Split a into 3
print("e = np.hsplit(d, 3) = \n{}\n".format(e))
print("e[1] = \n{}\n".format(e[1]))



```

这段代码输出如下：

```
e = np.hsplit(d, 3) = 
[array([[ 1.,  2.],
       [ 3.,  4.],
       [ 5.,  6.]]), array([[ 11.,  12.],
       [ 14.,  15.],
       [ 17.,  18.]]), array([[ 13.,   1.],
       [ 16.,   1.],
       [ 19.,   1.]])]

e[1] = 
[[ 11.  12.]
 [ 14.  15.]
 [ 17.  18.]]


```

另外，假设我们设置的拆分数量使得原先的数组无法平均拆分，则操作会失败：

```
# np.hsplit(d, 4) # ValueError: array split does not result in an equal division


```

除了指定数量平均拆分，我们也可以指定列数进行拆分。下面是将数组 d 从第 1 列和第 3 列两个地方进行拆分：

```
# shape_manipulation.py

f = np.hsplit(d, (1, 3)) # # Split a after the 1st and the 3rd column
print("f = np.hsplit(d, (1, 3)) = \n{}\n".format(f))


```

这段代码输出如下。数组 d 被拆分成了分别包含 1，2，3 列的三个数组：

```
f = np.hsplit(d, (1, 3)) = 
[array([[ 1.],
       [ 3.],
       [ 5.]]), array([[  2.,  11.],
       [  4.,  14.],
       [  6.,  17.]]), array([[ 12.,  13.,   1.],
       [ 15.,  16.,   1.],
       [ 18.,  19.,   1.]])]


```

最后我们再将数组 d 在垂直方向进行拆分。同样的，如果指定的拆分数无法平均拆分则会失败：

```
# shape_manipulation.py

g = np.vsplit(d, 3)
print("np.hsplit(d, 2) = \n{}\n".format(g))

# np.vsplit(d, 2) # ValueError: array split does not result in an equal division


```

`np.vsplit(d, 3)`将产生三个一维数组：

```
np.vsplit(d, 3) = 
[array([[  1.,   2.,  11.,  12.,  13.,   1.]]), array([[  3.,   4.,  14.,  15.,  16.,   1.]]), array([[  5.,   6.,  17.,  18.,  19.,   1.]])]


```

接下来我们看看如何访问 NumPy 数组中的数据。

同样的，为了测试方便，我们先创建一个一维数组。它的内容是 [100，200）区间的整数。

最基本的，我们可以通过`array[index]`的方式指定下标来访问数组的元素，这一点对于有一点编程经验的人来说应该都是很熟悉的。

```
# array_index.py

import numpy as np

base_data = np.arange(100, 200)
print("base_data\n={}\n".format(base_data))

print("base_data[10] = {}\n".format(base_data[10]))


```

上面这段代码输出如下：

```
base_data
=[100 101 102 103 104 105 106 107 108 109 110 111 112 113 114 115 116 117
 118 119 120 121 122 123 124 125 126 127 128 129 130 131 132 133 134 135
 136 137 138 139 140 141 142 143 144 145 146 147 148 149 150 151 152 153
 154 155 156 157 158 159 160 161 162 163 164 165 166 167 168 169 170 171
 172 173 174 175 176 177 178 179 180 181 182 183 184 185 186 187 188 189
 190 191 192 193 194 195 196 197 198 199]

base_data[10] = 110


```

在 NumPy 中，我们可以创建一个包含了若干个下标的数组来获取目标数组中的元素。如下所示：

```
# array_index.py
every_five = np.arange(0, 100, 5)
print("base_data[every_five] = \n{}\n".format(
    base_data[every_five]))


```

`every_five`是包含了我们要获取的下标的数组，它的内容大家应该很容易理解。我们可以直接通过方括号的形式来获取到所有我们指定了下标的元素，它们如下：

```
base_data[every_five] = 
[100 105 110 115 120 125 130 135 140 145 150 155 160 165 170 175 180 185
 190 195]


```

下标数组可以是一维的，当然也可以是多维的。假设我们要获取一个 2X2 的矩阵，这个矩阵的内容来自于目标数组中 1，2，10，20 这四个下标的元素，则可以这样写：

```
# array_index.py
a = np.array([(1,2), (10,20)])
print("a = \n{}\n".format(a))
print("base_data[a] = \n{}\n".format(base_data[a]))


```

这段代码输出如下：

```
a = 
[[ 1  2]
 [10 20]]
 
base_data[a] = 
[[101 102]
 [110 120]]


```

上面我们看到的是目标数组是一维的情况，下面我们把这个数组转换成一个 10X10 的二维数组。

```
# array_index.py
base_data2 = base_data.reshape(10, -1)
print("base_data2 = np.reshape(base_data, (10, -1)) = \n{}\n".format(base_data2))


```

`reshape`函数前面已经介绍过，大家应该能够想到它的结果：

```
base_data2 = np.reshape(base_data, (10, -1)) = 
[[100 101 102 103 104 105 106 107 108 109]
 [110 111 112 113 114 115 116 117 118 119]
 [120 121 122 123 124 125 126 127 128 129]
 [130 131 132 133 134 135 136 137 138 139]
 [140 141 142 143 144 145 146 147 148 149]
 [150 151 152 153 154 155 156 157 158 159]
 [160 161 162 163 164 165 166 167 168 169]
 [170 171 172 173 174 175 176 177 178 179]
 [180 181 182 183 184 185 186 187 188 189]
 [190 191 192 193 194 195 196 197 198 199]]


```

对于二维数组来说：

*   假设我们只指定了一个下标，则访问的结果仍然是一个数组。
*   假设我们指定了两个下标，则访问得到的是其中的元素
*   我们也可以通过”-1”来指定 “最后一个” 的元素

```
# array_index.py
print("base_data2[2] = \n{}\n".format(base_data2[2]))
print("base_data2[2, 3] = \n{}\n".format(base_data2[2, 3]))
print("base_data2[-1, -1] = \n{}\n".format(base_data2[-1, -1]))


```

这段代码输出如下。

对于更高维的数组，原理是一样的，读者可以自行推理。

```
base_data2[2] = 
[120 121 122 123 124 125 126 127 128 129]

base_data2[2, 3] = 
123

base_data2[-1, -1] = 
199


```

除此之外，我们还可以通过”:“的形式来指定范围，例如：`2:5` 这样。只写”:“则表示全部范围。

请看下面这段代码：

```
# array_index.py
print("base_data2[2, :]] = \n{}\n".format(base_data2[2, :]))
print("base_data2[:, 3]] = \n{}\n".format(base_data2[:, 3]))
print("base_data2[2:5, 2:4]] = \n{}\n".format(base_data2[2:5, 2:4]))


```

它的含义是：

1.  获取下标为 2 的行的所有元素
2.  获取下标为 3 的列的所有元素
3.  获取下标为 [2,5) 行，下标为 [2,4) 列的所有元素。请读者仔细观察一下下面的输出结果：

```
base_data2[2, :]] = 
[120 121 122 123 124 125 126 127 128 129]

base_data2[:, 3]] = 
[103 113 123 133 143 153 163 173 183 193]

base_data2[2:5, 2:4]] = 
[[122 123]
 [132 133]
 [142 143]]


```

NumPy 中自然也少不了大量的数学运算函数，下面是一些例子，更多的函数请参见这里 [NumPy manual contents](https://docs.scipy.org/doc/numpy/contents.html)：

```
# operation.py

import numpy as np

base_data = (np.random.random((5, 5)) - 0.5) * 100
print("base_data = \n{}\n".format(base_data))

print("np.amin(base_data) = {}".format(np.amin(base_data)))
print("np.amax(base_data) = {}".format(np.amax(base_data)))
print("np.average(base_data) = {}".format(np.average(base_data)))
print("np.sum(base_data) = {}".format(np.sum(base_data)))
print("np.sin(base_data) = \n{}".format(np.sin(base_data)))


```

这段代码输出如下：

```
base_data = 
[[ -9.63895991   6.9292461   -2.35654712 -48.45969283  13.56031937]
 [-39.75875796 -43.21031705 -49.27708561   6.80357128  33.71975059]
 [ 36.32228175  30.92546582 -41.63728955  28.68799187   6.44818484]
 [  7.71568596  43.24884701 -14.90716555  -9.24092252   3.69738718]
 [-31.90994273  34.06067289  18.47830413 -16.02495202 -44.84625246]]

np.amin(base_data) = -49.277085606595726
np.amax(base_data) = 43.24884701268845
np.average(base_data) = -3.22680706079886
np.sum(base_data) = -80.6701765199715
np.sin(base_data) = 
[[ 0.21254814  0.60204578 -0.70685739  0.9725159   0.8381861 ]
 [-0.88287359  0.69755541  0.83514527  0.49721505  0.74315189]
 [-0.98124746 -0.47103234  0.7149727  -0.40196147  0.16425187]
 [ 0.99045239 -0.66943662 -0.71791164 -0.18282139 -0.5276184 ]
 [-0.4741657   0.47665553 -0.36278223  0.31170676 -0.76041722]]


```

接下来我们看一下以矩阵的方式使用 NumPy。

首先，我们创建一个 5X5 的随机数整数矩阵。有两种方式可以获得矩阵的转置：通过`.T`或者`transpose`函数。另外， 通过`dot`函数可以进行矩阵的乘法，示例代码如下：

```
# matrix.py

import numpy as np

base_data = np.floor((np.random.random((5, 5)) - 0.5) * 100)
print("base_data = \n{}\n".format(base_data))

print("base_data.T = \n{}\n".format(base_data.T))
print("base_data.transpose() = \n{}\n".format(base_data.transpose()))

matrix_one = np.ones((5, 5))
print("matrix_one = \n{}\n".format(matrix_one))

minus_one = np.dot(matrix_one, -1)
print("minus_one = \n{}\n".format(minus_one))

print("np.dot(base_data, minus_one) = \n{}\n".format(
    np.dot(base_data, minus_one)))


```

这段代码输出如下：

```
base_data = 
[[-49.  -5.  11. -13. -41.]
 [ -6. -33. -33. -47.  -4.]
 [-38.  26.  28. -18.  18.]
 [ -3. -19. -15. -39.  45.]
 [-43.   6.  18. -15. -21.]]

base_data.T = 
[[-49.  -6. -38.  -3. -43.]
 [ -5. -33.  26. -19.   6.]
 [ 11. -33.  28. -15.  18.]
 [-13. -47. -18. -39. -15.]
 [-41.  -4.  18.  45. -21.]]

base_data.transpose() = 
[[-49.  -6. -38.  -3. -43.]
 [ -5. -33.  26. -19.   6.]
 [ 11. -33.  28. -15.  18.]
 [-13. -47. -18. -39. -15.]
 [-41.  -4.  18.  45. -21.]]

matrix_one = 
[[ 1.  1.  1.  1.  1.]
 [ 1.  1.  1.  1.  1.]
 [ 1.  1.  1.  1.  1.]
 [ 1.  1.  1.  1.  1.]
 [ 1.  1.  1.  1.  1.]]

minus_one = 
[[-1. -1. -1. -1. -1.]
 [-1. -1. -1. -1. -1.]
 [-1. -1. -1. -1. -1.]
 [-1. -1. -1. -1. -1.]
 [-1. -1. -1. -1. -1.]]

np.dot(base_data, minus_one) = 
[[  97.   97.   97.   97.   97.]
 [ 123.  123.  123.  123.  123.]
 [ -16.  -16.  -16.  -16.  -16.]
 [  31.   31.   31.   31.   31.]
 [  55.   55.   55.   55.   55.]]


```

本文的最后，我们来看一下随机数的使用。

随机数是我们在编程过程中非常频繁用到的一个功能。例如：生成演示数据，或者将已有的数据顺序随机打乱以便分割出建模数据和验证数据。

[numpy.random](https://docs.scipy.org/doc/numpy/reference/routines.random.html) 包中包含了很多中随机数的算法。下面我们列举四种最常见的用法：

```
# rand.py

import numpy as np

print("random: {}\n".format(np.random.random(20)));

print("rand: {}\n".format(np.random.rand(3, 4)));

print("randint: {}\n".format(np.random.randint(0, 100, 20)));

print("permutation: {}\n".format(np.random.permutation(np.arange(20))));


```

在四种用法分别是：

1.  生成 20 个随机数，它们每一个都是`[0.0, 1.0)`之间
2.  根据指定的`shape`生成随机数
3.  生成指定范围内（`[0, 100)`）的指定数量（20）的随机整数
4.  对已有的数据（`[0, 1, 2, ..., 19]`）的顺序随机打乱顺序

这段代码的输出如下所示：

```
random: [0.62956026 0.56816277 0.30903156 0.50427765 0.92117724 0.43044905
 0.54591323 0.47286235 0.93241333 0.32636472 0.14692983 0.02163887
 0.85014782 0.20164791 0.76556972 0.15137427 0.14626625 0.60972522
 0.2995841  0.27569573]

rand: [[0.38629927 0.43779617 0.96276889 0.80018417]
 [0.67656892 0.97189483 0.13323458 0.90663724]
 [0.99440473 0.85197677 0.9420241  0.79598706]]

randint: [74 65 51 34 22 69 81 36 73 35 98 26 41 84  0 93 41  6 51 55]

permutation: [15  3  8 18 14 19 16  1  0  4 10 17  5  2  6 12  9 11 13  7]


```

*   [NumPy Quickstart tutorial](https://docs.scipy.org/doc/numpy-dev/user/quickstart.html)
*   [NumPy User Guide](https://docs.scipy.org/doc/numpy-dev/user/index.html)
*   [NumPy Reference](https://docs.scipy.org/doc/numpy-dev/reference/index.html#reference)
*   [NumPy Array Challenge](https://www.dataquest.io/blog/numpy-tutorial-python/)
*   [Python Numpy Tutorial](http://cs231n.github.io/python-numpy-tutorial/)
*   [Numpy Tutorial](https://www.python-course.eu/numpy.php)
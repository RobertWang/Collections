> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1625565234&ver=3174&signature=gqfWiYVA5z-udCJlqGuWV8vUbtinT2uSnvF84SB0XZxrTxj42tzBfLmyKHfDbdc4xKbSL2b2K8YNHNfhkLvN380k*7KLjGRtzQlZtlgE43pz4jCE5G*7f7tM-hWkOc2W&new=1)

> 来，检测检测你的 NumPy 掌握得如何

Numpy 仍然是 Python 做数据分析所必须要掌握的基础库之一，以下题是 github 上的开源项目，主要为了检**测你的 Numpy 能力**，同时对你的学习作为一个补充。

> **来源：**https://github.com/rougier/numpy-100

### 1. 导入 numpy 库并取别名为 np (★☆☆)

```
import numpy as np

```

### 2. 打印输出 numpy 的版本和配置信息 (★☆☆)

```
print(np.__version__)
print(np.show_config())

```

### 3. 创建一个长度为 10 的空向量 (★☆☆)

```
Z = np.zeros(10)
print(Z)

```

### 4. 如何找到任何一个数组的内存大小？(★☆☆)

```
Z = np.zeros((10,10))
print("%d bytes" % (Z.size * Z.itemsize))

```

### 5. 如何从命令行得到 numpy 中 add 函数的说明文档? (★☆☆)

```
import numpy as np
np.info(numpy.add)

```

### 6. 创建一个长度为 10 并且除了第五个值为 1 的空向量 (★☆☆)

```
Z = np.zeros(10)
Z[4] = 1
print(Z)

```

### 7. 创建一个值域范围从 10 到 49 的向量 (★☆☆)

```
Z = np.arange(10,50)
print(Z)

```

### 8.  反转一个向量 (第一个元素变为最后一个) (★☆☆)

```
Z = np.arange(50)
Z = Z[::-1]
print(Z)

```

### 9. 创建一个 3x3 并且值从 0 到 8 的矩阵 (★☆☆)

```
Z = np.arange(9).reshape(3,3)
print(Z)

```

### 10. 找到数组 [1,2,0,0,4,0] 中非 0 元素的位置索引 (★☆☆)

```
nz = np.nonzero([1,2,0,0,4,0])
print(nz)

```

### 11. 创建一个 3x3 的单位矩阵 (★☆☆)

```
Z = np.eye(3)
print(Z)

```

### 12. 创建一个 3x3x3 的随机数组 (★☆☆)

```
Z = np.random.random((3,3,3))
print(Z)

```

### 13. 创建一个 10x10 的随机数组并找到它的最大值和最小值 (★☆☆)

```
Z = np.random.random((10,10))
Zmin, Zmax = Z.min(), Z.max()
print(Zmin, Zmax)

```

### 14. 创建一个长度为 30 的随机向量并找到它的平均值 (★☆☆)

```
Z = np.random.random(30)
m = Z.mean()
print(m)

```

### 15. 创建一个二维数组，其中边界值为 1，其余值为 0 (★☆☆)

```
Z = np.ones((10,10))
Z[1:-1,1:-1] = 0
print(Z)

```

### 16. 对于一个存在在数组，如何添加一个用 0 填充的边界? (★☆☆)

```
Z = np.ones((5,5))
Z = np.pad(Z, pad_width=1, mode='constant', constant_values=0)
print(Z)

```

### 17. 下面表达式运行的结果是什么？(★☆☆)

```
0 * np.nan                        nan
np.nan == np.nan                  False
np.inf > np.nan                   False
np.nan - np.nan                   nan
0.3 == 3 * 0.1                    False

```

### 18. 创建一个 5x5 的矩阵，并设置值 1,2,3,4 落在其对角线下方位置 (★☆☆)

```
Z = np.diag(1+np.arange(4),k=-1)
print(Z)

```

### 19. 创建一个 8x8 的矩阵，并且设置成棋盘样式 (★☆☆)

```
Z = np.zeros((8,8),dtype=int)
Z[1::2,::2] = 1
Z[::2,1::2] = 1
print(Z)

```

### 20. 考虑一个 (6,7,8) 形状的数组，其第 100 个元素的索引 (x,y,z) 是什么?

```
print(np.unravel_index(100,(6,7,8)))

```

### 21. 用 tile 函数去创建一个 8x8 的棋盘样式矩阵 (★☆☆)

```
Z = np.tile( np.array([[0,1],[1,0]]), (4,4))
print(Z)

```

### 22. 对一个 5x5 的随机矩阵做归一化 (★☆☆)

```
Z = np.random.random((5,5))
Zmax, Zmin = Z.max(), Z.min()
Z = (Z - Zmin)/(Zmax - Zmin)
print(Z)

```

### 23. 创建一个将颜色描述为 (RGBA) 四个无符号字节的自定义 dtype？(★☆☆)

```
color = np.dtype([("r", np.ubyte, 1),
                  ("g", np.ubyte, 1),
                  ("b", np.ubyte, 1),
                  ("a", np.ubyte, 1)])
color

```

### 24. 一个 5x3 的矩阵与一个 3x2 的矩阵相乘，实矩阵乘积是什么？(★☆☆)

```
Z = np.dot(np.ones((5,3)), np.ones((3,2)))
print(Z)

```

### 25. 给定一个一维数组，对其在 3 到 8 之间的所有元素取反 (★☆☆)

```
Z = np.arange(11)
Z[(3 < Z) & (Z <= 8)] *= -1
print(Z)

```

### 26. 下面脚本运行后的结果是什么? (★☆☆)

```
print(sum(range(5),-1))                 
from numpy import *                     
print(sum(range(5),-1))                 

```

### 27. 考虑一个整数向量 Z, 下列表达合法的是哪个? (★☆☆)

```
Z**Z                        
2 << Z >> 2                 
Z <- Z                      
1j*Z                        
Z/1/1                       
Z<Z>Z                       

```

### 28. 下面表达式的结果分别是什么？(★☆☆)

```
np.array(0) / np.array(0)                           #nan
np.array(0) 
np.array([np.nan]).astype(int).astype(float)        #-2.14748365e+09

```

### 29. 如何从零位开始舍入浮点数组？(★☆☆)

```
Z = np.random.uniform(-10,+10,10)
print (np.copysign(np.ceil(np.abs(Z)), Z))

```

### 30. 如何找出两个数组公共的元素? (★☆☆)

```
Z1 = np.random.randint(0, 10, 10)
Z2 = np.random.randint(0, 10, 10)
print (np.intersect1d(Z1, Z2))

```

### 31. 如何忽略所有的 numpy 警告 (尽管不建议这么做)? (★☆☆)

```
defaults = np.seterr(all="ignore")
Z = np.ones(1) / 0
_ = np.seterr(**defaults)
with np.errstate(divide='ignore'):
    Z = np.ones(1) / 0

```

### 32. 下面的表达式是否为真? (★☆☆)

```
np.sqrt(-1) == np.emath.sqrt(-1)     #False

```

### 33. 如何获得昨天，今天和明天的日期? (★☆☆)

```
yesterday = np.datetime64('today', 'D') - np.timedelta64(1, 'D')
today = np.datetime64('today', 'D')
tomorrow = np.datetime64('today', 'D') + np.timedelta64(1, 'D')

```

### 34. 怎么获得所有与 2016 年 7 月的所有日期? (★★☆)

```
Z = np.arange('2016-07', '2016-08', dtype='datetime64[D]')
print (Z)

```

### 35. 如何计算 ((A+B)*(-A/2)) (不使用中间变量)? (★★☆)

```
A = np.ones(3) * 1
B = np.ones(3) * 1
C = np.ones(3) * 1
np.add(A, B, out=B)
np.divide(A, 2, out=A)
np.negative(A, out=A)
np.multiply(A, B, out=A)

```

### 36. 用 5 种不同的方法提取随机数组中的整数部分 (★★☆)

```
Z = np.random.uniform(0, 10, 10)
print(Z - Z % 1)
print(np.floor(Z))
print(np.cell(Z)-1)
print(Z.astype(int))
print(np.trunc(Z))

```

### 37. 创建一个 5x5 的矩阵且每一行的值范围为从 0 到 4 (★★☆)

```
Z = np.zeros((5, 5))
Z += np.arange(5)
print(Z)

```

### 38. 如何用一个生成 10 个整数的函数来构建数组 (★☆☆)

```
def generate():
    for x in range(10):
      yield x
Z = np.fromiter(generate(), dtype=float, count=-1)
print(Z)

```

### 39. 创建一个大小为 10 的向量， 值域为 0 到 1，不包括 0 和 1 (★★☆)

```
Z = np.linspace(0, 1, 12, endpoint=True)[1: -1]
print (Z)

```

### 40. 创建一个大小为 10 的随机向量，并把它排序 (★★☆)

```
Z = np.random.random(10)
Z.sort()
print (Z)

```

### 41.  对一个小数组进行求和有没有办法比 np.sum 更快? (★★☆)

```
Z = np.arange(10)
np.add.reduce(Z)

```

### 42. 如何判断两和随机数组相等 (★★☆)

```
A = np.random.randint(0, 2, 5)
B = np.random.randint(0, 2, 5)
equal = np.allclose(A,B)
print(equal)
equal = np.array_equal(A,B)
print(equal)

```

### 43. 把数组变为只读 (★★☆)

```
Z = np.zeros(5)
Z.flags.writeable = False
Z[0] = 1

```

### 44. 将一个 10x2 的笛卡尔坐标矩阵转换为极坐标 (★★☆)

```
Z = np.random.random((10, 2))
X, Y = Z[:, 0], Z[:, 1]
R = np.sqrt(X**2 + Y**2)
T = np.arctan2(Y, X)
print (R)
print (T)

```

### 45. 创建一个大小为 10 的随机向量并且将该向量中最大的值替换为 0(★★☆)

```
Z = np.random.random(10)
Z[Z.argmax()] = 0
print (Z)

```

### 46. 创建一个结构化数组，其中 x 和 y 坐标覆盖 [0, 1]x[1, 0] 区域 (★★☆)

```
Z = np.zeros((5, 5), [('x', float), ('y', float)])
Z['x'], Z['y'] = np.meshgrid(np.linspace(0, 1, 5), np.linspace(0, 1, 5))
print (Z)

```

### **47. 给定两个数组 X 和 Y，构造柯西 (Cauchy) 矩阵 C (****) (★★☆)**

```
X = np.arange(8)
Y = X + 0.5
C = 1.0 / np.subtract.outer(X, Y)
print (C)
print(np.linalg.det(C)) 

```

### 48. 打印每个 numpy 类型的最小和最大可表示值 (★★☆)

```
for dtype in [np.int8, np.int32, np.int64]:
   print(np.iinfo(dtype).min)
   print(np.iinfo(dtype).max)
for dtype in [np.float32, np.float64]:
   print(np.finfo(dtype).min)
   print(np.finfo(dtype).max)
   print(np.finfo(dtype).eps)

```

### 49. 如何打印数组中所有的值？(★★☆)

```
np.set_printoptions(threshold=np.nan)
Z = np.zeros((16,16))
print(Z)

```

### 50. 如何在数组中找到与给定标量接近的值? (★★☆)

```
Z = np.arange(100)
v = np.random.uniform(0, 100)
index = (np.abs(Z-v)).argmin()
print(Z[index])

```

### 51. 创建表示位置 (x, y) 和颜色 (r, g, b, a) 的结构化数组 (★★☆)

```
Z = np.zeros(10, [('position', [('x', float, 1), 
                                ('y', float, 1)]),
                  ('color',    [('r', float, 1), 
                                ('g', float, 1), 
                                ('b', float, 1)])])
print (Z)

```

### 52. 思考形状为 (100, 2) 的随机向量，求出点与点之间的距离 (★★☆)

```
Z = np.random.random((100, 2))
X, Y = np.atleast_2d(Z[:, 0], Z[:, 1])
D = np.sqrt((X-X.T)**2 + (Y-Y.T)**2)
print (D)

```

```
import scipy.spatial
Z = np.random.random((100,2))
D = scipy.spatial.distance.cdist(Z,Z)
print(D)

```

### 53. 如何将类型为 float(32 位) 的数组类型转换位 integer(32 位)? (★★☆)

```
Z = np.arange(10, dtype=np.int32)
Z = Z.astype(np.float32, copy=False)
print(Z)

```

### 54. 如何读取下面的文件? (★★☆)

```
'''1, 2, 3, 4, 5
6,  ,  , 7, 8
 ,  , 9,10,11'''
Z = np.genfromtxt("example.txt", delimiter=",")  
print(Z)

```

### 55. numpy 数组枚举 (enumerate) 的等价操作? (★★☆)

```
Z = np.arange(9).reshape(3,3)
for index, value in np.ndenumerate(Z):
    print(index, value)
for index in np.ndindex(Z.shape):
    print(index, Z[index])

```

### 56. 构造一个二维高斯矩阵 (★★☆)

```
X, Y = np.meshgrid(np.linspace(-1, 1, 10), np.linspace(-1, 1, 10))
D = np.sqrt(X**2 + Y**2)
sigma, mu = 1.0, 0.0
G = np.exp(-( (D-mu)**2 / (2.0*sigma**2) ))
print (G)

```

### 57. 如何在二维数组的随机位置放置 p 个元素? (★★☆)

```
n = 10
p = 3
Z = np.zeros((n,n))
np.put(Z, np.random.choice(range(n*n), p, replace=False),1)
print(Z)

```

### 58. 减去矩阵每一行的平均值 (★★☆)

```
X = np.random.rand(5, 10)
Y = X - X.mean(axis=1, keepdims=True)
print(Y)

```

### 59. 如何对数组通过第 n 列进行排序? (★★☆)

```
Z = np.random.randint(0,10,(3,3))
print(Z)
print(Z[ Z[:,1].argsort() ])

```

### 60. 如何判断一个给定的二维数组存在空列? (★★☆)

```
Z = np.random.randint(0,3,(3,10))
print((~Z.any(axis=0)).any())

```

### 61. 从数组中找出与给定值最接近的值 (★★☆)

```
Z = np.random.uniform(0,1,10)
z = 0.5
m = Z.flat[np.abs(Z - z).argmin()]
print(m)

```

### 62. 思考形状为 (1, 3) 和(3, 1)的两个数组形状，如何使用迭代器计算它们的和? (★★☆)

```
A = np.arange(3).reshape(3, 1)
B = np.arange(3).reshape(1, 3)
it = np.nditer([A, B, None])
for x, y, z in it:
    z[...] = x + y
print (it.operands[2])

```

### 63. 创建一个具有 name 属性的数组类 (★★☆)

```
class NameArray(np.ndarray):
    def __new__(cls, array, ):
        obj = np.asarray(array).view(cls)
        obj.name = name
        return obj
    def __array_finalize__(self, obj):
        if obj is None: return
        self.info = getattr(obj, 'name', "no name")
Z = NameArray(np.arange(10), "range_10")
print (Z.name)

```

### 64. 给定一个向量，如何让在第二个向量索引的每个元素加 1(注意重复索引)? (★★★)

```
Z = np.ones(10)
I = np.random.randint(0,len(Z),20)
Z += np.bincount(I, minlength=len(Z))
print(Z)
np.add.at(Z, I, 1)
print(Z)

```

### 65. 如何根据索引列表 I 将向量 X 的元素累加到数组 F? (★★★)

```
X = [1,2,3,4,5,6]
I = [1,3,9,3,4,1]
F = np.bincount(I,X)
print(F)

```

### 66. 思考 (dtype = ubyte) 的(w, h, 3)图像，计算唯一颜色的值(★★★)

```
w,h = 16,16
I = np.random.randint(0,2,(h,w,3)).astype(np.ubyte)
F = I[...,0]*256*256 + I[...,1]*256 +I[...,2]
n = len(np.unique(F))
print(np.unique(I))

```

### 67. 思考如何求一个四维数组最后两个轴的数据和 (★★★)

```
A = np.random.randint(0,10,(3,4,3,4))
sum = A.sum(axis=(-2,-1))
print(sum)
sum = A.reshape(A.shape[:-2] + (-1,)).sum(axis=-1)
print(sum)

```

### 68. 考虑一维向量 D，如何使用相同大小的向量 S 来计算 D 的子集的均值，其描述子集索引？(★★★)

```
D = np.random.uniform(0,1,100)
S = np.random.randint(0,10,100)
D_sums = np.bincount(S, weights=D)
D_counts = np.bincount(S)
D_means = D_sums / D_counts
print(D_means)
import pandas as pd
print(pd.Series(D).groupby(S).mean())

```

### 69. 如何获得点积的对角线？(★★★)

```
A = np.random.uniform(0,1,(5,5))
B = np.random.uniform(0,1,(5,5))
np.diag(np.dot(A, B))
np.sum(A * B.T, axis=1)
np.einsum("ij,ji->i", A, B)

```

### 70. 考虑向量 [1,2,3,4,5]，如何建立一个新的向量，在每个值之间交错有 3 个连续的零？(★★★)

```
Z = np.array([1,2,3,4,5])
nz = 3
Z0 = np.zeros(len(Z) + (len(Z)-1)*(nz))
Z0[::nz+1] = Z
print(Z0)

```

### 71. 考虑一个维度 (5,5,3) 的数组，如何将其与一个 (5,5) 的数组相乘？(★★★)

```
A = np.ones((5,5,3))
B = 2*np.ones((5,5))
print(A * B[:,:,None])

```

### 72. 如何对一个数组中任意两行做交换? (★★★)

```
A = np.arange(25).reshape(5,5)
A[[0,1]] = A[[1,0]]
print(A)

```

### 73. 思考描述 10 个三角形（共享顶点）的一组 10 个三元组，找到组成所有三角形的唯一线段集 (★★★)

```
faces = np.random.randint(0,100,(10,3))
F = np.roll(faces.repeat(2,axis=1),-1,axis=1)
F = F.reshape(len(F)*3,2)
F = np.sort(F,axis=1)
G = F.view( dtype=[('p0',F.dtype),('p1',F.dtype)] )
G = np.unique(G)
print(G)

```

### 74. 给定一个二进制的数组 C，如何生成一个数组 A 满足 np.bincount(A)==C? (★★★)

```
C = np.bincount([1,1,2,3,4,4,6])
A = np.repeat(np.arange(len(C)), C)
print(A)

```

### 75. 如何通过滑动窗口计算一个数组的平均数? (★★★)

```
def moving_average(a, n=3) :
    ret = np.cumsum(a, dtype=float)
    ret[n:] = ret[n:] - ret[:-n]
    return ret[n - 1:] / n
Z = np.arange(20)
print(moving_average(Z, n=3))

```

### 76. 思考以为数组 Z，构建一个二维数组，其第一行是 (Z[0],Z[1],Z[2])， 然后每一行移动一位，最后一行为 (Z[-3],Z[-2],Z[-1]) (★★★)

```
from numpy.lib import stride_tricks
def rolling(a, window):
    shape = (a.size - window + 1, window)
    strides = (a.itemsize, a.itemsize)
    return stride_tricks.as_strided(a, shape=shape, strides=strides)
Z = rolling(np.arange(10), 3)
print(Z)

```

### 77. 如何对布尔值取反，或改变浮点数的符号 (sign)? (★★★)

```
Z = np.random.randint(0,2,100)
np.logical_not(Z, out=Z)
Z = np.random.uniform(-1.0,1.0,100)
np.negative(Z, out=Z)

```

### 78. 思考两组点集 P0 和 P1 去描述一组线 (二维) 和一个点 p, 如何计算点 p 到每一条线 i (P0[i],P1[i])的距离？(★★★)

```
def distance(P0, P1, p):
    T = P1 - P0
    L = (T**2).sum(axis=1)
    U = -((P0[:,0]-p[...,0])*T[:,0] + (P0[:,1]-p[...,1])*T[:,1]) / L
    U = U.reshape(len(U),1)
    D = P0 + U*T - p
    return np.sqrt((D**2).sum(axis=1))
P0 = np.random.uniform(-10,10,(10,2))
P1 = np.random.uniform(-10,10,(10,2))
p  = np.random.uniform(-10,10,( 1,2))
print (distance(P0, P1, p))

```

### 79. 考虑两组点集 P0 和 P1 去描述一组线 (二维) 和一组点集 P，如何计算每一个点 j(P[j]) 到每一条线 i (P0[i],P1[i])的距离? (★★★)

```
P0 = np.random.uniform(-10, 10, (10,2))
P1 = np.random.uniform(-10,10,(10,2))
p = np.random.uniform(-10, 10, (10,2))
print(np.array([distance(P0,P1,p_i) for p_i in p]))

```

### 80. 思考一个任意的数组，编写一个函数，该函数提取一个具有固定形状的子部分，并以一个给定的元素为中心 (在该部分填充值) (★★★)

```
Z = np.random.randint(0,10,(10,10))
shape = (5,5)
fill  = 0
position = (1,1)
R = np.ones(shape, dtype=Z.dtype)*fill
P  = np.array(list(position)).astype(int)
Rs = np.array(list(R.shape)).astype(int)
Zs = np.array(list(Z.shape)).astype(int)R_start = np.zeros((len(shape),)).astype(int)
R_stop  = np.array(list(shape)).astype(int)
Z_start = (P-Rs//2)
Z_stop  = (P+Rs//2)+Rs%2
R_start = (R_start - np.minimum(Z_start,0)).tolist()
Z_start = (np.maximum(Z_start,0)).tolist()
R_stop = np.maximum(R_start, (R_stop - np.maximum(Z_stop-Zs,0))).tolist()
Z_stop = (np.minimum(Z_stop,Zs)).tolist()
r = [slice(start,stop) for start,stop in zip(R_start,R_stop)]
z = [slice(start,stop) for start,stop in zip(Z_start,Z_stop)]
R[r] = Z[z]
print(Z)
print(R)

```

### 81. 考虑一个数组 Z = [1,2,3,4,5,6,7,8,9,10,11,12,13,14], 如何生成一个数组 R = [[1,2,3,4], [2,3,4,5], [3,4,5,6], ...,[11,12,13,14]]? (★★★)

```
Z = np.arange(1,15,dtype=np.uint32)
R = stride_tricks.as_strided(Z,(11,4),(4,4))
print(R)

```

### 82. 计算矩阵的秩 (★★★)

```
Z = np.random.uniform(0,1,(10,10))
U, S, V = np.linalg.svd(Z) 
rank = np.sum(S > 1e-10)
print(rank)

```

### 83. 如何找出数组中出现频率最高的值?(★★★)

```
Z = np.random.randint(0,10,50)
print(np.bincount(Z).argmax())

```

### 84. 从一个 10x10 的矩阵中提取出连续的 3x3 区块 (★★★)

```
Z = np.random.randint(0,5,(10,10))
n = 3
i = 1 + (Z.shape[0]-3)
j = 1 + (Z.shape[1]-3)
C = stride_tricks.as_strided(Z, shape=(i, j, n, n), strides=Z.strides + Z.strides)
print(C)

```

### 85. 创建一个满足 Z[i,j] == Z[j,i] 的二维数组子类 (★★★)

```
class Symetric(np.ndarray):
    def __setitem__(self, index, value):
        i,j = index
        super(Symetric, self).__setitem__((i,j), value)
        super(Symetric, self).__setitem__((j,i), value)
def symetric(Z):
    return np.asarray(Z + Z.T - np.diag(Z.diagonal())).view(Symetric)
S = symetric(np.random.randint(0,10,(5,5)))
S[2,3] = 42
print(S)

```

### 86. 考虑 p 个 nxn 矩阵和一组形状为 (n,1) 的向量，如何直接计算 p 个矩阵的乘积(n,1)? (★★★)

```
p, n = 10, 20
M = np.ones((p,n,n))
V = np.ones((p,n,1))
S = np.tensordot(M, V, axes=[[0, 2], [0, 1]])
print(S)

```

### 87. 对于一个 16x16 的数组，如何得到一个区域的和 (区域大小为 4x4)? (★★★)

```
Z = np.ones((16,16))
k = 4
S = np.add.reduceat(np.add.reduceat(Z, np.arange(0, Z.shape[0], k), axis=0), np.arange(0, Z.shape[1], k), axis=1)
print(S)

```

### 88. 如何利用 numpy 数组实现 Game of Life? (★★★)

```
def iterate(Z):
    N = (Z[0:-2,0:-2] + Z[0:-2,1:-1] + Z[0:-2,2:] +
         Z[1:-1,0:-2]                + Z[1:-1,2:] +
         Z[2:  ,0:-2] + Z[2:  ,1:-1] + Z[2:  ,2:])
    birth = (N==3) & (Z[1:-1,1:-1]==0)
    survive = ((N==2) | (N==3)) & (Z[1:-1,1:-1]==1)
    Z[...] = 0
    Z[1:-1,1:-1][birth | survive] = 1
    return Z
Z = np.random.randint(0,2,(50,50))
for i in range(100): Z = iterate(Z)
print(Z)

```

### 89. 如何找到一个数组的第 n 个最大值? (★★★)

```
Z = np.arange(10000)
np.random.shuffle(Z)
n = 5
print (Z[np.argsort(Z)[-n:]])
print (Z[np.argpartition(-Z,n)[:n]])

```

### 90. 给定任意个数向量，创建笛卡尔积 (每一个元素的每一种组合) (★★★)

```
def cartesian(arrays):
    arrays = [np.asarray(a) for a in arrays]
    shape = (len(x) for x in arrays)
    ix = np.indices(shape, dtype=int)
    ix = ix.reshape(len(arrays), -1).T
    for n, arr in enumerate(arrays):
        ix[:, n] = arrays[n][ix[:, n]]
    return ix
print (cartesian(([1, 2, 3], [4, 5], [6, 7])))

```

### 91. 如何从一个常规数组中创建记录数组 (record array)? (★★★)

```
Z = np.array([("Hello", 2.5, 3),
              ("World", 3.6, 2)])
R = np.core.records.fromarrays(Z.T, 
                               names='col1, col2, col3',
                               formats = 'S8, f8, i8')
print(R)

```

### 92. 思考一个大向量 Z, 用三种不同的方法计算它的立方 (★★★)

```
x = np.random.rand(5e7)
%timeit np.power(x,3)
%timeit x*x*x
%timeit np.einsum('i,i,i->i',x,x,x)

```

### 93. 考虑两个形状分别为 (8,3) 和(2,2) 的数组 A 和 B. 如何在数组 A 中找到满足包含 B 中元素的行？(不考虑 B 中每行元素顺序)？(★★★)

```
A = np.random.randint(0,5,(8,3))
B = np.random.randint(0,5,(2,2))
C = (A[..., np.newaxis, np.newaxis] == B)
rows = np.where(C.any((3,1)).all(1))[0]
print(rows)

```

### 94. 思考一个 10x3 的矩阵，如何分解出有不全相同值的行 (如 [2,2,3]) (★★★)

```
Z = np.random.randint(0,5,(10,3))
print(Z)
E = np.all(Z[:,1:] == Z[:,:-1], axis=1)
U = Z[~E]
print(U)
U = Z[Z.max(axis=1) != Z.min(axis=1),:]
print(U)

```

### 95. 将一个整数向量转换为二进制矩阵 (★★★)

```
I = np.array([0, 1, 2, 3, 15, 16, 32, 64, 128])
B = ((I.reshape(-1,1) & (2**np.arange(8))) != 0).astype(int)
print(B[:,::-1])
I = np.array([0, 1, 2, 3, 15, 16, 32, 64, 128], dtype=np.uint8)
print(np.unpackbits(I[:, np.newaxis], axis=1))

```

### 96. 给定一个二维数组，如何提取出唯一的行?(★★★)

```
Z = np.random.randint(0,2,(6,3))
T = np.ascontiguousarray(Z).view(np.dtype((np.void, Z.dtype.itemsize * Z.shape[1])))
_, idx = np.unique(T, return_index=True)
uZ = Z[idx]
print(uZ)

```

### 97. 考虑两个向量 A 和 B，写出用 einsum 等式对应的 inner, outer, sum, mul 函数 (★★★)

```
A = np.random.uniform(0,1,10)
B = np.random.uniform(0,1,10)
np.einsum('i->', A)       
np.einsum('i,i->i', A, B) 
np.einsum('i,i', A, B)    
np.einsum('i,j->ij', A, B)    

```

### 98. 考虑一个由两个向量描述的路径 (X,Y)，如何用等距样例(equidistant samples) 对其进行采样(sample)(★★★)?

```
phi = np.arange(0, 10*np.pi, 0.1)
a = 1
x = a*phi*np.cos(phi)
y = a*phi*np.sin(phi)
dr = (np.diff(x)**2 + np.diff(y)**2)**.5 
r = np.zeros_like(x)
r[1:] = np.cumsum(dr)                # integrate path
r_int = np.linspace(0, r.max(), 200) 
x_int = np.interp(r_int, r, x)       
y_int = np.interp(r_int, r, y)

```

### 99. 给定一个整数 n 和一个二维数组 X，从 X 中选择可以被解释为从多 n 度的多项分布式的行，即这些行只包含整数对 n 的和. (★★★)

```
X = np.asarray([[1.0, 0.0, 3.0, 8.0],
                [2.0, 0.0, 1.0, 1.0],
                [1.5, 2.5, 1.0, 0.0]])
n = 4
M = np.logical_and.reduce(np.mod(X, 1) == 0, axis=-1)
M &= (X.sum(axis=-1) == n)
print(X[M])

```

### 100. 对于一个一维数组 X，计算它 boostrapped 之后的 95% 置信区间的平均值. (★★★)

```
X = np.random.randn(100) 
N = 1000 
idx = np.random.randint(0, X.size, (N, X.size))
means = X[idx].mean(axis=1)
confint = np.percentile(means, [2.5, 97.5])
print(confint)

```

End
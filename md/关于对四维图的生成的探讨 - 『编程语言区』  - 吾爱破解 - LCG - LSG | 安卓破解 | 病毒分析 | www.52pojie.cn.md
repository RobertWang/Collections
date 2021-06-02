> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1451799&extra=page%3D1%26filter%3Dauthor%26orderby%3Ddateline) ![](https://avatar.52pojie.cn/data/avatar/000/82/06/70_avatar_middle.jpg) 笔墨纸砚  [Python] _纯文本查看_ _复制代码_

```
import numpy as np
from matplotlib import pyplot as plt
from mpl_toolkits.mplot3d import Axes3D
 
# 定义坐标轴
fig = plt.figure()
ax1 = plt.axes(projection='3d')
# ax = fig.add_subplot(111,projection='3d')  #这种方法也可以画多个子图
 
z = range(100)
x = range(100)
y = 5 * np.cos(z)
# zd = 13 * np.random.random(100)
# xd = 5 * np.sin(zd)
# yd = 5 * np.cos(zd)
 
cmp = plt.cm.get_cmap('rainbow')
#                                                                   ax1.scatter3D(xd,yd,zd, cmap='rainbow')  #绘制散点图
for i in range(len(x) - 1):
 
    plt.plot([x[i], x[i + 1]], [y[i], y[i + 1]], linewidth=5, color=cmp(x[i] / max(x)))
    #ax1.scatter3D(z, z, z, cmap='rainbow', color=cmp(x[i] / max(x)))  # 绘制散点图
```

但是在绘制 3D 图时 却无法让颜色多样  （四维 就  第四维就是颜色）
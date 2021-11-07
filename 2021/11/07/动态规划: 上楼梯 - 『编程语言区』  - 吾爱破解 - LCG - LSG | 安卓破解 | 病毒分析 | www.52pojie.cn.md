> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1539116&extra=page%3D1%26filter%3Dauthor%26orderby%3Ddateline) ![](https://avatar.52pojie.cn/data/avatar/001/73/82/20_avatar_middle.jpg)TES286  _ 本帖最后由 TES286 于 2021-11-6 11:43 编辑_  

有这样一个问题

> 有一个人要上楼梯, 每次可以上一个或两个楼梯, 求当有 x 级阶梯时有多少种上法

![](https://attach.52pojie.cn/forum/202111/06/011614vgzl287may81gfgg.png)

**1.png** _(11.19 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjM0NDM0MXxkYTNmZTE1NHwxNjM2MjgwMzM5fDEwNzQxNTd8MTUzOTExNg%3D%3D&nothumb=yes)

2021-11-6 01:16 上传

我们可以知道当 x 的值很小时的答案:

f(1) = 1

f(2) = 2

而后面的推到如下

![](https://attach.52pojie.cn/forum/202111/06/011617n5t33eyyyyhod56e.png)

**2.png** _(18.71 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjM0NDM0Mnw5ZDlmZDcxNHwxNjM2MjgwMzM5fDEwNzQxNTd8MTUzOTExNg%3D%3D&nothumb=yes)

2021-11-6 01:16 上传

也就是

f(x) = f(x-1) + f(x-2)

每次都会调用到自身, 这是一个递归

可以构建以下代码

```
def a(n):
  if n == 1:
    return 1
  elif n == 2:
    return 2
  else:
    return a(n-1) + a(n-2)
```

这个方法可以解决, 但较为耗时, 例如这里入参 35, 总共整了 2 秒多, 到了 40, 直接飙到 26 秒

![](https://attach.52pojie.cn/forum/202111/06/011619caemctlve4e4lref.png)

**3.png** _(6.45 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjM0NDM0M3w1OTM2YzE4ZHwxNjM2MjgwMzM5fDEwNzQxNTd8MTUzOTExNg%3D%3D&nothumb=yes)

2021-11-6 01:16 上传

![](https://attach.52pojie.cn/forum/202111/06/011621lyssskyshs6dsb0a.png)

**4.png** _(6.43 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjM0NDM0NHxlOTkxNTZlOHwxNjM2MjgwMzM5fDEwNzQxNTd8MTUzOTExNg%3D%3D&nothumb=yes)

2021-11-6 01:16 上传

那么它就要优化

我们以入参为 5 是看看

f(5) = f(4) + f(3) = f(3) + f(2) + f(2) + f(1) = f(2) + f(1) + f(2) + f(2) + f(1) = 3 _f(2) + 2_ f(1)

可见, 在入参为 5 时, f(2) 运行 3 次, f(1) 运行一次

那么, 在很高的入参下, 有很多次重复计算

那么这些重复的运算可以省掉吗?

可以, 代码如下

```
def b(n, *, l=dict()):
  # 因为dict是可变对象,在定义时初始化的一个dict()不会随着函数的运行结束而丢失
  if n == 1:
    return 1
  elif n == 2:
    return 2
  else:
    if n in l:
      return l[n]
    else:
      r = b(n-1) + b(n-2)
      l[n] = r
      return r
```

经过测试, 40 次运算时间在 0.003s 左右

![](https://attach.52pojie.cn/forum/202111/06/011623iy0uj00dj9nvnd3t.png)

**5.png** _(6.43 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjM0NDM0NXxlZTFjM2RhMXwxNjM2MjgwMzM5fDEwNzQxNTd8MTUzOTExNg%3D%3D&nothumb=yes)

2021-11-6 01:16 上传

经过优化后, 时间基本稳在 0.01s 之下

![](https://attach.52pojie.cn/forum/202111/06/011625yz8qbjl988612plk.png)

**6.png** _(37.9 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjM0NDM0Nnw1ZDE1OTMxNnwxNjM2MjgwMzM5fDEwNzQxNTd8MTUzOTExNg%3D%3D&nothumb=yes)

2021-11-6 01:16 上传

![](https://attach.52pojie.cn/forum/202111/06/011627ri06qd55id02kqho.png)

**7.png** _(15.47 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjM0NDM0N3w4OGQyMDYyZnwxNjM2MjgwMzM5fDEwNzQxNTd8MTUzOTExNg%3D%3D&nothumb=yes)

2021-11-6 01:16 上传

代码:

```
import time
def a(n):
  if n == 1:
    return 1
  elif n == 2:
    return 2
  else:
    return a(n-1) + a(n-2)

def b(n, *, l=dict()):
  if n == 1:
    return 1
  elif n == 2:
    return 2
  else:
    if n in l:
      return l[n]
    else:
      r = b(n-1) + b(n-2)
      l[n] = r
      return r

if __name__ == '__main__':
    #func = a
    func = b
    #x = 300
    x = int(input('times:'))
    t1=time.time()
    print(func(x))
    t2=time.time()
    print(t2-t1)
```

![](https://avatar.52pojie.cn/data/avatar/000/47/93/36_avatar_middle.jpg)平淡最真  能打败数学的只有数学了，![](https://static.52pojie.cn/static/image/smiley/default/victory.gif)  
[Python] _纯文本查看_ _复制代码_

```
def c(x,y,z,n):
    return (y**(n+1)-z**(n+1))/x
if __name__ == '__main__':
    x=5**0.5
    y=0.5+x/2
    z=0.5-x/2
    aa= int(input('times:'))
    t1=time.time()
    print(int(c(x,y,z,aa)))
    t2=time.time()
    print(t2-t1)
```

![](https://avatar.52pojie.cn/data/avatar/000/47/93/36_avatar_middle.jpg)平淡最真  时间是够快，但是结果可能不准，毕竟纯浮点数运算![](https://avatar.52pojie.cn/data/avatar/000/37/34/00_avatar_middle.jpg)魔术使 nqy  动态规划，明白原理，但到自己写就不知道怎么做了 ![](https://avatar.52pojie.cn/data/avatar/001/18/37/35_avatar_middle.jpg) mokson 如果跨一级走了几步，接着跨二级走了几步，如何算？![](https://avatar.52pojie.cn/data/avatar/001/44/93/85_avatar_middle.jpg)QingYi.  斐波那契数列 ![](https://avatar.52pojie.cn/data/avatar/001/27/89/26_avatar_middle.jpg) sam 喵喵 感谢分享，确实快 ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) trieszhang 崇拜数学大神
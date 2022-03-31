> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/bfYtz4j8IYFrUYmjlEp3QA)

> Python GUI 实战

文章：快学 Python

作者：黄伟呢  

  

本期案例是带着大家制作一个属于自己的 GUI 图形化界面—> 用于设计签名的哦 (效果如下图)，是不是感觉很好玩，是不是很想学习呢？限于篇幅，今天我们首先详细讲述一下 Tkinter 的使用方法。本来不准备详细讲述这个基础知识，但是我怕那些想学习的同学，学起来不过瘾，还是补充了这一章。  

![](https://mmbiz.qpic.cn/mmbiz_png/hRU7GdO3rMVgWwPkEujxdbiaFeJvz39sh62PeFJhXvxMzw9vugOa34GyqGSyl3IEicoCAzWmMwCuTdSLyOa1SKDQ/640?wx_fmt=png)

**tkinter 的简单应用**
-----------------

Tkinter 是 Python 的标准 GUI 库。Python 使用 Tkinter 可以快速地创建 GUI 应用程序。当然常用的 GUI 库还有 PyQt5，我们只需要知道这两个常用的即可，如果你真的想学习的话。由于 Tkinter 属于 Python 标准库，就不需要使用 pip 安装，直接导入使用即可。

**0****1**

**显示窗口**

  

  

*   root.mainloop() 显示窗口；
    
*   窗口默认会显示在电脑屏幕的左上角，非常小 (后面需要改进)；
    
      
    

```
from tkinter import *
from tkinter import messagebox
# 创建窗口：实例化一个窗口对象。
root = Tk()
# 显示窗口
root.mainloop()
"""
注意到：该窗口默认的显示位置在哪里，观察我下面的截图。
窗口默认显示在整个电脑屏幕的左上角，并且窗口大小特别小。
"""

```

结果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/hRU7GdO3rMVgWwPkEujxdbiaFeJvz39shicfPMkGJmVjOAiawOer3DTV6GV7blzoefbZqFp38KNN6L33JuZpsI98A/640?wx_fmt=png)

注意：上面 2 行代码，首先实例化一个窗口对象，然后我们展示了这个窗口，让其真正显示出来。接下来我们的操作，就是针对这个窗口的一系列优化操作，请注意：这个优化操作使用的代码，都是放在这 2 句代码中间。

**0****2**

**设置窗口大小**

  

  

*   root.geometry("600x400") 调整窗口的大小；
    
*   该方法中传入的是 "宽 x 高"，但是需要注意这个乘号是小写的英文字母 x，而不是这个 * 表示的乘号；
    
      
    

```
from tkinter import *
from tkinter import messagebox
# 创建窗口：实例化一个窗口对象。
root = Tk()
# 窗口大小
root.geometry("600x450")
# 显示窗口
root.mainloop()

```

结果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/hRU7GdO3rMVgWwPkEujxdbiaFeJvz39shFOgvibb84OUex8BJ1bQx3GlicYASGLUgEZSMYnM1tCia6pMYHGuo02VTA/640?wx_fmt=png)

**0****3**

**调整窗口位置 (使用的是同一个方法)**

  

  

*   root.geometry("600x400+374+182") 调整窗口的大小 + 位置；
    
*   374,182 表示的是窗口顶点，距离电脑左上角的坐标。这个数字怎么得到的呢？直接借助微信截图就可以显示了。
    
      
    

![](https://mmbiz.qpic.cn/mmbiz_png/hRU7GdO3rMVgWwPkEujxdbiaFeJvz39shk9wkJVyfLo6K3s0Wd8qDmRQKicwetXIolEcOzCdM5jd7nJlq2YVj0uA/640?wx_fmt=png)

操作代码如下：

```
from tkinter import *
from tkinter import messagebox
# 创建窗口：实例化一个窗口对象。
root = Tk()
# 窗口大小
root.geometry("600x450+374+182")
# 显示窗口
root.mainloop()

```

结果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/hRU7GdO3rMVgWwPkEujxdbiaFeJvz39shXekqFwp4Qq7qdAVOiaTx11hibWOwzaNjC8cHm8T0HVicUrV4SEbELwMJw/640?wx_fmt=png)

**0****4**

**设置窗口的标题**

  

  

*   root.title() 设置窗口的标题；
    
*   默认的窗口标题是 tk；
    
      
    

```
from tkinter import *
from tkinter import messagebox
# 创建窗口：实例化一个窗口对象。
root = Tk()
# 窗口大小
root.geometry("600x450+374+182")
#  窗口标题
root.title("我的个性签名设计")
# 显示窗口
root.mainloop()

```

结果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/hRU7GdO3rMVgWwPkEujxdbiaFeJvz39shs4ILkyvSqCJILmV85XnXvNm1b7hx3yziaXMbU4A2b6pYg9Lw4kr6OYw/640?wx_fmt=png)

**0****5**

**添加标签控件**

**并定位**

  

  

*   Label(root,text="签名") 添加标签控件
    
*   第一个参数传入的就是实例化的那个 root 窗口对象；第二个参数传入的要显示的那个标签文本；
    
*   仅仅添加标签控件后，还不行，必须要指定一个位置后，该标签控件才会真正展示出来，即最后需要调用 grid() 方法后，才会显示标签控件；
    
      
    

```
from tkinter import *
from tkinter import messagebox
# 创建窗口：实例化一个窗口对象。
root = Tk()
# 窗口大小
root.geometry("600x450+374+182")
#  窗口标题
root.title("我的个性签名设计")
# 添加标签控件
label = Label(root)
# 定位
label.grid()
# 显示窗口
root.mainloop()

```

结果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/hRU7GdO3rMVgWwPkEujxdbiaFeJvz39shRV2CIPwoqm3zAJyibdJRXStGZYyFwIR3NdAS3rCibusXuDicnF4SkY2Hw/640?wx_fmt=png)

当然你也可以想到，这个方法肯定还可以修改字体样式、字体大小、字体颜色呀？具体怎么操作呢？我们接着往下面看。

```
from tkinter import *
from tkinter import messagebox
# 创建窗口：实例化一个窗口对象。
root = Tk()
# 窗口大小
root.geometry("600x450+374+182")
#  窗口标题
root.title("我的个性签名设计")
# 添加标签控件
label = Label(root,text="签名",font=("宋体",25),fg="red")
"""
text参数用于指定显示的文本；
font参数用于指定字体大小和字体样式；
fg参数用于指定字体颜色；
"""
# 定位
label.grid()
# 显示窗口
root.mainloop()

```

结果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/hRU7GdO3rMVgWwPkEujxdbiaFeJvz39shtticjUshkzUiboXDGRQ2lrybVQXXvib6X50OQic5CRiaicYjzUTro54x8ribg/640?wx_fmt=png)

**0****6**

**添加输入框**

**并定位**

  

  

*   Entry(root,font=("宋体",25),fg="red") 添加输入框
    
*   第一个参数传入的就是实例化的那个 root 窗口对象；第二个参数可写可不写，指的是我们输入的字体的字体样式和字体大小；第三个参数同样可写可不写，表示的是我们输入的字体的颜色。
    
*   同样，仅仅使用上述代码并不会显示输入框，只有调用 grid() 方法，定位后，才会真正显示这个输入框；
    
      
    

```
from tkinter import *
from tkinter import messagebox
# 创建窗口：实例化一个窗口对象。
root = Tk()
# 窗口大小
root.geometry("600x450+374+182")
#  窗口标题
root.title("我的个性签名设计")
# 添加标签控件
label = Label(root,text="签名",font=("宋体",25),fg="red")
# 定位
label.grid()
# 添加输入框
entry = Entry(root,font=("宋体",25),fg="red")
entry.grid()
# 显示窗口
root.mainloop()

```

结果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/hRU7GdO3rMVgWwPkEujxdbiaFeJvz39shA5ianOt95d77cRibWETCjxORFfGErGoN0s4iaAbP3jROkRbx4SVyMSgsg/640?wx_fmt=png)

注意：很明显这样的摆放方式，并不是我们想要的。我们需要调整一下，下面我们专门花一个小节时间，去讲述怎么调整这个摆放位置。

**0****7**

**调整控件的摆放位置**

  

  

首先我们需要搞明白，显示窗口究竟采用的是什么样子的布局方式呢？其实是网格式的布局方式。那么什么又是网格式的布局方式呢？excel 表格你知道吧，一个个的格子就是网格式的布局方式。

![](https://mmbiz.qpic.cn/mmbiz_png/hRU7GdO3rMVgWwPkEujxdbiaFeJvz39shjcibRJsLfJJgcic64ZRgiahVrfpwNSBwDLiaZDJnWTia6pRiaFhE93ic6VRjw/640?wx_fmt=png)

好了！知道了上述原理后，我们现在来真正的调整这个控件摆放位置啦。

```
from tkinter import *
from tkinter import messagebox
# 创建窗口：实例化一个窗口对象。
root = Tk()
# 窗口大小
root.geometry("600x450+374+182")
#  窗口标题
root.title("我的个性签名设计")
# 添加标签控件
label = Label(root,text="签名：",font=("宋体",25),fg="red")
# 定位
label.grid()
"""
label.grid()等价于label.grid(row=0,column=0)
"""
# 添加输入框
entry = Entry(root,font=("宋体",25),fg="red")
entry.grid(row=0,column=1)
"""
row=0,column=1表示我们将输入框控件，放在第1行第2列的位置；
python语言中，这个下标是从0开始的。
"""
# 显示窗口
root.mainloop()

```

结果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/hRU7GdO3rMVgWwPkEujxdbiaFeJvz39shGB3jRfC4rUL86QlSD02jYuuyV8NV8MCfibWBGydu6nw4hy1j2d4azLw/640?wx_fmt=png)

**0****8**

**添加点击按钮**

  

  

*   Button(root,text="签名设计",font=("宋体",25),fg="red") 添加输入框
    
*   第一个参数传入的就是实例化的那个 root 窗口对象；第二个参数展示的是我们这个点击按钮的标签；第三个参数可写可不写，指的是点击按钮字体的字体样式和字体大小；第四个参数同样可写可不写，表示的是点击按钮字体的颜色。
    
*   同样，仅仅使用上述代码并不会显示输入框，只有调用 grid() 方法，定位后，才会真正显示这个点击按钮；
    
      
    

```
from tkinter import *
from tkinter import messagebox
# 创建窗口：实例化一个窗口对象。
root = Tk()
# 窗口大小
root.geometry("600x450+374+182")
#  窗口标题
root.title("我的个性签名设计")
# 添加标签控件
label = Label(root,text="签名：",font=("宋体",25),fg="red")
# 定位
label.grid()
# 添加输入框
entry = Entry(root,font=("宋体",25),fg="red")
entry.grid(row=0,column=1)
# 添加点击按钮
button = Button(root,text="签名设计",font=("宋体",25),fg="blue")
button.grid(row=1,column=1)
# 显示窗口
root.mainloop()

```

结果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/hRU7GdO3rMVgWwPkEujxdbiaFeJvz39sh6rpsGHn79LHpGCnN37PZH4OCiaGvnqRmmOtr88IexAf89riakOrG5FAQ/640?wx_fmt=png)

至此界面已经简单搭建起来了，接下来要做的就是输入一个名字，点击签名设计后，会显示我的这个签名，此时就需要借助爬虫啦！明天我们将会发布该文的下篇哦，敬请期待。

**0****9**

**点击按钮自定义功能**

  

  

这里最后补充这个知识点，我们点击按钮后，总是希望能够给我们返回点什么，所以呢，需要我们自定义函数。

```
from tkinter import *
from tkinter import messagebox
def func():
    print("我是黄同学")
# 创建窗口：实例化一个窗口对象。
root = Tk()
# 窗口大小
root.geometry("600x450+374+182")
#  窗口标题
root.title("我的个性签名设计")
# 添加标签控件
label = Label(root,text="签名：",font=("宋体",25),fg="red")
# 定位
label.grid()
# 添加输入框
entry = Entry(root,font=("宋体",25),fg="red")
entry.grid(row=0,column=1)
# 添加点击按钮
button = Button(root,text="签名设计",font=("宋体",25),fg="blue",command=func)
button.grid(row=1,column=1)
"""
command=func表示调用最开始定义的func函数。
func函数一定要在这句代码之前，因为这里需要调用这个func函数。
"""
# 显示窗口
root.mainloop()

```

结果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/hRU7GdO3rMVgWwPkEujxdbiaFeJvz39shmzEbdUM3gbb2yIK7VLLO1HEMhlDEKjGWvyhpZA95TSCmlyvick8ruBw/640?wx_fmt=png)
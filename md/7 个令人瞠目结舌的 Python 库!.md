> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/yf5ITSrtOXBKcLscyN4_oA)

在编程的世界中，Python 一直以其简洁、易读的语法而备受推崇。然而，除了 Python 本身的强大功能之外，还有许多令人瞠目结舌的 Python 库，它们为开发者们带来了无尽的惊喜和创造力。在本文中，笔者为大家分享 7 个这样的 Python 库，建议收藏。

1. rembg
--------

rembg 是一个强大的 Python 库，用于图像背景的自动去除。它基于深度学习和人工智能技术，能够高度准确地将图像中的背景抠出，留下前景图像。

**「安装 rembg」**

```
#Installation
pip install rembg


```

**「示例」**

```
# Importing libraries
from rembg import remove
import cv2 
# path of input image (my file: image.jpeg)
input_path = 'demo.jpg'
# path for saving output image and saving as a output.jpeg
output_path = 'output.jpg'
# Reading the input image
input = cv2.imread(input_path)
# Removing background
output = remove(input)
# Saving file 
cv2.imwrite(output_path, output)


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Zvl9ickIYtdITEQAEezY6BdXF4t4sdJUFOCRuljeL2xHSKqsicD8n9O81d1NkLuFhTgbKFKEakx7iaZ3uUwauBTAA/640?wx_fmt=png)

2. Ipyvolume
------------

Ipyvolume 是一个基于 Jupyter Notebook 的 Python 库，用于创建交互式的 3D 可视化和动画。它提供了丰富的功能和工具，使得在 Notebook 中可视化数据变得更加简单和直观。

**「示例代码」**

```
from colormaps import parula
X = np.arange(-5, 5, 0.25*1)
Y = np.arange(-5, 5, 0.25*1)
X, Y = np.meshgrid(X, Y)
R = np.sqrt(X**2 + Y**2)
Z = np.sin(R)

colormap = parula
znorm = Z - Z.min()
znorm /= znorm.ptp()
znorm.min(), znorm.max()
color = colormap(znorm)
ipv.figure()
mesh = ipv.plot_surface(X, Z, Y, color=color[...,:3])
ipv.show()


```

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Zvl9ickIYtdITEQAEezY6BdXF4t4sdJUFoYQGPy1ibuTuvtQ4uLZibu9jNtAFGmNsQKjdF7xsMiaqr5a6CLbpyE3Yw/640?wx_fmt=gif)

3. Pandas-Bokeh
---------------

Pandas-Bokeh 是一个使用 Bokeh 为 Pandas 数据帧提供交互式绘图的库，它对于创建交互式可视化数据非常有用。

**「安装 pandas-bokeh」**

```
pip install pandas-bokeh


```

**「交互式可视化效果」**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Zvl9ickIYtdITEQAEezY6BdXF4t4sdJUF8AicmTRB5XohMGr5pWEwjTw22Eiauia3Lg86ibAicrJeZREd1aJAw6ts7sQ/640?wx_fmt=gif)

**「示例代码」**

```
import pandas as pd
import pandas_bokeh

data = {
    'fruits':
    ['Apples', 'Pears', 'Nectarines', 'Plums', 'Grapes', 'Strawberries'],
    '2015': [2, 1, 4, 3, 2, 4],
    '2016': [5, 3, 3, 2, 4, 6],
    '2017': [3, 2, 4, 4, 5, 3]
}
df = pd.DataFrame(data).set_index("fruits")

p_bar = df.plot_bokeh.bar(
    ylabel="Price per Unit [€]", 
    title="Fruit prices per Year", 
    alpha=0.6)


```

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/Zvl9ickIYtdITEQAEezY6BdXF4t4sdJUFQdzcXlRmze9FS5SUgAibs1qZkMfaHkUEHhY7E40Y07CXXLfdukVq32Q/640?wx_fmt=gif)

4. Humanize
-----------

Humanize 是一个 Python 库，旨在将复杂的数据类型和单位转换为更易读的形式，以增加人类可理解性。它提供了一些有用的函数，用于将数字、时间、文件大小等转换为更友好和可读性强的格式。

使用 Humanize 库，你可以将整数转换为带有逗号的易读形式，例如将 1000 转换为 "1,000"；将时间间隔转换为更具描述性的形式，例如将 60 秒转换为 "1 分钟"；将字节数转换为更易理解的文件大小表示，例如将 1024 转换为 "1KB"。

**「安装 Humanize」**

```
pip install Humanize


```

**「示例（integers）」**

```
# Importing library

import humanize
import datetime as dt

# Formatting  numbers with comma
a =  humanize.intcomma(951009)

# converting numbers into words
b = humanize.intword(10046328394)

#printing

print(a)
print(b)


```

输出：

951,009 10.0 billion

**「示例（Date＆Time）」**

```
import humanize
import datetime as dt

a = humanize.naturaldate(dt.date(2023, 9,7))
b = humanize.naturalday(dt.date(2023, 9,7))

print(a)
print(b)


```

输出：

```
today
today


```

5. Pendulum
-----------

Pendulum 扩展了内置的 Python DateTime 模块，添加了一个更直观的 API，用于处理时区，对日期和时间进行操作，如添加时间间隔、删去日期以及在时区之间进行转换。它为格式化日期和时间提供了一个简单、人性化的 API。

**「安装 Pendulum」**

```
pip install pendulum


```

**「示例」**

```
# import library
import pendulum

dt = pendulum.datetime(2023, 8, 31)
print(dt)
 
#local() creates datetime instance with local timezone

local = pendulum.local(2023, 8, 31)
print("Local Time:", local)
print("Local Time Zone:", local.timezone.name)

# Printing UTC time
utc = pendulum.now('UTC')
print("Current UTC time:", utc)
 
# Converting UTC timezone into Europe/Paris time

europe = utc.in_timezone('Europe/Paris')
print("Current time in Paris:", europe)


```

输出：

2023-08-31T00:00:00+00:00 

Local Time: 2023-08-31T00:00:00+08:00 

Local Time Zone: Asia/Shanghai Current 

UTC time: 2023-09-07T04:06:05.436553+00:00 

Current time in Paris: 2023-09-07T06:06:05.436553+02:00

6. Sketchpy
-----------

Sketchpy 是一个用于对图像进行动画绘制的 Python 模块。sketchpy 模块是在 Python 中的 turtle 模块之上创建的。

**「安装 Sketchpy」**

```
pip install sketchpy


```

**「示例 - 使用 Python 绘制 Vijay」**

```
from sketchpy import library
myObject = library.vijay()
myObject.draw()


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Zvl9ickIYtdITEQAEezY6BdXF4t4sdJUFFXj92wKCHLBrexCxN4rqnNwSicmuicCt908XpkJI2OBiblhFMFPacEuRQ/640?wx_fmt=png)

7. FTFY
-------

FTFY 是一个 Python 库，它的全称是 "Fixes Text For You"，用于修复和纠正文本中的常见编码问题和 Unicode 字符问题。它可以自动检测和修复各种编码问题，使得文本在处理和显示时更加准确和一致。

**「安装 FTFY」**

```
pip install ftfy


```

**「示例」**

```
print(ftfy.fix_text('Correct the sentence using â€œftfyâ€\x9d.'))
print(ftfy.fix_text('âœ” No problems with text'))
print(ftfy.fix_text('Ã perturber la rÃ©flexion'))


```

**「输出」**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Zvl9ickIYtdITEQAEezY6BdXF4t4sdJUFzPXeaxSYNQI6pfN4gYfFOapWbzMXrvAqzoctaSG3tenmeLYxOVFMQQ/640?wx_fmt=png)
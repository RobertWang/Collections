> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652577852&idx=2&sn=d7e29796661f66a8ee54e692011f1554&chksm=84650876b312816020566f40db4a9b7bc81b8a2b62b129a9741a8f5821dec85e706799a32c70&scene=21#wechat_redirect)

所以开发一个图像化的小窗口，就变得很有必要。

今天，小 F 就给大家介绍七个 Python 必备的 GUI 库，每一个都值得学习。

**01. PyQt5**

PyQt5 由 Riverbank Computing 开发。基于 Qt 框架构建，是一个跨平台框架，可以给各种平台创建应用程序，包括：Unix、Windows、Mac OS。

PyQt 将 Qt 和 Python 结合在一起。它不只是一个 GUI 工具包。还包括了线程，Unicode，正则表达式，SQL 数据库，SVG，OpenGL，XML 和功能完善的 Web 浏览器，以及许多丰富的 GUI 小部件集合。

使用 pip 安装一下。  

```
# 安装PyQt5
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple PyQt5


```

安装成功后，来个 Hello Word 简单示例。  

```
import sys
from PyQt5.QtWidgets import QApplication, QWidget, QLabel, QVBoxLayout

# 建立application对象
app = QApplication(sys.argv)
# 建立窗体对象
w = QWidget()
# 设置窗体大小
w.resize(500, 500)

# 设置样式
w.layout = QVBoxLayout()
w.label = QLabel("Hello World!")
w.label.setStyleSheet("font-size:25px;margin-left:155px;")
w.setWindowTitle("PyQt5 窗口")
w.layout.addWidget(w.label)
w.setLayout(w.layout)

# 显示窗体
w.show()
# 运行程序
sys.exit(app.exec_())


```

结果如下。

![](https://mmbiz.qpic.cn/mmbiz_png/y0SBuxfLhakK1ZcBmkgK0AlfzU2rOFCefibXnhrHpn4ic9NibuqlIFyJjJ5F5XEbbIgeVmIlSo3gjPJ1QMRBg9cug/640?wx_fmt=png)

文档地址：

_https://riverbankcomputing.com/software/pyqt/intro_

教程链接：

_https://www.guru99.com/pyqt-tutorial.html_

**02. Tkinter**

Tkinter 是 Python 中最受欢迎的 GUI 库之一。由于它简单易学的语法，成为 GUI 开发初学者的首选之一。

Tkinter 提供了各种小部件，例如标签，按钮，文本字段，复选框和滚动按钮等。

支持 Grid(网格) 布局，由于我们的程序大多数都是矩形显示，这样即使是复杂的设计，开发起来也变得简单些。 

```
# 安装tkinter
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple tkinter


```

下面使用 Tkinter 设计一个 BMI 计算器。

以重量和高度作为输入，并在弹出框中返回 BMI 系数作为输出。

```
from tkinter import *
from tkinter import messagebox

def get_height():
    # 获取身高数据(cm)
    height = float(ENTRY2.get())
    return height

def get_weight():
    # 获取体重数据(kg)
    weight = float(ENTRY1.get())
    return weight

def calculate_bmi():
    # 计算BMI系数
    try:
        height = get_height()
        weight = get_weight()
        height = height / 100.0
        bmi = weight / (height ** 2)
    except ZeroDivisionError:
        messagebox.showinfo("提示", "请输入有效的身高数据!!")
    except ValueError:
        messagebox.showinfo("提示", "请输入有效的数据!")
    else:
        messagebox.showinfo("你的BMI系数是: ", bmi)

if __name__ == '__main__':
    # 实例化object，建立窗口TOP
    TOP = Tk()
    TOP.bind("<Return>", calculate_bmi)
    # 设定窗口的大小(长 * 宽)
    TOP.geometry("400x400")
    # 窗口背景颜色
    TOP.configure(background="#8c52ff")
    # 窗口标题
    TOP.title("BMI 计算器")
    TOP.resizable(width=False, height=False)
    LABLE = Label(TOP, bg="#8c52ff", fg="#ffffff", text="欢迎使用 BMI 计算器", font=("Helvetica", 15, "bold"), pady=10)
    LABLE.place(x=55, y=0)
    LABLE1 = Label(TOP, bg="#ffffff", text="输入体重(单位：kg):", bd=6,
                   font=("Helvetica", 10, "bold"), pady=5)
    LABLE1.place(x=55, y=60)
    ENTRY1 = Entry(TOP, bd=8, width=10, font="Roboto 11")
    ENTRY1.place(x=240, y=60)
    LABLE2 = Label(TOP, bg="#ffffff", text="输入身高(单位：cm):", bd=6,
                   font=("Helvetica", 10, "bold"), pady=5)
    LABLE2.place(x=55, y=121)
    ENTRY2 = Entry(TOP, bd=8, width=10, font="Roboto 11")
    ENTRY2.place(x=240, y=121)
    BUTTON = Button(bg="#000000", fg='#ffffff', bd=12, text="BMI", padx=33, pady=10, command=calculate_bmi,
                    font=("Helvetica", 20, "bold"))
    BUTTON.grid(row=5, column=0, sticky=W)
    BUTTON.place(x=115, y=250)
    TOP.mainloop()


```

界面如下。

![](https://mmbiz.qpic.cn/mmbiz_png/y0SBuxfLhakzVWMoE185VacLWxibkbKL0h5B228DxkruHtLHx1RSMHRQTD0kEkAbxEKNVr69zzTW98Sybm9jSMQ/640?wx_fmt=png)

当没有数据时，点击 BMI 按钮，会有与之对应的提示。

下面我们使用正确的数据，来看看结果。

![](https://mmbiz.qpic.cn/mmbiz_png/y0SBuxfLhakzVWMoE185VacLWxibkbKL0c1qhqCGg91w8miayYQZE9mhWyXxEqZnZCLquHibyY07jC16ibvPtBwtGg/640?wx_fmt=png)

使用起来感觉还是不错的。  

**03. Kivy**

Kivy 是另一个开源的 Python 库，最大的优点就是可以快速地编写**移动应用程序** (手机)。

Kivy 可以在不同的平台上运行，包括 Windows、Mac OS、Linux、Android、iOS 和树莓派。

此外也是免费使用的，获得了 MIT 许可。

```
# 安装kivy
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple kivy


```

一个基于 Kivy 的 Hello World 窗口。

```
from kivy.app import App
from kivy.uix.button import Button

class TestApp(App):
    def build(self):
        return Button(text=" Hello Kivy World ")

TestApp().run()


```

结果如下。  

![](https://mmbiz.qpic.cn/mmbiz_png/y0SBuxfLhakK1ZcBmkgK0AlfzU2rOFCe4rhWG0MpaKkVhoUBGNGIGgOVKwO6f0S3WGCVfRyLWpKLG8XicDpCjaQ/640?wx_fmt=png)

**04. wxPython**

wxPython 是一个跨平台 GUI 的 Python 库，可轻松创建功能强大稳定的 GUI，毕竟是用 C++ 编写的～

目前，支持 Windows，Mac OS X，macOS 和 Linux。

使用 wxPython 创建的应用程序 (GUI) 在所有平台上都具有原生外观。

```
# 安装wxPython
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple wxPython


```

下面使用 wxPython 创建一个基本的 GUI 示例。  

```
import wx

myapp = wx.App()
init_frame = wx.Frame(parent=None, title='WxPython 窗口')

init_frame.Show()
myapp.MainLoop()


```

结果如下。

![](https://mmbiz.qpic.cn/mmbiz_png/y0SBuxfLhakzVWMoE185VacLWxibkbKL0jn2iczicibd8vlNliakzkWyufJRxvUzgFqvSrFNIoCrrSnoM161d5fg89A/640?wx_fmt=png)

文档链接：_https://www.wxpython.org/_

**05. PySimpleGUI**

PySimpleGUI 也是基于 Python 的 GUI 框架。可以轻松制作自定义的 GUI。

采用了四种最流行的 GUI 框架 QT、Tkinter、WxPython 和 Remi，能够实现大多数样例代码，**降低了学习难度**。

Remi 将应用程序的界面转换为 HTML，以便在 Web 浏览器中呈现。

```
# 安装PySimpleGUI
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple PySimpleGUI


```

下面是一个简单的案例。  

```
import PySimpleGUI as sg

layout = [[sg.Text("测试 PySimpleGUI")], [sg.Button("OK")]]
window = sg.Window("样例", layout)
while True:
    event, values = window.read()
    if event == "OK" or event == sg.WIN_CLOSED:
        break
window.close()


```

结果如下。

![](https://mmbiz.qpic.cn/mmbiz_png/y0SBuxfLhakzVWMoE185VacLWxibkbKL0iaI3GR7cMoOYSOiaL7ebnaiaFsVaHbIYVpBibrpiaycPMSB90ljGnMPeKfQ/640?wx_fmt=png)

点击 OK 按钮，窗口消失。

**06. PyGUI**

PyGUI 是一个以简单 API 而闻名的 GUI 框架，减少 Python 应用与平台底层 GUI 之间的代码量。

轻量级的 API，可以让你的应用程序运行起来更流畅，更快速。

同时还开源代码，跨平台项目。目前可在基于 Unix 的系统，Windows 和 Mac OS 上运行。

Python2 和 Python3，都是可以支持的。

文档地址：

_https://www.cosc.canterbury.ac.nz/greg.ewing/python_gui/_

教程链接：

_https://realpython.com/pysimplegui-python/_

**07. Pyforms**

Pyforms 是用于开发 GUI 应用程序的一个跨平台框架。

![](https://mmbiz.qpic.cn/mmbiz_png/y0SBuxfLhakzVWMoE185VacLWxibkbKL04Z5FZMd6G3Bp2XpPwUemvzwGSQx2O9jcxakibXqy7uYpbDpaiaB4U2lQ/640?wx_fmt=png)

Pyforms 是一个 Python2.7/3.x 跨环境图形应用开发框架，模块化和代码复用可以节省大量工作。

允许应用程序在桌面，Web 和终端上运行，无需修改代码。

```
# 安装PyFroms
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple PyFroms


```

文档地址：_https://pyforms.readthedocs.io/en/v4/_

- EOF -

推荐阅读  点击标题可跳转

1、[Python 优化提速的 8 个小技巧](http://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652577669&idx=1&sn=17a24ecff586e09551f58619324e0472&chksm=846537cfb312bed91374b2183aac02f26b155125cdccc6252bd910c326b45151211185b14be4&scene=21#wechat_redirect)

2、[Python 最佳代码实践：性能、内存和可用性！](http://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652577647&idx=1&sn=85415edfb8206e1184dac9e3c0a664b1&chksm=84653725b312be3391e2db83d13f37d216feea8a61e1eb9f5679813f2830f6964d413753f6db&scene=21#wechat_redirect)

3、[Python 数据可视化，被 Altair 圈粉了！](http://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652577636&idx=2&sn=b4a9bbd288516b41958525c6ed467bc7&chksm=8465372eb312be386f8f58e721650ce2c34f4a6b7a906d97a1d3a40bf613f4773539cac88539&scene=21#wechat_redirect)
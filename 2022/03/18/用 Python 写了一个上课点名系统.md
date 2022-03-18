> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247512967&idx=1&sn=c61a9619661b35523defbf85f85783ec&chksm=e969ed8ade1e649cf5db3e60882e744f32bb64f334ab4746e767405ad0fb022912edd5cced37&mpshare=1&scene=1&srcid=0314I25ZM8OcdWnZoQvd2tT1&sharer_sharetime=1647233029141&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

[Pythn 人工智能技术 (ID:coder_experience) 第 766 期推文](http://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247489525&idx=1&sn=26e9ce9d935a845d03ce4a3d30ebc975&chksm=e96a49f8de1dc0eeda78a0f1fa1ab74bfa13222fecbb924f52e59b59cd8b431d6e0fd00cc1cb&scene=21#wechat_redirect)

**正文**

**大家好，我是 Python 人工智能技术**

今天刷到了一个这样的短视频，我寻思我是不是也可以写一个类似的上课点名程序，想法经不起等待，说写就写~

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/iaahNUMjJc7tYb5iaXZTbka46rvsF6ZK1zLNmeZjtySKnlGWk5zwoZ9DKAaekhJLkDzuzbiaabvQX88VSNAgj0pVw/640?wx_fmt=jpeg)

**一．准备工作**
==========

1.Tkinter
---------

> Tkinter 是 python 内置的 TK GUI 工具集。TK 是 Tcl 语言的原生 GUI 库。作为 python 的图形设计工具，它所使用的 Tcl 语言环境已经完全嵌入到了 python 解释器中。

我们使用 Tkinter 开发 GUI 界面。

2.PIL
-----

> PIL(Python Image Library) 库是 Python 语言的第三方库，需要通过 pip 工具安装。安装 PIL 库的方法如下，需要注意，安装库的名字是 pillow。
> 
> PIL 库支持图像储存、显示和处理，他能够处理几乎所有图片格式，可以完成对图像的缩放、剪裁、叠加以及向图像添加线条、图像和文字等操作。

使用 PIL 中的 Image,ImageTk 处理、引入一张图片，可以使用下面代码安装一下。

> `pip install pillow`

**二．预览**
========

1. 启动
-----

![图片](https://mmbiz.qpic.cn/mmbiz_png/ULibHgXIt3jwLtsmIo8Iss6qx1MiaOkVfuIJO7sNnPxtvDXzCP5zVqJ5k7HpS8WYIubeOBsDUCTy11FE5LtFqzZA/640?wx_fmt=png)

双击打开后，进入软件主界面，所有功能一目了然。程序会自动识别软件目录下的 names.txt，将里面的名字导入。

2. 开始点名 - 顺序点名
--------------

![图片](https://mmbiz.qpic.cn/mmbiz_gif/ULibHgXIt3jwLtsmIo8Iss6qx1MiaOkVfuZJodtRw0tL4RPY4yNCq27OibvGoysZ1zp9Czcicd5vusNEleX6GPPrmQ/640?wx_fmt=gif)

选择顺序点名后，点击开始，屏幕上就开始滚动出现人名，人名出现的概率是相同的，点击停止，人名就停止滚动，点名结束。

3. 开始点名 - 随机点名
--------------

![图片](https://mmbiz.qpic.cn/mmbiz_gif/ULibHgXIt3jwLtsmIo8Iss6qx1MiaOkVfuXFoibvutFJhOssmGn4Zl6Vib7rFEszKr9R47P5F48VpTqt3KmGzRtWmw/640?wx_fmt=gif)

点击随机点名，程序就会进行随机点名，人名出现的概率是随机的。

4. 手动加载人名单
----------

![图片](https://mmbiz.qpic.cn/mmbiz_gif/ULibHgXIt3jwLtsmIo8Iss6qx1MiaOkVfuWOCW9d6CmBicMSPIa4A3rRIl1WdFvZT9AIibsqHPSsPdUeMUJ1mo7rLw/640?wx_fmt=gif)

可以自己手动选择人名单，前提是人名单格式为 txt，且每个名字占一行。

5. 开始点名 - 顺序点名 - Pyqt5 版本
-------------------------

![图片](https://mmbiz.qpic.cn/mmbiz_gif/ULibHgXIt3jwLtsmIo8Iss6qx1MiaOkVfuGSaPTOiayuTXyX42iaDWJD2YQicOkyMLyGgAwO6UHZA6FVlVuaiakXfVtg/640?wx_fmt=gif)

用 Pyqt5 也写了一个版本，实现逻辑与 TK 版本相同，界面可能更好看了一些，但是文件大了许多，大家可以在后面总结部分自取。

**三．思路**
========

1. 整体实现思路
---------

![图片](https://mmbiz.qpic.cn/mmbiz_png/ULibHgXIt3jwLtsmIo8Iss6qx1MiaOkVfuvTedkAiaiaBQQJd1ckhMefoEzCaeQu8iaQlvJyr5TJathLfRWl6Bef34A/640?wx_fmt=png)

2. 点名实现思路
---------

![图片](https://mmbiz.qpic.cn/mmbiz_png/ULibHgXIt3jwLtsmIo8Iss6qx1MiaOkVfurqfh3CzrT78ArmicMvAHj3RiawSr1mkiaSpvky4KHHvOFYWhibYqv0icyKA/640?wx_fmt=png)

**四．源代码**
=========

point_names-GUI.py（主程序 GUI）
---------------------------

```
import random
import re
import time
import threading
from tkinter import *
from tkinter import ttk
from base64 import b64decode
from PIL import Image,ImageTk
from tkinter import messagebox
from tkinter.filedialog import askopenfilename
""""
2021-11-10点名/抽奖程序
主要亮点：
1.两种模式：
①顺序点名
②随机点名
2.自动识别人名单
3.支持手动导入人名单
4.人名单导入校验
5.人名显示位置自动矫正
6.最多显示五个大字
另外，搜索公众号顶级算法后台回复“算法心得”，获取一份惊喜礼包。
"""
imgs=['./point_name.png']
class APP:
    def __init__(self):
        self.root = Tk()
        self.running_flag=False #开始标志
        self.time_span=0.05 #名字显示间隔
        self.root.title('Point_name-V1.0')
        width = 680
        height = 350
        left = (self.root.winfo_screenwidth() - width) / 2
        top = (self.root.winfo_screenheight() - height) / 2
        self.root.geometry("%dx%d+%d+%d" % (width, height, left, top))
        self.root.resizable(0,0)
        self.create_widget()
        self.set_widget()
        self.place_widget()
        self.root.mainloop()
    def create_widget(self):
        self.label_show_name_var=StringVar()
        self.label_show_),foreground = '#1E90FF')
        self.btn_start=ttk.Button(self.root,text="开始",)
        self.btn_load_names=ttk.Button(self.root,text="手动加载人名单",)
        self.lf1=ttk.LabelFrame(self.root,text="点名方式")
        self.radioBtn_var=IntVar()
        self.radioBtn_var.set(1)
        self.radioBtn_sequence=ttk.Radiobutton(self.lf1,text="顺序点名",variable=self.radioBtn_var, value=1)
        self.radioBtn_random=ttk.Radiobutton(self.lf1,text="随机点名",variable=self.radioBtn_var, value=2)
        self.label_show_name_num=ttk.Label(self.root,font=('Arial', 20),foreground = '#FF7F50')
        paned = PanedWindow(self.root)
        self.img = imgs
        img_=b'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'
        the_img = b64decode(img_)#将图片硬编码到GUI
        paned.image = ImageTk.PhotoImage(data=the_img)
        self._img = Label(self.root, image=paned.image,background='black')
    def set_widget(self):
        default_name_="会是谁？"
        self.label_show_name_var.set(default_name_)
        self.label_show_name_adjust(default_name_)
        self.btn_start.config(command=lambda :self.thread_it(self.start_point_name))
        self.btn_load_names.config(command=self.load_names)
        init_names=self.load_names_txt("./names.txt")
        self.root.protocol('WM_DELETE_WINDOW',self.quit_window)
        self.root.bind('<Escape>',self.quit_window)
        if init_names:
            self.default_names=init_names   #1.文件存在但是无内容。2.文件不存在
            self.label_show_name_num.config(text=f"一共加载了{len(self.default_names)}个姓名")
        else:
            self.btn_start.config(state=DISABLED)
            self.label_show_name_num.config(text=f"请先手动导入人名单！")
    def place_widget(self):
        self.lf1.place(x=300,y=160,width=250,height=50)
        self.radioBtn_sequence.place(x=20,y=0)
        self.radioBtn_random.place(x=150,y=0)
        self.btn_start.place(x=300,y=220,width=100,height=30)
        self.btn_load_names.place(x=450,y=220,width=100,height=30)
        self._img.place(x=90, y=165, height=120, width=180)
        self.label_show_name_num.place(x=300,y=260)
    def label_show_name_adjust(self,the_name):
        if len (the_name)==1:
            self.label_show_name.place(x=280, y=10)
        elif len(the_name) == 2:
            self.label_show_name.place(x=180, y=10)
        elif len(the_name) == 3:
            self.label_show_name.place(x=120, y=10)
        elif len(the_name) == 4:
            self.label_show_name.place(x=80, y=10)
        else:
            self.label_show_name.place(x=0, y=10)
    def start_point_name(self):
        """
        启动之前进行判断，获取点名模式
        :return:
        """
        if len(self.default_names)==1:
            messagebox.showinfo("提示",'人名单就一个人，不用选了！')
            self.label_show_name_var.set(self.default_names[0])
            self.label_show_name_adjust(self.default_names[0])
            return
        if self.btn_start["text"]=="开始":
            self.btn_load_names.config(state=DISABLED)
            self.running_flag=True
            if isinstance(self.default_names,list):
                self.btn_start.config(text="就你了")
                if self.radioBtn_var.get()==1:
                    mode="sequence"
                elif self.radioBtn_var.get()==2:
                    mode="random"
                else:
                    pass
                self.thread_it(self.point_name_begin(mode))
            else:
                messagebox.showwarning("警告","请先导入人名单！")
        else:
            self.running_flag=False
            self.btn_load_names.config(state=NORMAL)
            self.btn_start.config(text="开始")
    def point_name_begin(self,mode):
        """
        开始点名，点名主函数
        :param mode:
        :return:
        """
        if mode == "sequence":
            if self.running_flag:
                self.always_ergodic()
        elif mode=="random":
            while True:
                    if self.running_flag:
                        random_choice_name=random.choice(self.default_names)
                        self.label_show_name_var.set(random_choice_name)
                        self.label_show_name_adjust(random_choice_name)
                        time.sleep(self.time_span)
                    else:
                        break
    def always_ergodic(self):
        """
        一直遍历此列表,使用死循环会造成线程阻塞
        :return:
        """
        for i in self.default_names:
            if self.running_flag:
                self.label_show_name_var.set(i)
                self.label_show_name_adjust(i)
                time.sleep(self.time_span)
                if i==self.default_names[-1]:
                    self.always_ergodic()
            else:
                break
    def load_names(self):
        """
        手动加载txt格式人名单
        :return:
        """
        filename = askopenfilename(
                filetypes = [('文本文件', '.TXT'), ],
                title = "选择一个文本文件",
            initialdir="./"
                )
        if filename:
            names=self.load_names_txt(filename)
            if names:
                self.default_names=names
                no_Chinese_name_num=len([n for n in names if not self.load_name_check(n)])
                if no_Chinese_name_num==0:
                    pass
                else:
                    messagebox.showwarning("请注意",f'导入名单有{no_Chinese_name_num}个不是中文名字')
                self.label_show_name_num.config(text=f"一共加载了{len(self.default_names)}个姓名")
                default_name_ = "会是谁？"
                self.label_show_name_var.set(default_name_)
                self.label_show_name_adjust(default_name_)
                self.btn_start.config(state=NORMAL)
            else:
                messagebox.showwarning("警告","导入失败，请检查！")
    def load_names_txt(self,txt_file):
        """
        读取txt格式的人名单
        :param txt_file:
        :return:
        """
        try:
            with open(txt_file,'r',encoding="utf-8")as f:
                names=[name.strip() for name in f.readlines()]
                if len(names)==0:
                    return False
                else:
                    return names
        except:
            return False
    def load_name_check(self,name):
        """
        对txt文本中的人名进行校验
        中文汉字->True
        非中文汉字->False
        :param name:
        :return:
        """
        regex = r'[\u4e00-\u9fa5]+'
        if re.match(regex,name):
            return True
        else:
            return False
    def thread_it(self,func,*args):
        t=threading.Thread(target=func,args=args)
        t.setDaemon(True)
        t.start()
    def quit_window(self,*args):
        """
        程序退出触发此函数
        :param args:
        :return:
        """
        ret=messagebox.askyesno('退出','确定要退出？')
        if ret:
            self.root.destroy()
if __name__ == '__main__':
    a=APP()
```

**五．总结**
========

本次使用 Tkinter 开发了一款上课点名程序，此程序可以用于点名、抽奖… 代码不到 200 行，程序简单又实用，主要有以下六个亮点：

1. 两种模式：顺序点名、随机点名

2. 自动识别人名单

3. 支持手动导入人名单

4. 人名单导入校验

5. 人名显示位置自动矫正

6. 最多显示五个大字

****你还有什****么想要补充的吗？****  

> 免责声明：本文内容来源于网络，文章版权归原作者所有，意在传播相关技术知识 & 行业趋势，供大家学习交流，若涉及作品版权问题，请联系删除或授权事宜。  

--END--

[保姆级别！带你搭建一台服务器！](http://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247510687&idx=2&sn=aad0db260df36d4bfb94b0a28ec92c47&chksm=e969f492de1e7d84dab0d87f6ba9d491a1992c09b91da3e9b41f04aa9e19e6760b50ea7e231a&scene=21#wechat_redirect)

[一个很酷的博客系统](http://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247511873&idx=1&sn=e47d03bd0d618f26876f5e06e625e58a&chksm=e969f14cde1e785af0b15c025418dfc1e7d4d6d09753c859d4a80499122558e054ddce860619&scene=21#wechat_redirect)  

[68 个 Python 内置函数详解](http://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247512721&idx=2&sn=fc70dc0ed8be50d197aa253341a312a8&chksm=e969ec9cde1e658a9e1942d25954882cf7dad2c4377e0e1453711ec94b44ce09f8fbd37ddc2c&scene=21#wechat_redirect)  

[一个悄然成为世界最流行的操作系统](http://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247512900&idx=2&sn=c6e45d76002a3331dac48dc276f41250&chksm=e969ed49de1e645ff38d87a03eefc8c19c0fc826234993aa7c6b3cfe9695ef314ac5e3d2727c&scene=21#wechat_redirect)  

[Python | AioHttp 异步抓取火星图片](http://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247512917&idx=2&sn=2fab61a05e1f77f37e91d169834a5277&chksm=e969ed58de1e644ee363af63f109c298318bfb7851c8658410ee4241e880719c604807b16ea5&scene=21#wechat_redirect)  

[Python 的打包神器——Nuitka](http://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247512942&idx=2&sn=9d6d97430cacddb30431e333171d44bf&chksm=e969ed63de1e64752544fdc41a2458fad4a6ba9fc7e0611bfcfb0c0611f2ed1fa5d24fe94977&scene=21#wechat_redirect)  

[实战：带你用 Python 爬取抖音 app 视频](http://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247512830&idx=2&sn=510c904b25c482a20f4f6ba35b4a4794&chksm=e969ecf3de1e65e5364266f413ebb9d95aba440bdd4157b86b0cf68df2929fbcebbf3a853c9b&scene=21#wechat_redirect)  

[史上首次！苹果 / 谷歌 / 微软 / 火狐合力解决 Web 兼容性问题](http://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247512873&idx=2&sn=c3a79f5dad27cd4e90960488ea265e92&chksm=e969ed24de1e6432af8fd96f696c187789aec189a8dfa63249719bc3655adb0932cb954adb63&scene=21#wechat_redirect)
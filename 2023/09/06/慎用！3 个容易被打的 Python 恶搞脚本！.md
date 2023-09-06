> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/594773470)

Python 无限恶搞朋友电脑，别提有多爽了，哈哈，打造自己的壁纸修改器，电脑无限锁屏， 无限弹窗，都在这里！！！

**1、修改电脑桌面壁纸**  
**工具使用**  

*   开发环境：python3.7， Windows10
*   使用工具包：win32api，win32con, win32gui, os, random
*   win32 的工具下载命令：

```
pip install pywin32

```

**项目解析思路**  
桌面数据信息是保存在注册表上的内容，数据保存在第二项 的 Control PanelDesktop 子项里就可以了。  

  
通过 win32api 打开注册表选择配置的对应子项生成对应句柄

```
k = win32api.RegOpenKeyEx(win32con.HKEY_CURRENT_USER, 'Control 
PanelDesktop', 0, win32con.KEY_SET_VALUE)

```

将桌面样式调整拉伸模式 2 拉伸壁纸 0 壁纸居中 6 适应 10 填充  
准备好需要修改的图片壁纸（壁纸数据通过爬虫技术进行采集）  

![](https://pic1.zhimg.com/v2-ef5cbf301605392a943f62313b6080e4_r.jpg)

  
win32gui 提交数据将桌面修改成自己准备的桌面壁纸

```
win32gui.SystemParametersInfo(win32con.SPI_SETDESKWALLPAPER, img_path, 
win32con.SPIF_SENDWININICHANGE)

```

```
源码分享

import win32api   # 调用Windows底层的接口配置   pip install pywin32
import win32con   # 修改数据
import win32gui   # 提交对应的数据
import os       # Python 管理文件工具包
import random   # 取出对应的随机值
import time   # 时间管理模块

def set_wallpaper():
    path = os.listdir(r'图片文件夹')
    for i in path:
        img_path = r'图片文件夹' + "\" + i
        print(img_path)
        # 打开注册表  句柄
        k = win32api.RegOpenKeyEx(win32con.HKEY_CURRENT_USER, 'Control PanelDesktop', 0, win32con.KEY_SET_VALUE)
        # 2 拉伸壁纸   0 壁纸居中  6 适应 10 填充
        win32api.RegSetValueEx(k, "WallpaperStyle", 0, win32con.REG_SZ, '2')
        win32gui.SystemParametersInfo(win32con.SPI_SETDESKWALLPAPER, img_path, win32con.SPIF_SENDWININICHANGE)
        time.sleep(10)

set_wallpaper()

```

**2、电脑无限锁屏工具使用**  

*   开发环境：python3.7， Windows10
*   使用工具包：ctypes ctypes

ctypes ctypes 是 Python 的外部函数库。它提供了与 C 兼容的数据类型，并允许调用 DLL 或共享库中的函数。可使用该模块以纯 Python 形式对这些库进行封装。通过操作系统底层的 user32.dll 实现锁屏效果  

![](https://pic4.zhimg.com/v2-87df70da3199407e22f055de95ba896f_r.jpg)

```
def lock_windows(): while True: user = windll.LoadLibrary("user32.dll") 
user.LockWorkStation() lock_windows()

```

代码以提供各位大佬可自行尝试  

![](https://pic1.zhimg.com/v2-9995d0700af2a22f61057aa7a1ab86cc_r.jpg)

  
打包的方法：pyinstaller -F 你的文件名  
打包之后可给你朋友同事尝试一下（为了朋友同事间的友谊最好加个延时操作）  
**3、无限弹窗**

之前大家应该都了解这样的， 通过 os 模块执行打开 cmd 窗口页面（确保是环境变量里有的选项）  

```
for i in range(2000):
    os.system('start cmd')

```

电脑配置低的请勿轻易尝试 win 关闭页面命令 ：**start taskkill /f /im cmd.exe /t**
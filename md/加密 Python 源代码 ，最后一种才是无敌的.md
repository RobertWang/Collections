> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIzMzMzOTI3Nw==&mid=2247502467&idx=1&sn=f662d4cbcb7f819f6cb3213111ba1a0c&chksm=e885aa61dff223775cc6cc2e54c95db83afaaa4de2ca6f0e8bc72a5cb54ce4ae25b988d51438&mpshare=1&scene=1&srcid=0708QNoc2vM4P85qF278Snnr&sharer_sharetime=1625734040890&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

Python 语法简单，使用方便，我们可以使用它快速地编写程序和构建应用。在编写好程序之后，我们必然要进行程序的分发。

**如果我们写的是图形界面程序，可能会打包成相应操作系统平台的二进制运行文件**（当然也可能直接发 Python 代码给别人运行）。

**如果我们写的是 Web 应用程序，则需要部署在指定的服务器上**。

而这，就涉及到了**源码保护的问题**。我们不需要程序的使用者能够看到程序的源码。但是，Python 作为一门动态语言和脚本语言，运行通过它编写的程序，并不需要进行静态编译和打包的过程，**对其代码进行加密是一件很麻烦、复杂和困难的事情**。

如果构建好的 Python 应用程序只是我们内部使用，或者部署在服务器上以 SaaS 化的形式供使用者使用，那么也根本无需考虑 Python 代码加密和源码泄露的问题。

但是，如果我们编写的程序是要进行商业授权的呢？

**源码的保护则是必须要做的一件事情。**

虽然很难，虽然不是十分完美，但是多增加一道门槛，也就多抵挡一些闲得蛋疼的人搞破解。

下面，介绍几种常见 Python 应用程序的代码加密方式，以供参考：

# **一、桌面图形程序加密**
----------------

通常情况下，我们使用 PyQt5、Tkinter、WxPython 等框架编写的图形程序会使用 PyInstaller 进行打包，生成平台的二进制运行文件，比如 Windows 下的 exe 文件。

不过，**使用 PyInstaller 编译打包出来的程序，很容易很反编译回去**。

比如，使用`pyinstxtractor`这个工具，就能把 PyInstallers 编译出来的 exe 还原回去；之后，再对还原出来的 pyc 文件进行反编译即可。

具体的使用方法，大家可以网上搜索，都有很多文章。

如何提高图形程序打包出二进制文件的安全性呢？

之前我们在介绍 PyQt5 程序打包时，有提到过使用 Nuitka 这个工具来减少生成二进制文件的大小。

其实，**Nuitka 会将 Python 程序转化为 C 语言程序，然后再进行编译打包为二进制文件**。众所周知，反编译 C 程序的难度是巨大的。以此，我们就极高地保障了图形界面程序的源码安全性。

# **二 、Web 应用程序**
-----------------

对于 Python 编写的 Web 应用程序，我们一般直接将其部署在服务器上然后对外进行服务。

但是如果是一个私有化部署的应用程序，既需要部署在客户的机器上，又不想客户看到应用程序的源码。

这时候，可以考虑**将 Python 代码文件编译为 C 文件，然后再将 C 文件编译为操作系统的动态链接库文件**（Linux 下的 .so 文件和 Windows 下的 .pyd 文件）。

以上步骤需要使用第三方库 cython，然后编写一个 setup.py 文件用来指定需要处理的 Python 文件，例如：

```
from distutils.core import setupfromCython.Buildimport cythonizesetup(ext_modules = cythonize(["zmister.py"]))


```

这样，就可以把 Python 文件编译为特定操作系统平台的动态链接库文件了。

同时，有一个第三方库 **jmpy3** 对上述流程进行了优化，支持单个文件和整个项目进行编译，使用起来更加友好：

![](https://mmbiz.qpic.cn/mmbiz_png/AnkmC6oCTPB1V4RKVB9B8Vx13NSWSlCTouzoJDbs3lojYFKlBMhgkS39tUXdt6eMDTqXWUYpaIAeYcH7ZepeKg/640?wx_fmt=png)

需要注意的是，使用这种方式加密后的文件**需要使用生成时的 Python 版本**，这也算是一个小缺点。但是这个缺点可以**通过打包为 Docker 镜像的方式解决**掉。

# **三、通用加密**
------------

除了上述两种方案，还有一个工具——PyArmor 能够实现 Python 代码的加密。

![](https://mmbiz.qpic.cn/mmbiz_png/AnkmC6oCTPB1V4RKVB9B8Vx13NSWSlCT5AHwNOl1hCjEPren59MmBFicvKky2M657AEUgfbdm6GJaboHtD8AmSg/640?wx_fmt=png)

PyArmor 是一个用于加密和保护 Python 脚本的工具。它能够在运行时刻保护 Python 脚本的二进制代码不被泄露，设置加密后 Python 源代码的有效期限，绑 定加密后的 Python 源代码到硬盘、网卡等硬件设备。它的保障机制主要包括:

*   加密编译后的代码块，保护模块中的字符串和常量
    
*   在脚本运行时候动态加密和解密每一个函数（代码块）的二进制代码
    
*   代码块执行完成之后清空堆栈局部变量
    
*   通过授权文件限制加密后脚本的有效期和设备环境
    

**除了对 Python 代码进行加密，PyArmor 还能设置 Python 程序的许可方式，比如设置程序的使用期限、设置允许运行的设备、扩展其他认证方式**等：

![](https://mmbiz.qpic.cn/mmbiz_png/AnkmC6oCTPB1V4RKVB9B8Vx13NSWSlCTENKUwMhm5MJicPasmer99jibDsVibFbrX2nDiaMsg103NFwjAEkwC5dv4g/640?wx_fmt=png)

我们直接使用 pip 命令即可对其进行安装：

```
pip install pyarmor


```

然后，使用`obfuscate`选项就能对代码进行加密：

```
pyarmor obfuscate foo.py


```

使用`licenses`选项即可生成许可文件：

```
pyarmor licenses \--expired "2018-12-31" \--bind-disk "100304PBN2081SF3NJ5T" \--bind-mac "70:f1:a1:23:f0:94" \--bind-ipv4 "202.10.2.52" \        r001


```

使用`--with-license`参数即可指定许可文件：

```
pyarmor obfuscate --with-license licenses/r001/license.lic foo.py


```

使用`pack`选项即可打包脚本：

```
pyarmor pack foo.py


```

需要注意的是，pyarmor 是一个共享软件，安装之后处于试用模式，在试用模式下有一些限制，如果购买的话，也不贵，298 的价格还是很良心的。

![](https://mmbiz.qpic.cn/mmbiz_png/AnkmC6oCTPB1V4RKVB9B8Vx13NSWSlCTAmqGskpBweMEtgWdwBXicq03CfiaXGxjxZNpHGjAxwxwup1UMDefibvAg/640?wx_fmt=png)

# **四、最后**
----------

除了代码加密，Python 社区内的很多观点也认为，加密是徒劳的，任何加密都有可能被破解，有一个良好的**法律约束条款**可能是更好的选择，而且如今的商业模式倾向于**靠服务收费**而非产品收费。
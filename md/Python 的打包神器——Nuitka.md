> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247512942&idx=2&sn=9d6d97430cacddb30431e333171d44bf&chksm=e969ed63de1e64752544fdc41a2458fad4a6ba9fc7e0611bfcfb0c0611f2ed1fa5d24fe94977&scene=21#wechat_redirect)

[Pythn 人工智能技术 (ID:coder_experience) 第 766 期推文](http://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247489525&idx=1&sn=26e9ce9d935a845d03ce4a3d30ebc975&chksm=e96a49f8de1dc0eeda78a0f1fa1ab74bfa13222fecbb924f52e59b59cd8b431d6e0fd00cc1cb&scene=21#wechat_redirect)

**正文**

**大家好，我是 Python 人工智能技术**

### 一. pyinstaller 和 Nuitka 使用感受  

#### 1.1 使用需求

> 这次也是由于项目需要，要将 python 的代码转成 exe 的程序，在找了许久后，发现了 2 个都能对 python 项目打包的工具——pyintaller 和 nuitka。

这 2 个工具同时都能满足项目的需要：

*   隐藏源码。这里的 pyinstaller 是通过设置 key 来对源码进行加密的；而 nuitka 则是将 python 源码转成 C++（这里得到的是二进制的 pyd 文件，防止了反编译），然后再编译成可执行文件。
    
*   方便移植。用户使用方便，不用再安装什么 python 啊，第三方包之类的。
    

#### 1.2 使用感受

2 个工具使用后的最大的感受就是：

*   pyinstaller 体验很差！
    

*   一个深度学习的项目最后转成的 exe 竟然有近 3 个 G 的大小（pyinstaller 是将整个运行环境进行打包），对，你没听错，一个 EXE 有 3 个 G！
    
*   打包超级慢，启动超级慢。
    

*   nuitka 真香！
    

*   同一个项目，生成的 exe 只有 7M！
    
*   打包超级快（1min 以内），启动超级快。
    

### 二. Nuitka 的安装及使用

#### 2.1 nuitka 的安装

*   直接利用 pip 即可安装：`pip install Nuitka`
    
*   下载 vs2019(MSVS) 或者 MinGW64，反正都是 C++ 的编译器，随便下。
    

#### 2.2 使用过程

对于第三方依赖包较多的项目（比如需要 import torch,tensorflow,cv2,numpy,pandas,geopy 等等）而言，这里最好打包的方式是只将属于自己的代码转成 C++，不管这些大型的第三方包！另外，搜索公众号 Linux 就该这样学后台回复 “git 书籍”，获取一份惊喜礼包。

以下是我 demo 的一个目录结构（这里使用了 pytq5 框架写的界面）：

```
├─utils//源码1文件夹├─src//源码2文件夹├─logo.ico//demo的图标└─demo.py//main文件
```

使用以下命令（调试）直接生成 exe 文件：

```
nuitka --standalone --show-memory --show-progress --nofollow-imports --plugin-enable=qt-plugins --follow-import-to=utils,src --output-dir=out --windows-icon-from-ico=./logo.ico demo.py
```

这里简单介绍下我上面的 nuitka 的命令：

*   `--standalone`：方便移植到其他机器，不用再安装 python
    
*   `--show-memory --show-progress`：展示整个安装的进度过程
    
*   `--nofollow-imports`：不编译代码中所有的 import，比如 keras，numpy 之类的。
    
*   `--plugin-enable=qt-plugins`：我这里用到 pyqt5 来做界面的，这里 nuitka 有其对应的插件。
    
*   `--follow-import-to=utils,src`：需要编译成 C++ 代码的指定的 2 个包含源码的文件夹，这里用`,`来进行分隔。
    
*   `--output-dir=out`：指定输出的结果路径为 out。
    
*   `--windows-icon-from-ico=./logo.ico`：指定生成的 exe 的图标为 logo.ico 这个图标，这里推荐一个将图片转成 ico 格式文件的网站（比特虫）。
    
*   `--windows-disable-console`：运行 exe 取消弹框。这里没有放上去是因为我们还需要调试，可能哪里还有问题之类的。
    

经过 1min 的编译之后，你就能在你的目录下看到：

```
├─utils//源码1文件夹├─src//源码2文件夹├─out//生成的exe文件夹
    ├─demo.build 
    └─demo.dist
		└─demo.exe//生成的exe文件├─logo.ico//demo的图标└─demo.py//main文件
```

当然这里你会发现真正运行 exe 的时候，会报错：`no module named torch,cv2,tensorflow`等等这些没有转成 C++ 的第三方包。

这里需要找到这些包（我的是在 software\python3.7\Lib\site-packages 下）复制（比如 numpy,cv2 这个文件夹）到`demo.dist`路径下。

至此，exe 能完美运行啦！

****你还有什****么想要补充的吗？****  

> 免责声明：本文内容来源于网络，文章版权归原作者所有，意在传播相关技术知识 & 行业趋势，供大家学习交流，若涉及作品版权问题，请联系删除或授权事宜。
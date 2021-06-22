> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU2NTczODU3NA==&mid=2247494541&idx=1&sn=b4074db40c755dc6f569bed23a61a8af&chksm=fcb586accbc20fbaf4b04144a55edd963a92af8ce2602cd8045cb54549835a995c7dab114e3d&mpshare=1&scene=1&srcid=0622NWyGwOAhxoRwVr49UWC3&sharer_sharetime=1624361303916&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

作者：写代码的明哥  
来源：Python 编程时光

所有的 Python 开发者都清楚，Python 之所以如此受欢迎，能够在众多高级语言中，脱颖而出，除了语法简单，上手容易之外，更多还要归功于 Python 生态的完备，有数以万计的 Python 爱好者愿意以 Python 为基础封装出各种有利于开发的第三方工具包。

这才使用我们能够以最快的速度开发出一个满足基本需要的项目，而不是每次都重复造轮子。

Python 从 1991 年诞生到现在，已经过去 28 个年头了，这其间产生了数以万计的第三方包，且每个包都会不断更新，会有越来越多的版本。

当你在一个复杂的项目环境中，如果没有一个有效的依赖包管理方案，项目的维护将会是一个大问题。

pip 是官方推荐的包管理工具，在大多数开发者眼里，pip 几乎是 Python 的标配。

 说到 pip ，大家都不会陌生。但我相信不少人，只是熟悉几个常用的用法，而对于其他几个低频且实用的用法，却知之甚少，这两天，我查阅官方文档，把这些用法整理了一下，应该是网络上比较全的介绍。

1. 查询软件包
--------

查询当前环境安装的所有软件包

```
$ pip list


```

查询 pypi 上含有某名字的包

```
$ pip search pkg


```

查询当前环境中可升级的包

```
$ pip list --outdated


```

查询一个包的详细内容

```
$ pip show pkg


```

2. 下载软件包
--------

在不安装软件包的情况下下载软件包到本地

```
$ pip download --destination-directory /local/wheels -r requirements.txt


```

下载完，总归是要安装的，可以指定这个目录中安装软件包，而不从 pypi 上安装。

```
$ pip install --no-index --find-links=/local/wheels -r requirements.txt


```

当然你也从你下载的包中，自己构建生成 wheel 文件

```
$ pip install wheel
$ pip wheel --wheel-dir=/local/wheels -r requirements.txt


```

3. 安装软件包
--------

使用 `pip install <pkg>` 可以很方便地从 pypi 上搜索下载并安装 python 包。

如下所示

```
$ pip install requests


```

这是安装包的基本格式，我们也可以为其添加更多参数来实现不同的效果。

**3.1 只从本地安装，而不从 pypi 安装**

```
# 前提你得保证你已经下载 pkg 包到 /local/wheels 目录下
$ pip install --no-index --find-links=/local/wheels pkg


```

**3.2 限定版本进行软件包安装**

以下三种，对单个 python 包的版本进行了约束

```
# 所安装的包的版本为 2.1.2
$ pip install pkg==2.1.2

# 所安装的包必须大于等于 2.1.2
$ pip install pkg>=2.1.2

# 所安装的包必须小于等于 2.1.2
$ pip install pkg<=2.1.2


```

以下命令用于管理 / 控制整个 python 环境的包版本

```
# 导出依赖包列表
pip freeze >requirements.txt

# 从依赖包列表中安装
pip install -r requirements.txt

# 确保当前环境软件包的版本(并不确保安装)
pip install -c constraints.txt


```

**3.3 限制不使用二进制包安装**

由于默认情况下，wheel 包的平台是运行 pip download 命令 的平台，所以可能出现平台不适配的情况。

比如在 MacOS 系统下得到的 pymongo-2.8-cp27-none-macosx_10_10_intel.whl 就不能在 linux_x86_64 安装。

使用下面这条命令下载的是 tar.gz 的包，可以直接使用 pip install 安装。

比 wheel 包，这种包在安装时会进行编译，所以花费的时间会长一些。

```
# 下载非二进制的包
$ pip download --no-binary=:all: pkg

#　安装非二进制的包
$ pip install pkg --no-binary


```

**3.4 指定代理服务器安装**

当你身处在一个内网环境中时，无法直接连接公网。这时候你使用`pip install` 安装包，就会失败。

面对这种情况，可以有两种方法：

1.  下载离线包拷贝到内网机器中安装
    
2.  使用代理服务器转发请求
    

第一种方法，虽说可行，但有相当多的弊端

*   步骤繁杂，耗时耗力
    
*   无法处理包的依赖问题
    

这里重点来介绍，第二种方法：

```
$ pip install --proxy [user:passwd@]http_server_ip:port pkg


```

每次安装包就发输入长长的参数，未免有些麻烦，为此你可以将其写入配置文件中：`$HOME/.config/pip/pip.conf`

对于这个路径，说明几点

*   不同的操作系统，路径各不相同
    

```
# Linux/Unix:
/etc/pip.conf
~/.pip/pip.conf
~/.config/pip/pip.conf

# Mac OSX:
~/Library/Application Support/pip/pip.conf
~/.pip/pip.conf
/Library/Application Support/pip/pip.conf

# Windows:
%APPDATA%\pip\pip.ini
%HOME%\pip\pip.ini
C:\Documents and Settings\All Users\Application Data\PyPA\pip\pip.conf (Windows XP)
C:\ProgramData\PyPA\pip\pip.conf (Windows 7及以后) 


```

*   若在你的机子上没有此文件，则自行创建即可
    

如何配置，这边给个样例：

```
[global]
index-url = http://mirrors.aliyun.com/pypi/simple/ 

# 替换出自己的代理地址，格式为[user:passwd@]proxy.server:port
proxy=http://xxx.xxx.xxx.xxx:8080 

[install]
# 信任阿里云的镜像源，否则会有警告
trusted-host=mirrors.aliyun.com 


```

**3.5 安装用户私有软件包**

很多人可能还不清楚，python 的安装包是可以用户隔离的。

如果你拥有管理员权限，你可以将包安装在全局环境中。在全局环境中的这个包可被该机器上的所有拥有管理员权限的用户使用。

如果一台机器上的使用者不只一样，自私地将在全局环境中安装或者升级某个包，是不负责任且危险的做法。

面对这种情况，我们就想能否安装单独为我所用的包呢？

庆幸的是，还真有。

我能想到的有两种方法：

1.  使用虚拟环境
    
2.  将包安装在用户的环境中
    

虚拟环境，之前写过几篇文章，这里不再展开讲。

今天的重点是第二种方法，教你如何安装用户私有的包？

命令也很简单，只要加上 `--user` 参数，pip 就会将其安装在当前用户的 `~/.local/lib/python3.x/site-packages` 下，而其他用户的 python 则不会受影响。

```
pip install --user pkg


```

来举个例子

```
# 在全局环境中未安装 requests
[root@localhost ~]# pip list | grep requests   
[root@localhost ~]# su - wangbm
[root@localhost ~]# 

# 由于用户环境继承自全局环境，这里也未安装
[wangbm@localhost ~]# pip list | grep requests 
[wangbm@localhost ~]# pip install --user requests  
[wangbm@localhost ~]# pip list | grep requests 
requests (2.22.0)
[wangbm@localhost ~]# 

# 从 Location 属性可发现 requests 只安装在当前用户环境中
[wangbm@ws_compute01 ~]$ pip show requests
---
Metadata-Version: 2.1
Name: requests
Version: 2.22.0
Summary: Python HTTP for Humans.
Home-page: http://python-requests.org
Author: Kenneth Reitz
Author-email: me@kennethreitz.org
Installer: pip
License: Apache 2.0
Location: /home/wangbm/.local/lib/python2.7/site-packages
[wangbm@localhost ~]$ exit
logout

# 退出 wangbm 用户，在 root 用户环境中发现 requests 未安装
[root@localhost ~]$ pip list | grep requests
[root@localhost ~]$ 


```

当你身处个人用户环境中，python 导包时会先检索当前用户环境中是否已安装这个包，已安装则优先使用，未安装则使用全局环境中的包。

验证如下：

```
>>> import sys
>>> from pprint import pprint 
>>> pprint(sys.path)
['',
 '/usr/lib64/python27.zip',
 '/usr/lib64/python2.7',
 '/usr/lib64/python2.7/plat-linux2',
 '/usr/lib64/python2.7/lib-tk',
 '/usr/lib64/python2.7/lib-old',
 '/usr/lib64/python2.7/lib-dynload',
 '/home/wangbm/.local/lib/python2.7/site-packages',
 '/usr/lib64/python2.7/site-packages',
 '/usr/lib64/python2.7/site-packages/gtk-2.0',
 '/usr/lib/python2.7/site-packages',
 '/usr/lib/python2.7/site-packages/pip-18.1-py2.7.egg',
 '/usr/lib/python2.7/site-packages/lockfile-0.12.2-py2.7.egg']
>>> 


```

**3.6 延长超时时间**

若网络情况不是很好，在安装某些包时经常会因为 ReadTimeout 而失败。

对于这种情况，一般重试几次就好了。

但是这样难免有些麻烦，有没有更好的解决方法呢？

有的，可以通过延长超时时间。

```
$ pip install --default-timeout=100 <packages>


```

4. 卸载软件包
--------

就一条命令，不再赘述

```
$ pip uninstall pkg


```

5. 升级软件包
--------

想要对现有的 python 进行升级，其本质上也是先从 pypi 上下载最新版本的包，再对其进行安装。所以升级也是使用 `pip install`，只不过要加一个参数 `--upgrade`。

```
$ pip install --upgrade pkg


```

在升级的时候，其实还有一个不怎么用到的选项 `--upgrade-strategy`，它是用来指定升级策略。

它的可选项只有两个：

*   `eager` ：升级全部依赖包
    
*   `only-if-need`：只有当旧版本不能适配新的父依赖包时，才会升级。
    

在 pip 10.0 版本之后，这个选项的默认值是 `only-if-need`，因此如下两种写法是一互致的。

```
pip install --upgrade pkg1 
pip install --upgrade pkg1 --upgrade-strategy only-if-need


```

6. 配置文件
-------

由于在使用 pip 安装一些包时，默认会使用 pip 的官方源，所以经常会报网络超时失败。

常用的解决办法是，在安装包时，使用 `-i` 参数指定一个国内的镜像源。但是每次指定就很麻烦呀，还要打超长的一串字母。

这时候，其实可以将这个源写进 pip 的配置文件里。以后安装的时候，就默认从你配置的这个 源里安装了。

那怎么配置呢？文件文件在哪？

使用`win+r` 输入 `%APPDATA%` 进入用户资料文件夹，查看有没有一个 pip 的文件夹，若没有则创建之。

然后进入这个 文件夹，新建一个 `pip.ini` 的文件，内容如下

```
[global]
time-out=60
index-url=https://pypi.tuna.tsinghua.edu.cn/simple/
[install]
trusted-host=tsinghua.edu.cn

```

以上几乎包含了 pip 的所有使用场景，也许有不少用法你还没有用过，不过没关系，你只要收藏本文，等到要用的时候再来查阅即可。
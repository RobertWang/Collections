> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/33345935)

```
          _ __      _,.---._      .=-.-.
          .-`.' ,`.  ,-.' - ,  `.   /==/_ /
         /==/, -   \/==/ ,    -  \ |==|, |  
        |==| _ .=. |==| - .=.  ,  ||==|  |  
        |==| , '=',|==|  : ;=:  - ||==|- |  
        |==|-  '..'|==|,  '='  ,  ||==| ,|  
        |==|,  |    \==\ _   -    ;|==|- |  
        /==/ - |     '.='.  ,  ; -\/==/. /  
        `--`---'       `--`--'' `--`--`-`   

                        ---- by Hangfeng Yang

```

由于国内通过 pip 下载 python 包的速度真的很慢，特别是下载包文件比较大的情况下经常会导致下载失败，因此我自己写了一个 pip 终端换源工具 pqi，pqi 可以把默认的 PyPi 源迅速切换化为国内源 tuna, douban, aliyun, ustc 从而加快 python 包的安装速度 - [项目地址](https://link.zhihu.com/?target=https%3A//github.com/yhangf/PyQuickInstall)欢迎 star 和 fork，下图为提速效果展示。

![](https://pic1.zhimg.com/v2-f2d73f9c971de428355aab3b82599600_b.jpg)

**怎么使用 (兼容 py2/py3/linux/windows/MacOS)**

**1. 安装**

*   方法一（推荐）

```
>>> pip install pqi

```

*   方法二

```
>>> git clone https://github.com/yhangf/PyQuickInstall.git
>>> python3 setup.py install

```

**2. 命令行输入 `pqi` 回车**

```
>>> pqi
Usage:
  pqi ls
  pqi use <name>
  pqi show
  pqi add <name> <url>
  pqi remove <name>
  pqi (-h | --help)
  pqi (-v | --version)
Options:
  -h --help        Show this screen.
  -v --version     Show version.

```

*   列举所有支持的 pip 源

```
>>> pqi ls

```

*   改变 pip 源

```
>>> pqi use <name>

```

例子，比如运行 pqi use tuna 即把当前 pip 源改为清华的 pip 源

*   显示当前 pip 源

```
>>> pqi show

```

*   添加新的 pip 源 (如添加 USTC 源）

```
>>> pqi add ustc https://mirrors.ustc.edu.cn/pypi/web/simple

```

*   移除 pip 源（如官方 PyPi 源）

```
>>> pqi remove pypi

```
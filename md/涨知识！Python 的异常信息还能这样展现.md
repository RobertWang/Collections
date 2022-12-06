> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666566029&idx=2&sn=ee9c57dce9fdf5fd02d3f58f9e777552&chksm=80dc5526b7abdc302dcb75192b66288fa15e261acfd83fbfa98a9965fa1107b072b2a0999162&mpshare=1&scene=1&srcid=1021I9vCW2wSwsmxfQUBrSSS&sharer_sharetime=1666324601452&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

> 【导语】：在日常开发的过程中，当代码报错时，我们通常要不断打印、阅读 traceback 提示信息，来调试代码，这篇文章介绍了如何实现一个 Exception Hooks，使得 traceback 模块的提示信息更加精确；同时还介绍了一些第三方库，这些库也提供了 Exception Hooks 的功能。

【导语】：在日常开发的过程中，当代码报错时，我们通常要不断打印、阅读 traceback 提示信息，来调试代码，这篇文章介绍了如何实现一个 Exception Hooks，使得 traceback 模块的提示信息更加精确；同时还介绍了一些第三方库，这些库也提供了 Exception Hooks 的功能。

简介
--

在日常开发中，我们的大部分时间都会花在阅读 traceback 模块信息以及调试代码上。本文我们将改进 traceback 模块，让其中的提示信息更加简洁准确。

基于这一目的，我们将会自定义 Exception Hooks(异常处理钩子)，用来去除 traceback 中的冗余信息，只留下解决报错所需的内容。此外，我还会介绍一些好用的第三方库，你可以直接使用其中的 Exception Hooks，来简化 traceback 模块。

### Exception Hooks

假如程序的异常信息没有被 try/catch 捕获到，python 解释器就会调用 sys.excepthook() 函数，它会接收 3 个参数，分别是：`type`, `value`, `traceback`。这个函数也被称为 Exception Hook，会输出程序的异常信息。

我们来看看下面这个例子：

```
import sys

def exception_hook(exc_type, exc_value, tb):
    print('Traceback:')
    filename = tb.tb_frame.f_code.co_filename
    name = tb.tb_frame.f_code.co_name
    line_no = tb.tb_lineno
    print(f"File {filename} line {line_no}, in {name}")

    # Exception type 和 value
    print(f"{exc_type.__name__}, Message: {exc_value}")

sys.excepthook = exception_hook

```

在这个例子中，我们可以从`traceback (tb)`对象中获取到异常信息出现的位置，位置信息包括：文件名 (`f_code.co_filename`), 函数 / 模块名 (`f_code.co_name`), 和行数 (`tb_lineno`)。此外，我们可以使用 exc_type 和 exc_value 变量来获取异常信息的内容。

当我们调用一个会产生错误的函数时，exception_hook 会输出如下内容：

```
def do_stuff():
    # 写一段会产生异常的代码
    raise ValueError("Some error message")

do_stuff()

# Traceback:
# File /home/some/path/exception_hooks.py line 22, in <module>
# ValueError, Message: Some error message

```

上述例子提供了一部分异常信息，但要想获取调试代码所需的全部信息，并知道异常出现的时间及位置，我们还需要深入研究下 traceback 对象：

```
def exception_hook(exc_type, exc_value, tb):

    local_vars = {}
    while tb:
        filename = tb.tb_frame.f_code.co_filename
        name = tb.tb_frame.f_code.co_name
        line_no = tb.tb_lineno
        print(f"File {filename} line {line_no}, in {name}")

        local_vars = tb.tb_frame.f_locals
        tb = tb.tb_next
    print(f"Local variables in top frame: {local_vars}")

...

# File /home/some/path/exception_hooks.py line 41, in <module>
# File /home/some/path/exception_hooks.py line 7, in do_stuff
# Local variables in top frame: {'some_var': 'data'}

```

由上面的例子可以看出，traceback 对象 (tb) 本质上是一个链表 - 存储着所有出现的 exceptions。因此可以使用 tb_next 对 tb 进行遍历，并打印每一个异常的信息。在此基础上，还可以使用 tb_frame.f_locals 属性将变量输出到 console 中，这有助于调试代码。

使用 traceback 对象输出异常信息是可行的，但是比较麻烦，此外输出的信息可读性较差。更方便的做法是使用 traceback 模块，该模块内置了许多提取异常信息的辅助函数。

目前我们已经介绍了 Exception Hooks 的基础知识，接下来我们可以自定义一个 exception hooks，并加入一些实用的特性。

### 自定义 Exception Hooks

1.  我们还可以让异常信息自动存入文件中，在之后调试代码的时候查看：
    

```
LOG_FILE_PATH = "./some.log"
FILE = open(LOG_FILE_PATH, mode="w")

def exception_hook(exc_type, exc_value, tb):
    FILE.write("*** Exception: ***\n")
    traceback.print_exc(file=FILE)

    FILE.write("\n*** Traceback: ***\n")
    traceback.print_tb(tb, file=FILE)

# *** Exception: ***
# NoneType: None
#
# *** Traceback: ***
#   File "/home/some/path/exception_hooks.py", line 82, in <module>
#     do_stuff()
#   File "/home/some/path/exception_hooks.py", line 7, in do_stuff
#     raise ValueError("Some error message")

```

2.  异常信息默认会存储到 stderr 中，如果你想改变存储位置，可以这样做：
    

```
import logging
logging.basicConfig(
    level=logging.CRITICAL,
    format='[%(asctime)s] {%(pathname)s:%(lineno)d} %(levelname)s - %(message)s',
    datefmt='%H:%M:%S',
    stream=sys.stdout
)

def exception_hook(exc_type, exc_value, exc_traceback):
    logging.critical("Uncaught exception:", exc_info=(exc_type, exc_value, exc_traceback))

# [17:28:33] {/home/some/path/exception_hooks.py:117} CRITICAL - Uncaught exception:
# Traceback (most recent call last):
#   File "/home/some/path/exception_hooks.py", line 122, in <module>
#     do_stuff()
#   File "/home/some/path/exception_hooks.py", line 7, in do_stuff
#     raise ValueError("Some error message")
# ValueError: Some error message

```

3.  我们还可以给提示信息的某一部分设置颜色：
    

```
# pip install colorama
from colorama import init, Fore
init(autoreset=True)  # Reset the color after every print

def exception_hook(exc_type, exc_value, tb):

    local_vars = {}
    while tb:
        filename = tb.tb_frame.f_code.co_filename
        name = tb.tb_frame.f_code.co_name
        line_no = tb.tb_lineno
        # Prepend desired color (e.g. RED) to line
        print(f"{Fore.RED}File {filename} line {line_no}, in {name}")

        local_vars = tb.tb_frame.f_locals
        tb = tb.tb_next
    print(f"{Fore.GREEN}Local variables in top frame: {local_vars}")

```

除了上面介绍的例子，你还可以输出每一帧的局部变量，或者找到出现异常的行中所引用的变量。这些 Exception Hooks 已经很成熟了，相比于自定义 Exception hooks，我建议你阅读下其他开发者的源码，学习下他们的设计思路。

*   输出每一帧的局部变量 [1]
    
*   找到出现异常的行中所引用的变量 [2]
    

### 第三方库中的 Exception Hooks

自定义一个 Exception Hook 很有趣，但许多第三方库已经实现这一功能了。与其自己造轮子，不如看看其他优秀的工具。

1.  首先，我个人最喜欢的是`Rich`, 可以直接用 pip 进行安装，随后导入使用。如果你只想在一个例子中使用，可以这样做：`python -m rich.traceback`
    

```
# https://rich.readthedocs.io/en/latest/traceback.html
# pip install rich
# python -m rich.traceback

from rich.traceback import install
install(show_locals=True)

do_stuff()  # Raises ValueError

```

2.  `better_exceptions`也很受欢迎，我们需要先设置环境变量 BETTER_EXCEPTIONS=1，再用 pip 安装。此外，如果你的 TERM 变量不是 xterm，还要把 SUPPORTS_COLOR 设置为 True。
    

```
# https://github.com/Qix-/better-exceptions
# pip install better_exceptions
# export BETTER_EXCEPTIONS=1

import better_exceptions
better_exceptions.MAX_LENGTH = None
# 检查你的 TERM 变量是否被设置为 `xterm`, 如果没有执行以下操作
# See issue: https://github.com/Qix-/better-exceptions/issues/8
better_exceptions.SUPPORTS_COLOR = True
better_exceptions.hook()

do_stuff()  # Raises ValueError

```

3.  使用最方便的库是`pretty_errors`，只需导入即可：
    

```
# https://github.com/onelivesleft/PrettyErrors/
# pip install pretty_errors

import pretty_errors
# 如果你对默认配置满意的话，则无需修改
pretty_errors.configure(
    filename_display    = pretty_errors.FILENAME_EXTENDED,
    line_number_first   = True,
    display_link        = True,
    line_color          = pretty_errors.RED + '> ' + pretty_errors.default_config.line_color,
    code_color          = '  ' + pretty_errors.default_config.line_color,
    truncate_code       = True,
    display_locals      = True
)

do_stuff()

```

除了直接导入外，上面的代码还显示了该库的一些可选配置。更多的配置可以查看这里：配置 [3]

4.  IPython 的`ultratb`模块
    

```
# https://ipython.readthedocs.io/en/stable/api/generated/IPython.core.ultratb.html
# pip install ipython
import IPython.core.ultratb

# Also ColorTB, FormattedTB, ListTB, SyntaxTB
sys.excepthook = IPython.core.ultratb.VerboseTB(color_scheme='Linux')  # Other colors: NoColor, LightBG, Neutral

do_stuff()

```

5.  `stackprinter`库
    

```
# https://github.com/cknd/stackprinter
# pip install stackprinter
import stackprinter
stackprinter.set_excepthook(style='darkbg2')

do_stuff()

```

结论
--

本文我们学习了如何自定义 Exception Hooks, 但我更推荐使用第三方库。你可以在本文介绍的第三方库中任选一个喜欢的，用到项目中。需要注意的是使用自定义 Exception Hooks 可能会丢失某些关键信息，例如：本文中的某些例子中，输出中缺少文件路径，在远程调试代码这无疑很不方便，因此，需要谨慎使用。

### 参考资料

[1]

输出每一帧的局部变量: _https://github.com/Infinidat/infi.traceback/blob/develop/src/infi/traceback/**init**.py_

[2]

找到出现异常的行中所引用的变量: _https://github.com/albertz/py_better_exchook/blob/master/better_exchook.py_

[3]

配置: _https://github.com/onelivesleft/PrettyErrors/#configuration-settings_

[4]

参考原文: _https://martinheinz.dev/blog/66_

- EOF -
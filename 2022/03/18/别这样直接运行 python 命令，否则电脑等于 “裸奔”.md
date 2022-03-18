> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIzMzMzOTI3Nw==&mid=2247505843&idx=1&sn=646dbaf127eb082fe1073b72743baf43&chksm=e885b751dff23e47a60b6a78cb4663805ff298ab541e045a51857a7863e8190d9e1cd90c36af&mpshare=1&scene=1&srcid=0315cZAvtbXmj5dAztEQfSO8&sharer_sharetime=1647322883063&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

##### 来源：量子位

而且 Python 语言很容易上手模块。比如你编写了一个模块 _my_lib.py_，只需在调用这个模块的程序中加入一行 _import my_lib_ 即可。

别再图方便了
------

为何这样做会有危险？首先，我们要了解 Python 程序安全运行需要满足的三个条件：

1.  系统路径上的每个条目都处于安全的位置；
    
2.  “主脚本” 所在的目录始终位于系统路径中；
    
3.  若 python 命令使用 - c 和 - m 选项，调用程序的目录也必须是安全的。
    

如果你运行的是正确安装的 Python，那么 Python 安装目录和 virtualenv 之外唯一会自动添加到系统路径的位置，就是当前主程序的安装目录。

![](https://mmbiz.qpic.cn/mmbiz_jpg/YicUhk5aAGtBnTPps2GpGRmNVuHhcRQbEtkKfPVHqVfS9M8BDHm4DFZF2icYyv9sbJcwicnMVHQBNdGYLGUz6orVQ/640?wx_fmt=jpeg)

这就是安全隐患的来源，下面用一个实例告诉你为什么。

如果你把 pip 安装在 _/usr/bin_ 文件夹下，并运行 pip 命令。由于 _/usr/bin_ 是系统路径，因此这是一个非常安全的地方。

但是，有些人并不喜欢直接使用 pip，而是更喜欢调用 _/path/to/python -m pip_。

这样做的好处是可以避免环境变量 _$PATH_ 设置的复杂性，而且对于 Windows 用户来说，也可以避免处理安装各种 exe 脚本和文档。

所以问题就来了，如果你的下载文件中有一个叫做 **pip.py** 的文件，那么你将它将取代系统自带的 pip，接管你的程序。

下载文件夹并不安全
---------

比如你不是从 PyPI，而是直接从网上直接下载了一个 Python wheel 文件。你很自然地输入以下命令来安装它：

```
~$ cd Downloads
~/Downloads$ python -m pip install ./totally-legit-package.whl
```

这似乎是一件很合理的事情。但你不知道的是，这么操作很有可能访问带有 **XSS JavaScript** 的站点，并将带有恶意软件的的 _pip.py_ 到下载文件夹中。

下面是一个恶意攻击软件的演示实例：

```
~$ mkdir attacker_dir
~$ cd attacker_dir
~/attacker_dir$ echo 'print("lol ur pwnt")' > pip.py
~/attacker_dir$ python -m pip install requests
lol ur pwnt
```

看到了吗？这段代码生成了一个 _pip.py_，并且代替系统的 pip 接管了程序。

设置 $PYTHONPATH 也不安全
-------------------

前面已经说过，Python 只会调用系统路径、virtualenv 虚拟环境路径以及当前主程序路径

你也许会说，那我手动设置一下 _$PYTHONPATH_ 环境变量，不把当前目录放在环境变量里，这样不就安全了吗？

非也！不幸的是，你可能会遭遇另一种攻击方式。下面让我们模拟一个 “脆弱的”Python 程序：

```
# tool.py
try:
    import optional_extra
except ImportError:
    print("extra not found, that's fine")
```

然后创建 2 个目录：_install_dir_ 和 _attacker_dir_。将上面的程序放在 install_dir 中。然后 _cd attacker_dir_ 将复杂的恶意软件放在这里，并把它的名字改成 _tool.py_ 调用的 _optional_extra_ 模块：

```
# optional_extra.py
print("lol ur pwnt")
```

我们运行一下它：

```
~/attacker_dir$ python ../install_dir/tool.py
extra not found, that's fine
```

到这里还很好，没有出现任何问题。

但是这个习惯用法有一个严重的缺陷：第一次调用它时，如果 $PYTHONPATH 以前是空的或者未设置，那么它会包含一个空字符串，该字符串被解析为当前目录。

让我们再尝试一下：

```
~/attacker_dir$ export PYTHONPATH="/a/perfectly/safe/place:$PYTHONPATH";
~/attacker_dir$ python ../install_dir/tool.py
lol ur pwnt
```

看到了吗？恶意脚本接管了程序。

为了安全起见，你可能会认为，清空 _$PYTHONPATH_ 总该没问题了吧？Naive！还是不安全！

```
~/attacker_dir$ export PYTHONPATH="";
~/attacker_dir$ python ../install_dir/tool.py
lol ur pwnt
```

设置 $PYTHONPATH 曾经是设置 Python 开发环境的最常用方法。但你以后最好别再用它了，virtualenv 可以更好地满足开发者需求。如果你过去设置了一个 $PYTHONPATH，现在是很好的机会，把它删除了吧。

如果你确实需要在 shell 中使用 PYTHONPATH，请用以下方法：

```
export PYTHONPATH="${PYTHONPATH:+${PYTHONPATH}:}new_entry_1"
export PYTHONPATH="${PYTHONPATH:+${PYTHONPATH}:}new_entry_2"
```

在 bash 和 zsh 中，$PYTHONPATH 变量的值会变成：

```
$ echo "${PYTHONPATH}"
new_entry_1:new_entry_2
```

如此便保证了环境变量 $PYTHONPATH 中没有空格和多余的冒号。

如果你仍在使用 $PYTHONPATH，请确保始终使用**绝对路径**！

另外，在下载文件夹中直接运行 Jupyter Notebook 也是一样危险的，比如 _jupyter notebook ~/Downloads/anything.ipynb_ 也有可能将恶意程序引入到代码中。

预防措施
----

最后总结一下要点。

1.  如果要在下载文件夹~/Downloads 中使用 Python 编写的工具，请养成良好习惯，使用 pip 所在路径 / path/to/venv/bin/pip，而不是输入 / path/to/venv/bin/python -m pip。
    
2.  避免将~/Downloads 作为当前工作目录，并在启动之前将要使用的任何软件移至更合适的位置。
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/YicUhk5aAGtBnTPps2GpGRmNVuHhcRQbEKv6p0HphkYP3HHJooDokDZzlwqiaLURyNiarfysP5AYI1C4AExkQPOfw/640?wx_fmt=jpeg)

了解 Python 从何处获取执行代码非常重要。赋予其他人执行任意 Python 命令的能力等同于赋予他对你电脑的完全控制权！  

希望以上文字对初学 Python 的你有所帮助。

_原文链接：  
__https://glyph.twistedmatrix.com/2020/08/never-run-python-in-your-downloads-folder.html_
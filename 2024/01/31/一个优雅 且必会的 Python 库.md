> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI4MzMyNjQwMw==&mid=2247486082&idx=1&sn=49bb26120c79289f9031ae9a608f4705&chksm=eb8d282edcfaa1389443a3df285903742414c82b0b1b56f50039c724c899c0e7693d725e0254&mpshare=1&scene=1&srcid=0130iHPLfD65lKd5a4yZHyO0&sharer_shareinfo=a0f931e6d333620d5d5744e46901f2f3&sharer_shareinfo_first=a0f931e6d333620d5d5744e46901f2f3#rd)

你正在为你的 Python 项目编写一个命令行接口。可能是一个数据处理工具，也可能是游戏的启动器，又或者是部署脚本。

你希望用户能通过简明的参数和选项来控制程序的行为。比如，用户可以通过 `--help` 选项来查看帮助信息，通过 `-v` 来获取版本信息，或者通过 `--input=file.txt` 来指定输入文件。

你可能已经试过如何实现了，然后令人沮丧地发现，解析复杂的命令行参数是一项乏味且容易出错的工作。

正当你在代码中苦苦挣扎时，**docopt** 悄悄地向你招手，帮你轻松构建一个强大、直观且易于维护的命令行程序。

### 什么是 docopt

**docopt** 不是那些典型的参数解析库，它基于一个简单的前提：如果你可以写出一个良好的帮助信息，那么你已经定义了程序的界面。简单地说，**docopt** 允许你直接将帮助文档字符串作为定义参数的界面。它帮助你定义一个接口规范，并且能够自动解析与这个规范相符的命令行参数。

开发者 Vladimir Keleshev 在 2012 年推出了 docopt，并开源在 GitHub 上。随后，有越来越多的开发者参与到这个项目中，使其成为最受欢迎的 Python 命令行解析工具之一。docopt 简化了命令行处理流程，将我们从繁琐的参数解析代码中解放出来。

与它类似的库有 `argparse` 和`click`，但 docopt 的独特之处在于它的声明式界面设计。你直接写出命令行应该如何被调用的文档，然后 docopt 为你解析出所有的选项和参数。这种接口设计方法是直观且易于记忆的，减少了学习成本，让开发者可以更专注于其他更重要的工作。

### 安装

安装 docopt 十分简单，只需要一行 pip 命令：

```
pip install docopt


```

### 命令行参数格式化

要理解 **docopt** 的运用方法，我们首先要理解命令行参数的结构。

**docopt** 的一个重要概念是**形式语法**（格式），它允许使用下述的几种模式：

*   尖括号 `<like_this>` 用于参数，这些是必须由用户提供的值。
    
*   方括号 `[like_this]` 用于可选的元素。
    
*   大括号 `{like_this}` 基本上不使用，但你可以用它们来构成自己的语法标记。
    
*   圆括号 `(like_this)` 用于表示一组选项中必须选择一个。
    
*   管道符 `|` 用于分割可供选择的各个选项。
    
*   双破折号 `--like_this` 通常用于长选项式样的命令行选项。
    
*   单破折号 `-l` 通常用于单字母的命令行选项。
    
*   点点点 `...` 表示可以重复的元素。
    

通过以上这些格式标记，你可以创建几乎任何你需要的命令行界面。

下面我们看看，如何定义和解析参数。

### 解析命令行参数

首先，定义一个简单的帮助文档作为字符串，这也将是我们程序的命令行接口规范。

例子如下：

```
"""Naval Fate.

Usage:
  naval_fate.py ship new <name>...
  naval_fate.py ship <name> move <x> <y> [--speed=<kn>]
  naval_fate.py ship shoot <x> <y>
  naval_fate.py mine (set|remove) <x> <y> [--moored|--drifting]
  naval_fate.py (-h | --help)
  naval_fate.py --version

Options:
  -h --help     Show this screen.
  --version     Show version.
  --speed=<kn>  Speed in knots [default: 10].
  --moored      Moored (anchored) mine.
  --drifting    Drifting mine.
"""

from docopt import docopt

if __name__ == '__main__':
    arguments = docopt(__doc__, version='Naval Fate 2.0')
    print(arguments)


```

当你运行这个脚本时，你可以像下面这样输入命令行参数：

```
python naval_fate.py ship new "Black Pearl" "Flying Dutchman"


```

你会看到程序输出了解析后的参数，例如：

```
{"ship": true,
 "new": true,
 "<name>": ["Black Pearl", "Flying Dutchman"],
 "move": false, ...}


```

每个命令行选项和参数都会得到一个字典条目。值为 `true` 或`false` 代表布尔型选项，其他如 `<name>` 和`--speed` 则包含传递的值或默认值。

### 子命令

docopt 非常适用于设计具有子命令的 CLI 程序，比如 git 那样有许多子命令的程序。

示例：

```
"""Git.

Usage:
  git.py add <file>...
  git.py commit <message>
  git.py push <remote> <branch>
"""

arguments = docopt(__doc__)


```

如果某个用户尝试执行以下命令：

```
python git.py commit "Initial commit"


```

解析后的参数将是：

```
{"add": false,
 "commit": true,
 "<message>": "Initial commit",
 "push": false, ...}


```

在你的程序中，你可以根据这些参数执行相应的逻辑，如提交代码、添加文件等。

### 其他功能

docopt 甚至支持更高级的模式匹配，比如选择 (one-of)、可选项 (optional)、重复项和可能的细分 (arguments, commands, options, ...)。比如，`(set|remove)` 意味着这是一个选择模式，必须匹配 `set` 或`remove`。

详细信息请参考项目说明：**https://github.com/docopt/docopt**，限于篇幅，这里就不展开了。

### 实践

现在你已经了解了 docopt 的基础知识，让我们来点互动性的练习吧：

1.  尝试编写一个帮助文档字符串，用于一个简单的文件复制工具，该工具接受源文件和目标文件的路径作为参数。
    
2.  增加一个可选的 `--verbose` 选项，当用户希望工具提供更多的执行详情时可以使用这个选项。
    
3.  实现源文件路径可以是单个文件也可以是多个文件（使用 `...`）。
    
4.  为工具添加一个版本号 `1.0.0` 并支持 `--version` 选项。
    

练习程度从易到难，希望通过这些练习，你能更好地理解并应用 docopt。记得，练习时可以参考 docopt 的官方文档，以获取更多灵感和解决方案。

### 总结

在我们的 Python 编程旅程中，`docopt` 是一个强大而灵活的工具，它使命令行参数的处理简洁而优雅。通过直接解析帮助信息来确定命令行参数，我们可以更快地创建一个用户友好的命令行界面，这样不仅节省了我们宝贵的时间，也为最终用户提供了更佳的体验。而当你的项目成长，命令行界面变得日益庞大时，docopt 的简明性和可维护性尤为宝贵。

随着你继续探索 Python 的深处，希望你能善用 docopt，让它成为你工具箱中的瑰宝。让编程像艺术一样流畅而富有创意。
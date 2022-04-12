> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/90203080)

> 文章首发微信公众号，微信搜索：猿说 python

学习过 C 语言或者 Java 语言的盆友应该都知道程序运行必然有主程序入口 main 函数，而 python 却不同，即便没有主程序入口，程序一样可以自上而下对代码块依次运行，然后 python 不少开源项目或者模块中依然存在 __name__ == “__main__” 这种写法，具体是上面意思呢？

![](https://pic3.zhimg.com/v2-ba7f4b05f01f172072538ae55fb82266_b.jpg)

### 一. 语义解释

### 1.__name__

**__name__** 是 python 的内置属性，是系统全局变量！每一个 py 文件都有一个属于自己的__name__：

**如果 py 文件作为模块被导入 (import)，那么__name__就是该 py 文件的文件名 (也称 模块名)；**

**如果 py 文件直接运行时 (Ctrl+Shift+F10)，那么__name__默认等于字符串”__main__”;**

举个简单的例子：假如你名字是张三，在朋友眼中，你是张三`(__name__ == '张三')`；在你自己眼中，你是你自己`(__name__ == '__main__')`

![](https://pic4.zhimg.com/v2-495db65738e4eda652abcca57ff62aeb_b.jpg)

### 2.”__main__”

**“_ _main_ _”** 实际上就是一个字符串，用来鉴别程序入口，没有太多花里胡哨的东西.

### 3.__name__ == “__main__”

当. py 文件被直接运行 (Ctrl+Shift+F10) 时， **if** __name__ == “__main__” 之下的代码块将被运行，该语句就相当与 python 的 main 主函数入口，示例代码如下：

a. 新建一个 my_name.py 文件，作为模块文件：

```
# !usr/bin/env python
# -*- coding:utf-8 _*-
"""
@Author:何以解忧
@Blog(个人博客地址): https://www.codersrc.com
@Github:www.github.com
 
@File:my_name.py
@Time:2019/10/14 22:02
 
@Motto:不积跬步无以至千里，不积小流无以成江海，程序人生的精彩需要坚持不懈地积累！
"""
 
# 定义一个函数并打印 __name__
def prit_name():
 print("my_name.py __name__:", __name__)
 
if __name__ == "__main__":
 prit_name()

```

b. 新建一个 python_main.py 文件，**作为启动文件 (Ctrl+Shift+F10)**：

```
# 导入 my_name 模块
import my_name
 
# 定义一个函数并打印 __name__
def prit_name():
 my_name.prit_name()
 print("python_main.py __name__:", __name__)
 
if __name__ == "__main__":
 prit_name()

```

输出结果：

```
my_name.py __name__: my_name
python_main.py __name__: __main__

```

由此可见，作为启动文件 python_main.py ，该文件的内置属性 __name__ 等于 “__main__”，而 my_name.py 作为导入模块，该模块的 __name__ 等于文件名 (也称模块名字)，所以 my_name.py 中的 表达式 if __name__ == “__main__” 并不成立！

当直接将 my_name.py 作为启动文件时 (Ctrl+Shift+F10)，输出：

```
my_name.py __name__: __main__

```

**如果 py 文件作为模块被导入 (import)，那么__name__就是该 py 文件的文件名 (也称 模块名)；**

**如果 py 文件直接运行时 (Ctrl+Shift+F10)，那么__name__默认等于字符串”__main__”;**

![](https://pic4.zhimg.com/v2-45de7af6ac31aacf1120fe3b642d620b_b.jpg)

### 二. 作用

1.__name__ == “__main__” 作为启动 py 文件的 main 函数入口

2. 一个项目中必然会包含多个模块文件，每个模块文件在自己写完代码之后会做一些简单的测试用于检测 bug 或者 对自己的函数调用写一个简单的示例，而恰到好处的是：__name__ == “__main__” 既不会影响你的测试代码，也不会影响别人调用你的接口函数。

![](https://pic3.zhimg.com/v2-1de9a82e6feb0d3b5279383a2622c732_b.jpg)

**猜你喜欢:**
---------

[1.python 模块](https://link.zhihu.com/?target=https%3A//www.codersrc.com)

[2.python 异常处理](https://link.zhihu.com/?target=https%3A//www.codersrc.com)

[3.python return 逻辑运算表达式](https://link.zhihu.com/?target=https%3A//www.codersrc.com)

[4.python 字典推导式](https://link.zhihu.com/?target=https%3A//www.codersrc.com)

[5.python 列表推导式](https://link.zhihu.com/?target=https%3A//www.codersrc.com)

**转载请注明**：[猿说 Python](https://link.zhihu.com/?target=https%3A//www.codersrc.com) » [python __name__ == “__main__” 详细解释](https://link.zhihu.com/?target=https%3A//www.codersrc.com)
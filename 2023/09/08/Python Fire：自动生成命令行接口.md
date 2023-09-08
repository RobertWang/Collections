> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Rp7yghuqujeTnzaV4K9xgw)

> 命令行程序是平时写一些小工具时最常用的方式。为了让命令行程序更加灵活，我们常常会设置一些参数，根据参数让程序

命令行程序是平时写一些小工具时最常用的方式。

为了让命令行程序更加灵活，我们常常会设置一些参数，根据参数让程序执行不同的功能。  
这样就不用频繁的修改代码来执行不同的功能。

随着命令行程序功能的丰富，也就是参数多了以后，解析和管理参数之间的关系会变得越来越繁重。  
而本次介绍的 `Fire` 库，正好可以解决这个问题。  
使用 `Fire` 库，我们只要关心具体功能的实现，最后`Fire`会帮助我们自动把所有功能组织成一个命令行程序。

`Fire`库在 github 上的地址：https://github.com/google/python-fire

1. 一般命令
=======

一般的命令，也就是带有几个参数的一段程序，比如：

```
# -*- coding:utf-8 -*-

def import_file(fp):
    print("import file from: {}".format(fp))

def export_file(fp):
    print("EXPORT file to: {}".format(fp))


```

这是模拟文件导出功能的两个函数。

使用 `Fire` 转换成命令行程序非常简单，下面介绍几种常用的方式。

1.1. 默认方式
---------

```
# -*- coding:utf-8 -*-

import fire

def import_file(fp):
    print("IMPORT file from: {}".format(fp))

def export_file(fp):
    print("EXPORT file to: {}".format(fp))

if __name__ == "__main__":
    # fire默认会将所有函数转换成子命令
    fire.Fire()


```

然后，就可以通过子命令的方式执行导入导出功能。

```
$ python main.py import_file --fp ./path/xxx.csv
IMPORT file from: ./path/xxx.csv

$ python main.py export_file --fp ./path/xxx.csv
EXPORT file to: ./path/xxx.csv


```

**「函数的名称」**自动变为**「子命令的名称」**，**「函数的参数」**自动变成**「子命令的参数」**。

1.2. Fire<fn>
-------------

`Fire`库的默认方式会把所有的函数都转换为子命令，  
如果只想导出一个函数的话，可以用 `Fire<fn>`的方式。

```
if __name__ == "__main__":
    # 只导出 import_file 函数作为子命令
    fire.Fire(import_file)


```

只导出一个函数的时候，执行命令的时候不需要输入子命令的名称。

```
$ python main.py --fp ./path/xxx.csv
IMPORT file from: ./path/xxx.csv


```

1.3. Fire<dict>
---------------

导出多个函数作为子命令时，默认是使用函数名作为子命令名称的，函数名称有时候会非常长，输入很麻烦。  
这时，可以用 `Fire<dict>` 方式。

```
if __name__ == "__main__":
    # 子命令的名称分别是：import 和 export
    fire.Fire({
        "import": import_file,
        "export": export_file,
    })


```

执行时，使用简化的子命令名称。

```
$ python main.py import --fp ./path/xxx.csv
IMPORT file from: ./path/xxx.csv

$ python main.py export --fp ./path/xxx.csv
EXPORT file to: ./path/xxx.csv


```

这种方式非常灵活，不仅可以设置子命令名称，还可以控制需要导出哪些函数。

1.4. Fire<object>
-----------------

除了导出函数，`Fire<object>`方式也可以导出对象的公有方法。

```
import fire

class FileHandler():

    def __init__(self):
        pass

    def import_file(self, fp):
        print("IMPORT file from: {}".format(fp))

    def export_file(self, fp):
        print("EXPORT file to: {}".format(fp))

if __name__ == "__main__":
    fh = FileHandler()
    fire.Fire(fh)


```

使用方式如下：

```
$ python main.py import_file --fp ./path/xxx.csv
IMPORT file from: ./path/xxx.csv

$ python main.py export_file --fp ./path/xxx.csv
EXPORT file to: ./path/xxx.csv


```

使用对象的方式没有直接使用函数那么简单，但有个好处是可以在初始化时传入一些状态。

```
import fire
import os

class FileHandler():

    def __init__(self, folder=""):
        self.folder = folder

    def import_file(self, fp):
        print("IMPORT file from: {}".format(os.path.join(self.folder, fp)))

    def export_file(self, fp):
        print("EXPORT file to: {}".format(os.path.join(self.folder, fp)))

if __name__ == "__main__":
    # 设置了默认文件夹，使用时直接传入文件名即可
    fh = FileHandler("./default_path")
    fire.Fire(fh)


```

```
$ python main.py import_file --fp xxx.csv
IMPORT file from: ./default_path/xxx.csv

$ python main.py export_file --fp xxx.csv
EXPORT file to: ./default_path/xxx.csv


```

1.5. Fire<class>
----------------

`Fire<class>`的方式也可以直接作用在类上，不用初始化对象。

```
if __name__ == "__main__":
    fire.Fire(FileHandler)


```

和 `Fire<object>` 不同的是，`__init__`方法的参数也变成了命令的参数，也可以在命令行中调整了。

```
$ python main.py import_file --fp xxx.csv
IMPORT file from: xxx.csv

$ python main.py import_file --fp xxx.csv --folder ./my_folder
IMPORT file from: ./my_folder/xxx.csv



```

2. 组合命令
=======

当功能越来越多时，可能就会需要组合一些功能一起运行，省得输入一个一个的子命令。

```
class FileHandler():

    def __init__(self, folder="./defalut_dir"):
        self.folder = folder

    def import_file(self, fp):
        print("IMPORT file from: {}".format(os.path.join(self.folder, fp)))

    def export_file(self, fp):
        print("EXPORT file to: {}".format(os.path.join(self.folder, fp)))

class DatabaseHandler():

    def __init__(self, src="aliyun-mysql", dst="tecent-mysql"):
        self.src = src
        self.dst = dst

    def import_db(self):
        print("IMPORT data from: {} to: {}".format(self.src, self.dst))

    def export_db(self):
        print("EXPORT data from: {} to: {}".format(self.src, self.dst))

# 组合 FileHandler 和 DatabaseHandler
class ComposeHandler():
    def __init__(self):
        self.fh = FileHandler()
        self.dh = DatabaseHandler()

    def import_all(self, fp):
        self.fh.import_file(fp)
        self.dh.import_db()


    def export_all(self, fp):
        self.fh.export_file(fp)
        self.dh.export_db()

if __name__ == "__main__":
    fire.Fire(ComposeHandler)



```

导出组合命令之后，不仅可以执行组合命令，也可以只执行子命令。

```
$ python main.py import_all --fp xxx.csv
IMPORT file from: ./defalut_dir/xxx.csv
IMPORT data from: aliyun-mysql to: tecent-mysql

$ python main.py export_all --fp xxx.csv
EXPORT file to: ./defalut_dir/xxx.csv
EXPORT data from: aliyun-mysql to: tecent-mysql

$ python main.py fh export_file --fp xxx.csv
EXPORT file to: ./defalut_dir/xxx.csv

$ python main.py dh export_db
EXPORT data from: aliyun-mysql to: tecent-mysql


```

3. 链式命令
=======

链式命令和组合命令不一样的地方在于：  
组合命令中，每个命令之间一般是相互独立的，  
而链式命令中，上一个命令的执行结果会对下一个命令造成影响。  
比如：

```
class Stat():
    def __init__(self):
        self.total = 0
        self.avg = 0
        self.n = 0

    # 模拟统计合计值
    def sum(self, n):
        self.n += n
        for i in range(n):
            self.total += i

        return self

    # 模拟求平均值
    def average(self):
        if self.n == 0:
            self.avg = 0
        else:
            self.avg = self.total / self.n

        return self

    # 显示分析结果
    def show(self):
        print("SUM: {}, and AVERAGE: {}".format(self.total, self.avg))

if __name__ == "__main__":
    fire.Fire(Stat)


```

执行链式命令时，可以先求和，再求平均值，最后显示结果：

```
$ python main.py sum 10 average show
SUM: 45, and AVERAGE: 4.5


```

因为是链式命令，所以可以多次执行：

```
$ python main.py sum 10 sum 10 average show
SUM: 90, and AVERAGE: 4.5

$ python main.py sum 10 sum 20 sum 30 average show
SUM: 670, and AVERAGE: 11.166666666666666



```

4. 复杂命令参数
=========

上面的示例中，参数都是简单的数据类型，比如字符串，数字之类的。

最后，介绍下复杂的命令参数如何使用，所谓复杂的参数，就是元组，列表，字典等等。

```
def hello(data):
    tp = type(data).__name__
    if  tp == "tuple" or tp == "list":
        for item in data:
            print("hello: {}".format(item))

    if tp == "dict":
        for k, v in data.items():
            print("hello: key {}, val {}".format(k, v))

if __name__ == "__main__":
    fire.Fire(hello)


```

`python`是弱类型语言，函数的参数可以是任何类型。  
主要看看命令行中如何传入复杂的类型：

```
$ python main.py "(aa, bb, cc)"
hello: aa
hello: bb
hello: cc

$ python main.py "[aa, bb, cc]"
hello: aa
hello: bb
hello: cc

$ python main.py "{aa: 11, bb: 22}"
hello: key aa, val 11
hello: key bb, val 22


```

5. 总结
=====

`Python`的`Fire`库是一个构思非常巧妙的命令行接口库，各种语言的命令行接口我接触过不少，还没有在其他编程语言中看到过类似的库。

`Fire`库最方便的地方在于，你不用再关心命令行的参数（参数的解析一直是命令行程序中最让人头疼的地方），只要专注于自己要实现的功能。  
此外，如果你已经有一些**「python 脚本」**的话，通过这个库把它们改造成命令行程序也非常简单。
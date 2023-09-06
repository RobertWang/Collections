> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/-j9n7PycP_PxJ4WjVjnt8g)

> 多一句没有，少一句不行，用最短的时间，教会你有用的技术。

> ❝
> 
> 多一句没有，少一句不行，用最短的时间，教会你有用的技术。
> 
> ❞

sys.argv 模块
-----------

最常见的 Python 脚本传入命令行参数的方式，也是最简单（粗暴）的方式。

```
#test_01.py
import sys

def test_for_sys(year, name, body):
    print('the year is', year)
    print('the name is', name)
    print('the body is', body)

if __name__ == '__main__':
    try:
        year, name, body = sys.argv[1:4]
        test_for_sys(year, name, body)
        print(sys.argv)
    except Exception as e:
        print(sys.argv)
        print(e)
#  python test_01.py  1 2 3 4
#['test_10.py', '1', '2', '3']
# sys.argv 是命令行参数列表
# len(sys.argv) 是命令行参数个数
# sys.argv[0] 表示脚本名。


```

**「注意：」**`sys.argv` 形式传入参数的方式比较简单，但是也很死板，因为传入的参数是一个有序的列表，所以在命令行中必须按照脚本规定的顺序去输入参数，这种方法比较适合脚本中需要的参数个数很少且参数固定的脚本。

click 库
-------

Click 是一个利用很少的代码以可组合的方式创造优雅命令行工具接口的 Python 库。它是高度可配置的，但却有合理默认值的 “命令行接口创建工具”。

```
# 2023/5/27 13:06 , @File: test_09.py, @Comment : python3.9.7
import click

@click.command()
@click.option('--count', default=1, help='Number of greetings.')
@click.option('--name', prompt='Your name',
              help='The person to greet.')
def hello(count, name):
    """Simple program that greets NAME for a total of COUNT times."""
    for x in range(count):
        click.echo('Hello %s!' % name)


if __name__ == '__main__':
    hello()
    
# python test_09.py --count 2 
# Your name: milong
# Hello milong!
# Hello milong!


```

argparse 模块
-----------

这个是今天咱们要讲的重点哦。

### 概念介绍

> ❝
> 
> The argparse module makes it easy to write user-friendly command-line interfaces. The program defines what arguments it requires, and argparse will figure out how to parse those out of sys.argv. The argparse module also automatically generates help and usage messages. The module will also issue errors when users give the program invalid arguments.
> 
> argparse 模块使编写友好的命令行界面变得容易。程序定义了它所需要的参数，argparse 将从 sys.argv 中解析出这些参数。argparse 模块还自动生成帮助和使用信息。当用户给程序提供无效的参数时，该模块也会发出错误。
> 
> ❞

### `Argparse`使用的步骤

> ❝
> 
> 创建一个对象   ---->>  添加参数 ----->>    解析
> 
> *   Create a parser object.    创建一个`parser`对象。
>     
> *   Add arguments your program accepts to the parser object.    将你程序接受的参数添加到解析器对象中。
>     
> *   Tell the parser object to parse your script's argv (short for argument vector, the list of arguments that were supplied to the script on launch); it checks them for consistency and stores the values.
>     
>     告诉解析器对象解析你的脚本的 argv（即启动时提供给脚本的参数）；它检查它们是否一致，并存储其值。
>     
> *   Use the object returned from the parser object in your script to access the values supplied in the arguments.
>     
>     使用从解析器对象返回的对象来访问参数中提供的值。
>     
> 
> ❞

### 示例解读

官方给的例子还是蛮难理解的，需要看后面的解释。**「后面会给出例子，帮助大家更好的理解各个参数的使用方法。」**

```
# 2023/5/27 00:40 , @File: demo_4.py, @Comment : python3.9.7
import argparse
# metavar 帮助信息会用到，type定义了返回的类型
parser = argparse.ArgumentParser(description='Process some integers.')
parser.add_argument('integers', metavar='N', type=int, nargs='+',
                    help='an integer for the accumulator')

# add_argument(self, *args, **kwargs)
# dest的作用就是后面可以使用args.xxx
parser.add_argument('--sum', dest='accumulate', action='store_const',
                    const=sum, default=max,
                    help='sum the integers (default: find the max)')

args = parser.parse_args()
print(args.accumulate(args.integers))

# 默认会求最大值
# python demo_4.py 1 2 3
# 3

# 如果第二个参数const指定了，就会使用const=sum，计算和
# python demo_4.py 1 2 3  --sum
# 6


```

```
python demo_4.py -h  # 查看帮助 


```

```
usage: demo_4.py [-h] [--sum] N [N ...]

Process some integers.  # 描述信息

positional arguments:   # 位置参数
  N           an integer for the accumulator

optional arguments:  # 可选参数 使用-或者--
  -h, --help  show this help message and exit
  --sum       sum the integers (default: find the max)



```

### 参数解读

> ❝
> 
> https://docs.python.org/3/library/argparse.html#the-add-argument-method
> 
> ❞

![](https://mmbiz.qpic.cn/sz_mmbiz_png/h988a0nsgw6y9fWic3PqnCjicEF6UtbTWD7ap5anaso49gDfcAUhmxgSXLUg0XoN4zsjMkO1EBCyNv8Fh3Hf14fQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/sz_mmbiz_png/h988a0nsgw6y9fWic3PqnCjicEF6UtbTWDsIRTIrMaXI6yh1ZvDvDxjKx8G2XR3oklYN8b08DoHmyeH696eGpnhQ/640?wx_fmt=png)

简单的翻译了下，有不准确的地方请大家批评指正：

```
"""
add_argument(dest, ..., name=value, ...)
add_argument(option_string, option_string, ..., name=value, ...)
"""

action 定义一个参数会被如何处理
choices 参数只能在这个范围内选择参数的值
const  存储一个常量（这属性跟 action 属性一起使用,具体什么意思可以使用的时候来体会）

default- 当命令行中没有参数就会使用它,给参数设置一个默认值
dest  将被添加到parse_args()返回的对象中的属性名称（可以args.xxx）
help  关于参数作用的简要描述

metavar  参数使用方法的名称（也就是说help中显示的那个名称）
nargs  命令行参数的数量,这个属性规定了参数可以输入的个数
required 参数的值为必填
type  命令行参数应该被转换为的类型。


```

#### **「`metavar`、`dest`以及`action`、`const` 结合使用的例子:」**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/h988a0nsgw6y9fWic3PqnCjicEF6UtbTWD6aTE1ictkSkcpOE6suicgtRibXWgkyF7ialSrOibQ9SJRlCLMjibJVv7zuFQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/sz_mmbiz_png/h988a0nsgw6y9fWic3PqnCjicEF6UtbTWDmePrZXGDibDge48cko2Sx05gtbkkbrfdlnn0fhzCIxIELFs0lmWIG9Q/640?wx_fmt=png)![](https://mmbiz.qpic.cn/sz_mmbiz_png/h988a0nsgw6y9fWic3PqnCjicEF6UtbTWDdPnVWX7rJU65XuzVd9KcHS4WJhaJFgw59xfyKAfGzejjfNCSb5SHVA/640?wx_fmt=png)

#### `action`使用的几个例子:

刚接触可能觉得难以理解，写几个 demo 示例就能理解了。

```
# 2023/5/27 12:25 , @File: test_08.py, @Comment : python3.9.7
import argparse

# 'store' - This just stores the argument’s value. This is the default action. For example:

# parser = argparse.ArgumentParser()
# parser.add_argument('--foo')
# print(parser.parse_args('--foo 1'.split())) # 这个就相当于在命令行后面跟的参数为1

# 'store_const' - This stores the value specified by the const keyword argument; note that the const keyword argument defaults to None.
# The 'store_const' action is most commonly used with optional arguments that specify some sort of flag. For example:

# parser = argparse.ArgumentParser()
# parser.add_argument('--foo', action='store_const', const=42) #无论你的参数是什么，都输出为42
# print(parser.parse_args(['--foo']))

# 'store_true' and 'store_false' - These are special cases of 'store_const' used for storing the values True and False respectively.
# In addition, they create default values of False and True respectively. For example:

# parser = argparse.ArgumentParser()
# parser.add_argument('--foo', action='store_true')
# parser.add_argument('--bar', action='store_false')
# parser.add_argument('--baz', action='store_false')
# print(parser.parse_args('--foo --bar'.split()))  # 触发就跟随action的，不触发就跟action相反
# print(parser.parse_args())

# 'append' - This stores a list, and appends each argument value to the list.
parser = argparse.ArgumentParser()
parser.add_argument('--foo', action='append')
print(parser.parse_args('--foo 1 --foo 2'.split()))


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/h988a0nsgw6y9fWic3PqnCjicEF6UtbTWDfjicibxYoqXtVLtTtBMJNA2uMn2XPKmPbZTuk0RwqBGf0TcxvWEnt8Cg/640?wx_fmt=png)![](https://mmbiz.qpic.cn/sz_mmbiz_png/h988a0nsgw6y9fWic3PqnCjicEF6UtbTWDtBicaPXur4vvueGDnAS3YicsGSZSGkpV7vpeQPb21CWDtdEHCgTWGaPg/640?wx_fmt=png)

#### **「`const`和`action`的结合使用的更多例子：」**

##### 求和

```
# !/usr/bin/env python3
# 2023/5/27 00:16 , @File: demo_1.py, @Comment : python3.9.7
import argparse

# Initialize the Parser
parser = argparse.ArgumentParser(description='Process some integers.')

# Adding Arguments
parser.add_argument('integers', metavar='N',
                    type=int, # 返回int
                    nargs='+',
                    help='an integer for the accumulator')

parser.add_argument(dest='accumulate',
                    action='store_const',
                    const=sum, # 求和
                    help='sum the integers')

args = parser.parse_args()
print(args.accumulate(args.integers))
# python demo_1.py 1 2 3
# 6


```

##### 排序

```
# 2023/5/27 00:10 , @File: demo_2.py, @Comment : python3.9.7
import argparse

# Initializing Parser
parser = argparse.ArgumentParser(description='sort some integers.')

# Adding Argument
parser.add_argument('integers',
                    metavar='N',
                    type=int, # 返回int
                    nargs='+',
                    help='an integer for the accumulator')

parser.add_argument(dest='accumulate',
                    action='store_const',
                    const=sorted, # 排序
                    help='arranges the integers in ascending order')

args = parser.parse_args()
print(args.accumulate(args.integers))
# python demo_2.py 11 2 35 44
# [2, 11, 35, 44]


```

##### 求平均值

```
# 2023/5/27 00:16 , @File: demo_3.py, @Comment : python3.9.7
import argparse

# Initializing Parser
parser = argparse.ArgumentParser(description='sort some integers.')

# Adding Argument  返回浮点数float
parser.add_argument('integers',metavar='N',type=float,nargs='+',help='an integer for the accumulator')

parser.add_argument('sum',action='store_const',const=sum)

parser.add_argument('len',action='store_const',const=len)

args = parser.parse_args()  # 解析参数
add = args.sum(args.integers)  # 求和
length = args.len(args.integers) # 求数据的长度
average = add / length # 求平均值
print(average)
# python demo_3.py 10 20 30
# 20.0


```

##### 更多的例子

```
# 添加一个可选参数
# 2023/5/27 01:14 , @File: demo_6.py, @Comment : python3.9.7
import argparse

tank_to_fish = {
    "tank_a": "shark, tuna, herring",
    "tank_b": "cod, flounder",
}

parser = argparse.ArgumentParser(description="List fish in aquarium.")
parser.add_argument("tank", type=str)
parser.add_argument("--upper-case", default=False, action="store_true")
args = parser.parse_args()

fish = tank_to_fish.get(args.tank, "")

if args.upper_case:
    fish = fish.upper()

print(fish)

# python demo_6.py   --upper-case tank_a
# SHARK, TUNA, HERRING

# python demo_6.py tank_a
# shark, tuna, herring



```

```
# 2023/5/27 01:24 , @File: demo_8.py, @Comment : python3.9.7
# Python参数传递单个值
import argparse

parser = argparse.ArgumentParser()
parser.add_argument('-H', '--host',
                    default='localhost',
                    dest='host',
                    help='Provide destination host. Defaults to localhost',
                    type=str
                    )
args = parser.parse_args()
print(f'The host is "{args.host}"')
# python demo_8.py -H 127.0.0.1
# The host is "127.0.0.1"

# python demo_8.py
# The host is "localhost"

# python demo_8.py --host 127.0.0.1
# The host is "127.0.0.1"


```

```
# 2023/5/26 22:49 , @File: test_03.py, @Comment : python3.9.7

import argparse


def test_for_sys(year, name, body):
    print('the year is', year)
    print('the name is', name)
    print('the body is', body)


parser = argparse.ArgumentParser(description='Test for argparse')
parser.add_argument('--name', '-n', help='name 属性，非必要参数')
parser.add_argument('--year', '-y', help='year 属性，非必要参数，但是有默认值', default=2017)
parser.add_argument('--body', '-b', help='body 属性，必要参数', required=True)
args = parser.parse_args()

if __name__ == '__main__':
    try:
        test_for_sys(args.year, args.name, args.body)
    except Exception as e:
        print(e)
#  python test_03.py  --name 1 --year 2 --body 3
#  python test_03.py  --name=1 --year=2 --body=3
#  python test_03.py -n 1 -y 2 -b 3


```

### 参考资料

<table><thead><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); font-size: 14px; min-width: 85px;">序号</th><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); font-size: 14px; min-width: 85px;" class="">参考链接</th><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); font-size: 14px; min-width: 85px;">备注</th></tr></thead><tbody><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">1</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">https://docs.python.org/3/library/argparse.html#</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">官方文档</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">2</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">https://click-docs-zh-cn.readthedocs.io/zh/latest/</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">click 库的文档</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">3</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">https://tendcode.com/article/python-shell/</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;"><br></td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">4</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">https://blog.csdn.net/tsinghuahui/article/details/89279152</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; min-width: 85px;">辅助理解 store_true or false</td></tr></tbody></table>
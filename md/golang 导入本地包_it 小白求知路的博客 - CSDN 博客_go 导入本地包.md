> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_39248122/article/details/115113241)

golang 导入本地包
============

创建初始化文件
-------

创建项目
----

创建一个名字为 test 的项目，该项目目录下有 calculator 目录和 tt 目录，calculator 目录下有 Sum.go（实现功能的文件），tt 下面有 main.go 项目入口文件。项目结构如下  
![](https://img-blog.csdnimg.cn/20210323112725313.png)

编写函数功能
------

编写 Sum.go 中的功能函数, 这是一个简单的两数相加功能

```
package calculator

var logMessage = "[LOG]"

// Version of the calculator
var Version = "1.0"

func internalSum(number int) int {
   return number - 1
}

// Sum two integer numbers
func Sum(number1, number2 int) int {
   return number1 + number2
}

```

> go 规范准则  
> 如需将某些内容设为专用内容，请以小写字母开始。  
> 如需将某些内容设为公共内容，请以大写字母开始。

创建初始化文件
-------

创建 calculator 包的初始化文件，命令方式。要注意不能把包的名字拼写错误  
![](https://img-blog.csdnimg.cn/2021032311322877.png)  
go.mod 文件创建成功  
![](https://img-blog.csdnimg.cn/20210323113401419.png)  
文件内容有 go 的版本和包名

```
module calculator

go 1.15

```

在初始化文件中引入本地包
------------

然后因为要在 main.go 中调用 calculator 包，所以 tt 目录也要初始化，一样的操作  
![](https://img-blog.csdnimg.cn/20210323142235676.png)  
生成之后要在初始化文件中指定所引入本地包的地址

```
module tt

go 1.15

replace test/calculator => ../calculator

require test/calculator v0.0.0


```
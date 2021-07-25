> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI2MDY2NzQwMQ==&mid=2247492637&idx=2&sn=e33ec6b107ba97ed1ac9b8b0a6dd5ade&chksm=ea64857bdd130c6dbf6fd83ffbbd68bae78b2d198fd1fb1f58116b814f197abe4e3c102ee813&mpshare=1&scene=1&srcid=0724UwvMRZjMbopCtSYhVQr0&sharer_sharetime=1627115472735&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

  

一、解析 “鸡兔同笼” 问题
--------------

“鸡兔同笼”最早记载于 1500 多年前的中国古代数学著作《孙子算经》中的 “卷下” 第 31 题（后传至日本演变为 “鹤龟算”），原题为：“今有雉兔同笼，上有三十五头，下有九十四足，问雉兔各几何？” 意思是“鸡和兔的总头数是 35，总脚数是 94，鸡和兔各有几只？”。

#### 1. 问题求解

假设鸡有 x 只，兔有 y 只，根据题意列方程为：

```
x+y=35，2x+4y=94。
求解，得：x=23，y=12；即鸡有23只（46只脚）、兔有12只（48只脚）。
```

#### 2.Python 编程求解

如果使用 Python 语言来编写程序的话，可使用 for 循环、range() 函数和 if 条件判断来完成。

*   先使用 “heads = 35” 和“feet = 94”两个赋值语句，保存鸡和兔的总头数和总脚数；
    
*   接着使用 range() 函数进行 for 循环，让鸡的数目从 1 开始计数加 1 循环，循环体中的 if 条件为 “2_x + 4_y ==feet”，即 “鸡数目的两倍加兔数目的四倍之和等于总脚数”，条件成立的话，使用 print 语句进行最终鸡兔数目的输出。
    
*   保存程序为 “鸡兔同笼 1.py”，运行结果显示为 “鸡有 23 只，兔有 12 只。”（如下图）。
    

![](https://mmbiz.qpic.cn/mmbiz_png/P33BNFicia1AmhuD2Ppno2ibxSNicXQQQA5HCvvTZRISBMiaPv25N6HrlibD0XXoBlCVCtxGow4LuC8dQ186pSTBDTkw/640?wx_fmt=png)

#### 3. 升级版 “鸡兔同笼” 问题的 Python 编程求解

首先使用标准输入函数 input 来接收用户从键盘上输入的信息，比如`“heads = input('请输入鸡和兔的总头数：')”`和`“feet = input('请输入鸡和兔的总脚数：')”`。但在此需要特别注意的是，Python 的 input 函数接收到的输入数据是 str 字符串（虽然表面上看是数字），必须要使用 int 来转换成整数型才能进行数学运算，语句为`“heads = int(heads)”和“feet = int(feet)”`。

接下来仍然是使用 range() 函数进行 for 循环：`“for x in range(0,(heads+1))”`。此时要充分考虑到用户所输入数据的计算结果，很有可能会出现 “只有鸡” 或“只有兔”的情况。举例：用户输入的总头数是 10、总脚数是 20，运算结果就应该是`“10只鸡、0只兔”`；或输入总头数是 10、总脚数是 40，运算结果则是`“0只鸡、10只兔”`。因为在计算机编程语言中，数字 0 总是被看作是最起始的值，Python 的列表、字符串和元组等的元素均是从 0 开始进行索引的。不管是`“0只鸡”`还是`“0只兔”`，在计算机看来，这都是 “鸡兔同笼”，只不过数目是 0 而已。另外，由于 range（）函数的两个参数是“左闭右开” 型的区间，即第一个参数是被包括计算在内，而第二个参数却是不包括在内的（只计算到它的前一个元素）；所以，第二个参数应该设置为 “heads+1”，这样就能在循环时计算到它的前一个元素（即“heads”），也就是“0 只兔” 的情况（“x=0”则是“0 只鸡”）。

循环体与之前类似，仍然是 if 条件判断`“2*x + 4*y == feet”`是否成立，成立的话则使用 print 输出结果，然后使用 break 语句跳出循环。因为不确定用户从键盘上输入的两个数据是否恰好为 “有效解”——鸡和兔的数目必须是整数只，所以在循环体外应该再添加一个`“if 2*x + 4*y != feet”`判断语句，将这种无法进行整数结果计算的情况进行提示 “输入的总头数和总脚数不合法”。没有该 print 语句的话，程序也能正常运行，但对于这种`“意外”`没有任何提示，程序缺少必要的友好性。

最后将程序保存为`“鸡兔同笼2.py”`，运行几次进行测试，输入的总头数和总脚数包括原题目中的`“35、94”`、鸡兔各为 0 只、`“30、110”`四种合法数值，程序均输出了正确的计算结果；最后一个测试输入`“8、100”`，结果就提示 “输入不合法”（如下图）。

![](https://mmbiz.qpic.cn/mmbiz_png/P33BNFicia1AmhuD2Ppno2ibxSNicXQQQA5HM6iaombRiasUIib4iaGqOmZy9ZQgJSicInVE9b6zcJicyj1CzjUjMs70nyicQ/640?wx_fmt=png)

二、解析 “Fibonacci 数列” 问题
----------------------

Fibonacci（音译为：`斐波那契`）数列又称`“黄金分割数列”`，最早是由意大利数学家 Fibonacci 以兔子繁殖为例引入，因此又称 “兔子数列”。其规则为：数列的第 0 项是 0，第 1 项是第一个 1，从第三项开始，每一项均等于前两项之和，即：0，1，1，2，3，5，8，13，21……

#### 1.Fibonacci 数列的数学解析

一般而言，兔子在出生两个月之后就会有繁殖能力，一对兔子每月能生出一对小兔子。假设兔子不死亡，一年之后会繁殖出多少对兔子？经分析后不难发现，成年兔子的对数符合这样的函数定义：

```
F(0)=0，F(1)=1，F(n)=F(n-1)+F(n-2)(n≥2,n∈N)
```

如何使用 Python 编程来求解这样的 Fibonacci 数列呢？

#### 2. 常规的 Python“递归” 编程求解

`“递归”`即函数在运行过程中不断地直接或间接调用自身的一种算法，比如在 Python 中通过`“def fib1(n):”`来定义 fib1() 函数，其主体内容为`“三分支”`结构：前两种（if 和 elif）通过判断参数 n 是 0 还是 1 来分别对应 Fibonacci 数列的前两项 0 和 1，二者均通过 return 语句来返回对应的数值。注意判断条件中的双等号的含义是`“等于”`，一个等号是 “赋值” 运算。第三个分支（else）是`“return fib1(n-1)+fib1(n-2)”`，意思是递归运算返回该项前两项值的和：F(n)=F(n-1)+F(n-2)。

最后使用`“print('一年之后会繁殖出的兔子对数为：',fib1(12))”`来输出运算结果，其中的`“fib(12)”`作用是调用 fib1() 函数，参数为 12（一年的月数）。保存程序为 fibonacci1.py，运行后得到结果是 144（如下图）。

![](https://mmbiz.qpic.cn/mmbiz_png/P33BNFicia1AmhuD2Ppno2ibxSNicXQQQA5HnTiabcibaFXEpxxQeBPWdeuMSygRibH7pwTfBe3Cat5DSvKiaH20t3SqIg/640?wx_fmt=png)

#### 3. “升级版”Python 编程求解

Python 支持多变量在一行语句中同时赋值的运算，比如`“x,y=y,x”`，意思是 x 和 y 这两个变量的值进行 “互换”。对于这种两个变量进行值互换的运算，其它编程语言几乎都是通过第三方变量来“暂存” 中间数据的方式来完成的，例如最初有`“x=3”和“y=4”`两个赋值语句，分别将 3 和 4 这两个数据给变量 x 和 y；接着需要再通过三个赋值语句完成 x 和 y 数据的互换：`“z=x”、“x=y”和“y=z”`，意思分别是 `“将 x 的值（3）给 z”、“将 y 的值（4）给 x” 和 “将 z 的值（3）给 y”，此时 x 的值变成 4、y 的值变成 3。

如果使用 Python 的多变量同时赋值方法来编程，就可以通过`“def fib2(n):”`来定义 fib2() 函数。首先通过`“a,b = 0,1”`语句，实现变量 a 和 b 同时被分别赋值 0 和 1，对应 Fibonacci 数列的前两项；接着使用 for 循环和 range() 函数`“for i in range(n):”`，其循环体为`“a,b = b,a+b”`，意思是将 b 的值给 a、将 a+b 的值给 b，实现之前使用递归算法完成的第三项及之后项的 Fibonacci 数列运算；for 循环体结束后，通过 “return a” 语句将变量 a 的值返回；最后仍是通过 print 语句的`“fib2(12)”`来调用函数计算并输出，保存程序为 fibonacci2.py，运行结果仍是 144（如下图）。

![](https://mmbiz.qpic.cn/mmbiz_png/P33BNFicia1AmhuD2Ppno2ibxSNicXQQQA5HwG6Gn8yibwV6ZqVt76UDAn3rXLDPpZE096ObXx778YQ2quB4uAbYjJA/640?wx_fmt=png)

#### 4. 求任意项 Fibonacci 数列的 Python 编程

理论上讲，Fibonacci 数列的值是无穷的，如何使用 Python 编程来实现输出 Fibonacci 数列任意项？仍然可以先通过 input 函数来接收用户从键盘上输入的`“要求”`，注意一定要使用 int() 函数将该字符串型数据转换为整数型数据；接着定义 fib3() 函数，内容与上面的 fib2() 完全相同，同样是返回 a 的值；然后使用 print 语句输出提示信息，再同样是通过 for 循环加 range() 函数，循环体内的`“print(fib3(i),end=' ')”`是调用 fib3() 函数，其中的`“end=' '”`作用是控制打印输出的各项 Fibonacci 数列值之间使用一个空格来分隔（默认是回车）。将程序保存为 fibonacci3.py，运行测试，分别尝试输入 10、20 和 50，程序就会根据要求输出 Fibonacci 数列的前 10、20 和 50 个数值（如下图）。

![](https://mmbiz.qpic.cn/mmbiz_png/P33BNFicia1AmhuD2Ppno2ibxSNicXQQQA5HHI9mFuluOmlYRJ5SxJIC2JHOFtWGPUhTHgXye3xNibdREHpr6Gkxmicw/640?wx_fmt=png)

三、解析 “棋盘米粒倍增” 和“九九乘法表”问题
------------------------

印度有个古老传说：舍罕王打算奖赏国际象棋的发明人——西萨宰相，在被问及想要得到的赏赐时，宰相回答说：“在棋盘的第 1 格放 1 粒大米，第 2 格放 2 粒，第 3 格放 4 粒，之后的每一格中的米粒数目都是相邻前一格的两倍，一直放到最后的第 64 格，我只要这一棋盘的大米。”

最初国王不以为然，但最终的结果却是举全国之力都无法填满这个棋盘。果真是这样吗？我们使用 Python 编程来解决这个`“棋盘米粒倍增”`问题。

#### 1. 常规的循环求和法

首先通过`“sum = 0”`语句建立并为变量 sum 赋值为 0，准备存放最终的米粒数目；接着使用 for 循环：`“for i in range(64):”`，其中的 range() 函数负责提供从 0 到 63 共 64 个循环计数；由于每格中米粒的数目可表示为`“2的（n-1）次方”`，所以循环体语句为`“sum += 2 ** i”`，将每次循环得到的该格子中米粒的数量与之前所有格子中米粒的数量和进行求和；循环结束后通过 print 语句将求和结果输出。

将程序保存为 chessrice1.py，运行后得到结果（如下图）：

![](https://mmbiz.qpic.cn/mmbiz_png/P33BNFicia1AmhuD2Ppno2ibxSNicXQQQA5H4kY9hQZQqlt0wSZF3F7CGUt7Gs81W3gHYZfFLZDMIicibUiaCbvpvJmgA/640?wx_fmt=png)

棋盘米粒的总数为：18446744073709551615 粒。

#### 2. 使用列表推导式计算

Python 的列表推导式在逻辑上等同于循环语句，优点是形式简洁且速度快，它能够以非常简洁的方式对列表（或其他可迭代对象）中的元素进行遍历、过滤或再次计算，从而快速生成满足特定需求的列表。

Python 的列表推导式可分解为`“表达式+循环”`两部分，比如通过`“sum = sum([2**i for i in range(64)])”`这一个语句即可完成所有 64 格子中米粒的数量求和，其中的 “2**i” 即`“表达式”`部分，作用是计算每格中的米粒数量；后面的 “for i in range(64)” 是`“循环”`部分，作用是控制完成从 0 到 63 共 64 次循环；sum 变量的赋值，是通过内置求和 sum() 函数来完成的。

之前使用常规循环求和法得到的结果是一个 20 位长的天文数字，单位是 “粒”，不够直观。经查询，1 千克大米约有 52000 粒，通过`“mass = int(sum / 52000000)”`语句，将这些大米的数目转换成单位为 “吨” 并进行求整，赋值给 mass 变量，最后打印输出。将程序保存为 chessrice2.py，运行后得到结果（如下图）：

![](https://mmbiz.qpic.cn/mmbiz_png/P33BNFicia1AmhuD2Ppno2ibxSNicXQQQA5HMxR5KNNzM5OXHWJmFiauVQX3leUTG9A4juCSXdAibf8UVSzjV6yia1tLw/640?wx_fmt=png)

棋盘米粒的总数为：18446744073709551615 粒。

这些米粒的总质量为：354745078340 吨。

米粒总数的计算结果与循环求和法一致，它们的总质量是个 12 位数字，约是 3547.5 亿吨！当时，国王无论如何也拿不出数量如此庞大的大米，根本就填不满宰相的棋盘。

#### 3. 两种方法打印 “九九乘法表”

不管是使用常规循环求和还是使用列表推导式，我们都可以正确求解 “棋盘米粒倍增” 问题，二者在各种问题的求解过程中都比较方便，包括循环的嵌套，比如打印“九九乘法表”。

###### （1）常规的双层循环嵌套

外层循环语句为`“for i in range(1,10):”`，作用是从 1 到 9 循环；

内层循环语句为`“for j in range(1,i+1):”`，同样是使用 range() 进行对应次数的循环；

循环体语句为`“print('{0}*{1} = {2}'.format(j,i,i*j),end=' ') ”`，这个 print 语句用到了 Python 的 format() 方法进行字符串格式化，其中的`“{0}”、“{1}”和“{2}”`是位置参数，作用是将后面`“format(j,i,i*j)”`中的三个变量的对应数值进行占位输出；`“end=' '”`的作用是设置末尾不换行，而不是 print 的默认 “换行” 值；内层循环结束后是一个`“print()”`空语句，作用是换行，即打印完同一个乘数（比如同是乘以 3）的一行循环后，回车换行。将程序保存为 ninenine1.py，运行后得到 “九九乘法表”（如下图）。

![](https://mmbiz.qpic.cn/mmbiz_png/P33BNFicia1AmhuD2Ppno2ibxSNicXQQQA5HTCKNPGrl2T0oZRY2dUJ6Pou2xd9ibXHr4RDKj6fmB6eUAUYpFATKJGA/640?wx_fmt=png)

###### （2）列表推导式循环嵌套

外层循环语句仍为`“for i in range(1,10):”`，内层直接就是一个列表推导式（因为本身就是一层循环）：`“print(" ".join(["%d*%d=%-2d"%(j,i,j*i) for j in range(1,i+1)]))”`。这个 print 语句中的 “join()” 方法是将序列中的元素以指定的字符连接生成一个新字符串，依次连接到前面的 " " 空串后面；其中的`“%d”`的作用是将数据按照整型格式化输出，“-” 表示左对齐，“2” 表示数字不足两位时进行位数补齐（不足位置用空格）。

列表推导式后面的循环部分是`“for j in range(1,i+1)”`语句，与常规双层循环嵌套的内层循环语句完全相同。将程序保存为 ninenine2.py，运行后，同样也得到了 “九九乘法表”（如下图）。

![](https://mmbiz.qpic.cn/mmbiz_png/P33BNFicia1AmhuD2Ppno2ibxSNicXQQQA5HSM97yl5iavibcG7Slz2dl0SGggsth5Jl0uV8R5bFacZ6LdGGNsmS4KfQ/640?wx_fmt=png)

四、多法解析 “随机抽奖” 问题
----------------

假设要从 10000 个人中随机抽取出 10 人作为`“中奖者”`，每人对应一个 0-9999 中的整数，要求使用 Python 编程按从小到大的顺序输出中奖者数字代号。类似的 “随机抽奖” 程序一般均需要先导入 random 模块，然后借助其中的 randint()、shuffle()和 sample()等函数进行随机数的选取；最后使用列表或集合对数据进行存储、排序和输出。

#### 1.randint() 生成随机整数后进行 in 成员运算判断

首先，通过 “import random” 导入 random 模块（下同）；

接着，建立空列表 “my_list1 = []”；建立 while 循环结构，判断条件为`“len(my_list1) <= 10”`，即列表 my_list1 中元素的个数达到 10 为止（通过 len() 检测列表的长度）；在循环体中，第一条语句为`“x = random.randint(0,9999)”`，变量 x 取值为 0-9999 中的随机某个整数（包括 0 和 9999）；条件判断语句`“if x not in my_list1”`的作用是，查看生成的随机数 x 是否在列表 my_list1 中，防止多次生成的随机数中有重复值出现；如果不重复，则使用 append() 方法将 x 追加到列表 my_list1 中：`“my_list1.append(x)”`；当循环结束时，列表 my_list1 中就会保存有 10 个 0-9999 间的不重复数据。

最后，通过 sorted() 函数对列表 my_list1 进行默认参数排序（升序）：`“my_list2 = sorted(my_list1)”`，得到的列表 my_list2 就是从小到大顺序中奖号码，再使用 print()输出结果即可。运行程序，得到了 10 个 “中奖” 号码（如图 10）。

![](https://mmbiz.qpic.cn/mmbiz_png/P33BNFicia1AmhuD2Ppno2ibxSNicXQQQA5HjpTHVomkgRnjFr5YCom2I0WTSiaPua46LjRRwdF4cWI3NUib71Kn4XicQ/640?wx_fmt=png)

#### 2.randint() 生成随机整数后存入集合 “去重”

与法 1 类似，只不过是使用集合而非列表来存储生成的随机数：`“my_set = set()”`，建立一个空集合；接着，仍然是在 while 循环中，通过 randint 生成 0-9999 间的某随机数，再将它追加到集合 my_set 中。由于集合中的元素是不可能存在重复数据的，因此不必像法 1 中的列表元素进行 in 成员运算判断，相当于直接进行了 “去重” 操作。循环结束后，仍然是使用 sorted()函数进行排序并保存至列表 my_list 中，进行 print 打印输出（如下图）。

![](https://mmbiz.qpic.cn/mmbiz_png/P33BNFicia1AmhuD2Ppno2ibxSNicXQQQA5HHAlLnFKnr82WcAicZcxeSffrXr0hOrwZMzibT7AoGicR8kobWzeFCu62w/640?wx_fmt=png)

#### 3.shuffle() 随机排序后进行 “切片”

首先建立列表 my_list1，其值为`“list(range(10000))”`，通过 list() 将 0 至 9999 共 10000 个数据保存至列表 my_list1 中；接着使用 random 中的 shuffle()，将列表 my_list1 中的数据进行随机排序：`“random.shuffle(my_list1)”`；

然后对列表 my_list1 进行切片操作，任意截取出 10 个数据，比如 “my_list1[:10]” 是指从索引的第 0 个切至第 9 个（当然也可以使用`“my_list2 = my_list1[99:109]”`，意思是从第 99 个切至第 109 个），将它们存入列表 my_list2 中；仍然是使用 sorted() 函数进行排序并保存至第 3 个列表 my_list3 中，进行 print 打印输出（如下图）。

![](https://mmbiz.qpic.cn/mmbiz_png/P33BNFicia1AmhuD2Ppno2ibxSNicXQQQA5Ho0ewjdRW83xuVzIB37qltrwiaia0oPfzUjSlnfoGd4UotWKBjYia9O7kA/640?wx_fmt=png)

#### 4.sample() 随机多个 “取样”

Random 中的 sample() 功能是从序列中随机多个`“取样”`。首先建立列表 my_list1，其值为从 0-9999 中随机抽取 10 个不重复的数据：`“my_list1 = random.sample(range(10000),10)”`；然后就可以使用 sorted() 函数进行排序，将结果保存至列表 my_list2 中，最后进行 print 打印输出（如下图）。

![](https://mmbiz.qpic.cn/mmbiz_png/P33BNFicia1AmhuD2Ppno2ibxSNicXQQQA5HxE1iaN9AbHhnQBRT0pIibm09IHgWDvOicRXh3VUfBibiaymnFsMW3QBfFaw/640?wx_fmt=png)

#### 5. numpy 中的 random.choice() 随机项提取

Numpy 中有个`random.choice()`，可以随机从指定列表中提取若干个元素。

首先，通过 “import numpy as np” 导入 numpy；

接着建立列表 my_list1，存储的数据是 0-9999 共 10000 个数据：`“my_list1 = list(range(10000))”`；建立列表 my_list2，值为从列表 my_list1 中随机提取 10 个不重复的数据：`“my_list2 = np.random.choice(my_list1,10,replace=False)”`，其中的参数 “replace=False” 即为控制随机数“不重复”。

最后，使用`sorted()`函数进行排序并保存至第 3 个列表 my_list3 中，进行 print 打印输出即可（如下图）。

![](https://mmbiz.qpic.cn/mmbiz_png/P33BNFicia1AmhuD2Ppno2ibxSNicXQQQA5Hz2mfO31WkEcdGpnhljqJgNKibH9sZyDuibO7fSaoO5MFey5rXiaiaKqOTg/640?wx_fmt=png)

五、多法解析 “自幂数” 问题
---------------

在编程语言的学习过程中，有一道经典的`“水仙花数”`求解问题，即某个三位整数每个数位上数字的三次幂之和等于它本身，比如`“153 = 1^3 + 5^3 + 3^3”`。其实，水仙花数只是 “自幂数” 的一种，类似的还有四位数的 “四叶玫瑰数”、五位数的“五角星数”、六位数的“六合数” 等等。

Python 语法灵活，可以使用多种方法编程来完成自幂数的求解，在此略举几种水仙花数的编程方法：

#### 1.“整除”和 “求余” 数位分解法

在 Python 中，运算符 “//” 代表`“整除”`运算，即求 “整商”；而运算符“%” 则是进行 “求余”，利用这两种运算符可以将一个多位数的各位数字“分解” 提取。

在判断一个三位数是否为水仙花数时，首先构建循环结构 “for i in range(100,1000):”，百位上的数字提取方法是通过“bai_wei = i//100” 求`“整商”`来完成，比如计算 “365//100”，结果就是“3”；十位上的数字提取方法是“shi_wei = (i%100)//10”，即先以 100 为除数进行“求余”，再将这个中间结果除以 10 求“整商”，比如计算“（365%100）//10”，会先得到余数 65，然后计算“65//10” 得到 6；个位上的数字提取方法是 “ge_wei  = i%10”，即除以 10 求余数，比如“365%10” 的结果是 5。

循环中的 if 判断条件是 “bai_wei**3 + shi_wei**3 + ge_wei**3 == i:”，即各数位上的数字的三次方之和与该数相等。最后，通过 print 打印输出变量 i 的数值，结果得到四个水仙花数：153、370、371 和 407（如下图）。

![](https://mmbiz.qpic.cn/mmbiz_png/P33BNFicia1AmhuD2Ppno2ibxSNicXQQQA5HAwK6B6PkicuvQxOHOpNlnI7mqZHDRl2CnvibWyI921LoU8XLtqib9V6YA/640?wx_fmt=png)

#### 2. 三层循环嵌套法

因为水仙花数是对一个三位数进行判断，所以直接构建三层循环嵌套来实现从 100 到 999 的顺序递增。最外层的`“for bai_wei in range(1,10):”`控制百位数字循环，注意要从 1 开始（range()中的起始值和终止值参数为 “左闭右开” 区间）；中间层的十位数字循环是`“for shi_wei in range(0,10):”`；内部的个位数字循环是 “for ge_wei in range(0,10):”，变量 my_data 是计算存储每个三位数的数值大小，即 “bai_wei_100+shi_wei_10+ge_wei”；判断条件与之前相同，最后也是打印输出结果，同样会得到四个水仙花数：153、370、371 和 407（如下图）。

![](https://mmbiz.qpic.cn/mmbiz_png/P33BNFicia1AmhuD2Ppno2ibxSNicXQQQA5HSGz01RCd7bnNR38Y1nmjFNEEaIVtqQL8xsUoSml5polK6ZRfLrgprA/640?wx_fmt=png)

#### 3.map() 函数映射法

如果充分利用 Python 中的各种内置函数，比如`map()`映射函数，可以非常巧妙地快速 “提取” 出每个多位数上各数位的数字。首先，同样是通过 “for i in range(100,1000):” 构建出循环结构；然后使用`“序列解包”`的方式，同时为三个变量赋值——“bai_wei,shi_wei,ge_wei = map(int,str(i))”，借助 map()函数将每个三位数先通过 “str(i)” 转换为字符串，再将 int()函数映射至刚刚生成的字符串序列（迭代对象），就`“还原”`得到了三个整形数字，分别赋值给三个对应的变量。

接下来仍是使用相同的判断语句和 print() 输出语句，同样会得到四个水仙花数：153、370、371 和 407（如下图 ）。

![](https://mmbiz.qpic.cn/mmbiz_png/P33BNFicia1AmhuD2Ppno2ibxSNicXQQQA5HVicSF8flfLib25O1OIDc1AoGL1kyfD9BXSia2v1Wzic0WneLUkicaWIxOZQ/640?wx_fmt=png)

六、多法解析 “均匀浮点数生成” 问题
-------------------

众所周知，在 Python 中可构造 “for i in range(100)” 语句来执行 100 次循环，因为 “range(100)” 就相当于 “range(0,100,1)”，是以 1 为步长、“左闭”（包括 0）“右开”（不包括 100）的；如果在该循环中被执行的语句是“print(i,end='')” 的话，那就会打印输出从 0、1、2……98、99 共 100 个整数。按照这个规律，是否可以使用 range()函数来生成类似的均匀浮点数呢？比如从 0.00、0.01、0.02……0.98、0.99 共 100 个浮点数。如果直接构造 “for i in range(0,1,0.01)”，Python 就会给出“TypeError:'float' object cannot be interpreted as an integer” 的错误提示，意思是 “类型错误：浮点型对象不能解释为整数型”，因为 range() 函数接收的参数必须是整数（可以是负数），而不能直接处理 float 浮点数。那么，如何解决均匀浮点数生成问题呢？

#### 1.while 循环控制变量 i 自增

首先建立并给变量 i 赋值为 0.00；接着构造 “while i <= 1.00:” 循环，其中的第一条语句为`“print('%.2f'%i,end=' ')”`，即以一个空格分隔并保留两位小数输出变量 i 的值；第二条语句为`“i += 0.01”`，即控制 i 的自增，步长为 0.01。运行程序，得到了从 0.00 到 0.99 共 100 个均匀浮点数（如下图）。

![](https://mmbiz.qpic.cn/mmbiz_png/P33BNFicia1AmhuD2Ppno2ibxSNicXQQQA5HnBFsK6MN8MuL2R0ibesAaXXHOZIRoiaQQ5g9UkASJSH0FKunf6q3utRw/640?wx_fmt=png)

#### 2. 使用列表推导式

Python 的列表推导式非常灵活，能够以非常简洁的方式来快速生成满足特定需求的列表。比如直接使用一条`“my_list = [i/100 for i in range(100)]”`语句，即可在列表 my_list 中得到符合要求的 100 个浮点数，其实就是将 “for i in range(100)” 所得到的 0-99 分别进行了 “i/100” 的计算。最后再使用 for 循环以同样的方式来打印输出，同样也得到了 100 个均匀浮点数（如下图）。

![](https://mmbiz.qpic.cn/mmbiz_png/P33BNFicia1AmhuD2Ppno2ibxSNicXQQQA5HobICybdvSudjYY6smWLofWmH7ABKgqibwib8JbnkU3NavQUEAN4LPFZw/640?wx_fmt=png)

#### 3. 借用 numpy 库中的 arange()

numpy 库中有个与 Python 的`range()`函数功能类似的`arange()`，它是支持浮点数运算的，而且同样是使用`“初始值、终值、步长”`三个类似的参数进行调用。在使用 “import numpy as np” 语句以 np 为别名导入 numpy 库之后，再使用 “my_list = list(np.arange(0,1,0.01))” 语句，即可将 arange()生成的 ndarray 数组对象转换为列表数据。最后，同样是使用 for 循环打印输出 my_list 中的所有元素，就得到了 100 个均匀浮点数（如下图）。

![](https://mmbiz.qpic.cn/mmbiz_png/P33BNFicia1AmhuD2Ppno2ibxSNicXQQQA5HoNqSeU45VCchVFfnfzGCz4HBWzeFOEI5yfp6RvX5q2CeuOJq48yUvQ/640?wx_fmt=png)

#### 4. 自定义函数使用 yield 表达式

既然 Python 内置的 range() 函数不提供对浮点数的运算，那我们就可以自定义一个 float_data() 函数，三个参数依次为 start、end 和 step，同样是对应`“初始值、终值、步长”`。函数中使用变量 i 来接收初始值，然后通过 while 循环（当 i<end 时）中的 “yield i” 来向外返回 i 的值，当然还要有变量 i 的步长自增语句：`“i += step”`。

在主程序中调用 float_data() 函数，接收到的数据存储至变量 my_generator 中，最后仍然是通过 for 循环来将它们打印输出，也可以得到 100 个均匀浮点数（如图 22）。

![](https://mmbiz.qpic.cn/mmbiz_png/P33BNFicia1AmhuD2Ppno2ibxSNicXQQQA5HsE0kb8hne2hmPaX9iasSlibfJxNUqBvEtflfdwRo0BbCWmlZ5mkjZKlg/640?wx_fmt=png)

四种方法均能实现均匀浮点数的生成，大家可根据自己的编程习惯来使用。当然，如果想生成的是 0.000、0.001、0.002……0.999 这样的千分位均匀浮点数，只要在程序中将步长修改为 0.001、print 输出`“%.3f”`以及方法 2 中将 “i/100” 修改为 “i/1000” 等等即可。
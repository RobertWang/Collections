> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652598980&idx=1&sn=4019de15dcd36dbe29134e8fbcb003b8&chksm=84655a8eb312d39892631734a14262439c0e721f7e38346814482d8f3c4f4161fdeec300d29b&mpshare=1&scene=1&srcid=1123ItZH3PrxBFLbim9zThsv&sharer_sharetime=1669213887085&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

一、引言
----

CPU 如何工作？是困扰初级用户一个迷雾般的难题。我们可能知道诸如程序计数器、RAM、寄存器的只言片语，但尚未对这些部件的工作原理及整个系统的协同有清晰和总体的认识。

本文使用四十行 Python 代码来实现一个最简单的 CPU。使它可编程，支持加减法运算、读写内存、无条件跳转、条件跳转的功能。之所以实现一个相对简单的 CPU，是想让大家从整体上理解 CPU 工作原理，不要过早被细节羁绊住。

> “
> 
> 真实 CPU 是采用在硅片上蚀刻的方式生产三极管或者 MOS 管，这些三级管充当的是开关：在开关开闭之间，实现了加法、减法和存储。之前我用 Python 代码从一个开关开始，模拟出一个类似本文的 CPU。但是这里，我们从更高层次上模拟 CPU：用代码模拟大的部件，使大家从原理上理解 CPU 工作。
> 
> ”

二、CPU 的组成
---------

整个 CPU 大的部件组成，如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/l7Q86F4y59sCKqS5ZD89NW7iaRI7GEHwVQlrW9iaHz0yU5GZW6b5v72MJVjL3JRIMognlFEmrhHWgCicq74HBgqag/640?wx_fmt=png)哈佛结构 CPU

本 CPU 采用的是 Harvard architecture（哈佛结构）。哈佛结构比较简单易懂、实现起来比较容易。上图中有指令 RAM 和数据 RAM，两个 RAM 就是哈佛结构的重要标志。

> “
> 
> 常见的两种 CPU 结构为哈佛和冯诺依曼结构。哈佛结构是将程序指令和数据存储在同一块 RAM 中的 CPU 设计方案。von Neumann architecture（冯诺伊曼结构），也称普林斯顿结构，是一种将程序指令存储器和数据存储器合并在一起的 CPU 设计方案。
> 
> ”

哈佛结构和冯诺依曼结构的主要区别：前者程序和数据分为两个 RAM 存储，后者统一存储在一个 RAM 中。

三、工作原理
------

让我们分别介绍各部件工作原理，之后介绍它们怎么协同工作。

#### 3.1 各部件工作原理

上图中各部件，在真实 CPU 中，都有相应物理电路与其对应，它们的功能分别是：

*   pc 计数器，从 0 开始产生 0，1，2，…… 计数可以清零，也可以从外部输入一个数，从这个数从新开始计数，这被称为置位。用于指示程序和数据存取位置。
    
*   RAM，存储数据的随机存储器，支持根据地址（0x01 这种整形）读取数据，根据地址和写入信号 w 写入数据。用于存储程序和数据。
    
*   寄存器，存储 8 bit 信息的存储器，根据 w 信号为 1 写入当前数据，w 为 0 表示读取。类似 RAM，但只能存储 8 bit 信息。常用于存储指令、地址和计算中间量。
    
*   加法器，完成两数加减法运算，sub 为 1 时表示减法，ci 为 1 时表示进位。这个器件是核心器件，用于构成 ALU（算数逻辑单元）。真实 CPU 是采用逻辑门搭建，还有乘法器、逻辑运算单元，等等。
    
*   21 选择器，相当于单刀双掷开关，根据 s21 信号，决定 8 bit 输出来自或左或右 8 bit 输入端。
    

#### 3.2 协同工作原理

上图中箭头表示的是数据流向，也称为数据通路图。从数据通路图上我们能分析出 CPU 是怎么设计出来的。

整个数据通路从程序计数器 pc 开始，计数器从 0 开始输出数字 0，1，2，3，4……。指令 RAM 和数据 RAM 中分别存储程序代码和数据。RAM 采用数字表示的位置访问、存储数据。根据计数器地址 0，1，2 之类，将 RAM 中的数据分别放入指令寄存器 IR 和数据寄存器 DR。寄存器相当于容器、变量，存储了 RAM 给它的数据。

指令寄存器中的指令码解码产生 CPU 控制指令，这些 0 和 1 分别表示低电平和高电平信号，而电平信号则控制诸如加法器进位与否，是否打开减法，是否使能寄存器写入，选择 21 选择器哪一个输入作输出，是否重置计数器，等等。所以，指令其实是控制 CPU 各部件协同工作的电信号。

数据寄存器中的数据分别走向加法器 adder 来进行加法、减法运算后流向 21 选择器，也可能直接流向 21 选择器等待选择。21 选择器选择后，数据进入累加寄存器 AC 。累加器的数据根据 ac 信号是否为高电平 1 ，来决定写入与否。AC 累加器的数据会参与下次计算或者根据 w 信号存入数据 RAM 中。

至此，我们完成了一次计算，程序计数器加 1，然后执行下一次计算。如果本条指令是跳转指令的话，将跳转目的地址直接赋值给程序计数器，程序重新地址开始执行。

下面我们用一个实际例子来说明 CPU 执行过程。

四、CPU 指令工作详细剖析
--------------

一个完整程序是由指令和数据构成。指令负责控制 CPU 各部件协同工作，数据则参与具体运算。例子程序完成 10+2-3，然后循环减 3 直到结果为 0 时程序完成。指令寄存器为 ramc，数据寄存器为 ramd。

```
ramc = [0x18, 0x19, 0x1d, 0x02, 0x31, 0x30, 0x00]
ramd = [10, 2, 3, 0xff, 0x06, 0x02]


```

下面我们来学习指令怎么控制 CPU 的。

#### 4.1 指令

上文中我们知道，程序计数器从 0 开始输出。CPU 完成计算操作后，整个计数器会加 1，取得下一条指令继续执行。我们以 pc 为零时取到的第一条指令 0x18 为例讲解指令是怎么解码，进而控制 CPU 各器件的。

0x18 二进制形式为 0b0001 1000。每一位二进制，都指向一个器件的使能端。所谓使能端，就是让这个部件工作的开关。比如这里的两个 1 代表高电平，分别连接数据寄存器 DR 和累加寄存器 AC 的使能端 w。这样当我们读到 0x18 时，我们知道 CPU 会把当前数据通路上的数据写入数据寄存器和累加寄存器。

具体每个二进制位和 CPU 各器件使能端间的对应关系，就是指令。本文设计连线如下。当前状态就是 0x18 指令控制 CPU 器件的状态。

![](https://mmbiz.qpic.cn/mmbiz_png/l7Q86F4y59sCKqS5ZD89NW7iaRI7GEHwVGiaKLa0CdRoZbhFKQ71ZAXdfukPp9J3mo06Z4HPNlYfVr3gSJ76yJgg/640?wx_fmt=png)指令之 0x18

以此类推，当需要什么功能时，就把相应器件使能信号连接的位置设置为高电平，也就是设置为 1 ，可得指令集如下表。

![](https://mmbiz.qpic.cn/mmbiz_png/l7Q86F4y59sCKqS5ZD89NW7iaRI7GEHwV3icMRBBpkeqjATwAjLFDymCiaTB5liaqaiakoC7u8IarSTdibW3pmms2dHw/640?wx_fmt=png)CPU 指令集

下面我们以上例剖析具体执行过程。

#### 4.2 指令执行过程剖析

让我们开机，pc 从零开始运行。

![](https://mmbiz.qpic.cn/mmbiz_png/l7Q86F4y59sCKqS5ZD89NW7iaRI7GEHwVGiaKLa0CdRoZbhFKQ71ZAXdfukPp9J3mo06Z4HPNlYfVr3gSJ76yJgg/640?wx_fmt=png)指令之 0x18

*   上图中，当程序计数器为 0 时，访问指令 RAM 和数据 RAM 的第 0 个空间， 0x18 和 10 分别存入指令寄存器和数据寄存器。
    

*   指令 0x18 ，二进制 0b0001 1000，此为 Load 载入指令，分别指示 DR 寄存器和 AC 寄存器使能端 w 为 1。
    
*   数据 10 作为数据存入 DR 和 AC 寄存器
    

以上完成载入 10 的操作。

![](https://mmbiz.qpic.cn/mmbiz_png/l7Q86F4y59sCKqS5ZD89NW7iaRI7GEHwVmOjdJoSqiaXUjkaibDxamTVq6F9Qtlb1553DOHkJkNDDAK6euibwYPrFw/640?wx_fmt=png)指令之 0x19

*   当 pc 为 1 时，访问指令 RAM 和数据 RAM 的第 1 个空间，0x19 和 2 分别存入指令寄存器和数据寄存器。
    

*   指令 0x19，二进制 0b0001 1001，此为 Add 加法指令，分别指示：DR 寄存器保存数据；21 选择器被选择，输出加法器计算结果；结果被保存进 AC。
    
*   数据 2 作为数据存入 DR 和上一步  AC 内容 10 相加再存入 AC。
    

以上完成 10+2 的操作。

![](https://mmbiz.qpic.cn/mmbiz_png/l7Q86F4y59sCKqS5ZD89NW7iaRI7GEHwV5gRVEGVuicIo4CfJ8tSyBmHVGE4pp5c1CicxFh4WxiauRk9bM2Ryyw5KQ/640?wx_fmt=png)指令之 0x1d

*   pc 为 2 时，访问指令 RAM 和数据 RAM 的第 2 个空间，0x1d 和 3 分别存入指令寄存器和数据寄存器。
    

*   指令 0x1d，二进制 0b0001 1101，此为 Sub 减法指令，分别指示：DR 寄存器保存数据；加法器支持的减法启动；21 选择器被选择；运算结果被保存进 AC。
    
*   数据，3 作为数据存入 DR 和上一步 AC 结果 12 的差 9 存入 AC。
    

以上完成 12-3 的操作。

![](https://mmbiz.qpic.cn/mmbiz_png/l7Q86F4y59sCKqS5ZD89NW7iaRI7GEHwViaEovBVoMjB45FushLicbrKvLsHskbTY22moYrjVc9Fuj9FIYT7j6EOQ/640?wx_fmt=png)指令之 0x02

*   pc 为 3 时，访问指令 RAM 和数据 RAM 的第 3 个空间，0x02 存入指令寄存器。
    

*   指令 0x02，二进制 0b0000 0010，此为 Store 存储指令，此时，w 信号为 1，指示打开数据 RAM 的使能信号，这样 AC 寄存器中的 9 存入数据 RAM 的 3 位置中。
    
*   数据 0xff，由于 dr 为 0，所以数据寄存器不存入数据。
    

以上完成将 9 写入数据 RAM 位置 3 处的操作。

![](https://mmbiz.qpic.cn/mmbiz_png/l7Q86F4y59sCKqS5ZD89NW7iaRI7GEHwVicyCupIPH15ibpib90ds7Ot2sc9U4Prr37Y6GiaC3QA9CKzJOtibawOZKWw/640?wx_fmt=png)指令之 0x31

*   pc 为 4 时，访问指令 RAM 和数据 RAM 的第 4 个空间，0x31 存入指令寄存器。
    

*   若 pre 为 1，AC 为零，则 0x06 写入 pc 计算器，程序跳转到 pc 为 6 时执行，也就是最后一步命令 HLT 停机指令。
    
*   若 AC 不为零或 pre 不为 1，继续向下执行 pc+1 也就是 pc 为 5。
    
*   指令 0x31，二进制 0b0011 0001，此为 Jz 零跳转指令，指示根据 AC 结果是否为零及程序计数器置位信号 pre 是否为 1，来重置 pc 计数器。
    
*   数据，0x06 作为数据存入 DR。根据 pre 信号为 1 和 AC 为 0 否，重置 pc 计数器。
    

以上完成根据计算结果是否为零分别跳转倒不同位置的功能。

![](https://mmbiz.qpic.cn/mmbiz_png/l7Q86F4y59sCKqS5ZD89NW7iaRI7GEHwVwx99RFVHv8Mvt1oHjHkaiaEL3KwgUesm18lcwsiatqvFRePZCm5yOamQ/640?wx_fmt=png)指令之 0x30

*   pc 为 5 时，访问指令 RAM 和数据 RAM 的第 5 个空间，0x30 存入指令寄存器。
    

*   指令 0x30，二进制 0b0011 0000，此为无条件跳转指令 Jmp ，指示重置 pc 计数器。
    
*   数据， 0x02 作为数据存入 DR，根据 pre 信号，重置 pc 计数器。
    
*   0x02 指向地址的指令是 0x1d，这是减法指令，也就是继续进行 -3 操作。
    
*   此时 pc 指向 0x02，重新进行减法计算。
    

以上完成继续 -3 的操作。

![](https://mmbiz.qpic.cn/mmbiz_png/l7Q86F4y59sCKqS5ZD89NW7iaRI7GEHwVwjmLCKobwlDRhVs36ZndorcuOayibOicP3yJibwj0Lk4QrADxEiaXia8Fog/640?wx_fmt=png)指令之 0x00

*   pc 为 6 时，访问指令 RAM 和数据 RAM 的第 6 个空间，0x00 存入指令寄存器。
    

*   指令 0x00，二进制 0b0000 0000，此为 Hlt 指令，指示停机。
    
*   此命令数据无意义。
    
*   本指令地址只能从 pc 为 4 时跳转过来。
    

其实看懂以上执行流程和例子，基本就懂 CPU 的基本工作原理了。下面我们用 Python 语言来实现这些器件吧。

五、 Python 实现 CPU 各组成部分
----------------------

#### 5.1 RAM 存储器

我们用 list 来存储数据。这是一个很简单和直接的设计。

```
ramc = [0x18, 0x19, 0x1d, 0x02, 0x31, 0x30, 0x00]


```

对存储器的读写，根据 pc 指针来，`ramc[pc]=data` 表示写入内存，读就是 `ramc[pc]`。

#### 5.2 Adder 加法器

```
def adder(a=0, b=0, ci=0, sub=0):
    return a-b+ci if sub == 1 else a+b+ci


```

真正的加法器使用逻辑门，相当于一堆开关按某种关系堆叠在一起，这里我们用高级语言模拟，极大简化了实现。这个加法器实现了 a 和 b 的加法，同时 ci 表示进位，sub 表示减法。

#### 5.3 Register 寄存器

寄存器采用 Python 的闭包概念来设计，这是为了用自由变量记住寄存器上次的状态。当我们用 `AC = register()` 调用时，AC 相相当于返回的内部函数 register_inner，此时 temp 作为自由变量和 register_inner 同属一个闭包。所以此后对 temp 变量读、写都是一个持久的变量。相当于维持住了状态。

w 信号为 1 时写入，相当于寄存器使能端 w。

```
def register():
    temp = 0

    def register_inner(data=0, w=0):
        nonlocal temp
        if w == 1:
            temp = data
        return temp
    return register_inner


```

> “
> 
> 真实 CPU 设计当中，如何设计寄存器是一门大学问。即使在微机原理课程粗浅的 CPU 模型学习中，理解继电器和 三极管能记忆，也需要费一番心思。本文用高级语言模拟底层硬件，我们只能再兜兜转转一次，所以此处需要深刻理解闭包概念。
> 
> ”

#### 5.4 8bit 21 选择器

21 选择器是在 sel 端为 1 时，返回 b 。当 sel 为零时，返回 a。也就是两个输入端选择一个作为输出。

```
def b8_21selector(a=0, b=0, sel=0):
    return a if sel == 0 else b


```

六、集成 CPU
--------

当我们集成 CPU 各部件时，首先将各部件新建出来，然后进行初始化，最后将 pc 置零，开始无限循环。

循环过程中，首先将程序指令 RAM 中的数据写入指令寄存器，根据指令寄存器解码各控制信号，此后操作都是在指令控制信号控制下进行。

首先如果 IR 指令寄存器中是 HLt 停机指令的话，那么系统 Break。否则根据 dr 决定是否将数据信号写入 DR 数据寄存器。

对加法器的操作，是自动的，它的一个输入是 AC 累加器存器，另一个输入是 DR 数据寄存器，同时受到 sub 减法控制信号的控制。

加法运算器运算后，结果传到 21 选择器，同数据总线上直接过来的数据一块，等待 s21 信号选择，再根据 ac 信号存进 AC 累加寄存器，以备下一计算。

zf 作为零标志位寄存器，如果 AC 累加器存起结果为零的话，则 zf 为 1。此时如果 pre 为 1 的话，那么就可以将 pc 设置为 DR 数据寄存器的值，实现了运算结果为零跳转功能。否则继续向下执行。

总体集成后，代码如下：

```
def adder(a=0, b=0, ci=0, sub=0):
    return a-b+ci if sub == 1 else a+b+ci
def b8_21selector(a=0, b=0, sel=0):
    return a if sel == 0 else b
def register():
    temp = 0
    def register_inner(data=0, w=0):
        nonlocal temp
        if w == 1:
            temp = data
        return temp
    return register_inner
def int2bin(data=0, length=8, tuple_=1, string=0, humanOrder=0):
    #辅助函数，整数转换为二进制字符串或者元祖。
    r = bin(data)[2:].zfill(length)
    r = r[::-1] if humanOrder == 0 else r
    return r if string == 1 else tuple(int(c) for c in r)
def cpu():
    pc = 0 # pc 计数器从 0 开始，无限循环。
    IR, DR, AC = register(), register(), register() # 新建寄存器
    ramc = [0x18, 0x19, 0x1d, 0x02, 0x31, 0x30, 0x00] # 初始化代码
    ramd = [10, 2, 3, 0xff, 0x06, 0x02] # 初始化数据

    IR(0, w=1) # 初始化寄存器
    DR(0, w=1)
    AC(0, w=1)
    while True:
        IR(ramc[pc], w=1) # 指令读写
        *_, pre, dr, ac, sub, w, s21 = int2bin(IR(), humanOrder=1) # 指令解码
        if IR() == 0:
            break # HLT信号
        DR(ramd[pc], w=dr) # 数据读写
        r = adder(AC(), DR(), sub=sub) # 加法器自动加法
        AC(b8_21selector(DR(), r, s21), w=ac) # 选择器选择后，累加寄存器读写
        ramd[pc] = AC() if w else ramd[pc] # 根据 w 信号，数据写入 RAM
        zf = (AC() == 0) # 零标志位寄存器
        pc = DR() if (pre == 1 and zf == True and s21 == 1) else pc + 1 # Jz 指令跳转
        pc = DR() if (pre == 1 and s21 == 0) else pc # 无条件跳转 Jmp
        print(AC()) 
if __name__ == '__main__':
    cpu()


```

可见输出结果：`10,12,9,9,9,9,6,6,6,6,3,3,3,3,0,0,0`，程序工作正常，CPU 工作了！

七、为 CPU 编程，体会上古程序员 工作流程
-----------------------

下面我们给我们的玩具 CPU 编写一个从 5 减 1，直到为 0 的程序：首先需要载入 5，然后减去 1，判断是否为零，为零跳转到停机，不为零继续跳转到减 1 处。

代码和数据分别写：

```
    ramc = [0x18, 0x1d, 0x31, 0x30, 0x00]
    ramd = [5,    1,    0x04, 0x01]


```

程序输出：

```
5,4,4,4,3,3,3,2,2,2,1,1,1,0,0


```

工作正常！

> “
> 
> 如果我们把这些数据转换为二进制，明显是 8 bit 信息，每单元 8 个小孔顺序镂刻在纸带上，就可以拿到上古计算机上运行了。这就是第一代程序员们干的事情。
> 
> ”

八、总结
----

理解 CPU 工作原理，重要的是理解 pc 不停地自增地址，顺序执行程序指令。当遇到跳转指令时，会将 pc 重置为新地址。在顺序执行程序指令的过程中，每一步都是解析程序指令、产生控制信号，进而控制所有 CPU 相关器件的工作状态，产生程序计算结果，保存进各寄存器或者 RAM 中。

从宏观上，CPU 工作原理是读取内存数据，在 ALU 中完成计算，然后保存进内存，输入输出系统完成了同其他外设交互；从中观上看，CPU 工作原理就是本文讲述的 pc 从 0 开始，读取程序指令寄存器，然后解析指令，控制各部件工作的具体过程；

从微观上看，pc 程序计数器、ALU 数字逻辑运算单元，RAM 存储器在内的所有 CPU 相关部件，其实都是一个个三极管，这些三极管在电流作用下导通或者截止，完成了数字逻辑运算、保持记忆状态、产生脉冲信号的所有功能。

本文是从中观层次构建、模拟 CPU，使用 40 行 Python 代码实现了一个简单的玩具级 CPU。使他完成加减法运算，且具备读写内存、跳转、条件跳转的功能。全文较干，感谢阅读！
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650528718&idx=3&sn=577b24dcffd665fbda737566d7656207&chksm=83babe2ab4cd373c87a521421f39d374109be795a5a94b776f7e8860df76651f0a716f57dc59&mpshare=1&scene=1&srcid=1205Jri3vdZ5jDCwzGnfR9ze&sharer_sharetime=1638683776281&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> **原文来源 ：Freebuf**

1 前言
----

使用 Shell 脚本在 Linux 服务器上能够控制、毁坏或者获取任何东西，通过一些巧妙的攻击方法黑客可能会获取巨大的价值，但大多数攻击也留下踪迹。当然，这些踪迹也可通过 Shell 脚本等方法来隐藏。

寻找攻击证据就从攻击者留下的这些痕迹开始，如文件的修改日期。每一个 Linux 文件系统中的每个文件都保存着修改日期。系统管理员发现文件的最近修改时间，便提示他们系统受到攻击，采取行动锁定系统。然而幸运的是，修改时间不是绝对可靠的记录，修改时间本身可以被欺骗或修改，通过编写 Shell 脚本，攻击者可将备份和恢复修改时间的过程自动化。

2 操作步骤
------

### 第一步：查看和操作时间戳

多数 Linux 系统中包含一些允许我们快速查看和修改时间戳的工具，其中最具影响的当数 “Touch”，它允许我们创建新文件、更新文件 / 文件组最后一次被“touched” 的时间。

```
touch file
```

若该文件不存在, 运行上面的命令将创建一个名为 “file” 的新文件；若它已经存在，该命令将会更新修改日期为当前系统时间。我们也可以使用一个通配符，如下面的字符串。

```
touch *
```

这个命令将更新它运行的文件夹中的每个文件的时间戳。在创建和修改文件之后，有几种方法可以查看它的详细信息，第一个使用的为 “stat” 命令。

```
stat file
```

![](https://mmbiz.qpic.cn/mmbiz_png/cbCLgfJZibpoYibFzXBIS1Jl3loKrCpvQ602e3qsUSuI0ahxB28ELSasGHwlbjnj1VzEZU1fPGeCIPe9HM6QoKwA/640?wx_fmt=png)

运行 stat 会返回一些关于文件的信息，包含访问、修改或更新时间戳。针对一批文件可使用 ls 参数查看各文件的时间戳，使用 “-l” 或者“long”，该命令会列出文件详细信息，包含输出时间戳。

```
ls –l
```

![](https://mmbiz.qpic.cn/mmbiz_png/cbCLgfJZibpoYibFzXBIS1Jl3loKrCpvQ6VFCI1GKCrlvpaS9qyiciarKObIPWUTLZs2rAL53tC4ZN7GQH4Lt0da6w/640?wx_fmt=png)

现在就可以设置当前时间戳并查看已经设置的时间戳，也可使用 touch 来定义一个自定义时间戳，可使用 “d” 标志，用 yyyy-mm-dd 格式定义日期，紧随其后设置时间的小时、分钟及秒，如下：

```
touch -d"2001-01-01 20:00:00" file
```

通过 ls 命令来确认修改信息：

```
ls -l file
```

![](https://mmbiz.qpic.cn/mmbiz_png/cbCLgfJZibpoYibFzXBIS1Jl3loKrCpvQ6XvPzyxC7tjYgrTaFl6aq310YQaOhJ9ph8coJbpFZMBhiccTtS793xpA/640?wx_fmt=png)

这种方法适用于修改个别时间戳，对于隐藏服务器上的操作痕迹，这个方法不太奏效，可以使用 shell 脚本将该过程自动化。

### 步骤二：组织 Shell 脚本

在开始编写脚本之前需要考虑清楚需要执行哪些过程。为了在服务器上隐藏痕迹，攻击者需要将文件夹的原始时间戳写入一个文件，同时能够在我们进行任何修改设置之后还能回到原始文件。

这两个不同的功能会根据用户的输入或者参数的不同而触发，脚本会根据这些参数执行相应的功能，同时我们需要有一种方法来处理错误。根据用户的输入将会进行三种可能的操作：

我们可以使用嵌套语句 if/or 语句来创建脚本，也可以根据条件将每个函数分配给自己的 “if” 语句，可选择在文本编辑器或者 nano 中开始编写脚本。

### 步骤三：开始脚本

从命令行启动 nano 并创建一个名为 “timestamps.sh” 的脚本，命令如下：

```
nano timestamps.sh
```

然后进行下列命令：

```
#!/bin/bash
if [ $# -eq 0 ];then
echo “Use asave (-s) or restore (-r) parameter.”
exit 1
fi
```

![](https://mmbiz.qpic.cn/mmbiz_png/cbCLgfJZibpoYibFzXBIS1Jl3loKrCpvQ6g7k65Av8aOqQUsHiaibcoUatCpBFbjWb29AUjpsfYgMoJiaC1DrlgOsDg/640?wx_fmt=png)

在 nano 中按下 Ctrl + O 保存这个文件，通过 chmod 命令将它标记为可运行的脚本。

```
chmod +x timestamps.sh
```

然后运行脚本，测试无参数时返回错误信息的功能。如果脚本返回我们的 echo 语句，我们就可以继续下一个条件了。

```
./timestamps.sh
```

![](https://mmbiz.qpic.cn/mmbiz_png/cbCLgfJZibpoYibFzXBIS1Jl3loKrCpvQ6ictEZRSP2H7pqAppytLKqnaVo7QffTDcicoRcHgr1t3xB1IfaYZvsvSw/640?wx_fmt=png)

### 步骤四：将时间戳写入文件

定义 if 语句的条件，“-s” 表示执行保存功能：

```
if [ $1 ="-s" ] ; then
fi
```

当然，需要检查计划保存的时间戳文件是否存在，如果存在，我们可以删除它（名为 timestamps 的文件），避免重复或错误的输入，使用下面的命令：

```
rm -f timestamps;
```

然后使用 “ls” 命令列出所有文件和它的修改时间，可将其输出到另一个程序，如 sed，以帮助我们稍后清理这个输入。

```
ls –l
```

通常会出现下面的显示结果：

```
-rw-r--r-- 1 user user 0 Jan 1 2017 file
```

为了保存时间戳，我们只需要年、月、日及文件名，下面命令可以清除 “Jan” 之前的信息：

```
ls -l file | sed 's/^.*Jan/Jan/p'
```

这样显示的就是我们程序需要的信息，只是需要修改月份格式为数字格式：

```
ls -l file | sed 's/^.*Jan/01/p'
```

将所有月份都替换为数字：

```
ls -l | sed -n 's/^.*Jan/01/p;s/^.*Feb/02/p;s/^.*Mar/03/p;s/^.*Apr/04/p;s/^.*May/05/p;s/^.*Jun/06/p;s/^.*Jul/07/p;s/^.*Aug/08/p;s/^.*Sep/09/p;s/^.*Oct/10/p;s/^.*Nov/11/p;s/^.*Dec/12/p;'
```

在一个文件夹中运行我们会看到如下图所示的结果：

![](https://mmbiz.qpic.cn/mmbiz_png/cbCLgfJZibpoYibFzXBIS1Jl3loKrCpvQ6JWZmztTMewUPVcfhwFQQA0mxjdC1lNzoBzWFxicOkiaKWygYt1sU4Qdg/640?wx_fmt=png)

然后将输出结果通过 “>>” 发送到名为 “timestamps” 的文件中：

```
do echo $x | ls -l | sed -n 's/^.*Jan/01/p;s/^.*Feb/02/p;s/^.*Mar/03/p;s/^.*Apr/04/p;s/^.*May/05/p;s/^.*Jun/06/p;s/^.*Jul/07/p;s/^.*Aug/08/p;s/^.*Sep/09/p;s/^.*Oct/10/p;s/^.*Nov/11/p;s/^.*Dec/12/p;' >> timestamps
```

此脚本的前两个操作就完成了，显示结果如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/cbCLgfJZibpoYibFzXBIS1Jl3loKrCpvQ63ncsWNX3IRiclJunLQicBkoceR52dNq7013YdxZ6wSBn7BRr8dpPWbsw/640?wx_fmt=png)

下面可用 “-s” 标示测试脚本，用 cat 检查保存的信息：

```
./timestamps.sh –s
cat timestamps
```

![](https://mmbiz.qpic.cn/mmbiz_png/cbCLgfJZibpoYibFzXBIS1Jl3loKrCpvQ6iclUPcblMuSQRsz7ICgLicPl3iaickXBI7oPSuAAh9WZzlhA6POy9Pgicdw/640?wx_fmt=png)

### 步骤五：恢复文件的时间戳

在保存好原始时间戳后，需要恢复时间戳让别人觉察不到文件被修改过，可使用下面命令：

```
if $1 = "-r" ; then
fi
```

然后使用下面命令，转发文本文件的内容，并一行一行运行：

```
cat timestamps |while read line
do
done
```

然后再分配一些变量让文件数据的使用更简单：

```
MONTH=$(echo $line | cut -f1 -d\ );
DAY=$(echo $line| cut -f2 -d\ );
FILENAME=$(echo $line | cut -f4 -d\ );
YEAR=$(echo $line | cut -f3 -d\ )
```

虽然这四个变量在保存的时间戳文件中是一致的，但是如果时间戳是在过去一年中发生的，它只会显示时间而不是年份。如果需要确定当前年份，我们可以分配为写脚本的年份，也可以从系统中返回年份，使用 cal 命令可以查看日历。

![](https://mmbiz.qpic.cn/mmbiz_png/cbCLgfJZibpoYibFzXBIS1Jl3loKrCpvQ6lauq0N0ibbcG3Mo7ibaN5O6PB0j0L7vnZaib6ibWibHPR9iaruGK27PnaLHA/640?wx_fmt=png)

然后检索第一行，只让显示想要的年份信息：

```
CURRENTYEAR=$(cal | head -1 | cut -f6- -d\ | sed 's/ //g')
```

![](https://mmbiz.qpic.cn/mmbiz_png/cbCLgfJZibpoYibFzXBIS1Jl3loKrCpvQ6yUnfbianCGRABV6xYQYiblPLick73yVY3b6111aIRbw7lBD4NoXXhxqnQ/640?wx_fmt=png)

定义了所有变量之后可以使用 “if else” 语句，根据格式化的日期更新文件的时间戳，使用 touch 语法：

```
touch -d "2001-01-01 20:00:00" file
```

由于每个时间都包含冒号，因此可使用下面的 “ifelse” 语句完成操作，整体操作如下图所示：

```
if [ $YEAR == *:* ]; then
touch -d $CURRENTYEAR-$MONTH-$DAY\ $YEAR:00 $FILENAME;
else
touch -d ""$YEAR-$MONTH-$DAY"" $FILENAME;
fi
```

![](https://mmbiz.qpic.cn/mmbiz_png/cbCLgfJZibpoYibFzXBIS1Jl3loKrCpvQ64UrmbPduXgU2icJft2h8p30hiaEKHR5PtMoSma1XAZYgQD1riamNV1ryw/640?wx_fmt=png)

### 步骤六：使用脚本

使用的命令主要有以下几个：

*   ./timestamps.sh –s   保存文件时间戳
    
*   touch -d “2050-10-12 10:00:00″ *   修改目录下的所有文件时间戳
    
*   ls –a   确认修改的文件
    
*   ./timestamps.sh –r   恢复文件原始时间戳
    

最后可以再次运行 “ls -a” 来查看文件的时间戳是否和之前备份的时间戳一致，整个的脚本就执行完成了，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/cbCLgfJZibpoYibFzXBIS1Jl3loKrCpvQ6UsfYLqWf5Vk5LmPleUl6oibsZhQTGsvtthmofWicvVRMzR5ibd26ds6GA/640?wx_fmt=png)

3 总结
----

该脚本只是用来清除攻击服务器之后遗留的一些痕迹。为了隐藏痕迹，黑客在针对服务器实施具体的攻击时，必须仔细考虑使用的每一个方法，以及入侵服务器之后如何隐藏自己的痕迹。

通过上面的介绍我们了解到，时间戳也是 “会撒谎的”，因此系统管理员必须意识到他们的许多日志和保护措施是可以被操纵的，虽然看起来好像没有异常。
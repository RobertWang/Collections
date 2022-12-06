> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.linuxmi.com](https://www.linuxmi.com/bash-programs-writing.html)

> 您可以使用 Bash 脚本自动执行各种任务。掌握基础知识并开始您的 Bash 脚本之旅。 Bash 脚本可用于

您可以使用 Bash 脚本自动执行各种任务。掌握基础知识并开始您的 Bash 脚本之旅。

Bash 脚本可用于自动化任务，您会发现它们非常适合构建简单的命令行应用程序。Bash shell 解释 Bash 脚本，因此您无需安装任何依赖项即可编写和运行它们。Bash 脚本也是可移植的，因为大多数基于 Unix 的操作系统都使用相同的 shell 解释器。

每个开发人员都必须具备 Bash 脚本知识，尤其是在使用基于 Unix 的系统时。

Bash 中的变量
---------

Bash 变量区分大小写。要声明变量，请使用等号 **(=)**，名称在左侧，值在右侧：

```
STATE=LinuxMi

```

此声明分配给 **STATE** 的值是一个单词。如果您的值中需要空格，请在其周围使用引号：

```
STATE="Ubuntu Linux"

```

您需要使用美元符号 **($)** 前缀来引用其他变量或语句中的变量：

```
STATE=LinuxMi
LOCATION="My Site is $STATE"

```

在 Bash 中打印值
-----------

有几种方法可以在 Bash 中打印变量。您可以使用 **echo** 命令进行基本输出，或使用 C 风格的 **printf** 命令进行字符串格式化。

```
STATE=LinuxMi
LOCATION="My Site is $STATE"
echo $LOCATION

```

声明 **STATE** 变量后，此脚本通过引用 STATE 来定义 **LOCATION 。**如果 then 使用 echo 打印 LOCATION 变量的最终值。

![](https://www.linuxmi.com/wp-content/uploads/2022/09/1-1.png)

**printf** 关键字允许您使用格式化动词来输出数据。字符串格式化动词类似于 C 和 Go 中的动词，但动词有限。

<table><thead><tr><th>动词</th><th>功能性</th></tr></thead><tbody><tr><td>％C</td><td>打印单个字符</td></tr><tr><td>%o</td><td>打印八进制</td></tr><tr><td>%s</td><td>打印字符串，独立于大小写</td></tr><tr><td>％X</td><td>打印小写十六进制</td></tr><tr><td>％X</td><td>打印大写十六进制</td></tr><tr><td>%d</td><td>打印整数</td></tr><tr><td>%e</td><td>以小写形式打印科学概念浮点数</td></tr><tr><td>%E</td><td>以大写形式打印科学概念浮点数</td></tr><tr><td>％F</td><td>打印浮点数</td></tr><tr><td>%%</td><td>打印一个百分比符号。</td></tr></tbody></table>

这是一个使用带有 **print** 关键字的动词的示例。

```
STATE=LinuxMi.com
printf "My Site is %s" $STATE

```

![](https://www.linuxmi.com/wp-content/uploads/2022/09/2-1.png)

**printf** 函数将在 **%s** 动词的位置替换 **STATE** 变量，输出将是 “My Location is Lagos”。

您可以在 Bash 中使用井号或井号 ( **#** ) 符号进行注释。shell 会自动忽略注释。

```
#!/bin/bash




```

没有多行注释。大多数 IDE 和文本编辑器允许您使用 Ctrl/Command + 正斜杠 (/) 快捷方式进行注释。您应该能够使用快捷方式创建多个单行注释。

在 Bash 中接收用户输入
--------------

与许多其他编程语言一样，您可以在 Bash 中接收用户输入，以使您的程序 / 脚本更具交互性。您可以使用 **read** 命令来请求用户的输入。

```
read response


```

在这种情况下，**response** 变量将保存用户在交付时的输入。

```
echo "What do you want ?: "
read response
echo $response


```

在上面的示例中，用户输入请求将位于新行上。

![](https://www.linuxmi.com/wp-content/uploads/2022/09/3-1.png)

您可以将 **-n** 标志添加到 **echo** print 语句以保留用户输入输入的行。

```
echo -n "What do you want."
read response
echo $response

```

![](https://www.linuxmi.com/wp-content/uploads/2022/09/4.png)
-------------------------------------------------------------

在 Bash 中声明数组
------------

Bash 中的数组就像大多数语言一样。您可以通过在括号中指定元素来在 Bash 中声明一个数组变量。

```
Countries=('Ubuntu' 'Debian' 'CentOS', "openSUSE", "Linuxmi.com")


```

通过引用变量名访问数组将获取第一个元素。您可以使用星号作为索引来访问整个数组。

```
echo ${Countries[*]}


```

您还可以指定数组的索引来访问特定元素。数组的索引从零开始。

```
echo "${Countries[4]}"

```

![](https://www.linuxmi.com/wp-content/uploads/2022/09/5.png)
-------------------------------------------------------------

Bash 中的条件语句
-----------

Bash 为程序中的决策提供条件。

这是 Bash 中 if-else 语句的剖析。您必须使用分号来指定条件的结束。

```
if [[ condition ]]; then
   echo statement1
elif [[condition ]]; then
   echo statement2
else [[condition ]]; then
   echo statement3
fi


```

您必须以 **fi** 关键字结束每个 **if** 语句。

```
if [ 1 == 2 ]; then
   echo one 
elif [ 2 == 3 ]; then 
   echo two
else [ 4 > 3 ]; 
   echo "correct, 3"
fi


```

您可以使用 **case** 关键字在 Bash 程序中使用 case 语句。您必须指定模式，然后在语句之前加上括号。

```
NAME=LinuxMi
case $NAME in
  "Debian") 
    echo "Debian是目前世界最大的非商业性Linux发行版之一" 
    ;; 
  "LinuxMi" | "Ubuntu")
    echo  "openSUSE"
    ;;
  "CentOS" | "oracle linux")
    echo  "linux"
    ;;
  *) 
    echo "linuxmi.com" 
    ;;
esac 


```

您可以使用星号 (*) 符号作为模式定义默认大小写。case 语句必须以 **esac** 关键字结尾。

Bash 中的循环
---------

根据您的需要，您可以使用 while 循环、范围 for 循环或 C 风格的 for 循环进行重复操作。

这是 C 风格的 for 循环的示例。For 循环必须以 **done** 关键字结尾，并且您必须以分号后跟 **do** 关键字结束 for 语句。

```
for ((a = 0 ; a < 10 ; a+2)); do
  echo $a
done


```

对于处理文件和许多其他操作，for 循环的范围很方便。您需要将 **in** 关键字与范围 for 循环一起使用。

```
for i in {1..7}; do
    echo $1
done


```

![](https://www.linuxmi.com/wp-content/uploads/2022/09/6.png)

这是一个简单的无限循环，用于演示 Bash **while** 循环的实际作用。

```
linuxmi=1
while [ 1 -le 5 ] 
do
  echo $linuxmi
done


```

条件语句中的 **-le** 是小于的二元运算符。

Bash 中的函数
---------

在 Bash 中声明函数不需要关键字。您可以使用名称声明函数，然后在函数体之前加上括号。

```
print_working_directory() {
  echo $PWD  
}
echo "当前的目录是 $(print_working_directory)"


```

![](https://www.linuxmi.com/wp-content/uploads/2022/09/7.png)

函数可以在 Bash 中返回变量。您所需要的只是 **return** 关键字。

```
print_working_directory() {
  return $PWD
}


```

print_working_directory 函数返回**文件**的工作目录。

你可以用其他语言编写 Shell 脚本
-------------------

Bash 并不是您可以用来与操作系统的 shell 交互或构建命令行应用程序的唯一语言。您可以使用许多其他语言，例如 Go、Python、Ruby 和 Rust。

许多操作系统都预装了 Python3，而 Python 是一种流行的语言。如果您需要比 Bash 脚本提供的更多功能，请考虑使用 Python。
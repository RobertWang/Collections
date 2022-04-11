> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.linuxidc.com](https://www.linuxidc.com/Linux/2018-11/155618.htm)

> 除了基本的获取脚本执行时的传入参数外, 还有更便捷的语法糖: 参数默认值, 自动赋值.

简介
--

除了基本的获取脚本执行时的传入参数外, 还有更便捷的语法糖: 参数默认值, 自动赋值.

基本传参
----

先来一个示例:

```
#!/bin/sh
echo 参数0: $0;
echo 参数1: $1;
echo 参数2: $2;
echo 参数3: $3;
echo 参数4: $4;

```

执行测试脚本

```
linuxidc@linuxidc:~/linuxidc.com$ sh linuxidc.sh a b c d

```

```
所有参数: a b c d
参数0: linuxidc.sh
参数1: a
参数2: b
参数3: c
参数4: d

```

![](https://www.linuxidc.com/upload/2018_11/18113020348293.png)

<table><thead><tr><th>参数处理</th><th>说明</th></tr></thead><tbody><tr><td>$#</td><td>传递到脚本的参数个数</td></tr><tr><td>$*</td><td>以一个单字符串显示所有向脚本传递的参数。 如 "$*" 用「"」括起来的情况、以"$1 $2 … $n" 的形式输出所有参数。</td></tr><tr><td>$$</td><td>脚本运行的当前进程 ID 号</td></tr><tr><td>$!</td><td>后台运行的最后一个进程的 ID 号</td></tr><tr><td>$@</td><td>与 $* 相同，但是使用时加引号，并在引号中返回每个参数。 如 "$@" 用「"」括起来的情况、以"$1""$2" … "$n" 的形式输出所有参数。</td></tr><tr><td>$-</td><td>显示 Shell 使用的当前选项，与 set 命令功能相同。</td></tr><tr><td>$?</td><td>显示最后命令的退出状态。0 表示没有错误，其他任何值表明有错误。</td></tr></tbody></table>

### `$*` 与 `$@` 区别

$* 与 $@ 区别：

*   相同点：都是引用所有参数。
*   不同点：只有在双引号中体现出来。假设在脚本运行时写了三个参数 1、2、3，，则 "*" 等价于 "1 2 3"（传递了一个参数），而 "@" 等价于 "1" "2" "3"（传递了三个参数）。

```
#!/bin/bash

echo "-- \$* 演示 ---"
for i in "$*"; do
    echo $i
done

echo "-- \$@ 演示 ---"
for i in "$@"; do
    echo $i
done

```

执行脚本，输出结果如下所示：

```
$ chmod +x linuxidc.sh 
$ ./linuxidc.sh 1 2 3
-- $* 演示 ---
1 2 3
-- $@ 演示 ---
1
2
3

```

![](https://www.linuxidc.com/upload/2018_11/18113020326955.png)

默认参数 (变量默认值)
------------

### `if` 繁琐方式

```
if [ ! $1 ]; then
    $1='default'
fi

```

### `-` 变量为 null

取默认值

*   变量 为 null

`${vari-defaultValue}`

实践

```
[root@linuxidc /]# unset name
[root@linuxidc /]# echo ${name}

[root@linuxidc /]# echo ${name-linuxmi}
linuxmi
[root@linuxidc /]# 
[root@linuxidc /]# echo ${name-linuxmi}

[root@linuxidc /]# echo ${name}

[root@linuxidc /]#

```

![](https://www.linuxidc.com/upload/2018_11/18113020384766.png)

### `=` 变量为 null 时, 同时改变变量值

```
[root@linuxidc /]# unset name
[root@linuxidc /]# echo ${name=linuxmi}
linuxmi
[root@linuxidc /]# echo $name
linuxmi
[root@linuxidc /]# 
[root@linuxidc /]# echo ${name=linuxmi}

[root@linuxidc /]#

```

### `:-` 变量为 null 或 空字符串

取默认值

*   变量为 null
*   变量为空字符串

`${vari:-defaultValue}`

### `:=` 变量为 null 或 空字符串, 同时改变变量值

`{$vari:=defaultValue}`

测试 null

```
[root@linuxidc /]# unset name

[root@linuxidc /]# echo ${name:=linuxmi}
linuxmi
[root@linuxidc /]# echo ${name}
linuxmi
[root@linuxidc /]#

```

测试 空字符串

```
[root@linuxidc /]# 
[root@linuxidc /]# echo ${name:=linuxmi}
linuxmi
[root@linuxidc /]# echo $name
linuxmi

```

### `:?` 变量为 null 或 空字符串时报错并退出

```
[root@linuxidc /]# unset name
[root@linuxidc /]# echo ${name:?linuxmi}
-bash: name: linuxmi
[root@linuxidc /]# 
[root@linuxidc /]# echo ${name:?linuxmi}
-bash: name: linuxmi
[root@linuxidc /]# 
[root@linuxidc /]# echo ${name:?linuxmi}
guest


```

### `:+` 变量不为空时使用默认值

与 `:-` 相反

```
[root@linuxidc /]# 
[root@linuxidc /]# echo ${name:+linuxmi}
linuxmi
[root@linuxidc /]# echo $name
guest


```

Linux 公社的 RSS 地址：[https://www.linuxidc.com/rssFeed.aspx](https://www.linuxidc.com/rssFeed.aspx)

**本文永久更新链接地址**：[https://www.linuxidc.com/Linux/2018-11/155618.htm](https://www.linuxidc.com/Linux/2018-11/155618.htm)

[![](https://www.linuxidc.com/linuxfile/logo.gif)](http://www.linuxidc.com/)
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1643088605&ver=3579&signature=Cy2aGqMwUA5ZlEd4I9PZHIDy7uhZldvGWlmR3Gdf5VY96ElEgdiadUky-SDNECqmry06pNzKvLuAiD2yAhNYc52jqehnavqN8OAc2*iDkF-z9GAF8i26bdlRQEVOyuP2&new=1)

数组是一个包含多个值的变量，这些值可以是相同类型或不同类型。没有数组大小限制，也没有要求成员变量被连续索引或连续分配的限制。数组索引从 0 开始。

1. 声明一个数组并赋值

  

  

在 bash 中，使用以下格式的变量时会自动创建数组：

```
name[index]=value

```

name 是数组的名字。

index 可以是任何数字或表达式，值必须等于或大于零。

要访问数组元素，请使用大括号，例如 ${name[index]}。下面是访问 Unix 数组中的第二个元素, 以为数组索引从 0 开始，所以`Unix[1]`就是第二个元素了。

```
[root@localhost ~]# cat arraymanip.sh
#! /bin/bash
Unix[0]='Debian'
Unix[1]='Red hat'
Unix[2]='Ubuntu'
Unix[3]='Suse'

echo ${Unix[1]}

```

执行脚本，以下是输出内容：

```
[root@localhost ~]# ./arraymain.sh 
Red hat

```

![](https://mmbiz.qpic.cn/mmbiz_png/K0TMNq37VN0UqYH7fM2FFCTyebAOaRiaA88EpH6WWIW8wG81pJXyHYicRfvAib6qD9xj0EBWXMLEiaG572Piaew5LqQ/640?wx_fmt=png)

2. 在声明期间初始化数组

  

  

不必单独初始化数组的每个元素，可以通过使用括号指定元素列表（由空格分隔）来声明和初始化数组。下面是语法：

```
declare -a arrayname=(element1 element2 element3)

```

如果元素具有空格字符，需要使用引号：

```
[root@localhost ~]# cat arraymain2.sh 

#! /bin/bash
declare -a Unix=('Debian' 'Red hat' 'Ubuntu' 'Suse' 'Fedora')

echo ${Unix[4]}

```

下面是输出结果：

```
[root@localhost ~]# ./arraymain2.sh 
Fedora

```

![](https://mmbiz.qpic.cn/mmbiz_png/K0TMNq37VN0UqYH7fM2FFCTyebAOaRiaAtLbjdFAEzE1LA8ibxqTGeO7kMsZlywurgzSYD0gsJsicCicVYTjcEsO2A/640?wx_fmt=png)

  
`declare -a`声明一个数组，括号中的所有元素都是数组的元素。

3. 打印整个数组

  

  

有多种方法可以打印整个数组。如果索引号是 `@ 或者 *` ，则引用数组的所有成员。可以使用 bash shell 中的循环语句遍历数组元素并进行打印。

```
[root@localhost ~]# cat arraymain.sh 
#! /bin/bash
Unix[0]='Debian'
Unix[1]='Red hat'
Unix[2]='Ubuntu'
Unix[3]='Suse'

echo ${Unix[@]}

```

下面是输出结果：

```
[root@localhost ~]# ./arraymain.sh 
Debian Red hat Ubuntu Suse

```

![](https://mmbiz.qpic.cn/mmbiz_png/K0TMNq37VN0UqYH7fM2FFCTyebAOaRiaAJeCOEGLoddxUIr69dzgzFWvgcmyWXglDjJ2yKjiatlatBMcMOcfvxLA/640?wx_fmt=png)

4. 获取数组的长度

  

  

可以使用`$和#`的特殊参数来获取数组的长度。`${#arrayname[@]}` 可以获取数组长度。

```
[root@localhost ~]# cat arraymain2.sh 
#! /bin/bash
declare -a Unix=('Debian' 'Red hat' 'Ubuntu' 'Suse' 'Fedora')

echo ${#Unix[@]} #数组中元素的数量
echo ${#Unix}  #数组中第一个元素的字符数。

```

下面是输出，可以看到第一行输出参数为 5 个。第二行输出第一个元素的字符数量是 6 个。

```
[root@localhost ~]# ./arraymain2.sh 
5
6

```

![](https://mmbiz.qpic.cn/mmbiz_png/K0TMNq37VN0UqYH7fM2FFCTyebAOaRiaA0BX3EPgrRlXYjwDh9NGN2BYHDgKOsbmOCN0AWuXlYNjXBaYna2fcSw/640?wx_fmt=png)

5. 数组中某个元素的长度

  

  

${#arrayname[n]} 给出数组中的 n 个元素的长度。

```
[root@localhost ~]# cat arraymain.sh 
#! /bin/bash
Unix[0]='Debian'
Unix[1]='Red hat'
Unix[2]='Ubuntu'
Unix[3]='Suse'

echo ${#Unix[3]} #位于索引3的元素的长度。

```

以下是输出，可以看到输出 index 为 3 的值为 "Suse" 的字符长度是 4。

```
[root@localhost ~]# ./arraymain.sh 
4

```

![](https://mmbiz.qpic.cn/mmbiz_png/K0TMNq37VN0UqYH7fM2FFCTyebAOaRiaALZ0LoEEFnBNqINYy0RSFGb3fB1RkC21SQmia3ZkX6ibms7geYTeiaz6Fg/640?wx_fmt=png)

6. 按数组的偏移量和长度提取

  

  

下面的示例显示了从名为 Unix 的数组中从索引位置 3 开始提取 2 个元素的方法。

```
[root@localhost ~]# cat arraymain.sh 
#! /bin/bash
Unix=('Debian' 'Red hat' 'Ubuntu' 'Suse' 'Fedora' 'UTS' 'OpenLinux')
echo ${Unix[@]:3:2}


```

以下是输出，  

```
[root@localhost ~]# ./arraymain.sh 
Suse Fedora

```

![](https://mmbiz.qpic.cn/mmbiz_png/K0TMNq37VN0UqYH7fM2FFCTyebAOaRiaAzGDnpR9dQzlrE4VPjUibEygibBTPVnIAicf8qAOic1x0r13uGxoGZ9nS4Q/640?wx_fmt=png)

  
上面的示例返回第三、第四个索引的值。索引始终以零开头。

7. 对于数组的特定元素，使用 offset 和 length 提取

  

  

从数组元素中仅提取前四个元素。例如，获取数组中的第二个元素，并且获取这个元素的前三个字符：

```
[root@localhost ~]# cat arraymain.sh 
#! /bin/bash
Unix=('Debian' 'Red hat' 'Ubuntu' 'Suse' 'Fedora' 'UTS' 'OpenLinux')

echo ${Unix[1]:0:3}

```

以下是输出：

```
[root@localhost ~]# ./arraymain.sh 
Red

```

![](https://mmbiz.qpic.cn/mmbiz_png/K0TMNq37VN0UqYH7fM2FFCTyebAOaRiaAN4QyDW3Wfsfw4SDibEI23J2BFOePhicwefpz5Q80EFm5NBPuhibLb3yVQ/640?wx_fmt=png)

8. 搜索并替换数组元素

  

  

下面的示例在数组中搜索 Ubuntu，并将其替换为单词 “FreeBSD”。

```
[root@localhost ~]# cat arraymain.sh 
#! /bin/bash
Unix=('Debian' 'Red hat' 'Ubuntu' 'Suse' 'Fedora' 'UTS' 'OpenLinux')

echo ${Unix[@]/Ubuntu/FreeBSD}

```

以下是输出：

```
[root@localhost ~]# ./arraymain.sh 
Debian Red hat FreeBSD Suse Fedora UTS OpenLinux
[root@localhost ~]#

```

![](https://mmbiz.qpic.cn/mmbiz_png/K0TMNq37VN0UqYH7fM2FFCTyebAOaRiaApChhaFiaJEp0iaIoCtAnyBqP2ZwvRK8p69vVQhiauBQ0npfnImnFDCMVA/640?wx_fmt=png)

  
注意，该数组替换，不会写入到数组里面。

  

9. 向现有的数组添加元素

  

  

以下示例显示了将元素添加到现有数组的方法。

```
[root@localhost ~]# cat arraymain.sh 
#! /bin/bash
Unix=('Debian' 'Red hat' 'Ubuntu' 'Suse' 'Fedora' 'UTS' 'OpenLinux')
Unix=("${Unix[@]}" "AIX" "HP-UX")

echo ${Unix[@]}

```

以下是输出：

```
[root@localhost ~]# ./arraymain.sh 
Debian Red hat Ubuntu Suse Fedora UTS OpenLinux AIX HP-UX

```

![](https://mmbiz.qpic.cn/mmbiz_png/K0TMNq37VN0UqYH7fM2FFCTyebAOaRiaAS0dZWibyfZ1icq8zSIan388UGO9QGUibEZCBXWGpJiaPxsPStRyt4vg4Zw/640?wx_fmt=png)

  
在名为 Unix 的数组中，元素 “AIX” 和“ HP-UX”分别添加在第 7 个索引和第 8 个索引中。并输出所有数组元素。

  

10. 从数组中删除一个元素

  

  

`unset`用于从数组中删除元素。unset 和分配给元素 “Null" 值具有相同的效果。

```
[root@localhost ~]# cat arraymain.sh 
#! /bin/bash
Unix=('Debian' 'Red hat' 'Ubuntu' 'Suse' 'Fedora' 'UTS' 'OpenLinux')

unset Unix[1]

echo ${Unix[@]}
echo ${Unix[1]}

```

以下为输出：

```
[root@localhost ~]# ./arraymain.sh 
Debian Ubuntu Suse Fedora UTS OpenLinux



```

上面的脚本将打印整个数组，还有索引为 "1" 的值，索引为 “1” 的值是 null。  

![](https://mmbiz.qpic.cn/mmbiz_png/K0TMNq37VN0UqYH7fM2FFCTyebAOaRiaANFK7rZ2c4baJdO3cntticZk1goibZjGIiaHgun4wzvSE0xGndwwTdibuZQ/640?wx_fmt=png)

  
以下示例显示了从数组中完全删除元素的一种方法，下面还是删除索引为 1 的元素。

```
[root@localhost ~]# cat arraymain.sh 
#! /bin/bash
Unix=('Debian' 'Red hat' 'Ubuntu' 'Suse' 'Fedora' 'UTS' 'OpenLinux')
pos=1
Unix=(${Unix[@]:0:$pos} ${Unix[@]:$(($pos + 1))})
echo ${Unix[@]}

```

以下为输出：

```
[root@localhost ~]# ./arraymain.sh 
Debian Ubuntu Suse Fedora UTS OpenLinux

```

![](https://mmbiz.qpic.cn/mmbiz_png/K0TMNq37VN0UqYH7fM2FFCTyebAOaRiaA5xUWial1Vjm89UP0nllUWSBtOvnviaZ4Nh3P23dZbeVPQtjw9jnTcq8Q/640?wx_fmt=png)

  
在此示例中，${Unix[@]:0:$pos} 值获取第 1 个索引的元素，，而 ${Unix[@]:$(($pos + 1))} 将从第 3 个索引到最后一个索引。并合并以上两个输出。这是从数组中删除元素的解决方法之一。

  

11. 使用正则表达式删除数组中的元素

  

  

在搜索条件中，可以给出正则表达式，并将剩余的元素存储到另一个数组中，如下所示。

```
[root@localhost ~]# cat arraymain2.sh 
#! /bin/bash
declare -a Unix=('Debian' 'Red hat' 'Ubuntu' 'Suse' 'Fedora')
declare -a pattern=( ${Unix[@]/Red*/} )
echo ${pattern[@]}

```

以下为输出：

```
[root@localhost ~]# ./arraymain2.sh 
Debian Ubuntu Suse Fedora

```

![](https://mmbiz.qpic.cn/mmbiz_png/K0TMNq37VN0UqYH7fM2FFCTyebAOaRiaAeXDrAOjGlfKJRAWtkMbpN2MUzliazTeBrD0AgqH3XHrxwx2nEmuHENA/640?wx_fmt=png)

  
上面的示例中删除了包含 "Red" 字符的元素。实际是将 "Red*" 替换为空字符。

  

12. 复制数组

  

  

以下实例是将 Unix 数组复制到 Linux 数组中：

```
[root@localhost ~]# cat arraymain.sh 
#!/bin/bash
Unix=('Debian' 'Red hat' 'Ubuntu' 'Suse' 'Fedora' 'UTS' 'OpenLinux')
Linux=("${Unix[@]}")

echo ${Linux[@]}

```

以下为输出：

```
[root@localhost ~]# ./arraymain.sh 
Debian Red hat Ubuntu Suse Fedora UTS OpenLinux

```

![](https://mmbiz.qpic.cn/mmbiz_png/K0TMNq37VN0UqYH7fM2FFCTyebAOaRiaAERwwFqYdLwLmRSaVADx8vjic941l1HzticLAsHhIYnMqAy7mQVNO4lIA/640?wx_fmt=png)

  

13. 两个数组的关联

  

  

展开两个数组的元素，然后将其分配给新数组：

```
[root@localhost ~]# cat arraymain.sh 
#!/bin/bash
Unix=('Debian' 'Red hat' 'Ubuntu' 'Suse' 'Fedora' 'UTS' 'OpenLinux');
Shell=('bash' 'csh' 'jsh' 'rsh' 'ksh' 'rc' 'tcsh');

UnixShell=("${Unix[@]}" "${Shell[@]}")
echo ${UnixShell[@]}
echo ${#UnixShell[@]}

```

以下为输出：

```
[root@localhost ~]# ./arraymain.sh 
Debian Red hat Ubuntu Suse Fedora UTS OpenLinux bash csh jsh rsh ksh rc tcsh
14

```

![](https://mmbiz.qpic.cn/mmbiz_png/K0TMNq37VN0UqYH7fM2FFCTyebAOaRiaAH56zISnYdqRAbaSm0FaKjGd14ibJ0Zz23nCnUd75WibYMImuhnE5g5AA/640?wx_fmt=png)

  
该实例同时打印数组 “Unix” 和“ Shell”数组的元素，并且新数组的元素数为 14 个。

  

14. 删除整个数组

  

  

unset 用于删除整个数组。

```
[root@localhost ~]# cat arraymain.sh 
#!/bin/bash
Unix=('Debian' 'Red hat' 'Ubuntu' 'Suse' 'Fedora' 'UTS' 'OpenLinux');
Shell=('bash' 'csh' 'jsh' 'rsh' 'ksh' 'rc' 'tcsh');

UnixShell=("${Unix[@]}" "${Shell[@]}")

unset UnixShell
echo ${#UnixShell[@]}

```

以下为输出：

```
[root@localhost ~]# ./arraymain.sh 
0

```

![](https://mmbiz.qpic.cn/mmbiz_png/K0TMNq37VN0UqYH7fM2FFCTyebAOaRiaAdUTibd1GK6LAcYdOjDGRuY9KbFGQcjY1jD4edeiaaeXKHwpXicMLjZTCg/640?wx_fmt=png)

  
`unset`数组之后，其长度将为零，如上所示。

  

15. 将文件内容加载到数组中

  

  

您可以将文件内容逐行加载到数组中。

```
[root@localhost ~]# cat loadcontent.sh 
#!/bin/bash
file=(`cat ./pic.txt`)

for i in "${file[@]}"
do
echo $i
done

echo -e "\033[31m Read file content! \033[0m"

```

以下为输出：

```
[root@localhost ~]# ./loadcontent.sh 
https://www.linuxprobe.com/wp-content/uploads/2021/01/windows7.png
https://www.linuxprobe.com/wp-content/uploads/2016/12/bigdata.jpg
https://www.linuxprobe.com/wp-content/uploads/2021/01/write-games-and-learn-python.jpg
https://www.linuxprobe.com/wp-content/uploads/2021/01/data-center-inspection.jpg
https://www.linuxprobe.com/wp-content/uploads/2020/03/devolop-like-linux-09.jpg
 Read file content!

```

![](https://mmbiz.qpic.cn/mmbiz_png/K0TMNq37VN0UqYH7fM2FFCTyebAOaRiaAO6hnYQHXyFeyoTeiaozc23xhiaHWeoqf8VTjfQtq3phxgQKfFXNsibdVQ/640?wx_fmt=png)

  
在上面的示例中，使用 for 循环，逐一显示文件中每行的内容。

  

总结

  

  

本文到此结束，如果你喜欢这篇文章还请点个在看点个赞，转发一下支持作者吧~

**END**
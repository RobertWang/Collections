> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/yyl327291564/article/details/52871788)

前言
==

以前在使用 git 时，做版本比较的 git diff 是有颜色显示，绿色表示新增加的，红色表示删除的，一目了然。

> ![](https://img-blog.csdn.net/20161020133137175)

可是在 svn 下面使用 svn diff 是显示的都是白色的，虽然有 “+”、“-” 号可以知道增加的还是删除了的，可是看着不是很明显，感觉效率还是受到了影响

> ![](https://img-blog.csdn.net/20161020133221317)

并且 git diff 输出的是通过 less 输出的对于长的文本是可以通过上下和空格进行翻页查看，而 svn diff 是直接打印到终端的，当输出的文本过长是查看不是很方便。于是想要把 svn diff 的输出重新排版为跟 git diff 一样。

正文
==

1. 转义序列
-------

在 linux 终端控制台可以用彩色来显示文本，要使用彩色显示文本就需要使用到**_转义序列_**。转义序列就是一个让 shell 执行一个特殊步骤的控制指令。 转义序列通常都是以 ESC 开头（这也是它的命名原因）。 在 shell 里表示为 ^[ 。这种表示法需要一点时间去适应， 也可以用 \033 完成相同的工作（ESC 的 ASCII 码用十进制表示就是 27，等于用八进制表示的 033）。  
比如用下面的语句

```
echo -e "\033[1;031;040mtest text\033[0m"
```

输出的结果

> ![](https://img-blog.csdn.net/20161020133305582)

*   echo -e  
    是为了让 echo 能认得转义字符
*   \033[  
    转义序列的开头
*   1;031;040m  
    *   1  
        表示文本属性，可取的值有 0、1、22、4、24、5、25、7、27，分别表示：默认值、粗体、非粗体、下划线、非下划线、闪烁、非闪烁、 反显、非反显。
    *   031  
        表文字的颜色，可取的值从 30 到 37 共 8 个数值，他们对应的颜色代码是：30（黑色）、31（红色）、32（绿色）、 33（黄色）、34（蓝色）、35（洋红）、36（青色）、37（白色）
    *   041  
        表背景的颜色，与文字颜色的差异就是，把 3 换成 4，可取的值从 40 到 47 共 8 个数值，同样的他们对应的颜色代码是：40（黑色）、41（红色）、42（绿色）、 43（黄色）、44（蓝色）、45（洋红）、46（青色）、47（白色）
*   test text 是测试用的字符串，随意取
*   \033[0m  
    让终端恢复原有颜色

> 这里的 m 必须和后面的字符串（test text）连在一起，中间不能有空格隔开
> 
> 不同颜色值，可以通过一个 shell 脚本，全部打印出来  
> ![](https://img-blog.csdn.net/20161020133409568)
> 
> 打印的 shell，如下：

```
#!/bin/sh

for attr in 0 1 4 5 7; do
    echo "--------------------------"
    printf "Esc[%s;fore;back - \n" $attr
    for fore in 30 31 32 33 34 35 36 37; do
        for back in 40 41 42 43 44 45 46 47; do
            printf '\033[%s;%s;%sm%02s;%02s\033[0m ' $attr $fore $back $fore $back
        done
        printf '\n'
    done
done
```

> 参考资料  
> [http://www.360doc.com/content/11/0103/05/3688062_83518447.shtml](http://www.360doc.com/content/11/0103/05/3688062_83518447.shtml)  
> [http://wenku.baidu.com/link?url=RD7zoDxvFO5R6ZIaB7XRf7Nclwr1OS89oG6JFWZwlp9pJ2qqVDFHWBi0HRgjDxV5oxiGAdBZ4wDdNhyzGF8RC-akZYRKJ661GvRFHQ2y34m](http://wenku.baidu.com/link?url=RD7zoDxvFO5R6ZIaB7XRf7Nclwr1OS89oG6JFWZwlp9pJ2qqVDFHWBi0HRgjDxV5oxiGAdBZ4wDdNhyzGF8RC-akZYRKJ661GvRFHQ2y34m)

2、awk
-----

现在已经能改变终端输出的颜色了，现在要根据不同的类型配上不同颜色在输出，这是就要用到 awk 了.  
awk 可以遍历一个文件中的每一行，然后分别对文件的每一行进行处理，一个完整的 awk 命令形式如下：

```
awk [options] 'BEGIN{ commands } pattern{ commands } END{ commands }' file
```

其中 options 表示 awk 的可选的命令行选项，其中最常用的恐怕是 -F 它指定将文件中每一行分隔成列的分隔符号。而紧接着后面的单引号里面的所有内容是 awk 的程序脚本，awk 需要对文件每一行分割后的每一列做处理。file 则是 awk 要处理的文件名称。让我们通过 demo 来体会 awk 的功能。比如，awk 对每一行进行分割处理

```
echo '11 22 33 44' | awk '{print $3" "$2" "$1}'
```

输出：33 22 11

awk 不但能这样简单的处理每一行，还能有像 C 语言一样的使用判断，循环结构比如：  
新建一个 test.txt，里面内容如下：

```
1 2 3

4 5 6

7 8 9

10 11 12
```

执行

```
awk '{if($1%2==0)print $1" "$2" "$3}' test.txt
```

输出结果

```
4 5 6

10 11 12
```

我们知道 svn diff 输出中添加的每一行前面有 “+”, 删除的每一行前面有 “-”，据此可以用 awk 来判断，在通过转义序列着色输出。每个 “+” 和 “-” 都与后面的有空格隔开，因此可以使用空格作为分割符，来处理没一行

```
svn diff test_file.c | awk '{if($1 == "+"){print "\033[1;032;040m"$0"\033[0m"}else if($1 == "-"){print "\033[1;031;040m"$0"\033[0m"}else{print $0}}'
```

下面是在我的工程中使用了上面语句后的结果

> ![](https://img-blog.csdn.net/20161020133524024)
> 
> 参考资料  
> [http://www.toutiao.com/a6329234254174470401/?tt_from=mobile_qq&utm_campaign=client_share&app=news_article&utm_source=mobile_qq&iid=5593788734&utm_medium=toutiao_android](http://www.toutiao.com/a6329234254174470401/?tt_from=mobile_qq&utm_campaign=client_share&app=news_article&utm_source=mobile_qq&iid=5593788734&utm_medium=toutiao_android)

3、less
------

上面的代码是直接打印到终端的，对于长文件查看不方便，于是可以通过管道输出到 less 来查看

```
svn diff test_file.c | awk '{if($1 == "+"){print "\033[1;032;040m"$0"\033[0m"}else if($1 == "-"){print "\033[1;031;040m"$0"\033[0m"}else{print $0}}' | less -r
```

-r 参数是为了保持颜色，如果没有 -r 输出的将是原来的白色

> 参考资料  
> [http://zhidao.baidu.com/link?url=9fSn6aHUYXnqOyYJZRdymlLznCLK1Dl_l9f57dSfw_23GhOnwieezQXWC6xaVIoAzpCtg5D5ehk8yjYsS2kzhULiDRQSPT3FrorYLqjyhSS](http://zhidao.baidu.com/link?url=9fSn6aHUYXnqOyYJZRdymlLznCLK1Dl_l9f57dSfw_23GhOnwieezQXWC6xaVIoAzpCtg5D5ehk8yjYsS2kzhULiDRQSPT3FrorYLqjyhSS)

4、别名
----

效果是出来了，但每次都要输入这么一大串字符，非常的麻烦，还好 linux 给我们提供了别名（alias），别名通俗点来说就是自己定义命令。  
在 linux 终端可以直接通过 alias 来查看已经定义的别名。  
比如

```
alias
```

输出

```
    alias egrep='egrep --color=auto'
    alias fgrep='fgrep --color=auto'
    alias grep='grep --color=auto'
    alias l='ls -CF'
    alias la='ls -A'
    alias ll='ls -alF'
    alias ls='ls --color=auto'
```

通过 alias 可以查看已经设置的别名，同样通过该命令可以设置别名，一般格式如下

```
alias[别名]=[指令名称]
```

那么可以通过下面的语句来设置了

```
alias svndiff='svn diff test_file.c | awk '{if($1 == "+"){print "\033[1;032;040m"$0"\033[0m"}else if($1 == "-"){print "\033[1;031;040m"$0"\033[0m"}else{print $0}}' | less -r'
```

可是竟然报错了

```
-bash: 未预期的符号 `(' 附近有语法错误
```

仔细看看这里用了单引号、双引号、$ 符号、反斜杠 \、这些在 shell 里都是有特殊涵义的，会不会是这个原因呢，试着添加转义的反斜杠 \，修改为

```
alias svndiff='svn diff test_file.c | awk '\''{if($1 == "+"){print "\033[1;032;40m"$0" \033[0m"}else if($1 == "-"){print "\033[1;031;40m"$0" \033[0m"}else{$0}}'\'' | less -r'
```

设置成功了。  
可是，这样设置后就只能针对某个文件进行查看，不能对所有文件都使用这个别名了，但使用 alias 不能传递参数，不能传递参数就无法通过不同的参数来打开不同的文件。还好 shell 的函数可以实现这个功能，于是就变成这样

```
alias svndiff='function __svndiff(){svn diff $1 | awk '\''{if($1 == "+"){print "\033[1;032;40m"$0" \033[0m"}else if($1 == "-"){print "\033[1;031;40m"$0" \033[0m"}else{$0}}'\'' | less -r; unset -f __svndiff;}; __svndiff'
```

这样设置后就可以使用

```
svndiff test_file
```

这样的形式来查看了。

然而这样却需要每次登陆系统时都设置一次 alias。 如果想一劳永逸的设置，可以把他写入 home 目录下的 .bashrc 里面。打开 .bashrc，直接添加到文件的最后面，当然为了好看一点也可以添加到默认的 alias 的后面，比如我的 .bashrc 文件打开后有这么一段，

> ![](https://img-blog.csdn.net/20161020133627182)

直接把上面 alias 语句添加到后面，但需要注意的是由于引号、$ 符号、反斜杠 \ 等在 shell 里面有特殊的含义所以需要用转义和引号包括的形式就行转义，需要修改为

```
alias svndiff='function __svndiff() { svn diff $1 | awk "'"{if("'"\$1"'" == "'"\""'"+"'"\""'"){print "'"\""'"\033[32;1m"'"\""'""'"\$0"'""'"\""'" \033[0m"'"\""'"}else if("'"\$1"'" == "'"\""'"-"'"\""'"){print "'"\""'"\033[31;40m"'"\""'""'"\$0"'""'"\""'" \033[0m"'"\""'"}else{print "'"\$0"'"}}"'" | less -r; unset -f __svndiff; };__svndiff'
```

这样的全部写在一行的看的有点不好看，可以分开多行, 但是在换行是一定要注意美一行的空白

```
alias svndiff='function __svndiff() { svn diff $1 | \
  awk "'"{if("'"\$1"'" == "'"\""'"+"'"\""'"){print "'"\""'"\033[32;1m"'"\""'""'"\$0"'""'"\""'" \033[0m"'"\""'"}
  else if("'"\$1"'" == "'"\""'"-"'"\""'"){print "'"\""'"\033[31;40m"'"\""'""'"\$0"'""'"\""'" \033[0m"'"\""'"}
  else{print "'"\$0"'"}}"'" | less -r; unset -f __svndiff; };
__svndiff'

```

> 参考资料  
> [http://blog.sina.com.cn/s/blog_517b43f70100yavr.html](http://blog.sina.com.cn/s/blog_517b43f70100yavr.html)  
> [http://zhidao.baidu.com/link?url=Fj-0np1rp9N1LHuOdxO68GKnCaFuZCvdUrodixOA6kzdzJD-9M5oGNd5c-jEgWtNdXyRexb_LXNz8LYiklcCzXznD9QJXXFRsrEwYai8nXu](http://zhidao.baidu.com/link?url=Fj-0np1rp9N1LHuOdxO68GKnCaFuZCvdUrodixOA6kzdzJD-9M5oGNd5c-jEgWtNdXyRexb_LXNz8LYiklcCzXznD9QJXXFRsrEwYai8nXu)

结语
==

linux 下的工具很好用，可能单一的工具无法完成需求，但通过几个工具的组合使用，可以很容易达到需要的效果的。上面只是一个应该的例子，同样的方法还可以用于其他情形，如看 android log 时让他带点颜色，看起来更舒服一点。
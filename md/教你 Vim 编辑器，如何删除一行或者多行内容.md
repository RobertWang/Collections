> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1627456035&ver=3217&signature=mIRZCRH5JWevczC6pIEf4xUXN0QDUqCkFDwdvIQcJ-wgxSvS4xxkJ-gk7kJSddtQd2Ij2bY4SY*xn9UnSLrsgZ9EAvWz68NFWPnSCFDzIp9yOiBHSgDjr8BYTVcCWkex&new=1)

  

安装 Vim

  

```
#在Ubuntu/Debian中的安装方式
$ sudo apt install vim
#在RHEL/Centos中的安装方式
#] yum -y install vim 

```

删除单行内容

*   将光标移动到需要删除的行
    
*   按一下 ESC 键，确保退出编辑模式
    
*   按两次键盘上面的 d 键，就可以删除了。
    

**第一种方式**

*   按一下 ESC 键，确保退出编辑模式
    
*   按两次键盘上面的 g 键，让光标移动到文本的首行
    
*   然后按键盘上面的 d 和 G 键。其中 d 键是小写，G 键要切换成大写的。
    

这样就可以删除所有内容了。

**第二种方式**

*   按一下 ESC 键，确保退出编辑模式
    
*   按一下: 冒号键，(shift + ;) 就可以输入：冒号了。
    
*   然后输入 1,$d
    

**第三种方式**

*   按一下 ESC 键，确保退出编辑模式
    
*   按一下: 冒号键，shift + ; 就可以输入：冒号了。
    
*   然后输入 %d。% 表示文件中的所有行。
    

  

删除多行

  

*   将光标移动到需要删除的行
    
*   按一下 ESC 键，确保退出编辑模式
    
*   在 dd 命令前面加上要删除的行数。例如，如果要删除第 4 行以下的 3 行，请按下 3 dd
    

  

删除给定范围的行

  

**实例一**

如果你想要删除指定范围的行，比如从第 3 行到第 5 行，按 ESC，然后输入下面的命令，然后回车。

```
:3,5d 

```

< 以上代码可复制粘贴，可往左滑 >

**实例二**

删除最后一行，按 ESC，然后输入下面的命令，然后回车。

```
:$d


```

< 以上代码可复制粘贴，可往左滑 >

**实例三**

删除当前行之前的所有行

```
:1,.-1d


```

< 以上代码可复制粘贴，可往左滑 >

![](https://mmbiz.qpic.cn/mmbiz_png/K0TMNq37VN1jWRGyTrGNxIGoupjvtXzzBzlZYv1GW2dzLnTDq2DDBwwZnpTGKRPJFxlUEwNRZAK586g9emxG2w/640?wx_fmt=png)

**实例四**

删除当前行之后的所有行

```
:.+1,$d


```

< 以上代码可复制粘贴，可往左滑 >

![](https://mmbiz.qpic.cn/mmbiz_png/K0TMNq37VN1jWRGyTrGNxIGoupjvtXzzL0b2nZciatoiafeqUcTibiclia3icUDQ726rhXEQGGXA3J6Lbh1HQd0gcjVw/640?wx_fmt=png)

  

通过条件匹配删除行

  

**实例一**

删除包含 text 关键字的行

```
:g/text/d


```

< 以上代码可复制粘贴，可往左滑 >

![](https://mmbiz.qpic.cn/mmbiz_png/K0TMNq37VN1jWRGyTrGNxIGoupjvtXzzea4WlAIwVdpdWTlq3v482Oia5Pr4F2gufchPUa7xgKiccjQySldBdiacw/640?wx_fmt=png)

**实例二**

删除不包含#关键字的行

```
:%g!/#/d
#或者
:v/#/d


```

< 以上代码可复制粘贴，可往左滑 >

![](https://mmbiz.qpic.cn/mmbiz_png/K0TMNq37VN1jWRGyTrGNxIGoupjvtXzz665nEcWjRFA4JmUEX5IXhBiako2AoRPcQ1rUwsb4oXOk7n6K4hXwysA/640?wx_fmt=png)

**实例三**

删除以#开的的注释内容。

```
:g/^#/d


```

< 以上代码可复制粘贴，可往左滑 >

![](https://mmbiz.qpic.cn/mmbiz_png/K0TMNq37VN1jWRGyTrGNxIGoupjvtXzzo1eSQ8Yc2WRJ34jrSMA7uxXMJAkpVLyiaqBKcv1tbYU9pJqK1ZV91PQ/640?wx_fmt=png)

**实例四**

删除所有空行

```
:g/^$/d


```

< 以上代码可复制粘贴，可往左滑 >

![](https://mmbiz.qpic.cn/mmbiz_png/K0TMNq37VN1jWRGyTrGNxIGoupjvtXzzXvCPqC29sW9bpHKLjtVZXs1C5W1mYGB7sQDccXuHLJS3gphpJkHABw/640?wx_fmt=png)

总    结

Vim 有许多有用的功能，它们包括支持正则表达式的搜索，轻松重复命令的能力，直接记录和执行宏，自动完成，文件合并，鼠标集成，拼写检查，语法突出显示，分支撤消 / 重做历史，支持流行网络协议和文件存档格式等。

_**END**_
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/7d70082fb00c)

1、安装 bash-completion

> sudo apt-get install bash-completion

2、编辑~/.bashrc 文件

添加如下内容：

> if  [-f /etc/bash_completion]; then
> 
> . /etc/bash_completion
> 
> fi

3、使其立即生效

> source ~/.bashrc
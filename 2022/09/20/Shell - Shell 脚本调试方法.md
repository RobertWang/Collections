> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/anliven/p/6032081.html)

**目录**

*   [Shell 脚本调试选项](#_label0)
*   [ShellCheck](#_label1)
*   [ExplainShell](#_label2)
*   [BASH Debugger](#_label3)
*   [参考信息](#_label4)

[回到顶部](#_labelTop)

Shell 脚本调试选项
============

Shell 本身提供一些调试方法选项：

*   -n，读一遍脚本中的命令但不执行，用于检查脚本中的语法错误。
*   -v，一边执行脚本，一边将执行过的脚本命令打印到标准输出。
*   -x，提供跟踪执行信息，将执行的每一条命令和结果依次打印出来。

使用这些选项有三种方法 (注意: 避免几种调试选项混用)

*   1. 在命令行提供参数：`$sh -x script.sh`
*   2. 脚本开头提供参数：`#!/bin/sh -x`
*   3. 在脚本中用 set 命令启用 or 禁用参数：其中`set -x`表示启用，`set +x`表示禁用。

**set 命令的详细说明**

*   [http://man.linuxde.net/set](http://man.linuxde.net/set)
*   [https://www.runoob.com/linux/linux-comm-set.html](https://www.runoob.com/linux/linux-comm-set.html)

[回到顶部](#_labelTop)

ShellCheck
==========

*   [http://www.shellcheck.net/](http://www.shellcheck.net/)
*   是一个 Shell 脚本分析工具，可以为 bash/sh shell 脚本提出警告和建议。
*   GitHub：[https://github.com/koalaman/shellcheck](https://github.com/koalaman/shellcheck)

[回到顶部](#_labelTop)

ExplainShell
============

*   [https://www.explainshell.com/](https://www.explainshell.com/)
*   write down a command-line to see the help text that matches each argument

[回到顶部](#_labelTop)

BASH Debugger
=============

*   主页：[http://bashdb.sourceforge.net/](http://bashdb.sourceforge.net/)
*   下载地址：[https://sourceforge.net/projects/bashdb/files/](https://sourceforge.net/projects/bashdb/files/)
*   使用手册：[http://bashdb.sourceforge.net/bashdb-man.html](http://bashdb.sourceforge.net/bashdb-man.html)

借助第三方工具 bashd 可以更加精细地调试 Shell 脚本。具有断点、单步执行、观察变量等功能。  
下载时需根据所使用的 bash 版本选择相应的 bashdb，否则会提示因为版本不符合而无法安装。

*   查看 bash 版本：`bash --version`
*   典型用法：`bashdb --debug script.sh`

[回到顶部](#_labelTop)

参考信息
====

*   [Shell 脚本调试技术](http://www.ibm.com/developerworks/cn/linux/l-cn-shell-debug/)
*   [如何在 Linux 中启用 Shell 脚本的调试模式](https://linux.cn/article-8028-1.html)
*   [shell bashdb 调试](http://blog.techbeta.me/2015/10/shell-debug/)
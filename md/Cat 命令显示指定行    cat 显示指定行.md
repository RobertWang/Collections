> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/lelin/p/12833348.html)

[cat 显示指定行](https://www.cnblogs.com/suanec/p/5881463.html)
==========================================================

 

【一】从第 3000 行开始，显示 1000 行。即显示 3000~3999 行

cat filename | tail -n +3000 | head -n 1000

【二】显示 1000 行到 3000 行

cat filename| head -n 3000 | tail -n +1000

* 注意两种方法的顺序

分解：

    tail -n 1000：显示最后 1000 行

    tail -n +1000：从 1000 行开始显示，显示 1000 行以后的

    head -n 1000：显示前面 1000 行

【三】用 sed 命令

 sed -n '5,10p' filename 这样你就可以只查看文件的第 5 行到第 10 行

【一】从第 3000 行开始，显示 1000 行。即显示 3000~3999 行

cat filename | tail -n +3000 | head -n 1000

【二】显示 1000 行到 3000 行

cat filename| head -n 3000 | tail -n +1000

* 注意两种方法的顺序

分解：

    tail -n 1000：显示最后 1000 行

    tail -n +1000：从 1000 行开始显示，显示 1000 行以后的

    head -n 1000：显示前面 1000 行

【三】用 sed 命令

 sed -n '5,10p' filename 这样你就可以只查看文件的第 5 行到第 10 行
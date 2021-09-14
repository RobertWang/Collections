> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/mliudong/p/3770385.html)

　　大致有两种方式：

　　1，在 bashrc 中添加如下脚本

```
1 svndiff()
2 {
3   svn diff "${@}" | colordiff
4 }

```

     2，修改 svn 的配置文件

```
1 $ vim ~/.subversion/config
2 [helpers]
3 diff-cmd = colordiff

```

颜色配置，添加. colordiffrc 文件，可以做到和 git color 一样的效果

```
1 $ vim ~/.colordiffrc
2 banner=no
3 color_patches=no
4 plain=off
5 newtext=darkgreen
6 oldtext=darkred
7 diffstuff=white
8 cvsstuff=darkyellow

```

以上都要依赖 colordiff，安装

```
sudo apt-get install colordiff

```
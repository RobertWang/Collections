> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1450109&extra=page%3D1%26filter%3Dauthor%26orderby%3Ddateline)  ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) 牧星 

首先最近在用 sublime 这款软件但其环境配置着实麻烦不得已就全部看了一遍，也是一篇总结，在此分享给论坛的友友们希望能帮到你  
(第一次发帖，不足之处还请指正谢谢大家，如有违规请直接删帖)  
**以下出现的. sublime-build 文件都可以通过 packageresourcesviewer 这个插件来实现，安装后在命令面板输入 open resouce 回车打开你要配置的语言即可，或者仿照下面 go 语言的方法做**  
**下不了这个插件的我把文件打包的链接扔在这了，下载:[https://wwi.lanzoui.com/i2Rdapmferi](https://wwi.lanzoui.com/i2Rdapmferi) 密码: 52pj 解压后放入自己资源包的文件中就可以用了**  
**一切基于自己的编译环境配置好的情况下 (也就是可以在命令行时使用的情况下)**

**c/c++**
=========

在 csingle.sublime-build 或者 c++.sublime-build 中更改配置如下，一般改一个就行了
-------------------------------------------------------------

这个要装 mingw 且配置好环境变量，具体自行百度
--------------------------

{  
"cmd": ["g++", "${file}","-std=c++11","-o","${file_path}\\${file_base_name}","&","start","cmd","/c","${file_path}\\${file_base_name} & echo. & pause"],"file_regex":"^(..[^:]*):([0-9]+):?([0-9]+)?:? (.*)$","working_dir":"${file_path}","selector":"source.c, source.c++","shell": true,"encoding":"cp936","variants": [ {"name":"Build Only","cmd":["g++","${file}","-std=c++11","-o","${file_path}\\\\${file_base_name}"]  
},  
{  
"name" : "Run Only",  
"cmd" : ["start", "cmd", "/c", "${file_path}\\\\${file_base_name} & echo. & pause"]  
},  
{  
"name" : "Pipe Build and Run",  
"cmd":["g++", "${file}","-std=c++11","-o","${file_path}\\${file_base_name}","&","${file_path}\\${file_base_name}","&lt;","${file_path}\\in",">","${file_path}\\\\out"] }, {"name":"Pipe Run Only","cmd": ["${file_base_name}","<","in",">","out"]  
},  
{  
"name" : "Project Build & Run",  
"cmd" : ["g++", "${file_path}\\\\*.cpp","-std=c++11","-o","${file_path}/${file_base_name}","&","start","cmd","/c","${file_path}/${file_base_name} & echo. & pause"] }, {"name":"Project Build Only","cmd": ["g++","${file_path}\\*.cpp","-std=c++11","-o","${file_path}/${file_base_name}"]  
},  
{  
"name" : "Project Run Only",  
"cmd" : ["start", "cmd", "/c", "${file_path}/${file_base_name} & echo. & pause"]  
}  
]  
}

**javac**
=========

javac.sublime-build
-------------------

{  
"shell_cmd": "runJava.bat \"$file\"",  
"file_regex": "^(..._?):([0-9]_):?([0-9]*)",  
"selector": "source.java",  
"encoding": "utf-8"  
}

其中 runjava.bat 应配置文件如下, 记得放到自己的 jdk 的 bin 目录下
---------------------------------------------

@ECHO OFF  
cd %~dp1  
ECHO Compiling %~nx1.......  
IF EXIST %~n1.class (  
DEL %~n1.class  
)  
javac %~nx1  
IF EXIST %~n1.class (  
ECHO -----------OUTPUT-----------  
java %~n1  
)

**go**
======

#### 这个自己在自己安装的 sublime 路径下 \ Data\Packages 中创建一个文件夹，而后在该文件中创建一个 Go.sublime-build 文件

Go.sublime-build 内容
-------------------

{  
"cmd": ["go", "run", "$file_name"],"file_regex":"^[]*File \"(…*?)\", line ([0-9]*)","working_dir":"$file_path",  
"selector": "source.go"  
}

bash
====

这个需要装 linux 子系统，在设置中启用适用于 linux 的 windows 子系统
---------------------------------------------

而后在 shellscript.sublime-build 中改为如下配置
-------------------------------------

{  
"cmd" : ["bash", "-c", "bash ${file_name}"],"shell": true,"working_dir":"${file_path}",  
}

python
======

在 python.sublime-build 文件中改为
----------------------------

{  
"cmd": ["python","-u","$file"],  
"file_regex": "^[ ]_File \"(..._?)\", line ([0-9]*)",  
"selector": "source.python",  
"shell":"true",  
"encoding":"cp936"  
}

![](https://avatar.52pojie.cn/data/avatar/000/65/73/04_avatar_middle.jpg)Link_Stark  您好有一个问题想请教一下  
我现在的目标是 sublime text3 调用 cl.exe 编译 c++ 文件  
1、需要先打开这个 bat，C:\Program Files (x86)\Microsoft Visual Studio\2017\Professional\VC\Auxiliary\Build\vcvars64.bat（初始化环境，否则直接 cl 编译报缺少头文件 iostream）  
2、再在命令行运行命令 cll mian.cpp  
按上述步骤自己手动操作能编译成功  
但是现在 "shell_cmd": "vcvars64.bat cl  $file_name" 会把命令当成一行，怎么让命令换行运行  
试过 "shell_cmd": "vcvars64.bat",  
"shell_cmd": "cl  $file_name"  
这样操作是不行的![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)牧星 

> [Link_Stark 发表于 2021-5-31 18:35](https://www.52pojie.cn/forum.php?mod=redirect&goto=findpost&pid=38754852&ptid=1450109)  
> 您好有一个问题想请教一下  
> 我现在的目标是 sublime text3 调用 cl.exe 编译 c++ 文件  
> 1、需要先打开这个 bat，C: ...

查官方文档应该是使用管道连接, 可以试试  
"shell_cmd":"vcvars64.bat I cl \"$file_name\""  
或者 "cmd":["cl", "${file_path}\\\\*.cpp", "&", "start", "vcvars64.bat"]  
大该就是这样，不知道你可不可以，不行的话我也不会了![](https://static.52pojie.cn/static/image/smiley/default/14.gif)，捣鼓了半天  
![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)牧星  自己占个沙发，大家如果出现什么问题欢迎交流，共同学习进步 ![](https://avatar.52pojie.cn/data/avatar/001/06/12/25_avatar_middle.jpg) cysyzqw 谢谢楼主分享![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)C 哥 888  感谢楼主的热心分享 ![](https://avatar.52pojie.cn/data/avatar/000/20/45/43_avatar_middle.jpg) b0y - - 这个软件配置起来太麻烦了![](https://avatar.52pojie.cn/data/avatar/000/08/22/58_avatar_middle.jpg)冥界 3 大法王  折腾过 Python 和 AHK 的![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)牧星 

> [b0y 发表于 2021-5-31 06:55](https://www.52pojie.cn/forum.php?mod=redirect&goto=findpost&pid=38745298&ptid=1450109)  
> - - 这个软件配置起来太麻烦了

确实麻烦，折腾了有一个多月![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)牧星 > [冥界 3 大法王 发表于 2021-5-31 07:20](https://www.52pojie.cn/forum.php?mod=redirect&goto=findpost&pid=38745352&ptid=1450109)  
> 折腾过 Python 和 AHK 的

好家伙，下回要搞 AHK 不会直接问你，嘿嘿 &#128513; ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)lovehfs  慢慢地折腾和学习中... ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) 牧星 _ 本帖最后由 牧星 于 2021-5-31 08:21 编辑_  

> [lovehfs 发表于 2021-5-31 08:15](https://www.52pojie.cn/forum.php?mod=redirect&goto=findpost&pid=38745632&ptid=1450109)  
> 慢慢地折腾和学习中...

一起加油，配置环境可以问我，毕竟折腾了快几个月，知道一点 &#128514;，还有谢谢评分，嘿嘿
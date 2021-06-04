> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/cb690fc395ee)

Fbterm 简介
---------

Fbterm (Frame buffer terminal) 是內核終端的直接替代：一個沒有 Xorg 也能使用的終端模擬器。

**Fbterm 作用：**在字符终端 (tty 终端) 下正常显示中文，并可结合相关输入法进行中文输入。

**fbv 图形查看器作用：**在字符终端下显示和查看图片。可以将图片显示为 Fbterm 的背景图。

**截图：** 终端截屏也可利用 ImageMagic 程序的一个 import 命令

更具體的介紹見：

[Fbterm (简体中文) - ArchWiki](https://link.jianshu.com?t=https://wiki.archlinux.org/index.php/Fbterm_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))  
[字符终端中文支持软件 fbterm 配置安装](https://link.jianshu.com?t=http://bbs.chinaunix.net/thread-1921227-1-1.html)  
[Ubuntu 中在 tty 终端显示和输入汉字](https://link.jianshu.com?t=http://blog.csdn.net/xiajian2010/article/details/9625131)  
[Linux 控制台汉化 Fbterm 和 Yong](https://link.jianshu.com?t=http://blog.csdn.net/flytreeleft/article/details/6679638)

> 文章末尾附 Fbterm 安装脚本

安装 fbv
------

> 实际的操作步骤见附加的安装脚本，这里只讲思路。

这里使用`checkinstall`打包源码的方式安装，所以先安装`checkinstall` :

```
$ apt-get install checkinstall
```

通过 checkinstall 打包源码便于之后的卸载，实际安装时需要先运行如下命令：

```
./configure && make && sudo checkinstall
```

> checkinstall 的相关配置见 [http://www.ibm.com/developerworks/cn/linux/l-cn-checkinstall/](https://link.jianshu.com?t=http://www.ibm.com/developerworks/cn/linux/l-cn-checkinstall/)

如果不用 checkinstall 则直接:

```
./configure && make && sudo make install
```

编译 fbv 需依赖这三个 `libungif` `libjpeg-dev` `libpng12-dev` 其中`libungif`需另行下载源码编译:

```
sudo apt-get install libjpeg-dev libpng12-dev
```

下载 libungif 并安装:

```
下载链接：http://sourceforge.net/projects/giflib/files/libungif-4.x/
解压文件，再 ./configure && make && sudo checkinstall
```

下载 fbv 源码并安装:

```
http://www.eclis.ch/fbv/
解压文件， 再 ./configure && make && sudo checkinstall  

在最后 sudo checkinstall 的时候可能出现: 
/bin/sh: 1: cannot create  /usr/local/man/man1/fbv.1.gz: Directory nonexistent 
解决办法： 查看 ll /usr/local/man  原来这个软链接指向 /usr/local/share/man 
现在在 /usr/local/share/man/ 中建立一个目录 man1 即可。
```

安装运行 fbv ***.jpg 测试，如若出现不能打开文件或目录等消息，则先执行命令 `sudo ldconfig`

完成 fbv 安装

> 安装 fbv 参考：  
> [http://www.cnblogs.com/makefile/p/3952393.html](https://link.jianshu.com?t=http://www.cnblogs.com/makefile/p/3952393.html)  
> [http://askubuntu.com/questions/278863/how-do-i-set-up-a-background-image-for-console-in-ubuntu](https://link.jianshu.com?t=http://askubuntu.com/questions/278863/how-do-i-set-up-a-background-image-for-console-in-ubuntu)

安装 fbterm
---------

> 略：见附加的安装脚本

要支持中文输入如果使用的是 fcitx 则还要安装： `fcitx-frontend-fbterm`

> 安装 fbterm 参考：  
> [Fbterm (简体中文) - ArchWiki](https://link.jianshu.com?t=https://wiki.archlinux.org/index.php/Fbterm_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))  
> [Fbterm - openSUSE](https://link.jianshu.com?t=https://zh.opensuse.org/index.php?title=SDB:Fbterm&variant=zh-cn)

相关配置
----

建立脚本文件用于打开 fbterm 时调用 fbv 图形查看器，显示背景图片：

```
#!/bin/bash

# fbterm-bi: a wrapper script to  enable  background image with fbterm
# usage: fbterm-bi /path/to/image fbterm-options

 echo -ne "\e[?25l" # hide cursor
                  
#或fbv -ciuker "$1" << EOF  但要在启动脚本时同时指定图片路径，见上面的usage;
#fbv -ciuker "这里是你的图片的完整路径" << EOF
fbv -ciuker "/$HOME/backup/fbterm-bi.jpg" << EOF
q
EOF
                                                   
shift
export FBTERM_BACKGROUND_IMAGE=1
exec fbterm "$@"
```

**实际的运行脚本:** 每次都调用此脚本运行 fbterm

```
#!/bin/bash

# fbterm-bi: a wrapper script to  enable  background image with fbterm
# usage: fbterm-bi /path/to/image fbterm-options

 echo -ne "\e[?25l" # hide cursor
                  
#或fbv -ciuker "$1" << EOF  但要在启动脚本时同时指定图片路径，见上面的usage;
#fbv -ciuker "这里是你的图片的完整路径" << EOF

# 文件路径
filepath="~/.local/bin"

fbv -ciuker ${filepath}/fbterm-bi.jpg << EOF
#fbv -ciuker "/$HOME/.bin/.fbterm-bi.jpg" << EOF
#fbv -ciuker "/$HOME/.bin/.haizeiwangLufei.jpg" << EOF
#fbv -ciuker "/$HOME/.bin/.HaiZeiWangKuLou1.png" << EOF
q
EOF
                                                   
shift
export FBTERM_BACKGROUND_IMAGE=1
exec fbterm "$@"
```

附：安装脚本
------

```
#!/bin/bash
#安装 fbterm 和 fbv 

echo 安装时不要离开

#安装 fbterm 
sudo apt-get install fbterm -y


echo 如果安装的输入法是 fcitx 则需安装 以下程序，以在fbterm中支持中文输入法
echo 如果不是 fcitx 则选择 n ，在脚本执行之后，记得还要安装此项。 
sudo apt-get install fcitx-frontend-fbterm 


#若想使非root用户 fbterm 和 fbv 需要把自己加入 video 组
sudo gpasswd -a $USER video 

#若想非root用户可使用键盘快捷方式，(如切换输入法的快捷方式)需要：  
sudo chmod u+s /usr/bin/fbterm 



#安装fbv 的依赖项 libjpeg-dev  libpng12-dev  
sudo apt-get install libjpeg-dev libpng12-dev  -y

#下载安装另一个fbv 的依赖项 libungif-4.x.tar.gz 保存到目录 /tmp 
wget http://nchc.dl.sourceforge.net/project/giflib/libungif-4.x/libungif-4.1.4/libungif-4.1.4.tar.gz -P /tmp

#编译libungif , 使用 checkinstall ,则需安装该程序(使用其更易于卸载)
#如不使用 checkinstall 则直接 make install 即可
sudo apt-get install checkinstall -y 

cd /tmp
tar -xzvf libungif-4.1.4.tar.gz 
cd libungif-4.1.4 
./configure && make && sudo checkinstall
# 然后 手动运行(根据生成的deb软件包)  dpkg -i libungif-4.1.4.xxx.deb 


#下载 fbv-1.0b.tar.gz   
wget http://s-tech.elsat.net.pl/fbv/fbv-1.0b.tar.gz -P /tmp

cd /tmp 
tar -xzvf fbv-1.0b.tar.gz
cd fbv-1.0   

#运行下一步，可能出现错误： man1 目录不存在 ； 解决办法： 
sudo mkdir /usr/share/man/man1 
#完成 上次手动安装后，执行 ：  
#./configure && make && sudo checkinstall  


##############################
#  下面的步骤，手动执行
#############################

# 再手动安装 fbv.xxx.deb 包 
# sudo dpkg -i fbv.xxx.deb 


#安装fbv 后还要执行这一步 
# sudo ldconfig 



#之后再进行相关配置，  fbterm 的配置文件位于 ~/.fbtermrc 


# 最终配置好的fbterm命令，位于 ~/.local/bin/cfbterm  中

#最后在编写 运行脚本，以使 fbterm 运行时 调用 fbv 显示图片 等等  


#最后两项可参考如下连接： https://zh.opensuse.org/index.php?title=SDB:Fbterm&variant=zh-cn
# 和 ：  https://wiki.archlinux.org/index.php/Fbterm_(简体中文)
#注意: 第二个链接要包含"(简体中文)"
```
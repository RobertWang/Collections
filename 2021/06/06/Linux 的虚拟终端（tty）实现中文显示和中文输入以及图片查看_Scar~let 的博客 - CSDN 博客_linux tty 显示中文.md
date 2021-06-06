> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_43669969/article/details/104577615?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~aggregatepage~first_rank_v2~rank_aggregation-20-104577615.pc_agg_rank_aggregation&utm_term=linux%E7%BB%88%E7%AB%AF%E6%98%BE%E7%A4%BA%E5%9B%BE%E7%89%87&spm=1000.2123.3001.4430)

Linux 的虚拟终端（tty）实现中文显示和中文输入以及图片查看
=================================

### 前言

因为 Linux 系统的 tty 好像是不能直接支持中文显示的，所以要在另外一个程序中运行 tty，我的操作系统是 kali-linux5.4.0，基于 debian 的发行版，所以应该会 ubuntu、debian 是一样的操作

1. 先要安装 fbterm，才能在 tty 下显示中文字符，而且只有进入了 fbterm，才能切换中文
----------------------------------------------------

> aptitude install fbterm  
> 如果你的 linux 版本的库里面没有 fbterm，[请参考该文章](https://www.jianshu.com/p/cb690fc395ee)

**安装好后，直接在 tty 界面下输入 fbterm 就可以进入**

2. 安装 yong 输入法
--------------

> 直接百度搜索小小输入法（就是 yong 输入法），下载压缩包，解压  
> 住：7z 文件的解压命令：**7za x 你的压缩文件明 -r -o/xxx/xxx**  
> x 代表压缩文件，并且按原始目录解压，-r 代表递归解压缩所有子文件夹，-o 指定目录，-o 后面没有空格，直接接目录  
> 进入到解压好的文件目录  
> 运行 yong-tool.sh 即可  
> ./yong-tool.sh --install

> 相关的参数  
> –install  
> –uninstall  
> –select

安装好了之后，注销一下重新进入，终端输入：

> yong &  
> 让其在后台运行

或者在这个目录下添加一个可执行文件，名称随意

> /etc/X11/Xsession.d

里面的内容就是 **yong &**

注：yong 输入法不能在 xfce4 的终端下运行，但是可以在 fbterm 虚拟终端下运行，运行 yong 输入法时不能切换 ibus 输入法工具，所以要切换回原来的 ibus，需要卸载 yong 输入法，注销后重新登录
----------------------------------------------------------------------------------------------------------------

> ./yong-tool.sh --uninstall

需要用到 yong 输入法时，重新安装、登录即可。

3. 安装 fbv 可以在 tty 下查看图片，支持多张图片查看 (fbterm 下不能使用 fbv，因为 fbterm 不是 tty)
--------------------------------------------------------------------

> fbv xxx.png xxx.jpeg

### 我这里写出安装 fbv 的坑，后来人可以借鉴以下！！！

下载安装 fbv 压缩包的网址——>s-tech.elsat.net.pl/fbv
-----------------------------------------

#### 1. 安装 fbv 时需要几个依赖工具

可以在解压后的包里面参考这个

> cat README  
> **里面有这样几段话**  
> 2. REQUIREMENTS  
> Linux, configured to provide the framebuffer device interface  
> (/dev/fb0 or /dev/fb/0)  
> libungif for GIF support  
> libjpeg for JPEG support  
> libpng for PNG support

*   第一个插件 libungif：  
    我用的 kali5.4.0 系统，刚开始用 apt 直接安装 **libungif-bin**，发现 fbv 在运行./configure 的时候是没有检测到 libungif 的，所以要从源安装，在[这里去下载 tar.gz 文件](https://sourceforge.net/projects/giflib/files/libungif-4.x/libungif-4.1.4/)，安装过程不再赘述
*   第二个插件 libjpeg  
    直接 apt 安装即可，**aptitude install libjpeg-dev**
*   第三个插件 libpng  
    **aptitude install libpng-dev**，然后还有一个必然的插件 **libpng16-16**。注：这里开始出现了坑

#### 2. 开始安装 fbv

安装好上三个插件后再运行./configure 发现都支持了，然后运行 make 指令  
这时候据出现了错误，导致编译失败

> error: dereferencing pointer to incomplete type  
> if (setjmp (png_ptr->jmpbuf))  
>                                  ^

并且问题是出在 png.c 这个源文件的 if (setjmp (png_ptr->jmpbuf)) 语句，报错类型是指针指向不完整的结构类型，作为一个程序员就要找到原因所在：

> 首先查看 png_ptr 这个指针的定义，看到这个指针的在每个函数的声明为：  
> png_structp png_ptr;  
> 在 vim 里面搜索 png_structp 的定义，发现没有，但是在包含头文件中有这一行  
> #include<png.h>; 包含了这样一个库函数，这些库函数所在的文件目录为 / usr/include 这个文件夹中  
> 所以我们去看一下 png.h 这个头文件中是不是 png_structp 这个结构体不完整  
> 结果发现这个头文件有对 png_structp 的模板定义，  
> 所以猜测是不是 png_ptr 这个 png_structp 的指针没有 jmpbuf 成员，  
> 所以在 png.h 中搜索 jmpbuf，发现了这个宏定义，是这样的：  
> **#define png_jmpbuf(png_ptr)**

##### 好的，问题就出在这里，我们之前下的 libpng 库是——>libpng16-16，如果系统是之前的下的是 libpng12-dev 不会出现这个问题，最新的 libpng16 对 jmpbuf 重新定义了一下，而 fbv 已经很久没有更新了所以我们需要把解压后的 fbv 文件夹中的 png.c 中的所有 if(setjmp (png_ptr->jmpbuf)) 改成 png.h 中定义的指针调用改成即可：

> **if(setjmp(png_jmpbuf(png_ptr)))**

注：png_ptr 是在 png.c 中定义的结构指针，名字可能随着以后 fbv 的更新而有所不同哦 (但开发人员应该不会去改)，程序员自己细品

#### 现在重新 make，发现成功了，然后 make install 即可，结束！！！

更新
--

### 安装 fbgrab 在 tty 下可以截图

##### aptitude install fbgrab

注：fbgrab 只能在 tty 下截图，比如 fbgrab -c 1 xxx.png，-c 表示选择第几个 tty，这里是 tty1

[参考博客](https://www.jianshu.com/p/cb690fc395ee)
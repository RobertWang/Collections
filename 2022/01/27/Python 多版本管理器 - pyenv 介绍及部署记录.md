> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/kevingrace/p/10130801.html)

**一. pyenv 简单介绍**

在日常运维中, 经常遇到这样的情况: 系统自带的 Python 是 2.x，而业务部署需要 Python 3.x 环境, 此时需要在系统中安装多个 Python 版本，但又不能影响系统自带的 Python 版本, 即需要实现 Python 的多版本环境共存, pyenv 就是这样一个 Python 版本管理器, 可以同时管理多个 python 版本共存! 简单的说，pyenv 可以根据需求使用户在系统里安装和管理多个 Python 版本：  
- 配置当前用户的 python 的版本;  
- 配置当前 shell 的 python 版本;  
- 配置某个项目（目录及子目录）的 python 版本;  
- 配置多个虚拟环境.

由于 python 的各种优点，当前学习及使用 python 的人越来越多, 学习 python 有一个不容忽视的问题就是 python 的版本问题! 到现在为止，python 的版本有很多，但是问题在于 python2 与 python3 的区别。python3 的对一些模块进行了改变，导致了 python2 写的代码有的不被 python3 兼容，从而导致程序运行报错。因此，在学习和工作中使用 python 的时候，最好是安装一个 pyenv 管理器, 多安装几个 python 版本进行管理, 然后再针对不同项目安装各自项目的 python 虚拟环境, 相互隔离, 这样便于使用和管理。

**二. pyenv 工作原理**

pyenv 是利用系统环境变量 PATH 的优先级，劫持 python 的命令到 pyenv 上，根据用户所在的环境或目录，使用不同版本的 python。

```
how it works：
At a high level, pyenv intercepts Python commands using shim executables injected into your PATH, determines which Python version
has been specified by your application, and passes your commands along to the correct Python installation.
 
它是如何工作的：
在较高级别上，pyenv使用注入到PATH中的shim可执行文件拦截Python命令，确定应用程序指定了哪个Python版本，并将命令传递到正确的Python安装。
```

对于系统环境变量 PATH ，里面包含了一串由冒号分隔的路径，例如 /usr/local/bin:/usr/bin:/bin。每当在系统中执行一个命令时，例如 python 或 pip，操作系统就会在 PATH 的所有路径中从左至右依次寻找对应的命令。因为是依次寻找，因此排在左边的路径具有更高的优先级。在 PATH 最前面插入一个 $(pyenv root)/shims 目录，$(pyenv root)/shims 目录里包含名称为 python 以及 pip 等可执行脚本文件；当用户执行 python 或 pip 命令时，根据查找优先级，系统会优先执行 shims 目录中的同名脚本。pyenv 正是通过这些脚本，来灵活地切换至我们所需的 Python 版本。

**三. pyenv 安装和使用说明**

**1) pyenv 安装**

```
[root@localhost ~]# cat /etc/redhat-release
CentOS release 6.9 (Final)
   
系统默认是Python 2.6 版本
[root@localhost ~]# python -V            
Python 2.6.6
   
1) 安装依赖环境
[root@localhost ~]# yum -y install gcc zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel git
   
2) 安装pyenv包
pyenv可以通过多种方式安装，可以参考项目在github上的Installtion, 地址为: https://github.com/pyenv/pyenv-installer
   
推荐采用The automatic installer的方式安装，可以一键安装pyenv的所有插件。
[root@localhost ~]# curl -L https://github.com/pyenv/pyenv-installer/raw/master/bin/pyenv-installer | bash
 
pyenv套件下插件：
- pyenv-doctor
- pyenv-installer
- pyenv-update
- pyenv-virtualenv
- pyenv-which-ext
 
==================================================================================
温馨提示:
以上https://github.com/pyenv/pyenv-installer/raw/master/bin/pyenv-installer的访问内容
, 可以将内容粘出来放在服务器的一个shell脚本文件中, 然后执行该脚本用以安装pyenv
      
该脚本下载地址:  https://pan.baidu.com/s/1wW9ylrmc4Q9wxu_i3-1wYA
提取密码: rhtj
      
执行脚本进行安装(执行前授予755权限)
# chmod 755 pyenv-installer
# /bin/bash pyenv-installer
=================================================================================
   
分析一下上面的pyenv-installer脚本，可以发现在centos上，其实它做了以下事情:
git clone --depth 1"git://github.com/pyenv/pyenv.git"            "${HOME}/.pyenv"
git clone --depth 1"git://github.com/pyenv/pyenv-doctor.git"     "${HOME}/.pyenv/plugins/pyenv-doctor"
git clone --depth 1"git://github.com/pyenv/pyenv-installer.git"  "${HOME}/.pyenv/plugins/pyenv-installer"
git clone --depth 1"git://github.com/pyenv/pyenv-update.git"     "${HOME}/.pyenv/plugins/pyenv-update"
git clone --depth 1"git://github.com/pyenv/pyenv-virtualenv.git""${HOME}/.pyenv/plugins/pyenv-virtualenv"
git clone --depth 1"git://github.com/pyenv/pyenv-which-ext.git"  "${HOME}/.pyenv/plugins/pyenv-which-ext"
   
上面安装完成后，还需要执行下面的命令，将pyenv安装到系统环境变量中。
[root@localhost ~]# ll -d /root/.pyenv
drwxr-xr-x 11 root root 4096 Dec 17 10:48 /root/.pyenv
   
在~/.bash_profile文件底部添加下面三行内容，　让系统可以找到 pyenv 安装的 Python
[root@localhost ~]# vim ~/.bash_profile      
export PATH="/root/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
   
使上面配置生效
[root@localhost ~]# source ~/.bash_profile
   
查看pyenv安装情况
[root@localhost ~]# pyenv --version            //或者"pyenv -v"
pyenv 1.2.8
   
更新pyenv
[root@localhost ~]# pyenv update
   
3) 卸载pyenv
先删除pyenv的安装目录,  这里即是/root/.pyenv
[root@localhost ~]# rm -fr /root/.pyenv
   
接着删除~/.bash_profile里面配置的系统环境变量
[root@localhost ~]# vim ~/.bash_profile　　　　　//删除下面三行
export PATH="/root/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
   
[root@localhost ~]# source ~/.bash_profile
   
这样pyenv就被卸载了，　卸载pyenv后，　当前终端shell里会出现"-bash: pyenv: command not found"
的提示信息，　不过不影响使用．　再打开其他的终端窗口，　就不会出现该提示信息．
```

**2) pyenv 使用**

```
当前系统默认的Python版本
[root@localhost ~]# python -V
Python 2.6.6
   
pyenv的命令，可以通过pyenv help查看
[root@localhost ~]# pyenv help
Usage: pyenv  []
   
Some useful pyenv commands are:
   commands    List all available pyenv commands
   local       Set or show the local application-specific Python version
   global      Set or show the global Python version
   shell       Set or show the shell-specific Python version
   install     Install a Python version using python-build
   uninstall   Uninstall a specific Python version
   rehash      Rehash pyenv shims (run this after installing executables)
   version     Show the current Python version and its origin
   versions    List all Python versions available to pyenv
   which       Display the full path to an executable
   whence      List all Python versions that contain the given executable
   
See `pyenv help ' for information on a specific command.
For full documentation, see: https://github.com/pyenv/pyenv#readme
   
使用 pyenv install --list 查看可以安装的python版本
[root@localhost ~]# pyenv install --list
Available versions:
  2.1.3
  2.2.3
  .........
  2.7.4
  2.7.5
  .........
  3.0.1
  3.1
  .........
  3.6.1
  3.6.2
  .........
  3.7.1
  3.8-dev
   
查看当前pyenv可检测到的所有版本,处于激活状态的版本前以 * 标示.
[root@localhost ~]# pyenv versions
* system (set by /root/.pyenv/version)
   
使用pyenv install  安装python，可以使用-v参数，查看安装过程。
如下, 分别安装三个版本的python
[root@localhost ~]# pyenv install -v 2.7.5
[root@localhost ~]# pyenv install -v 3.1
[root@localhost ~]# pyenv install -v 3.6.1
   
查看当前pyenv可检测到的python版本 (即pyenv当前安装了哪些python版本)
[root@localhost ~]# pyenv versions
* system (set by /root/.pyenv/version)
  2.7.5
  3.1
  3.6.1
   
查看当前pyenv使用的python版本   (注意是version, 而不是上面的versions)
[root@localhost ~]# pyenv version
system (set by /root/.pyenv/version)
   
配置及管理python版本
pyenv管理python版本的三个基础命令(即使用下面三个命令的途径进行python版本的切换和激活状态):
- pyenv global // 配置当前用户的系统使用的python版本. 可以使用这个命令进行python版本的切换!
- pyenv shelll // 配置当前shell的python版本，退出shell则失效
- pyenv local // 配置所在项目（目录）的python版本
   
特别注意:
在使用上面pyenv三个基础命令进行python版本切换后:
如果想要切回到系统默认的python版本, 也就是这里默认的python2.6.6, 则需要下面命令进行切回操作!!!!!!
[root@localhost p_b]# pyenv local system
[root@localhost p_b]# python -V      
Python 2.6.6
   
下面分别介绍下pyenv这三个基础命令切换python版本的操作
a) 使用pyenv global 配置当前用户的系统使用的python版本
[root@localhost ~]# pyenv versions
* system (set by /root/.pyenv/version)
  2.7.5
  3.1
  3.6.1
   
[root@localhost ~]# python -V
Python 2.6.6
   
使用下面命令进行python版本的切换
[root@localhost ~]# pyenv global 3.6.1
   
[root@localhost ~]# pyenv versions
  system
  2.7.5
  3.1
* 3.6.1 (set by /root/.pyenv/version)
   
[root@localhost ~]# python -V
Python 2.6.6
   
需要执行下面命令进行数据库更新后, pyenv切换的python版本才会生效!
[root@localhost ~]# pyenv rehash
   
[root@localhost ~]# python -V      
Python 3.6.1
   
[root@localhost ~]# which python
/root/.pyenv/shims/python
 
=================================================================================
特别注意：如果使用"pyenv global xxx" 以及 "pyenv rehash" 后仍然无法成功切换版本！
这种情况一般是因为用pyenv指定了local版本！！
解决办法：取消设置local版本，即执行"pyenv local --unset"即可！
=================================================================================
   
b) 使用pyenv shelll 配置当前shell的python版本，退出shell则失效
[root@localhost ~]# python -V
Python 3.6.1
[root@localhost ~]# pyenv versions
  system
  2.7.5
  3.1
* 3.6.1 (set by /root/.pyenv/version)
[root@localhost ~]# pyenv shell 2.7.5
[root@localhost ~]# python -V     
Python 2.7.5
   
如上设置后, 只在当前shell终端窗口有效, 退出重新登录 或 再打开另外一个窗口就不生效了 (即pytho版本还是之前的)
即使执行"pyenv rehash" 进行更新操作, 在别的shell窗口也是不生效的!
   
当前shell下，取消配置的使用python shell --unset；若退出此shell，配置也会失效。
[root@localhost ~]# pyenv shell --unset
[root@localhost ~]# python -V       
Python 3.6.1
   
c) 使用pyenv local 配置所在项目（目录）的python版本
新建一个文件夹~/project，在此文件夹下使用python local [root@localhost ~]# python -V 
Python 3.6.1
[root@localhost ~]# mkdir project
[root@localhost ~]# cd project/
[root@localhost project]# pyenv local 3.1
[root@localhost project]# pyenv versions
  system
  2.7.5
* 3.1 (set by /root/project/.python-version)
  3.6.1
   
在当前项目目录下查看python版本
[root@localhost project]# python -V
Python 3.1
   
新建目录~/project/p_a,切换到~/project/p_a,并查看版本
[root@localhost project]# mkdir p_a&& cd p_a
[root@localhost p_a]# python -V
Python 3.1
   
[root@localhost p_b]# cd /root/
[root@localhost ~]# python -V
Python 3.6.1
   
如上可知, 第三种命令操作后, 切换的python版本只能在当前所在项目目录下生效!
在其他目录下就不会生效了!!
  
=============================================
如果要想卸载掉pyenv安装的python版本, 就使用"pyenv uninstall " 命令
[root@localhost ~]# pyenv versions
  system
  2.7.5
  3.1
* 3.6.1 (set by /root/.pyenv/version)
  
[root@localhost ~]# pyenv uninstall 3.1
pyenv: remove /root/.pyenv/versions/3.1? y
  
[root@localhost ~]# pyenv versions   
  system
  2.7.5
* 3.6.1 (set by /root/.pyenv/version) 
```

**接下来稍微深入的了解下 pyenv 是如何进行 python 版本管理的?**

```
使用which命令，可以看到，python命令已经不是本来的python命令，而是shims中的脚本文件
[root@localhost ~]# which python
/root/.pyenv/shims/python
[root@localhost ~]# which python3               
/root/.pyenv/shims/python3
[root@localhost ~]# which pip3                  
/root/.pyenv/shims/pip3
 
查看~/.pyenv/shims/python
[root@localhost ~]# cat /root/.pyenv/shims/python
#!/usr/bin/env bash
set -e
[ -n "$PYENV_DEBUG" ] && set -x
 
program="${0##*/}"
if [[ "$program" = "python"* ]]; then
  for arg; do
    case "$arg" in
    -c* | -- ) break ;;
    */* )
      if [ -f "$arg" ]; then
        export PYENV_FILE_ARG="$arg"
        break
      fi
      ;;
    esac
  done
fi
 
export PYENV_ROOT="/root/.pyenv"
exec "/root/.pyenv/libexec/pyenv" exec "$program" "$@"
 
可以看到python的命令最终被~/.pyenv/libexec/pyenv接管运行. 根据pyenv官方的解释, 大致了解到的意思是:
当使用的python命令被pyenv接管以后，到底使用哪个python版本，是由下面这些信息依次决定的：
1) 如果PYENV_VERSION这个变量存在，则使用这个变量里的版本；这个变量是由pyenv shell 配置的；
2) 按照往父目录查找的顺序查找直到根目录，第一个被查找到的.python-version文件作为版本文件,其指定的版本作为使用的python版本；
    这个文件使用pyenv local 配置
3) $(pyenv root)/version 这个文件若存在，则使用这个文件里制定的版本作为python版本；若不存在，则使用系统的版本；
    这个文件使用pyenv global 配置
4) 如果以上变量或文件都没有找到，就按照系统默认制定的python版本了。
 
另外，用户还可以在一个环境下同时配置多个版本的python；
回看一下上面pyenv versions命令，输出的结果中会有一个set by的提示，也向用户展示了，pyenv是基于什么指定的python版本。
[root@localhost ~]# pyenv versions
  system
  2.7.5
  3.1
* 3.6.1 (set by /root/.pyenv/version) 
```

**四. python 虚拟环境部署**

```
为了对不同的项目进行隔离，使每个项目使用独立的python解释器及依赖，需要配置python虚拟环境.
每个项目都有一个单独的python虚拟环境, 这样项目之前的python环境相互隔离, 便于使用和管理!
 
[root@localhost ~]# pyenv versions    
  system
  2.7.5
* 3.6.1 (set by /root/.pyenv/version)
 
上面使用pyenv install安装的python版本，比如3.6.1
python3.6.1解释器安装的路径为~/.pyenv/versions/3.6.1/;
插件的安装的路径为~/.pyenv/versions/3.6.1/lib/python3.6/site-packages
 
[root@localhost ~]# ll -d /root/.pyenv/versions/3.6.1/
drwxr-xr-x 6 root root 4096 Dec 17 12:57 /root/.pyenv/versions/3.6.1/
[root@localhost ~]# ll -d /root/.pyenv/versions/3.6.1/lib/python3.6/site-packages
drwxr-xr-x 8 root root 4096 Dec 17 12:57 /root/.pyenv/versions/3.6.1/lib/python3.6/site-packages
 
使用pyenv-virtualenv创建python虚拟环境，实质上是在~/.pyenv/versions/3.6.1/下创建一个文件夹evns，存放该虚拟环境python的解释器；
并且在~/.pyenv/下创建一个软连接，该虚拟环境可以通过pyenv进行管理；
 
1) 比如创建某个项目的python虚拟环境, 虚拟环境的命令为kevin_py (名称随便起), 该虚拟环境的python版本是2.7.5
[root@localhost ~]# pyenv virtualenv 2.7.5 kevin_py
 
查看, 发现在~/.pyenv/versions目录下会有一个kevin_py虚拟环境的软连接
[root@localhost ~]# ll ~/.pyenv/versions/
total 8
drwxr-xr-x 7 root root 4096 Dec 17 14:53 2.7.5
drwxr-xr-x 6 root root 4096 Dec 17 12:57 3.6.1
lrwxrwxrwx 1 root root   41 Dec 17 14:53 kevin_py -> /root/.pyenv/versions/2.7.5/envs/kevin_py
 
查看python虚拟环境
[root@localhost ~]# pyenv virtualenvs
  2.7.5/envs/kevin_py (created from /root/.pyenv/versions/2.7.5)
  kevin_py (created from /root/.pyenv/versions/2.7.5)
 
有四种方法用于切换到python虚拟环境
 
1) 方法一  (推荐这一个切换方法)
[root@localhost ~]# pyenv activate kevin_py
pyenv-virtualenv: prompt changing will be removed from future release. configure `export PYENV_VIRTUALENV_DISABLE_PROMPT=1' to simulate the behavior.
(kevin_py) [root@localhost ~]#
 
2) 方法二  (这种切换方法也推荐)
[root@localhost ~]# source activate kevin_py
pyenv-virtualenv: activate kevin_py
pyenv-virtualenv: prompt changing will be removed from future release. configure `export PYENV_VIRTUALENV_DISABLE_PROMPT=1' to simulate the behavior.
(kevin_py) [root@localhost ~]# python -V
Python 2.7.5
(kevin_py) [root@localhost ~]#
 
3) 方法三
[root@localhost ~]# source /root/.pyenv/versions/2.7.5/envs/kevin_py/bin/activate kevin_py
(kevin_py) [root@localhost ~]# python -V
Python 2.7.5
(kevin_py) [root@localhost ~]#
 
4) 方法三
[root@localhost ~]# pyenv shell kevin_py
(kevin_py) [root@localhost ~]# python -V
Python 2.7.5
(kevin_py) [root@localhost ~]#
 
使用"source deactivate" 命令 或者 "pyenv deactivate"命令 退出python虚拟环境!!!!!!
退出后, 在python虚拟环境里部署的应用程序不受影响!
(kevin_py) [root@localhost ~]# source deactivate
pyenv-virtualenv: deactivate 2.7.5/envs/kevin_py
[root@localhost ~]#
 
[root@localhost ~]# pyenv activate kevin_py
pyenv-virtualenv: prompt changing will be removed from future release. configure `export PYENV_VIRTUALENV_DISABLE_PROMPT=1' to simulate the behavior.
(kevin_py) [root@localhost ~]# pyenv deactivate
[root@localhost ~]#
 
 
2) 再创建另一个项目的python虚拟环境, 虚拟环境的命令为bobo_py, 该虚拟环境下的python版本为3.6.1
[root@localhost ~]# pyenv virtualenv 3.6.1 bobo_py
 
[root@localhost ~]# ll ~/.pyenv/versions/
total 8
drwxr-xr-x 7 root root 4096 Dec 17 14:53 2.7.5
drwxr-xr-x 7 root root 4096 Dec 17 15:00 3.6.1
lrwxrwxrwx 1 root root   40 Dec 17 15:00 bobo_py -> /root/.pyenv/versions/3.6.1/envs/bobo_py
lrwxrwxrwx 1 root root   41 Dec 17 14:53 kevin_py -> /root/.pyenv/versions/2.7.5/envs/kevin_py
 
[root@localhost ~]# pyenv virtualenvs
  2.7.5/envs/kevin_py (created from /root/.pyenv/versions/2.7.5)
  3.6.1/envs/bobo_py (created from /root/.pyenv/versions/3.6.1)
  bobo_py (created from /root/.pyenv/versions/3.6.1)
  kevin_py (created from /root/.pyenv/versions/2.7.5)
 
先切换到bobo_py虚拟环境
[root@localhost ~]# pyenv activate bobo_py
pyenv-virtualenv: prompt changing will be removed from future release. configure `export PYENV_VIRTUALENV_DISABLE_PROMPT=1' to simulate the behavior.
(bobo_py) [root@localhost ~]# python -V
Python 3.6.1
(bobo_py) [root@localhost ~]#
 
然后再从bobo_py环境环境切换到另一个虚拟环境kevin_py
(bobo_py) [root@localhost ~]# source activate kevin_py
pyenv-virtualenv: activate kevin_py
pyenv-virtualenv: prompt changing will be removed from future release. configure `export PYENV_VIRTUALENV_DISABLE_PROMPT=1' to simulate the behavior.
 
退出虚拟环境
(kevin_py) [root@localhost ~]# source deactivate
pyenv-virtualenv: deactivate 2.7.5/envs/kevin_py
[root@localhost ~]#
 
注意: 上面介绍的四种虚拟环境的切换方法, 强烈建议采用第一种和第二种切换方法!
因为采用前两种切换方法进去后, 使用"source deactivate" 或者 "pyenv deactivate"命令可以正常退出来!
如果采用后两种切换方法进去后, 使用"source deactivate" 或者 "pyenv deactivate"命令退不出来!
 
使用"pyenv virtualenv-delete"命令 删除虚拟环境
[root@localhost ~]# pyenv virtualenvs
  2.7.5/envs/kevin_py (created from /root/.pyenv/versions/2.7.5)
  3.6.1/envs/bobo_py (created from /root/.pyenv/versions/3.6.1)
  bobo_py (created from /root/.pyenv/versions/3.6.1)
  kevin_py (created from /root/.pyenv/versions/2.7.5)
 
[root@localhost ~]# pyenv virtualenv-delete bobo_py
pyenv-virtualenv: remove /root/.pyenv/versions/3.6.1/envs/bobo_py? y
 
[root@localhost ~]# pyenv virtualenvs              
  2.7.5/envs/kevin_py (created from /root/.pyenv/versions/2.7.5)
  kevin_py (created from /root/.pyenv/versions/2.7.5)
```
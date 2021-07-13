> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU0NjgzMDIxMQ==&mid=2247536904&idx=5&sn=fc4a22d13eb0576731cd96585fcc70e0&chksm=fb55b3e4cc223af242f4a53059edb13debace9a2bcd8f3b9e3e6f6d279e6ca5fd1f5793eb17e&mpshare=1&scene=1&srcid=0713KLuCoZfUix4P1X5eYpfq&sharer_sharetime=1626143724973&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

```
本文转自|机器学习实验室


```

Linux 环境变量配置
------------

*   系统：Ubuntu 14.0
    
*   用户名：uusama
    
*   需要配置 MySQL 环境变量路径：/home/uusama/mysql/bin
    

### Linux 读取环境变量

*   `export`命令显示当前系统定义的所有环境变量
    
*   `echo $PATH`命令输出当前的`PATH`环境变量的值
    

```
uusama@ubuntu:~$ export
declare -x HOME="/home/uusama"
declare -x LANG="en_US.UTF-8"
declare -x LANGUAGE="en_US:"
declare -x LESSCLOSE="/usr/bin/lesspipe %s %s"
declare -x LESSOPEN="| /usr/bin/lesspipe %s"
declare -x LOG
declare -x MAIL="/var/mail/uusama"
declare -x PATH="/home/uusama/bin:/home/uusama/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
declare -x SSH_TTY="/dev/pts/0"
declare -x TERM="xterm"
declare -x USER="uusama"
uusama@ubuntu:~$ echo $PATH
/home/uusama/bin:/home/uusama/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

```

### Linux 环境变量配置方法一：  export  PATH

```
export PATH=/home/uusama/mysql/bin:$PATH
# 或者把PATH放在前面
export PATH=$PATH:/home/uusama/mysql/bin

```

*   生效时间：立即生效
    
*   生效期限：当前终端有效，窗口关闭后无效
    
*   生效范围：仅对当前用户有效
    
*   配置的环境变量中不要忘了加上原来的配置，即`$PATH`部分，避免覆盖原来配置
    

### Linux 环境变量配置方法二：     vim ~/.bashrc

```
vim ~/.bashrc
# 在最后一行加上
export PATH=$PATH:/home/uusama/mysql/bin

```

*   生效时间：使用相同的用户打开新的终端时生效，或者手动`source ~/.bashrc`生效
    
*   生效期限：永久有效
    
*   生效范围：仅对当前用户有效
    
*   如果有后续的环境变量加载文件覆盖了`PATH`定义，则可能不生效
    

### Linux 环境变量配置方法三：  vim ~/.bash_profile

*   生效时间：使用相同的用户打开新的终端时生效，或者手动`source ~/.bash_profile`生效
    
*   生效期限：永久有效
    
*   生效范围：仅对当前用户有效
    
*   如果没有`~/.bash_profile`文件，则可以编辑`~/.profile`文件或者新建一个
    

### Linux 环境变量配置方法四：vim /etc/bashrc 

```
# 如果/etc/bashrc文件不可编辑，需要修改为可编辑
chmod -v u+w /etc/bashrc
vim /etc/bashrc
# 在最后一行加上
export PATH=$PATH:/home/uusama/mysql/bin

```

*   生效时间：新开终端生效，或者手动`source /etc/bashrc`生效
    
*   生效期限：永久有效
    
*   生效范围：对所有用户有效
    

### Linux 环境变量配置方法五：    vim /etc/profile

*   生效时间：新开终端生效，或者手动`source /etc/profile`生效
    
*   生效期限：永久有效
    
*   生效范围：对所有用户有效
    

### Linux 环境变量配置方法六：vim /etc/environment

```
# 如果/etc/bashrc文件不可编辑，需要修改为可编辑
chmod -v u+w /etc/environment
vim /etc/profile
# 在最后一行加上
export PATH=$PATH:/home/uusama/mysql/bin

```

注意事项：

*   生效时间：新开终端生效，或者手动`source /etc/environment`生效
    
*   生效期限：永久有效
    
*   生效范围：对所有用户有效
    

 Linux 环境变量加载原理解析
-----------------

特定的加载顺序会导致相同名称的环境变量定义被覆盖或者不生效。

### 环境变量的分类

*   用户级别环境变量定义文件：`~/.bashrc`、`~/.profile`（部分系统为：`~/.bash_profile`）
    
*   系统级别环境变量定义文件：`/etc/bashrc`、`/etc/profile`(部分系统为：`/etc/bash_profile`）、`/etc/environment`
    

另外在用户环境变量中，系统会首先读取`~/.bash_profile`（或者`~/.profile`）文件，如果没有该文件则读取`~/.bash_login`，根据这些文件中内容再去读取`~/.bashrc`。

### 测试 Linux 环境变量加载顺序的方法

需要修改的文件如下：

*   /etc/environment
    
*   /etc/profile
    
*   /etc/profile.d/test.sh，新建文件，没有文件夹可略过
    
*   /etc/bashrc，或者 / etc/bash.bashrc
    
*   ~/.bash_profile，或者~/.profile
    
*   ~/.bashrc
    

`export UU_ORDER="$UU_ORDER:~/.bash_profile"`

可以推测出 Linux 加载环境变量的顺序如下：

1.  /etc/environment
    
2.  /etc/profile
    
3.  /etc/bash.bashrc
    
4.  /etc/profile.d/test.sh
    
5.  ~/.profile
    
6.  ~/.bashrc
    

### Linux 环境变量文件加载详解

系统环境变量 -> 用户自定义环境变量  
/etc/environment -> /etc/profile -> ~/.profile

```
# /etc/profile: system-wide .profile file for the Bourne shell (sh(1))
# and Bourne compatible shells (bash(1), ksh(1), ash(1), ...).
if [ "$PS1" ]; then
  if [ "$BASH" ] && [ "$BASH" != "/bin/sh" ]; then
    # The file bash.bashrc already sets the default PS1.
    # PS1='\h:\w\$ '
    if [ -f /etc/bash.bashrc ]; then
      . /etc/bash.bashrc
    fi
  else
    if [ "`id -u`" -eq 0 ]; then
      PS1='# '
    else
      PS1='$ '
    fi
  fi
fi
if [ -d /etc/profile.d ]; then
  for i in /etc/profile.d/*.sh; do
    if [ -r $i ]; then
      . $i
    fi
  done
  unset i
fi

```

其次再打开`~/.profile`文件，会发现该文件中加载了`~/.bashrc`文件。

### 一些小技巧

可以自定义一个环境变量文件，比如在某个项目下定义`uusama.profile`，在这个文件中使用`export`定义一系列变量，然后在`~/.profile`文件后面加上：`sourc uusama.profile`，这样你每次登陆都可以在 Shell 脚本中使用自己定义的一系列变量。
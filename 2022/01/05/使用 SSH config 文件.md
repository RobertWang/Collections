> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_34273046/article/details/92211773?utm_term=ssh_config&utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~all~sobaiduweb~default-0-92211773&spm=3001.4430)

[2019 独角兽企业重金招聘 Python 工程师标准 >>>](https://my.oschina.net/u/2663968/blog/3061697) ![](https://www.oschina.net/img/hot3.png)

使用 [SSH](https://so.csdn.net/so/search?q=SSH) config 文件
-------------------------------------------------------

`ssh`的介绍及使用参看：[`SSH简介`](http://daemon369.github.io/ssh/2015/03/16/01-brief-introduction-for-ssh)、[`创建SSH密钥对`](http://daemon369.github.io/ssh/2015/03/08/generating-ssh-keys)。

*   [https://nerderati.com/2011/03/17/simplify-your-life-with-an-ssh-config-file/](https://nerderati.com/2011/03/17/simplify-your-life-with-an-ssh-config-file/)
*   [https://linux.die.net/man/5/ssh_config](https://linux.die.net/man/5/ssh_config)

配置文件
====

`ssh`程序可以从以下途径获取配置参数：

1.  命令行选项
2.  用户配置文件 (~/.ssh/config)
3.  系统配置文件 (/etc/ssh/ssh_config)

配置文件可分为多个配置区段，每个配置区段使用`Host`来区分。我们可以在命令行中输入不同的`host`来加载不同的配置段。

对每一个配置项来说，首次获取的参数值将被采用，因此通用的设置应该放到文件的后面，特定`host`相关的配置项应放到文件的前面。

常用配置项
=====

下面介绍一些常用的`SSH`配置项：

Host
----

`Host`配置项标识了一个配置区段。

`ssh`配置项参数值可以使用通配符：`*`代表 0～n 个非空白字符，`?`代表一个非空白字符，`!`表示例外通配。

我们可以在系统配置文件中看到一个匹配所有`host`的默认配置区段：

```
$ cat /etc/ssh/ssh_config | grep '^Host'
Host *
```

这里有一些默认配置项，我们可以在用户配置文件中覆盖这些默认配置。

GlobalKnownHostsFile
--------------------

指定一个或多个全局认证主机缓存文件，用来缓存通过认证的远程主机的密钥，多个文件用空格分隔。默认缓存文件为：/etc/ssh/ssh_known_hosts, /etc/ssh/ssh_known_hosts2.

HostName
--------

指定远程主机名，可以直接使用数字 IP 地址。如果主机名中包含 ‘%h’ ，则实际使用时会被命令行中的主机名替换。

IdentityFile
------------

指定密钥认证使用的[私钥](https://so.csdn.net/so/search?q=%E7%A7%81%E9%92%A5)文件路径。默认为 ~/.ssh/id_dsa, ~/.ssh/id_ecdsa, ~/.ssh/id_ed25519 或 ~/.ssh/id_rsa 中的一个。文件名称可以使用以下转义符：

```
'%d' 本地用户目录
'%u' 本地用户名称
'%l' 本地主机名
'%h' 远程主机名
'%r' 远程用户名
```

可以指定多个密钥文件，在连接的过程中会依次尝试这些密钥文件。

Port
----

指定远程主机端口号，默认为 22 。

User
----

指定登录用户名。

UserKnownHostsFile
------------------

指定一个或多个用户认证主机缓存文件，用来缓存通过认证的远程主机的密钥，多个文件用空格分隔。默认缓存文件为： ~/.ssh/known_hosts, ~/.ssh/known_hosts2.

还有更多参数的介绍，可以参看用户手册：

```
$ man ssh config


```

示例
==

以下连接为例：

```
SSH 服务器： ssh.test.com
端口号： 2200
账户： user
密钥文件： ~/.ssh/id_rsa_test
```

#### ## 密码认证登录方式为：

```
$ ssh -p 2200 -i ~/.ssh/id_rsa_test user@ssh.test.com
user@ssh.test.com's password:
```

#### ## 密钥认证登录方式：

```
$ ssh-copy-id -i ~/.ssh/id_rsa_test user@ssh.test.com
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
user@ssh.test.com's password:
Number of key(s) added: 1
Now try logging into the machine, with:   "ssh 'user@ssh.test.com'"
and check to make sure that only the key(s) you wanted were added.
$ ssh user@ssh.test.com
```

#### ## 使用配置文件方式

有如下配置文件：

```
$ vim ~/.ssh/config
Host sshtest
    HostName ssh.test.com
    User user
    Port 2200
    IdentityFile ~/.ssh/id_rsa_test
 
Host ssttest2
    HostName ssh.test2.com
    User user2
    Port 2345
    IdentityFile ~/.ssh/id_rsa_test2
```

使用配置文件登录：

```
$ ssh sshtest


```

#### ## SSH sock5 channel

```
$ ssh -f -N -L 9906:127.0.0.1:3306 coolio@database.example.com
# -f puts ssh in background
# -N makes it not execute a remote command
```

This will forward all local port `9906` traffic to port `3306` on the remote `database.example.com` server, letting me point my desktop GUI to localhost (`127.0.0.1:9906`) and have it behave exactly as if I had exposed port `3306` on the remote server and connected directly to it.

Now I don't know about you, but remembering that sequence of flags and options for [SSH](http://linux.die.net/man/1/ssh) can be a complete pain. Luckily, our config file can help alleviate that:

```
Host tunnel
    HostName database.example.com
    IdentityFile ~/.ssh/coolio.example.key
    LocalForward 9906 127.0.0.1:3306
    User coolio
```

Which means I can simply do:

```
$ ssh -f -N tunnel

```

参看
==

*   [SSH 简介](http://daemon369.github.io/ssh/2015/03/16/01-brief-introduction-for-ssh)
*   [创建 SSH 密钥对](http://daemon369.github.io/ssh/2015/03/08/generating-ssh-keys)

*   [https://nerderati.com/2011/03/17/simplify-your-life-with-an-ssh-config-file/](https://nerderati.com/2011/03/17/simplify-your-life-with-an-ssh-config-file/)

转载于: https://my.oschina.net/u/2306127/blog/3030253
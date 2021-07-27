> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/liuxiao723846/article/details/106940400)

coder-server 提供一个在线 IDE，界面和 vscode 一模一样。其公司 coder 之前本身就提供 IDE 的在线使用（ide.coder.com），但不知道什么原因，网站的该功能已经关闭了。  
现在只能以自建的形式使用：将 code-server 远程部署在服务器上，在任何浏览器上使用网页版 VScode。

官网：[http://www.coder.com/](http://www.coder.com/)  
github：[https://github.com/cdr/code-server](https://github.com/cdr/code-server)

### 一、安装：

**1、下载、解压：**

```
#环境
$ cat /etc/redhat-release 
CentOS Linux release 7.2.1511 (Core) 
 
cd /usr/local
#github上的下载连接：https://github.com/cdr/code-server/releases，根据不同环境下载二进制解压即可使用。
wget https://github.com/cdr/code-server/releases/download/3.4.1/code-server-3.4.1-linux-x86_64.tar.gz
 
tar -xvzf code-server-3.4.1-linux-x86_64.tar.gz
```

**2、启动：**

```
pwd
/usr/local
cd code-server-3.4.1-linux-x86_64
./code-server
***** Please use the script in bin/code-server instead!
***** This script will soon be removed!
***** See the release notes at https://github.com/cdr/code-server/releases/tag/v3.4.0
info  Using config file ~/.config/code-server/config.yaml
info  Using user-data-dir ~/.local/share/code-server
info  code-server 3.4.1 48f7c2724827e526eeaa6c2c151c520f48a61259
info  HTTP server listening on http://127.0.0.1:8080
info      - Using password from ~/.config/code-server/config.yaml
info      - To disable use `--auth none`
info    - Not serving HTTPS
 
```

在 windows 的浏览器上输入：http://remoteip:8080 后没有反应， code-server 没有输出任何错误。所以，怀疑是这种方式的启动，只能在远程 linux 的本地通过浏览器访问，无法在 windows 的浏览器上访问，无奈远程 linux 没有窗口界面，输入 --help 查询看是否有其他参数：

```
./code-server --help
***** Please use the script in bin/code-server instead!
***** This script will soon be removed!
***** See the release notes at https://github.com/cdr/code-server/releases/tag/v3.4.0
info  Using config file ~/.config/code-server/config.yaml
code-server 3.4.1 48f7c2724827e526eeaa6c2c151c520f48a61259
 
Usage: code-server [options] [path]
 
Options
      --auth                The type of authentication to use. [password, none]
      --password            The password for password authentication (can only be passed in via $PASSWORD or the config file).
      --cert                Path to certificate. Generated if no path is provided.
      --cert-key            Path to certificate key when using non-generated cert.
      --disable-telemetry   Disable telemetry.
   -h --help                Show this output.
      --open                Open in browser on startup. Does not work remotely.
      --bind-addr           Address to bind to in host:port. You can also use $PORT to override the port.
      --config              Path to yaml config file. Every flag maps directly to a key in the config file.
      --socket              Path to a socket (bind-addr will be ignored).
   -v --version             Display version information.
      --user-data-dir       Path to the user data directory.
      --extensions-dir      Path to the extensions directory.
      --list-extensions     List installed VS Code extensions.
      --force               Avoid prompts when installing VS Code extensions.
      --install-extension   Install or update a VS Code extension by id or vsix.
      --uninstall-extension Uninstall a VS Code extension by id.
      --show-versions       Show VS Code extension versions.
      --proxy-domain        Domain used for proxying ports.
 -vvv --verbose             Enable verbose logging.
```

于是，使用下面方式启动：（也可以使用./code-server --port 8888 --host 0.0.0.0）

```
./code-server --bind-addr 0.0.0.0:8888
***** Please use the script in bin/code-server instead!
***** This script will soon be removed!
***** See the release notes at https://github.com/cdr/code-server/releases/tag/v3.4.0
info  Using config file ~/.config/code-server/config.yaml
info  Using user-data-dir ~/.local/share/code-server
info  code-server 3.4.1 48f7c2724827e526eeaa6c2c151c520f48a61259
info  HTTP server listening on http://0.0.0.0:8888
info      - Using password from ~/.config/code-server/config.yaml
info      - To disable use `--auth none`
info    - Not serving HTTPS
```

 在 windows 的浏览器上即可访问了。初次访问，需要输入登录密码，在命令行输出上粘贴即可。

![](https://img-blog.csdnimg.cn/20200630210111748.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdXhpYW83MjM4NDY=,size_16,color_FFFFFF,t_70)

为了不让 ssh 会话断开重新登陆，可以使用 nohup ./code-server --bind-addr 0.0.0.0:8888 >/dev/null 2>&1 & 来启动。或者安装一个 Screen 软件，Screen 可以建立如 Windows 多任务的效果进行切换并且不会因为 SSH 断开而导致 code-server 自动退出。

注：screen 使用见：[https://blog.csdn.net/liuxiao723846/article/details/107384190](https://blog.csdn.net/liuxiao723846/article/details/107384190)

有一个疑问：为什么 code-server 的浏览器界面和桌面版的 vscode 如此相似（不，应该说是一模一样），而且在使用上也是一样额，要做到这中模仿基本上是不可能。不过想到 vscode 是一个开源项目，也有可能 coder 公司将其代码翻译成 web 版本... 直到有一天，看了这篇文章，才知道真正的原因 [https://blog.csdn.net/liuxiao723846/article/details/107051209](https://blog.csdn.net/liuxiao723846/article/details/107051209)

### 二、使用：

code-server 在使用上和 vscode 一模一样，早期的 code-server 还不支持在线安装插件，不过它提供了. VSIX 方式的安装，目前版本已经支持在线商店安装插件了。

**1、git 版本升级：**

远程 linux 上如果 git 的版本比较低，code-sever 在浏览器上也会提示，要求升级。linux 上升级 git 见：[https://blog.csdn.net/liuxiao723846/article/details/106940099](https://blog.csdn.net/liuxiao723846/article/details/106940099)

参考：

[https://juejin.im/post/5e2dd4566fb9a02fb75d6975](https://juejin.im/post/5e2dd4566fb9a02fb75d6975)

补充：

在 coder 官网上看到，除了 code-server 之外，他们还提供了一个 sshcode 的项目（[https://github.com/cdr/sshcode](https://github.com/cdr/sshcode)），大概看了介绍，应该是通过 ssh 的方式远程连接服务，然后提供在线 IDE 功能。（code-server 是通过 http + 密码的方式和远程建立连接的）
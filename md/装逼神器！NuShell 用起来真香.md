> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NzU0MzU0Nw==&mid=2651398336&idx=1&sn=f2b76332c079ae213896173c78bdece3&chksm=bd25ebd48a5262c2bbcb75fbe68f5da5ea2d2b47dd09d522ad7cf5af1bf8ba392b22cf9df29b&mpshare=1&scene=1&srcid=0708F9sMxyngRa636puIitR1&sharer_sharetime=1625740358211&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

*   创建具有现代感的灵活的跨平台 Shell
    
*   允许你将命令行应用程序与可理解数据结构的 Shell 进行混合和匹配
    
*   具有现代命令行应用程序提供的用户体验优化
    

**Ubuntu / Debian：**

```
sudo apt update
sudo apt install pkg-config libssl-dev -y
sudo apt install libxcb-composite0-dev libx11-dev -y


```

**基于 RHEL 的系统：**

```
sudo yum install libxcb openssl-devel libX11-devel -y


```

**苹果系统：**

```
brew install openssl cmake


```

**第 2 步：在 Linux 上安装 Nushell**

**下载最新的二进制归档文件：**

```
cd /tmp
curl -s  https://api.github.com/repos/nushell/nushell/releases/latest | grep browser_download_url |  cut -d  "  -f 4 | grep  linux.tar.gz  | wget -i -


```

**解压下载的文件：**

```
tar -xvf nu_*_linux.tar.gz


```

**将二进制文件复制到您的 PATH：**

```
sudo mv nu_*_linux/nushell-*/nu /usr/local/bin


```

Nushell 将在启动时在您的 PATH 中查找插件。虽然 Nushell 在没有它们的情况下会有一些功能，但要获得完整的功能，你需要将它们复制到您的路径中，以便加载它们。

```
sudo mv nu_*_linux/nushell-*/nu_plugin* /usr/local/bin


```

Fedora 用户可以使用 COPR repo 安装 Nushell：

```
sudo dnf copr enable atim/nushell -y && sudo dnf install nushell -y


```

**第 3 步：在 macOS 上安装 Nushell**

**对于二进制安装方法，请使用 brew：**

```
$ brew install nushell


```

**从二进制文件手动安装**

在 macOS 系统上运行以下命令来下载 Nushell 的最新版本：

```
cd /tmp
curl -s  https://api.github.com/repos/nushell/nushell/releases/latest | grep browser_download_url |  cut -d  "  -f 4 | grep  macOS.zip  | wget -i -


```

**解压下载的文件：**

```
unzip nu_*_macOS.zip


```

**将 nu 二进制文件复制到你的 PATH：**

```
sudo mv nu_*_macOS/nushell-*/nu /usr/local/bin


```

**复制 Nu 插件：**

```
sudo mv nu_*_macOS/nushell-*/nu_plugin* /usr/local/bin


```

**第 4 步：将用户 Shell 设置为 Nushell**

**创建一个名为 techviewleo 的新用户：**

```
$ sudo adduser techviewleo
Adding user `techviewleo  ...
Adding new group `techviewleo  (1000) ...
Adding new user `techviewleo  (1000) with group `techviewleo  ...
Creating home directory `/home/techviewleo  ...
Copying files from `/etc/skel  ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for techviewleo
Enter the new value, or press ENTER for the default
    Full Name []:
    Room Number []:
    Work Phone []:
    Home Phone []:
    Other []:
Is the information correct? [Y/n] y


```

**将用户默认 shell 设置为 Nu：**

```
sudo chsh -s /usr/local/bin/nu techviewleo


```

**切换到创建的用户帐户：**

```
$ su - techviewleo
Password:
Welcome to Nushell 0.28.0 (type  help  for more info)
/home/techviewleo>


```

**测试 ls 命令在 Nushell 中的工作方式：**

```
$ su - techviewleo
Password:
Welcome to Nushell 0.28.0 (type  help  for more info)
/home/techviewleo>


```

**运行效果展示：**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/kOTNkic5gVBGuwSJPX3z6rHhq7oYHn0rlVb0lCuB9ygSGbzw25dnCvQNjpELic6Vpzx5qDa7PhDJwo8jicks9U4ng/640?wx_fmt=gif)

最后附上 nushell 地址：https://github.com/nushell/nushell
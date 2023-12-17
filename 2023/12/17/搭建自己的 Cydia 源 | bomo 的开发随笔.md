> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.bombox.org](https://blog.bombox.org/2019-10-10/make-cydia-source/)

> 对于自己开发的插件，也需要使用软件源来维护和备份，可以向其他源一样，直接安装插件，这个记录一下搭建的过程

对于自己开发的插件，也需要使用软件源来维护和备份，可以向其他源一样，直接安装插件，这个记录一下搭建的过程

[](#准备 "准备")准备
--------------

1.  创建一个目录 `cydia`
2.  在`cydia`里面再创建一个目录`debs`，把所有的插件都放到里面
3.  新建一个文本文件`Release`，用于描述源信息
4.  在`cydia`添加一个图标`CydiaIcon.png`，在软件源列表显示

目录如下

```
- cydia
    |- CydiaIcon.png                    源图标
    |- Release                          描述源信息
    |- debs                             插件
        |- com.bomo.tweaksb_0.0.1-7_iphoneos-arm.deb
        |- com.bomo.tweakjianshu_0.0.1-7_iphoneos-arm.deb
```

`Release` 文件如下

```
Origin: BigBoss
Label: BigBoss
Suite: stable
Version: 1.0
Codename: BigBoss
Support: http://cydia.saurik.com/support/*
Architectures: darwin-arm iphoneos-arm
Components: main
Description: apps & tweaks
MD5Sum:
  be8806290d5904cdf45b542706f6a3ad 165020 main/binary-darwin-arm/Packages
  03026ac993187b0eecae50466f64fb3c 35049 main/binary-darwin-arm/Packages.gz
```

`Release`相关说明

```
必须
Origin: 软件源名称，可以使用中文（Cydia的软件源列表中显示的标题）
Label:  同上，也可以使用中文
Suite: 软件源的类型，比如正式源，测试源等，可以分别用stable, beta, unstable等来表示，一般填stable就可以了
Version: 版本号，这个其实不重要，随便填，一般都是写1.0
Codename: 代码代号，比如BigBoss的就写BigBoss，威锋的就写WeiPhone，也没什么限制，只能用英文
Architectures: 结构。iPhone平台统一写iphoneos-arm
Components: main
Description: 软件介绍，可以使用中文和html代码，具体能使用哪些代码在下面会介绍。

可选
Support: 支持，没什么作用，除非特别需要，否则可以不要这个。
MD5Sum: 不是必须的，但如果Packages文件位置不与Release文件在同一目录下，则必须有此项。另外，如果需要签名Release文件，也必须有这个。关于MD5Sum的格式，在下文也会介绍。
```

[](#打包插件 "打包插件")打包插件
--------------------

接下来我们需要 linux 环境，并且需要安装`dpkg-dev`，我这里使用 docker 创建的环境，其他 linux 环境也可以

```
apt-get update


apt-get install dpkg-dev
```

把我们准备好的文件拷贝到 linux 系统

```
docker cp ~/Desktop/cydia 75119aae5029:/var/cydia
```

生成 Packages

```
cd /var/cydia


dpkg-scanpackages debs /dev/null > Packages

tar zcvf Packages.gz Packages
bzip2 -k Packages Packages.bz2
```

上面步骤生成三个文件

*   Package
*   Packages.bz2
*   Package.gz

接下来是先生成一个密钥

提示输入`Email`和`Real name`，输入密码，生成过程可能需要等一会

签名 Package

```
gpg -abs -r "你刚才的输入的 Real name" -o Release.gpg Release
```

输入密码，生成`Release.gpg`，文件目录如下

```
- cydia
    |- CydiaIcon.png                    源图标
    |- Release                          描述源信息
    |- debs                             插件
        |- com.bomo.tweaksb_0.0.1-7_iphoneos-arm.deb
        |- com.bomo.tweakjianshu_0.0.1-7_iphoneos-arm.deb
        |- Release.gpg
    |- Packages
    |- Packages.bz2
    |- Packages.gz
    |- Release.gpg
```

接下来把整个`cydia`文件夹放到可以被 web 访问到的地方，访问的路径就是源地址，如`http://192.168.0.3:8080/cydia`，然后可以添加到 `Cydia` 安装插件了

[![](https://blog.bombox.org/images/post/cydia-source-list.png)

cydia-source-list

](https://blog.bombox.org/images/post/cydia-source-list.png "cydia-source-list")

[![](https://blog.bombox.org/images/post/cydia-myrepo-package-list.png)

cydia-myrepo-package-list

](https://blog.bombox.org/images/post/cydia-myrepo-package-list.png "cydia-myrepo-package-list")

[![](https://blog.bombox.org/images/post/cydia-myrepo-tweak.png)

cydia-myrepo-tweak

](https://blog.bombox.org/images/post/cydia-myrepo-tweak.png "cydia-myrepo-tweak")

[](#更新 "更新")更新
--------------

如果有 deb 更新，重新执行一下上面命令重新导出即可

```
cd cydia;

rm Packages; rm Packages.gz; rm Packages.bz2;


dpkg-scanpackages debs /dev/null > Packages && tar zcvf Packages.gz Packages && bzip2 -k Packages Packages.bz2
```

然后替换原来的文件即可，在 Cydia 就会收到更新

[![](https://blog.bombox.org/images/post/cydia-repo-update.png)

cydia-repo-update

](https://blog.bombox.org/images/post/cydia-repo-update.png "cydia-repo-update")

[](#引用 "引用")引用
--------------

1.  [http://www.saurik.com/id/7](http://www.saurik.com/id/7)
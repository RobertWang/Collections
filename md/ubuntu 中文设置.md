> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/xlhblog/archive/2011/05/13/2045651.html)

【参考】

[http://wiki.ubuntu.org.cn/Locale](http://wiki.ubuntu.org.cn/Locale)

**Locale 配置文件在 /etc/default/locale**

linux 文件系统的一些概念，linux 的文件系统的文件名是基于字节流的，也就是说文件名的编码以 byte 为单位读取的，所以文件名可以是多字节编码方案，比如 gb18030, utf8，不能是宽字节编码，比如如 windows 使用的 utf16。所以只要是多字编码方案的文件名，linux 都是可以读取的。   
1）为 glib 的国际化支持产生国标编码支持，ubuntu 系统默认状态下是没有 GB18030 和 GBK 的本地 locale 的，所以为了设置 locale 为 zh_CN.GB18030，我们需要先为 glib 产生 GB18030 和 GBK 编码的支持。   
1、首先设置 sudo vi /var/lib/locales/supported.d/local   
添加一行 zh_CN.GBK GBK 和 zh_CN.GB18030 GB18030，**把原来的 UTF-8 那一行仍旧保留在最后！**  
2、sudo locale-gen， 或者可以直接 sudo locale-gen zh_CN.GB18030 和 sudo locale-gen zh_CN.GBK

**安装 Chinese 包**

先更新源：在 sudo gedit /etc/apt/soures.list 里面添加如下内容

deb http://ubuntu.cn99.com/ubuntu/ gutsy main restricted universe multiverse   
deb http://ubuntu.cn99.com/ubuntu/ gutsy-security main restricted universe multiverse   
deb http://ubuntu.cn99.com/ubuntu/ gutsy-updates main restricted universe multiverse   
deb http://ubuntu.cn99.com/ubuntu/ gutsy-proposed main restricted universe multiverse   
deb http://ubuntu.cn99.com/ubuntu/ gutsy-backports main restricted universe multiverse   
deb-src http://ubuntu.cn99.com/ubuntu/ gutsy main restricted universe multiverse   
deb-src http://ubuntu.cn99.com/ubuntu/ gutsy-security main restricted universe multiverse   
deb-src http://ubuntu.cn99.com/ubuntu/ gutsy-updates main restricted universe multiverse   
deb-src http://ubuntu.cn99.com/ubuntu/ gutsy-proposed main restricted universe multiverse   
deb-src http://ubuntu.cn99.com/ubuntu/ gutsy-backports main restricted universe multiverse   
deb http://ubuntu.cn99.com/ubuntu-cn/ gutsy main restricted universe multiverse

然后 system->system manager->language support, 安装 chinese 语言包，更新即可。原来以为安装的时候选择中文就可以了，谁知道安装好后，还需要继续安装 chinese 语言包。或者用命令单独下载中文就直接 sudo apt-get install language-pack-zh.

### 二、到底什么是 locale？

locale 这个单词中文翻译成地区或者地域，其实这个单词包含的意义要宽泛很多。Locale 是根据计算机用户所使用的语言，所在国家或者地区，以及当地的文化传统所定义的一个软件运行时的语言环境。

这个用户环境可以按照所涉及到的文化传统的各个方面分成几个大类，通常包括用户所使用的语言符号及其分类 (LC_CTYPE)，数字 (LC_NUMERIC)，比较和排序习惯(LC_COLLATE)，时间显示格式(LC_TIME)，货币单位(LC_MONETARY)，信息主要是提示信息, 错误信息, 状态信息, 标题, 标签, 按钮和菜单等(LC_MESSAGES)，姓名书写方式(LC_NAME)，地址书写方式(LC_ADDRESS)，电话号码书写方式 (LC_TELEPHONE)，度量衡表达方式(LC_MEASUREMENT)，默认纸张尺寸大小(LC_PAPER) 和 locale 对自身包含信息的概述(LC_IDENTIFICATION)。

所以说，locale 就是某一个地域内的人们的语言习惯和文化传统和生活习惯。一个地区的 locale 就是根据这几大类的习惯定义的，这些 locale 定义文件放在 / usr/share/i18n/locales 目录下面，例如 en_US, zh_CN and de_DE@euro 都是 locale 的定义文件，这些文件都是用文本格式书写的，你可以用写字板打开，看看里边的内容，当然出了有限的注释以外，大部分东西可能你都看不懂，因为是用的 Unicode 的字符索引方式。

对于 de_DE@euro 的一点说明，@后边是修正项，也就是说你可以看到两个德国的 locale：

```
/usr/share/i18n/locales/de_DE@euro
/usr/share/i18n/locales/de_DE


```

打开这两个 locale 定义，你就会知道它们的差别在于 de_DE@euro 使用的是欧洲的排序、比较和缩进习惯，而 de_DE 用的是德国的标准习惯。

上面我们说到了 zh_CN.GB18030 的前半部分，后半部分是什么呢？大部分 Linux 用户都知道是系统采用的字符集。

### 七、怎样设定 locale 呢？

设定 locale 就是设定 12 大类的 locale 分类属性，即 12 个 LC_*。除了这 12 个变量可以设定以外，为了简便起见，还有两个变量：LC_ALL 和 LANG。它们之间有一个优先级的关系：

```
LC_ALL>LC_*>LANG

```

可以这么说，LC_ALL 是最上级设定或者强制设定，而 LANG 是默认设定值。

*   *   如果你设定了 LC_ALL＝zh_CN.UTF-8，那么不管 LC_* 和 LANG 设定成什么值，它们都会被强制服从 LC_ALL 的设定，成为 zh_CN.UTF-8。
    *   假如你设定了 LANG＝zh_CN.UTF-8，而其他的 LC_*=en_US.UTF-8，并且没有设定 LC_ALL 的话，那么系统的 locale 设定以 LC_*=en_US.UTF-8。
    *   假如你设定了 LANG＝zh_CN.UTF-8，而其他的 LC_*，和 LC_ALL 均未设定的话，系统会将 LC_* 设定成默认值，也就是 LANG 的值 zh_CN.UTF-8 。
    *   假如你设定了 LANG＝zh_CN.UTF-8，而其他的 LC_CTYPE=en_US.UTF-8，其他的 LC_*，和 LC_ALL 均未设定的话，那么系统的 locale 设定将是：LC_CTYPE=en_US.UTF-8，其余的 LC_COLLATE，LC_MESSAGES 等等均会采用默认值，也就是 LANG 的值，也就是 LC_COLLATE＝LC_MESSAGES＝……＝ LC_PAPER＝LANG＝zh_CN.UTF-8。

所以，locale 是这样设定的：

*   *   如果你需要一个纯中文的系统的话，设定 LC_ALL= zh_CN.XXXX，或者 LANG= zh_CN.XXXX 都可以，当然你可以两个都设定，但正如上面所讲，LC_ALL 的值将复盖所有其他的 locale 设定，不要作无用功。
    *   如果你只想要一个可以输入中文的环境，而保持菜单、标题，系统信息等等为英文界面，那么只需要设定 LC_CTYPE＝zh_CN.XXXX，LANG= en_US.XXXX 就可以了。这样 LC_CTYPE＝zh_CN.XXXX，而 LC_COLLATE＝LC_MESSAGES＝……＝ LC_PAPER＝LANG＝en_US.XXXX。

```
export LC_CTYPE="zh_CN.UTF-8"
使用export命令

```

*   *   假如你高兴的话，可以把 12 个 LC_* 一一设定成你需要的值，打造一个古灵精怪的系统：

```
LC_CTYPE＝zh_CN.GBK/GBK(使用中文编码内码GBK字符集)；
LC_NUMERIC=en_GB.ISO-8859-1(使用大不列颠的数字系统)
LC_MEASUREMEN=de_DE.ISO-8859-15@euro(德国的度量衡使用ISO-8859-15字符集)
罗马的地址书写方式，美国的纸张设定……。估计没人这么干吧。

```

```
注意，修改这些LC_XXX 的位置在 /etc/environment

```

```
还有一个地方叫做 /etc/default/locale

```

```
但是Ubuntu Server的console就是不支持中文，因此装完机要改回英文

```

```
用vim配置语言环境变量
vim /etc/environment
改成：
LANG=”en_US.UTF-8″
LANGUAGE=”en_US:en”
sudo vim /var/lib/locales/supported.d/local
加一行
en_US.UTF-8 UTF-8
保存后，执行命令：
sudo locale-gen
sudo vim /etc/default/locale
修改为：
LANG=”en_US.UTF-8″
LANGUAGE=”en_US:en”
重启Ubuntu Server
sudo reboot
至此 方格乱码解决


```
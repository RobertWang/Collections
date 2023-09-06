> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/mUR89U-3YSTtPZY3JqOvMA)

前言
==

> 什么是 ZTS？ZTS 就是 Zend Thread Safely 的简写，意思是说这个版本的 PHP 是基于线程安全的，相对的有一个版本叫做 NTS，就是 None Thread Safely，意思是非线程安全的

> php 为什么默认禁用 ZTS？PHP 最初是作为 CGI 二进制文件开始的，然后作为 Apache 的一个模块。这两种方式都不需要 PHP 成为线程运行时，因为它们都将按顺序处理请求

一、下载 php 源码
===========

我这里下载的是 php8.1，系统用的是 DeBian

```
cd /usr/local/src
wget https://www.php.net/distributions/php-8.1.21.tar.gz

```

二、解压源码
======

```
tar -zxvf ./php-8.1.1.tar.gz
cd php-8.1.1

```

三、执行编译
======

```
./configure --prefix=/usr/local/php81 --enable-debug --enable-zts --with-config-file-path=/usr/local/php81/etc --with-config-file-scan-dir=/usr/local/php81/conf.d --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --with-iconv=/usr/local --with-freetype=/usr/local/freetype --with-jpeg --with-zlib --enable-xml --disable-rpath --enable-bcmath --enable-shmop --enable-sysvsem --with-curl --enable-mbregex --enable-mbstring --enable-intl --enable-pcntl --enable-ftp --enable-gd --with-openssl --with-mhash --enable-pcntl --enable-sockets --with-zip --enable-soap --with-gettext --enable-opcache --with-xsl --with-pear

```

这里要特别注意 **--enable-debug --enable-zts,** 只有加入 enable-zts，编译起来的 php 才是 ZTS 版本。  
本来以为很顺利，没变想到安装过程报了很多错误，老俊见招拆招，花了不少时间，总算顺利安装成功。错误一般都是发生在编译阶段。下面我一一记录。后面如果有同学遇到同样问题，可以参考我的解决方法。

No package 'sqlite3' found
--------------------------

如果遇到：“No package 'sqlite3' found” 的报错，则运行 apt-get remove sqlite3 删除 sqlite3  
然后运行：

```
apt install sqlite3 libsqlite3-dev

```

No package 'zlib' found
-----------------------

如果报 zlib 找不到，则运行

```
apt-get install zlib1g-dev

```

No package 'libcurl' found
--------------------------

如果报 libcurl 找不到，则运行

```
apt-get install libcurl4-openssl-dev

```

No package 'libpng' found
-------------------------

如果 libpng 找不到，则运行

```
apt install libpng-dev

```

configure: error: iconv does not support errno
----------------------------------------------

```
# cd /usr/local/src/
# wget https://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.16.tar.gz
# tar zxvf ./libiconv-1.16.tar.gz
# cd libiconv-1.16
# ./configure --prefix=/usr
# make & make install

```

No package 'freetype2' found
----------------------------

```
apt install libfreetype-dev libfreetype6-dev

```

No package 'libxslt' found
--------------------------

```
apt install libxslt-dev

```

No package 'libzip' found
-------------------------

```
apt install libzip-dev

```

> 注意：如果您使用的是 Centos，centos 一般是用 yum 来安装软件包，yum 源的包名和 Debian 是有点不一样的。一般需要把 dev 改为 devel 即可，如在 Debian 中用 apt install libzip-dev，而在 centos 中可能是用 yum install libzip-level

构建
==

```
make && make install

```

```
ln -s /usr/local/php81/bin/php /usr/local/bin/php

```

![](https://mmbiz.qpic.cn/mmbiz_png/x0Iuy6awYQfcpuNFswSLGE2oXI23EialK2wEHM5qoKWc1VNRmickrPVch962n0x4kw7g8swC4NiartROaRiayI8IIA/640?wx_fmt=png)image.png 老俊说技术公众号
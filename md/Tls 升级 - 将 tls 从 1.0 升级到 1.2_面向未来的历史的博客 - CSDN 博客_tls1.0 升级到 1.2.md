> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/a1368783069/article/details/85064682)

背景： 某人在开发微信小程序时，调用测试环境的 https 接口，该接口由 [nginx](https://so.csdn.net/so/search?q=nginx&spm=1001.2101.3001.7020) 提供代理服务，报错，说是不支持 tls1 ，需要升级到 tls1.2

查看 [ssl](https://so.csdn.net/so/search?q=ssl&spm=1001.2101.3001.7020) 版本
========================================================================

### 1 [cmd](https://so.csdn.net/so/search?q=cmd&spm=1001.2101.3001.7020)

[openssl](https://so.csdn.net/so/search?q=openssl&spm=1001.2101.3001.7020) s_client -connect domain:443 -tls1 （-tls1_1, -tls1_2）  
其中 domain 表示 nginx 域名配置中使用 https 的域名，以上命令可以查看，目前 nginx 支持的 tls 版本。

### 2 chrome

依次打开： 右键 ->审查 (inspect) -> 安全(security)。

然后在当前页面访问配置的域名。然后 安全 (security) 下点击要查看的域名，即可在右侧的连接中的协议下，看到 tls 版本号。  
![](https://img-blog.csdnimg.cn/20181218151934818.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ExMzY4NzgzMDY5,size_16,color_FFFFFF,t_70)

升级操作总共分三步：  
1 升级 openssl 2 重新编译 nginx （这一步可以尝试不操作，因为自己看 openssl 是以动态库的方式使用，但是自己没有试过跳过这一步） 3，修改配置文件

1 升级安装 openssl
==============

常见的比较方便的做法是使用系统的软件管理工具， 比如 apt， yum 等。具体做法，可以在网上找资料。  
此处还是使用源码手动安装：

### 下载 openssl

在官网下载相应源码， [https://www.openssl.org/source/](https://www.openssl.org/source/)

解压后安装

```
./config 
make
sudo make install

```

默认的安装目录：

```
/usr/local/ssl/bin/openssl
/usr/local/ssl/include/openssl

```

所以现在可以查看你安装的 openssl 版本：

```
/usr/local/ssl/bin/openssl version -a

```

### 创建软连接

依次执行以下操作

```
mv /usr/bin/openssl /usr/bin/openssl.old
mv /usr/include/openssl /usr/include/openssl.old
ln -s /usr/local/ssl/bin/openssl /usr/bin/openssl
ln -s /usr/local/ssl/include/openssl/ /usr/include/openssl

```

然后

```
sudo vim /etc/ld.so.conf

```

将 `/usr/local/ssl/lib` 添加到上面文件中，报错退出，然后通过以下命令，可以看到自己安装的 openssl 版本号。

```
openssl version -a

```

2 nginx 重编译
===========

由于之前别人是使用 apt 之间安装的 nginx，当然可以直接重新编译生成 nginx，然后替换 sbin/nginx, 即可，具体实现，参见我之前文章，  
给网站添加 ssl 证书（https）[https://blog.csdn.net/a1368783069/article/details/80719768](https://blog.csdn.net/a1368783069/article/details/80719768) 中的 nginx 安装 ssl 模块 方法。  
个人建议安装 nginx 使用手动编辑安装。  
从官网下载 nginx 源码。解压文件，使用默认安装。

```
./configure
make
sudo make install

```

执行 `sudo /usr/local/nginx/sbin/nginx` 会提示没有安装与 ssl 相关的 modules， 则重新编译 nginx

> 1、 生成文件 进入 nginx 安装文件目录，执行：
> 
> `./configure --with-http_ssl_module`  
> `make`  
> 1 2 2、 替换 nginx  
> 首先关闭 nginx 服务（很重要）  
> 在 nginx 安装文件目录下面，会生成一个 objs 目录，将 objs/nginx 替换  
> /usr/local/nginx/sbin/nginx(两者都是 nginx 的可执行文件)。 重启 nginx 服务就行。  
> /usr/local/nginx/sbin/nginx -s reload

以上的操作，主要是理解 nginx 这种更新操作过程。  
如果想一步到位，安装 nginx，检测环境的时候，可以直接

```
./configure --with-http_ssl_module
make
sudo make install

```

3 更改 nginx 配置文件
===============

原来的配置文件中，  
对 ssl 的设置时

```
ssl_protocols  TLSv1 TLSv1.1 TLSv1.2 ;

```

**所有配置文件**中的 ssl_protocols 都改为（有的会配置多个网站，每个都需要改）：

```
ssl_protocols   TLSv1.2 TLSv1.1 TLSv1 ;

```

或者只保留 TLSv1.2 。  
一般，可以检测配置文件是否有问题

```
/usr/local/nginx/sbin/nginx   -t

```

然后查看 ssl 版本号。

其他
==

查看 nginx 安装使用的配置，以及添加的模块

```
nginx -V

```

参考文章：  
为 CentOS 升级 OpenSSL 让 Nginx 支持 TLS 1.2  
[https://www.sunbloger.com/2016/12/26/543.html](https://www.sunbloger.com/2016/12/26/543.html)

nginx 配置 TLS1.2 无效总是 TLS1.0 [https://blog.csdn.net/wanderstarrysky/article/details/53336129](https://blog.csdn.net/wanderstarrysky/article/details/53336129)
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/335485834)

Composer 是 PHP 的包管理（依赖管理）工具，类似 node.js 的 npm 。如果在一个 PHP 项目中一些库依赖于其他库，就声明所依赖的库，Composer 就会找出相应版本的包并安装，默认情况下下载到项目的某个目录中（例如 vendor）进行安装。使用 Composer 可以很轻松的使用一个命令将其他人的优秀代码引用到我们的项目中来。

一、检查 PHP 环境
-----------

首先要确保已经安装好了 PHP 环境，在命令行输入命令：`php -v` 要能正确输出版本号信息。

然后在 Linux 系统中下载 composer.phar ，移动到全局目录中并改名为 composer ，就可以全局调用 composer 了。

二、下载 Composer。
--------------

首先下载安装脚本 composer-setup.php 到当前目录，然后执行安装，简单地检测 php.ini 中的参数设置，如果某些参数未正确设置则会给出警告。

```
php -r "copy('https://install.phpcomposer.com/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"

```

三、全局安装 composer
---------------

在 Linux 环境中，用以下命令下载最新版本的 composer.phar 文件到当前目录。然后移动到 /usr/local/bin 目录并更改名称为 composer ，这样就可以全局运行 composer 了。

```
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer

```

用 `composer --version` 命令查看 composer 版本，或直接用 `composer` 命令查看。

四、配置 composer 源地址
-----------------

由于网络原因，默认的官方 php 源码包在国内速度很慢，需要更换为国内镜像，参数 -g 表示全局配置。按自己的网络情况选择一个速度比较快的。

```
#阿里云
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
#腾讯云
composer config -g repos.packagist composer https://mirrors.cloud.tencent.com/composer/
#中国全量镜像
composer config -g repo.packagist composer https://packagist.phpcomposer.com

```

查看 composer 全局配置命令：

```
composer config -l -g

```

查看全局配置，可以看到仓库 URL 地址 `repositories.packagist.org.url` 已经变成了阿里云的镜像。

![](https://pic1.zhimg.com/v2-8c16d1d428c1d04e8f7f102fe71ca2b4_b.png)

如果需要解除镜像并恢复到 packagist 官方仓库，就用 composer 默认值重置源地址。命令如下：

```
composer config -g --unset repos.packagist

```

五、用 composer 配置 Laravel 项目示例
----------------------------

```
composer create-project --prefer-dist laravel/laravel my_site_dir "7.2.*"

```

laravel/laravel：composer 安装包的名称，表示创建一个 laravel 项目

my_site_dir：项目目录

--prefer-dist：使用压缩版

7.2.*：表示要安装的 laravel 版本

六、问题排除
------

**1、警告信息：**

> Do not run Composer as root/super user! See [https://getcomposer.org/root](https://link.zhihu.com/?target=https%3A//getcomposer.org/root) for details

**解决方法：**

某些 Composer 命令（包括 exec，install 和 update）允许第三方代码在您的系统上执行。 这来自其 “插件” 和“脚本”功能。 插件和脚本对运行 Composer 的用户帐户具有完全访问权限。 因此，强烈建议避免以超级用户 / root 身份运行 Composer。

您可以在包安装或更新期间使用以下语法禁用插件和脚本，以便只执行 Composer 的代码，而不执行第三方代码

```
composer install --no-plugins --no-scripts ...
composer update --no-plugins --no-scripts ...

```

官方文档：[https://getcomposer.org/doc/faqs/how-to-install-untrusted-packages-safely.md](https://link.zhihu.com/?target=https%3A//getcomposer.org/doc/faqs/how-to-install-untrusted-packages-safely.md)

**2、警告信息：**

> As there is no 'unzip' command installed zip files are being unpacked using the PHP zip extension.  
> This may cause invalid reports of corrupted archives. Besides, any UNIX permissions (e.g. executable) defined in the archives will be lost.  
> Installing 'unzip' may remediate them.

**解决方法：**

表示系统需要安装 unzip ，在 CentOS 中安装命令：`yum install -y unzip zip`

七、github 访问速度慢的问题
-----------------

1、命令提示符中输入 `ping github.com`，查看 IP 地址，简单测试一下访问 github 的情况。

2、获取 Github 相关网站的 ip

访问 [https://www.ipaddress.com](https://link.zhihu.com/?target=https%3A//www.ipaddress.com)，在搜索框内分别输入 [http://github.global.ssl.fastly.net](https://link.zhihu.com/?target=http%3A//github.global.ssl.fastly.net) 和 [http://github.com](https://link.zhihu.com/?target=http%3A//github.com)，查询 IP 地址，得到如下 IP 和域名的映射关系：（不同的网络查询到的结果可能不一致，以自己查询到的结果为准。）

```
140.82.113.3    github.com
140.82.113.3    www.github.com
140.82.112.5    api.github.com
199.232.69.194  github.global.ssl.fastly.net

```

3、修改本地 hosts 文件

A. Windows 系统的 hosts 文件是： C:\Windows\System32\drivers\etc\hosts（此文件没有扩展名）

以管理员身份运行记事本文本编辑器，编辑 hosts 文件，在末尾增加上一步查询到的 IP 地址与 Github 相关网站映射。

B. Linux 系统的 hosts 文件是： /etc/hosts，编辑此文件：`vi /etc/hosts` ，在末尾增加上一步查询到的 IP 地址与 Github 相关网站映射。

4、刷新 DNS 缓存，立即生效

A. 在 windows 中打开 cmd，运行命令：`ipconfig /flushdns`

B. 在 Linux 中运行命令：`/etc/init.d/network restart`

5、命令提示符中输入 ping [http://github.com](https://link.zhihu.com/?target=http%3A//github.com)，简单测试修改后的效果。

八、Composer 的使用
--------------

1、composer 包依赖关系

项目目录下的 composer.json 文件，其中必须的是 require 项，描述了项目的依赖关系。此时只要运行 `composer install` 就可以安装 requiere 项中所有的依赖包。冒号后的符号及数字是版本约束。

```
// laravel项目的 composer.js 示例 
 {
    "require": {
        "php": "^7.1.3",
        "barryvdh/laravel-debugbar": "^3.2",
        "barryvdh/laravel-ide-helper": "^2.6",
        "fideloper/proxy": "^4.0",
        "fruitcake/laravel-cors": "^2.0",
        "kalnoy/nestedset": "^5.0",
        "laravel/framework": "^6.0",
        "laravel/passport": "^7.2"
    }
 }

```

2、require 命令

如果没有在 composer.json 文件手动添加某个依赖包信息，但是要快速安装此依赖包时，可以使用 `require` 命令。

```
# requier 命令示例 
composer require fruitcake/laravel-cors

```

3、update 更新包命令

```
# 按照 composer.json 文件，更新所有的依赖包
composer update
# 更新指定的包
composer update laravel/passport
# 更新指定的多个包
composer update laravel/passport fruitcake/laravel-cors
# 通过通配符匹配包,即更新 Laravel/ 下的项目依赖的所有的包
composer update laravel/*

```

4、remove 命令

移除一个包及其依赖，如果依赖被其他包使用，则无法移除。

5、search 命令：

搜索包，输出包及其描述信息，如果只想输出包名可以使用 --only-name 参数

6、show 命令

列出当前项目使用到包的信息：

```
# 列出所有已经安装的包
composer show
# 通过通配符进行筛选
composer show laravel/*
# 显示具体某个包的信息
composer show laravel/passport

```

```
composer clearcache

```

------------------END--------------------
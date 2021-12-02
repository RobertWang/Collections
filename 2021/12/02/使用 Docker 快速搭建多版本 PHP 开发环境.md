> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/-vOmWMmzkzAEPbcbJQfmJw)

> 基于 Docker 安装 PHP 5.6x 和 PHP 7.2.x

**文章目录：**

*   目标
    
*   下载
    
*   代理设置
    
*   配置环境
    

*   PHP 7.2.x，占用本地端口 8081
    
*   PHP 5.6.x，占用本地端口 8082
    

*   端口映射
    

*   local.php72.com -> 127.0.0.1:8081
    
*   local.php56.com -> 127.0.0.1:8082
    

*   备注
    

*   docker-compose 相关命令
    
*   php7-2-x 目录介绍
    
*   php5-6-x 目录介绍
    
*   zip 文件如何生成的？
    

文章中使用的软件：

*   Mac：11.4(macOS Big Sur) ，处理器为：Intel Core。
    
*   Docker：3.3.3
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/go9jpG3BuhQS1Eemp1t8y7L4p11pmMMpjtYZicicEava8NayFAa9EIEmjmgKjJbraj1yQZRxC6YQes2WNyQyiaSKQ/640?wx_fmt=png)

目标
--

*   支持 PHP 5.6.x 环境
    
*   支持 PHP 7.2.x 环境
    

下载
--

Docker 软件下载安装，不做过多解释，一步步安装即可。

下载地址：Docker 官网 [1]

代理设置
----

```
"registry-mirrors" : [
    "http://registry.docker-cn.com",
    "http://hub-mirror.c.163.com"
  ],


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/go9jpG3BuhQS1Eemp1t8y7L4p11pmMMpAnSCe6dIEtjdVS8zFicQDbpyic0YQibWpxtwmyE5oqcyQ5sWWQSOteB3Q/640?wx_fmt=png)

配置环境
----

### PHP 7.2.x，占用本地端口 8081

1.  启动 docker；
    
2.  下载压缩包：php7-2-x.zip 并进行解压；
    
3.  进入到 php7-2-x 目录，直接运行 `docker-compose up` 即可；
    
4.  浏览器输入：http://127.0.0.1:8081/；
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/go9jpG3BuhQS1Eemp1t8y7L4p11pmMMpHMxicKxFicKiboMgiaIPEUkd4DWsZapJx4CYSv7h9Ec7ibfBLIQQsb2Z3QQ/640?wx_fmt=png)

### PHP 5.6.x，占用本地端口 8082

1.  启动 docker；
    
2.  下载压缩包：php5-6-x.zip 并进行解压；
    
3.  进入到 php5-6-x 目录，直接运行 `docker-compose up` 即可；
    
4.  浏览器输入：http://127.0.0.1:8082/；
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/go9jpG3BuhQS1Eemp1t8y7L4p11pmMMp45JLeXp7bdibgFC6Ss6TDuZESia1iaZ4Wykwlb0ghywfUSYLvosQnVLEA/640?wx_fmt=png)

端口映射
----

### local.php72.com -> 127.0.0.1:8081

因为在 `/etc/hosts` 文件中不能做端口映射，需要借助其他工具。

我借助的工具为 Chrome 浏览器插件：`Simple Proxy`。

下载方式：

*   Chrome 应用商店下载，搜索 `Simple Proxy`。
    
*   加载本地扩展程序，下载地址：chrome-simply-proxy[2]
    

看下安装好界面：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/go9jpG3BuhQS1Eemp1t8y7L4p11pmMMpTVkrFawQI0kVfyQrRG0ibrBQlltAtgeXhE3ZePdueyYLJD6iag8fialhg/640?wx_fmt=png)

配置成功后，访问 http://local.php72.com/

![](https://mmbiz.qpic.cn/sz_mmbiz_png/go9jpG3BuhQS1Eemp1t8y7L4p11pmMMpHMxicKxFicKiboMgiaIPEUkd4DWsZapJx4CYSv7h9Ec7ibfBLIQQsb2Z3QQ/640?wx_fmt=png)

### local.php56.com -> 127.0.0.1:8082

同上。

备注
--

### docker-compose 相关命令

*   docker-compose up 构建容器 参数 [-d] 为后台运行
    
*   docker-compose start 启用容器
    
*   docker-compose stop 停止容器
    
*   docker-compose restart 重启容器
    
*   docker-compose down 删除容器
    
*   docker-compose ps 查看当前容器状态
    

### php7-2-x 目录介绍

```
.
├── docker-compose.yml
├── log
│   └── nginx
│       └── local.php72.com_access.log
├── phpdocker
│   ├── README.html
│   ├── README.md
│   ├── nginx
│   │   └── default.conf
│   └── php-fpm
│       ├── Dockerfile
│       └── php-ini-overrides.ini
└── web
    └── phpinfo
        └── index.php


```

1、docker-compose.yml，容器编排的配置文件，文件无需更改。

```
version: "3.1"
services:

    webserver:
      image: nginx:alpine
      container_name: php7-2-x-webserver
      working_dir: /application
      volumes:
          - .:/application
          - ./phpdocker/nginx:/etc/nginx/conf.d
      ports:
       - "8081:80"

    php-fpm:
      build: phpdocker/php-fpm
      container_name: php7-2-x-php-fpm
      working_dir: /application
      volumes:
        - .:/application
        - ./phpdocker/php-fpm/php-ini-overrides.ini:/etc/php/7.2/fpm/conf.d/99-overrides.ini

```

2、log/nginx 为日志目录，包含 xxx_access.log 和 xxx_php_errors.log，xxx 为配置的虚拟域名。

3、phpdocker/nginx 为虚拟域名配置目录，其中 default.conf 配置的虚拟域名为 local.php72.com，不过多解释，大家一看就懂，其他目录和文件无需调整。

```
server {
    listen 80;

    server_name local.php72.com;

    client_max_body_size 108M;

    access_log /application/log/nginx/${server_name}_access.log;

    root /application/web/phpinfo;
    index index.php;

    # try to serve file directly, fallback to index.php
    location / {
        try_files $uri /index.php$is_args$args;
    }

    if (!-e $request_filename) {
        rewrite ^.*$ /index.php last;
    }

    location ~ \.php$ {
        fastcgi_pass php-fpm:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PHP_VALUE "error_log=/application/log/nginx/${server_name}_php_errors.log";
        fastcgi_buffers 16 16k;
        fastcgi_buffer_size 32k;
        include fastcgi_params;
    }

    }

```

4、web 为代码仓库目录，其中 phpinfo 为域名 local.php72.com 指向的代码目录。

如果想要新增虚拟域名（local.abc.com）配置时，只需 3 步：

1.  将代码文件夹 abc 放到 web 目录下；
    
2.  新增文件 local.abc.com.conf，在配置文件中将代码目录指向到 abc 目录；
    
3.  重启容器 docker-compose restart；
    

### php5-6-x 目录介绍

同上。

### zip 文件如何生成的？

大家可能会有疑问，zip 文件如何生成的，如果我想搭建其他版本的环境怎么办？

这些文件是在线生成的，网址：https://phpdocker.io/generator[3]

![](https://mmbiz.qpic.cn/sz_mmbiz_png/go9jpG3BuhQS1Eemp1t8y7L4p11pmMMpCnLsbW3hvjB21wufwT7kQP30xOjboNVNfLIIoJxiaXgOaBRXbdN6uicQ/640?wx_fmt=png)

支持的 PHP 版本有：`5.6.x`、`7.0.x`、`7.1.x`、`7.2.x`、`7.3.x`、`7.4.x` 等。

同时还支持 `MySQL` 、`MariaDB`、`Elasticsearch` 等。

按需选择后，点击 `Generate project archive` 即可生成压缩包。

上面的 php5-6-x.zip 和 php7-2-x.zip 就是这种方式生成的，仅仅是对其进行微调，比如配置 log 目录，web 目录等。

更多功能，大家去探索吧。

下载文章中用到的 zip 文件，请在「新亮笔记」公众号内回复：phpdocker 即可。

想了解更多技术细节，快来加入到我的星球吧。

### 参考资料

[1]

Docker 官网: https://www.docker.com/products/docker-desktop

[2]

chrome-simply-proxy: https://github.com/Riant/chrome-simply-proxy

[3]

https://phpdocker.io/generator: https://phpdocker.io/generator
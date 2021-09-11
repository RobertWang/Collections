> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/105340106)

nginx 在绝大数的场景中我们使用其用于做 web 中间件或反向代理使用，但是 nginx 实际上也提供了正向代理的功能。下面我们来进行 nginx 正向代理配置操作，以便大家能够掌握 nginx 正向代理配置方法。

**第一步：获取 nginx 正向代理模块**
-----------------------

# git clone [https://github.com/chobits/ngx_http_proxy_connect_module](https://link.zhihu.com/?target=https%3A//github.com/chobits/ngx_http_proxy_connect_module)

**第二步：下载 nginx 源码包**
--------------------

# wget [http://nginx.org/download/nginx-1.9.12.tar.gz](https://link.zhihu.com/?target=http%3A//nginx.org/download/nginx-1.9.12.tar.gz)

# tar xf nginx-1.9.12.tar.gz

**第三步：通过补丁方法把上述下载的正向代理模块导入到 nginx 模块存储目录**
------------------------------------------

# cd nginx-1.9.12/

# patch -p1 < /root/ngx_http_proxy_connect_module/patch/proxy_connect.patch

**第四步：编译安装 nginx**

# yum -y install openssl-devel zlib-devel prce-devel

# ./configure --add-dynamic-module=/root/ngx_http_proxy_connect_module

# make && make install

**第五步：配置所允许通过代理主机的主机列表**

# cat /usr/local/nginx/conf/client-allow.conf

allow 127.0.0.1;

allow 192.168.216.1;

allow 192.168.216.185;

**第六步：修改 nginx 配置文件**

# cat /usr/local/nginx/conf/nginx.conf

#user nobody;

worker_processes 1;

#error_log logs/error.log;

#error_log logs/error.log notice;

#error_log logs/error.log info;

#pid logs/nginx.pid;

load_module /usr/local/nginx/modules/ngx_http_proxy_connect_module.so; #位置注意

events {

worker_connections 1024;

}

http {

include mime.types;

default_type application/octet-stream;

#log_format main '$remote_addr - $remote_user [$time_local]"$request" '

# '$status $body_bytes_sent"$http_referer" '

# '"$http_user_agent" "$http_x_forwarded_for"';

#access_log logs/access.log main;

sendfile on;

#tcp_nopush on;

#keepalive_timeout 0;

keepalive_timeout 65;

#gzip on;

server {

listen 8080; #代理端口

resolver 119.29.29.29; #域名解析服务器

proxy_connect;

proxy_connect_allow 443 563;

proxy_connect_connect_timeout 10s;

proxy_connect_read_timeout 10s;

proxy_connect_send_timeout 10s;

location / {

proxy_pass http://$host;

proxy_set_header Host $host;

}

include client-allow.conf; #主机白名单

deny all; #除了主机白名单中的主机，拒绝所有

#error_page 404 /404.html;

# redirect server error pages to the static page /50x.html

#

error_page 500 502 503 504 /50x.html;

location = /50x.html {

root html;

}

# proxy the PHP scripts to Apache listening on 127.0.0.1:80

#

#location ~ .php$ {

# proxy_pass http://127.0.0.1;

#}

# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000

#

#location ~ .php$ {

# root html;

# fastcgi_pass 127.0.0.1:9000;

# fastcgi_index index.php;

# fastcgi_param SCRIPT_FILENAME /scripts$fastcgi_script_name;

# include fastcgi_params;

#}

# deny access to .htaccess files, if Apache's document root

# concurs with nginx's one

#

#location ~ /.ht {

# deny all;

#}

}

# another virtual host using mix of IP-, name-, and port-based configuration

#

#server {

# listen 8000;

# listen somename:8080;

# server_name somename alias another.alias;

# location / {

# root html;

# index index.html index.htm;

# }

#}

# HTTPS server

#

#server {

# listen 443 ssl;

# server_name localhost;

# ssl_certificate cert.pem;

# ssl_certificate_key cert.key;

# ssl_session_cache shared:SSL:1m;

# ssl_session_timeout 5m;

# ssl_ciphers HIGH:!aNULL:!MD5;

# ssl_prefer_server_ciphers on;

# location / {

# root html;

# index index.html index.htm;

# }

#}

}

**第七步：检查并启动 nginx 服务**

# /usr/local/nginx/sbin/nginx -t #检查配置文件

# /usr/local/nginx/sbin/nginx # 启动服务

# /usr/local/nginx/sbin/nginx -s stop #关闭

# /usr/local/nginx/sbin/nginx -s reload #重启加载配置文件

# ss -anput | grep ":8080" #检查端口

**第八步：被代理主机配置**

![](https://pic4.zhimg.com/v2-bb0f44619fc2ff9fce54b41efe0c97f3_b.jpg)![](https://pic1.zhimg.com/v2-91e61694485d5fd5f427faa552464fb4_b.jpg)

**第九步：被代理主机验证 nginx 正向代理可用性**

# ss -anput | grep ":8080"

tcp LISTEN 0 128 *:8080 *:* users:(("nginx",pid=19515,fd=6),("nginx",pid=19514,fd=6))

tcp ESTAB 0 0 192.168.216.184:8080 192.168.216.185:35718 users:(("nginx",pid=19515,fd=11))

tcp ESTAB 0 0 192.168.216.184:8080 192.168.216.185:35712 users:(("nginx",pid=19515,fd=3))

原作者：黑马程序员

原平台：黑马程序员官网

原链接：[五分钟 9 步搞定 nginx 正向代理配置方法](https://link.zhihu.com/?target=http%3A//yun.itheima.com/jishu/198.html)
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/jkko123/p/12172513.html)

nginx 从 1.9.0 版本开始，新增了 ngx_stream_core_module 模块，使 nginx 支持四层代理和负载均衡。  
默认编译时该模块未编译进去，需要编译时添加 --with-stream，--with-stream_ssl_module，使其支持 stream 代理。  
在之前的版本如果想支持，需要打补丁，安装模块 nginx_tcp_proxy_module。

http 代理，通常就是我们说的七层代理，工作在第七层应用层。  
而 tcp 代理，就是我们常说的四层代理，工作在网络层和传输层。

一、查看 nginx 是否安装 stream 模块

```
2>&1 nginx -V | tr ' ' '\n'|grep stream
```

如果出现下面两项，说明支持

```
--with-stream
--with-stream_ssl_module
```

二、tcp 代理 (代理 mysql 为例)

1、tcp 代理与我们平常说的网站反向代理不一样，它是基于 tcp，udp 协议。  
2、stream 反向代理模块与 http 和 events 是平级的，不要把配置写到 http 里面了。

为了方便添加 stream 配置，我们单独在 nginx/conf 目录创建一个 stream 目录，存放 tcp 代理配置文件。

  
然后在 nginx.conf 中加入如下：

```
stream {
    proxy_connect_timeout 3s;
    include stream/*conf;
}
```

注意，不要加在 http 配置里了。

然后我们在 nginx/conf/stream 下创建一个 mysql.conf 配置文件。

```
upstream mysql {
    server 192.168.10.46:3306;
}
server {
    listen 3306;
    proxy_connect_timeout 3s;
    proxy_timeout 3s;
    proxy_pass mysql;
}
```

然后重载 nginx

```
/usr/local/nginx/sbin/nginx -s reload
```

当我们访问本地的 3306 端口时，会自动代理到 192.168.10.46 主机的 3306 端口上。

三、实现 SSH 转发

在 nginx/conf/stream 下创建一个 ssh.conf 配置文件。

```
upstream ssh {
    server 0.0.0.0:22;
}
 
server {
    listen 22122;
    proxy_pass ssh;
}
```

实现了访问主机的 22122 端口，会自动代理到 22 端口。

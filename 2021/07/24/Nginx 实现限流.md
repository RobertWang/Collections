> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/moris5013/p/12366921.html)

为了防止用户的恶意访问，可以在在 nginx 设置限流，防止服务发生雪崩效应

Nginx 限流分为两种

一是根据 ip 控制速率

二是控制并发连接数

1》 根据 ip 控制速率限流的配置

　　在 http 模块添加配置

　　[![](https://img2018.cnblogs.com/i-beta/1458513/202002/1458513-20200226142126258-1228357322.png)](https://img2018.cnblogs.com/i-beta/1458513/202002/1458513-20200226142126258-1228357322.png)

　　binary_remote_addr 是一种 key，表示基于 remote_addr(客户端 IP) 来做限流，binary_ 的目的是压缩内存占用量。  
　　zone：定义共享内存区来存储访问信息， contentRateLimit:10m 表示一个大小为 10M，名字为 contentRateLimit 的内存区域。  
　　1M 能存储 16000 IP 地址的访问信息，10M 可以存储 16W IP 地址访问信息。  
　　rate 用于设置最大访问速率，rate=10r/s 表示每秒最多处理 10 个请求。  
　　Nginx 实际上以毫秒为粒度来跟踪请求信息，因此 10r/s 实际上是限制：每 100 毫秒处理一个请求。这意味着，自上一个请求处理完后，若后续 100 毫秒内又有请求到达，将拒绝处理该请求.  
　　给某个 location 配置 limit_req

　　[![](https://img2018.cnblogs.com/i-beta/1458513/202002/1458513-20200226142612578-1536046911.png)](https://img2018.cnblogs.com/i-beta/1458513/202002/1458513-20200226142612578-1536046911.png)

       该配置的意思是 ， 当请求路径是 / read_content 时，会根据 contentRateLimit 来限流，每个 ip 访问的速率限制是 2r/s, 能够突发访问的请求数量是 4，不延迟处理请求。

完整配置如下

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
user  root root;
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    #cache
    lua_shared_dict dis_cache 128m;

    #限流设置
    limit_req_zone $binary_remote_addr zone=contentRateLimit:10m rate=2r/s;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        location /update_content {
            content_by_lua_file /root/lua/update_content.lua;
        }

        location /read_content {
            limit_req zone=contentRateLimit burst=4 nodelay;
            content_by_lua_file /root/lua/read_content.lua;
        }
    }
}
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

 2》根据并发连接数来限流

       http 模块添加

　　limit_conn_zone $binary_remote_addr zone=perip:10m;

　　limit_conn_zone $server_name zone=perserver:10m;

       location 添加配置

　　location / {

           limit_conn perip 10;#单个客户端 ip 与服务器的连接数．

           limit_conn perserver 100; ＃限制与服务器的总连接数

          root html; index index.html index.htm;

      }

完整配置如下

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
http {
    include       mime.types;
    default_type  application/octet-stream;

    #cache
    lua_shared_dict dis_cache 128m;

    #限流设置
    limit_req_zone $binary_remote_addr zone=contentRateLimit:10m rate=2r/s;

    limit_conn_zone $binary_remote_addr zone=perip:10m;

    limit_conn_zone $server_name zone=perserver:10m; 

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;
        #所有以brand开始的请求，单个客户端ip与服务端的连接数是10，总共不超过100
        location /brand {
             limit_conn perip 10;#单个客户端ip与服务器的连接数．
             limit_conn perserver 100; ＃限制与服务器的总连接数
             proxy_pass http://192.168.211.1:18081;
        }

        location /update_content {
            content_by_lua_file /root/lua/update_content.lua;
        }

        location /read_content {
            limit_req zone=contentRateLimit burst=4 nodelay;
            content_by_lua_file /root/lua/read_content.lua;
        }
    }
}
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")
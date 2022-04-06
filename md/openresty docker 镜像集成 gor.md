> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/rongfengliang/p/13342065.html)

openresty 是一个很不错的 nginx 增强版本，以下是 openresty 集成 gor 的尝试

问题
--

很多时候我们会基于 nginx（openresty） 进行接口的代理，但是我们需要获取请求信息，同时进行回放

解决
--

gor 是一个很不错的工具，但是我们希望能够进行控制（按需数据捕捉）所以我使用了 supervisord 进行  
管理，而且我们使用的容器部署，使用 supervisord 是一个不错的选择（我没有使用 python 版本的，而是使用  
了 golang 版本的）

参考配置
----

*   Dockerfile

*   supervisor.conf 配置  
    集成 gor 以及 supervisord 的管理还有 openresty 的启动，gor 部分比较简单，自己可以按需扩展

 

```
[program:website]

```

```
command = /usr/local/openresty/bin/openresty

```

```
•

```

```
[inet_http_server]

```

```
port = :9001

```

```
​

```

```
[program:gor]

```

```
command = gor --input-raw :80 --output-file=/app/capture/requests.gor

```

使用
--

> 基于 docker-compose 搞的一个环境

*   docker-compose 文件  
    注意容器需要使用 cap 能力，为了简单我添加了 ALL

*   启动

*   效果

![](https://img2020.cnblogs.com/blog/562987/202007/562987-20200719233603966-1135877990.png)

说明
--

以上是一个简单的集成，实际上我们可以集成 minio 的，这样就可以灵活的进行数据回放了，如果感觉 supervisord 的控制不是很灵活, 我们可以  
基于 jpillora/webproc 进行包装，或者自己编写已给 rest api 方便控制，个人感觉 supervisord 的 admin 控制基本够用了

参考资料
----

[https://github.com/buger/goreplay](https://github.com/buger/goreplay)  
[https://github.com/rongfengliang/openresty_rewrite_static/tree/minio](https://github.com/rongfengliang/openresty_rewrite_static/tree/minio)
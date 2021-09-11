> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1631368493&ver=3308&signature=uhK6W4o6h0YG5tWRdPo9H0MW9BCfCFinC-iEI8PXKwP4vJiaaC12nWUqQcYO26Tvp85iul5m8bcVeWBCzGjbMCoVcAw0wkJRdYBWFpvPeTtL9UqhMYYIHTLc4CwKy-L0&new=1)

> Nginx 的代理功能与负载均衡功能是最常被用到的，先描述一些关于代理功能的配置，再说明负载均衡详细。

Nginx 的代理功能与负载均衡功能是最常被用到的，关于 nginx 的基本语法常识与配置已在上篇文章中有说明，这篇就开门见山，先描述一些关于代理功能的配置，再说明负载均衡详细。  

1、上一篇中我们在 http 模块中有下面的配置，**当代理遇到状态码为 404 时**，我们把 404 页面导向百度。

```
error_page 404 https://www.baidu.com; #错误页
```

然而这个配置，细心的朋友可以发现他并没有起作用。

如果我们想让他起作用，我们必须配合着下面的配置一起使用

```
proxy_intercept_errors on;    #如果被代理服务器返回的状态码为400或者大于400，设置的error_page配置起作用。默认为off。
```

**2、如果我们的代理只允许接受 get，post 请求方法的一种**

```
proxy_method get;    #支持客户端的请求方法。post/get；
```

**3、设置支持的 http 协议版本**  

```
proxy_http_version 1.0 ; #Nginx服务器提供代理服务的http协议版本1.0，1.1，默认设置为1.0版本
```

4、如果你的 nginx 服务器给 2 台 web 服务器做代理，负载均衡算法采用轮询，那么当你的一台机器 web 程序 iis 关闭，也就是说 web 不能访问，那么 nginx 服务器分发请求还是会给这台不能访问的 web 服务器，如果这里的响应连接时间过长，就会导致客户端的页面一直在等待响应，对用户来说体验就打打折扣，这里我们怎么避免这样的情况发生呢。这里我配张图来说明下问题。  

![](https://mmbiz.qpic.cn/mmbiz_jpg/Xruic8OIYw5v3xCBIM5mBU1ibP9hgfoKZCFngdwa3z1xMX0jE8peVp83FbPIwicSmgXiarDMhiciaQ8ia3BHrcWT8Ol0w/640?wx_fmt=jpeg)

如果负载均衡中其中 web2 发生这样的情况，nginx 首先会去 web1 请求，但是 nginx 在配置不当的情况下会继续分发请求道 web2，然后等待 web2 响应，直到我们的响应时间超时，才会把请求重新分发给 web1，这里的响应时间如果过长，用户等待的时间就会越长。

下面的配置是解决方案之一。

```
proxy_connect_timeout 1;   #nginx服务器与被代理的服务器建立连接的超时时间，默认60秒
proxy_read_timeout 1; #nginx服务器想被代理服务器组发出read请求后，等待响应的超时间，默认为60秒。
proxy_send_timeout 1; #nginx服务器想被代理服务器组发出write请求后，等待响应的超时间，默认为60秒。
proxy_ignore_client_abort on;  #客户端断网时，nginx服务器是否终端对被代理服务器的请求。默认为off。
```

5、如果使用 upstream 指令配置啦一组服务器作为被代理服务器，服务器中的访问算法遵循配置的负载均衡规则，同时可以使用该指令配置在发生哪些异常情况时，将请求顺次交由下一组服务器处理。  

```
proxy_next_upstream timeout;  #反向代理upstream中设置的服务器组，出现故障时，被代理服务器返回的状态值。
error|timeout|invalid_header|http_500|http_502|http_503|http_504|http_404|off
```

**error**：建立连接或向被代理的服务器发送请求或读取响应信息时服务器发生错误。

**timeout**：建立连接，想被代理服务器发送请求或读取响应信息时服务器发生超时。

**invalid_header**: 被代理服务器返回的响应头异常。

off: 无法将请求分发给被代理的服务器。

**http_400**，....: 被代理服务器返回的状态码为 400，500，502，等。

6、如果你想通过 http 获取客户的真是 ip 而不是获取代理服务器的 ip 地址，那么要做如下的设置。

```
proxy_set_header Host $host; #只要用户在浏览器中访问的域名绑定了 VIP VIP 下面有RS；则就用$host ；host是访问URL中的域名和端口  www.taobao.com:80
proxy_set_header X-Real-IP $remote_addr;  #把源IP 【$remote_addr,建立HTTP连接header里面的信息】赋值给X-Real-IP;这样在代码中 $X-Real-IP来获取 源IP
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;#在nginx 作为代理服务器时，设置的IP列表，会把经过的机器ip，代理机器ip都记录下来，用 【，】隔开；代码中用 echo $x-forwarded-for |awk -F, '{print $1}' 来作为源IP
```

关于 X-Forwarded-For 与 X-Real-IP 的一些相关文章我推荐一位博友的：HTTP 请求头中的 X-Forwarded-For ，这位博友对 http 协议有一系列的文章阐述，推荐大家去关注下。

7、下面是我的一个关于代理配置的配置文件部分，仅供参考。

```
include       mime.types;   #文件扩展名与文件类型映射表
default_type  application/octet-stream; #默认文件类型，默认为text/plain
#access_log off; #取消服务日志    
log_format myFormat ' $remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for'; #自定义格式
access_log log/access.log myFormat;  #combined为日志格式的默认值
sendfile on;   #允许sendfile方式传输文件，默认为off，可以在http块，server块，location块。
sendfile_max_chunk 100k;  #每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
keepalive_timeout 65;  #连接超时时间，默认为75s，可以在http，server，location块。
proxy_connect_timeout 1;   #nginx服务器与被代理的服务器建立连接的超时时间，默认60秒
proxy_read_timeout 1; #nginx服务器想被代理服务器组发出read请求后，等待响应的超时间，默认为60秒。
proxy_send_timeout 1; #nginx服务器想被代理服务器组发出write请求后，等待响应的超时间，默认为60秒。
proxy_http_version 1.0 ; #Nginx服务器提供代理服务的http协议版本1.0，1.1，默认设置为1.0版本。
#proxy_method get;    #支持客户端的请求方法。post/get；
proxy_ignore_client_abort on;  #客户端断网时，nginx服务器是否终端对被代理服务器的请求。默认为off。
proxy_ignore_headers "Expires" "Set-Cookie";  #Nginx服务器不处理设置的http相应投中的头域，这里空格隔开可以设置多个。
proxy_intercept_errors on;    #如果被代理服务器返回的状态码为400或者大于400，设置的error_page配置起作用。默认为off。
proxy_headers_hash_max_size 1024; #存放http报文头的哈希表容量上限，默认为512个字符。
proxy_headers_hash_bucket_size 128; #nginx服务器申请存放http报文头的哈希表容量大小。默认为64个字符。
proxy_next_upstream timeout;  #反向代理upstream中设置的服务器组，出现故障时，被代理服务器返回的状态值。error|timeout|invalid_header|http_500|http_502|http_503|http_504|http_404|off
#proxy_ssl_session_reuse on; 默认为on，如果我们在错误日志中发现“SSL3_GET_FINSHED:digest check failed”的情况时，可以将该指令设置为off。
```

上一篇中我说啦 nginx 有哪些中负载均衡算法。这一结我就给如果操作配置的给大家做详细说明下。  

首先给大家说下 upstream 这个配置的，这个配置是写一组被代理的服务器地址，然后配置负载均衡的算法。这里的被代理服务器地址有 2 中写法。

```
upstream mysvr { 
      server 192.168.10.121:3333;
      server 192.168.10.122:3333;
    }
 server {
        ....
        location  ~*^.+$ {         
           proxy_pass  http://mysvr;  #请求转向mysvr 定义的服务器列表         
        } 
```

```
upstream mysvr { 
      server  http://192.168.10.121:3333;
      server  http://192.168.10.122:3333;
    }
 server {
        ....
        location  ~*^.+$ {         
           proxy_pass  mysvr;  #请求转向mysvr 定义的服务器列表         
        } 
```

然后，就来点实战的东西。

**1、热备**：如果你有 2 台服务器，当一台服务器发生事故时，才启用第二台服务器给提供服务。服务器处理请求的顺序：AAAAAA 突然 A 挂啦，BBBBBBBBBBBBBB.....

```
upstream mysvr { 
   server 127.0.0.1:7878; 
   server 192.168.10.121:3333 backup;  #热备     
}
```

**2、轮询**：nginx 默认就是轮询其权重都默认为 1，服务器处理请求的顺序：ABABABABAB....

```
upstream mysvr { 
   server 127.0.0.1:7878;
   server 192.168.10.121:3333;       
}
```

**3、加权轮询**：跟据配置的权重的大小而分发给不同服务器不同数量的请求。如果不设置，则默认为 1。下面服务器的请求顺序为：ABBABBABBABBABB....

```
 upstream mysvr { 
      server 127.0.0.1:7878 weight=1;
      server 192.168.10.121:3333 weight=2;
}
```

**4、ip_hash**:nginx 会让相同的客户端 ip 请求相同的服务器。

```
upstream mysvr { 
    server 127.0.0.1:7878; 
    server 192.168.10.121:3333;
    ip_hash;
}
```

5、如果你对上面 4 种均衡算法不是很理解，那么麻烦您去看下我上一篇配的图片，可能会更加容易理解点。

到这里你是不是感觉 nginx 的负载均衡配置特别简单与强大，那么还没完，咱们继续哈，这里扯下蛋。

关于 nginx 负载均衡配置的几个状态参数讲解。

*   **down**，表示当前的 server 暂时不参与负载均衡。
    
*   **backup**，预留的备份机器。当其他所有的非 backup 机器出现故障或者忙的时候，才会请求 backup 机器，因此这台机器的压力最轻。
    
*   **max_fails**，允许请求失败的次数，默认为 1。当超过最大次数时，返回 proxy_next_upstream 模块定义的错误。
    
*   **fail_timeout**，在经历了 max_fails 次失败后，暂停服务的时间。max_fails 可以和 fail_timeout 一起使用。
    

```
 upstream mysvr { 
      server 127.0.0.1:7878 weight=2 max_fails=2 fail_timeout=2;
      server 192.168.10.121:3333 weight=1 max_fails=2 fail_timeout=1;    
}
```

到这里应该可以说 nginx 的内置负载均衡算法已经没有货啦。如果你像跟多更深入的了解 nginx 的负载均衡算法，nginx 官方提供一些插件大家可以了解下。

> 张龙豪
> 
> https://www.cnblogs.com/knowledgesea/p/5199046.html
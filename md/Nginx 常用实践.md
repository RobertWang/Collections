> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/xMNTmru3fePAWzpauFiESQ)

> Nginx 常用实践配置实例

### Nginx 介绍

Nginx 是开源、高性能、高可靠的 Web 和反向代理服务器，而且支持热部署，几乎可以做到 `7 * 24` 小时不间断运行，即使运行几个月也不需要重新启动，还能在不间断服务的情况下对软件版本进行热更新。性能是 Nginx 最重要的考量，其占用内存少、并发能力强、能支持高达 5w 个并发连接数，最重要的是，Nginx 是免费的并可以商业化，配置使用也比较简单。

Nginx 最重要的几个使用场景：

*   • 静态资源服务，通过本地文件系统提供服务
    
*   • 反向代理服务，延伸出包括缓存、负载均衡等
    
*   • API 服务，OpenResty
    

### 相关概念

*   • 简单请求
    
*   • 非简单请求
    
*   • 跨域
    
*   • 正向代理
    
*   • 反向代理
    
*   • 负载均衡
    
*   • 动静分离
    

**简单请求**

简单请求，浏览器请求头信息中会增加`Origin`字段，用来说明本次请求来自的哪个源（协议 + 域名 + 端口）。

*   • 请求方法是 HEAD、GET、POST 三种之一
    
*   • HTTP 头信息不超过右边着几个字段：`Accept`、`Accept-Language`、`Content-Language`、`Last-Event-ID Content-Type` 只限于三个值 `application/x-www-form-urlencoded`、`multipart/form-data`、`text/plain`
    

同时满足上面两个条件，就属于简单请求，反之则属于非简单请求。

**非简单请求**

非简单请求是对服务器有特殊要求的请求，请求方式是`PUT`、`DELETE`，或者`Content-Type`为`application/json`的。

在请求前会发送一次`HTTP`预检`OPTIONS`请求，询问服务器当前请求所在的域名是否在服务器的许可名单之中，只有得到肯定答复，才会发出正式的`XHR`请求，否则报错。

### 常用命令

控制台中输入 `nginx -h` 就可以看到完整的命令，常用的命令：

```
nginx -s reload  # 向主进程发送信号，重新加载配置文件，热重启
nginx -s reopen  # 重启 Nginx
nginx -s stop    # 快速关闭
nginx -s quit    # 等待工作进程处理完成后关闭
nginx -T         # 查看当前 Nginx 最终的配置
nginx -t -c <配置路径>    # 检查配置是否有问题，如果已经在配置目录，则不需要-c

```

### 配置语法

Nginx 的主配置文件是 `/etc/nginx/nginx.conf`，可以使用 `cat -n nginx.conf`来查看配置。

`nginx.conf`结构图：

```
main        # 全局配置，对全局生效
├── events  # 配置影响 Nginx 服务器或与用户的网络连接
├── http    # 配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置
│   ├── upstream # 配置后端服务器具体地址，负载均衡配置不可或缺的部分
│   ├── server   # 配置虚拟主机的相关参数，一个 http 块中可以有多个 server 块
│   ├── server
│   │   ├── location  # server 块可以包含多个 location 块，location 指令用于匹配 uri
│   │   ├── location
│   │   └── ...
│   └── ...
└── ...

```

Nginx 的典型配置：

```
user  nginx;                        # 运行用户，默认即是nginx，可以不进行设置
worker_processes  1;                # Nginx 进程数，一般设置为和 CPU 核数一样
error_log  /var/log/nginx/error.log warn;   # Nginx 的错误日志存放目录
pid        /var/run/nginx.pid;      # Nginx 服务启动时的 pid 存放位置

events {
    use epoll;     # 使用epoll的I/O模型(如果你不知道Nginx该使用哪种轮询方法，会自动选择一个最适合你操作系统的)
    worker_connections 1024;   # 每个进程允许最大并发数
}

http {   # 配置使用最频繁的部分，代理、缓存、日志定义等绝大多数功能和第三方模块的配置都在这里设置
    # 设置日志模式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;   # Nginx访问日志存放位置

    sendfile            on;   # 开启高效传输模式
    tcp_nopush          on;   # 减少网络报文段的数量
    tcp_nodelay         on;
    keepalive_timeout   65;   # 保持连接的时间，也叫超时时间，单位秒
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;      # 文件扩展名与类型映射表
    default_type        application/octet-stream;   # 默认文件类型

    include /etc/nginx/conf.d/*.conf;   # 加载子配置项
    
    server {
        listen       80;       # 配置监听的端口
        server_name  localhost;    # 配置的域名
        
        location / {
            root   /usr/share/nginx/html;  # 网站根目录
            index  index.html index.htm;   # 默认首页文件
            deny 172.168.27.36;   # 禁止访问的ip地址，可以为all
            allow 172.168.27.44； # 允许访问的ip地址，可以为all
        }
        
        error_page 500 502 503 504 /50x.html;  # 默认50x对应的访问页面
        error_page 400 404 error.html;   # 同上
    }
}

```

server 块可以包含多个 location 块，location 指令用于匹配 uri，语法：

```
location [ = | ~ | ~* | ^~] uri {
    ...
}

```

指令：

*   • `=` 精确匹配路径，用于不含正则表达式的 uri 前，如果匹配成功，不再进行后续的查找；
    
*   • `^~` 用于不含正则表达式的 uri； 前，表示如果该符号后面的字符是最佳匹配，采用该规则，不再进行后续的查找；
    
*   • `~` 表示用该符号后面的正则去匹配路径，区分大小写；
    
*   • `~*` 表示用该符号后面的正则去匹配路径，不区分大小写。跟 ~ 优先级都比较低，如有多个 location 的正则能匹配的话，则使用正则表达式最长的那个；
    

> 如果 uri 包含正则表达式，则必须要有 `~` 或 `~*` 标志。

### 全局变量

<table><thead data-style="line-height: 1.75; background: rgba(0, 0, 0, 0.05); font-weight: bold; color: rgb(63, 63, 63);"><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em;">全局变量</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em;">功能</td></tr></thead><tbody><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">$host</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">请求信息中的 Host，如果请求中没有 Host 行，则等于设置的服务器名，不包含端口</td></tr><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">$request_method</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">客户端请求类型，如 GET、POST</td></tr><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">$remote_addr</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">客户端的 IP 地址</td></tr><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">$args</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">请求中的参数</td></tr><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">$arg_PARAMETER</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">GET 请求中变量名 PARAMETER 参数的值，例如：$http_user_agent(Uaer-Agent 值), $http_referer</td></tr><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">$content_length</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">请求头中的 Content-length 字段</td></tr><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">$http_user_agent</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">客户端 agent 信息</td></tr><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">$http_cookie</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">客户端 cookie 信息</td></tr><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">$remote_addr</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">客户端的 IP 地址</td></tr><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">$remote_port</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">客户端的端口</td></tr><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">$http_user_agent</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">客户端 agent 信息</td></tr><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">$server_protocol</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">请求使用的协议，如 HTTP/1.0、HTTP/1.1</td></tr><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">$server_addr</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">服务器地址</td></tr><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">$server_name</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">服务器名称</td></tr><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">$server_port</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">服务器的端口号</td></tr><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">$scheme</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">HTTP 方法（如 http，https）</td></tr></tbody></table>

### 配置反向代理

反向代理是工作中最常用的服务器功能，经常被用来解决跨域问题。

如监听 9001 端口，然后把访问不同路径的请求进行反向代理：

*   • 访问 http://127.0.0.1:9000/text1 的请求转发到 http://127.0.0.1:8080
    
*   • 访问 http://127.0.0.1:9000/text2 的请求转发到 http://127.0.0.1:8081
    

```
vim /etc/nginx/nginx.conf

server {
  listen 9000;
  server_name *.tansci.com;

  location ~ /text1/ {
    proxy_pass http://127.0.0.1:8080;
  }
  
  location ~ /text2/ {
    proxy_pass http://127.0.0.1:8081;
  }
}

```

保存退出，`nginx -s reload` 重新加载

反向代理指令：

*   • `proxy_set_header`：在将客户端请求发送给后端服务器之前，更改来自客户端的请求头信息。
    
*   • `proxy_connect_timeout`：配置 Nginx 与后端代理服务器尝试建立连接的超时时间。
    
*   • `proxy_read_timeout`：配置 Nginx 向后端服务器组发出 read 请求后，等待相应的超时时间。
    
*   • `proxy_send_timeout`：配置 Nginx 向后端服务器组发出 write 请求后，等待相应的超时时间。
    
*   • `proxy_redirect`：用于修改后端服务器返回的响应头中的 Location 和 Refresh。
    

### 跨域 CORS 配置

在浏览器上当前访问的网站向另一个网站发送请求获取数据的过程就是跨域请求。

跨域是浏览器的同源策略决定的，是一个重要的浏览器安全策略，用于限制一个`origin`的文档或者它加载的脚本与另一个源的资源进行交互，它能够帮助阻隔恶意文档，减少可能被攻击的媒介，可以使用`CORS`配置解除这个限制。

例如:

```
# 同源的例子
http://tansci.com/test1/index.html  # 只是路径不同
http://tansci.com/test2/index.html

# 不同源的例子
http://tansci.com/test1   # 协议不同
https://tansci.com/test2

http://tansci.com        # host 不同
http://www.tansci.com
http://app.tansci.com

http://tansci.com        # 端口不同
http://tansci.com:8080

```

**反向代理解决跨域**

前端服务地址为 `a.tansci.com` 的页面请求 `b.tansci.com` 的后端服务导致的跨域，可以这样配置：

```
server {
  listen 9000;
  server_name a.tansci.com;

  location / {
    proxy_pass b.tansci.com;
  }
}

```

这样就将对前一个域名 `a.tansci.com` 的请求全都代理到了 `b.tansci.com`，前端的请求都被我们用服务器代理到了后端地址下，绕过了跨域。

对静态文件的请求和后端服务的请求都以 `fa.tansci.com` 开始，不易区分，所以为了实现对后端服务请求的统一转发，通常我们会约定对后端服务的请求加上 `/api/` 前缀或者其他的 `path` 来和对静态资源的请求加以区分，此时可以这样配置：

```
# 请求跨域，约定代理后端服务请求path以/apis/开头
location ^~/api/ {
    # 这里重写了请求，将正则匹配中的第一个分组的path拼接到真正的请求后面，并用break停止后续匹配
    rewrite ^/api/(.*)$ /$1 break;
    proxy_pass b.tansci.com;
  
    # 两个域名之间cookie的传递与回写
    proxy_cookie_domain b.tansci.com a.tansci.com;
}

```

这样静态资源我们使用 `b.tansci.com/xx.html`，动态资源我们使用 `b.tansci.com/api/getXxx`，浏览器页面看起来仍然访问的前端服务器，绕过了浏览器的同源策略，毕竟我们看起来并没有跨域。

**配置 header 解决跨域**

当浏览器在访问跨源的服务器时，也可以在跨域的服务器上直接设置 Nginx，从而前端就可以无感地开发，不用把实际上访问后端的地址改成前端服务的地址，这样可适性更高。

比如前端站点是 `a.tansci.com`，这个地址下的前端页面请求 `b.tansci.com` 下的资源，比如前者的 `a.tansci.com/index.html` 内容是这样的：

```
<html>
<body>
    <h1>welcome fe.sherlocked93.club!!<h1>
    <script type='text/javascript'>
        var xmlhttp = new XMLHttpRequest()
        xmlhttp.open("GET", "http://b.tansci.com/index.html", true);
        xmlhttp.send();
    </script>
</body>
</html>

```

浏览器访问 `http://a.tansci.com/index.html` 就会报跨域，直接访问 `http://b.tansci.com/index.html` 是可以访问的。

在 /etc/nginx/conf.d/ 文件夹中新建一个配置文件，对应二级域名 `b.tansci.com` ：

```
# /etc/nginx/conf.d/b.tansci.com.conf

server {
    listen       80;
    server_name  b.tansci.com;
  
    add_header 'Access-Control-Allow-Origin' $http_origin;   # 全局变量获得当前请求origin，带cookie的请求不支持*
    add_header 'Access-Control-Allow-Credentials' 'true';    # 为 true 可带上 cookie
    add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';  # 允许请求方法
    add_header 'Access-Control-Allow-Headers' $http_access_control_request_headers;  # 允许请求的 header，可以为 *
    add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
    
    if ($request_method = 'OPTIONS') {
        add_header 'Access-Control-Max-Age' 1728000;   # OPTIONS 请求的有效期，在有效期内不用发出另一条预检请求
        add_header 'Content-Type' 'text/plain; charset=utf-8';
        add_header 'Content-Length' 0;
    
        return 204;                  # 200 也可以
    }
  
    location / {
        root  /usr/share/nginx/html/b;
        index index.html;
    }
}

```

`nginx -s reload` 重新加载配置，再访问 `http://a.tansci.com/index.html` 就正常访问了。

### 配置负载均衡

主要思想就是把负载均匀合理地分发到多个服务器上，实现压力分流的目的。

主要配置如下：

```
http {
  upstream myserver {
      # ip_hash;  # ip_hash 方式
    # fair;   # fair 方式
    server 127.0.0.1:8081;  # 负载均衡目的服务地址
    server 127.0.0.1:8080;
    server 127.0.0.1:8082 weight=10;  # weight 方式，不写默认为 1
  }
 
  server {
    location / {
        proxy_pass http://myserver;
      proxy_connect_timeout 10;
    }
  }
}

```

分配方式:

*   • `轮询` 默认方式，每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务挂了，能自动剔除；
    
*   • `weight` 权重分配，指定轮询几率，权重越高，在被访问的概率越大，用于后端服务器性能不均的情况；
    
*   • `ip_hash` 每个请求按访问 IP 的 hash 结果分配，这样每个访客固定访问一个后端服务器，可以解决动态网页 session 共享问题；
    
*   • `fair（第三方）` 按后端服务器的响应时间分配，响应时间短的优先分配，依赖第三方插件 nginx-upstream-fair，需要先安装；·
    

### 配置动静分离

动静分离就是把动态和静态的请求分开。方式主要有两种：

*   • 一种 是纯粹把静态文件独立成单独的域名，放在独立的服务器上，也是目前主流推崇的方案。
    
*   • 另外一种方法就是动态跟静态文件混合在一起发布， 通过 Nginx 配置来分开。
    

```
server {
  location /www/ {
      root /data/;
    index index.html index.htm;
  }
  
  location /image/ {
      root /data/;
    autoindex on;
  }
}

```

### 配置 HTTPS

先申请好证书，在`/usr/local/nginx/conf/`目录下创建文件夹 `cert`:

```
[root@t2 conf]# mkdir cert

# 查看上传的证书
[root@t2 cert]# ll
-rw-r----- 1 root root 4101 Jul  3 17:13 tansci.com_bundle.crt
-rw-r----- 1 root root 4101 Jul  3 17:13 tansci.com_bundle.pem
-rw-r----- 1 root root 1012 Jul  3 17:13 tansci.com.csr
-rw-r----- 1 root root 1702 Jul  3 17:13 tansci.com.key


```

上传证书文件至 `/usr/local/nginx/conf/cert` 目录下。

修改 `nginx.conf` 文件:

```
# HTTPS server
server {
    listen       443 ssl;
    server_name  www.tansci.com;

    # 证书配置
    ssl_certificate      cert/tansci.com_bundle.crt;
    ssl_certificate_key  cert/tansci.com.key;
    ssl_session_cache    shared:SSL:1m;
    ssl_session_timeout  5m;
    ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers  on;

    location / {
        root   /usr/web;
        index  index.html index.htm;
        try_files $uri $uri/ /index.html;
    }

    # 静态文件代理
    location /static {
        root /usr/web;
        autoindex on;
    }

    # api 接口代理
    location /api/ {
        proxy_pass http://localhost:8080/;
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Credentials' 'true';
        add_header 'Access-Control-Allow-Methods' 'GET, POST';
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Port $server_port;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header REMOTE-HOST $remote_addr;
    }
}

```

### 常用技巧

**静态服务**

```
server {
  listen       80;
  server_name  static.tansci.com;
  charset utf-8;    # 防止中文文件名乱码

  location /download {
    alias   /usr/share/nginx/html/static;  # 静态资源目录
    
    autoindex               on;    # 开启静态资源列目录
    autoindex_exact_size    off;   # on(默认)显示文件的确切大小，单位是byte；off显示文件大概大小，单位KB、MB、GB
    autoindex_localtime     off;   # off(默认)时显示的文件时间为GMT时间；on显示的文件时间为服务器时间
  }
}

```

**图片防盗链**

```
server {
  listen       80;        
  server_name  *.tansci.com;
  
  # 图片防盗链
  location ~* \.(gif|jpg|jpeg|png|bmp|swf)$ {
    valid_referers none blocked server_names ~\.google\. ~\.baidu\. *.qq.com;  # 只允许本机 IP 外链引用，感谢 @木法传 的提醒，将百度和谷歌也加入白名单
    if ($invalid_referer){
      return 403;
    }
  }
}

```

**请求过滤**

```
# 非指定请求全返回 403
if ( $request_method !~ ^(GET|POST|HEAD)$ ) {
  return 403;
}

location / {
  # IP访问限制（只允许IP是 192.168.27.2 机器访问）
  allow 192.168.27.2;
  deny all;
  
  root   html;
  index  index.html index.htm;
}

```

**配置图片、字体等静态文件缓存**

```
# 图片缓存时间设置
location ~ .*\.(css|js|jpg|png|gif|swf|woff|woff2|eot|svg|ttf|otf|mp3|m4a|aac|txt)$ {
    expires 10d;
}

# 如果不希望缓存
expires -1;

```

**HTTP 请求转发到 HTTPS**

```
server {
    listen      80;
    server_name www.tansci.com;

    # 单域名重定向
    if ($host = 'www.tansci.com'){
        return 301 https://www.tansci.com$request_uri;
    }
    # 全局非 https 协议时重定向
    if ($scheme != 'https') {
        return 301 https://$server_name$request_uri;
    }

    # 或者全部重定向
    return 301 https://$server_name$request_uri;

    # 以上配置选择自己需要的即可，不用全部加
}

```

**泛域名路径分离**

这是一个非常实用的技能，经常有时候我们可能需要配置一些二级或者三级域名，希望通过 Nginx 自动指向对应目录，如：

*   • test1.doc.tansci.com 自动指向 /usr/share/nginx/html/doc/test1 服务器地址
    
*   • test2.doc.tansci.com 自动指向 /usr/share/nginx/html/doc/test2 服务器地址
    

```
server {
    listen       80;
    server_name  ~^([\w-]+)\.doc\.tansci\.com$;

    root /usr/share/nginx/html/doc/$1;
}

```
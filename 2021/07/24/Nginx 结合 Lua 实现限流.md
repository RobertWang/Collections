> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1627139152&ver=3210&signature=5E9*3Y4dpqGliiZUTYrNNNfTdRYAhhUO0-B-uGEOuGMKoNoardYtQiXz*polhuUVbV4wb*f0ozh0EPqjPdmYnJMoHP7rCc2tu7ipsLoZQWguGl3FWLTPqLGgBK-sOZgf&new=1)

![](https://mmbiz.qpic.cn/mmbiz_gif/f5u8u3lDeLchGaIcFTOuEzUYJyA2UibBrdxZP9z1Zt8agFR0Yf9TPYqdic361E7GyvVRGVQ8SKMMjsV8WsH0ObLA/640?wx_fmt=gif)  

*   初始化限流策略信息，例如按照渠道或者 ip 地址。每个的处理速度也就是处理的并发量，还有就是桶容量大小（超过每秒处理的量后还看可以缓存多少）
    
*   获取请求参数
    
*   根据参数在策略中查找并获取相应的阀值，若不存在则采用默认值。
    
*   使用 lua-resty-limit-traffic 进行限流。
    

01

两个工具类

*   字符串分割 splitutil.lua
    

```
1/** Created by Erick.
 2**  DateTime: 2019-1-4 16:59
 3**/
 4local _M = {}
 5function _M.split(str,delim)
 6    if type(delim) ~= "string" or string.len(delim) <= 0 then
 7        return
 8    end
 9    local start = 1
10    local tab = {}
11    while true do
12        local pos = string.find(str,delim,start,true)
13        if not pos then
14            break
15        end
16        table.insert(tab,string.sub (str,start,pos-1))
17        start = pos+string.len(delim)
18    end
19    table.insert(tab,string.sub(str,start))
20    return tab
21end
22return _M
```

*   缓存的 get 和 set，sharedoperutil.lua
    

02

初始化策略信息

```
1/** Created by Erick.
 2**  DateTime: 2019-1-4 16:35
 3**  初始化文件限流数据到table中
 4**/
 5local limit_table = ngx.shared.my_limit_store
 6local splitutil = require("splitutil")
 7file = io.open("../lua/syslimitrate.txt","r")
 8if nil == file then
 9    ngx.log(ngx.INFO, "文件读取失败，将采用默认值进行赋值")
10    local suc,err = limit_table:set("default" , "15-15")
11else
12    for line in file:lines() do
13        local splitTable = splitutil.split(line , "-")
14        local sysFlag = splitTable[1]
15        local limitRate = splitTable[2]
16        local bursts = splitTable[3]
17        local tableVal = string.format("%s-%s" , limitRate , bursts)
18        ngx.log(ngx.INFO, "sysFlag = " , sysFlag , "限流阀值：流速-桶容量：" , tableVal)
19        limit_table:set(sysFlag , tableVal)
20    end
21end
22file:close()
```

初始化数据主要是从文件中读取然后放到 nginx 的定义的内存空间 ngx.shared.my_limit_store 中，若是读取文件失败则设置一个默认值。

*   syslimitrate.txt 文件内容
    

```
1/**格式为：渠道-每秒处理并发量-缓存容量**/
2taobao-10-200
3baidu-50-200
```

03

获取请求参数

```
1/** Created by Erick.
 2**  DateTime: 2019-1-8 16:14
 3**/
 4// 获取请求参数
 5local splitutil = require("splitutil")
 6ngx.req.read_body()
 7local args, err = ngx.req.get_post_args()
 8local sysFlag = args["sys"]
 9local limit_table = ngx.shared.my_limit_store
10local limitRate = limit_table:get(sysFlag)
11if not limitRate then
12    ngx.log(ngx.INFO, "sysFlag can not found so set defalut value")
13    limitRate = limit_table:get("default")
14end
15// 获取到的值进行拆分并限流
16local limitValue = splitutil.split(limitRate,"-")
17local rate = tonumber(limitValue[1])
18local burst = tonumber(limitValue[2])
19local limit_req = require "resty.limit.req"
20ngx.say("rate====" , rate , "---burst=====",burst)
21// 根据配置项创建一个限流的table。
22local lim, err = limit_req.new("my_limit_req_store", rate, burst)
23if not lim then
24    ngx.log(ngx.ERR,
25            "failed to instantiate a resty.limit.req object: ", err)
26    return ngx.exit(500)
27end
28// 根据渠道标识进行限流
29local delay, err = lim:incoming(sysFlag, true)
30if not delay then
31    if err == "rejected" then
32        ngx.say("rejected access service..............")
33        ngx.log(ngx.INFO,"rejected access service..............",err)
34        return ngx.exit(503)
35    end
36    ngx.log(ngx.INFO, "failed to limit req: ", err)
37    return ngx.exit(500)
38end
39ngx.log(ngx.INFO,"access received..............")
40ngx.say("access received..............")
```

04

动态调整阀值

动态调整阀值 limitvaluereset_content.lua

```
1/** Created by Erick.
 2**  DateTime: 2019-1-9 14:27
 3**  this lua script can reset limitrate value
 4**/
 5local splitutil = require("splitutil")
 6local sharedoperutil = require("sharedoperutil")
 7// get request param
 8local args, err = ngx.req.get_uri_args()
 9local limitvalue = args["limitvalue"]
10ngx.say("-----------limitvalue-----------" , limitvalue)
11local limitParam = splitutil.split(limitvalue , "-")
12local sysFlag = limitParam[1]
13ngx.say(sharedoperutil.shared_dic_get(ngx.shared.my_limit_store,sysFlag))
14local rateBurst = string.format("%s-%s" , limitParam[2] , limitParam[3])
15ngx.say("-------------rateBurst:" , rateBurst)
16sharedoperutil.shared_dic_set(ngx.shared.my_limit_store , sysFlag , rateBurst , 0)
17ngx.say("---------------" , sharedoperutil.shared_dic_get(ngx.shared.my_limit_store,sysFlag))
```

Nginx 整合

```
1worker_processes  2;
 2pid        logs/nginx.pid;
 4events {
 5    worker_connections  10240;
 6}
 7error_log  logs/access.log info;
 9http {
10    // 定义以及引入限流模块
11    lua_shared_dict my_limit_store 10m;
12    lua_shared_dict my_limit_req_store 100m;
13    lua_package_path "nginx-1.12.1/lua-resty-limit-traffic-0.05/lib/?.lua;nginx-1.12.1/nginx-1.12.1/lua/?.lua;;";
15    include       mime.types;
17    log_format  main  '$remote_addr - $time_local - $request_method - $request_uri - $status - $body_bytes_sent - $request_time';
18    access_log  logs/access.log  main;
20    sendfile       on;
21    tcp_nopush     on;
22    tcp_nodelay    on;
24    keepalive_timeout  10;
25    keepalive_requests 500;
27    client_header_timeout 60;
28    client_body_timeout   60;
29    send_timeout          60;
31    open_file_cache max=1024  inactive=120s;
32    open_file_cache_valid 30;
33    open_file_cache_min_uses 5;
35    gzip on;
36    gzip_http_version 1.0;
37    gzip_comp_level 2;
38    gzip_types text/plain text/css application/x-javascript text/xml application/xml application/xml+rss text/javascript application/javascript application/json;
39    gzip_disable msie6;
41    client_body_in_file_only off;    
42    client_body_temp_path  http_body_dir 1 2;
43    client_body_in_single_buffer on;
44    client_max_body_size 10m;
45    client_body_buffer_size 128k;
46    client_header_buffer_size 4k;
47    large_client_header_buffers 4 16k;
48    // ---------初始化限流信息start-------------
49    init_by_lua_file lua/limitdata_init.lua;
50    // ---------初始化限流信息end-------------
51      // 这部分用到一个自动上线下的模块，在文章最后简单说下
52    upstream myupstream {
53        zone zone_for_myupstream 1m;
54        server 192.168.1.10:8080;
55        server 192.168.1.11:8080;
56    }
57    server {
58        listen       8890;
59        server_name  localhost;
61        location /dynamic {
62            #allow 127.0.0.1;
63            #deny all;
64            dynamic_upstream;
65         }
66        location /limit{
67            #allow 127.0.0.1;
68            #deny all;
69            // --------重置限流阀值start--------
70            content_by_lua_file lua/limitvaluereset_content.lua;
71            //-------重置限流阀值end----------
72        }
73        location / {
74            //--------访问控制进行限流start---------
75            access_by_lua_file lua/limitrate_access.lua;
76            //---------访问控制进行限流end-----------
77            proxy_pass http://myupstream;
78            root   html;
79            index  index.html index.htm;
80        }
81        error_page   500 502 503 504  /50x.html;
82        location = /50x.html {
83            root   html;
84        }
85    }
86}
```

在使用 lua 脚本的地方增加了注释，至此限流的基本功能说完了，大家可以参考进行测试，在讲解的过程中涉及到一个自动上下线的功能用到了一个模块 ngx_dynamic_upstream，调用 api 接口可以实现例如服务器的上线，下线，配置权重，增加 / 删除机器，调整后不用修改比较方便。

【历史文章】

1.[Spring Cloud Config（一）访问加密](http://mp.weixin.qq.com/s?__biz=MzUyMTY4NTkyNg==&mid=2247483760&idx=1&sn=92fa37908fa94ae784106a86a7c0889b&chksm=f9d61fb9cea196af61d6bb8eaa00ace7d6a8d9a6f9eb055b90bfcea64d8e99346b26c7419e68&scene=21#wechat_redirect)  
2.[Spring Cloud Consul 多实例注册问题](http://mp.weixin.qq.com/s?__biz=MzUyMTY4NTkyNg==&mid=2247483768&idx=1&sn=ecb7ce7fcca6388655ba013658bd8f9e&chksm=f9d61fb1cea196a747d6735380b4e4fcc81cd7cd3f4fe9f463eaad5ac23829f778339533db90&scene=21#wechat_redirect)  
3.[Spring Cloud Sleuth 在 WildFly10 容器使用](http://mp.weixin.qq.com/s?__biz=MzUyMTY4NTkyNg==&mid=2247483801&idx=1&sn=12c5aba3a980673a4d1f8e3425227381&chksm=f9d61f50cea19646d4a55f2cdd5e8b2e1b4866f9e2a540a4c7f2364360edafa2add8cd64f8a3&scene=21#wechat_redirect)  
4.[Spring Cloud Gateway 自定义限流](http://mp.weixin.qq.com/s?__biz=MzUyMTY4NTkyNg==&mid=2247483796&idx=1&sn=25b0a8ff4e5bd84ac5a8b31d3fbfb3e9&chksm=f9d61f5dcea1964b8e9e1f3878db4672765835d1eb54f85a798921eaddbe5bcb913a26a3d01b&scene=21#wechat_redirect)
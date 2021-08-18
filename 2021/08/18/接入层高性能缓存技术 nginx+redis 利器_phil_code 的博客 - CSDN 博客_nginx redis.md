> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/phil_code/article/details/79154271)

一. OpenResty  
OpenResty 是一个基于 Nginx 与 Lua 的高性能 Web 平台，其内部集成了大量精良的 Lua 库、第三方模块以及大多数的依赖项。用于方便地搭建能够处理超高并发、扩展性极高的动态 Web 应用、Web 服务和动态网关。

![](https://img-blog.csdn.net/20180125143708939?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcGhpbF9jb2Rl/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  

二.Nginx +redis   
下图左边是常用的架构，http 请求经过 nginx 负载均衡转发到 tomcat，tomcat 再从 redis 读取数据，整个链路过程是串行的，当 tomcat 挂掉或者 tomcat 线程数被消耗完，就无法正常返回数据。  
使用 OpenResty 的 lua-resty-redis 模块使 nginx 具备直接访问 redis 的能力，不占用 tomcat 线程，Tomcat 暂时挂掉仍可正常处理请求，减少响应时长，提高系统并发能力。

![](https://img-blog.csdn.net/20180125143806991?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcGhpbF9jb2Rl/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  

三. 压缩减少带宽  
数据大于 1K，nginx 压缩再保存到 redis:  
1、提高 redis 的读取速度  
2、减少带宽的占用  
压缩会消耗 cpu 时间，小于 1K 的数据不压缩 tps 更高。  
OpenResty 并没有提供 redis 连接池的实现，需要自己用 lua 实现 redis 的连接池，在网上已有实现的例子 http://wiki.jikexueyuan.com/project/openresty/redis/out_package.html，直接参照使用。  
Redis 的 value 值用 json 格式保存 {length:xxx,content:yyy},content 是压缩后的页面内容，length 是 content 压缩前的大小，length 字段是为了在读取 redis 时，根据 length 的大小来判断是否要解压缩 content 的数据。  
使用 lua-zlib 库进行压缩。

![](https://img-blog.csdn.net/20180125145618936?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcGhpbF9jb2Rl/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  

四. 定时更新  
按下图第 1 和第 2 步定时执行，nginx lua 定时器定时请求 tomcat 页面的 url，返回的页面 html 保存在 redis。  
缓存有效期可设置长些，比如 1 个小时，可保证 1 个小时内 tomcat 挂掉，仍可使用缓存数据返回，缓存的定时更新时间可设置短些，比如 1 分钟，保证缓存快速更新

![](https://img-blog.csdn.net/20180125144211326?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcGhpbF9jb2Rl/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  

五. 请求转发  
浏览器打开页面:  
1、nginx 先从 redis 获取页面 html  
2、redis 不存在数据时，从 tomcat 获取页面，同时更新 redis  
3、返回页面 HTML 给浏览器

![](https://img-blog.csdn.net/20180125144419396?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcGhpbF9jb2Rl/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  

六. 单进程定时更新  
Nginx 的所有 worker 进程都可以处理前端请求转发到 redis, 只有 nginx worker 0 才运行定时任务定时更新 redis,lua 脚本中通过 ngx.worker.id() 获取 worker 进程编号。

![](https://img-blog.csdn.net/20180125144510402?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcGhpbF9jb2Rl/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

七

.

可配置化  

通过管理后台配置需要缓存的 URL, 可配置缓存 URL、缓存有效期、定时更新时间, 比如 modify?url=index&&expire=3600000&&intervaltime=300000&sign=xxxx,sign 的值是管理后台 secretkey 对 modify?url=index&&expire=3600000&&intervaltime=300000 签名运算得到的，nginx 端用相同的 secretkey 对 modify?url=index&&expire=3600000&&intervaltime=300000 签名运算，得到的值与 sign 的值相同则鉴权通过, 允许修改 nginx 的配置。

![](https://img-blog.csdn.net/20180125144611588?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcGhpbF9jb2Rl/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jb51.net](https://www.jb51.net/article/169388.htm)

> 本文以示例的形式，由浅入深讲解 Nginx 限流相关配置，是对简略的官方文档的积极补充，感兴趣的朋友跟随小编一起看看吧

本文以示例的形式，由浅入深讲解 Nginx 限流相关配置，是对简略的官方文档的积极补充。

Nginx 限流使用的是 leaky bucket 算法，如对算法感兴趣，可移步维基百科先行阅读。不过不了解此算法，不影响阅读本文。

**空桶**  

我们从最简单的限流配置开始：

```
limit_req_zone $binary_remote_addr zone=ip_limit:10m rate=10r/s;

server {

  location /login/ {

    limit_req zone=ip_limit;

  }

}
```

*   $binary_remote_addr 针对客户端 ip 限流；
*   zone=ip_limit:10m 限流规则名称为 ip_limit，允许使用 10MB 的内存空间来记录 ip 对应的限流状态；
*   rate=10r/s 限流速度为每秒 10 次请求
*   location /login/ 对登录进行限流  
    

限流速度为每秒 10 次请求，如果有 10 次请求同时到达一个空闲的 nginx，他们都能得到执行吗？

![](https://img.jbzj.com/file_images/article/201909/2019090514003136.png)

漏桶漏出请求是匀速的。10r/s 是怎样匀速的呢？每 100ms 漏出一个请求。

在这样的配置下，桶是空的，所有不能实时漏出的请求，都会被拒绝掉。

所以如果 10 次请求同时到达，那么只有一个请求能够得到执行，其它的，都会被拒绝。

这不太友好，大部分业务场景下我们希望这 10 个请求都能得到执行。

**Burst**  

我们把配置改一下，解决上一节的问题

```
limit_req_zone $binary_remote_addr zone=ip_limit:10m rate=10r/s;

server {

  location /login/ {

    limit_req zone=ip_limit burst=12;

  }

}
```

burst=12 漏桶的大小设置为 12  

![](https://img.jbzj.com/file_images/article/201909/2019090514003237.png)

逻辑上叫漏桶，实现起来是 FIFO 队列，把得不到执行的请求暂时缓存起来。

这样漏出的速度仍然是 100ms 一个请求，但并发而来，暂时得不到执行的请求，可以先缓存起来。只有当队列满了的时候，才会拒绝接受新请求。

这样漏桶在限流的同时，也起到了削峰填谷的作用。

在这样的配置下，如果有 10 次请求同时到达，它们会依次执行，每 100ms 执行 1 个。

虽然得到执行了，但因为排队执行，延迟大大增加，在很多场景下仍然是不能接受的。

**NoDelay**  

继续修改配置，解决 Delay 太久导致延迟增加的问题

```
limit_req_zone $binary_remote_addr zone=ip_limit:10m rate=10r/s;

server {

  location /login/ {

    limit_req zone=ip_limit burst=12 nodelay;

  }

}
```

nodelay 把开始执行请求的时间提前，以前是 delay 到从桶里漏出来才执行，现在不 delay 了，只要入桶就开始执行  

![](https://img.jbzj.com/file_images/article/201909/2019090514003438.png)

要么立刻执行，要么被拒绝，请求不会因为限流而增加延迟了。

因为请求从桶里漏出来还是匀速的，桶的空间又是固定的，最终平均下来，还是每秒执行了 5 次请求，限流的目的还是达到了。

但这样也有缺点，限流是限了，但是限得不那么匀速。以上面的配置举例，如果有 12 个请求同时到达，那么这 12 个请求都能够立刻执行，然后后面的请求只能匀速进桶，100ms 执行 1 个。如果有一段时间没有请求，桶空了，那么又可能出现并发的 12 个请求一起执行。

大部分情况下，这种限流不匀速，不算是大问题。不过 nginx 也提供了一个参数才控制并发执行也就是 nodelay 的请求的数量。

```
limit_req_zone $binary_remote_addr zone=ip_limit:10m rate=10r/s;

server {

  location /login/ {

    limit_req zone=ip_limit burst=12 delay=4;

    proxy_pass http://login_upstream;

  }

}
```

delay=4 从桶内第 5 个请求开始 delay  

![](https://img.jbzj.com/file_images/article/201909/2019090514003539.png)

这样通过控制 delay 参数的值，可以调整允许并发执行的请求的数量，使得请求变的均匀起来，在有些耗资源的服务上控制这个数量，还是有必要的。

Reference  

[http://nginx.org/en/docs/http/ngx_http_limit_req_module.html](http://nginx.org/en/docs/http/ngx_http_limit_req_module.html)  
[https://www.nginx.com/blog/rate-limiting-nginx/](https://www.nginx.com/blog/rate-limiting-nginx/)

**总结**

以上所述是小编给大家介绍的 Nginx 限流配置, 希望对大家有所帮助，如果大家有任何疑问请给我留言，小编会及时回复大家的。在此也非常感谢大家对脚本之家网站的支持！  
如果你觉得本文对你有帮助，欢迎转载，烦请注明出处，谢谢！
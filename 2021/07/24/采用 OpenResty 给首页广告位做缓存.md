> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/moris5013/p/12363378.html)

OpenResty 是国人章亦春发起的一个开源项目，可以理解成封装了 nginx, 并且集成了 LUA 脚本, 足以胜任 10K 乃至 1000K 以上单机并发连接。

首页的访问频率一般都比较高，对于首页上的广告位，广告变更频率比较低，可以利用缓存提高访问速度

广告位缓存架构图

[![](https://img2018.cnblogs.com/i-beta/1458513/202002/1458513-20200225194216382-702477280.png)](https://img2018.cnblogs.com/i-beta/1458513/202002/1458513-20200225194216382-702477280.png)

openresty 安装

1》添加仓库

　　yum install yum-utils

　　yum-config-manager --add-repo https://openresty.org/package/centos/openresty.repo

2》执行安装

　　yum install openresty

3》安装成功后会在默认的目录下

　　/usr/local/openresty  

 安装成功后默认启动了 nginx 进程，访问域名就可以看到默认的首页了。

openresty 的启动，停止，重启

cd    /usr/local/openresty/nginx  

停止  ./sbin/nginx  -s stop

启动  ./sbin/nginx  -c conf/nginx.conf

重启  ./sbin/nginx  -s reload

设置缓存

1)  修改 / usr/local/openresty/nginx/conf/nginx.conf, 将配置文件使用的根设置为 root, 目的就是将来要使用 lua 脚本的时候 ，直接可以加载在 root 下的 lua 脚本。

[![](https://img2018.cnblogs.com/i-beta/1458513/202002/1458513-20200226131607813-2047071465.png)](https://img2018.cnblogs.com/i-beta/1458513/202002/1458513-20200226131607813-2047071465.png)

 2) 定义 lua 缓存命名空间大小

[![](https://img2018.cnblogs.com/i-beta/1458513/202002/1458513-20200226131800113-284700153.png)](https://img2018.cnblogs.com/i-beta/1458513/202002/1458513-20200226131800113-284700153.png)

 3）新建 lua 脚本文件  /root/lua/read_content.lua  ， 内容如下

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
ngx.header.content_type="application/json;charset=utf8"
local uri_args = ngx.req.get_uri_args();
local id = uri_args["id"];
local cache_ngx = ngx.shared.dis_cache;
local contentCache = cache_ngx:get('content_cache_'..id);

if contentCache == "" or contentCache == nil then
    local redis = require("resty.redis");
    local red = redis:new()
    red:set_timeout(2000)
    red:connect("192.168.200.128", 6379)
    local rescontent=red:get("content_"..id);

    if ngx.null == rescontent then
        local cjson = require("cjson");
        local mysql = require("resty.mysql");
        local db = mysql:new();
        db:set_timeout(2000)
        local props = {
            host = "192.168.200.128",
            port = 3306,
            database = "changgou_content",
            user = "root",
            password = "123456"
        }
        local res = db:connect(props);
        local select_sql = "select url,pic from tb_content where status ='1' and category_id="..id.." order by sort_order";
        res = db:query(select_sql);
        local responsejson = cjson.encode(res);
        red:set("content_"..id,responsejson);
        red:expire("content_"..id,3*60);
        ngx.say(responsejson);
        db:close()
    else
        cache_ngx:set('content_cache_'..id, rescontent, 2*60);
        ngx.say(rescontent)
    end
    red:close()
else
    ngx.say(contentCache)
end
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

4）在 / usr/local/openresty/nginx/conf/nginx.conf 中配置如下

```
location /read_content {
     content_by_lua_file /root/lua/read_content.lua;
}
```

5）前端访问地址为 [http://192.168.200.128/read_content?id=1](http://192.168.200.128/read_content?id=1)

 这样就实现了首页广告位的 lua 缓存 + redis 缓存 ， lua 的过期时间为 2 分钟，redis 的为 3 分钟。当修改数据库记录后，什么都不用更改，缓存失效后会自动同步为 mysql 中的数据。
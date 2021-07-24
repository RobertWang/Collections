> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/huahao1989/article/details/105911154)

【转载请注明出处】：[https://blog.csdn.net/huahao1989/article/details/105911154](https://blog.csdn.net/huahao1989/article/details/105911154)

Lua 代码：

```  
local locks = require "resty.lock"

local function acquire()  
    local lock =locks:new("locks")  
    local elapsed, err =lock:lock("limit_key") -- 互斥锁  
    local limit_counter =ngx.shared.limit_counter -- 计数器

    local key = "ip:" ..os.time()  
    local limit = 5 -- 限流大小  
    local current =limit_counter:get(key)

    if current ~= nil and current + 1> limit then -- 如果超出限流大小  
       lock:unlock()  
       return 0  
    end  
    if current == nil then  
       limit_counter:set(key, 1, 1) -- 第一次需要设置过期时间，设置 key 的值为 1，过期时间为 1 秒  
    else  
        limit_counter:incr(key, 1) -- 第二次开始加 1 即可  
    end  
    lock:unlock()  
    return 1  
end  
ngx.print(acquire())  
```

Nginx  
```  
http {

    ……  
    lua_shared_dict locks 10m;  
    lua_shared_dict limit_counter 10m;

}  
```
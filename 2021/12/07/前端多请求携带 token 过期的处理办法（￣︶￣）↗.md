> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6844904186794999815)

例：`login?callback=xxxxx`

多个请求，所以会触发好几次的响应拦截，并且触发的时机会有先后之分，甚至有可能进入登录页面之后，有的响应才刚从后台返回，这并不是我们所希望看到的，因为会引发很多未知的问题。 ![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/6/12/172a6753e91481f1~tplv-t2oaga2asx-watermark.awebp) 我现在遇到的情况就是，如果第一个响应返回 token 失效，那么我这边会做一个拦截的操作，获取当前失效页面的 URL 地址`order?id=1`这种地址，跳转到`login`页面

```
//我这边把 order?id=1 base64位加密
let path = "/order?id=1"  //vue的路由
path = Base64.encode(path)
let callback=`login?callback=${path}`
复制代码
```

那么，跳转到登录页面以后，会有一个响应才从后端返回，那么这个拦截函数又把地址处理了一遍

```
let path = "/login?callback=xxx" //当前获取到的地址变成了登录页
path = Base64.encode(path)
let callback=`login?callback=${path}`
复制代码
```

这种问题就出来了💥

解决方法🔑
------

### 第一种方法

在拦截函数上进行一个修改，我用的是`axios`

```
axios.interceptors.response.use(response => {
 //对于token过期的处理
 if (response.data.code == 102) {
        let path = location.hash.split("/")[1]
        if (!(/callback/g).test(path)) {
            let code = Base64.encode(path)
            router.replace("/login?callback=" + code)
        }
    }
})
复制代码
```

对于地址携带`callback`字段的 URL 地址，一律拦截，虽然粗糙但能解决问题😂

### 第二种方法

针对第一种方法的改进，使用`callback`字段终归觉得有点问题，[查阅 axios 的文档](https://link.juejin.cn?target=http%3A%2F%2Fwww.axios-js.com%2Fzh-cn%2Fdocs%2F "http://www.axios-js.com/zh-cn/docs/")，有一个阻止请求的配置`cancelToken`，那么就用它对这个拦截函数进行一个改造

```
const cancelToken = axios.CancelToken;  //阻止请求
const pendding = []  //把当前请求的状态都存到一个数组里
axios.interceptors.request.use(config => {
    config.cancelToken = new cancelToken((c) => {
        pendding.push({ fun: c, url: config.url })
    })
})
request.interceptors.response.use(response => {
    //对于token过期的处理
    if(response.data.code===200){
        //对于请求ok的数据
        for (let i in pendding) {
            if(response.config.url===pendding[i].url){
                pendding.splice(i, 1); //把这条记录从数组中移除
            }
        }
        return response.data
    }
    if(response.data.code===102){
        for (let i in pendding) {
            pendding[i].fun(); //执行取消操作
            pendding.splice(i, 1); //把这条记录从数组中移除
        }
        let path = location.hash.split("/")[1]
        let code = Base64.encode(path)
        router.replace("/login?callback=" + code)
    }
})
复制代码
```

目前整个逻辑

*   请求拦截方面，对于所有正在请求的 api 做一个数组存储
*   响应拦截方面，针对请求 200 的数据，从请求数组中剔除掉，对于 token 过期的请求，进行一个对已经发起请求还没返回响应的取消请求操作，并从请求数组中剔除掉。

小结😊
----

这篇文章针对自己所遇到的问题和当前的解决方案的总结。我相信这并不是最好的解决方法，奈何能力有限只能想到这两种方式。

现在目前还有一个可以优化修改的点，就是请求的同时返回 (间隔时间很短)，这就导致拦截函数还是会触发不止一次，其实如果第一次返回`token`失效之后，后面的请求就没必要再发了。针对这个问题，自己所想的就是先发一个预检请求，请求返回`token`正常，再发其他的请求，目前所想到的最笨的方法😥

```
created: async (){
    let result = await verifyToken()
    if(result.code===200){
        //其余的请求
    }
}
复制代码
```

多多指教，互相学习

参考资料📒
------

*   [《axios 中文文档》](https://link.juejin.cn?target=http%3A%2F%2Faxios-js.com%2Fzh-cn%2Fdocs%2Findex.html "http://axios-js.com/zh-cn/docs/index.html")

本文使用 [mdnice](https://link.juejin.cn?target=https%3A%2F%2Fmdnice.com "https://mdnice.com") 排版
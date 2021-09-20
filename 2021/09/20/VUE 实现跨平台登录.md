> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7008901692154445860) 要想弄清楚这个问题，我们就要思考当我们打开一个链接的时候，我们自身的框架都为我们做了什么处理？ 熟悉 vue 的人都知道路由前置守卫，没错进入页面时会触发 router.beforeEach 的钩子，你可以把他理解成通往成功跳转的必经之路

那在这个钩子里框架做了什么事呢？

a. 首先页面会判断有没有登录过帐号，判断有没有登录过帐号的方式一般都是看看本地有没有存储 token。这个 token 我们可以理解为一个登录过后得到的身份证，有了这个身份证你就可以畅通无阻，在任何界面自由穿梭。

b. 有 token 则成功跳转，没 token 则跳转到登录页面。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/76b3a620902f4261af11ab15daac9d00~tplv-k3u1fbpfcp-watermark.awebp?)

了解这些我们就知道其实只要能有一个 token，就能实现我们的目的。ok 接下来我们就想想怎么去获得这个 token？

### 2. 如何获取 token？

获取 token 的方式最直接的方式其实就是后台直接把 token 当作参数放到链接中，用户点击链接，前端直接获取浏览器的参数 token 存放到本地。但是这样做有一个弊端：这个 token 是有过期时间的，一旦过期还是会跳转到登录页面。所以最佳实践还是得每次跳转都手动去获取一个最新的 token。

如何获取最新 token？要么调登录接口，传入用户名和密码获取最新 token，要么就是直接传一个用户名和密码生成的加密字符串，传给后台解析再返回一个最新 token，这两种方式都可以达到目的。

ok 接下来就直接上代码，最开始我写的如下图所示：

```
router.beforeEach((to, from, next) => {
    //是否有自动登录，通过跳转链接是否携带token参数来判断
    if(!!(to.query.token)){
        // 调用接口获取token，并存储到本地
    }
    // 检测本地是否存储token
    if (localStorage.getItem('token')) {
        // 调用接口获取用户信息等
    }
}
复制代码
```

但是上面的代码会有一个问题就是并非同步执行，因为调用接口获取 token 是异步操作。变成同步也很简单，这里就不过多赘述。

```
export const init=(callback)=>{
    // 调用接口获取token
    // 调用接口获取用户信息等
    callback()
}
复制代码
```

```
import init from ".."
router.beforeEach((to, from, next) => {
    // 检测是否有自动登录或者本地是否存储token
    if (!!(to.query.token)||localStorage.getItem('token')) {
       init(callback) 
    }
}
复制代码
```

### 3. 优化

3.1 到这其实基本就已经实现功能了，但是有强迫症和完美主义的朋友一定会发现，你跳转过来的链接会多一些参数。看着很别扭而且搞不好还存在一定的风险。如何去除呢？很简单

```
// 去掉地址栏的token参数
if (Reflect.has(to.query, 'token')) {
    Reflect.deleteProperty(to.query, 'token');
}
// 使用replace是因为不想返回的时候跳转到登录页面，因为直接next路由记录里会留存登录路径
next({ ...to, replace: true });
复制代码
```

3.2 如果我是说如果，你的页面头部会有返回按钮的，比如一些详情页面头部往往会有返回按钮。这个时候由于是直接跳转过来的，路由记录里只存在你当前页面的一条记录，所以你的返回按钮点击是无效的。这个你可以仔细看浏览器的返回按钮，浏览器返回按钮是禁用的就说明没法回退。这个也有两种解决方法：要么也禁用返回按钮，要么就是判断如果无法回退就跳转到某一个指定链接

```
if (window.history.length > 1) {
    this.$router.go(-1); // 返回上一个界面
} else {
    this.$router.push({
        path: 'xx',
    });
}
复制代码
```
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6844903781990137864)*   目的是在原生 fetch 基础之上增加拦截请求的功能。
*   封装后，最终暴露在外面的接口应该于原生 fetch 的使用方法一致，同时增加拦截器功能
*   拦截器的 API 设计参考 axios 的拦截器（个人比较喜欢 axios 的拦截器设计）。

### 拦截器 API 使用方法

你可以在 then 和 catch 之前拦截请求和响应。

```
// 添加一个请求拦截器
c_fetch.interceptors.request.use(function (config) {
    // Do something before request is sent
    return config;
  });

// 添加一个响应拦截器
c_fetch.interceptors.response.use(function (response) {
    // Do something with response data
    return response;
  });
复制代码
```

### 拦截器实现方案

1.  为了保证封装后的 fetch 暴露在最外面的 api 跟原生 fetch 保持一致。所以，我们要暴露到最外面的 api 应该是个跟 fetch 接收相同参数的函数。  
    我们看下原生 fetch 方法语法解释和接收参数解释。  
    

```
语法: 
Promise<Response> fetch(input[, init]); 
复制代码
```

```
参数:
?input
定义要获取的资源。这可能是：
一个 USVString 字符串，包含要获取资源的 URL。一些浏览器会接受 blob: 和 data: 作为 schemes.
一个 Request 对象。

init 可选
一个配置项对象，包括所有对请求的设置。可选的参数有：
method: 请求使用的方法，如 GET、POST。
headers: 请求的头信息，形式为 Headers 的对象或包含 ByteString 值的对象字面量。
body: 请求的 body 信息：可能是一个 Blob、BufferSource、FormData、URLSearchParams 或者 USVString 对象。注意 GET 或 HEAD 方法的请求不能包含 body 信息。
mode: 请求的模式，如 cors、 no-cors 或者 same-origin。
credentials: 请求的 credentials，如 omit、same-origin 或者 include。为了在当前域名内自动发送 cookie ， 必须提供这个选项， 从 Chrome 50 开始， 这个属性也可以接受 FederatedCredential 实例或是一个 PasswordCredential 实例。
cache:  请求的 cache 模式: default 、 no-store 、 reload 、 no-cache 、 force-cache 或者 only-if-cached 。
redirect: 可用的 redirect 模式: follow (自动重定向), error (如果产生重定向将自动终止并且抛出一个错误), 或者 manual (手动处理重定向). 在Chrome中，Chrome 47之前的默认值是 follow，从 Chrome 47开始是 manual。
referrer: 一个 USVString 可以是 no-referrer、client或一个 URL。默认是 client。
referrerPolicy: Specifies the value of the referer HTTP header. May be one of no-referrer、 no-referrer-when-downgrade、 origin、 origin-when-cross-origin、 unsafe-url 。
integrity: 包括请求的  subresource integrity 值 （ 例如： sha256-BpfBw7ivV8q2jLiT13fxDYAe2tJllusRSZ273h2nFSE=）。
复制代码
```

> 原生 fetch 的详细文档请移步 MDN 查阅：  
> [developer.mozilla.org/zh-CN/docs/…](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FFetch_API%2FUsing_Fetch "https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API/Using_Fetch")

从上面的 fetch 的语法解释和参数解释，我们可以知道 fetch 是个函数；接收两个参数，第一个参数定义要获取的资源，第二个参数为可选项，一个配置项对象，包括所有对请求的设置；fetch 函数分返回结果是一个 promise 对象。

2.  我们先来实现封装后，暴露出来的 c_fetch 函数。

```
function c_fetch (input, init = {}) {
    //fetch默认请求方式设为GET
    if(!init.method){
      init.method = 'GET'
    }
    
    //interceptors_req是拦截请求的拦截处理函数集合
    //后面会讲解interceptors_req的定义与实现
    interceptors_req.forEach(interceptors => {
      init = interceptors(init);
    })
    
    //在原生fetch外面封装一个promise，为了在promise里面可以对fetch请求的结果做拦截处理。
    //同时，保证c_fetch函数返回的结果是个promise对象。
    return new Promise(function (resolve, reject) {
      //发起fetch请求，fetch请求的形参是接收上层函数的形参
      fetch(input, init).then(res => {
        //interceptors_res是拦截响应结果的拦截处理函数集合
        //后面会讲解interceptors_res的定义与实现
        interceptors_res.forEach(interceptors => {
          //拦截器对响应结果做处理，把处理后的结果返回给响应结果。
          res = interceptors(res);
        })
        //将拦截器处理后的响应结果resolve出去
        resolve(res)
      }).catch(err => {
        reject(err);
      })
    })

  }
复制代码
```

3.  c_fetch 函数实现完了，我们就差拦截器的实现了。我们在 c_fetch 函数的基础上增加 interceptors，用来注册拦截器。

```
//定义用来存储拦截请求和拦截响应结果的处理函数集合
  let interceptors_req = [], interceptors_res = [];

  function c_fetch (input, init = {}) {
    //fetch默认请求方式设为GET
    if(!init.method){
      init.method = 'GET'
    }
    
    //interceptors_req是拦截请求的拦截处理函数集合
    interceptors_req.forEach(interceptors => {
      init = interceptors(init);
    })
    
    //在原生fetch外面封装一个promise，为了在promise里面可以对fetch请求的结果做拦截处理。
    //同时，保证c_fetch函数返回的结果是个promise对象。
    return new Promise(function (resolve, reject) {
      //发起fetch请求，fetch请求的形参是接收上层函数的形参
      fetch(input, init).then(res => {
        //interceptors_res是拦截响应结果的拦截处理函数集合
        interceptors_res.forEach(interceptors => {
          //拦截器对响应结果做处理，把处理后的结果返回给响应结果。
          res = interceptors(res);
        })
        //将拦截器处理后的响应结果resolve出去
        resolve(res)
      }).catch(err => {
        reject(err);
      })
    })

  }
  
  //在c_fetch函数上面增加拦截器interceptors，拦截器提供request和response两种拦截器功能。
  //可以通过request和response的use方法来绑定两种拦截器的处理函数。
  //use方法接收一个参数，参数为一个callback函数，callback函数用来作为拦截器的处理函数；
  //request.use方法会把callback放在interceptors_req中，等待执行。
  //response.use方法会把callback放在interceptors_res中，等待执行。
  //拦截器的处理函数callback接收一个参数。
  //request拦截器的callback接收的是请求发起前的config；
  //response拦截器的callback接收的是网络请求的response结果。
  c_fetch.interceptors = {
    request: {
      use: function (callback) {
        interceptors_req.push(callback);
      }
    },
    response: {
      use: function (callback) {
        interceptors_res.push(callback);
      }
    }
  }
复制代码
```

4.  最后将整个封装之后，含有拦截器功能的 fetch 包装为一个插件暴露给开发者使用。

```
/**
 * c_fetch
 * 基于原生fetch封装了拦截器功能，暴露出来的c_fetch跟原生fetch用法一致，只是增加了拦截器功能。拦截器用法参考axios的拦截器用法。
 * 拦截器: c_fetch.interceptors
 * 注意: 拦截器不拦截reject类型的response结果
 */

  //定义用来存储拦截请求和拦截响应结果的处理函数集合
  let interceptors_req = [], interceptors_res = [];

  function c_fetch (input, init = {}) {
    //fetch默认请求方式设为GET
    if(!init.method){
      init.method = 'GET'
    }
    
    //interceptors_req是拦截请求的拦截处理函数集合
    interceptors_req.forEach(interceptors => {
      init = interceptors(init);
    })
    
    //在原生fetch外面封装一个promise，为了在promise里面可以对fetch请求的结果做拦截处理。
    //同时，保证c_fetch函数返回的结果是个promise对象。
    return new Promise(function (resolve, reject) {
      //发起fetch请求，fetch请求的形参是接收上层函数的形参
      fetch(input, init).then(res => {
        //interceptors_res是拦截响应结果的拦截处理函数集合
        interceptors_res.forEach(interceptors => {
          //拦截器对响应结果做处理，把处理后的结果返回给响应结果。
          res = interceptors(res);
        })
        //将拦截器处理后的响应结果resolve出去
        resolve(res)
      }).catch(err => {
        reject(err);
      })
    })

  }
  
  //在c_fetch函数上面增加拦截器interceptors，拦截器提供request和response两种拦截器功能。
  //可以通过request和response的use方法来绑定两种拦截器的处理函数。
  //use方法接收一个参数，参数为一个callback函数，callback函数用来作为拦截器的处理函数；
  //request.use方法会把callback放在interceptors_req中，等待执行。
  //response.use方法会把callback放在interceptors_res中，等待执行。
  //拦截器的处理函数callback接收一个参数。
  //request拦截器的callback接收的是请求发起前的config；
  //response拦截器的callback接收的是网络请求的response结果。
  c_fetch.interceptors = {
    request: {
      use: function (callback) {
        interceptors_req.push(callback);
      }
    },
    response: {
      use: function (callback) {
        interceptors_res.push(callback);
      }
    }
  }

  export default c_fetch;
复制代码
```

### 总结

本篇文章只是实现了一个最基本的拦截器功能，文章字数有限，没有深入讲解更加成熟的拦截器实现方式。有兴趣的朋友可以阅读下 axios 的拦截器实现，很有意思的。axios 的拦截器会更加完善。

我这里针对 fetch 的封装也只是最基本的封装，目的是讲解拦截器的实现，没有过于复杂化。上文中暴露出来的 c_fetch 其实可以在封装一层 cc_fetch, 用 bind 方法把 c_fetch 的方法绑定在 cc_fetch 上，最后暴露出来的是 cc_fetch。这样做的好处是保护了 c_fetch 的方法不会被外部所影响，篡改等。当然了，这只是我个人的一些看法，不代表所有人。

最后谢谢各位能够坚持阅读到最后，希望您阅读本篇文章能够有所收获。谢谢🙏～
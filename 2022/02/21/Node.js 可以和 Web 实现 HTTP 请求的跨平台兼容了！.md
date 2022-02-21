> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651598632&idx=2&sn=2cf791edc2f66dfa4dc84499e46cea74&chksm=8022f8e9b75571ffdac7953d6b5361cda3fb5455b0360371e3e8b2aced62e093c545dadb201f&mpshare=1&scene=1&srcid=0220yGzha5XTUFkwM5W64QDy&sharer_sharetime=1645335619202&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd) ![](https://mmbiz.qpic.cn/mmbiz_png/e5Dzv8p9XdSx2iblDKs1Jw18rhjY8SCNr1M10yd0rWniaUCj1sqrkWTLucpYoT0JAmBbYotiajZ9IciaZT8RJUfNLg/640?wx_fmt=png)

```
const https = require('https')
const options = {
  hostname: 'nodejs.cn',
  port: 443,
  path: '/todos',
  method: 'GET'
}

const req = https.request(options, res => {
  console.log(`状态码: ${res.statusCode}`)

  res.on('data', d => {
    process.stdout.write(d)
  })
})

req.on('error', error => {
  console.error(error)
})

req.end()
```

![](https://mmbiz.qpic.cn/mmbiz_png/e5Dzv8p9XdSx2iblDKs1Jw18rhjY8SCNrkAESEvE3AtRd4Df3JwOV0YZksgJNw35y3duVDNVgR7fdJGss7gE5ZQ/640?wx_fmt=png)

`Fetch API` 可能大家都比较熟悉了，他是当前最流行的跨平台 `HTTP Client API` ，目前已经可以在浏览器和 `Web/Service Workers` 中运行，当前 Web 环境里用到最多的请求方式应该就是它了。

`Node.js` 中的`Fetch API` 基于 `Undici` 实现，它提供了一个 `WHATWG` 标准接口来获取资源，并且也是基于 `Promise` 的，使用方式基本和浏览器中一致，包括四个核心模块：

*   `fetch()` - 用于发起请求的函数
    
*   `Headers` 类 - 用于处理请求头和响应头
    
*   `Request` 类 - 表示传入请求的实例
    
*   `Response` 类 - 表示传入响应的实例
    

```
const res = await fetch('https://www.conardli.top');
const json = await res.json();
console.log(json);
```

其实这并不是简单的支持了一个新的原生 `HTTP` 请求库那么简单，这意味着很多之前在 `Web` 中用到 `Fetch` 的 `NPM` 包也可以在 `Node.js` 里以同样的方式工作了，这些包同样可以实现跨平台兼容了～

在 `Node.js v17.5` 中，它还是个实验特性，现在想要试用的话可以通过 `node --experimental-fetch flag` 开启。
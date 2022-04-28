> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7021444877778944037)*   我们要做的拦截是什么意思呢？ 就是我们可以`拦截`页面上所有的`fetch请求`，我们可以在请求之前`拦截请求头的信息`，和在请求后`拦截得到服务器响应的结果`。
*   我们拦截到的内容，可以经过我们处理，想干嘛就干嘛。
*   拦截器用法参考`axios`的拦截器用法
*   最后我们也封装一下统一使用，在真实项目上的操作的封装
*   你需要掌握的知识点
    *   fetch 的使用 [Fetch API 教程](https://link.juejin.cn?target=https%3A%2F%2Fwww.ruanyifeng.com%2Fblog%2F2020%2F12%2Ffetch-tutorial.html "https://www.ruanyifeng.com/blog/2020/12/fetch-tutorial.html")
    *   promise 的用法 [promise 教程](https://link.juejin.cn?target=https%3A%2F%2Fes6.ruanyifeng.com%2F%23docs%2Fpromise "https://es6.ruanyifeng.com/#docs/promise")

### 实现原理

*   通过重写`fetch`方法，添加我们想要的功能，然后替换`window`下的`fetch`

例如： 下面这行代码就是可以替换原生`fetch`方法了

```
window.fetch = Cfetch
复制代码
```

### fetch 的拦截器实现

完整代码如下，代码写注释比较详细，我就不做过多解释了

```
/**
 * Cfetch
 * 基于原生fetch封装了拦截器功能，暴露出来的Cfetch跟原生fetch用法一致，只是增加了拦截器功能。拦截器用法参考axios的拦截器用法。
 * 拦截器: interceptors
 */

// 定义用来存储拦截请求和拦截响应结果的处理和错误结果处理的函数集合
let interceptorsReq = []
let interceptorsReqError = []
let interceptorsRes = []
let interceptorsResError = []

// 复制一份原生fetch的方法，后面我们还是得调用原生的fetch，只是我们在fetch之上做一层封装，添加我们想要的功能
const OriginFetch = window.fetch

function Cfetch(input, init = {}) {
  // interceptorsReq是拦截请求的拦截处理函数集合
  interceptorsReq.forEach(item => {
    init = item(init)
  })

  // 在原生fetch外面封装一个promise，为了在promise里面可以对fetch请求的结果做拦截处理。
  // 同时，保证Cfetch函数返回的结果是个promise对象。
  return new Promise((resolve, reject) => {
    // 发起fetch请求，fetch请求的形参是接收上层函数的形参
    OriginFetch(input, init).then(res => {
      // interceptorsRes是拦截响应结果的拦截处理函数集合
      interceptorsRes.forEach(item => {
        // 拦截器对响应结果做处理，把处理后的结果返回给响应结果。
        res = item(res)
      })
      // 将拦截器处理后的响应结果resolve出去
      resolve(res)
    })
      .catch(err => {
        // interceptorsResError是拦截响应错误结果的拦截处理函数集合
        interceptorsResError.forEach(item => {
        // 拦截器对响应错误结果做处理，把处理后的结果返回给响应结果。
          err = item(err)
        })
        reject(err)
      })
  })
}

// interceptors拦截器提供request和response两种拦截器功能。
// 可以通过request和response的use方法来绑定两种拦截器的处理函数。
// use方法接收两个参数，参数为一个callback函数，callback函数用来作为拦截器的成功处理函数，errorCallback作为错误处理函数
// request.use方法会把callback放在interceptorsReq中，等待执行。
// response.use方法会把callback放在interceptorsRes中，等待执行。
// 拦截器的处理函数callback接收一个参数。
// request拦截器的callback接收的是请求发起前的config；
// response拦截器的callback接收的是网络请求的response结果。
const interceptors = {
  request: {
    use(callback, errorCallback) {
      interceptorsReq.push(callback)
      errorCallback && interceptorsReqError.push(errorCallback)
    }
  },
  response: {
    use(callback, errorCallback) {
      interceptorsRes.push(callback)
      errorCallback && interceptorsResError.push(errorCallback)
    }
  }
}

// 暴露导出这个对象
export default {
  Cfetch,
  interceptors
}
复制代码
```

这里解释下为什么我们的`interceptors`对象，不写在 Cfetch 函数上面呢，如`Cfetch.interceptors = {}`, 因为这样写呢，有些网页他们做了反注入的话，会检查原生的 fetch 上面是不是多了其他属性，

所以我就不写在`Cfetch`函数上面了，这样他们就检查不出来。道高一丈，魔高一尺，哈哈哈，如果对方又做了其他反注入，大家可以发挥想法换一种思路注入

### fetch 封装使用

```
import { Cfetch, interceptors } from './fetch'
// 这里是我项目使用到的js-cookie库，主要是为了拿到token，你们这里改成你们获取token的方式即可
import Cookies from 'js-cookie'

/**
 * config 自定义配置项
 * @param withoutCheck 不使用默认的接口状态校验，直接返回 response
 * @param returnOrigin 是否返回整个 response 对象，为 false 只返回 response.data
 * @param showError 全局错误时，是否使用统一的报错方式
 * @param canEmpty 传输参数是否可以为空
 * @param mock 是否使用 mock 服务
 * @param timeout 接口请求超时时间，默认10秒
 */
let configDefault = {
  showError: true,
  canEmpty: false,
  returnOrigin: false,
  withoutCheck: false,
  mock: false,
  timeout: 10000
}

// 添加请求拦截器
interceptors.request.use(config => {
  // 这里是我项目使用到的js-cookie库，主要是为了拿到token，你们这里改成你们获取token的方式即可
  const token = Cookies.get('access_token')
  let configTemp = Object.assign({
    responseType: 'json',
    headers: {
      'Content-Type': 'application/json;charset=utf-8',
      authorization: `Bearer ${token}`
    },
  }, configDefault, config)
  console.log('添加请求拦截器 configTemp ==>', configTemp)
  return configTemp
})

// 添加响应拦截器
interceptors.response.use(async response => {
  console.log('拦截器response ==>', response)
  console.log('configDefault', configDefault)
  
  // TODO: 这里是复制一份结果处理，在这里可以做一些操作
  const res = await resultReduction(response.clone())

 // HTTP 状态码 2xx 状态入口，data.code 为 200 表示数据正确，无任何错误
  if (response.status >= 200 && response.status < 300) {
    return response
  } else { // 非 2xx 状态入口
    if (configDefault.withoutCheck) { // 不进行状态状态检测
      return Promise.reject(response)
    }
    return Promise.reject(response)
  }
})

// 结果处理，fetch请求响应结果是promise，还得处理
async function resultReduction(response) {
  let res = ''
  switch (configDefault.responseType) {
    case 'json':
      res = await response.json()
      break
    case 'text':
      res = await response.text()
      break
    case 'blod':
      res = await response.blod()
      break
    default:
      res = await response.json()
      break
  }
  console.log('结果处理', res)
  return res
}

function request(method, path, data, config) {
  let myInit = {
    method,
    ...configDefault,
    ...config,
    body: JSON.stringify(data),
  }
  if (method === 'GET') {
    let params = ''
    if (data) {
    // 对象转url参数
      params = JSON.stringify(data).replace(/:/g, '=')
        .replace(/"/g, '')
        .replace(/,/g, '&')
        .match(/\{([^)]*)\}/)[1]
    }
    return Cfetch(`${path}?${params}`, {
        ...configDefault,
        ...config,
    })
  }

  return Cfetch(path, myInit)
}

// get请求方法使用封装
function get(path, data, config) {
  return request('GET', path, data, config)
}

// post请求方法使用封装
function post(path, data, config) {
  return request('POST', path, data, config)
}

// put请求方法使用封装
function put(path, data, config) {
  return request('PUT', path, data, config)
}

// delete请求方法使用封装
function del(path, data, config) {
  return request('DELETE', path, data, config)
}

export default {
  fetch: Cfetch,
  get,
  post,
  put,
  delete: del
}

复制代码
```

### 注入页面，拦截页面的 fetch 请求

注入 js 代码，修改 fetch 方法，达到拦截器的作用，原理很简单， 就是我们通过重写 fetch 的方法，给 fetch 添加拦截器的功能，然后替换原生 windoe 下的 fetch 即可

```
window.fetch = Cfetch
复制代码
```

### 最后调用

```
import fetchApi from './fetchApi'

winodw.fetch = fetchApi.fetch

fetchApi.get('http://www.xxx', {id: '1'})

fetchApi.post('http://www.xxx', {id: '1'})
复制代码
```

成功结果打印如图：

![](https://347830076.github.io/myBlog/assets/img/js/fetch.png)

复制代码，开始你们的调试对接吧。有什么问题，欢迎留言交流

参考文章：

[基于原生 fetch 封装一个带有拦截器功能的 fetch，类似 axios 的拦截器](https://juejin.cn/post/6844903781990137864#heading-2 "https://juejin.cn/post/6844903781990137864#heading-2")

[Fetch API 教程](https://link.juejin.cn?target=https%3A%2F%2Fwww.ruanyifeng.com%2Fblog%2F2020%2F12%2Ffetch-tutorial.html "https://www.ruanyifeng.com/blog/2020/12/fetch-tutorial.html")
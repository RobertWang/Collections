> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/359007292)

准备工作
----

*   h5 引入 [JSSDK](https://link.zhihu.com/?target=https%3A//res.wx.qq.com/open/js/jweixin-1.3.2.js)
*   微信小程序后台配置 webview 引入的所有页面域名 / 业务域名

*   不配置业务域名 (本地调试)：本地调试可以在 `微信开发者工具 -> 详情 -> 本地设置` 中，勾选☑️`不校验合法域名、web-viewweb-view(业务域名)、TLS版本以及 HTTPS 证书`
*   `https` 协议的域名 (本地调试)：在本地调试可以采用 `内网穿透` 的方式实现

思路
--

### 微信小程序打开 h5 的方式

*   法一：直接嵌入 h5 页面，页面的跳转不做处理，沿用 h5 端的跳转，在一个 webview 打开所有页面

*   不足：没有返回标记，只能依赖于手机的物理返回键 / 右滑返回

*   法二（采用☑️）：每个打开的 h5 都是一个新的 webview 页面

*   不足：需要在跳转的地方判断终端环境；小程序原始页面栈的数量限制。

### 微信小程序与 h5 的通信：

*   （采用☑️）微信小程序 `webview` 向 h5 页面传递参数，通过 url 问号后所带的参数传递 `url?token=token` ；
*   h5 向微信小程序传递参数，h5 中使用 `wx.miniProgram.postMessage` ，向小程序发送消息，会在特定时机（小程序后退、组件销毁、分享）触发组件的 `bindmessage=onMessage` 事件；

### 小程序端向 h5 传递数据 (参数最好做 `encodeURIComponent()` 和 `decodeURIComponent()` 转码处理)

*   小程序端 (`/pages/webH5/webH5` 嵌入 h5 通用页面)：将所要传递的参数拼接到 `webview` 打开的 `url` 后面
*   h5 端 (需要被嵌入的页面)：通过对 `url` 的截取去获取参数

*   法 1: `location.search` 获取 '?token=token'
*   法 2： `vue` 可以通过 `this.$route.query` 获取

优化
--

### 需要优化的点

*   如果打开 h5 页面前，就已经拿到了所要传递的数据，那么这个过程没有什么问题。但现实情况涉及的数据可能是需要通过后端接口 / 其他页面业务去获取，如果是如下的过程：

> 在 `onLoad` 的时候或 `data` 中直接赋值 `url` ，然后在获取到数据的页面返回当前页面，本地页 `onShow` 的时候将参数拼接到 `url` ，重新 `this.setData({url})`

```
那么就会产生两个问题，先打开该页面，然后获取到数据，重新赋值 `url` ，页面重新加载(问题一)，产生历史记录(问题二)。

```

### 优化方案

*   问题一：页面重新加载，那就考虑如何在给 `url` 加上参数改变后，既能获取到参数，页面又不会刷新。 hash 变化，不会使页面刷新，锚连接、 vue-router 的 hash 单页面同理。
*   问题二：产生历史记录，那同样的页面，加载了两次，第二次获取到参数后，只要将数据保存下来返回就好了。 h5 页面通过监听 `onhashchange` ，当 hash 变化后，处理参数，将参数保存到 localStorage 中后，`history.go(-1)` 历史记录后退，回到没有 `hash` / 没有参数的页面。

实例代码
----

*   utils 工具 `/utils/utils`

```
// 与 原生定义 ua，检测终端
export const terminal = {
  ua: navigator.userAgent,
  isIos: () => {
    let ios = 'iOSAppMyAppName'
    return !!(~terminal.ua.indexOf(ios))
  },
  isMiniProgram: () => {
    let miniProgram = 'miniProgram'
    return !!(~terminal.ua.indexOf(miniProgram))
  }
}


```

*   hash 工具 `/utils/hash.js`

```
// 简单的 hash 注册
class Hash {
  constructor(callback) {
    this.init = false
    if (this.init) return
    callback && callback()
    this.init = true    // hashchange 只注册一次
    this.num = 0        //  callback 只执行一次
    window.addEventListener('hashchange', (e) => {
      if (this.num == 1) return
      callback && callback()
      history.go(-1)
      this.num += 1
    })
  }

  static toObj(hash) {
    let a = hash.split('?'),
      s = a.length > 1 ? a[1] : ''
    a = s.split('&')
    let o = {}
    a.forEach((v, i) => {
      let b = v.split("=")
      b[0] && (o[b[0]] = b[1])
    })
    return o
  }

  static toString(hashObj) {
    let s = '#/?'
    for (let k in hashObj) {
      s += `${k}=${hashObj[k]}&`
    }
    return s
  }
}

export default Hash


```

*   h5 页面 (vue)

```
<template>
  <div>
    <van-pull-refresh
     
      :loading-text="search.loadingRefreshLoadingText"
      v-model="search.loadingRefresh"
      @refresh="onRefresh"
    >

      <!-- 列表 -->
      <van-list
        v-model="search.loading"
        :error.sync="search.error"
        :finished="search.finished"
        :error-text="search.errorText"
        :finished-text="search.finishedText"
        :immediate-check="false"
        @load="onLoad"
        ref="searchList"
      >
        <div ref="moreModel">
          <div>
            <template v-for="(item, index) in search.list">
              <template v-for="(item, i) in item">
                <div :key="index+i" @click="handleRouterDetail(item)">
                  {{item}}
                </div>
              </template>
            </template>
          </div>
        </div>
      </van-list>
    </van-pull-refresh>
  </div>
</template>

<script>
import { terminal } from '@/utils/utils'
import Hash from '@/utils/hash'

const PAGE_NUMBER = 1,
  PAGE_SIZE = 10

export default {
  name: 'MyApp',
  data() {
    return {
      search: {
        pageNumber: PAGE_NUMBER,
        pageSize: PAGE_SIZE,
        loadingRefresh: false,
        loading: false,
        loadingRefreshLoadingText: '正在刷新',
        finished: false,
        finishedText: '暂无更多数据哦～',
        error: false,
        errorText: '请求失败，点击重新加载',
        list: [[],[],[]]
      },
      imgHost: process.env.IMG_ROOT,
      
    }
  },
  created() {
    let hash = new Hash(() => {
      let isMiniProgram = terminal.isMiniProgram()
      if (isMiniProgram) {
        let {
          token
        } = Hash.toObj(this.$route.hash)
        localStorage.setItem('token', token)
      }
    })
    this.init()
  },
  mounted() {},
  methods: {
    // 路由
    handleRouter(route) {
      if (route == null || route == '') return
      let isMiniProgram = terminal.isMiniProgram()
      if (typeof route == 'string' && /(http(s)?):\/\//.test(route)) {
        if (isMiniProgram) {
          return wx.miniProgram.navigateTo({
            url: `/pages/webH5/webH5?webUrl=${encodeURIComponent(route)}`
          })
        }
        return (window.location.href = route)
      }
      if (isMiniProgram)
        return wx.miniProgram.navigateTo({
          url: `/pages/webH5/webH5?webUrl=${encodeURIComponent(
            `${location.origin}${route}`
          )}`
        })
      this.$router.push(route)
    },
    
    // 详情页
    handleRouterDetail(item) {
      let { id } = item,
        route = {
          path: `/detail/${id}`
        }
      let isMiniProgram = terminal.isMiniProgram()
      if (isMiniProgram)
        return wx.miniProgram.navigateTo({
          url: `/pages/webH5/webH5?webUrl=${encodeURIComponent(
            `${location.origin}${route.path}`
          )}`
        })
      this.$router.push(route)
    },
    
    // 列表下拉刷新
    async onRefresh() {
      this.search.pageNumber = PAGE_NUMBER
      this.search.list = []
      this.search.loadingRefresh = true
      this.search.loading = false
      this.search.finished = false
      await this.getList()
    },
    // 列表上拉加载更多
    onLoad() {
      let { finished, pageNumber } = this.search
      if (finished || pageNumber == 1) return
      this.search.loadingRefresh = false
      this.search.loading = true
      this.getList()
    },
    // 初始化
    init() {
      this.onRefresh()
    },
    // 获取列表
    async getList() {
      let { pageNumber, pageSize, filter } = this.search,
        data = {
          pageNumber,
          pageSize,
          ...filter
        }
      try {
        let res = await getList(data)
        this.search.loadingRefresh = false
        this.search.loading = false
        if (res.code != 0) return (this.search.error = true)
        if (!res.data || !res.data.list) return (this.search.finished = true)
        this.search.list[pageNumber] = res.data.list
        if (res.data.list.length < pageSize)
          return (this.search.finished = true)
        this.search.pageNumber += 1
      } catch (e) {
        this.search.error = true
        this.search.loadingRefresh = false
        this.search.loading = false
      }
    },
  }
}
</script>

<style lang="scss" scoped>
$color: #333;

// 文本超出隐藏
@mixin ellipsis($line: 1) {
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: $line;
  overflow: hidden;
  word-break: break-all;
}

img {
  display: block;
  width: 100%;
}

.page {
  padding-bottom: 49px;
  width: 100%;
  min-height: 100vh;
  background: #f4f5f6;
  font-size: 12px;
  color: $color;
}

</style>

```

*   小程序 `/pages/webH5/webH5`

```
function queryToObj(s) {
  let o = {}
  if (!s) return o
  let a = s.split('&')
  a.forEach((v, i) => {
    let b = v.split("=")
    b[0] && (o[b[0]] = b[1])
  })
  return o
}

function queryToString(obj) {
  let s = ''
      for (let k in obj) {
        s += `${k}=${obj[k]}&`
      }
      s = s && s[s.length - 1] == '&' && s.slice(0, -1)
      return s
}

module.exports = {
  queryToObj,
  queryToString
}
<web-view src="{{url}}" bindmessage="onWebViewMessage" bindload="onWebViewLoad" binderror="onWebViewError"></web-view>
import {
  queryToObj,
  queryToString
} from '../../utils/util'

import {version} from '../../api/domain.config'

Page({

  /**
   * 页面的初始数据
   */
  data: {
    url: ''
  },

  /**
   * 生命周期函数--监听页面加载
   */
  onLoad: function (options) {
    console.log(options)
    let {
      webUrl
    } = options
    webUrl = decodeURIComponent(webUrl)
    this.setData({
      url: webUrl
    })
  },

  /**
   * 生命周期函数--监听页面初次渲染完成
   */
  onReady: function () {

  },

  /**
   * 生命周期函数--监听页面显示
   */
  onShow: function () {
    let token = wx.getStorageSync('token'),
      
      hash = this.data.url.split('?'),
      queryObj = hash.length > 1 ? queryToObj(hash[hash.length - 1]) : ''
    queryObj = {
      ...queryObj,
      token,
      version
    }
    let queryStr = queryToString(queryObj)
    this.setData({
      url: `${url}#/?${queryStr}`
    })
  },

  /**
   * 生命周期函数--监听页面隐藏
   */
  onHide: function () {

  },

  /**
   * 生命周期函数--监听页面卸载
   */
  onUnload: function () {

  },

  /**
   * 页面相关事件处理函数--监听用户下拉动作
   */
  onPullDownRefresh: function () {

  },

  /**
   * 页面上拉触底事件的处理函数
   */
  onReachBottom: function () {

  },

  /**
   * 用户点击右上角分享
   */
  onShareAppMessage: function (e) {
    let {
      from,
      target,
      webViewUrl
    } = e,
    bridge = '/pages/bridge/bridge',
      pages = getCurrentPages(),
      currentPage = pages[pages.length - 1],
      webUrl = `/pages/webH5/webH5?webUrl=${encodeURIComponent(webViewUrl.split('#/')[0])}`,
      queryObj = {
        webUrl: encodeURIComponent(webUrl),
      },
      queryStr = queryToString(queryObj),
      path = `${bridge}?${(queryStr)}`,
      shareInfo = wx.getStorageSync('shareInfo')
    console.log(path, shareInfo)
    if (shareInfo) {
      let {
        shareImg: imageUrl,
        shareTitle: title
      } = shareInfo
      return {
        title,
        path,
        imageUrl
      }
    }
    return {
      // title: '',
      path,
      // imageUrl: ''
    }
  },
  onWebViewMessage(e) {
    console.log('webH5 onWebViewMessage ', e)
  },
  onWebViewLoad(e) {
    console.log('webH5 onWebViewLoad', e)
  },
  onWebViewError(e) {
    console.log('webH5 onWebViewError', e)
  }
})


```

*   分享 `/pages/bridge/bridge`

注意⚠️：分享页面 `wxml` 留白不要有任何文本，因为作为中专页面，会有一瞬间的闪烁。

```
Page({

  /**
   * 页面的初始数据
   */
  data: {
    deep: 0,
    webUrl: ''
  },

  /**
   * 生命周期函数--监听页面加载
   */
  onLoad: function (options) {
    console.log(options)
    let {
      webUrl
    } = options
    this.setData({
      webUrl: decodeURIComponent(webUrl)
    })
  },

  /**
   * 生命周期函数--监听页面初次渲染完成
   */
  onReady: function () {

  },

  /**
   * 生命周期函数--监听页面显示
   */
  onShow: function () {
    // 分享卡片进入页面，判断是路由分享目标页，还是返回首页
    // 判断规则，首次进入，路由分享目标页；二次进入返回首页
    try {
      let {
        deep
      } = this.data
      if (deep == 0) {
        deep += 1
        this.setData({
          deep
        })
        let {
          webUrl: url
        } = this.data
        wx.navigateTo({
          url,
        })
      }
      if (deep == 1) {
        wx.switchTab({
          url: '/pages/index/index',
        })
      }
    } catch (error) {
      wx.switchTab({
        url: '/pages/index/index',
      })
    }
  },

})


```
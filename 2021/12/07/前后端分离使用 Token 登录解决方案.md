> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6844903664163749901)*   首次登录时，后端服务器判断用户账号密码正确之后，根据用户 id、用户名、定义好的秘钥、过期时间生成 token ，返回给前端；
*   前端拿到后端返回的 token , 存储在 localStorage 和 Vuex 里；
*   前端每次路由跳转，判断 localStorage 有无 token ，没有则跳转到登录页，有则请求获取用户信息，改变登录状态；
*   每次请求接口，在 Axios 请求头里携带 token;
*   后端接口判断请求头有无 token，没有或者 token 过期，返回 401；
*   前端得到 401 状态码，重定向到登录页面。

我这里前端使用 Vue ，地址：[vue-token-login](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fliruifengv%2Fvue-token-login "https://github.com/liruifengv/vue-token-login")

后端使用阿里的 egg，地址：[egg-token-login](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fliruifengv%2Fegg-token-login "https://github.com/liruifengv/egg-token-login")

首先，我们先轻微封装一下 Axios:
-------------------

我把 Token 存在 localStorage，检查有无 Token ，每次请求在 Axios 请求头上进行携带

```
if (window.localStorage.getItem('token')) {
  Axios.defaults.headers.common['Authorization'] = `Bearer ` + window.localStorage.getItem('token')
}
复制代码
```

使用 respone 拦截器，对 2xx 状态码以外的结果进行拦截。

如果状态码是 401，则有可能是 Token 过期，跳转到登录页。

```
instance.interceptors.response.use(
  response => {
    return response
  },
  error => {
    if (error.response) {
      switch (error.response.status) {
        case 401:
          router.replace({
            path: 'login',
            query: { redirect: router.currentRoute.fullPath } // 将跳转的路由path作为参数，登录成功后跳转到该路由
          })
      }
    }
    return Promise.reject(error.response)
  }
)
复制代码
```

定义路由：
-----

```
const router = new Router({
  mode: 'history',
  routes: [
    {
      path: '/',
      name: 'Index',
      component: Index,
      meta: {
        requiresAuth: true
      }
    },
    {
      path: '/login',
      name: 'Login',
      component: Login
    }
  ]
})
复制代码
```

上面我给首页路由加了 requiresAuth，所以使用路由钩子来拦截导航，localStorage 里有 Token ，就调用获取 userInfo 的方法，并继续执行，如果没有 Token ，调用退出登录的方法，重定向到登录页。

```
router.beforeEach((to, from, next) => {
  let token = window.localStorage.getItem('token')
  if (to.meta.requiresAuth) {
    if (token) {
      store.dispatch('getUser')
      next()
    } else {
      store.dispatch('logOut')
      next({
        path: '/login',
        query: { redirect: to.fullPath }
      })
    }
  } else {
    next()
  }
})
复制代码
```

这里使用了两个 Vuex 的 action 方法，马上就会说到 。

Vuex
----

首先，在 mutation_types 里定义：

```
export const LOGIN = 'LOGIN' // 登录
export const USERINFO = 'USERINFO' // 用户信息
export const LOGINSTATUS = 'LOGINSTATUS' // 登录状态
复制代码
```

然后在 mutation 里使用它们：

```
const mutations = {
  [types.LOGIN]: (state, value) => {
    state.token = value
  },
  [types.USERINFO]: (state, info) => {
    state.userInfo = info
  },
  [types.LOGINSTATUS]: (state, bool) => {
    state.loginStatus = bool
  }
}
复制代码
```

在之前封装 Axios 的 JS 里定义请求接口：

```
export const login = ({ loginUser, loginPassword }) => {
  return instance.post('/login', {
    username: loginUser,
    password: loginPassword
  })
}

export const getUserInfo = () => {
  return instance.get('/profile')
}
复制代码
```

在 Vuex 的 actions 里引入：

```
import * as types from './types'
import { instance, login, getUserInfo } from '../api'
复制代码
```

定义 action

```
export default {
  toLogin ({ commit }, info) {
    return new Promise((resolve, reject) => {
      login(info).then(res => {
        if (res.status === 200) {
          commit(types.LOGIN, res.data.token) // 存储 token
          commit(types.LOGINSTATUS, true)     // 改变登录状态为 
          instance.defaults.headers.common['Authorization'] = `Bearer ` + res.data.token // 请求头添加 token
          window.localStorage.setItem('token', res.data.token)  // 存储进 localStorage
          resolve(res)
        }
      }).catch((error) => {
        console.log(error)
        reject(error)
      })
    })
  },
  getUser ({ commit }) {
    return new Promise((resolve, reject) => {
      getUserInfo().then(res => {
        if (res.status === 200) {
          commit(types.USERINFO, res.data) // 把 userInfo 存进 Vuex
        }
      }).catch((error) => {
        reject(error)
      })
    })
  },
  logOut ({ commit }) { // 退出登录
    return new Promise((resolve, reject) => {
      commit(types.USERINFO, null)        // 情况 userInfo
      commit(types.LOGINSTATUS, false)  // 登录状态改为 false
      commit(types.LOGIN, '')          // 清除 token
      window.localStorage.removeItem('token')
    })
  }
}
复制代码
```

接口
--

这时候，我们该去写后端接口了。

我这里用了阿里的 egg 框架，感觉很强大。

首先定义一个 **LoginController** ：

```
const Controller = require('egg').Controller;
const jwt = require('jsonwebtoken'); // 引入 jsonwebtoken

class LoginController extends Controller {
  async index() {
    const ctx = this.ctx;
/*
 把用户信息加密成 token ,因为没连接数据库，所以都是假数据
正常应该先判断用户名及密码是否正确
*/
    const token = jwt.sign({       
      user_id: 1,      // user_id
      user_name: ctx.request.body.username // user_name
    }, 'shenzhouhaotian', { // 秘钥
        expiresIn: '60s' // 过期时间
    });
    ctx.body = {             // 返回给前端
      token: token
    };
    ctx.status = 200;           // 状态码 200
  }
}

module.exports = LoginController;
复制代码
```

**UserController：**

```
class UserController extends Controller {
  async index() {
    const ctx = this.ctx
    const authorization = ctx.get('Authorization');
    if (authorization === '') { // 判断请求头有没有携带 token ,没有直接返回 401
        ctx.throw(401, 'no token detected in http header "Authorization"');
    }
    const token = authorization.split(' ')[1];
    // console.log(token)
    let tokenContent;
    try {
        tokenContent = await jwt.verify(token, 'shenzhouhaotian');     //如果 token 过期或验证失败，将返回401
        console.log(tokenContent)
        ctx.body = tokenContent     // token有效，返回 userInfo ;同理，其它接口在这里处理对应逻辑并返回
    } catch (err) {
        ctx.throw(401, 'invalid token');
    }
  }
}
复制代码
```

在 router.js 里定义接口：

```
module.exports = app => {
  const { router, controller } = app;
  router.get('/', controller.home.index);
  router.get('/profile', controller.user.index);
  router.post('/login', controller.login.index);
};
复制代码
```

前端请求
----

接口写好了，该前端去请求了。

这里我写了个登录组件，下面是点击登录时的 login 方法：

```
login () {
      if (this.username === '') {
        this.$message.warning('用户名不能为空哦~~')
      } else if (this.password === '') {
        this.$message.warning('密码不能为空哦~~')
      } else {
        this.$store.dispatch('toLogin', {      // dispatch toLogin action
          loginUser: this.username,
          loginPassword: this.password
        }).then(() => {
          this.$store.dispatch('getUser')      // dispatch getUserInfo action
          let redirectUrl = decodeURIComponent(this.$route.query.redirect || '/')
          console.log(redirectUrl)
          // 跳转到指定的路由
          this.$router.push({
            path: redirectUrl
          })
        }).catch((error) => {
          console.log(error.response.data.message)
        })
      }
    }
复制代码
```

登录成功后，跳转到首页之前重定向过来的页面。

整体流程跑完了，实现的主要功能就是：

1.  访问登录注册之外的路由，都需要登录权限，比如首页，判断有无 token，有则访问成功，没有则跳转到登录页面；
2.  成功登录之后，跳转到之前重定向过来的页面；
3.  token 过期后，请求接口时，身份过期，跳转到登录页，继续第二步；这一步主要用了可以做 7 天自动登录等功能。

### 写的很糙，大家见谅，抱拳了！
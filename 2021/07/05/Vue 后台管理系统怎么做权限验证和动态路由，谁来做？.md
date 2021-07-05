> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6981031288803164173)

> 好久都没有写过文章了，主要是因为还是觉得自己才疏学浅，如果把自己学到的东西以那种类似教程的形式写出来的话，怕误认子弟。然后自己也还没有工作，也没有什么好的工作经验可以分享。

吹一下水
----

我好久都没有写过文章了，主要是因为还是觉得自己才疏学浅，如果把自己学到的东西以那种类似教程的形式写出来的话，怕误认子弟。然后自己也还没有工作，也没有什么好的工作经验可以分享。最后一个原因就是，实在是没时间，学校乱七八糟的事情一大堆，期末就忙的跟狗一样。也没怎么学新东西，然后现在感觉我已经 out 了，群里大佬们聊的东西已经听不懂了（其实也没那么夸张），哈哈哈，这可能就是前端人的痛吧。  
然后说说我为什么想起要写这篇文章吧，是因为最近打算做自己的一个商城项目，我是先做后台的，主要的前端技术栈就是 Vue，后端我是用的 PHP+Laravel。我做后台的时候第一个面临的问题就是权限的问题，单说权限可能太笼统了，到底是什么权限呢。其实我这个项目里面涉及到的权限包括两个，一个是**操作权限**，一个是**路由访问权限**（这里指前端路由）。我接下来就讲一下我的思路和一步一步的探索（简单粗暴的说就是踩坑历程）

思路准备
----

首先思考权限谁来做？前端还是后端，分情况来讨论。我觉得操作权限必须是后端来做的，操作权限就是对每一个操作进行权限限制，比如超级管理员可以添加后台用户、禁用后台用户，而其他的后台用户无法进行这些操作像这么细粒度的控制最好还是由后端来做。就是对每一个接口进行权限访问控制，如果由前端来做的话可能得累死，而且不安全。第二个就是路由的访问权限，即便是我们做了细粒度的操作权限限制，可我们还是希望不同的用户登录进后台的时候能够访问的路由是不同的，拥有的菜单也是不同的。因为这样其实是提高了用户体验的，不同角色的用户拥有的菜单不同，就相当于提前防止用户误操作（因为并不清楚哪些操作有权限），好过等到用户执行某一个具体的操作，然后被告知无权限好吧。其实这也不是完全可以做到的，比如两个用户都可以访问同一个页面，但是他们在这个页面里的操作权限却是不同的，总结一下就是，后端的验证是为了保证数据完整和操作安全的，而前端做验证，不管是什么验证都是为了提高可用性和用户体验的。

路由的访问权限也是我主要想说的，因为操作权限，大部分都是后端做的，前端顶多是根据接口响应给用户对应的提示。我的后台是基于开源项目 [vue-element-admin](https://github.com/PanJiaChen/vue-element-admin) 开发的，这个项目的路由访问控制是放在前端做的，按照花裤衩大佬的思路就是，前端有一份动态路由表，等到用户登录拿到用户的角色之后根据当前登录用户的角色去筛选出可以访问的路由，形成一份定制路由表，然后动态挂载路由。花裤衩大佬说，这样做的好处就是，前端每开发一个页面不需要让后端再去配一下路由和权限了，从而避免被后端支配，哈哈哈。

回到我自己的项目中。首先，我这是个人项目（可以理解为搞来玩玩的），前端和后端都是自己做，只可能自己折磨自己，当然，这不是最主要的原因。最主要的是，我这个项目中管理员是可以添加角色的，包括给角色分配权限，然后给用户分配角色。然而，路由表又是跟角色挂钩。考虑一种情况，项目上线之后，管理员添加了一个新角色，并且要给这个角色分配菜单。如果采用将路由表放在前端的话那么每个路由的可访问角色都是写死的，要给新添加的角色分配菜单，只能改前端代码，显然不是很合适。所以我才用了后者，就是把路由信息放在后端，后端将路由信息和角色关联起来，用户登录之后请求对应的接口拿到属于这个用户的路由信息（也就是菜单），然后前端对返回的数据格式化，转换成符合 vue-router 的路由格式，然后动态挂载。将路由信息放在后端，这样就可以对路由进行配置了，比如说超级管理员今天很不高兴，不想让某个角色下的用户访问某个路由，直接在该路由下剔除这个角色就可以了（之后会有图动态地演示这一过程），想想是不是很爽。

讲了这么多，不知道我表达清楚了没。小结一下，我这个项目涉及到**操作权限**和**路由访问权限**，前者完全放在后端做，把权限加到接口上。后者则需要前端和后端共同配合，接下会主要地讲我是怎样实现后者的，不会把完整的实现代码放出来，因为没必要，第一点原因是我觉得代码只是工具，重要的是思路，第二点原因我前面也提到过了，我不是很想写那种说教式教程式的文章，所以更多的是分享。只会讲一下大概的实现过程还有一些注意事项，前后端都会提到，但主要是前端。

实现过程
----

说实话，其实在实现过程中后端是占主要地位的，像角色的匹配，路由的筛选都需要后端去做。那么后端要想做到这些，就离不开数据库和数据表，所以先来看看我设计的表吧。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b6726cb879cc4eae963d7d5e0f8ad801~tplv-k3u1fbpfcp-watermark.image)

我给出的这个数据表关系图，除了有关菜单的表，还包括所有的权限表和角色表，可以说是支撑了后端所有有关权限角色的操作。那么其实和接下来要说的内容有关的表就只有三个`roles` 、`menu` 、`menu_role`，其实并不是复杂，不要被吓到了。`roles`表用来存放整个系统的角色，`menu`表用来存放菜单信息（也就是给前端的路由），`menu_role` 表是作为一张中间表来连接 `menu` 表和 `roles` 表的。**这里要补充一下就是，其实用户表和角色表也是有关联的，毕竟用户登录只能拿到用户信息，所以用户、角色、菜单这三者之间必须要有关联。也就说接下来的内容都是建立在基本的用户角色权限功能完整的情况下的**。说完了表，接下来讲一下后端接口要返回的内容

### 1、后端接口

要想知道后端该返回怎样的数据，就得先知道前端需要怎样的数据，前端需要的数据大概长这样：

```
[
  {
    path: '/permission',
    component: Layout,
    redirect: '/permission/page',
    name: 'Permission',
    meta: {
      title: '权限管理',
      icon: 'lock',
      roles: ['super_admin', 'editor']
    },
    children: [
      {
        path: 'role',
        component: () => import('@/views/permission/role'),
        name: 'RolePermission',
        meta: {
          title: '角色信息',
          roles: ['super_admin']
        }
      }
    ]
  },
  {
    path: '/icon',
    component: Layout,
    children: [
      {
        path: 'index',
        component: () => import('@/views/icons/index'),
        name: 'Icons',
        meta: { title: 'Icons', icon: 'icon', noCache: true }
      }
    ]
  }
]
复制代码
```

这上面放出来的数据是最理想的数据了，也就是如果后端返回这样的数据，那么前端直接用就行了。但这是不太可能的，因为可以看到我上面的数据库表有很多字段，为的就是覆盖到大多数情况，那么后端并不知道哪些东西对你前端来说是有用的，而且也不清楚数据组合的方式，要组合起来也很麻烦的，因为表是二维的，无法描述一些层级关系，最主要的是后端返回的`component`字段仅仅是前端组件的一个路径而已，前端肯定还是要转换成自定义的组件的。所以，综上所述**后端只能如数返回所有的字段**，而前端要自己筛选和组合数据。  
现在知道了后端要返回全部的字段，还有一个问题，就是路由的嵌套问题，虽然后端返回所有的字段，但是你至少得告诉前端各个路由之间的嵌套关系吧（也就是父级路由和子路由），细心的小伙伴可能看到了我上面的`menu` 表中有一个 `pid` 字段，这个字段就是用来描述路由之间的关系的，父级路由的 `pid`是 `0` 子路由的 `pid` 是父级路由在表中的 `id` 字段。把这些东西捋顺了之后，就可以写代码了，我还是贴一些后端代码出来吧。

```
public function index(Request $request)
{
    $user = auth('admin')->user();
    
    $menu_ids = MenuRole::whereIn('rid', $user->roles->pluck('id'))->distinct()->pluck('mid');
    $menus = Menu::whereIn('id', $menu_ids)->where('pid', 0)->with([
        'children' => function ($query) use($menu_ids) {
            return $query->whereIn('id', $menu_ids);
        }
    ])->get();
    return $this->response->array($menus->toArray());
}
复制代码
```

熟悉 Laravel 的小伙伴应该很快就能看懂这段代码，不熟悉的也没关系，关注最后返回的数据也可以的。上面这段代码先筛选出用户所有角色的 id（因为一个用户可能会有多个角色），然后在根据角色 id 去找到对应的菜单 id，然后再去 `menu` 表中查出记录即可。 **敲黑板**！这里要注意了，有可能用户拥有的多个角色都可访问某一个路由，这个时候就要去除重复的记录，要不然最后返回给前端的数据中就会有重复的数据，最后导致在页面中出现重复的菜单。

返回的数据大概长这样：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/16db606546e74915a1646aaa65d0a1d7~tplv-k3u1fbpfcp-watermark.image)

### 2、前端格式化路由数据

终于到前端干活了，前端做的事情很简单，就是请求接口，拿到数据，格式化数据，挂载路由。一步一步来吧。

可能大家会觉得拿数据不是很简单吗，但是也要考虑拿数据的方式、拿数据的时机和拿到数据之后保存到哪？  
这里我就不卖关子了，在这个项目中是把请求放在 Vuex 中的 action 中，因为最后拿到的数据是要保存在 Vuex 中，以便左侧菜单组件能够拿到路由信息渲染出对应的菜单，这样做就方便一点，至于时机，是在 router 的 beforeEach 里等到用户登录之后发起请求的。还是把代码贴出来

```
file: @/api/user.js
// 获取动态路由表
export function getAsyncRoutes() {
  return request({
    url: '/admin/menus',
    method: 'get'
  })
}

file: @/store/modules/permission.js
import { getAsyncRoutes } from '@/api/user'
import formatRoutes from '@/utils/formatRoutes'
import Layout from '@/layout'
// 有些依赖就没有完全列出来
// actions
const actions = {
  generateRoutes({ commit }) {
    return new Promise(resolve => {
      // 从服务器获取路由表
      getAsyncRoutes().then(routes => {
        // 格式化路由表
        const accessedRoutes = formatRoutes(routes, Layout)  // formatRoutes函数会在后面放出来
        // 将路由保存到Vuex中
        commit('SET_ROUTES', accessedRoutes)
        resolve(accessedRoutes)
      })
    })
  }
}

export default {
  namespaced: true,
  state,
  mutations,
  actions
}
复制代码
```

上面的代码只是定义了 action，接下来就需要在 vue-router 的路由钩子里面，dispath 定义好的 action 就可以拿到格式化好的数据了

```
import router from './router'
import store from './store'
import { Message } from 'element-ui'
import NProgress from 'nprogress'
import 'nprogress/nprogress.css'
import { getToken } from '@/utils/auth'
import getPageTitle from '@/utils/get-page-title'

NProgress.configure({ showSpinner: false })

const whiteList = ['/login', '/auth-redirect']

router.beforeEach(async(to, from, next) => {
  // 开启页面进度条
  NProgress.start()

  // 设置标题
  document.title = getPageTitle(to.meta.title)

  const hasToken = getToken()
  if (hasToken) {
    if (to.path === '/login') {
      // 已登录，跳转到: '/'
      next({ path: '/' })
      NProgress.done() // 关闭页面进度条
    } else {
      // 是否通过用户信息拿到角色信息
      const hasRoles = store.getters.roles && store.getters.roles.length > 0
      if (hasRoles) {
        // 登录过并且有角色信息，直接进入下一个路由
        next()
      } else {
        try {
          // 获取用户信息
          await store.dispatch('user/getInfo')

          // 重点在这。。。。
          // 根据角色生成路由表
          const accessRoutes = await store.dispatch('permission/generateRoutes')
          // 动态添加路由
          router.addRoutes(accessRoutes)

          next({ ...to, replace: true })
        } catch (error) {
          console.log(error)
          // 清除token，跳转登录页
          await store.dispatch('user/resetToken')
          Message.error(error || 'Has Error')
          next(`/login?redirect=${to.path}`)
          NProgress.done()
        }
      }
    }
  } else {

    if (whiteList.indexOf(to.path) !== -1) {
      // 访问的路径处于白名单中
      next()
    } else {
      // 没有登录，跳转登录页
      next(`/login?redirect=${to.path}`)
      NProgress.done()
    }
  }
})

router.afterEach(() => {
  NProgress.done()
})
复制代码
```

这个时候基本就好了，因为把路由格式化好的信息放到了 Vuex 中，菜单组件也可以拿到数据，会自动渲染的。还有一个要补充的地方就是 `formatRoutes` 函数，这个函数做的事情也很简单，先看代码吧

```
function loadView(component) {
  return (resolve) => require([`@/views/${component}`], resolve)
}

export default function formatRoutes(routes, Layout) {
  const formatRoutesArr = []
  routes.forEach(route => {
    const router = {
      meta: {}
    }
    const {
      pid,
      title,
      path,
      redirect,
      component,
      keep_alive,
      icon,
      name,
      children
    } = route
    if (component === 'Layout') {
      router['component'] = Layout
    } else {
      router['component'] = loadView(component)
    }
    if (redirect !== null) {
      router['redirect'] = redirect
    }
    if (icon !== null) {
      router['meta']['icon'] = icon
    }
    if (children && children instanceof Array && children.length > 0) {
      router['children'] = formatRoutes(children)
    }
    if (name !== null) {
      router['name'] = name
    }
    router['meta']['title'] = title
    router['path'] = path
    if (pid === 0) {
      router['alwaysShow'] = true
    }
    router['meta']['noCache'] = !keep_alive
    formatRoutesArr.push(router)
  })
  // 将404页面添加到最后面
  formatRoutesArr.push({ path: '*', redirect: '/404', hidden: true })
  return formatRoutesArr
}
复制代码
```

可以看到有很多的 if 判断语句，其实就是为了完全筛选出我们需要的信息，比如一些为值 null 的属性，不希望这些属性出现在最后的路由表中，好像只能用这种笨方法了, 其实再在 forEach 嵌一个 for 循环也差不多，都要判断。**敲黑板**！这里要注意的就是，在导入组件的时候最好不要使用 import() 导入，会出很多问题的，导入的时候也要多写一层路径，例如千万不要把`require(['@/views/${component}'], resolve)`写成`require(['@/${component}'], resolve)`因为我最初从数据库拿出来的 `component` 字段是包含了`/view`这层路径的，然后一直出错，都要哭了。

走完上面这些流程基本上都能成功，至于说菜单组件怎么渲染数据，这就不是重点了，然后就到了激动人心的时候了，看看效果。

效果演示
----

1. 先来看看不同用户登录后台的效果，先来看看超级管理员登录之后的界面

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6168fc52079f4d529128f49e1d1faad0~tplv-k3u1fbpfcp-watermark.image)

然后我们换一个帐号，这个帐号拥有的角色是发货员和仓库管理员

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b3da4fd0258c419699fb8dd2820b4ded~tplv-k3u1fbpfcp-watermark.image)

前面我提到的超级管理员可以控制路由可以被哪些角色访问，也来看看吧，具体实现也很简单。就不再这篇文章中讲了。比如现在超级管理员超级不高兴，不想让发货员访问订单管理这个菜单了，这也是可以的，好吧。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2bc211692f5d4be986245351568249d3~tplv-k3u1fbpfcp-watermark.image)

然后这个用户登录之后，就会发现没有订单管理这个菜单了

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a3d06d7b6888401eade69a8def9da8f7~tplv-k3u1fbpfcp-watermark.image)

最后
--

我也是第一次搞这种权限控制，还是前后端一起搞。之前没搞过，也不会，这次也是自己一点点摸索，问题层出不穷，一次一次摔倒又一次次站起来。虽然花了很多的时间，但是最后做出来的时候还是挺有成就感的。我还有太多的东西需要去学习，对自己的要求就是能够坚持下去就好了。

嘿，陌生人。你的点赞或许是对我最大的鼓励！
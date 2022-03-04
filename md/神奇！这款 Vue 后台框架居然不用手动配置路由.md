> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7069595464215035918)*   [vite-plugin-pages](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fhannoeru%2Fvite-plugin-pages "https://github.com/hannoeru/vite-plugin-pages")
*   [vite-plugin-vue-layouts](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FJohnCampionJr%2Fvite-plugin-vue-layouts "https://github.com/JohnCampionJr/vite-plugin-vue-layouts")

使用
==

安装
--

从这两款插件的名字可以看出，这俩是 vite 的插件，并且仅支持 Vue3 ，那我就搬出我的 [Fantastic-template](https://link.juejin.cn?target=https%3A%2F%2Fgitee.com%2Fhooray%2Ffantastic-template "https://gitee.com/hooray/fantastic-template") 项目模版拿来测试了。

首先先安装依赖。因为模版里自带了 vue-router ，所以不需要再安装。

```
pnpm add vite-plugin-pages vite-plugin-vue-layouts -D
复制代码
```

安装好依赖后，需要在 `vite.config.js` 里进行配置，由于这两个插件作用不同，我们一个个来介绍如何使用。

vite-plugin-pages
-----------------

这个是本次介绍的核心，它能够将文件系统生成对应的路由，从而省去手动配置路由的时间。

首先在 `vite.config.js` 增加以下配置：

```
import Pages from 'vite-plugin-pages'

export default {
    plugins: [
        Pages({
            dirs: 'src/views',  // 需要生成路由的文件目录
            exclude: ['**/components/*.vue']  // 排除在外的目录，即不将所有 components 目录下的 .vue 文件生成路由
        })
    ]
}
复制代码
```

目前只需要配置这 2 个参数就够了，其它还有更多参数可以去 [vite-plugin-pages](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fhannoeru%2Fvite-plugin-pages "https://github.com/hannoeru/vite-plugin-pages") 项目页了解。

接着在页面中引入就可以使用了。

```
import { createRouter } from 'vue-router'
import routes from '~pages'

const router = createRouter({
    // ...
    routes
})
复制代码
```

没错，就是这么简单，当然也有一些高阶用法。比如通过 params 传参的路由，要怎么配置呢？

在 vite-plugin-pages 里提供了一种专用的写法，就是 `[ ]` 。例如 `src/views/example/[id].vue` 这个文件，最终生成的路由 `path` 就是 `/example/:id` 。

针对 404 页面，我们可以在创建个这样的文件 `src/views/[...all].vue` ，它生成的路由就是 `/:all(.*)*` 。

除此之外，我们还可以在 `.vue` 文件中增加 `<route></route>` 代码块，这里面默认接收 `json5` 格式的路由配置。需要注意的是，如果在 `<route></route>` 里配置了 `path` 和 `name` ，将会覆盖自动生成的 `path` 和 `name` 。

```
<route>
{
    path: '/xxx/yyy',
    name: 'zzz',
    meta: {
        title: 'test page'
    }
}
</route>

<template>
	<div>
        This is a test page.
    </div>
</template>
复制代码
```

这时候似乎还缺点什么，对的，那就是嵌套路由。通过 vite-plugin-pages 自动生成的路由，都是一级路由。而实际项目开发中，我们会使用嵌套路由的特性，搭配 `<router-view></router-view>` 组件实现一些布局效果。

这时候就要介绍下面这款插件了

vite-plugin-vue-layouts
-----------------------

首先在 `vite.config.js` 修改下配置：

```
import Pages from 'vite-plugin-pages'
import Layouts from 'vite-plugin-vue-layouts'

export default {
    plugins: [
        Pages({
            dirs: 'src/views',  // 需要生成路由的文件目录
            exclude: ['**/components/*.vue']  // 排除在外的目录，即所有 components 目录下的 .vue 文件都不会生成路由
        }),
        Layouts({
            layoutsDirs: 'src/layout',  // 布局文件存放目录
            defaultLayout: 'index'  // 默认布局，对应 src/layout/index.vue
        })
    ]
}
复制代码
```

更多配置参数请查看 [vite-plugin-vue-layouts](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FJohnCampionJr%2Fvite-plugin-vue-layouts "https://github.com/JohnCampionJr/vite-plugin-vue-layouts") 项目页。

也还是一样，配置好后，就可以直接使用了。

```
import { createRouter } from 'vue-router'
import { setupLayouts } from 'virtual:generated-layouts'
import generatedRoutes from 'virtual:generated-pages'

const routes = setupLayouts(generatedRoutes)

const router = createRouter({
    // ...
    routes
})
复制代码
```

这个插件只做一件事，就是把通过 vite-plugin-pages 生成的一级路由处理成嵌套路由，大概就是这样：

```
// 处理前
{
    path: '/login',
    component: () => import('/src/views/login.vue'),
    name: 'login'
}

// 处理后
{
    path: '/login',
    component: () => import('/src/layout/index.vue'),
    children: [
        {
            path: '',
            component: () => import('/src/views/login.vue'),
            name: 'login'
        }
    ]
}
复制代码
```

如果你有多种布局，可以在 `<route><route>` 里进行设置：

```
<route>
{
    meta: {
        layout: 'other'
    }
}
</route>
复制代码
```

甚至还可以做一些魔改，比如项目中有的路由是需要用到布局页面的，有的则不需要，那我们可以将不需要的页面设置为 `layout: false` ：

```
<route>
{
    meta: {
        layout: false
    }
}
</route>
复制代码
```

同时在路由文件处使用下面这段代码：

```
import { createRouter } from 'vue-router'
import { setupLayouts } from 'virtual:generated-layouts'
import generatedRoutes from 'virtual:generated-pages'

let routes = []
generatedRoutes.forEach(v => {
    routes.push(v?.meta?.layout != false ? setupLayouts([v])[0] : v)
})

const router = createRouter({
    // ...
    routes
})
复制代码
```

总结
--

通过下面这张图配合总结吧：

```
文件系统                           路由地址                          路由 name

views
├─ example
│    ├─ components
│    │    └─ List
│    │         └─ index.vue
│    ├─ params
│    │    └─ [id].vue              /example/params/:id              example-params
│    ├─ axios.vue                  /example/axios                   example-axios
│    ├─ cookie.vue                 /example/cookie                  example-cookie
│    └─ svgicon.vue                /example/svgicon                 example-svgicon
├─ [...all].vue                    /:all(.*)*                       all
├─ index.vue                       /                                index
└─ login.vue                       /login                           login
复制代码
```

*   使用路由参数需通过 `[ ]` 将参数名包裹，并设为文件名
*   文件夹不会生成路由，例如 example 文件夹并没有生成 `/example` 路由
*   路由 name 会根据文件的目录结构自动生成，并用 `-` 连接，可确保 name 的唯一性
*   所有 components 目录均不会生成路由

点题
==

这么好用的特性，我也是第一时间集成进了我开发的 [Fantastic-admin](https://link.juejin.cn?target=https%3A%2F%2Fhooray.gitee.io%2Ffantastic-admin "https://hooray.gitee.io/fantastic-admin") 后台框架中，如果你感兴趣，可以[点我详细了解](https://link.juejin.cn?target=https%3A%2F%2Fhooray.gitee.io%2Ffantastic-admin%2Fguide%2Ffile-system-route.html "https://hooray.gitee.io/fantastic-admin/guide/file-system-route.html")如何使用。
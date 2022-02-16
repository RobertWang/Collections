> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651583514&idx=1&sn=f8ae9df2ad63ec48e4b248f821b43056&chksm=802523dbb752aacd42452c27685040ce03451c41a16b961e00fdd65df27f1f291a481555477d&scene=21#wechat_redirect)

创建项目  

-------

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHpzBfbCH4yXnD66ZyxnLymBemm6qCW2wlgI2NpgBbU3M49ZBuB9j4A6CAFoQD6YhbM84UovFkVVicw/640?wx_fmt=png)

基础语法
----

### 定义 data

*   script 标签上 lang="ts"
    
*   定义一个类型`type`或者接口`interface`来约束`data`
    
*   可以使用`ref`或者`toRefs`来定义响应式数据
    
*   使用`ref`在`setup`读取的时候需要获取`xxx.value`, 但在`template`中不需要
    
*   使用`reactive`时，可以用`toRefs`解构导出，在`template`就可以直接使用了
    

```
<script lang="ts">
import { defineComponent, reactive, ref, toRefs } from 'vue';

type Todo = {
  id: number,
  name: string,
  completed: boolean
}

export default defineComponent({
  const data = reactive({
    todoList: [] as Todo[]
  })
  const count = ref(0);
  console.log(count.value)
  return {
    ...toRefs(data)
  }
})
</script>
```

### 定义 props

`props`需要使用`PropType`泛型来约束。

```
<script lang="ts">
import { defineComponent, PropType} from 'vue';

interface UserInfo = {
  id: number,
  name: string,
  age: number
}

export default defineComponent({
  props: {
    userInfo: {
      type: Object as PropType<UserInfo>, // 泛型类型
      required: true
    }
  },
})
</script>
```

### 定义 methods

```
<script lang="ts">
import { defineComponent, reactive, ref, toRefs } from 'vue';

type Todo = {
  id: number,
  name: string,
  completed: boolean
}

export default defineComponent({
  const data = reactive({
    todoList: [] as Todo[]
  })
  // 约束输入和输出类型
  const newTodo = (name: string):Todo  => {
    return {
      id: this.items.length + 1,
      name,
      completed: false
    };
  }
  const addTodo = (todo: Todo): void => {
    data.todoList.push(todo)
  }
  return {
    ...toRefs(data),
    newTodo,
    addTodo
  }
})
</script>
```

vue-router
----------

*   `createRouter`创建`router`实例
    
*   `router`的模式分为：
    
*   `createWebHistory` -- history 模式
    
*   `createWebHashHistory` -- hash 模式
    
*   `routes`的约束类型是`RouteRecordRaw`
    

```
import { createRouter, createWebHistory, RouteRecordRaw } from 'vue-router';
import Home from '../views/Home.vue';
const routes: Array< RouteRecordRaw > = [
  {
    path: '/',
    name: 'Home',
    component: Home,
  },
  {
    path: '/about',
    name: 'About',
    component: () => import(/* webpackChunkName: "about" */ '../views/About.vue')
  }
];

const router = createRouter({
  history: createWebHistory(process.env.BASE_URL),
  routes
});

export default router;
```

### 扩展路由额外属性

在实际项目开发中，常常会遇到这么一个场景，某一个路由是不需要渲染到侧边栏导航上的，此时我们可以给该路由添加一个 hidden 属性来实现。

在 ts 的强类型约束下，添加额外属性就会报错，那么我们就需要扩展`RouteRecordRaw`类型。

```
// 联合类型
type RouteConfig = RouteRecordRaw & {hidden?: boolean}; //hidden 是可选属性
const routes: Array<RouteConfig> = [
  {
    path: '/',
    name: 'Home',
    component: Home,
    hidden: true,
    meta: {
      permission: true,
      icon: ''
    }
  }
];
```

### 在 setup 中使用

需要导入`useRouter`创建一个`router`实例。

```
<script lang="ts">
import { useRouter } from 'vue-router';
import { defineComponent } from 'vue';
export default defineComponent({
  setup () {
    const router = useRouter();
    goRoute(path) {
       router.push({path})
    }
  }
})
</script>
```

vuex
----

### 使用 this.$store

```
import { createStore } from 'vuex';
export type State = {
  count: number
}

export default createStore({
  state: {
    count: 0
  }
});
```

需要创建一个声明文件`vuex.d.ts`

```
// vuex.d.ts
import {ComponentCustomProperties} from 'vue';
import {Store} from 'vuex';
import {State} from './store'
declare module '@vue/runtime-core' {
    interface ComponentCustomProperties {
        $store: Store<State>
    }
}
```

### 在 setup 中使用

1.  ```
    定义InjecktionKey
    ```
    
2.  ```
    在安装插件时传入key
    ```
    
3.  ```
    在使用useStore时传入
    ```
    

```
import { InjectionKey } from 'vue';
import { createStore, Store } from 'vuex';

export type State = {
  count: number
}
// 创建一个injectionKey
export const key: InjectionKey<Store<State>> = Symbol('key');
```

```
// main.ts
import store, { key } from './store';
app.use(store, key);
```

```
<script lang="ts">
import { useStore } from 'vuex';
import { key } from '@/store';
export default defineComponent({
  setup () {
    const store = useStore(key);
    const count = computed(() => store.state.count);
    return {
      count
    }
  }
})
</script>
```

### 模块

新增一个`todo`模块。导入的模块，需要是一个`vuex`中的 interface `Module`的对象, 接收两个泛型约束，第一个是**该模块类型**，第二个是**根模块类型**。

```
// modules/todo.ts
import { Module } from 'vuex';
import { State } from '../index.ts';

type Todo = {
  id: number,
  name: string,
  completed: boolean
}

const initialState = {
  todos: [] as Todo[]
};

export type TodoState = typeof initialState;

export default {
  namespaced: true,
  state: initialState,
  mutations: {
    addTodo (state, payload: Todo) {
      state.todos.push(payload);
    }
  }
} as Module<TodoState, State>; //Module<S, R> S 该模块类型 R根模块类型
```

```
// index.ts
export type State = {
  count: number,
  todo?: TodoState // 这里必须是可选，不然state会报错
}

export default createStore({
  state: {
    count: 0
  }
  modules: {
    todo
  }
});
```

使用：

```
setup () {
  console.log(store.state.todo?.todos);
}
```

elementPlus
-----------

```
yarn add element-plus
```

### 完整引入

```
import { createApp } from 'vue'
import ElementPlus from 'element-plus';import 'element-plus/lib/theme-chalk/index.css';import App from './App.vue';
import 'dayjs/locale/zh-cn'
import locale from 'element-plus/lib/locale/lang/zh-cn'
const app = createApp(App)
app.use(ElementPlus, { size: 'small', zIndex: 3000, locale })
app.mount('#app')
```

### 按需加载

需要安装`babel-plugin-component`插件:

```
yarn add babel-plugin-component -D

// babel.config.js
plugins: [
    [收起
      'component',
      {
        libraryName: 'element-plus',
        styleLibraryName: 'theme-chalk'
      }
    ]
]
```

```
import 'element-plus/lib/theme-chalk/index.css';
import 'dayjs/locale/zh-cn';
import locale from 'element-plus/lib/locale';
import lang from 'element-plus/lib/locale/lang/zh-cn';
import {
  ElAside,
  ElButton,
  ElButtonGroup,
} from 'element-plus';

const components: any[] = [
  ElAside,
  ElButton,
  ElButtonGroup,
];

const plugins:any[] = [
  ElLoading,
  ElMessage,
  ElMessageBox,
  ElNotification
];

const element = (app: any):any => {
  // 国际化
  locale.use(lang);
  // 全局配置
  app.config.globalProperties.$ELEMENT = { size: 'small' };
  
  components.forEach(component => {
    app.component(component.name, component);
  });

  plugins.forEach(plugin => {
    app.use(plugin);
  });
};

export default element;
```

```
// main.ts
import element from './plugin/elemment'

const app = createApp(App);
element(app);
```

axios
-----

axios 的安装使用和 vue2 上没有什么大的区别，如果需要做一些扩展属性，还是需要声明一个新的类型。

```
type Config = AxiosRequestConfig & {successNotice? : boolean, errorNotice? : boolean}
```

```
import axios, { AxiosResponse, AxiosRequestConfig } from 'axios';
import { ElMessage } from 'element-plus';
const instance = axios.create({
  baseURL: process.env.VUE_APP_API_BASE_URL || '',
  timeout: 120 * 1000,
  withCredentials: true
});

// 错误处理
const err = (error) => {
  if (error.message.includes('timeout')) {
    ElMessage({
      message: '请求超时，请刷新网页重试',
      type: 'error'
    });
  }
  if (error.response) {
    const data = error.response.data;
    if (error.response.status === 403) {
      ElMessage({
        message: 'Forbidden',
        type: 'error'
      });
    }
    if (error.response.status === 401) {
      ElMessage({
        message: 'Unauthorized',
        type: 'error'
      });
    }
  }
  return Promise.reject(error);
};

type Config = AxiosRequestConfig & {successNotice? : boolean, errorNotice? : boolean}

// 请求拦截
instance.interceptors.request.use((config: Config) => {
  config.headers['Access-Token'] = localStorage.getItem('token') || '';
  return config;
}, err);

// 响应拦截
instance.interceptors.response.use((response: AxiosResponse) => {
  const config: Config = response.config;

  const code = Number(response.data.status);
  if (code === 200) {
    if (config && config.successNotice) {
      ElMessage({
        message: response.data.msg,
        type: 'success'
      });
    }
    return response.data;
  } else {
    let errCode = [402, 403];
    if (errCode.includes(response.data.code)) {
      ElMessage({
        message: response.data.msg,
        type: 'warning'
      });
    }
  }
}, err);

export default instance;
```

setup script
------------

官方提供了一个**实验性**的写法，直接在`script`里面写`setup`的内容，即：`setup script`。

之前我们写组件是这样的：

```
<template>
  <div>
    {{count}}
    <ImgReview></ImgReview >
  </div>
</template>
<script lang="ts">
import { ref, defineComponent } from "vue";
import ImgReview from "./components/ImgReview.vue";

export default defineComponent({
  components: {
    ImgReview,
  },
  setup() {
    const count = ref(0);
    return { count };
  }
});
</script>
```

启用`setup script`后：在`script`上加上`setup`

```
<template>
  <div>
    {{count}}
    <ImgReview></ImgReview>
  </div>
</template>
<script lang="ts" setup>
import { ref } from "vue";
import ImgReview from "./components/ImgReview.vue";
const count = ref(0);
</script>
```

是不是看起来简洁了很多，组件直接导入就行了，不用注册组件，数据定义了就可以用。其实我们可以简单的理解为`script`包括的内容就是`setup`中的，并做了`return`。

### 导出方法

```
<script lang="ts" setup>
const handleClick = (type: string) => {
  console.log(type);
}
</script>
```

### 定义 props

使用`props`需要用到`defineProps`来定义，具体用法跟之前的`props`写法类似：

#### 基础用法

```
<script lang="ts" setup>
import { defineProps } from "vue";
const props = defineProps(['userInfo', 'gameId']);
</script>
```

#### 构造函数进行检查 给 props 定义类型：

```
const props = defineProps({
  gameId: Number,
  userInfo: {
      type: Object,
      required: true
  }
});
```

#### 使用类型注解进行检查

```
defineProps<{
  name: string
  phoneNumber: number
  userInfo: object
  tags: string[]
}>()
```

可以先定义好类型：

```
interface UserInfo {
  id: number,
  name: string,
  age: number
}

defineProps<{
  name: string
  userInfo: UserInfo
}>()
```

### defineEmit

```
<script lang="ts" setup>
import { defineEmit } from 'vue';

// expects emits options
const emit = defineEmit(['kk', 'up']);
const handleClick = () => {
  emit('kk', '点了我');
};
</script>
```

```
<Comp @kk="handleClick"/>

<script lang="ts" setup>
const handleClick = (data) => {
  console.log(data)
}
</script>
```

### 获取上下文

在标准组件写法里，setup 函数默认支持两个入参：

<table><thead><tr data-darkmode-bgcolor-16450327850396="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16450327850396="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-darkmode-bgcolor-16450327850396="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16450327850396="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); text-align: center; min-width: 85px;">参数</th><th data-darkmode-bgcolor-16450327850396="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16450327850396="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); text-align: center; min-width: 85px;">类型</th><th data-darkmode-bgcolor-16450327850396="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16450327850396="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); text-align: center; min-width: 85px;">含义</th></tr></thead><tbody><tr data-darkmode-bgcolor-16450327850396="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16450327850396="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-bgcolor-16450327850396="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16450327850396="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); text-align: center; min-width: 85px;">props</td><td data-darkmode-bgcolor-16450327850396="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16450327850396="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); text-align: center; min-width: 85px;">object</td><td data-darkmode-bgcolor-16450327850396="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16450327850396="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); text-align: center; min-width: 85px;">由父组件传递下来的数据</td></tr><tr data-darkmode-bgcolor-16450327850396="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16450327850396="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-bgcolor-16450327850396="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16450327850396="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); text-align: center; min-width: 85px;">context</td><td data-darkmode-bgcolor-16450327850396="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16450327850396="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); text-align: center; min-width: 85px;">object</td><td data-darkmode-bgcolor-16450327850396="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16450327850396="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); text-align: center; min-width: 85px;">组件的执行上下文</td></tr></tbody></table>

在 setup script 中使用 useContext 获取上下文：

```
<script lang="ts" setup>
 import { useContext } from 'vue'
 const { slots, attrs } = useContext();
</script>
```

获取到的`slots`,`attrs`跟`setup`里面的是一样的。

> 作者：FinGet
> 
> https://juejin.cn/post/6980267119933931551

- EOF -
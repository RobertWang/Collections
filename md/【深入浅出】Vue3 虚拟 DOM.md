> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU2Mzk1NzkwOA==&mid=2247489774&idx=1&sn=86dbfcf03dc0a8f705298d4f820b73b7&chksm=fc530115cb24880357e0dce16d54407eeb6469a554160af82f25f11b1f33545536f2006ffa32&scene=132#wechat_redirect)

序言  

-----

> 首发在我的博客 深入 Vue3 虚拟 DOM
> 
> 译自：diving-into-the-vue-3s-virtual-dom-medium
> 
> 作者：Lachlan Miller

多数情况下我们不需要考虑 Vue 组件内部是如何构成的。但有一些库会帮助我们理解，比如 Vue Test Utils 的 findComponent 函数。还有一个我们都应该很熟悉的 Vue 开发工具 —— Vue DevTools，它显示了应用的组件层次结构，并且我们可以对它进行编辑操作等。

我们本篇要做的是：实现 Vue Test Utils API 的一部分，即 `findComponent` 函数。

设计 findComponent
----------------

```
- div 
  - span (show: true) 
    - 'Visible'
```

它的内部层次关系是：

```
HTMLDivElement -> HTMLSpanElement -> TextNode
```

如果 `show` 属性变成 `false`。Vue 虚拟 DOM 会进行如下更新：

```
- div 
  - span (show: false) 
    - 'Visible'
```

接着，Vue 会更新 DOM，移除`'span'` 元素。

那么，我们设想一下，`findComponent` 函数，它的调用可能会是类似这样的结构：

```
const { createApp } = require('vue')

const App = {
  template: `
    <C>
      <B>
        <A />
      </B>
    </C>
  `
}

const app = createApp(App).mount('#app')

const component = findComponent(A, { within: app })

// 我们通过 findComponent 方法找到了 <A/> 标签。
```

打印 findComponent
----------------

接着，我们先写几个简单组件，如下：

```
// import jsdom-global. We need a global `document` for this to work.
require('jsdom-global')()
const { createApp, h } = require('vue')

// some components
const A = { 
  name: 'A',
  data() {
    return { msg: 'msg' }
  },
  render() {
    return h('div', 'A')
  }
}

const B = { 
  name: 'B',
  render() {
    return h('span', h(A))
  }
}

const C = { 
  name: 'C',
  data() {
    return { foo: 'bar' }
  },
  render() {
    return h('p', { id: 'a', foo: this.foo }, h(B))
  }
}

// mount the app!
const app = createApp(C).mount(document.createElement('div'))
```

*   我们需要在 Node.js v14+ 环境，因为我们要用到 可选链。且需要安装 Vue、jsdom 和 jsdom-global。
    

我们可以看到 A , B , C 三个组件，其中 A , C 组件有 data 属性，它会帮助我们深入研究 VDOM。

你可以打印试试：

```
console.log(app)
console.log(Object.keys(app))
```

结果为 `{}`，因为 `Object.keys` 只会显示可枚举的属性。

我们可以尝试打印隐藏的不可枚举的属性：

```
console.log(app.$)
```

可以得到大量输出信息：

```
<ref *1> { 
  uid: 0, 
  vnode: {
    __v_isVNode: true, 
    __v_skip: true, 
    type: { 
      name: 'C', 
      data: [Function: data], 
      render: [Function: render], 
      __props: [] 
  }, // hundreds of lines ...
```

再打印：

```
console.log(Object.keys(app.$))
```

输出：

```
Press ENTER or type command to continue 
[ 
'uid', 'vnode', 'type', 'parent', 'appContext', 'root', 'next', 'subTree', 'update', 'render', 'proxy', 'withProxy', 'effects', 'provides', 'accessCache', 'renderCache', 'ctx', 'data', 'props', 'attrs', 'slots', 'refs', 'setupState', 'setupContext', 'suspense', 'asyncDep', 'asyncResolved', 'isMounted', 'isUnmounted', 'isDeactivated', 'bc', 'c', 'bm', 'm', 'bu', 'u', 'um', 'bum', 'da', 'a', 'rtg', 'rtc', 'ec', 'emit', 'emitted' 
]
```

我们可以看到一些很熟悉的属性：比如 `slots`、`data`，`suspense` 是一个新特性，`emit` 无需多言。还有比如 `attrs`、`bc`、 `c`、`bm` 这些是生命周期钩子：`bc` 是 `beforeCreate`, `c` 是 `created`。也有一些内部唯一的生命周期钩子，如 `rtg`，也就是 `renderTriggered`, 当 `props` 或 `data` 发生变化时，用于更新操作，从而再渲染。

> 本篇我们需要特别关注的是：`vnode`、`subTree`、`component`、`type` 和 `children`。

匹配 findComponent
----------------

来先看 `vnode`，它有很多属性，我们需要关注的是 `type` 和 `component` 这两个。

// 打印 console.log(app.$.vnode.component)

```
console.log(app.$.vnode.component) 
<ref *1> { 
  uid: 0, 
  vnode: { 
    __v_isVNode: true, 
    __v_skip: true, 
    type: { 
      name: 'C', 
      data: [Function: data], 
      render: [Function: render], 
      __props: [] 
  }, // ... many more things ... } }
```

`type` 很有意思！它与我们之前定义的 `C` 组件一样，我们可以看到它也有 `[Function: data]`（我们在前面定义了一个 `msg` 数据是我们的查找目标）。实际上我们尝试可以做以下打印：

```
console.log(C === app.$.vnode.component.type) //=> true
```

天呐！二者竟然是相等的！😮

```
console.log(C === app.$.vnode.type) //=> true
```

这样也是相等的！😮

（你是否会疑问这两个属性为什么会指向同一个对象？这里先暂且按下不表、自行探索。）

无论如何，我们算是得到了寻找到组件的途径。

通过这里的找寻过程，我们还能再进一步得到以下相等关系：

```
console.log( 
  app.$
  .subTree.children[0].component
  .subTree.children[0].component.type === A) //=> true
```

在本例中，`div` 节点的 `subTree.children` 数组长度是 2 。我们知道了虚拟 DOM 的递归机制，就可以沿着这个方向：`subTree -> children -> component` 来给出我们的递归解决方案。

实现 findComponent
----------------

我们首先实现 `matches` 函数，用于判断是当前 vnode 节点和目标是否相等。

```
function matches(vnode, target) { 
  return vnode?.type === target
}
```

然后是 `findComponent` 函数，它是我们调用并查找内部递归函数的公共 API。

```
function findComponent(comp, { within }) { 
  const result = find([within.$], comp) 
  if (result) { 
    return result 
  } 
}
```

此处的 `find` 方法的实现是我们要重点讨论的。

我们知道写递归，最重要的是判断什么时候结束 loop，所以 find 函数应该先是这样的：

```
function find(vnodes, target) { 
  if (!Array.isArray(vnodes)) { 
    return 
  } 
}
```

然后，在遍历 vnode 时，如果找到匹配的组件，则将其返回。如果找不到匹配的组件，则可能需要检查 vnode.subTree.children 是否已定义，从而更深层次的查询及匹配。最后，如果都没有，我们则返回累加器 acc。所以，代码如下：

```
function find(vnodes, target) {
  if (!Array.isArray(vnodes)) {
    return 
  }

  return vnodes.reduce((acc, vnode) => {
    if (matches(vnode, target)) {
      return vnode
    }

    if (vnode?.subTree?.children) {
      return find(vnode.subTree.children, target)
    }

    return acc
  }, {})
}
```

如果你在 `if (vnode?.subTree?.children) {` 这里进行一个打印 `console.log`，你能找到 `B` 组件，但是我们的目标 `A` 组件的路径如下：

```
app.$ 
  .subTree.children[0].component 
  .subTree.children[0].component.type === A) //=> true
```

所以我们再次调用了 `find` 方法：`find(vnode.subTree.children, target)`，在下一次迭代中查找的第一个参数将是`app.$.subTree.children`，它是 vnode 的数组。我们不仅需要检查`vnode.subTree.children`，还需要检查`vnode.component.subTree`。

所以，最后 find 方法如下：

```
function find(vnodes, target) {
  if (!Array.isArray(vnodes)) {
    return 
  }

  return vnodes.reduce((acc, vnode) => {
    if (matches(vnode, target)) {
      return vnode
    }

    if (vnode?.subTree?.children) {
      return find(vnode.subTree.children, target)
    }

    if (vnode?.component?.subTree) {
      return find(vnode.component.subTree.children, target)
    }

    return acc
  }, {})
}
```

然后我们再调用它：

```
const result = findComponent(A, { within: app })

console.log( result.component.proxy.msg ) // => 'msg'
```

我们成功了！通过 `findComponent`，找到了 `msg`!

如果你以前使用过 Vue Test Utils，可能见过类似的东西 `wrapper.vm.msg`，它实际上是在内部访问 `proxy`（对于 Vue 3）或 `vm`（对于 Vue 2）。

小结
--

本篇的实现并非完美，现实实现上还需要执行更多检查。例如，如果使用 `template`或 `Suspense`组件时，需要作更多判断。不过这些你可以在 Vue Test Utils 源码 中可以看到，希望能帮助你进一步理解虚拟 DOM。

如果你觉得这篇内容对你挺有启发，我想邀请你帮我三个小忙：  

*   点个【在看】，或者分享转发，让更多的人也能看到这篇内容
    
*   关注公众号【趣谈前端】，定期分享 **工程化** / **可视化** / **低代码** / **优秀开源**。
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7023906112843808804)*   [工作中遇到的 50 个 JavaScript 的基础知识点，满分找我拿奖品！](https://juejin.cn/post/7020940475133591566 "https://juejin.cn/post/7020940475133591566")【阅读：7.8k，点赞：285】
*   [万字总结」熬夜总结 50 个 JS 的高级知识点，全都会你就是神！！！](https://juejin.cn/post/7022795467821940773 "https://juejin.cn/post/7022795467821940773")【阅读：1.5w，点赞：812】

今天就给大家分享一下我笔记中的 **56 个 JavaScript 手写知识点**

> 注明：此文章不含**算法题**

面试常考
----

### 1、实现原生的 AJAX 请求

```
const ajax = {
    get(url, fn) {
        const xhr = new XMLHttpRequest()
        xhr.open('GET', url, true)// 第三个参数异步与否
        xhr.onreadystatechange = function() {
            if (xhr.readyState === 4) {
                fn(xhr.responeText)
            }
        }
        xhr.send()
    },
    post(url, data, fn) {
        const xhr = new XMLHttpRequest()
        xhr.open('POST', url, true)
        xhr.setRequestHeader('Content-type','application/x-www-form-urlencoded')
        xhr.onreadystatechange = function () {
            if (xhr.readyState === 4) {
                fn(xhr.responeText)
            }
        }
        xhr.send(data)
    }
}
复制代码

```

### 2、手写 new 的过程

```
function myNew(fn, ...args) {

    const obj = {}

    obj.__proto__ = fn.prototype

    fn.apply(obj, args)

    return obj
}
复制代码

```

### 3、instanceof 关键字

```
function instanceOf(father, child) {
    const fp = father.prototype
    var cp = child.__proto__

    while (cp) {
        if (cp === fp) {
            return true
        }
        cp = cp.__proto__
    }

    return false
}
复制代码

```

### 4、实现防抖函数

```
function debounce(fn, delay = 500) {
    let timer;
    return function () {
        if (timer) {
            clearTimeout(timer)
        }
        const args = arguments        
        timer = setTimeout(() => {
            fn.apply(this, args) // 改变this指向为调用debounce所指的对象
        }, delay)
    }
}
复制代码

```

### 5、实现节流函数

```
function throttle(fn, delay = 200) {
    let flag = true
    return function () {
        if (!flag) return
        flag = false
        const args = arguments
        setTimeout(() => {
            fn.apply(this, args)
            flag = true
        }, delay)
    }
}
复制代码

```

### 6、实现数组去重

> 题目描述：实现一个数组的去重

```
// 第一种：Map记录
function quchong1(arr) {
    const newArr = []
    arr.reduce((pre, next) => {
        if (!pre[next]) {
            pre[next] = 1
            newArr.push(next)
        }
        return pre
    }, {})
    return newArr
}

// 第二种：Set去重
function quchong2(arr) {
    return [...new Set(arr)]
}
复制代码

```

### 7、用 setTimeout 实现 setInterval

> 题目描述：setinterval 用来实现循环定时调用 可能会存在一定的问题 能用 settimeout 解决吗

```
function mySetTimout(fn, delay) {
    let timer = null
    const interval = () => {
        fn()
        timer = setTimeout(interval, delay)
    }
    setTimeout(interval, delay)
    return {
        cancel: () => {
            clearTimeout(timer)
        }
    }
}

// 测试
const { cancel } = mySetTimout(() => console.log(888), 1000)
setTimeout(() => {
    cancel()
}, 4000)
复制代码

```

### 8、用 setInterval 实现 setTimeout

> 题目说明：没有，就是想刁难你

```
function mySetInterval(fn, delay) {
    const timer = setInterval(() => {
        fn()
        clearInterval(timer)
    }, delay)
}

// 测试
mySetInterval(() => console.log(888), 1000)
复制代码

```

### 9、实现一个 compose 函数

> 题目说明：实现以下效果

```
function fn1(x) {
    return x + 1;
}
function fn2(x) {
    return x + 2;
}
function fn3(x) {
    return x + 3;
}
function fn4(x) {
    return x + 4;
}
const a = compose(fn1, fn2, fn3, fn4);
console.log(a)
console.log(a(1)); // 1+2+3+4=11
复制代码

```

> 实现如下：

```
function compose(...fn) {
    if (fn.length === 0) return (num) => num
    if (fn.length === 1) return fn[0]
    return fn.reduce((pre, next) => {
        return (num) => {
            return next(pre(num))
        }
    })
}
复制代码

```

### 10、实现一个科里化函数

> 题目要求：

```
const add = (a, b, c) => a + b + c;
const a = currying(add, 1);
console.log(a(2,3)) // 1 + 2 + 3=6
复制代码

```

> 实现如下：

```
function currying(fn, ...args1) {
  // 获取fn参数有几个
  const length = fn.length
  let allArgs = [...args1]
  const res = (...arg2) => {
    allArgs = [...allArgs, ...arg2]
    // 长度相等就返回执行结果
    if (allArgs.length === length) {
      return fn(...allArgs)
    } else {
      // 不相等继续返回函数
      return res
    }
  }
  return res
}

// 测试：
const add = (a, b, c) => a + b + c;
const a = currying(add, 1);
console.log(a(2,3))
复制代码

```

### 11、实现一个 LRU 缓存函数

> 题目说明：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/866902be3c894edebc9a70446ed664df~tplv-k3u1fbpfcp-watermark.awebp?)

> 实现如下：

```
class LRUCache {
  constructor(size) {
    this.size = size
    this.cache = new Map()
  }

  get(key) {
    const hasKey = this.cache.has(key)
    if (hasKey) {
      const val = this.cache.get(key)
      this.cache.delete(key)
      this.cache.set(key, val)
      return val
    } else {
      return -1
    }
  }

  put(key, val) {
    const hasKey = this.cache.has(key)
    if (hasKey) {
      this.cache.delete(key)
    }
    this.cache.set(key, val)
    if (this.cache.size > this.size) {
      this.cache.delete(this.cache.keys().next().value)
    }
  }

}
复制代码

```

### 12、简单实现 发布订阅模式

> 题目描述: 实现一个发布订阅模式拥有`on emit once off`方法

```
class EventEmitter {
    constructor() {
        this.cache = {}
    }

    on(name, fn) {
        const tasks = this.cache[name]
        if (tasks) {
            this.cache[name].push(fn)
        } else {
            this.cache[name] = [fn]
        }
    }

    off(name, fn) {
        const tasks = this.cache[name]
        if (task) {
            const index = tasks.findIndex(item => item === fn)
            if (index >= 0) {
                this.cache[name].splice(index, 1)
            }
        }
    }

    emit(name, once = false, ...args) {
        // 复制一份。防止回调里继续on，导致死循环
        const tasks = this.cache[name].slice()
        if (tasks) {
            for (let fn of tasks) {
                fn(...args)
            }
        }
        if (once) {
            delete this.cache[name]
        }
    }

    once(name, ...args) {
        this.emit(name, true, ...args)
    }
}
复制代码

```

### 13、实现 JSON.parse

> 题目描述：实现`JSON.parse`

```
function parse (json) {
    return eval("(" + json + ")");
}
复制代码

```

### 14、将 DOM 转化成树结构对象

> 题目描述：

```
<div>
    <span></span>
    <ul>
        <li></li>
        <li></li>
    </ul>
</div>

将上方的DOM转化为下面的树结构对象

{
    tag: 'DIV',
    children: [
        { tag: 'SPAN', children: [] },
        {
            tag: 'UL',
            children: [
                { tag: 'LI', children: [] },
                { tag: 'LI', children: [] }
            ]
        }
    ]
}
复制代码

```

> 实现如下：

```
function dom2tree(dom) {
    const obj = {}
    obj.tag = dom.tagName
    obj.children = []
    dom.childNodes.forEach(child => obj.children.push(dom2tree(child)))
    return obj
}
复制代码

```

### 15、将树结构转换为 DOM

> 题目描述：

```
{
    tag: 'DIV',
    children: [
        { tag: 'SPAN', children: [] },
        {
            tag: 'UL',
            children: [
                { tag: 'LI', children: [] },
                { tag: 'LI', children: [] }
            ]
        }
    ]
}

将上方的树结构对象转化为下面的DOM

<div>
    <span></span>
    <ul>
        <li></li>
        <li></li>
    </ul>
</div>
复制代码

```

> 实现如下：

```
// 真正的渲染函数
function _render(vnode) {
  // 如果是数字类型转化为字符串
  if (typeof vnode === "number") {
    vnode = String(vnode);
  }
  // 字符串类型直接就是文本节点
  if (typeof vnode === "string") {
    return document.createTextNode(vnode);
  }
  // 普通DOM
  const dom = document.createElement(vnode.tag);
  if (vnode.attrs) {
    // 遍历属性
    Object.keys(vnode.attrs).forEach((key) => {
      const value = vnode.attrs[key];
      dom.setAttribute(key, value);
    });
  }
  // 子数组进行递归操作
  vnode.children.forEach((child) => dom.appendChild(_render(child)));
  return dom;
}
复制代码

```

### 16、判断一个对象有环引用

> 题目描述：验证一个对象有无环引用

```
var obj = {
    a: {
        c: [
            1, 2
        ]
    },
    b: 1
}
obj.a.c.d = obj
console.log(cycleDetector(obj)) // true
复制代码

```

> 实现思路：用一个数组存储每一个遍历过的对象，下次找到数组中存在，则说明环引用

```
function cycleDetector(obj) {
    const arr = [obj]
    let flag = false

    function cycle(o) {
        const keys = Object.keys(o)
        for (const key of keys) {
            const temp = o[key]
            if (typeof temp === 'object' && temp !== null) {
                if (arr.indexOf(temp) >= 0) {
                    flag = true
                    return
                }
                arr.push(temp)
                cycle(temp)
            }
        }
    }

    cycle(obj)

    return flag
}
复制代码

```

### 17、计算一个对象的层数

> 题目描述：给你一个对象，统计一下它的层数

```
const obj = {
    a: { b: [1] },
    c: { d: { e: { f: 1 } } }
}

console.log(loopGetLevel(obj)) // 4
复制代码

```

> 实现如下:

```
function loopGetLevel(obj) {
    var res = 1;

    function computedLevel(obj, level) {
        var level = level ? level : 0;
        if (typeof obj === 'object') {
            for (var key in obj) {
                if (typeof obj[key] === 'object') {
                    computedLevel(obj[key], level + 1);
                } else {
                    res = level + 1 > res ? level + 1 : res;
                }
            }
        } else {
            res = level > res ? level : res;
        }
    }
    computedLevel(obj)

    return res
}
复制代码

```

### 18、对象的扁平化

> 题目描述：

```
const obj = {
  a: {
         b: 1,
         c: 2,
         d: {e: 5}
     },
  b: [1, 3, {a: 2, b: 3}],
  c: 3
 }
 
 flatten(obj) 结果返回如下
 // {
 //  'a.b': 1,
 //  'a.c': 2,
 //  'a.d.e': 5,
 //  'b[0]': 1,
 //  'b[1]': 3,
 //  'b[2].a': 2,
 //  'b[2].b': 3
 //   c: 3
 // }
复制代码

```

> 实现如下：

```
const isObject = (val) =>  typeof val === "object" && val !== null

function flatten(obj) {
  if (!isObject(obj)) return
  const res = {}
  const dfs = (cur, prefix) => {
    if (isObject(cur)) {
      if (Array.isArray(cur)) {
        cur.forEach((item, index) => {
          dfs(item, `${prefix}[${index}]`)
        })
      } else {
        for(let key in cur) {
          dfs(cur[key], `${prefix}${prefix ? '.' : ''}${key}`)
        }
      }
    } else {
      res[prefix] = cur
    }
  }
  dfs(obj, '')
  return res
}

// 测试
console.log(flatten(obj))
复制代码

```

### 19、实现 (a == 1 && a == 2 && a == 3) 为 true

> 题目描述：`实现(a == 1 && a == 2 && a == 3)为true`

```
// 第一种方法
var a = {
  i: 1,
  toString: function () {
    return a.i++;
  }
}
console.log(a == 1 && a == 2 && a == 3) // true

// 第二种方法
var a = [1, 2, 3];
a.join = a.shift;
console.log(a == 1 && a == 2 && a == 3); // true

// 第三种方法
var val = 0;
Object.defineProperty(window, 'a', {
    get: function () {
        return ++val;
    }
});
console.log(a == 1 && a == 2 && a == 3) // true
复制代码

```

### 20、实现限制并发的 Promise 调度器

> 题目描述：JS 实现一个带并发限制的异步调度器 Scheduler，保证同时运行的任务最多有两个

```
addTask(1000,"1");
addTask(500,"2");
addTask(300,"3");
addTask(400,"4");
的输出顺序是：2 3 1 4

整个的完整执行流程：

一开始1、2两个任务开始执行
500ms时，2任务执行完毕，输出2，任务3开始执行
800ms时，3任务执行完毕，输出3，任务4开始执行
1000ms时，1任务执行完毕，输出1，此时只剩下4任务在执行
1200ms时，4任务执行完毕，输出4
复制代码

```

> 实现如下：

```
class Scheduler {
  constructor(limit) {
    this.queue = []
    this.limit = limit
    this.count = 0
  }
  

  add(time, order) {
    const promiseCreator = () => {
      return new Promise((resolve, reject) => {
        setTimeout(() => {
          console.log(order)
          resolve()
        }, time)
      })
    }
    this.queue.push(promiseCreator)
  }

  taskStart() {
    for(let i = 0; i < this.limit; i++) {
      this.request()
    }
  }

  request() {
    if (!this.queue.length || this.count >= this.limit) return
    this.count++
    this.queue.shift()().then(() => {
      this.count--
      this.request()
    })
  }
}

// 测试
const scheduler = new Scheduler(2);
const addTask = (time, order) => {
  scheduler.add(time, order);
};
addTask(1000, "1");
addTask(500, "2");
addTask(300, "3");
addTask(400, "4");
scheduler.taskStart();
复制代码

```

### 21、实现 lazyMan 函数

> 题目描述：

```
实现一个LazyMan，可以按照以下方式调用:
LazyMan(“Hank”)输出:
Hi! This is Hank!

LazyMan(“Hank”).sleep(10).eat(“dinner”)输出
Hi! This is Hank!
//等待10秒..
Wake up after 10
Eat dinner~

LazyMan(“Hank”).eat(“dinner”).eat(“supper”)输出
Hi This is Hank!
Eat dinner~
Eat supper~

LazyMan(“Hank”).eat(“supper”).sleepFirst(5)输出
//等待5秒
Wake up after 5
Hi This is Hank!
Eat supper
复制代码

```

> 实现如下：

```
class _LazyMan {
  constructor(name) {
    this.tasks = []
    const task = () => {
      console.log(`Hi! This is ${name}`)
      this.next()
    }
    this.tasks.push(task)
    setTimeout(() => {
      this.next()
    }, 0)
  }
  next() {
    const task = this.tasks.shift()
    task && task()
  }
  sleep(time) {
    this.sleepWrapper(time, false)
    return this
  }
  sleepFirst(time) {
    this.sleepWrapper(time, true)
    return this
  }
  sleepWrapper(time, first) {
    const task = () => {
      setTimeout(() => {
        console.log(`Wake up after ${time}`)
        this.next()
      }, time * 1000)
    }
    if (first) {
      this.tasks.unshift(task)
    } else {
      this.tasks.push(task)
    }
  }
  eat(food) {
    const task = () => {
      console.log(`Eat ${food}`);
      this.next();
    };
    this.tasks.push(task);
    return this;
  }
}

// 测试
const lazyMan = (name) => new _LazyMan(name)

lazyMan('Hank').sleep(1).eat('dinner')

lazyMan('Hank').eat('dinner').eat('supper')

lazyMan('Hank').eat('supper').sleepFirst(5)
复制代码

```

### 22、实现 add 函数

> 题目描述：实现一个 add 方法 使计算结果能够满足如下预期：
> 
> *   add(1)(2)(3)()=6
> *   add(1,2,3)(4)()=10

```
function add(...args1) {
  let allArgs = [...args1]

  function fn(...args2) {
    if (!args2.length) return fn.toString()
    allArgs = [...allArgs, ...args2]
    return fn
  }

  fn.toString = function () {
    return allArgs.reduce((pre, next) => pre + next)
  }

  return fn
}

// 测试
console.log(add(1)(2)(3)())
console.log(add(1, 2)(3)())
复制代码

```

### 23、实现一个合格的深拷贝

推荐看我这篇：[深拷贝有这 5 个段位，你只是青铜段位？还想涨薪？](https://juejin.cn/post/7017991655009566728 "https://juejin.cn/post/7017991655009566728")

### 24、实现 Promise

推荐看我这篇：[看了就会，手写 Promise 原理，最通俗易懂的版本！！！](https://juejin.cn/post/6994594642280857630 "https://juejin.cn/post/6994594642280857630")【阅读：1.3w，点赞：460】

### 25、实现 async/await

推荐看我这篇：[7 张图，20 分钟就能搞定的 async/await 原理！为什么要拖那么久？](https://juejin.cn/post/7007031572238958629 "https://juejin.cn/post/7007031572238958629")【阅读：2.15w，点赞：460】

Array 篇
-------

定义一个测试数组

```
const players = [
    { name: '科比', num: 24 },
    { name: '詹姆斯', num: 23 },
    { name: '保罗', num: 3 },
    { name: '威少', num: 0 },
    { name: '杜兰特', num: 35 }
]
复制代码

```

### 26、forEach

参数代表含义

*   item：遍历项
*   index：遍历项的索引
*   arr：数组本身

```
Array.prototype.sx_forEach = function (callback) {
    for (let i = 0; i < this.length; i++) {
        callback(this[i], i, this)
    }
}

players.sx_forEach((item, index, arr) => {
    console.log(item, index)
})
// { name: '科比', num: 24 } 0
// { name: '詹姆斯', num: 23 } 1
// { name: '保罗', num: 3 } 2
// { name: '威少', num: 0 } 3
// { name: '杜兰特', num: 35 } 4
复制代码

```

### 27、map

参数代表含义

*   item：遍历项
*   index：遍历项的索引
*   arr：数组本身

```
Array.prototype.sx_map = function (callback) {
    const res = []
    for (let i = 0; i < this.length; i++) {
        res.push(callback(this[i], i, this))
    }
    return res
}

console.log(players.sx_map((item, index) => `${item.name}--${item.num}--${index}`))
// [ '科比--24--0', '詹姆斯--23--1', '保罗--3--2', '威少--0--3', '杜兰特--35--4' ]
复制代码

```

### 28、filter

参数代表含义

*   item：遍历项
*   index：遍历项的索引
*   arr：数组本身

```
Array.prototype.sx_filter = function (callback) {
    const res = []
    for (let i = 0; i < this.length; i++) {
        callback(this[i], i, this) && res.push(this[i])
    }
    return res
}

console.log(players.sx_filter(item => item.num >= 23))
// [
//     { name: '科比', num: 24 },
//     { name: '詹姆斯', num: 23 },
//     { name: '杜兰特', num: 35 }
// ]
复制代码

```

### 29、every

参数代表含义

*   item：遍历项
*   index：遍历项的索引
*   arr：数组本身

```
Array.prototype.sx_every = function (callback) {
    let flag = true
    for (let i = 0; i < this.length; i++) {
        flag = callback(this[i], i, this)
        if (!flag) break
    }

    return flag
}

console.log(players.sx_every(item => item.num >= 23)) // false
console.log(players.sx_every(item => item.num >= 0)) // true
复制代码

```

### 30、some

参数代表含义

*   item：遍历项
*   index：遍历项的索引
*   arr：数组本身

```
Array.prototype.sx_some = function (callback) {
    let flag = false
    for (let i = 0; i < this.length; i++) {
        flag = callback(this[i], i, this)
        if (flag) break
    }

    return flag
}

console.log(players.sx_some(item => item.num >= 23)) // true
console.log(players.sx_some(item => item.num >= 50)) // false
复制代码

```

### 31、reduce

参数代表含义

*   pre：前一项
*   next：下一项
*   index：当前索引
*   arr：数组本身

```
Array.prototype.sx_reduce = function (callback, ...args) {
  let start = 0, pre
  if (args.length) {
      pre = args[0]
  } else {
      pre = this[0]
      start = 1
  }
  for (let i = start; i < this.length; i++) {
      pre = callback(pre, this[i], i, this)
  }
  return pre
}

// 计算所有num相加
const sum = players.sx_reduce((pre, next) => {
    return pre + next.num
}, 0)
console.log(sum) // 85
复制代码

```

### 32、findIndex

参数代表含义

*   item：遍历项
*   index：遍历项的索引
*   arr：数组本身

```
Array.prototype.sx_findIndex = function (callback) {
    for (let i = 0; i < this.length; i++) {
        if (callback(this[i], i, this)) {
            return i
        }
    }
    return -1
}

console.log(players.sx_findIndex(item => item.name === '科比')) // 0
console.log(players.sx_findIndex(item => item.name === '安东尼')) // -1
复制代码

```

### 33、find

参数代表含义

*   item：遍历项
*   index：遍历项的索引
*   arr：数组本身

```
Array.prototype.sx_find = function (callback) {
    for (let i = 0; i < this.length; i++) {
        if (callback(this[i], i, this)) {
            return this[i]
        }
    }
    return undefined
}

console.log(players.sx_find(item => item.name === '科比')) // { name: '科比', num: 24 }
console.log(players.sx_find(item => item.name === '安东尼')) // undefined
复制代码

```

### 34、fill

用处：填充数组

参数代表含义

*   initValue：填充的值
*   start：开始填充索引，默认 0
*   end：结束填充索引，默认 length

```
Array.prototype.sx_fill = function (value, start = 0, end) {
  end = end || this.length
  for (let i = start; i < end; i++) {
      this[i] = value
  }
  return this
}

console.log(players.sx_fill('林三心', 1, 3))
// [
//     { name: '科比', num: 24 },
//     '林三心',
//     '林三心',
//     '林三心',
//     { name: '杜兰特', num: 35 }
// ]
复制代码

```

### 35、includes

用处：查找元素，查到返回`true`，反之返回`false`，可查找`NaN`

```
Array.prototype.sx_includes = function (value, start = 0) {
    if (start < 0) start = this.length + start
    const isNaN = Number.isNaN(value)
    for (let i = start; i < this.length; i++) {
        if (this[i] === value || Number.isNaN(this[i]) === isNaN) {
            return true
        }
    }
    return false
}

console.log([1, 2, 3].sx_includes(2)) // true
console.log([1, 2, 3, NaN].sx_includes(NaN)) // true
console.log([1, 2, 3].sx_includes(1, 1)) // false
复制代码

```

### 36、join

用处：将数组用分隔符拼成字符串，分隔符默认为`,`

```
Array.prototype.sx_join = function (s = ',') {
    let str = ''
    for(let i = 0; i < this.length; i++) {
        str = i === 0 ? `${str}${this[i]}` : `${str}${s}${this[i]}`
    }
    return str
}

console.log([1, 2, 3].sx_join()) // 1,2,3
console.log([1, 2, 3].sx_join('*')) // 1*2*3
复制代码

```

### 37、flat

```
Array.prototype.sx_flat = function () {
    let arr = this
    while (arr.some(item => Array.isArray(item))) {
        arr = [].concat(...arr)
    }
    return arr
}

const testArr = [1, [2, 3, [4, 5]], [8, 9]]

console.log(testArr.sx_flat())
// [1, 2, 3, 4, 5, 8, 9]
复制代码

```

### 38、splice

难点

*   截取长度和替换长度的比较，不同情况

```
Array.prototype.sx_splice = function (start, length, ...values) {
  if (length === 0) return []
  length = start + length > this.length - 1 ? this.length - start : length
  console.log(length)
  const res = [], tempArr = [...this]
  for (let i = start; i < start + values.length; i++) {
    this[i] = values[i - start]
  }
  this.length = start + values.length
  if (values.length < length) {
    const cha = length - values.length
    console.log(cha)
    for (let i = start + values.length; i < tempArr.length; i++) {
      this[i] = tempArr[i + cha]
    }
    this.length = this.length - cha 
  }
  if (values.length > length) {
    for (let i = start + length; i < tempArr.length; i++) {
      this.push(tempArr[i])
    }
  }
  for (let i = start; i < start + length; i++) {
    res.push(tempArr[i])
  }
  return res
}
复制代码

```

Object 篇
--------

定义一个测试对象

```
const obj = {
    name: '林三心',
    age: 22,
    gender: '男'
}
复制代码

```

### 39、entries

用处：将对象转成键值对数组

```
Object.prototype.sx_entries = function (obj) {
    const res = []
    for (let key in obj) {
        obj.hasOwnProperty(key) && res.push([key, obj[key]])
    }
    return res
}

console.log(Object.sx_entries(obj))
// [ [ 'name', '林三心' ], [ 'age', 22 ], [ 'gender', '男' ] ]
复制代码

```

### 40、fromEntries

用处：跟`entries`相反，将键值对数组转成对象

```
Object.prototype.sx_fromEntries = function (arr) {
    const obj = {}
    for (let i = 0; i < arr.length; i++) {
        const [key, value] = arr[i]
        obj[key] = value
    }
    return obj
}

console.log(Object.sx_fromEntries([['name', '林三心'], ['age', 22], ['gender', '男']]))
// { name: '林三心', age: 22, gender: '男' }
复制代码

```

### 41、keys

用处：将对象的 key 转成一个数组合集

```
Object.prototype.sx_keys = function (obj) {
    const keys = []
    for (let key in obj) {
        obj.hasOwnProperty(key) && res.push(key)
    }
    return keys
}

console.log(Object.keys(obj))
// [ 'name', 'age', 'gender' ]
复制代码

```

### 42、values

用处：将对象的所有值转成数组合集

```
Object.prototype.sx_values = function (obj) {
    const values = []
    for (let key in obj) {
        obj.hasOwnProperty(key) && values.push(obj[key])
    }
    return values
}

console.log(Object.sx_values(obj))
// [ '林三心', 22, '男' ]
复制代码

```

### 43、instanceOf

用处：A instanceOf B，判断 A 是否经过 B 的原型链

```
function instanceOf(father, child) {
    const fp = father.prototype
    var cp = child.__proto__

    while (cp) {
        if (cp === fp) {
            return true
        }
        cp = cp.__proto__
    }

    return false
}

function Person(name) {
    this.name = name
}
const sx = new Person('林三心')

console.log(instanceOf(Person, sx)) // true
console.log(instanceOf(Person, sx2)) // false
复制代码

```

### 44、is

用处：Object.is(a, b)，判断 a 是否等于 b

```
Object.prototype.sx_is = function (x, y) {
    if (x === y) {
        // 防止 -0 和 +0
        return x !== 0 || 1 / x === 1 / y
    }

    // 防止NaN
    return x !== x && y !== y
}

const a = { name: '林三心' }
const b = a
const c = { name: '林三心' }

console.log(Object.sx_is(a, b)) // true
console.log(Object.sx_is(a, c)) // false
复制代码

```

### 45、Object.assign

难点

*   assign 接收多个对象，并将多个对象合成一个对象
*   这些对象如果有重名属性，以后来的对象属性值为准
*   assign 返回一个对象，`这个对象 === 第一个对象`

```
Object.prototype.sx_assign = function (target, ...args) {
    if (target === null || target === undefined) {
        throw new TypeError('Cannot convert undefined or null to object')
    }
    target = Object(target)

    for (let nextObj of args) {
        for (let key in nextObj) {
            nextObj.hasOwnProperty(key) && (target[key] = nextObj[key])
        }
    }
    return target
}

const testa = { name: '林三心' }
const testb = { name: 'sunshine_lin', age: 22 }
const testc = { age: 18, gender: '男' }

const testd = Object.sx_assign(testa, testb, testc)
console.log(testd) // { name: 'sunshine_lin', age: 18, gender: '男' }
console.log(testa === testd) // true
复制代码

```

Function 篇
----------

### 46、call

```
Function.prototype.sx_call = function (obj, ...args) {
    obj = obj || window

    // Symbol是唯一的，防止重名key
    const fn = Symbol()
    obj[fn] = this

    // 执行，返回执行值
    return obj[fn](...args)
}

const testobj = {
    name: '林三心',
    testFn(age) {
        console.log(`${this.name}${age}岁了`)
    }
}
const testobj2 = {
    name: 'sunshine_lin'
}

testobj.testFn.sx_call(testobj2, 22) // sunshine_lin22岁了
复制代码

```

### 47、apply

```
Function.prototype.sx_apply = function (obj, args) {
    obj = obj || window

    // Symbol是唯一的，防止重名key
    const fn = Symbol()
    obj[fn] = this

    // 执行，返回执行值
    return obj[fn](...args)
}

const testobj = {
    name: '林三心',
    testFn(age) {
        console.log(`${this.name}${age}岁了`)
    }
}
const testobj2 = {
    name: 'sunshine_lin'
}

testobj.testFn.sx_apply(testobj2, [22]) // sunshine_lin22岁了
复制代码

```

### 48、Function.prototype.bind

难点：

*   bind 是返回一个函数，而不是执行结果
*   bind 返回的函数，拿来当做构造函数，该怎么处理

```
Function.prototype.sx_bind = function (obj, ...args) {
    obj = obj || window

    // Symbol是唯一的，防止重名key
    const fn = Symbol()
    obj[fn] = this
    const _this = this

    const res = function (...innerArgs) {
        console.log(this, _this)
        if (this instanceof _this) {
            this[fn] = _this
            this[fn](...[...args, ...innerArgs])
            delete this[fn]
        } else {
            obj[fn](...[...args, ...innerArgs])
            delete obj[fn]
        }
    }
    res.prototype = Object.create(this.prototype)
    return res
}
复制代码

```

String 篇
--------

### 49、slice

参数代表含义

*   start：开始截取的字符索引 (包含此字符)
*   end：结束截取的字符索引 (不包含此字符)

注意点

*   start > end：返回空字符串
*   start < 0：`start = 数组长度 + start`

```
String.prototype.sx_slice = function (start = 0, end) {
    start = start < 0 ? this.length + start : start
    end = !end && end !== 0 ? this.length : end

    if (start >= end) return ''
    let str = ''
    for (let i = start; i < end; i++) {
        str += this[i]
    }

    return str
}

console.log(str.sx_slice(2)) // nshine_lin
console.log(str.sx_slice(-2)) // in
console.log(str.sx_slice(-9, 10)) // shine_l
console.log(str.sx_slice(5, 1)) // ''
复制代码

```

### 50、substr

参数代表含义

*   start：开始截取的字符索引 (包含此字符)
*   length：截取的长度

注意点

*   start < 0：`start = 数组长度 + start`
*   length 超出所能截取范围，需要做处理
*   length < 0：返回空字符串

```
String.prototype.sx_substr = function (start = 0, length) {
    if (length < 0) return ''

    start = start < 0 ? this.length + start : start
    length = (!length && length !== 0) || length > this.length - start ? this.length : start + length

    let str = ''
    for (let i = start; i < length; i++) {
        str += this[i]
    }
    return str
}

console.log(str.sx_substr(3)) // shine_lin
console.log(str.sx_substr(3, 3)) // shi
console.log(str.sx_substr(5, 300)) // ine_lin
复制代码

```

### 51、substring

功能与`slice`大致相同

区别之处

*   start > end：互换值

```
String.prototype.sx_sunstring = function (start = 0, end) {
    start = start < 0 ? this.length + start : start
    end = !end && end !== 0 ? this.length : end

    if (start >= end) [start, end] = [end, start]
    let str = ''
    for (let i = start; i < end; i++) {
        str += this[i]
    }

    return str
}

console.log(str.sx_sunstring(2)) // nshine_lin
console.log(str.sx_sunstring(-2)) // in
console.log(str.sx_sunstring(-9, 10)) // shine_l
console.log(str.sx_sunstring(5, 1)) // unsh
复制代码

```

Promise 篇
---------

### 52、all

*   接收一个 Promise 数组，数组中如有非 Promise 项，则此项当做成功
*   如果所有 Promise 都成功，则返回成功结果数组
*   如果有一个 Promise 失败，则返回这个失败结果

```
    function all(promises) {
        const result = []
        let count = 0
        return new MyPromise((resolve, reject) => {
            const addData = (index, value) => {
                result[index] = value
                count++
                if (count === promises.length) resolve(result)
            }
            promises.forEach((promise, index) => {
                if (promise instanceof MyPromise) {
                    promise.then(res => {
                        addData(index, res)
                    }, err => reject(err))
                } else {
                    addData(index, promise)
                }
            })
        })
    }
复制代码

```

### 53、race

*   接收一个 Promise 数组，数组中如有非 Promise 项，则此项当做成功
*   哪个 Promise 最快得到结果，就返回那个结果，无论成功失败

```
    function race(promises) {
        return new MyPromise((resolve, reject) => {
            promises.forEach(promise => {
                if (promise instanceof MyPromise) {
                    promise.then(res => {
                        resolve(res)
                    }, err => {
                        reject(err)
                    })
                } else {
                    resolve(promise)
                }
            })
        })
    }
复制代码

```

### 54、allSettled

*   接收一个 Promise 数组，数组中如有非 Promise 项，则此项当做成功
*   把每一个 Promise 的结果，集合成数组，返回

```
    function allSettled(promises) {
        return new Promise((resolve, reject) => {
            const res = []
            let count = 0
            const addData = (status, value, i) => {
                res[i] = {
                    status,
                    value
                }
                count++
                if (count === promises.length) {
                    resolve(res)
                }
            }
            promises.forEach((promise, i) => {
                if (promise instanceof MyPromise) {
                    promise.then(res => {
                        addData('fulfilled', res, i)
                    }, err => {
                        addData('rejected', err, i)
                    })
                } else {
                    addData('fulfilled', promise, i)
                }
            })
        })
    }
复制代码

```

### 55、any

any 与 all 相反

*   接收一个 Promise 数组，数组中如有非 Promise 项，则此项当做成功
*   如果有一个 Promise 成功，则返回这个成功结果
*   如果所有 Promise 都失败，则报错

```
    function any(promises) {
        return new Promise((resolve, reject) => {
            let count = 0
            promises.forEach((promise) => {
                promise.then(val => {
                    resolve(val)
                }, err => {
                    count++
                    if (count === promises.length) {
                        reject(new AggregateError('All promises were rejected'))
                    }
                })
            })
        })
    }
}
复制代码

```

### 56、finally

*   接收一个回调函数，但无参数接收
*   无论成功失败状态，都会执行 finally

```
Promise.prototype.finally = function(callback) {
  return this.then(res => {
    callback()
    return res
  }, err => {
    callback()
    throw err
  })
}
复制代码

```

结语
--

如果你觉得此文对你有一丁点帮助，点个赞，鼓励一下林三心哈哈。

**如果你想一起学习前端或者摸鱼，那你可以加我，加入我的摸鱼学习群，点击这里** ---> [摸鱼沸点](https://juejin.cn/pin/7035153948126216206 "https://juejin.cn/pin/7035153948126216206")

**如果你是有其他目的的，别加我，我不想跟你交朋友，我只想简简单单学习前端，不想搞一些有的没的！！！**
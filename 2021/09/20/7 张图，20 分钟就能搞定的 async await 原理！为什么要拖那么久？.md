> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7007031572238958629)*   这个东西有什么用？
*   这个东西是怎么用的？

### 有什么用？

`async/await`的用处就是：**用同步方式，执行异步操作**，怎么说呢？举个例子

比如我现在有一个需求：先请求完`接口1`，再去请求`接口2`，我们通常会这么做

```
function request(num) { // 模拟接口请求
  return new Promise(resolve => {
    setTimeout(() => {
      resolve(num * 2)
    }, 1000)
  })
}

request(1).then(res1 => {
  console.log(res1) // 1秒后 输出 2

  request(2).then(res2 => {
    console.log(res2) // 2秒后 输出 4
  })
})
复制代码
```

或者我现在又有一个需求：先请求完`接口1`，再拿`接口1`返回的数据，去当做`接口2`的请求参数，那我们也可以这么做

```
request(5).then(res1 => {
  console.log(res1) // 1秒后 输出 10

  request(res1).then(res2 => {
    console.log(res2) // 2秒后 输出 20
  })
})
复制代码
```

其实这么做是没问题的，但是如果嵌套的多了，不免有点不雅观，这个时候就可以用`async/await`来解决了

```
async function fn () {
  const res1 = await request(5)
  const res2 = await request(res1)
  console.log(res2) // 2秒后输出 20
}
fn()
复制代码
```

### 是怎么用？

还是用刚刚的例子

需求一：

```
async function fn () {
  await request(1)
  await request(2)
  // 2秒后执行完
}
fn()
复制代码
```

需求二：

```
async function fn () {
  const res1 = await request(5)
  const res2 = await request(res1)
  console.log(res2) // 2秒后输出 20
}
fn()
复制代码
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c63ae650b71144e2a1ba8dc2bdaceed3~tplv-k3u1fbpfcp-watermark.awebp)

其实就类似于生活中的`排队`，咱们生活中排队买东西，肯定是要上一个人买完，才轮到下一个人。而上面也一样，在`async`函数中，`await`规定了异步操作只能一个一个排队执行，从而达到**用同步方式，执行异步操作**的效果，这里注意了：**await 只能在 async 函数中使用，不然会报错哦**

刚刚上面的例子`await`后面都是跟着异步操作`Promise`，那如果不接`Promise`会怎么样呢？

```
function request(num) { // 去掉Promise
  setTimeout(() => {
    console.log(num * 2)
  }, 1000)
}

async function fn() {
  await request(1) // 2
  await request(2) // 4
  // 1秒后执行完  同时输出
}
fn()
复制代码
```

可以看出，如果`await`后面接的不是`Promise`的话，其实是达不到`排队`的效果的

说完`await`，咱们聊聊`async`吧，`async`是一个位于 function 之前的前缀，只有`async函数`中，才能使用`await`。那`async`执行完是返回一个什么东西呢？

```
async function fn () {}
console.log(fn) // [AsyncFunction: fn]
console.log(fn()) // Promise {<fulfilled>: undefined}
复制代码
```

可以看出，`async函数`执行完会自动返回一个状态为`fulfilled`的 Promise，也就是成功状态，但是值却是 undefined，那要怎么才能使值不是 undefined 呢？很简单，函数有`return`返回值就行了

```
async function fn (num) {
  return num
}
console.log(fn) // [AsyncFunction: fn]
console.log(fn(10)) // Promise {<fulfilled>: 10}
fn(10).then(res => console.log(res)) // 10
复制代码
```

可以看出，此时就有值了，并且还能使用`then方法`进行输出

### 总结

总结一下`async/await`的知识点

*   await 只能在 async 函数中使用，不然会报错
*   async 函数返回的是一个状态为 fuifilled 的 Promise 对象，有无值看有无 return 值
*   await 后面只有接了 Promise 才能实现排队效果
*   async/await 作用是**用同步方式，执行异步操作**

什么是语法糖？
-------

前面说了，`async/await`是一种`语法糖`，诶！好多同学就会问，啥是`语法糖`呢？我个人理解就是，`语法糖`就是一个东西，这个东西你就算不用他，你用其他手段也能达到这个东西同样的效果，但是可能就没有这个东西这么方便了。

*   举个生活中的例子吧：你走路也能走到北京，但是你坐飞机会更快到北京。
*   举个代码中的例子吧：ES6 的`class`也是语法糖，因为其实用普通`function`也能实现同样效果

回归正题，`async/await`是一种`语法糖`，那就说明用其他方式其实也可以实现他的效果，我们今天就是讲一讲怎么去实现`async/await`，用到的是 ES6 里的`迭代函数——generator函数`

generator 函数
------------

### 基本用法

`generator函数`跟普通函数在写法上的区别就是，多了一个星号`*`，并且只有在`generator函数`中才能使用`yield`，什么是`yield`呢，他相当于`generator函数`执行的`中途暂停点`，比如下方有 3 个暂停点。而怎么才能暂停后继续走呢？那就得使用到`next方法`，`next方法`执行后会返回一个对象，对象中有`value 和 done`两个属性

*   value：暂停点后面接的值，也就是 yield 后面接的值
*   done：是否 generator 函数已走完，没走完为 false，走完为 true

```
function* gen() {
  yield 1
  yield 2
  yield 3
}
const g = gen()
console.log(g.next()) // { value: 1, done: false }
console.log(g.next()) // { value: 2, done: false }
console.log(g.next()) // { value: 3, done: false }
console.log(g.next()) // { value: undefined, done: true }
复制代码
```

可以看到最后一个是 undefined，这取决于你 generator 函数有无返回值

```
function* gen() {
  yield 1
  yield 2
  yield 3
  return 4
}
const g = gen()
console.log(g.next()) // { value: 1, done: false }
console.log(g.next()) // { value: 2, done: false }
console.log(g.next()) // { value: 3, done: false }
console.log(g.next()) // { value: 4, done: true }
复制代码
```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d56e8dea0f204cc18a2d86d1ffcf51ef~tplv-k3u1fbpfcp-watermark.awebp)

### yield 后面接函数

yield 后面接函数的话，到了对应暂停点 yield，会马上执行此函数，并且该函数的执行返回值，会被当做此暂停点对象的`value`

```
function fn(num) {
  console.log(num)
  return num
}
function* gen() {
  yield fn(1)
  yield fn(2)
  return 3
}
const g = gen()
console.log(g.next()) 
// 1
// { value: 1, done: false }
console.log(g.next())
// 2
//  { value: 2, done: false }
console.log(g.next()) 
// { value: 3, done: true }
复制代码
```

### yield 后面接 Promise

前面说了，函数执行返回值会当做暂停点对象的 value 值，那么下面例子就可以理解了，前两个的 value 都是 pending 状态的 Promise 对象

```
function fn(num) {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve(num)
    }, 1000)
  })
}
function* gen() {
  yield fn(1)
  yield fn(2)
  return 3
}
const g = gen()
console.log(g.next()) // { value: Promise { <pending> }, done: false }
console.log(g.next()) // { value: Promise { <pending> }, done: false }
console.log(g.next()) // { value: 3, done: true }
复制代码
```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39a4a3ec5ecc4e9e88a9515be0bc7bb1~tplv-k3u1fbpfcp-watermark.awebp)

其实我们想要的结果是，两个 Promise 的结果`1 和 2`，那怎么做呢？很简单，使用 Promise 的 then 方法就行了

```
const g = gen()
const next1 = g.next()
next1.value.then(res1 => {
  console.log(next1) // 1秒后输出 { value: Promise { 1 }, done: false }
  console.log(res1) // 1秒后输出 1

  const next2 = g.next()
  next2.value.then(res2 => {
    console.log(next2) // 2秒后输出 { value: Promise { 2 }, done: false }
    console.log(res2) // 2秒后输出 2
    console.log(g.next()) // 2秒后输出 { value: 3, done: true }
  })
})
复制代码
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/56134335085443e9a86da9c3a9c139b6~tplv-k3u1fbpfcp-watermark.awebp)

### next 函数传参

generator 函数可以用`next方法`来传参，并且可以通过`yield`来接收这个参数，注意两点

*   第一次 next 传参是没用的，只有从第二次开始 next 传参才有用
*   next 传值时，要记住顺序是，先右边 yield，后左边接收参数

```
function* gen() {
  const num1 = yield 1
  console.log(num1)
  const num2 = yield 2
  console.log(num2)
  return 3
}
const g = gen()
console.log(g.next()) // { value: 1, done: false }
console.log(g.next(11111))
// 11111
//  { value: 2, done: false }
console.log(g.next(22222)) 
// 22222
// { value: 3, done: true }
复制代码
```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c49ec193e19249d2876fba7909f89acc~tplv-k3u1fbpfcp-watermark.awebp)

### Promise+next 传参

前面讲了

*   yield 后面接 Promise
*   next 函数传参

那这两个组合起来会是什么样呢？

```
function fn(nums) {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve(nums * 2)
    }, 1000)
  })
}
function* gen() {
  const num1 = yield fn(1)
  const num2 = yield fn(num1)
  const num3 = yield fn(num2)
  return num3
}
const g = gen()
const next1 = g.next()
next1.value.then(res1 => {
  console.log(next1) // 1秒后同时输出 { value: Promise { 2 }, done: false }
  console.log(res1) // 1秒后同时输出 2

  const next2 = g.next(res1) // 传入上次的res1
  next2.value.then(res2 => {
    console.log(next2) // 2秒后同时输出 { value: Promise { 4 }, done: false }
    console.log(res2) // 2秒后同时输出 4

    const next3 = g.next(res2) // 传入上次的res2
    next3.value.then(res3 => {
      console.log(next3) // 3秒后同时输出 { value: Promise { 8 }, done: false }
      console.log(res3) // 3秒后同时输出 8

       // 传入上次的res3
      console.log(g.next(res3)) // 3秒后同时输出 { value: 8, done: true }
    })
  })
})
复制代码
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8db7c759079a404ebab41b9aacc90c77~tplv-k3u1fbpfcp-watermark.awebp)

实现 async/await
--------------

其实上方的`generator函数`的`Promise+next传参`，就很像`async/await`了，区别在于

*   gen 函数执行返回值不是 Promise，asyncFn 执行返回值是 Promise
*   gen 函数需要执行相应的操作，才能等同于 asyncFn 的排队效果
*   gen 函数执行的操作是不完善的，因为并不确定有几个 yield，不确定会嵌套几次

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2465072c79684ecabd9b0377e22b05f8~tplv-k3u1fbpfcp-watermark.awebp)

那我们怎么办呢？我们可以封装一个高阶函数。什么是`高阶函数`呢？`高阶函数`的特点是：**参数是函数，返回值也是函数**。下方的`highorderFn`就是一个`高阶函数`

```
function highorderFn(函数) {
    // 一系列处理
    
    return 函数
}
复制代码
```

我们可以封装一个高阶函数，接收一个 generator 函数，并经过一系列处理，返回一个具有 async 函数功能的函数

```
function generatorToAsync(generatorFn) {
  // 经过一系列处理
  
  return 具有async函数功能的函数
}
复制代码
```

### 返回值 Promise

之前我们说到，async 函数的执行返回值是一个 Promise，那我们要怎么实现相同的结果呢

```
function* gen() {

}

const asyncFn = generatorToAsync(gen)

console.log(asyncFn()) // 期望这里输出 Promise
复制代码
```

其实很简单，`generatorToAsync函数`里做一下处理就行了

```
function* gen() {

}
function generatorToAsync (generatorFn) {
  return function () {
    return new Promise((resolve, reject) => {

    })
  }
}

const asyncFn = generatorToAsync(gen)

console.log(asyncFn()) // Promise
复制代码
```

### 加入一系列操作

咱们把之前的处理代码，加入`generatorToAsync函数`中

```
function fn(nums) {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve(nums * 2)
    }, 1000)
  })
}
function* gen() {
  const num1 = yield fn(1)
  const num2 = yield fn(num1)
  const num3 = yield fn(num2)
  return num3
}
function generatorToAsync(generatorFn) {
  return function () {
    return new Promise((resolve, reject) => {
      const g = generatorFn()
      const next1 = g.next()
      next1.value.then(res1 => {

        const next2 = g.next(res1) // 传入上次的res1
        next2.value.then(res2 => {

          const next3 = g.next(res2) // 传入上次的res2
          next3.value.then(res3 => {

            // 传入上次的res3
            resolve(g.next(res3).value)
          })
        })
      })
    })
  }
}

const asyncFn = generatorToAsync(gen)

asyncFn().then(res => console.log(res)) // 3秒后输出 8
复制代码
```

可以发现，咱们其实已经实现了以下的`async/await`的结果了

```
async function asyncFn() {
  const num1 = await fn(1)
  const num2 = await fn(num1)
  const num3 = await fn(num2)
  return num3
}
asyncFn().then(res => console.log(res)) // 3秒后输出 8
复制代码
```

### 完善代码

上面的代码其实都是死代码，因为一个 async 函数中可能有 2 个 await，3 个 await，5 个 await ，其实 await 的个数是不确定的。同样类比，generator 函数中，也可能有 2 个 yield，3 个 yield，5 个 yield，所以咱们得把代码写成活的才行

```
function generatorToAsync(generatorFn) {
  return function() {
    const gen = generatorFn.apply(this, arguments) // gen有可能传参

    // 返回一个Promise
    return new Promise((resolve, reject) => {

      function go(key, arg) {
        let res
        try {
          res = gen[key](arg) // 这里有可能会执行返回reject状态的Promise
        } catch (error) {
          return reject(error) // 报错的话会走catch，直接reject
        }

        // 解构获得value和done
        const { value, done } = res
        if (done) {
          // 如果done为true，说明走完了，进行resolve(value)
          return resolve(value)
        } else {
          // 如果done为false，说明没走完，还得继续走

          // value有可能是：常量，Promise，Promise有可能是成功或者失败
          return Promise.resolve(value).then(val => go('next', val), err => go('throw', err))
        }
      }

      go("next") // 第一次执行
    })
  }
}

const asyncFn = generatorToAsync(gen)

asyncFn().then(res => console.log(res))
复制代码
```

这样的话，无论是多少个 yield 都会排队执行了，咱们把代码写成活的了

### 示例

`async/await`版本

```
async function asyncFn() {
  const num1 = await fn(1)
  console.log(num1) // 2
  const num2 = await fn(num1)
  console.log(num2) // 4
  const num3 = await fn(num2)
  console.log(num3) // 8
  return num3
}
const asyncRes = asyncFn()
console.log(asyncRes) // Promise
asyncRes.then(res => console.log(res)) // 8
复制代码
```

使用`generatorToAsync函数`的版本

```
function* gen() {
  const num1 = yield fn(1)
  console.log(num1) // 2
  const num2 = yield fn(num1)
  console.log(num2) // 4
  const num3 = yield fn(num2)
  console.log(num3) // 8
  return num3
}

const genToAsync = generatorToAsync(gen)
const asyncRes = genToAsync()
console.log(asyncRes) // Promise
asyncRes.then(res => console.log(res)) // 8
复制代码
```

结语
--

如果你觉得此文对你有一丁点帮助，点个赞，鼓励一下林三心哈哈。或者可以加入我的摸鱼群 想进学习群，摸鱼群，请点击这里[摸鱼](https://juejin.cn/pin/6969565162885873701 "https://juejin.cn/pin/6969565162885873701")，我会定时直播模拟面试，答疑解惑

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/53480cd479cc4a3e9393bce93e9f2c23~tplv-k3u1fbpfcp-watermark.awebp)

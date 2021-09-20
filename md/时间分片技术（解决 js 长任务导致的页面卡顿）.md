> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7008416027700789255) 1. Promise.then 2. Object.observe 3. MutaionObserver 复制代码

宏任务：

```
1. script(整体代码)
2. setTimeout
3. setInterval
4. I/O
5. UI交互事件
6. postMessage
7. MessageChannel
复制代码
```

浏览器渲染时机
-------

除去特殊情况，页面的渲染会在微任务队列清空后，宏任务执行前，所以我们可以让推入主执行栈的函数执行到一定时间就去休眠，然后在渲染之后的宏任务里面叫醒他，这样渲染或者用户交互都不会卡顿了！

原始代码
----

我们先模拟一个 js 长任务

### 代码

```
// style
@keyframes move {
    from {
        left: 0;
    }
    to {
        left: 100%;
    }
}
.move {
    position: absolute;
    animation: move 5s linear infinite;
}

// dom
<div class="move">123123123</div>

// script
function fnc () {
    let i = 0
    const start = performance.now()
    while (performance.now() - start <= 5000) {
        i++
    }

    return i
}

setTimeout(() => {
    fnc()
}, 1000)
复制代码
```

### 效果

如下图，动画运行 1s 的时候，js 函数开始运行，动画会先停止渲染，然后等 js 主执行栈空闲之后动画才继续进行。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1c76dc0555964482928d952e879256c7~tplv-k3u1fbpfcp-watermark.awebp?)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8739ac8e7e2d488ca81c543d95663f53~tplv-k3u1fbpfcp-watermark.awebp?)

函数改造
----

我们把原来的函数改造为 generator 函数

### 代码

```
// generator 处理原来的函数
function * fnc_ () {
    let i = 0
    const start = performance.now()
    while (performance.now() - start <= 5000) {
        yield i++
    }

    return i
}

// 简易时间分片
function timeSlice (fnc) {
    if(fnc.constructor.name !== 'GeneratorFunction') return fnc()

    return async function (...args) {
        const fnc_ = fnc(...args)
        let data

        do {
            data = fnc_.next()
            // 每执行一步就休眠，注册一个宏任务 setTimeout 来叫醒他
            await new Promise( resolve => setTimeout(resolve))
        } while (!data.done)

        return data.value
    }
}

setTimeout(async () => {
    const fnc = timeSlice(fnc_)
    const start = performance.now()
    console.log('开始')
    const num = await fnc()
    console.log('结束', `${(performance.now() - start)/ 1000}s`)
    console.log(num)
}, 1000)
复制代码
```

### 效果

动画根本不受影响，fps 一直很稳定，因为我们把耗时任务拆成很多个块来执行。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/84b4dce997ce43aeaad5f7167709982e~tplv-k3u1fbpfcp-watermark.awebp?)

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bd2ab5a54958469da1c782b4384fcddb~tplv-k3u1fbpfcp-watermark.awebp?)

优化时间分片
------

上面的时间分片函数每执行一步，就会休眠，然后通过一个宏任务来唤醒他，但是这样的执行效率肯定是比较低的，我们再优化一下执行的效率，提升连续执行时间。

### 代码

```
// 精准时间分片
function timeSlice_ (fnc, time = 25) {
    if(fnc.constructor.name !== 'GeneratorFunction') return fnc()

    return function (...args) {
        const fnc_ = fnc(...args)

        function go () {
            const start = performance.now()
            let data

            do {
                data = fnc_.next()
            } while (!data.done && performance.now() - start < time)

            if (data.done) return data.value

            return new Promise((resolve, reject) => {
                setTimeout(() => {
                    try {
                        resolve(go())
                    } catch(e) {
                        reject(e)
                    }
                })
            })
        }

        return go()
    }
}

setTimeout(async () => {
    const fnc1 = timeSlice_(fnc_)
    let start = performance.now()

    console.log('开始')
    const num = await fnc1()
    console.log('结束', `${(performance.now() - start)/ 1000}s`)
    console.log(num)
}, 1000);
复制代码
```

### 效果

我们把函数分成了较大的块，这样函数执行的效率就会变高，fps 会稍微收到影响，但是在接受范围内。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/59c4040b0838418d8d17ba036c7d2fde~tplv-k3u1fbpfcp-watermark.awebp?)

对比优化前后
------

我们对比一下优化时间分片函数前后的效果

### 代码

```
setTimeout(async () => {
    const fnc = timeSlice(fnc_)
    const fnc1 = timeSlice_(fnc_)
    let start = performance.now()

    console.log('开始')
    const a = await fnc()
    console.log('结束', `${(performance.now() - start)/ 1000}s`)

    console.log('开始')
    start = performance.now()
    const b = await fnc1()
    console.log('结束', `${(performance.now() - start)/ 1000}s`)

    console.log(a, b)
}, 1000);
复制代码
```

### 效果

对比优化后的时间分片函数，是之前效率的 4452 倍，我们做的只是提升了函数连续执行时间。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7033078c3d02405190d5c6d1e74161b4~tplv-k3u1fbpfcp-watermark.awebp?)

最后
--

generator 函数中 yield 的位置非常关键，需要放到耗时的地方，优化后的时间分片函数也提供了 time 变量，你可以根据实际情况来改变你的 time 值。
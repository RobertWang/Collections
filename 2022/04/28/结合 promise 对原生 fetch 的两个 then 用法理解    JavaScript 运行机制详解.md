> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/hanguidong/p/9497922.html)

前言：该问题是由于看到 fetch 的 then 方法的使用，产生的疑问，在深入了解并记录对 promise 的个人理解

首先看一下 fetch 请求使用案例：

案例效果：点击页面按钮，请求当前目录下的 arr.txt 里面的内容

疑问地方：

1. fetch 为什么可以使用 then？（个人理解 then 方法是定义在原型对象`Promise.prototype`上的）

2. 为什么使用两次 then 才能取出数据？（重点疑惑是这里，疑惑第二个 then 没有进行其他操作，只是将上一个 then 的返回值进行输出，就可以获取到 arr.txt 的数据）

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
let promise = new Promise(function(resolve, reject) {
  console.log('Promise');
  resolve();
});
 
promise.then(function() {
  console.log('resolved.');
});
 
console.log('Hi!');
 
// Promise
// Hi!
// resolved
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

 **所以去查询了 then 的用法，查询到阮一峰写了关于 promise 的文章 http://es6.ruanyifeng.com/?search=fecth&x=0&y=0#docs/promise，里面介绍到 promise 和 then 的具体用法：**

**看到一个案例：任务执行顺序问题：（中间插曲）**

```
// Promise
// Hi!
// resolved<br><br><strong>继续介绍then用法：<br></strong>
```

解释：

1.  Promise 新建后立即执行，所以首先输出的是`Promise`。

2. 当请求到数据后，执行 resolve 方法，改变 promise 状态为 resolved，使`then`方法执行第一个回调函数，第一个函数参数为 resolve 传递出来的数据，将在当前脚本所有同步任务执行完才会执行，所以`resolved`最后输出。为啥最后执行呢？

　　我又去查询下 [JavaScript 运行机制详解](https://www.cnblogs.com/hanguidong/p/9483460.html)：也是理解与阮一峰的：

　　理解出一条：先执行主线程（同步任务放置在主线程），主线程执行完，系统去读取任务队列中（异步任务放置在任务队列），js 的运行机制是这样设定的。嗯，没毛病

 3. 意思就是 resolve 是异步任务，放置在任务队列中，console.log("HI")  是同步任务，放置在主程序中，当主程序中的执行完，才会去查看任务队列。

执行结果：

```
还是但是fetch和promise还是没啥关系？<br>当我又看到一个案例：为啥getJSON（）这个可以使用then,应该是new promise才可以使用then吗?我才明白getJSON是封装的一个函数，返回值是new promise，所以执行getJSON()可以使用then的方法
```

　　Promise 实例具有`then`方法，也就是说，`then`方法是定义在原型对象`Promise.prototype`上的。它的作用是为 Promise 实例添加状态改变时的回调函数。前面说过，`then`方法的第一个参数是`resolved`状态的回调函数，第二个参数（可选）是`rejected`状态的回调函数。

 `then`方法返回的是一个新的`Promise`实例（注意，不是原来那个`Promise`实例）。因此可以采用链式写法，即`then`方法后面再调用另一个`then`方法

　　**这句话跟 fetch 的用法是一样的由于 then 的返回值是一个 promise 实例，可以采用的是链式写法，**

```
到现在才明白fetch（）其实就是封装了的promise的函数，返回值是promise的实例，所以才能调用then用法，所以说，fetch（）其实就是promise的实例。回到最初的疑问？
```

```
<strong>为什么使用两次then才能正常取出数据？我将最初的案例运行：查看第一个then的返回值是啥？<br></strong>
```

```
<strong>解释两次then用法：<br></strong><br><strong>第一次then用法</strong>：then是根据promise的状态变化而执行的回调函数，promise的状态变化由resolve()函数决定（取到数据执行resolve），then的参数为resolve函数传递出来的数据，<br>直接输出res是一个对象不是我们需要的数据，使用res.json()或者res.test()获取到我们需要的数据。<br>res.json（）/res.text（）获取到的是一个新的promise实例，arr.txt的值在[[[PromiseValue]]里面，但是直接取是取不出来的。没有方法取出来，<br>Promise的设计文档中说了，[[PromiseValue]]是个内部变量，外部无法得到，只能在then中获取。所以就会用到第二次then了
```

1. fetch 为什么可以使用 then？（个人理解 then 方法是定义在原型对象`Promise.prototype`上的）

2. 为什么使用两次 then 才能取出数据？（重点疑惑是这里，疑惑第二个 then 没有进行其他操作，只是将上一个 then 的返回值进行输出，就可以获取到 arr.txt 的数据）

```
<strong>第二次then用法：就是怎么将</strong>[[[PromiseValue]]里面的数据取出来
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
现在就重点理解下[[[PromiseValue]]这个怎么获取到的？<br>代码中的resolve（）就是说明resolve内部是怎么运行的，改变promise的状态，给PromiseValue复制，
```

　　　　　　　　　//　Promise  
　　　　　　　　　//　__proto__  
　　　　　　　　　//:  
　　　　　　　　　//Promise  
　　　　　　　　　//[[PromiseStatus]]:"resolved"  
　　　　　　　　　//[[PromiseValue]]:Array[3]

```
<strong>这句话解决了第二个then的用法：<br></strong>onFulfilled(value)和onRejected(reason)：<strong>参数value和reason的实参都是PromiseValue</strong>。这句话是说then的回调函数参数使用的都是<strong>PromiseValue，</strong>所以直接输出就会获取到PromiseValue的值<br><br><strong>这里有一点值得注意：第一个then 的return返回值是一个promise实例对象，所以回调链转交给了新的实例对象，第二个then的回调函数参数为为</strong><strong>PromiseValue的值，当返回值不是对象时，返回值是数据类型时，会将该返回值</strong>
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
<strong>赋值给PromiseValue，供下次的then函数使用<br></strong>
```

```
如果onFulfilled(value)和onRejected(reason)这两个回调函数中return返回值不是一个Promise的对象，（then）
```

```
那么这个返回值会被赋给PromiseValue，并在下一个then()的onFulfilled(value)和onRejected(reason)中做为实参使用。
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
但如果这个返回值是一个Promise的对象，那么剩下的由then()构造的回调链会转交给新的Promise对象并完成调用。
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
<strong>回调链是啥？？<br><strong>then(onFulfilled, onRejected)：</strong>这个方法实际上是把onFulfilled()函数和onRejected()函数添加到Promise对象的回调链中。<br>回调链就像一个由函数组构成的队列，每一组函数都是由至少一个函数构成（onFulfilled() 或者 onRejected() 或者 onFulfilled() 和 onRejected()）。<br>当resolve()或者reject()方法执行的时候，回调链中的回调函数会根据PromiseStatus的状态情况而被依次调用。<br></strong>
```

```
<em id="__mceDel"><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br></em>
```

```
如果onFulfilled(value)和onRejected(reason)这两个回调函数中return返回值不是一个Promise的对象，（then）
```

```
那么这个返回值会被赋给PromiseValue，并在下一个then()的onFulfilled(value)和onRejected(reason)中做为实参使用。
```

```
但如果这个返回值是一个Promise的对象，那么剩下的由then()构造的回调链会转交给新的Promise对象并完成调用。
```

```
**回调链是啥？？  
**then(onFulfilled, onRejected)：**这个方法实际上是把onFulfilled()函数和onRejected()函数添加到Promise对象的回调链中。  
回调链就像一个由函数组构成的队列，每一组函数都是由至少一个函数构成（onFulfilled() 或者 onRejected() 或者 onFulfilled() 和 onRejected()）。  
当resolve()或者reject()方法执行的时候，回调链中的回调函数会根据PromiseStatus的状态情况而被依次调用。  
**
```

```

```
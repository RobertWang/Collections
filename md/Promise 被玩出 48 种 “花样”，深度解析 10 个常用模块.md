> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6999804617320038408)

⚠️ 本文为掘金社区首发签约文章，未获授权禁止转载

最近，阿宝哥在梳理 [CLI](https://link.juejin.cn?target=https%3A%2F%2Fzh.wikipedia.org%2Fwiki%2F%25E5%2591%25BD%25E4%25BB%25A4%25E8%25A1%258C%25E7%2595%258C%25E9%259D%25A2 "https://zh.wikipedia.org/wiki/%E5%91%BD%E4%BB%A4%E8%A1%8C%E7%95%8C%E9%9D%A2")（Command Line Interface）的相关内容，就对优秀的 [Lerna](https://link.juejin.cn?target=https%3A%2F%2Flerna.js.org%2F "https://lerna.js.org/") 产生了兴趣，于是开始 “啃” 起了它的源码。在阅读开源项目时，阿宝哥习惯先阅读项目的 **README.md** 文档和 **package.json** 文件，而在 **package.json** 文件的 **dependencies** 字段中，阿宝哥见到了多个 **p-*** 的依赖包：

```
{
  "name": "lerna-monorepo",
  "version": "4.0.0-monorepo", 
  "dependencies": {
    "p-map": "^4.0.0",
    "p-map-series": "^2.1.0",
    "p-pipe": "^3.1.0",
    "p-queue": "^6.6.2",
    "p-reduce": "^2.1.0",
    "p-waterfall": "^2.1.1"
  }
}
复制代码
```

> 提示：如果你想知道阿宝哥如何阅读开源项目的话，可以阅读 [使用这些思路与技巧，我读懂了多个优秀的开源项目](https://juejin.cn/post/6887689159918485511 "https://juejin.cn/post/6887689159918485511") 这篇文章。

之后，阿宝哥顺藤摸瓜找到了 [promise-fun (3.5K)](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fpromise-fun "https://github.com/sindresorhus/promise-fun") 这个项目。该项目的作者 **[sindresorhus](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus "https://github.com/sindresorhus")** 是一个全职做开源的大牛，Github 上拥有 **43.9K** 的关注者。同时，他还维护了多个优秀的开源项目，比如 [awesome (167K)](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fawesome "https://github.com/sindresorhus/awesome")、[awesome-nodejs (42K)](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fawesome-nodejs "https://github.com/sindresorhus/awesome-nodejs")、[got (9.8K)](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fgot "https://github.com/sindresorhus/got")、[ora (7.1K)](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fora "https://github.com/sindresorhus/ora") 和 [screenfull.js (6.1K)](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fscreenfull.js "https://github.com/sindresorhus/screenfull.js") 等项目。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a3f0aadad8dd4fedbaf9cff6dfd50e90~tplv-k3u1fbpfcp-watermark.awebp)

（图片来源 —— [github.com/sindresorhu…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%25EF%25BC%2589 "https://github.com/sindresorhus%EF%BC%89")

[promise-fun](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fpromise-fun "https://github.com/sindresorhus/promise-fun") 项目收录了 **[sindresorhus](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus "https://github.com/sindresorhus")** 写过的 **48** 个与 [Promise](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FPromise "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise") 相关的模块，比如前面见到的 **[p-map](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-map "https://github.com/sindresorhus/p-map")**、**[p-map-series](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-map-series "https://github.com/sindresorhus/p-map-series")**、**[p-pipe](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-pipe "https://github.com/sindresorhus/p-pipe")** 和 **[p-waterfall](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-waterfall "https://github.com/sindresorhus/p-waterfall")** 等模块。本文阿宝哥将筛选一些比较常用的模块，来详细分析每个模块的用法和具体的代码实现。

这些模块提供了很多有用的方法，利用这些方法，可以帮我们解决工作中一些很常见的问题，比如实现并发控制、异步任务处理等，特别是处理多种控制流，比如 **series**、**waterfall**、**all**、**race** 和 **forever** 等。

掘友们，准备好了没？让我们一起开启 [promise-fun](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fpromise-fun "https://github.com/sindresorhus/promise-fun") 项目的 “探秘” 之旅。

### 初始化示例项目

创建一个新的 **learn-promise-fun** 项目，然后在该项目的根目录下，执行 `npm init -y` 命令进行项目初始化操作。当该命令成功运行后，在项目根目录下将会生成 **package.json** 文件。由于 [promise-fun](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fpromise-fun "https://github.com/sindresorhus/promise-fun") 项目中的很多模块使用了 ES Module，为了保证后续的示例代码能够正常运行，我们需要在 **package.json** 文件中，增加 **type** 字段并设置它的值为 **"module"**。

由于阿宝哥本地 [Node.js](https://link.juejin.cn?target=https%3A%2F%2Fnodejs.org%2Fen%2F "https://nodejs.org/en/") 的版本是 **v12.16.2**，要运行 ES Module 模块，还要添加 **--experimental-modules** 命令行选项。而如果你不想看到警告消息，还可以添加 **--no-warnings** 命令行选项。此外，为了避免每次运行示例代码时，都需要输入以上命令行选项，我们可以在 **package.json** 的 **scripts** 字段中定义相应的 **npm script** 命令，具体如下所示：

```
{
  "name": "learn-promise-fun",
  "type": "module",
  "scripts": {
    "pall": "node --experimental-modules ./p-all/p-all.test.js",
    "pfilter": "node --experimental-modules ./p-filter/p-filter.test.js",
    "pforever": "node --experimental-modules ./p-forever/p-forever.test.js",
    "preduce": "node --experimental-modules ./p-reduce/p-reduce.test.js",
    ...
  },
}
复制代码
```

在完成项目初始化之后，我们先来回顾一下大家平时用得比较多的 `reduce`、`map` 和 `filter` 数组方法的特点：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/db8a77d6eb8e40dc92861826659fc1ba~tplv-k3u1fbpfcp-watermark.awebp)

> 提示：上图通过👉 **[carbon.now.sh/](https://link.juejin.cn?target=https%3A%2F%2Fcarbon.now.sh%2F "https://carbon.now.sh/")** 在线网页制作生成。

相信大家对图中的 **[Array.prototype.reduce](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FArray%2FReduce "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce")** 方法不会陌生，该方法用于对数组中的每个元素执行一个 **reducer** 函数，并将结果汇总为单个返回值。对应的使用示例，如下所示：

```
const array1 = [1, 2, 3, 4];
const reducer = (accumulator, currentValue) => accumulator + currentValue;

// 1 + 2 + 3 + 4
console.log(array1.reduce(reducer)); // 10
复制代码
```

其中 **reducer** 函数接收 4 个参数：

*   acc（Accumulator）：累计器
*   cur（Current Value）： 当前值
*   idx（Current Index）：当前索引
*   src（Source Array）： 源数组

而接下来，我们要介绍的 [p-reduce](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-reduce "https://github.com/sindresorhus/p-reduce") 模块，就提供了跟 **[Array.prototype.reduce](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FArray%2FReduce "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce")** 方法类似的功能。

### [p-reduce](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-reduce "https://github.com/sindresorhus/p-reduce")

> Reduce a list of values using promises into a promise for a value
> 
> [github.com/sindresorhu…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-reduce "https://github.com/sindresorhus/p-reduce")

#### 使用说明

[p-reduce](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-reduce "https://github.com/sindresorhus/p-reduce") 适用于需要根据异步资源计算累加值的场景。该模块默认导出了一个 **pReduce** 函数，该函数的签名如下：

> **[pReduce(input, reducer, initialValue): Promise](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-reduce "https://github.com/sindresorhus/p-reduce")**

*   `input: Iterable<Promise|any>`
*   `reducer(previousValue, currentValue, index): Function`
*   `initialValue: unknown`

了解完 **pReduce** 函数的签名之后，我们来看一下该函数如何使用。

#### 使用示例

```
// p-reduce/p-reduce.test.js
import delay from "delay";
import pReduce from "p-reduce";

const inputs = [Promise.resolve(1), delay(50, { value: 6 }), 8];

async function main() {
  const result = await pReduce(inputs, async (a, b) => a + b, 0);
  console.dir(result); // 输出结果：15
}

main();
复制代码
```

在以上示例中，我们导入了 **[delay](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fdelay "https://github.com/sindresorhus/delay")** 模块默认导出的 `delay` 方法，该方法可用于按照给定的时间，延迟一个 Promise 对象。即在给定的时间之后，Promise 状态才会变成 **resolved**。默认情况下，**[delay](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fdelay "https://github.com/sindresorhus/delay")** 模块内部是通过 [setTimeout](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FWindowOrWorkerGlobalScope%2FsetTimeout "https://developer.mozilla.org/zh-CN/docs/Web/API/WindowOrWorkerGlobalScope/setTimeout") API 来实现延迟功能的。示例中 `delay(50, { value: 6 })` 表示延迟 50ms 后，Promise 对象的返回值为 **6**。而在 `main` 函数内部，我们使用了 `pReduce` 函数来计算 `inputs` 数组元素的累加值。当以上代码成功运行之后，命令行会输出 **15**。

下面我们来分析一下 `pReduce` 函数内部是如何实现的。

#### 源码分析

```
// https://github.com/sindresorhus/p-reduce/blob/main/index.js
export default async function pReduce(iterable, reducer, initialValue) {
  return new Promise((resolve, reject) => {
    const iterator = iterable[Symbol.iterator](); // 获取迭代器
    let index = 0; // 索引值

    const next = async (total) => {
      const element = iterator.next(); // 获取下一项

      if (element.done) { // 判断迭代器是否迭代完成
        resolve(total);
        return;
      }

      try {
        const [resolvedTotal, resolvedValue] = await Promise.all([
          total,
          element.value,
        ]);
        // 迭代下一项
        // reducer(previousValue, currentValue, index): Function
        next(reducer(resolvedTotal, resolvedValue, index++));
      } catch (error) {
        reject(error);
      }
    };

    // 使用初始值，开始迭代
    next(initialValue);
  });
}
复制代码
```

在以上代码中，核心的流程就是通过获取 `iterable` 对象内部的迭代器，来不断地进行迭代。此外，在 `pReduce` 函数中，使用了 [**Promise.all**](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FPromise%2Fall "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/all") 方法，该方法会返回一个 promise 对象，当输入的所有 promise 对象的状态都变成 `resolved` 时，返回的 promise 对象就会以数组的形式，返回每个 promise 对象 resolve 后的结果。当输入的任何一个 promise 对象状态变成 `rejected` 时，则返回的 promise 对象会 reject 对应的错误信息。

不过，需要注意的是，[**Promise.all**](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FPromise%2Fall "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/all") 方法存在兼容性问题，具体的兼容性如下图所示：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/622880e18e4149fcbb1fccff73634df6~tplv-k3u1fbpfcp-watermark.awebp)

（图片来源 —— [caniuse.com/?search=Pro…](https://link.juejin.cn?target=https%3A%2F%2Fcaniuse.com%2F%3Fsearch%3DPromise.all%25EF%25BC%2589 "https://caniuse.com/?search=Promise.all%EF%BC%89")

可能有一些小伙伴对 [**Promise.all**](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FPromise%2Fall "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/all") 还不熟悉，它又是一道比较高频的手写题。所以，接下来我们来手写一个简易版的 [**Promise.all**](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FPromise%2Fall "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/all")：

```
Promise.all = function (iterators) {
  return new Promise((resolve, reject) => {
    if (!iterators || iterators.length === 0) {
      resolve([]);
    } else {
      let count = 0; // 计数器，用于判断所有任务是否执行完成
      let result = []; // 结果数组
      for (let i = 0; i < iterators.length; i++) {
        // 考虑到iterators[i]可能是普通对象，则统一包装为Promise对象
        Promise.resolve(iterators[i]).then(
          (data) => {
            result[i] = data; // 按顺序保存对应的结果
            // 当所有任务都执行完成后，再统一返回结果
            if (++count === iterators.length) {
              resolve(result);
            }
          },
          (err) => {
            reject(err); // 任何一个Promise对象执行失败，则调用reject()方法
            return;
          }
        );
      }
    }
  });
};
复制代码
```

### [p-map](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-map "https://github.com/sindresorhus/p-map")

> Map over promises concurrently
> 
> [github.com/sindresorhu…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-map "https://github.com/sindresorhus/p-map")

#### 使用说明

[p-map](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-map "https://github.com/sindresorhus/p-map") 适用于使用不同的输入多次运行 **promise-returning** 或 **async** 函数的场景。与前面介绍的 [Promise.all](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FPromise%2Fall "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/all") 方法的区别是，你可以控制并发，也可以决定是否在出现错误时停止迭代。该模块默认导出了一个 **pMap** 函数，该函数的签名如下：

> **[pMap(input, mapper, options): Promise](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-map "https://github.com/sindresorhus/p-map")**

*   `input: Iterable<Promise | unknown>`
*   `mapper(element, index): Function`
*   `options: object`
    *   `concurrency: number` —— 并发数，默认值 `Infinity`，最小值为 `1`；
    *   `stopOnError: boolean` —— 出现异常时，是否终止，默认值为 `true`。

了解完 **pMap** 函数的签名之后，我们来看一下该函数如何使用。

#### 使用示例

```
// p-map/p-map.test.js
import delay from "delay";
import pMap from "p-map";

const inputs = [200, 100, 50];
const mapper = (value) => delay(value, { value });

async function main() {
  console.time("start");
  const result = await pMap(inputs, mapper, { concurrency: 1 });
  console.dir(result); // 输出结果：[ 200, 100, 50 ]
  console.timeEnd("start");
}

main();
复制代码
```

在以上示例中，我们也使用了 **[delay](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fdelay "https://github.com/sindresorhus/delay")** 模块导出的 `delay` 函数，用于实现延迟功能。当成功执行以上代码后，命令行会输出以下信息：

```
[ 200, 100, 50 ]
start: 368.708ms
复制代码
```

而当把 `concurrency` 属性的值更改为 `2` 之后，再次执行以上代码。那么命令行将会输出以下信息：

```
[ 200, 100, 50 ]
start: 210.322ms
复制代码
```

观察以上的输出结果，我们可以看出并发数为 `1` 时，程序的运行时间大于 350ms。而如果并发数为 `2` 时，多个任务是并行执行的，所以程序的运行时间只需 210 多毫秒。那么 `pMap` 函数，内部是如何实现并发控制的呢？下面来分析一下 `pMap` 函数的源码。

#### 源码分析

```
// https://github.com/sindresorhus/p-map/blob/main/index.js
import AggregateError from "aggregate-error";

export default async function pMap(
  iterable,
  mapper,
  { concurrency = Number.POSITIVE_INFINITY, stopOnError = true } = {}
) {
  return new Promise((resolve, reject) => {
    // 省略参数校验代码
    const result = []; // 存储返回结果
    const errors = []; // 存储异常对象
    const skippedIndexes = []; // 保存跳过项索引值的数组
    const iterator = iterable[Symbol.iterator](); // 获取迭代器
    let isRejected = false; // 标识是否出现异常
    let isIterableDone = false; // 标识是否已迭代完成
    let resolvingCount = 0; // 正在处理的任务个数
    let currentIndex = 0; // 当前索引

    const next = () => {
      if (isRejected) { // 若出现异常，则直接返回
        return;
      }

      const nextItem = iterator.next(); // 获取下一项
      const index = currentIndex; // 记录当前的索引值
      currentIndex++;

      if (nextItem.done) { // 判断迭代器是否迭代完成
        isIterableDone = true;

        // 判断是否所有的任务都已经完成了
        if (resolvingCount === 0) { 
          if (!stopOnError && errors.length > 0) { // 异常处理
            reject(new AggregateError(errors));
          } else {
            for (const skippedIndex of skippedIndexes) {
              // 删除跳过的值，不然会存在空的占位
              result.splice(skippedIndex, 1); 
            }
            resolve(result); // 返回最终的处理结果
          }
        }
        return;
      }

      resolvingCount++; // 正在处理的任务数加1

      (async () => {
        try {
          const element = await nextItem.value;

          if (isRejected) {
            return;
          }

          // 调用mapper函数，进行值进行处理
          const value = await mapper(element, index);
          // 处理跳过的情形，可以在mapper函数中返回pMapSkip，来跳过当前项
          // 比如在异常捕获的catch语句中，返回pMapSkip值
          if (value === pMapSkip) { // pMapSkip = Symbol("skip")
            skippedIndexes.push(index);
          } else {
            result[index] = value; // 把返回值按照索引进行保存
          }

          resolvingCount--;
          next(); // 迭代下一项
        } catch (error) {
          if (stopOnError) { // 出现异常时，是否终止，默认值为true
            isRejected = true;
            reject(error);
          } else {
            errors.push(error);
            resolvingCount--;
            next();
          }
        }
      })();
    };

    // 根据配置的concurrency值，并发执行任务
    for (let index = 0; index < concurrency; index++) {
      next();
      if (isIterableDone) {
        break;
      }
    }
  });
}

export const pMapSkip = Symbol("skip");
复制代码
```

`pMap` 函数内部的处理逻辑还是蛮清晰的，把核心的处理逻辑都封装在 `next` 函数中。在调用 `pMap` 函数之后，内部会根据配置的`concurrency` 值，并发调用 `next` 函数。而在 `next` 函数内部，会通过 **async/await** 来控制任务的执行过程。

在 `pMap` 函数中，作者巧妙设计了 **pMapSkip**。当我们在 `mapper` 函数中返回了 **pMapSkip** 之后，将会从返回的结果数组中移除对应索引项的值。了解完 **pMapSkip** 的作用之后，我们来举个简单的例子：

```
import pMap, { pMapSkip } from "p-map";

const inputs = [200, pMapSkip, 50];
const mapper = async (value) => value;

async function main() {
  console.time("start");
  const result = await pMap(inputs, mapper, { concurrency: 2 });
  console.dir(result); // [ 200, 50 ]
  console.timeEnd("start");
}

main();
复制代码
```

在以上代码中，我们的 `inputs` 数组中包含了 `pMapSkip` 值，当使用 `pMap` 函数对 `inputs` 数组进行处理后，`pMapSkip` 值将会被过滤掉，所以最终 `result` 的结果为 **[200 , 50]**。

### [p-filter](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-filter "https://github.com/sindresorhus/p-filter")

> Filter promises concurrently
> 
> [github.com/sindresorhu…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-filter "https://github.com/sindresorhus/p-filter")

#### 使用说明

[p-filter](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-filter "https://github.com/sindresorhus/p-filter") 适用于使用不同的输入多次运行 **promise-returning** 或 **async** 函数，并对返回的结果进行过滤的场景。该模块默认导出了一个 **pFilter** 函数，该函数的签名如下：

> **[pFilter(input, filterer, options): Promise](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-filter "https://github.com/sindresorhus/p-filter")**

*   `input: Iterable<Promise | any>`
*   `filterer(element, index): Function`
*   `options: object`
    *   `concurrency: number` —— 并发数，默认值 `Infinity`，最小值为 `1`。

了解完 **pFilter** 函数的签名之后，我们来看一下该函数如何使用。

#### 使用示例

```
// p-filter/p-filter.test.js
import pFilter from "p-filter";

const inputs = [Promise.resolve(1), 2, 3];
const filterer = (x) => x % 2;

async function main() {
  const result = await pFilter(inputs, filterer, { concurrency: 1 });
  console.dir(result); // 输出结果：[ 1, 3 ]
}

main();
复制代码
```

在以上示例中，我们使用 `pFilter` 函数对包含 `Promise` 对象的 `inputs` 数组，应用了 `(x) => x % 2` 过滤器。当以上代码成功运行后，命令行会输出 **[1, 3]**。

#### 源码分析

```
// https://github.com/sindresorhus/p-filter/blob/main/index.js
const pMap = require('p-map');

const pFilter = async (iterable, filterer, options) => {
	const values = await pMap(
		iterable,
		(element, index) => Promise.all([filterer(element, index), element]),
		options
	);
	return values.filter(value => Boolean(value[0])).map(value => value[1]);
};
复制代码
```

由以上代码可知，在 `pFilter` 函数内部，使用了我们前面已经介绍过的 `pMap` 和 `Promise.all` 函数。要理解以上代码，我们还需要来回顾一下 `pMap` 函数的关键代码：

```
// https://github.com/sindresorhus/p-map/blob/main/index.js
export default async function pMap(
  iterable, mapper,
  { concurrency = Number.POSITIVE_INFINITY, stopOnError = true } = {}
) {
  const iterator = iterable[Symbol.iterator](); // 获取迭代器
  let currentIndex = 0; // 当前索引
  
  const next = () => {
    const nextItem = iterator.next(); // 获取下一项
    const index = currentIndex;
    currentIndex++;
    (async () => {
        try {
          // element => await Promise.resolve(1);
          const element = await nextItem.value;
          // mapper => (element, index) => Promise.all([filterer(element, index), element])
          const value = await mapper(element, index);
          if (value === pMapSkip) {
            skippedIndexes.push(index);
          } else {
            result[index] = value; // 把返回值按照索引进行保存
          }
          resolvingCount--;
          next(); // 迭代下一项
        } catch (error) {
          // 省略异常处理代码
        }
    })();
  }	
}
复制代码
```

因为 `pFilter` 函数中所用的 `mapper` 函数是 `(element, index) => Promise.all([filterer(element, index), element])`，所以 `await mapper(element, index)` 表达式的返回值是一个数组。数组的第 1 项是 `filterer` 过滤器的处理结果，而数组的第 2 项是当前元素的值。因此在调用 `pMap` 函数后，它的返回值是一个二维数组。所以在获取 `pMap` 函数的返回值之后， 会使用以下语句对返回值进行处理：

```
values.filter(value => Boolean(value[0])).map(value => value[1])
复制代码
```

其实，对于前面的 `pFilter` 示例来说，除了 `inputs` 可以含有 Promise 对象，我们的 `filterer` 过滤器也可以返回 Promise 对象：

```
import pFilter from "p-filter";

const inputs = [Promise.resolve(1), 2, 3];
const filterer = (x) => Promise.resolve(x % 2);

async function main() {
  const result = await pFilter(inputs, filterer);
  console.dir(result); // [ 1, 3 ]
}

main();
复制代码
```

以上代码成功执行后，命令行的输出结果也是 **[1, 3]**。好的，现在我们已经介绍了 [p-reduce](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-reduce "https://github.com/sindresorhus/p-reduce")、[p-map](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-map "https://github.com/sindresorhus/p-map") 和 [p-filter](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-filter "https://github.com/sindresorhus/p-filter") 3 个模块。下面我们来继续介绍另一个模块 —— [p-waterfall](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-waterfall "https://github.com/sindresorhus/p-waterfall")。

### [p-waterfall](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-waterfall "https://github.com/sindresorhus/p-waterfall")

> Run promise-returning & async functions in series, each passing its result to the next
> 
> [github.com/sindresorhu…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-waterfall "https://github.com/sindresorhus/p-waterfall")

#### 使用说明

[p-waterfall](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-waterfall "https://github.com/sindresorhus/p-waterfall") 适用于串行执行 **promise-returning** 或 **async** 函数，并把前一个函数的返回结果自动传给下一个函数的场景。该模块默认导出了一个 **pWaterfall** 函数，该函数的签名如下：

> **[pWaterfall(tasks, initialValue): Promise](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-waterfall "https://github.com/sindresorhus/p-waterfall")**

*   `tasks: Iterable<Function>`
*   `initialValue: unknown`：将作为第一个任务的 `previousValue`

了解完 **pWaterfall** 函数的签名之后，我们来看一下该函数如何使用。

#### 使用示例

```
// p-waterfall/p-waterfall.test.js
import pWaterfall from "p-waterfall";

const tasks = [
  async (val) => val + 1,
  (val) => val + 2,
  async (val) => val + 3,
];

async function main() {
  const result = await pWaterfall(tasks, 0);
  console.dir(result); // 输出结果：6
}

main();
复制代码
```

在以上示例中，我们创建了 3 个任务，然后使用 `pWaterfall` 函数来执行这 3 个任务。当以上代码成功执行后，命令行会输出 **6**。对应的执行流程如下图所示：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/23640c8512e346df8b2325b0df11df6e~tplv-k3u1fbpfcp-watermark.awebp)

#### 源码分析

```
// https://github.com/sindresorhus/p-waterfall/blob/main/index.js
import pReduce from 'p-reduce';

export default async function pWaterfall(iterable, initialValue) {
	return pReduce(iterable, (previousValue, function_) => function_(previousValue), initialValue);
}
复制代码
```

在 `pWaterfall` 函数内部，会利用前面已经介绍的 `pReduce` 函数来实现 **waterfall** 流程控制。同样，要搞清楚内部的控制流程，我们需要来回顾一下 `pReduce` 函数的具体实现：

```
export default async function pReduce(iterable, reducer, initialValue) {
  return new Promise((resolve, reject) => {
    const iterator = iterable[Symbol.iterator](); // 获取迭代器
    let index = 0; // 索引值

    const next = async (total) => {
      const element = iterator.next(); // 获取下一项

      if (element.done) {
        // 判断迭代器是否迭代完成
        resolve(total);
        return;
      }

      try {
        // 首次调用next函数的状态：
        // resolvedTotal => 0
        // element.value => async (val) => val + 1
        const [resolvedTotal, resolvedValue] = await Promise.all([
          total,
          element.value,
        ]);
        // reducer => (previousValue, function_) => function_(previousValue)
        next(reducer(resolvedTotal, resolvedValue, index++));
      } catch (error) {
        reject(error);
      }
    };

    // 使用初始值，开始迭代
    next(initialValue);
  });
}
复制代码
```

现在我们已经知道 **pWaterfall** 函数会把前一个任务的输出结果，作为输入传给下一个任务。但有些时候，在串行执行每个任务时，我们并不关心每个任务的返回值。针对这种场合，我们可以考虑使用 [p-series](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-series "https://github.com/sindresorhus/p-series") 模块提供的 `pSeries` 函数。

### [p-series](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-series "https://github.com/sindresorhus/p-series")

> Run promise-returning & async functions in series
> 
> [github.com/sindresorhu…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-series "https://github.com/sindresorhus/p-series")

#### 使用说明

[p-series](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-series "https://github.com/sindresorhus/p-series") 适用于串行执行 **promise-returning** 或 **async** 函数的场景。

#### 使用示例

```
// p-series/p-series.test.js
import pSeries from "p-series";

const tasks = [async () => 1 + 1, () => 2 + 2, async () => 3 + 3];

async function main() {
  const result = await pSeries(tasks);
  console.dir(result); // 输出结果：[2, 4, 6]
}

main();
复制代码
```

在以上示例中，我们创建了 3 个任务，然后使用 `pSeries` 函数来执行这 3 个任务。当以上代码成功执行后，命令行会输出 **[2, 4, 6]**。对应的执行流程如下图所示：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eba0f1a8c2fb47d4a0f01536197ff7a1~tplv-k3u1fbpfcp-watermark.awebp)

#### 源码分析

```
// https://github.com/sindresorhus/p-series/blob/main/index.js
export default async function pSeries(tasks) {
	for (const task of tasks) {
		if (typeof task !== 'function') {
			throw new TypeError(`Expected task to be a \`Function\`, received \`${typeof task}\``);
		}
	}

	const results = [];

	for (const task of tasks) {
		results.push(await task()); // eslint-disable-line no-await-in-loop
	}

	return results;
}
复制代码
```

由以上代码可知，在 `pSeries` 函数内部是利用 **[for...of](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FStatements%2Ffor...of "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/for...of")** 语句和 **[async/await](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FLearn%2FJavaScript%2FAsynchronous%2FAsync_await "https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Asynchronous/Async_await")** 特性来实现串行任务流控制。因此在实际的项目中，你也可以无需使用该模块，就可以轻松的实现串行任务流控制。

### [p-all](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-all "https://github.com/sindresorhus/p-all")

> Run promise-returning & async functions concurrently with optional limited concurrency
> 
> [github.com/sindresorhu…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-all "https://github.com/sindresorhus/p-all")

#### 使用说明

[p-all](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-all "https://github.com/sindresorhus/p-all") 适用于并发执行 **promise-returning** 或 **async** 函数的场景。该模块提供的功能，与 [**Promise.all**](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FPromise%2Fall "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/all") API 类似，主要的区别是该模块允许你限制任务的并发数。在日常开发过程中，一个比较常见的场景就是控制 HTTP 请求的并发数，这时你也可以考虑使用 [async-pool](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Frxaviers%2Fasync-pool "https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Frxaviers%2Fasync-pool") 这个库来解决并发控制的问题，如果你对该库的内部实现原理感兴趣的话，可以阅读 [JavaScript 中如何实现并发控制？](https://juejin.cn/post/6976028030770610213 "https://juejin.cn/post/6976028030770610213") 这篇文章。

下面我们来继续介绍 [p-all](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-all "https://github.com/sindresorhus/p-all") 模块，该模块默认导出了一个 **pAll** 函数，该函数的签名如下：

> **[pAll(tasks, options)](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-all "https://github.com/sindresorhus/p-all")**

*   `tasks: Iterable<Function>`
*   `options: object`
    *   `concurrency: number` —— 并发数，默认值 `Infinity`，最小值为 `1`；
    *   `stopOnError: boolean` —— 出现异常时，是否终止，默认值为 `true`。

#### 使用示例

```
// p-all/p-all.test.js
import delay from "delay";
import pAll from "p-all";

const inputs = [
  () => delay(200, { value: 1 }),
  async () => {
    await delay(100);
    return 2;
  },
  async () => 8,
];

async function main() {
  console.time("start");
  const result = await pAll(inputs, { concurrency: 1 });
  console.dir(result); // 输出结果：[ 1, 2, 8 ]
  console.timeEnd("start");
}

main();
复制代码
```

在以上示例中，我们创建了 3 个异步任务，然后通过 `pAll` 函数来执行已创建的任务。当成功执行以上代码后，命令行会输出以下信息：

```
[ 1, 2, 8 ]
start: 312.561ms
复制代码
```

而当把 `concurrency` 属性的值更改为 `2` 之后，再次执行以上代码。那么命令行将会输出以下信息：

```
[ 1, 2, 8 ]
start: 209.469ms
复制代码
```

可以看出并发数为 `1` 时，程序的运行时间大于 300ms。而如果并发数为 `2` 时，前面两个任务是并行的，所以程序的运行时间只需 200 多毫秒。

#### 源码分析

```
// https://github.com/sindresorhus/p-all/blob/main/index.js
import pMap from 'p-map';

export default async function pAll(iterable, options) {
	return pMap(iterable, element => element(), options);
}
复制代码
```

很明显在 `pAll` 函数内部，是通过 [p-map](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-map "https://github.com/sindresorhus/p-map") 模块提供的 `pMap` 函数来实现并发控制的。如果你对 `pMap` 函数的内部实现方式，还不清楚的话，可以回过头再次阅读 [p-map](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-map "https://github.com/sindresorhus/p-map") 模块的相关内容。接下来，我们来继续介绍另一个模块 —— [p-race](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-race "https://github.com/sindresorhus/p-race")。

### [p-race](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-race "https://github.com/sindresorhus/p-race")

> A better `Promise.race()`
> 
> [github.com/sindresorhu…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-race "https://github.com/sindresorhus/p-race")

#### 使用说明

[p-race](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-race "https://github.com/sindresorhus/p-race") 这个模块修复了 [Promise.race](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FPromise%2Frace "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/race") API 一个 “愚蠢” 的行为。当使用空的可迭代对象，调用 [Promise.race](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FPromise%2Frace "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/race") API 时，将会返回一个永远处于 **pending** 状态的 Promise 对象，这可能会产生一些非常难以调试的问题。而如果往 [p-race](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-race "https://github.com/sindresorhus/p-race") 模块提供的 `pRace` 函数中传入一个空的可迭代对象时，该函数将会立即抛出 **RangeError: Expected the iterable to contain at least one item** 的异常信息。

**`pRace(iterable)`** 方法会返回一个 promise 对象，一旦迭代器中的某个 promise 对象 **resolved** 或 **rejected**，返回的 promise 对象就会 resolve 或 reject 相应的值。

#### 使用示例

```
// p-race/p-race.test.js
import delay from "delay";
import pRace from "p-race";

const inputs = [delay(50, { value: 1 }), delay(100, { value: 2 })];

async function main() {
  const result = await pRace(inputs);
  console.dir(result); // 输出结果：1
}

main();
复制代码
```

在以上示例中，我们导入了 **[delay](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fdelay "https://github.com/sindresorhus/delay")** 模块默认导出的 `delay` 方法，该方法可用于按照给定的时间，延迟一个 Promise 对象。利用 `delay` 函数，我们创建了 2 个 Promise 对象，然后使用 `pRace` 函数来处理这两个 Promise 对象。以上代码成功运行后，命令行始终会输出 **1**。那么为什么会这样呢？下面我们来分析一下 `pRace` 函数的源码。

#### 源码分析

```
// https://github.com/sindresorhus/p-race/blob/main/index.js
import isEmptyIterable from 'is-empty-iterable';

export default async function pRace(iterable) {
	if (isEmptyIterable(iterable)) {
		throw new RangeError('Expected the iterable to contain at least one item');
	}

	return Promise.race(iterable);
}
复制代码
```

观察以上源码可知，在 `pRace` 函数内部会先判断传入的 `iterable` 参数是否为空的可迭代对象。检测参数是否为空的可迭代对象，是通过 `isEmptyIterable` 函数来实现，该函数的具体代码如下所示：

```
// https://github.com/sindresorhus/is-empty-iterable/blob/main/index.js
function isEmptyIterable(iterable) {
	for (const _ of iterable) {
		return false;
	}

	return true;
}
复制代码
```

当发现是空的可迭代对象时，`pRace` 函数会直接抛出 `RangeError` 异常。否则，会利用 `Promise.race` API 来实现具体的功能。需要注意的是，[**Promise.race**](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FPromise%2Frace "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/race") 方法也存在兼容性问题，具体如下图所示：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/431968918a034a2c8597bc8bfcb48a46~tplv-k3u1fbpfcp-watermark.awebp)

（图片来源 —— [caniuse.com/?search=Pro…](https://link.juejin.cn?target=https%3A%2F%2Fcaniuse.com%2F%3Fsearch%3DPromise.race%25EF%25BC%2589 "https://caniuse.com/?search=Promise.race%EF%BC%89")

同样，可能有一些小伙伴对 [**Promise.race**](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FPromise%2Frace "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/race") 还不熟悉，它也是一道挺高频的手写题。所以，接下来我们来手写一个简易版的 [**Promise.race**](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FPromise%2Frace "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/race")：

```
Promise.race = function (iterators) {
  return new Promise((resolve, reject) => {
    for (const iter of iterators) {
      Promise.resolve(iter)
        .then((res) => {
          resolve(res);
        })
        .catch((e) => {
          reject(e);
        });
    }
  });
};
复制代码
```

### [p-forever](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-forever "https://github.com/sindresorhus/p-forever")

> Run promise-returning & async functions repeatedly until you end it
> 
> [github.com/sindresorhu…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-forever "https://github.com/sindresorhus/p-forever")

#### 使用说明

[p-forever](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-forever "https://github.com/sindresorhus/p-forever") 适用于需要重复不断执行 **promise-returning** 或 **async** 函数，直到用户终止的场景。该模块默认导出了一个 **pForever** 函数，该函数的签名如下：

> **[pForever(fn, initialValue)](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-forever "https://github.com/sindresorhus/p-forever")**

*   `fn: Function`：重复执行的函数；
*   `initialValue`：传递给 `fn` 函数的初始值。

了解完 **pForever** 函数的签名之后，我们来看一下该函数如何使用。

#### 使用示例

```
// p-forever/p-forever.test.js
import delay from "delay";
import pForever from "p-forever";

async function main() {
  let index = 0;
  await pForever(async () => (++index === 10 ? pForever.end : delay(50)));
  console.log("当前index的值: ", index); // 输出结果：当前index的值: 10
}

main();
复制代码
```

在以上示例中，传入 `pForever` 函数的 `fn` 函数会一直重复执行，直到该 `fn` 函数返回 `pForever.end` 的值，才会终止执行。因此以上代码成功执行后，命令行的输出结果是：**当前 index 的值: 10**。

#### 源码分析

```
// https://github.com/sindresorhus/p-forever/blob/main/index.js
const endSymbol = Symbol('pForever.end');

const pForever = async (function_, previousValue) => {
	const newValue = await function_(await previousValue);
	if (newValue === endSymbol) {
		return;
	}
	return pForever(function_, newValue);
};

pForever.end = endSymbol;
export default pForever;
复制代码
```

由以上源码可知，`pForever` 函数的内部实现并不复杂。当判断 `newValue` 的值为 `endSymbol` 时，就直接返回了。否则，就会继续调用 `pForever` 函数。除了一直重复执行任务之外，有时候我们会希望显式指定任务的执行次数，针对这种场景，我们就可以使用 [p-times](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-times "https://github.com/sindresorhus/p-times") 模块。

### [p-times](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-times "https://github.com/sindresorhus/p-times")

> Run promise-returning & async functions a specific number of times concurrently
> 
> [github.com/sindresorhu…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-times "https://github.com/sindresorhus/p-times")

#### 使用说明

[p-times](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-times "https://github.com/sindresorhus/p-times") 适用于显式指定 **promise-returning** 或 **async** 函数执行次数的场景。该模块默认导出了一个 **pTimes** 函数，该函数的签名如下：

> **[pTimes(count, mapper, options): Promise](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-times "https://github.com/sindresorhus/p-times")**

*   `count: number`：调用次数；
*   `mapper(index): Function`：mapper 函数，调用该函数后会返回 Promise 对象或某个具体的值；
*   `options: object`
    *   `concurrency: number` —— 并发数，默认值 `Infinity`，最小值为 `1`；
    *   `stopOnError: boolean` —— 出现异常时，是否终止，默认值为 `true`。

了解完 **pTimes** 函数的签名之后，我们来看一下该函数如何使用。

#### 使用示例

```
// p-times/p-times.test.js
import delay from "delay";
import pTimes from "p-times";

async function main() {
  console.time("start");
  const result = await pTimes(5, async (i) => delay(50, { value: i * 10 }), {
    concurrency: 3,
  });
  console.dir(result);
  console.timeEnd("start");
}

main();
复制代码
```

在以上示例中，我们通过 `pTimes` 函数配置 **mapper** 函数的执行次数为 **5** 次，同时设置任务的并发数为 **3**。当以上代码成功运行后，命令行会输出以下结果：

```
[ 0, 10, 20, 30, 40 ]
start: 116.090ms
复制代码
```

对于以上示例，你可以通过改变 `concurrency` 的值，来对比输出的程序运行时间。那么 `pTimes` 函数内部是如何实现并发控制的呢？其实该函数内部也是利用 `pMap` 函数来实现并发控制。

#### 源码分析

```
// https://github.com/sindresorhus/p-times/blob/main/index.js
import pMap from "p-map";

export default function pTimes(count, mapper, options) {
  return pMap(
    Array.from({ length: count }).fill(),
    (_, index) => mapper(index),
    options
  );
}
复制代码
```

在 `pTimes` 函数中，会通过 [Array.from](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FArray%2Ffrom "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/from") 方法创建指定长度的数组，然后通过 `fill` 方法进行填充。最后再把该数组、`mapper` 函数和 `options` 配置对象，作为输入参数调用 `pMap` 函数。写到这里，阿宝哥觉得 `pMap` 函数提供的功能还是蛮强大的，很多模块的内部都使用了 `pMap` 函数。

### [p-pipe](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-pipe "https://github.com/sindresorhus/p-pipe")

> Compose promise-returning & async functions into a reusable pipeline
> 
> [github.com/sindresorhu…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-pipe "https://github.com/sindresorhus/p-pipe")

#### 使用说明

[p-pipe](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-pipe "https://github.com/sindresorhus/p-pipe") 适用于把 **promise-returning** 或 **async** 函数组合成可复用的管道。该模块默认导出了一个 **pPipe** 函数，该函数的签名如下：

> **[pPipe(input...)](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-pipe "https://github.com/sindresorhus/p-pipe")**

*   `input: Function`：期望调用后会返回 Promise 或任何值的函数。

了解完 **pPipe** 函数的签名之后，我们来看一下该函数如何使用。

#### 使用示例

```
// p-pipe/p-pipe.test.js
import pPipe from "p-pipe";

const addUnicorn = async (string) => `${string} Unicorn`;
const addRainbow = async (string) => `${string} Rainbow`;

const pipeline = pPipe(addUnicorn, addRainbow);

(async () => {
  console.log(await pipeline("❤️")); // 输出结果：❤️ Unicorn Rainbow
})();
复制代码
```

在以上示例中，我们通过 `pPipe` 函数把 `addUnicorn` 和 `addRainbow` 这两个函数组合成一个新的可复用的管道。被组合函数的执行顺序是从左到右，所以以上代码成功运行后，命令行会输出 **❤️ Unicorn Rainbow**。

#### 源码分析

```
// https://github.com/sindresorhus/p-pipe/blob/main/index.js
export default function pPipe(...functions) {
	if (functions.length === 0) {
		throw new Error('Expected at least one argument');
	}

	return async input => {
		let currentValue = input;

		for (const function_ of functions) {
			currentValue = await function_(currentValue); // eslint-disable-line no-await-in-loop
		}
		return currentValue;
	};
}
复制代码
```

由以上代码可知，在 `pPipe` 函数内部是利用 **[for...of](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FStatements%2Ffor...of "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/for...of")** 语句和 **[async/await](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FLearn%2FJavaScript%2FAsynchronous%2FAsync_await "https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Asynchronous/Async_await")** 特性来实现管道的功能。分析完 [promise-fun](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fpromise-fun "https://github.com/sindresorhus/promise-fun") 项目中的 10 个模块之后，再次感受到 **[async/await](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FLearn%2FJavaScript%2FAsynchronous%2FAsync_await "https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Asynchronous/Async_await")** 特性给前端异步编程带来的巨大便利。其实对于异步的场景来说，除了可以使用 [promise-fun](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fpromise-fun "https://github.com/sindresorhus/promise-fun") 项目中收录的模块之外，还可以使用 [async](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fcaolan%2Fasync "https://github.com/caolan/async") 或 [neo-async](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsuguru03%2Fneo-async "https://github.com/suguru03/neo-async") 这两个异步处理模块提供的工具函数。在 Webpack 项目中，就用到了 [neo-async](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsuguru03%2Fneo-async "https://github.com/suguru03/neo-async") 这个模块，该模块的作者是希望用来替换 [async](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fcaolan%2Fasync "https://github.com/caolan/async") 模块，以提供更好的性能。建议需要经常处理异步场景的小伙伴，抽空浏览一下 [neo-async](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsuguru03%2Fneo-async "https://github.com/suguru03/neo-async") 这个模块的 **[官方文档](https://link.juejin.cn?target=http%3A%2F%2Fsuguru03.github.io%2Fneo-async%2Fdoc%2Findex.html "http://suguru03.github.io/neo-async/doc/index.html")**。

### 总结

[promise-fun](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fpromise-fun "https://github.com/sindresorhus/promise-fun") 项目共收录了 **50** 个与 Promise 有关的模块，该项目的作者 **[sindresorhus](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus "https://github.com/sindresorhus")** 个人就开发了 **48** 个模块，不愧是全职做开源的大牛。由于篇幅有限，阿宝哥只介绍了其中 **10** 个比较常用的模块。其实该项目还包含一些挺不错的模块，比如 **[p-queue](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-queue "https://github.com/sindresorhus/p-queue")**、**[p-any](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-any "https://github.com/sindresorhus/p-any")**、**[p-some](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-some "https://github.com/sindresorhus/p-some")**、**[p-debounce](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-debounce "https://github.com/sindresorhus/p-debounce")**、**[p-throttle](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-throttle "https://github.com/sindresorhus/p-throttle")** 和 **[p-timeout](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Fp-timeout "https://github.com/sindresorhus/p-timeout")** 等。感兴趣的小伙伴，可以自行了解一下其他的模块。

### 参考资源

*   [MDN — Array.prototype.reduce()](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FArray%2FReduce "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce")
*   [MDN — Array.from](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FArray%2Ffrom "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/from")
*   [Promise.race with empty lists](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fdomenic%2Fpromises-unwrapping%2Fissues%2F75 "https://github.com/domenic/promises-unwrapping/issues/75")
*   [neo-async 官方文档](https://link.juejin.cn?target=http%3A%2F%2Fsuguru03.github.io%2Fneo-async%2Fdoc%2Findex.html "http://suguru03.github.io/neo-async/doc/index.html")
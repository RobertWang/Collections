> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/436757974)

前言
--

之前我们已经谈过了 fetch 劫持，但是由于那个代码相对较为复杂，cxxjackie 提供了一个相对简单的方式，并未使用 proxy 劫持，我们这节课来分析一下。

```
let oldfetch = fetch;
function fuckfetch() {
    return new Promise((resolve, reject) => {
        oldfetch.apply(this, arguments).then(response => {
            const oldJson = response.json;
            response.json = function() {
                return new Promise((resolve, reject) => {
                    oldJson.apply(this, arguments).then(result => {
                        result.hook = 'success';
                        resolve(result);
                    });
                });
            };
            resolve(response);
        });
    });
}
window.fetch = fuckfetch;
```

oldfetch 保存了原 fetch 的引用

这时候我们对 window.fetch 挂载成我们的劫持函数，fuckfetch

因为 fetch 需要返回一个 promise，所以这里我们通过

```
return new Promise((resolve, reject) => {})
```

包裹了一下，并且在原 fetch 函数内调用，并获取返回内容，对其进行一些处理并 resolve 返回过去。

附注: 函数一旦返回一个 Promise，我们只考虑输出相同的结果即可，而无须考虑 Promise 内处理过程的一致性。

```
oldfetch.apply(this, arguments).then(response => {
            const oldJson = response.json;
            response.json = function() {
                return new Promise((resolve, reject) => {
                    oldJson.apply(this, arguments).then(result => {
                        result.hook = 'success';
                        resolve(result);
                    });
                });
            };
            resolve(response);
        });
```

这里我们使用 fetch 的原函数，通过 apply 更改了 this 指针至自身，并且传入了参数。注意这点有一个需要注意的是，我们劫持函数的时候，由劫持函数调用原函数的过程中一定要使用 call/apply 进行修改 this 指针，来符合原来的调用过程。

在 then 后则是我们处理 response 的过程

```
const oldJson = response.json;
            response.json = function() {
                return new Promise((resolve, reject) => {
                    oldJson.apply(this, arguments).then(result => {
                        result.hook = 'success';
                        resolve(result);
                    });
                });
            };
```

这部分代码是针对某些特定的函数进行过滤，我们可以对网页进行分析以及调试，或去返回内容进行查看，来判断调用了哪些函数。

这里以 json 为例进行劫持

首先保存了原 json 的引用

然后修改 json 属性为一个劫持函数

由于 json 返回的是一个 promise 对象，所以我们这里也需要返回一个 promise

在 promise 内依然是对其原 json 函数进行调用，并修改了 this 指向以及参数，最后对其结果进行一定的修改，然后通过 resolve(result) 进行返回。

那么这节课我们就了解到了一个相对较为轻量的 fetch 劫持方法！

结语
--

撒花
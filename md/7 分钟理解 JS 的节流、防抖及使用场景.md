> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6844903669389885453) 。 糟了，这可触碰到我的知识盲区了，好像听也没听过这 2 个东西，痛定思痛，赶紧学习学习。here we go！

概念和例子
=====

### 函数防抖 (debounce)

> 在事件被触发 n 秒后再执行回调，如果在这 n 秒内又被触发，则重新计时。

看一个🌰（栗子）：

```
//模拟一段ajax请求
function ajax(content) {
  console.log('ajax request ' + content)
}

let inputa = document.getElementById('unDebounce')

inputa.addEventListener('keyup', function (e) {
    ajax(e.target.value)
})

复制代码
```

看一下运行结果：

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/9/4/165a252be5c94d6b~tplv-t2oaga2asx-watermark.awebp)

可以看到，我们只要按下键盘，就会触发这次 ajax 请求。不仅从资源上来说是很浪费的行为，而且实际应用中，用户也是输出完整的字符后，才会请求。下面我们优化一下：

```
//模拟一段ajax请求
function ajax(content) {
  console.log('ajax request ' + content)
}

function debounce(fun, delay) {
    return function (args) {
        let that = this
        let _args = args
        clearTimeout(fun.id)
        fun.id = setTimeout(function () {
            fun.call(that, _args)
        }, delay)
    }
}
    
let inputb = document.getElementById('debounce')

let debounceAjax = debounce(ajax, 500)

inputb.addEventListener('keyup', function (e) {
        debounceAjax(e.target.value)
    })
复制代码
```

看一下运行结果：

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/9/4/165a252b4b429b56~tplv-t2oaga2asx-watermark.awebp)

可以看到，我们加入了防抖以后，当你在频繁的输入时，并不会发送请求，只有当你在指定间隔内没有输入时，才会执行函数。如果停止输入但是在指定间隔内又输入，会重新触发计时。 再看一个🌰：

```
let biu = function () {
    console.log('biu biu biu',new Date().Format('HH:mm:ss'))
}

let boom = function () {
    console.log('boom boom boom',new Date().Format('HH:mm:ss'))
}


setInterval(debounce(biu,500),1000)
setInterval(debounce(boom,2000),1000)
复制代码
```

看一下运行结果：

这个🌰就很好的解释了，如果在时间间隔内执行函数，会重新触发计时。biu 会在第一次 1.5s 执行后，每隔 1s 执行一次，而 boom 一次也不会执行。因为它的时间间隔是 2s，而执行时间是 1s，所以每次都会重新触发计时

> ### 个人理解 函数防抖就是法师发技能的时候要读条，技能读条没完再按技能就会重新读条。

### 函数节流 (throttle)

> 规定在一个单位时间内，只能触发一次函数。如果这个单位时间内触发多次函数，只有一次生效。

看一个🌰：

```
function throttle(fun, delay) {
        let last, deferTimer
        return function (args) {
            let that = this
            let _args = arguments
            let now = +new Date()
            if (last && now < last + delay) {
                clearTimeout(deferTimer)
                deferTimer = setTimeout(function () {
                    last = now
                    fun.apply(that, _args)
                }, delay)
            }else {
                last = now
                fun.apply(that,_args)
            }
        }
    }

    let throttleAjax = throttle(ajax, 1000)

    let inputc = document.getElementById('throttle')
    inputc.addEventListener('keyup', function(e) {
        throttleAjax(e.target.value)
    })
复制代码
```

看一下运行结果：

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/9/4/165a252b4c1a9686~tplv-t2oaga2asx-watermark.awebp)

可以看到，我们在不断输入时，ajax 会按照我们设定的时间，每 1s 执行一次。

结合刚刚 biubiubiu 的🌰：

```
let biubiu = function () {
        console.log('biu biu biu', new Date().Format('HH:mm:ss'))
    }

    setInterval(throttle(biubiu,1000),10)

复制代码
```

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/9/4/165a252b46818296~tplv-t2oaga2asx-watermark.awebp)

不管我们设定的执行时间间隔多小，总是 1s 内只执行一次。

> ### 个人理解 函数节流就是 fps 游戏的射速，就算一直按着鼠标射击，也只会在规定射速内射出子弹。

总结
==

*   函数防抖和函数节流都是防止某一时间频繁触发，但是这两兄弟之间的原理却不一样。
*   函数防抖是某一段时间内只执行一次，而函数节流是间隔时间执行。

### 结合应用场景

*   debounce
    *   search 搜索联想，用户在不断输入值时，用防抖来节约请求资源。
    *   window 触发 resize 的时候，不断的调整浏览器窗口大小会不断的触发这个事件，用防抖来让其只触发一次
*   throttle
    *   鼠标不断点击触发，mousedown(单位时间内只触发一次)
    *   监听滚动事件，比如是否滑到底部自动加载更多，用 throttle 来判断

拓展
==

> 参考链接：[www.cnblogs.com/zichi/p/533…](https://link.juejin.cn?target=http%3A%2F%2Fwww.cnblogs.com%2Fzichi%2Fp%2F5331426.html "http://www.cnblogs.com/zichi/p/5331426.html")

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/9/4/165a252b4c074274~tplv-t2oaga2asx-watermark.awebp)

这是高程中的经典代码：

```
function throttle(method, context) {
        clearTimeout(method.tId);
        method.tId = setTimeout(function () {
            method.call(context);
        }, 100)
    }

复制代码
```

我们通过上面的例子知道，其实这段函数应该是 debounce 函数防抖，而不是函数节流，很多文章也都会拿这段代码来做例子，函数本身没错，但是命名错了。

原作者的这段话就写的很好，

> 就以 throttle 为例，某日，老师给你布置了一个作业，让你深入理解一下 throttle，第二天上课来聊聊。张三心里非常高兴，这个概念在经典书籍《JavaScript 高级程序设计》中见过，打开一看，就两页，而且解释地非常清晰，看完就高兴地干别的事情去了。而李四，觉得高程三讲的有点少，而去谷歌了下其他关于 throttle 的知识点，兴奋地看到 throttle 函数的好几种写法，发现高程三只是用了最简单的方式，还有更优雅运用场景更多的写法，或许此时他已经发现和 throttle 同时出现的还有个 debounce，这是什么鬼？反正老师没说，以后再看吧，于是心满意足地玩游戏去了。而王五，和李四一样发现了 debounce，这是什么？一起了解了吧，继而发现 debounce 的用法居然和高程三中的 throttle 一样！继续挖下去，发现高程三中的 throttle 函数其实应该叫 debounce，看到最后，王五已经把 throttle 和 debounce 彻底理解了。

我们要做王五，并且争取早日产出一手知识！加油！
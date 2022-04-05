> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA5NjUxMTM2MQ==&mid=2247491112&idx=1&sn=2d46783cfb194e07519cd3ce147b08df&chksm=90afaac6a7d823d07100eab1bdd68f540962ad3725a86f85fa6f744e0727de1d0efd8db4b193&mpshare=1&scene=1&srcid=0405hVuSJ1GrfiDoEUqcy2LF&sharer_sharetime=1649137288658&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

背景  

-----

大家开发中经常会跟 DOM 的事件打交道，也会经常用到`e.target`和`e.currentTarget`这两个对象，但是却有很多人根本就不知道这两个有什么区别~~~

冒泡 & 捕获
-------

当你触发一个元素的事件的时候，该事件从该元素的祖先元素传递下去，此过程为`捕获`，而到达此元素之后，又会向其祖先元素传播上去，此过程为`冒泡`

```
    <div id="a">
      <div id="b">
        <div id="c">
          <div id="d">哈哈哈哈哈</div>
        </div>
      </div>
    </div>
```

![](https://mmbiz.qpic.cn/mmbiz_png/TZL4BdZpLdhZtq6QIIcbqvEExAwZM0OvW3JERQSmchHibqTCQIqL30rgz433o0srNicXmU9eKL4jYlltfC2a7wAA/640?wx_fmt=png)

addEventListener
----------------

`addEventListener`是为元素绑定事件的方法，他接收三个参数：

*   第一个参数：绑定的事件名
    
*   第二个参数：执行的函数
    
*   第三个参数：
    

*   false：默认，代表冒泡时绑定
    
*   true：代表捕获时绑定
    

target & currentTarget
----------------------

### false

我们给四个 div 元素绑定事件，且`addEventListener`第三个参数不设置，则默认设置为`false`

```
const a = document.getElementById('a')
const b = document.getElementById('b')
const c = document.getElementById('c')
const d = document.getElementById('d')
a.addEventListener('click', (e) => {
  const {
    target,
    currentTarget
  } = e
  console.log(`target是${target.id}`)
  console.log(`currentTarget是${currentTarget.id}`)
})
b.addEventListener('click', (e) => {
  const {
    target,
    currentTarget
  } = e
  console.log(`target是${target.id}`)
  console.log(`currentTarget是${currentTarget.id}`)
})
c.addEventListener('click', (e) => {
  const {
    target,
    currentTarget
  } = e
  console.log(`target是${target.id}`)
  console.log(`currentTarget是${currentTarget.id}`)
})
d.addEventListener('click', (e) => {
  const {
    target,
    currentTarget
  } = e
  console.log(`target是${target.id}`)
  console.log(`currentTarget是${currentTarget.id}`)
})
```

现在我们点击，看看输出的东西，可以看出触发的是 d，而执行的元素是冒泡的顺序

```
target是d currentTarget是d
target是d currentTarget是c
target是d currentTarget是b
target是d currentTarget是a
```

### true

我们把四个事件第三个参数都设置为`true`，我们看看输出结果，可以看出触发的是 d，而执行的元素是捕获的顺序

```
target是d currentTarget是a
target是d currentTarget是b
target是d currentTarget是c
target是d currentTarget是d
```

### 区别

我们可以总结出：

*   `e.target`：**触发**事件的元素
    
*   `e.currentTarget`：**绑定**事件的元素
    

- EOF -

推荐阅读  点击标题可跳转

1、[React 18 这次是真的来了](http://mp.weixin.qq.com/s?__biz=MzA5NjUxMTM2MQ==&mid=2247490775&idx=1&sn=41ed22051da02541ef532f8b9f350b6d&chksm=90afa839a7d8212f781625265c0303e0f107d9a21b7ee8d45a54645f739db25fecca8008164f&scene=21#wechat_redirect)

2、[type 与 interface 的区别，你真的懂了吗？](http://mp.weixin.qq.com/s?__biz=MzA5NjUxMTM2MQ==&mid=2247490772&idx=1&sn=d264529ca782d02036c10fb7370f1255&chksm=90afa83aa7d8212c6b0485fd2a8324ff3499aaf64999211ce588f56e411e54041bf67bffa36c&scene=21#wechat_redirect)

3、[56 个 JavaScript 实用工具函数助你提升开发效率！](http://mp.weixin.qq.com/s?__biz=MzA5NjUxMTM2MQ==&mid=2247490778&idx=1&sn=ede786cff1f37df81fc351dece3e4419&chksm=90afa834a7d82122b0518962125c2acbac132d9156a0fa4d6f4d30077111eeb86b33c8782782&scene=21#wechat_redirect)
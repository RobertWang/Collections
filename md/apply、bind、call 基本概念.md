> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7008928728055021604) function fun() {     console.log("hello world") }; // call可以调用方法 fun.call(); 复制代码

输出为：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a2ec6631a1a04ca797df71e63c3dcb82~tplv-k3u1fbpfcp-watermark.awebp?)

我们可以看到 call 直接被调用了。说明函数的`call`方法可以直接`调用函数`。 除了可直接被调用往外，`call`还可以改变`this指向`，我们来看以下代码:

```
function fun() {
    console.log(this)
    console.log(this.name);
};
let cat = {
    name: "喵喵"
}
fun(); // 空
//call可以改变函数中this 指向
fun.call(cat); // 喵喵
复制代码
```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce6265131a1d429cbd5d4bbde83d4763~tplv-k3u1fbpfcp-watermark.awebp?)

我们可以看到第一次打印出来的结果是：`this`是指`Window`，`this.name`打印出来是`空` 第二次`call`运行输出的结果是：`this`指向的是`cat`对象，`this.name`打印出来是`喵喵`(`cat`的属性)

call 深入了解
---------

### this 指向修改

接下来我们实现一个`this`指向的更改，我们来看以下代码：

```
let dog = {
    name: "旺财",
    sayName() {
        console.log("我是" + this.name);
    }
}
let cat = {
    name: "喵喵"
}
// call可以调用函数,cal1可以改变函数中this
//指向
dog.sayName() // 我是旺财
dog.sayName.call(cat); // 我是喵喵
复制代码
```

输出为:

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/724ffad419104d72848a5c890f739c52~tplv-k3u1fbpfcp-watermark.awebp?)

我们可以看到当`sayName`方法被直接调用时，输出为`我是旺财` 当我们用`call`方法把`this`指向更改为`cat`的时候，`sayName`方法就输出为`我是喵喵`

### call 的其他传参

我们在`dog`对象中新添加一个方法`eat`:

```
eat(food) {
   console.log('吃' + food)
}
复制代码
```

正常调用的话：

```
dog.eat('骨头') // 吃骨头
复制代码
```

如果换成`call`调用我们就需要将传入的参数作为第二参数传入如下：

```
dog.eat.call(cat,'鱼'); // 吃鱼
复制代码
```

当我们有更多参数时候，我们可以在 call 方法的 2、3、4...n 个参数中传入：

```
dog.eat.call(cat,'鱼','老鼠'); // 吃鱼、老鼠 
复制代码
```

所以`call`的第一个参数为改变`this`指向的对象，第 2 到 n 个参数为所传得`参数`

apply、bind、call 的区别
-------------------

`apply`的参数列表是数组传入的，具体用法如下：

```
dog.eat.apply(cat,['鱼','老鼠']); // 吃鱼、老鼠 
复制代码
```

`bind`和 call 的区别在于`bind`不会直接调用方法，方法将是它的一个返回：

```
let fn = dog.eat.bind(cat,'鱼','老鼠'); 
fn(); // 吃鱼、老鼠 
复制代码
```

整理下这三者的区别在于：

<table><thead><tr><th>区别项</th><th>apply</th><th>bind</th><th>call</th></tr></thead><tbody><tr><td>第 2 参数</td><td>数组</td><td>按参数传</td><td>按参数传</td></tr><tr><td>方法调用</td><td>立即调用</td><td>返回方法</td><td>立即调用</td></tr></tbody></table>
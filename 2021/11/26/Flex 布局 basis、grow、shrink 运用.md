> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7005195208304361480) <div class="box"> <div class="item">1</div> <div class="item">222</div> <div class="item">3</div> </div> <style> .box { width: 400px; height: 50px ; display: flex; background-color: #eee ; } .item { height: 50px ; } .item:nth-child(1) { background: red; } .item:nth-child(2) { width: 70px; flex-basis: auto; background: grey; } .item:nth-child(3) { width: 50px; flex-basis: 100px; background: yellow; } </style> 复制代码

*   对于子项 1，`flex-basis` 如果设置默认是`auto`，子项占用的宽度使用`width` 的宽度，`width`没设置也为 `auto`，所以子项占用空间由内容决定。
*   对于子项 2，`flex-basis` 为`auto`，子项占用宽度使用`width` 的宽度，`width` 为`70px`，所以子项子项占用空间是`70px`。
*   对于子项 3，`flex-basis` 为`100px`，覆盖`width` 的宽度，所以子项占用空间是`100px`。

flex-grow
---------

用来 “瓜分” 父项的“剩余空间”。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/504e296e295048afb4611a8623bc8cd4~tplv-k3u1fbpfcp-watermark.awebp)

```
<div class="box">
  <div class="item">1</div>
  <div class="item">222</div>
  <div class="item">3</div>
</div>
<style>
.box {
  width: 400px;
  height: 50px;
  display: flex;
  background-color: #eee;
}
. item {
	height: 50px;
}
.item:nth-child(1) {
  width: 50px; 
  background: red;
}
.item:nth-child(2) {
  width: 70px;
  flex-basis: auto;
  flex-grow: 2;
  background: grey;
}
.item:nth-child(3) {
  width: 50px;
  flex-basis: 100px;
  flex-grow: 1;
  background: yellow;
}
</style>
复制代码
```

容器的宽度为 400px, 子项 1 的占用的基础空间 (flex-basis) 为 50px，子项 2 占用的基础空间是 70px，子项 3 占用基础空间是 100px，剩余空间为 `400-50-70-100=180px`。 其中子项 1 的`flex-grow`: 0(未设置默认为 0)， 子项 2`flex-grow`: 2，子项 3`flex-grow`: 1，剩余空间分成 3 份，子项 2 占 2 份 (120px)，子项 3 占 1 份 (60px)。所以 子项 1 真实的占用空间为: `50+0=50px`， 子项 2 真实的占用空间为: `70+120=190px`， 子项 3 真实的占用空间为: `100+60=160px`。

flex-shrink
-----------

用来 “吸收” 超出的空间

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f7adf82cb1a34e6a975c1940e7a06997~tplv-k3u1fbpfcp-watermark.awebp)

```
<div class="box">
  <div class="item">1</div>
  <div class="item">222</div>
  <div class="item">3</div>
</div>
<style>
.box {
  width: 400px;
  height: 50px;
  display: flex;
  background-color: #eee;
}
.item {
	height: 50px;
}
.item:nth-child(1) {
  width:250px;
  background:red;
}
.item:nth-child(2) {
  width:150px;
  flex-basis: auto;
  flex-shrink: 2;
  background: grey ;
}
.item:nth-child(3) {
  width: 50px;
  flex-basis: 100px;
  flex-shrink: 2;
  background: yellow;
}
</style>
复制代码
```

容器的宽度为 400px, 子项 1 的占用的基准空间`flex-basis`为`250px`，子项 2 占用的基准空间是`150px`，子项 3 占用基准空间是`100px`，总基准空间为`250+150+100=500px`。容器放不下，多出来的空间需要被每个子项根据自己设置的`flex-shrink` 进行吸收。 子项 1 的`flex-shrink: 1`(未设置默认为 1)， 子项 2 `flex-shrink: 2`，子项 3 `flex-shrink: 2`。子项 1 需要吸收的的空间为 `(250*1)/(250*1+150*2+100*2) * 100 = 33.33px`，子项 1 真实的空间为 `250-33.33=216.67px`。同理子项 2 吸收的空间为`(150*2)/(250*1+150*2+100*2) * 100=40px`，子项 2 真实空间为 `150-40 = 110px`。子项 3 吸收的空间为`(100*2)/(250*1+150*2+100*2) * 100=26.67px`，真实的空间为`100-26.67=73.33px`。
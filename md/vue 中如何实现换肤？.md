> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6966568597418147877) $background-main-color1:#E9EDF0; // 背景色 $background-main-color2:#011E30; // 背景色 复制代码

解析文件 mixin.scss

```
@import "./theme.scss";    //引入声明的皮肤文件
@mixin background-main-color($color){ //@mixin 后面的函数名称为自定义
    background-color: $color;   //背景色默认为参数
    [background-main-color="background-main-color2"] & {    //如果条件成立，背景色则用$background-main-color2
        background-color: $background-main-color2;    //这个$background-main-color2已经在theme.scss中定义过了。  
    }
}
复制代码
```

页面中使用 xx.vue（全局或页面引入 mixin.scss）

```
<template>
  <div class="container">
    <button @click="changeTheme">changeTheme</button>
  </div>
</template>

<script>
export default {
    methods: {
        changeTheme() {
            //第一个"background-main-color" 指的是mixin中我们自定义声明的名称，"background-main-color2"指的是我们传的参数，相当于mixin中的 $color，如果条件成立则会用下面的样式。
            window.document.documentElement.setAttribute("background-main-color","background-main-color2");
        }
    }
}
</script>

<style>
@import '@/assets/style/mixin.scss';
.container{
    //background-main-color 为mixin中定义的名称，括号里为theme中声明的值。解析为 background:"#E9EDF0"
    @include background-main-color($background-main-color1);
}
</style>
复制代码
```

初来乍到，小伙伴们赶紧点赞支持一下把~
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7022836132224172069) npm install vue-contextmenujs 复制代码

或

```
yarn add vue-contextmenujs
复制代码
```

CDN
---

```
<script src="https://unpkg.com/vue-contextmenujs/dist/contextmenu.umd.js">
复制代码
```

### 使用

```
import Contextmenu from "vue-contextmenujs"
Vue.use(Contextmenu);
复制代码
```

CDN 引入则不需要`Vue.use(Contextmenu)`

### 代码实现

以`element-ui`图标为例实现右键菜单，图标会为被渲染为`<i></i>`，代码如下：

```
<template>
  <div style="width:100vw;height:100vh" @contextmenu.prevent="onContextmenu"></div>
</template>

<script>
import Vue from 'vue'
import Contextmenu from "vue-contextmenujs"
Vue.use(Contextmenu);
export default {
  methods: {
    onContextmenu(event) {
      this.$contextmenu({
        items: [
          {
            label: "返回(B)",
            onClick: () => {
              this.message = "返回(B)";
              console.log("返回(B)");
            }
          },
          { label: "前进(F)", disabled: true },
          { label: "重新加载(R)", divided: true, icon: "el-icon-refresh" },
          { label: "另存为(A)..." },
          { label: "打印(P)...", icon: "el-icon-printer" },
          { label: "投射(C)...", divided: true },
          {
            label: "使用网页翻译(T)",
            divided: true,
            minWidth: 0,
            children: [{ label: "翻译成简体中文" }, { label: "翻译成繁体中文" }]
          },
          {
            label: "截取网页(R)",
            minWidth: 0,
            children: [
              {
                label: "截取可视化区域",
                onClick: () => {
                  this.message = "截取可视化区域";
                  console.log("截取可视化区域");
                }
              },
              { label: "截取全屏" }
            ]
          },
          { label: "查看网页源代码(V)", icon: "el-icon-view" },
          { label: "检查(N)" }
        ],
        event, // 鼠标事件信息
        customClass: "custom-class", // 自定义菜单 class
        zIndex: 3, // 菜单样式 z-index
        minWidth: 230 // 主菜单最小宽度
      });
      return false;
    }
  }
};
</script>
复制代码
```

菜单选项都在`items`数组里面，可根据需要自行配置。这时候点击右键，菜单弹窗就神奇地出现了，是不是很简单!

### 自定义样式

打开控制台，查看元素即可查看到菜单的各个 class 名称。最外层的 class 为上面的`customClass`属性设置的值，样式可根据需求自行调整。

```
<style>
.custom-class .menu_item__available:hover,
.custom-class .menu_item_expand {
  background: #ffecf2 !important;
  color: #ff4050 !important;
}
</style>
复制代码
```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca3c62b8650b465088da89e63ae6447c~tplv-k3u1fbpfcp-watermark.awebp?)

### 总结

以上就基本使用方法，是不是比自己封装节省了大把时间。注意菜单会在点击左键或者滚轮滚动时自动销毁，同时也可调用`this.$contextmenu.destroy()`在其他场景自行销毁 。以下是插件的参数配置：

#### MenuOptions 菜单属性

```
菜单结构信息
```

#### MenuItemOptions 选项属性

```
菜单项名称
```

欢迎在评论区留言，掘金官方将在[掘力星计划](https://juejin.cn/post/7012210233804079141 "https://juejin.cn/post/7012210233804079141")活动结束后，在评论区抽送 100 份掘金周边。如果文章对你有所帮助，不要忘了点个赞~

听说点赞的人今年人品爆发，年终奖翻倍🤩
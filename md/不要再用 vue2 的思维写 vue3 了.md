> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651574367&idx=1&sn=75b6a14189036ff234b207c44758e7c0&chksm=8025079eb7528e880ec6b80750c96a35b1ba1e198128521fe5e6a0797aec8dff84752b769e88&scene=21#wechat_redirect)

↓推荐关注↓

公众号

升级 Vue3 后，让人最脑壳疼的应该是新的 Compostion API 语法，他的难点不是语法，而是他提供了全新的组织代码的思维方式。

踩过的坑
----

### 1. 按技术类型划分代码

在日常开发中，前端一般会收到交互稿或设计稿后开始布局，然后编写逻辑代码。在 Vue2 中，通常做法是响应数据放到`data`、逻辑方法放到`methods`，这样的做法非常方便，也让我们很容易组织代码。

当使用 vue3 的 Compostion API 时，如果还是用 Vue2 的形式组织代码，这不但不会提升代码质量，反而**因为缺乏约束**而降低可读性。

我在 github 随便找了一段代码，你觉得这段代码比 Vue2 简洁吗？

```
export default {
  setup () {
    const state = reactive({
      message: '',
      msgList: []
    })
    const router = useRouter()
    let username = ''
    onMounted(() => {
      username = localStorage.getItem('username')
      if (!username) {
        router.push('/login')
        return
      }
    })
    const onSendMessage = () => {
      const { message } = state
      if (!message.trim().length) return
      state.msgList.push({
        id: new Date().getTime(),
        user: username,
        dateTime: new Date().getTime(),
        message: state.message
      })
      state.message = ''
    }
    return {
      ...toRefs(state),
      onSendMessage
    }
  }
}


```

实际上我们过于关注语法层面改变，而忽略官方文档提到一个词叫：**`逻辑关注点！！！！！！`**, 逻辑关注点是指表达同一个业务的代码内聚到一起，这也是单一职责的指导思想，我们内聚的不应该技术类型，而是业务逻辑，因为触发代码变更的往往是业务需求，因此把相同变更理由的代码放在一起，这才不会导致散弹式修改。

### 2. 过于关注逻辑复用

compostion API 一个特点是提升逻辑复用，这是没有错的，但是当时我有一个错误观点，就是只有复用的逻辑才应该封装到 hook 中。

我们还是回到 Vue 的官方例子，你会发现他把原来放在一个 vue 文件的逻辑拆分到`composables`目录，目录下分别定义一个文件，表示不同的`逻辑关注点`。

![](https://mmbiz.qpic.cn/mmbiz_png/zPh0erYjkib0PJ0npnc8eKwRtF7kFRRTyCk3lgYf0w7nAIArCavk5jr1Npm0mddIyEsrztXPwd00H7mJTLp3MQg/640?wx_fmt=png)

image.png

官方文档地址 [1] | 参考代码仓库 [2]

这个文件夹的代码强调的并`不是逻辑复用`，而是`逻辑关注点`分离，这也是 compostion API 最核心要解决的问题，因为应用生命周期 60% 时间都是在维护的，而维护性体现在代码是否符合单一职责原则，单一职责就是把相同的业务代码内聚到一个地方。

所以你不要过于纠结代码是否需要复用，应用适当的冗余反而增加应用的维护性，《架构整洁之道》书中提到：**对于大多数应用，可维护性比可重用性更加重要。**

如果你的代码真的具有很高的复用性，那可以提升到项目外层目录，封装到独立的 hook 文件。

尤雨溪的看法
------

compostion API 在提案的时候，就有很多人持有不同意见，有反对有支持，实际上都没有错，只是大家碰到的场景不同而导致不同观点。我通过阅读 compostion API 的 RFC，找到了作者对一些问题的解答，整理了一些关键问题，内容不是完全翻译，完整内容建议查看原文。原文地址 [3]

### 问题一：compostion api 根本没有解决任何问题，只是追逐新玩意的东西

**尤雨溪：** 不同意这个观点。Vue 最开始很小，但是现在被广泛应用到不同级别复杂度的业务领域，有些可以基于 option API 很轻松处理，但是有些不可以。例如下面的场景：

1.  有很多逻辑的大型组件（数百行）
    
2.  在多个组件可复用的逻辑
    

对于问题 1，你需要把每个逻辑拆分到不同选项，例如，一段逻辑需要一些响应数据，一个计算属性，一些监听属性还有方法。你去了解这段逻辑时，需要不断上下移动阅读，虽然你知道一些属性是什么类型，但是你并不知道他具体的作用。当一个组件包含多个逻辑，情况就更糟糕了。如果用新的 API，可以将数据和逻辑组合在一起，最重要的是，你可以`干净的把这些逻辑提取到一个函数，甚至一个单独的文件中。`

### 问题二：使用新 API 导致逻辑分散到不同地方，违背 "关注点分离"

**尤雨溪：** 这个问题和项目文件组织方式问题类似。我们很多人都同意按文件类型组织（布局放 HTML，样式 CSS，逻辑 JS）并不是正确的方式，因为强制把相关代码分割到三个文件，只是给人一种 “关注点分离” 的错觉。这里的关键是 “关注点” 不是由文件类型定义。相反，我们大多数选择以`功能或者职责`来组织文件，`这正是人们喜欢Vue单文件组件的原因`。SFC 就是按功能组织代码的方法，但讽刺的是当首次引入 SFC 时，许多人也是拒绝的，认为它违反了关注点分离。

### 问题三：新的语法让 Vue 失去简单性，导致 "意大利面条式代码" 的出现，降低项目维护性。

**尤雨溪：** 正好相反，新的 API 就是为了提高项目长期维护性的。

如果我们查看任何 javascript 项目，都会从入口文件开始阅读，该文件的本质是你的应用启动时被隐式调用的 "main" 函数。如果只有一个函数入口，会导致意大利面条代码，那所有的 js 项目都是意大利面条代码。显然不是的，因为开发人员通过代码模块化或者较小的函数来组织代码。

另外，我同意新的 API 理论上会降低代码质量的最低门槛。但是我们可以使用以往防止代码变成意大利面条的手段缓解这种情况。另一方面，新的 API 可以提升代码质量的最高上限，相比 option api，你可以重构为质量更高的代码。而且，基于 Option api 你还得解决类似 mixins 的问题。

很多人认为 "Vue 失去简单性"，实际上只是失去组件内代码类型检查能力（就是你不知道一个变量时 data、method、还是 computed）。但是用新的 API，实现一个类型检测器也是非常容易实现以前的特性的。也就是说，你不应该被 option api 限制思维，而更多关注逻辑内聚问题。

总结
--

上面只是节选了 RFC 讨论的几个小问题，如果你对新 API 还有其他疑问，建议去 github 阅读原文，原文讨论了非常多问题，我就不一一总结了。

但是从讨论的内容和我实战的经验，用新的 API，一定要注意转变代码组织思维，记住一个词`"逻辑关注点"`。

### 参考资料

[1]

官方文档地址: _https://v3.cn.vuejs.org/guide/composition-api-introduction.html#%E4%BB%80%E4%B9%88%E6%98%AF%E7%BB%84%E5%90%88%E5%BC%8F-api_

[2]

参考代码仓库: _https://github.com/barnett617/composition-api-demo/blob/292cc63e2e/src/components/UserRepositoriesV3.vue_

[3]

原文地址: _https://github.com/vuejs/rfcs/issues/55_

> 作者：  蟹老板爱写代码 
> 
> https://juejin.cn/post/6946387745208172558

- EOF -

推荐阅读  点击标题可跳转

1、[Vue3.0 新特性以及使用经验总结](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651573081&idx=1&sn=318e9bd12e09b3bcd74abb82563a059c&chksm=80251c98b752958e9d616b1a8531ff03b095a303f41007556efbf49e2b9ec97048de58b0fd26&scene=21#wechat_redirect)

2、[](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651573240&idx=1&sn=3241251fd22402b8920a2ec524b6d5b6&chksm=80251c39b752952f57f4a1af9f01b6223c0bcd211450dc68f938822e403691f494f8770e3706&scene=21#wechat_redirect)[104 道 CSS 面试题，助你查漏补缺（上）](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651563264&idx=1&sn=ec486cabe5a0d62a1a3026f4d17b4dee&chksm=802572c1b752fbd7b5c38a200c57ff8707743039e5fb92d9eb30cedb6b53a2494b67f35c84b6&scene=21#wechat_redirect)

3、[手撕 32 个面试高频知识，轻松应对编程题](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651563026&idx=1&sn=d35e79f90e7eac9a65dca30f22ea71bf&chksm=802573d3b752fac57a916932c75d8f6a5df1bd073b9194fa77989fe44e73da431095f37e59ec&scene=21#wechat_redirect)

公众号

点赞和在看就是最大的支持❤️
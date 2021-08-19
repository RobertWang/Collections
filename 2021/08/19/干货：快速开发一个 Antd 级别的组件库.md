> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651557832&idx=1&sn=52264a735917292b386f91aef6202924&chksm=80255809b752d11f1d8c82ba04b7e793c9e92ae09de8935a55ef549f0d4e280ae386c856d1c1&scene=21#wechat_redirect)

> 作者：FlyTeng_1874
> 
> https://juejin.im/post/5df062da6fb9a0162a0b6e58

**始**  

--------

很多时候，我们为了复用或者归纳总结，会把组件抽离出来发到`npm`上。但是这个过程你会发现一个问题，就是应该怎么更好的`发布`和`管理`维护这些组件呢。最后会发现网上的其他教程不是太零散了，就是有些细节不大到位。这里借这个机会好好总结一下，要是看完觉得有帮助的话，不妨个`赞`，关注一下哈哈。

理解完之后我们就可以
----------

1.  编写发布一个可用的组件库
    
2.  能以`import { demoComponent } from 'xxxUI'`方式引入
    
3.  也能以`import demoComponent from xxxUI/component/demoComponent`方式引入
    
4.  各个组件`打包相对独立`，互不干扰
    
5.  输出的组件能够`简单易用`和具有良好的`兼容性`
    
6.  组件库能根据用户的配置实现`按需加载`
    
7.  组件库能根据用户的配置实现`Tree Shaking`
    
8.  组件通过`单元测试`
    
9.  打包发布到`npm`
    

就像这样，嘿嘿。

![](https://mmbiz.qpic.cn/mmbiz/iawKicic66ubH73bPe9BrUg9TLfaDk0S94gib7TzPoKSAmNXyxEicxyukxaF50eXEBbt8mbEQIILTxxPZKV71WL1hicA/640?wx_fmt=other)

> 以下都以 react 组件库为例，其实 vue 是也一样的，只是 babel 配置有所区别

**项目结构**
--------

### **结构解析**

先来看看组件库的项目结构

![](https://mmbiz.qpic.cn/mmbiz_jpg/iawKicic66ubH73bPe9BrUg9TLfaDk0S94gtnQY82twYEXLQQPibu5veib8ibMQDJZicgLibuiciaCZWjyUnoL1wSbSgMFuQ/640?wx_fmt=jpeg)

嗯，看起来很是复杂，第一印象应该都在想着都是些什么乱七八糟的文件，下面先来解释一下。

*   `src`存放核心代码
    
*   `dist`存放最后打包输出的代码
    
*   `sass`样式单独抽离放置（当然可以跟组件放一起，这里的目的是即使不用相关的组件，单独使用相关样式也是没问题）
    
*   `__mocks__`（mock 对象），`coverage`（覆盖率）,`test,jest.config.js`（jest 配置）这些都是与单元测试相关的下一章会有详细介绍
    
*   `.``npmignore`与`.``gitignore`作用类似
    
*   `.``babelrc`大名鼎鼎的 babel 应该都知道的
    
*   其他应该都非常熟悉了，再介绍下去就有凑字数的嫌疑了。
    

### 关键目录

我们先把目光聚焦到`src`核心代码目录下，首先我们将组件存放在`component`中，在外层用`index`去引用`component`中的组件，由于在不提供具体路径的情况下，`import`引入时会默认找到`index`。这样在打包输出后，就能通过`import { demoComponent } from 'xxxUI'`这种方式去引用组件了。

![](https://mmbiz.qpic.cn/mmbiz_png/iawKicic66ubH73bPe9BrUg9TLfaDk0S94goDXc8Gmibr7q3dG79o7ECygEd90A6jlyvAJ8lsYVEDhXNTAWpFbPibAA/640?wx_fmt=png)

然后使用这种形式去导出组件，就能通过`import demoComponent from xxxUI/component/demoComponent`这种形式单独引入组件了。

### 组件库按需加载

根据以上目录结构和引入方式我们可以知道，通过`import { demoComponent } from '``xxxUI``'`这种形式去引入会使得整个组件库都引入到开发项目中，有时只需要用到其中的两三个组件，这种情况是我们不想看到的。而通过`import demoComponent from` `xxxUI/component/demoComponent`这种形式去引用，就能做到只引入某个需要用到的组件，这刚好能解决这个问题。但是每次引入都要写这么长的一串，很不方便。这个时候就需要用到`babel-plugin-import`这个插件了。

```
import { demoComponent, demoComponent1, demoComponent2 } from 'xxxUI'


// 使用
babel-plugin-import
插件能自动将以上这种调用形式在
AST
(抽象语法树)中改写成以下形式。
// 这样就能方便地引入相关组件，又不用担心一次全部引入导致包过大的问题
```

![](https://mmbiz.qpic.cn/mmbiz_png/iawKicic66ubH73bPe9BrUg9TLfaDk0S94gUSX2VSvOdDWYaiaCrZjYCURt17Y3zA2VDjR4oVdj35QytFy3EAj5eyg/640?wx_fmt=png)

> 需要注意的是，这里需要组件库的使用者去配置，而不是写在组件库的`.babelrc`中。如果组件库支持按需加载，这个配置应该写在`README.md`中交由组件库的使用者去选择。按需加载的好坏处是由具体的项目环境而定，`需要具体情况具体分析`。

![](https://mmbiz.qpic.cn/mmbiz/iawKicic66ubH73bPe9BrUg9TLfaDk0S94ge9jBXMUS0ebxoVsicuU1lkAj1P69ZG7hmGWJkcr9ibOaVVIkRsrM7PrA/640?wx_fmt=other)

没设置按需加载时，整个组件库都打包进去了。

![](https://mmbiz.qpic.cn/mmbiz/iawKicic66ubH73bPe9BrUg9TLfaDk0S94grKhXtpl2WK4xnFFM73opOJ243TibK2VJU6OFbKuFKyJa8yMLK3ypzLQ/640?wx_fmt=other)

设置了按需加载，只加载用到的组件。

就这样，通过巧妙的文件结构，目标`2，3，6`已达成。

**输入**
------

明确了项目结构，接下来就是需要收集组件源码了。通常来讲，只需要在`webpack`的`entry`配置中只需要设置入口文件`index`就可以了，就像这样`entry: path.resolve(__dirname, 'src', 'index.jsx``')`。但由于我们需要每个组件互相独立单独打包，所以需要一个个组件去引入，同时也要保持相应的文件结构。

![](https://mmbiz.qpic.cn/mmbiz_png/iawKicic66ubH73bPe9BrUg9TLfaDk0S94gT13jwmcgkWyQf3VycA9HiaWib4OsEmJ0omBYhEK79Xf5qibiabnicJIDNGw/640?wx_fmt=png)

在这里使用了`glob`这个很好用的工具，能很方便匹配出对应的文件。最后返回的是一个文件路径的映射对象，我们可以在控制台看看输入了哪些文件。

![](https://mmbiz.qpic.cn/mmbiz_jpg/iawKicic66ubH73bPe9BrUg9TLfaDk0S94gR27XlZqLPjTvnR8BnrpiahSRVGI255QwfMzQu1AeuRibg1qehrcROIpw/640?wx_fmt=jpeg)

ok，接下来就是要怎样处理这些源文件了。

**编译处理和组件库** **Tree Shaking**
-----------------------------

这里的处理过程很简单，逻辑就是配置`babel`将`es6+`的源码处理成`es5`的兼容代码，顺便也将`svg`小图标转化为`base64`格式嵌入。这样做更多是为了让用户以尽量小的配置，尽量小的上手成本就能使用这个组件库。这里如果同时保留了`es6`代码，就能够让让开发者可以自由配置`Tree Shaking`了（比如开发者只用到了某个组件中的某个方法的场景下，就没必要引入整个组件了）。关于开发者如何配置`Tree Shaking`最后会讲到。

> Es6 Modules 从语法层面提供了模块化功能，`Tree Shaking`就是基于 ES6 模块化的，在编译打包节点可以在`AST`（抽象语法树）中静态分析，将没有用到的代码剔除掉。我们经过编译打包后的 es5 代码是无法进行`Tree Shaking`的。

![](https://mmbiz.qpic.cn/mmbiz_png/iawKicic66ubH73bPe9BrUg9TLfaDk0S94gCY1Neciay0hg86LFlicNovrgSZwMzxfic3IZNicHW5eRVwwwNoDmKD4Y2w/640?wx_fmt=png)

**达成目标的第 7 点。**

**输出**
------

打包编译输出到`dist`目录，要注意的是`dist`目录中的结构要与`src`目录保持一致才能使组件和组件间的引用路径不会乱，就像这样，`dist`目录结构跟`src`相似。

![](https://mmbiz.qpic.cn/mmbiz_jpg/iawKicic66ubH73bPe9BrUg9TLfaDk0S94gRicSs9RQkdKRrOibCNX9PdHw0X8kibxkJZVgNCMukvicYeDCSP1DFzphgA/640?wx_fmt=jpeg)

再来看看`output`的配置，由于我们在文件输入时保持了文件路径信息，所以这里直接更改后缀之后输出到 dist 即可。`libraryTarget`的作用在于设置打包格式，这里采用`umd`标准。如果设置了`library`，那么将会导出成单入口的引用形式`import xxxUI from 'xxxUI``'`，这是我们不希望的。`library`与`libraryTarget`的取值根据项目类型的不同而不同。

![](https://mmbiz.qpic.cn/mmbiz_png/iawKicic66ubH73bPe9BrUg9TLfaDk0S94gfkGP5qYpBb2XYCUzXl3IdRUbz2AyFcPXjZiaic6YHofoic7OTVH43kX1Q/640?wx_fmt=png)

万事俱备了，但按照这样打包后会发现，怎么第三方包`react,react-dom`也跟着打包进来了，这会导致打包之后组件库的体积很大。

![](https://mmbiz.qpic.cn/mmbiz/iawKicic66ubH73bPe9BrUg9TLfaDk0S94gV2oWntF7xwEicVEUTUr2PpCic6ukE79cgoQsVCvdtJpIp9icz2YE7mZBw/640?wx_fmt=other)

我们需要这样去配置，过滤掉`import`进来的第三方包

![](https://mmbiz.qpic.cn/mmbiz_png/iawKicic66ubH73bPe9BrUg9TLfaDk0S94glYuogvrWHTntSjj8U05kQkzBwLXIelPO40WUdkmDoPYrLU76uttlYg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz/iawKicic66ubH73bPe9BrUg9TLfaDk0S94g2bWtLzzzK6KneoyBAIId7xTzoMjediaTt0icQLK0EKZkLr86S96o1KXg/640?wx_fmt=other)

可以看到变化巨大！现在整个包大小只有`120kb`（除去样式）

由于样式是`独立抽离`出来的，只需要将样式 copy 到`dist`目录即可，当然可配置插件自动完成。

```
new CopyPlugin([{
from: './sass',
    to: './sass'
}])
```

**达成目标的****`4，5`点**  

**最终发布**
--------

1.  先去官网完成注册
    
2.  `npm login`登录 (这里一定要先切换到国外镜像源)
    
3.  添加`.``npmignore`文件，将需要忽略的文件列出来
    
4.  添加`README``.``md`，写出必要的说明，这是一个好习惯
    
5.  在`package.json`的`script`中添加命令`w``ebpack --mode production && npm publish ./dist`。这里意思是采用生产模式打包并将`dist`目录发布上`npm`。
    

到最后`README.md`使用手册可以这样写

> babel 中 modules 的选项有`'amd' | 'umd' | 'systemjs' | 'commonjs' | false`这几个，由于 Tree Shaking 基于 ES6 Modules，这里就不能转换成其他标准，只能选`false`，即采用原本文件的模块标准编译。

搞定，一个实用的组件库就发布完成了，快来动手试试吧。
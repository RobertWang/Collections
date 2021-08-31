> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/elesos/article/details/81872854)

跨平台一直是老生常谈的话题，cordova、ionic、react-native、weex、kotlin-native、flutter 等跨平台框架百花齐放，颇有一股推倒原生开发者的势头。

为什么我们需要跨平台开发？ 本质上，跨平台开发是为了增加代码复用，减少开发者对多个平台适配的工作量，降低开发成本，提高业务专注的同时，提供比 web 更好的体验。

目前移动端跨平台开发中，备受关注的方案大致归纳为以下几种情况：

*   1）react native、weex 均使用 JavaScript 作为编程语言，目前 JavaScript 在跨平台开发中，可谓占据半壁江山，大有 “一统天下” 的趋势；
*   2）kotlin-native 开始支持 iOS 和 Web 开发，（kotlin 已经成为 android 的一级语言）也想尝试 “一统天下”；
*   3）flutter 是 Google 跨平台移动 UI 框架，Dart 作为谷歌的亲儿子，毫无疑问 Dart 成为 flutter 的编程语言。作为巨头新生儿，在 flutter 官网也可以看出，flutter 同样 “心怀天下”（可支持 Web 端、Android 端、iOS 端等）。

本篇主要以 react-native、weex、flutter，深入聊聊当前最火的这 3 种跨平台移动开发方案的实现原理、现状与未来。至于为什么只讲它们，因为对比 ionic、phoneGap，它们更于 “naive” (˶ ⁻̫ ˵)。看完本篇，相信你会对于当下跨平台移动开发的现状、实现原理、框架的选择等有更深入的理解。

2、React Native 的原理与特性介绍

React Native 技术关键词：

*   1）Facebook 出品；
*   2）JavaScript 语言；
*   3）JSCore 引擎；
*   4）React 设计模式；
*   5）原生渲染。

2.1 理念架构

“Learn once, write anywhere” ，代表着 Facebook 对 react native 的定义：学习 react ，同时掌握 web 与 app 两种开发技能。 react native 用了 react 的设计模式，但 UI 渲染、动画效果、网络请求等均由原生端实现。开发者编写的 js 代码，通过 react native 的中间层转化为原生控件和操作，比 ionic 等跨平台应用，大大提高了的用户体验。

总结起来其实就是：React Native 是利用 JS 来调用 Native 端的组件，从而实现相应的功能。

如下图所示，react native 的跨平台实现主要由三层构成，其中 C++ 实现的动态连结库 (.so)，作为中间适配层桥接，实现了 js 端与原生端的双向通信交互。这里最主要是封装了 JavaScriptCore 执行 js 的解析，而 react native 运行在 JavaScriptCore 中，所以不存在浏览器兼容的问题。

其中在 IOS 上直接使用内置的 javascriptcore， 在 Android 则使用 webkit.org 官方开源的 jsc.so。

1.2、实现原理

 和前端开发不同，react native 所有的标签都不是真实控件，JS 代码中所写控件的作用，类似 Map 中的 key 值。JS 端通过这个 key 组合的 Dom ，最后 Native 端会解析这个 Dom ，得到对应的 Native 控件渲染，如 Android 中 <view> 标签对应 ViewGroup 控件。

 在 react native 中，JS 端是运行在独立的线程中（称为 JS Thread ）。JS Thread 作为单线程逻辑，不可能处理耗时的操作。那么如 fetch 、图片加载 、 数据持久化 等操作，在 Android 中实际对应的是 okhttp 、Fresco 、SharedPreferences 等。而跨线程通信，也意味着 Js Thread 和原生之间交互与通讯是异步的。

 可以看出，跨平台的关键在于 C++ 层，开发人员大部分时候，只专注于 JS 端的代码实现。 在原生端提供的各种 Native Module 模块（如网络请求，ViewGroup 控件），和 JS 端提供的各种 JS Module（如 JS EventEmiter 模块），都会在 C++ 实现的 so 中保存起来，双方的通讯通过 C++ 中的保存的映射，最终实现两端的交互。通信的数据和指令，在中间层会被转为 String 字符串传输，

1.3、打包加载

 最终，JS 代码会被打包成一个 bundle 文件，自动添加到 App 的资源目录下。react native 的打包脚本目录为 / node_modules/react-native/local-cli，打包最后会通过 metro 模块压缩 bundle 文件。而 bundle 文件只会打包 js 代码，自然不会包含图片等静态资源，所以打包后的静态资源，其实是被拷贝到对应的平台资源文件夹中。

 其中图片等存在资源的映射规则，比如放在 react native 项目根目录下的 img/pic/logo.png 的资源，编译时，会被重命名后，根据大小 merged 到对应的是 drawable 目录下，修改名称为 img_pic_logo.png。

 打包 Android 和 IOS，肯定需要相应的平台项目存在，在 react-native init 时创建的项目，就已经包含了 android 和 ios 的模版工程，打包完的工程会加载 bundle 文件，然后启动项目

2、WEEX

Alibaba 出品，JavaScript 语言，JS V8 引擎，Vue 设计模式，原生渲染

2.1、理念架构

 “Write once, run everywhere”, weex 的定义就像是：写个 vue 前端，顺便帮你编译成性能还不错的 apk 和 ipa（当然，现实有时很骨感）。基于 Vue 设计模式，支持 web、android、ios 三端，原生端同样通过中间层转化，将控件和操作转化为原生逻辑来提高用户体验。

 在 weex 中，主要包括三大部分：JS Bridge、Render、Dom，分别对应 WXBridgeManager、WXRenderManager、WXDomManager，三部分通过 WXSDKManager 统一管理。其中 JS Bridge 和 Dom 都运行在独立的 HandlerThread 中，而 Render 运行在 UI 线程。

 JS Bridge 主要用来和 JS 端实现进行双向通信，比如把 JS 端的 dom 结构传递给 Dom 线程。Dom 主要是用于负责 dom 的解析、映射、添加等等的操作，最后通知 UI 线程更新。而 Render 负责在 UI 线程中对 dom 实现渲染。

2.2、实现原理

 和 react native 一样，weex 所有的标签也不是真实控件，JS 代码中所生成存的 dom，最后都是由 Native 端解析，再得到对应的 Native 控件渲染，如 Android 中 <text> 标签对应 WXTextView 控件。

 weex 中文件默认为 .vue ，而 vue 文件是被无法直接运行的，所以 vue 会被编译成 .js 格式的文件，Weex SDK 会负责加载渲染这个 js 文件。Weex 可以做到跨三端的原理在于：在开发过程中，代码模式、编译过程、模板组件、数据绑定、生命周期等上层语法是一致的。不同的是在 JS Framework 层的最后，web 平台和 Native 平台，对 Virtual DOM 执行的解析方法是有区别的。

 实际上，在 Native 中对 bundle 文件的加载大致经历以下阶段：

*   weex 接收到 js 文件以后，JS Framework 根据文件为 Vue 模式，会调用 weex-vue-framework 中提供的 createInstance 方法创建实例。(也可能是 Rax 模式)
*   createInstance 中会执行 Js Entry 代码里 new Vue() 创建一个组件，通过其 render 函数创建出 Virtual DOM 节点。
*   由 JS V8 引擎上解析 Virtual DOM ，得到 Json 数据发送至 Dom 线，这里输出 Json 也是方便跨端的数据传输。
*   Dom 线程解析 Json 数据，得到对应的 WxDomObject，然后创建对应的 WxComponent 提交 Render 。
*   Render 在原生端最终处理处理渲染任务，并回调里 JS 方法。

 得益于上层的统一性，只是通过 weex-vue-framework 判断是由 Vue.js 生成真实的 Dom ，还是通过 Native Api 渲染组件，weex 一定程度上上，用 JS 实现了 vue 一统天下的效果。

  weex 在原生渲染 Render 时，在接收到渲染指令后，会逐步将数据渲染成原生组件。Render 通过解析渲染数据的描述，然后分发给不同的模块。

  比如 控件渲染属于 dom 模块中，页面跳转属于 navigator 模块等。模块的渲染过程并非一个执行完，再执行另一个的流程，而是类似流式的过程。如上一个 <text> 的组件还没渲染好，下一个 <div> 的渲染又发了过来。这样当一个组件的嵌套组件很多时，或者可以看到这个大组件内的 UI，一个一个渲染出来的过程。

 weex 比起 react native，主要是在 JS V8 的引擎上，多了 JS Framework 承当了重要的职责，使得上层具备统一性，可以支持跨三个平台。总的来说它主要负责是：管理 Weex 的生命周期；解析 JS Bundle，转为 Virtual DOM，再通过所在平台不同的 API 方法，构建页面；进行双向的数据交互和响应。

2.3、打包

 weex 作为 react-native 之后出现的跨平台实现方案，自然可以站在前人的肩膀上优化问题，比如：Bundle 文件过大问题。

 Bundle 文件的大小，很大程度上影响了框架的性能，而 weex 选择将 JS Framework 集成到 WeexSDK 中，一定程度减少了 JS Bundle 的体积，使得 bundle 里面只保留业务代码。

 打包时，weex 是通过 webpack 打包出 bundle 文件的。bundle 文件的打包和 entry.js 文件的配置数量有关，默认情况下之后一个 entry 文件，自然也就只有一个 bundle 文件。

 在 weex 项目的 webpack.common.conf.js 中可以看到，其实打包也是区分了 webConfig 和 weexConfig 的不同打包方式。如下图，其中 weexEntry 就是 weex 打包配置的地方，可以看到本来已经有 index 和 entry.js 存在了。如果有需要，可配置上你需要的打包页面，具体这里就不详细展开了。有兴趣可看：[Weex 原理之带你去蹲坑](https://www.jianshu.com/p/ae1d7a2b0a8a) 。

3、Flutter

Google 出品，Dart 语言，Flutter Engine 引擎，响应式设计模式，原生渲染

 Flutter 是谷歌 2018 年发布的跨平台移动 UI 框架。相较于本人已经在项目中使用过 react native 和 Weex，Flutter 目前仅仅是简单运行过 Demo，毕竟还是 beta 阶段，这里更多的聊一下它的实现机制和效果。

 与 react native 和 weex 的通过 Javascript 开发不同，Flutter 的编程语言是 Drat，（谷歌亲儿子，据说是因为 Drat 项目组就在 Flutter 隔壁而被选上 (◐‿◑)）所以执行时并不需要 Javascript 引擎，但实际效果最终也通过原生渲染。

 如上图，Flutter 主要分为 Framework 和 Engine，我们基于 Framework 开发 App，运行在 Engine 上。Engine 是 Flutter 的独立虚拟机，由它适配和提供跨平台支持，目前猜测 Flutter 应用程序在 Android 上，是直接运行 Engine 上 所以在是不需要 Dalvik 虚拟机。（这是比 kotlin 更彻底，抛弃 JVM 的纠缠？）

 如下图，得益于 Engine 层，Flutter 甚至不使用移动平台的原生控件， 而是使用自己 Engine 来绘制 Widget （Flutter 的显示单元），而 Dart 代码都是通过 AOT 编译为平台的原生代码，所以 Flutter 可以 直接与平台通信，不需要 JS 引擎的桥接。同时 Flutter 唯一要求系统提供的是 canvas，以实现 UI 的绘制。咦？这么想来，支持 web 端也没问题吧！

 在 Flutter 中，大多数东西都是 widget，而 widget 是不可变的，仅支持一帧，并且在每一帧上不会直接更新，要更新而必须使用 Widget 的状态。无状态和有状态 widget 的核心特性是相同的，每一帧它们都会重新构建，有一个 State 对象，它可以跨帧存储状态数据并恢复它。

 Flutter 上 Android 自带了 Skia，Skia 是一个 2D 的绘图引擎库，跨平台，所以可以被嵌入到 Flutter 的 iOS SDK 中，也使得 Flutter Android SDK 要比 iOS SDK 小很多。

三、对比

 这算是互相伤害的环节了吧。(///▽///)

<table><tbody><tr><td><p>类型</p></td><td><p>React Native</p></td><td><p>Weex</p></td><td><p>Flutter</p></td></tr><tr><td><p>平台实现</p></td><td><p>JavaScript</p></td><td><p>JavaScript</p></td><td><p>无桥接，原生编码</p></td></tr><tr><td><p>引擎</p></td><td><p>JS V8</p></td><td><p>JSCore</p></td><td><p>Flutter engine</p></td></tr><tr><td><p>核心语言</p></td><td><p>React</p></td><td><p>Vue</p></td><td><p>Dart</p></td></tr><tr><td><p>Apk 大小 (Release)</p></td><td><p>7.6M</p></td><td><p>10.6M</p></td><td><p>8.1M</p></td></tr><tr><td><p>bundle 文件大小</p></td><td><p>默认单一、较大</p></td><td><p>较小、多页面可多文件</p></td><td><p>不需要</p></td></tr><tr><td><p>上手难度</p></td><td><p>稍高？</p></td><td><p>容易</p></td><td><p>一般</p></td></tr><tr><td><p>框架程度</p></td><td><p>较重</p></td><td><p>较轻</p></td><td><p>重</p></td></tr><tr><td><p>特点 (不局限)</p></td><td><p>适合开发整体 App</p></td><td><p>适合单页面</p></td><td><p>适合开发整体 App</p></td></tr><tr><td><p>社区</p></td><td><p>丰富，Facebook 重点维护</p></td><td><p>有点残念，托管 apache</p></td><td><p>刚刚出道小鲜肉，拥护者众多</p></td></tr><tr><td><p>支持</p></td><td><p>Android、IOS</p></td><td><p>Android、IOS、Web</p></td><td><p>Android、IOS（并不止？）</p></td></tr></tbody></table>

1、大小

 上面 Apk 大小是通过 react-native init、weex create 和 flutter 创建出的工程后，直接不添加任何代码，打包出来的 release 签名 apk 大小。从下图可以看出，其中大比例都是在 so 库。

2、社群

 react native 作为 Facebook 主力开源项目之一，至今已有各类丰富的第三方库，甚至如 realm、lottie 等开源项目也有 react native 相关的版本，社群实际无需质疑。当然，因为并完全正统开发平台，第三库的健壮性和兼容性有时候总是良莠不齐。

 weex 其实有种生错在国内的感觉。其实 weex 的设计和理念都很优秀，性能也不错，但是对比 react native 的第三方支持，就显得有点后妈养的。2016 年开源至今，社区和各类文档都显得有点疲弱，作为跨平台开发人员，大多时候肯定不会希望，需要频繁的自己增加原生功能支持，因为这样的工作一多，反而会与跨平台开发的理念背道而驰，带来开发成本被维护难度增加。

 Flutter 目前还处理 beta 阶段，但是谷歌的号召力一直很可观，这一点无需质疑。

3、性能

 理论上 flutter 的性能应该是最好的，但是目前实际体验中，却并没有感受出来太大的差距，和 react native（0.5.0 之后）、weex 在性能上个人体验差异不是很大。当然，这里并没有实测渲染的毫秒时间和帧率数据。

4、其他区别

*   Weex 的多页面实现问题

 weex 在 native 端是不支持 <keep-alive> 的，这一点和 react-native 不同在与，如果在 native 需要实现页面跳转，使用 vue-router 将会惨不忍睹：返回后页面不做特别处理时，是会空白一片。参考官方 Demo [playground](https://github.com/apache/incubator-weex/tree/master/android/playground)，native 端 的采用 weex.requireModule('navigator') 跳转 Activity 是才正确实现。

 同时，weex 中 navigator 跳转的设计，也导致了多页面的页面间通讯的差异。weex 在多页面下的数据通讯，是通过 url 实现的，比如 file://assets/dist/SecondPage.js?params=0，而 vuex 和 vue-router 在跨页面是无法共用的；而 react native 在跨 Actvity 使用时，因为是同一个 bundle 文件，只要 manager 相同，那么 router 和 store 时可以照样使用的，数据通信方式也和当个 Actvity 没区别。

*   项目模板

  weex 和 react native 模板代码模式也不同。weex 的模板是从 cordova 模式修改过来的，根据 platform 需求，用命令添加固定模块，而在 .gitignore 对 platforms 文件夹是忽略跟踪。 react native 在项目创建时模版就存在了，特别是添加第三方插件原生端支持时，会直接修改模板代码，git 代码中也会添加跟踪修改。

四、未来趋势

  我们选择框架的时候，很多时候会关注框架的成熟度和生命力不是么 (◐‿◑)。

1、React Native

  [“Airbnb 宣布放弃使用 React Native，回归使用原生技术”](https://www.colabug.com/3238051.html) : Airbnb 作为 react native 平台上最大的支持者之一，其开源的 lottie 同样是支持原生和 react native。

 Airbnb 在宣布放弃的文中，也对 react native 的表示了很大量的肯定，而使得他们放弃的理由，其实主要还是集中于项目庞大之后的维护困难，第三方库的良莠不齐，兼容上需要耗费更多的精力导致放弃。

ps：（ Lottie 库 Airbnb 出的是一个能够帮助解析 AE 导出的包含动画信息的 json 文件。这很好的解决了一个矛盾，设计师可以更专注的设计出各种炫酷的动画效果，而开发只需要将其加入支持即可。）

  [Facebook 正在重构 React Native，将重写大量底层。](https://www.oschina.net/news/97129/state-of-react-native-2018)在经历了开源协议风波后，可以看出 Facebook 对于 react native 还是很看重的， 这些底层重构优化的地方，主要集中于：

 首先，改变线程模型。UI 更新不再需要在三个不同的线程上执行，而是可以在任意线程上同步调用 JavaScript 进行优先更新，同时将低优先级工作推出主线程，以便保持对 UI 的响应。

 其次，将异步渲染功能引入 React Native 中，允许执行多个渲染并简化异步数据处理。

 最后，简化桥接，让它更快、更轻量。原生和 JavaScript 之间的直接调用效率更高，并且可以更轻松地构建调试工具，如跨语言堆栈跟踪。

2、Weex

  [没有死！阿里公开 Weex 技术架构，还开源了一大波组件。](https://blog.csdn.net/alitech2017/article/details/80133769) 2018 年初的新闻可以看出，weex 的遭遇有点类似曾经的 Duddo（Dubbo 因为内部竞争被阿里一度放弃维护），这波诈尸后 weex 被托管到了 Apache，而 github 的 [weexteam](https://github.com/weexteam) 如今也还保持着更新，希望后续能有多好的发展，拭目以待吧。

3、Flutter

 Flutter 是 Google 跨平台移动 UI 框架，Dart 作为谷歌的亲儿子在 Flutter 中使用，并且谷歌新操作系统 Fuchsia 支持 Dart，使用 Flutter 作为操作 UI 框架。这些集合到一起难免让你怀疑 Android 是否要被谷歌抛弃的想法。

 或者如今先 Android 等平台上推广 Flutter 与 Dart，就是为了以后跟好的过渡到新系统上，毕竟开发者是操作系统的生命源泉之一。而 Java 与 JVM 或者可以被谷歌完全抛开。当然，目前看起来 Flutter 貌似还缺少一些语法糖，嵌套下来的代码有点不忍直视，或者到正式版之后，我们更能感受出它的美丽吧。
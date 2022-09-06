> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7047777138585370660)*   最底层是大部分 C++ 与小部分 Rust 组成的引擎层
*   在这之上使用 C 语言封装一个个原子性的 C 接口
*   大部分的 C 语言接口使用 Flutter Plugin + Method Channel 的方式进行交互，另外一部分使用 FFI 进行交互。
*   最顶层则是用 Dart 将原生操作全部封装起来，Flutter 业务层只会与这一层 Wrapper 交互。

FFI
===

我们最初全部的交互都由 FFI 来完成，Dart 代码则全部使用 [Package - ffigen](https://link.juejin.cn?target=https%3A%2F%2Fpub.dev%2Fpackages%2Fffigen "https://pub.dev/packages/ffigen") 自动生成。

这样比 Flutter Plugin 的好处是完全不需要写原生代码，而且调用也是同步的，操作的体验是很好的，但是这里出现了新的问题 - Dart 是一个 VM 语言，它有自己独立的线程工作，而相当多的原生 UI API（`NSWindow*`) 只能在 Main Thread 中工作，这样就造成无法直接通过 FFI 的方式来调度包含此类 API 的接口，另一个方面，FFI 是直接工作在 Dart 线程，则会造成线程阻塞，对于部分稍微耗时的操作就会严重影响 Dart VM 的执行。

因此，我们将大部分的交互迁移到了 Flutter Plugin

Flutter Plugin
==============

在前面我们提到了我们的 C 接口层，无论是 FFI 还是 Flutter Plugin 我们都是依赖的这一层来进行调度，在 FFI 中是直接把 C 接口生成 Dart 函数，而在 Flutter Plugin 中，我们需要手写 Method Channel 的原生代码来进行调度，为了继续享受到类似`ffigen`这样一键生成到处使用的优势，我们内部研发了一个基于`C Header`的代码生成器，自动生成不同平台的 Plugin 代码，假设我们有接口

```
EXPORT_API int engine_load_midi_file(const char* file);

复制代码

```

通过解析语法，得到了 AST

```
{
 "name" : "engine_load_midi_file",
 "return_type" : "int",
 "arguments" : [
     {
         "type" : "const char*"
     }
 ]   
}
复制代码

```

得到了 AST 并不能直接进行代码生成，因为不同平台的类型是不一样的，因此我们要主要类型转换，Flutter 提供了类型映射表来定义 Dart 与原生类型的关系， ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dd3a442249a54a5495075220c101fd21~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

同理，我们在进行代码生成的时候也需要把 C 语言类型与对应平台的类型进行映射，比如`int` 在 Dart 中是`Int`, 而在 SWift 中则可能是`Int32`，我们需要根据自己的实际情况进行类型映射，之后就可以很方便的利用这个 AST 自动生成不同平台的代码了，

比如，在 Swift 里面就是

```
let call_args = call.arguments as! [Any]
 switch method.call {
    case "engine_load_midi_file" :
        let ret = engine_load_midi_file(call_args[0] as! String);
        result(ret);
        break;
 }
复制代码

```

其他平台类似的进行自动代码生成。

Flutter Plugin 的方式最大的问题在于所有的操作都是基于`queue`的异步操作，这样对于一些低成本的接口会额外带来不必要的性能损失。

如何从原生代码调用 Dart
==============

这里，我们的实际应用主要是一些底层的事件通知到 Dart Callback，比如设备情况发生变更了，我们需要通知到 Flutter 层做相应的处理，这类需求我们都是通过 NativePort 的方式完成的，

```
setupNativeCallback() {
    //定义一个port
    final interactiveCppRequests = ReceivePort()..listen(midiKeyboardCallback);
    //转换成为native port，本身是一个int64类型
    final int nativePort = interactiveCppRequests.sendPort.nativePort;
    //传递native port给底层
    await np().device_add_listener(1, nativePort);
}

midiKeyboardCallback(dynamic message) {
      // DO YOU WANT TO DO
}
复制代码

```

在 CPP 中则使用`Dart_PostCObject_DL` 接口进行写数据到`NativePort`

总结
==

到目前为止，我们还是比较赞同基于操作 / 行为 / 事件的方式来设计 C 接口。在交互上，FFI 和 Flutter Plugin 两种方式各有利弊，甚至还有 RPC 等基于网络的交互方案也可以尝试。

**在 Flutter 原生交互有什么疑问都可以留言一起探讨**
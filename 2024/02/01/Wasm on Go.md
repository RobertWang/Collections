> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI2ODIzMDM5MQ==&mid=2650725947&idx=1&sn=71e35964a2e79f0bfaa4ca8e5ce55782&chksm=f2f8b5a3c58f3cb5ccb0d3e6cbcce4caafa994677444cec56fa76289b02eed07ffa26cfabbea&mpshare=1&scene=1&srcid=02017tKSQpeRFmxr7N6AjmZn&sharer_shareinfo=77be7b23dc9152ae1a575b45b3299de3&sharer_shareinfo_first=77be7b23dc9152ae1a575b45b3299de3#rd)

> 本篇内容，是对极客兔兔: Go WebAssembly (Wasm) 简明教程 [1] 的实践与记录，主体内容来自这

本篇内容，是对极客兔兔: Go WebAssembly (Wasm) 简明教程 [1] 的实践与记录，主体内容来自这篇博客，推荐阅读原文。

### 是否需要搭建 wasm 环境？

WebAssembly 上手 [2]

如果是 C/C++，需要借助 emcc，将 C 和 C++ 代码编译到 WebAssembly 和 JavaScript。

在 Mac 上，

`brew install emscripten`

然后就可以使用  **emcc** 命令了

通过`git clone https://github.com/emscripten-core/emsdk.git`的方式编译安装, 可能有一堆坑 (可能和 Python 有关, 这个项目是用 Python 写的,WebAssembly 开发环境搭建 - MAC[3], 直接绕道使用 brew)

emcc 是 Emscripten 的 C/C++ 到 WebAssembly 编译器。

Emscripten 是一个项目, 它可以将 C 和 C++ 代码编译到 WebAssembly 和 JavaScript, 从而能在浏览器和 Node.js 中运行本来需要本地编译的 C/C++ 代码。

emcc 的主要作用和功能如下:

*   将 C/C++ 源代码编译成 WebAssembly 二进制格式 (.wasm 文件)
    
*   生成 JavaScript 源代码用来加载和支持 WebAssembly 模块
    
*   为 C/C++ 代码连接必要的 JavaScript 运行时支持 (如文件 I/O、多线程等)
    
*   将 C/C++ 标准库封装成 JavaScript 接口方便调用
    
*   支持 C++ 标准特性 (如 RTTI、异常等) 的编译
    
*   优化编译配置以减小文件体积
    
*   嵌入编译进一步混淆代码以提升性能
    

emcc 实际上是一个非常强大的交叉编译器, 可以将大多数 C/C++ 代码通过几次编译转化成浏览器和 Node.js 可以理解和运行的 WebAssembly 与 JavaScript 组合。以实现在 web 环境中运行原本需要本地编译的代码。

**但如果用 Go 或者 Rust, 就不需要这东西, 这些新语言原生支持 wasm**

### Go 对 wasm 的支持

Go 在 2018 年 8 月 24 号发布的 1.11 版本 [4] 中, 增加了实验性的 js/wasm, 算是对 Wasm 进行了原生的支持 (当然这个版本更重大的更新是 go module 这种依赖管理方式)。可以使用 go build 命令将 Go 程序编译为 WebAssembly 字节码。

自那以后便可以说，Go 语言原生支持 WebAssembly 的编译，可以将 Go 语言编写的程序编译成 wasm 格式，并在浏览器或其他支持 wasm 的环境中运行。

此外，Go 语言还提供了一些标准库和工具，如 syscall/js 包和 wasm_exec.js 库，用于与 JavaScript 交互和加载 WebAssembly 模块。

> 而在此之前，如果想用 Go 开发前端，需用 GopherJS[5]，这是一个可将 Go 转换成能在浏览器中运行的 JavaScript 代码的编译器。
> 
>  而 Go1.11 之后可以直接将 Go 代码编译为 wasm 二进制文件，不再需要转为 JavaScript 代码。（实现 GopherJS 和在 Go 语言中内建支持 WebAssembly 的是同一批人，包括后面会提到的 dmitshur[6] 大佬）

> Go 语言实现的函数可以直接导出供 JavaScript 代码调用，同时，Go 语言内置了 syscall/js 包，可以在 Go 语言中直接调用 JavaScript 函数，包括对 DOM 树的操作

### 初入门径 -- 使用 Go, 在网页上弹出 Hello World

(1). 新建 main.go:

```
package main

import "syscall/js"

func main() {
 alert := js.Global().Get("alert")
 alert.Invoke("Hello World!")
}

```

IDE 一直飘红, 这是因为需要修改构建标记中的 OS 为 js,Arch 为 wasm

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Q1ZRKiavkSiaC4DEvqIKvfBKUZgu9FeTfIX056LAP3IVmHQ7roZNRqibL1vw7H3eRGkQrkBCxo39qDdVRGVFZZ7fw/640?wx_fmt=png&from=appmsg)

(2). 执行`GOOS=js GOARCH=wasm go build -o static/main.wasm`, 将 main.go 编译为 static/main.wasm (如果按上面设置了 GOOS 和 GOARCH，则可以直接`go build -o static/main.wasm`)

此时会新生成一个文件夹 static, 里面有一个 main.wasm 文件

(3). 执行 `cp "$(go env GOROOT)/misc/wasm/wasm_exec.js" static`, 将 `wasm_exec.js` (JavaScript 支持文件，加载 wasm 文件时需要) 拷贝到 static 文件夹

misc 是 Go 源码中的一个文件, 其目录结构如下:

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Q1ZRKiavkSiaC4DEvqIKvfBKUZgu9FeTfI1XKMFZJBghNbeUTVuq0JFNhEFNwOox2nwqShuib3IyI4hiaOn54iaAkFA/640?wx_fmt=png&from=appmsg)

在 Go 语言源码中的`misc`目录下，包含了一些与特定平台或用途相关的杂项文件。以下是其中的各个目录和文件的作用：

1.  **cgo/gmp**:
    

*   **fib.go**: 包含一个使用 GMP 库计算斐波那契数列的示例程序。
    
*   **gmp.go**: 提供对 GMP（GNU Multiple Precision Arithmetic Library）库的 Go 绑定。
    
*   **pi.go**: 包含一个使用 GMP 库计算圆周率的示例程序。
    

3.  **chrome/gophertool**:
    

*   **README.txt**: 有关 Chrome 扩展的说明文档。
    
*   **background.html**: Chrome 扩展的后台页面 HTML。
    
*   **background.js**: Chrome 扩展的后台页面 JavaScript。
    
*   **gopher.js**: Chrome 扩展的 Gopher 图标的 JavaScript 代码。
    
*   **gopher.png**: Chrome 扩展中使用的 Gopher 图标。
    
*   **manifest.json**: Chrome 扩展的清单文件。
    
*   **popup.html**: Chrome 扩展的弹出页面 HTML。
    
*   **popup.js**: Chrome 扩展的弹出页面 JavaScript。
    

5.  **go_android_exec**:
    

*   **README**: 有关在 Android 上执行 Go 程序的说明文档。
    
*   **exitcode_test.go**: 包含与退出代码相关的测试。
    
*   **main.go**: 包含一个在 Android 上执行的示例 Go 程序。
    

7.  **ios**:
    

*   **README**: 有关在 iOS 上执行 Go 程序的说明文档。
    
*   **clangwrap.sh**: 提供用于 iOS 的 Clang 包装脚本。
    
*   **detect.go**: 包含检测 iOS 环境的 Go 代码。
    
*   **go_ios_exec.go**: 包含在 iOS 上执行 Go 程序的 Go 代码。
    

9.  **linkcheck**:
    

*   **linkcheck.go**: 包含一个用于检查链接的工具。
    

11.  **wasm**:
    

*   **go_js_wasm_exec**: 提供 Go 与 JavaScript 之间通信的支持。
    
*   **go_wasip1_wasm_exec**: 提供 Go 与 JavaScript 之间通信的支持（估计与 IP 地址相关）。
    
*   **wasm_exec.html**: 用于在浏览器中运行 WebAssembly 程序的 HTML 文件。
    
*   **wasm_exec.js**: WebAssembly 的 JavaScript 执行器。
    
*   **wasm_exec_node.js**: 用于在 Node.js 中运行 WebAssembly 程序的 JavaScript 文件。
    

这些文件和目录主要包含了与 Go 语言在不同平台、环境下的一些特殊需求或功能相关的实用工具和示例。

(4). 在与 main.go 同级目录下, 新建 index.html，引用 static/main.wasm 和 static/wasm_exec.js

```
<html>
<script src="static/wasm_exec.js"></script>
<script>
    const go = new Go();
    WebAssembly.instantiateStreaming(fetch("static/main.wasm"), go.importObject)
        .then((result) => go.run(result.instance));
</script>

</html>

```

(5). 使用 goexec 或者 npx http-server  启动一个本地 Web 服务

其中 shurcooL/goexec[7] 是 Go 生态的, 是前面提到的社区中非常活跃的 Go 项目多次核心贡献者 dmitshur 大佬写的。 (GOTIME[8] 有采访他的谈话节目)

http-party/http-server[9] 则是一个简单零配置的命令行 http 服务器, nodejs 开发.

此处使用后者, 执行 `npx http-server`

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Q1ZRKiavkSiaC4DEvqIKvfBKUZgu9FeTfIw36F2KBMUqnuNicQIcOVq2afzVuDsUYAvIzhRIPzVCXW9mI6Tpdam2g/640?wx_fmt=png&from=appmsg)

在浏览器中打开`http://127.0.0.1:8080` (服务器默认使用 8080 端口，可以通过参数进行配置)

能看到如下弹框

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Q1ZRKiavkSiaC4DEvqIKvfBKUZgu9FeTfI61GL2rjSMWTgEeFoBGLhAicEJZhc1H60vrA5GWuhsAiaehcfouk6pFxg/640?wx_fmt=png&from=appmsg)

另外可以看到请求的日志

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Q1ZRKiavkSiaC4DEvqIKvfBKUZgu9FeTfIYCCGPyqNDpaQKkaeuRqZ8709god5xAP1Of3ftCJnnRkFTf2atfSNeQ/640?wx_fmt=png&from=appmsg)

### 再上层楼 -- 注册 (自定义) 函数(Register Functions)

上面是在 Go 中调用 js 的函数, 但 wasm 最大的价值之一, 是能在浏览器中执行一些对于 js 来说压力太大的计算密集型操作.

在此用 Go 实现计算斐波那契数列的函数, 并注册到 js 中, 可以让其他 js 代码调用

新建一个目录, 创建一个 main.go 文件:

```
package main

import "syscall/js"

// fib 函数计算斐波那契数列的值
func fib(i int) int {
 if i == 0 || i == 1 {
  return 1
 }
 return fib(i-1) + fib(i-2)
}

// fibFunc 是一个JS回调函数，用于在JS中调用fib函数
func fibFunc(this js.Value, args []js.Value) interface{} {
 return js.ValueOf(fib(args[0].Int()))
}

func main() {
 done := make(chan int, 0)

 // 在全局对象上设置一个名为 "fibFunc" 的JS函数，该函数调用fibFunc回调
 js.Global().Set("fibFunc", js.FuncOf(fibFunc))

 // 通过无限循环，使Wasm程序保持运行状态;fibFunc 如果在 JavaScript 中被调用，会开启一个新的子协程执行。
 <-done
}

```

以上这段程序演示如何在 WebAssembly 中使用 Go 语言编写函数，并通过 JavaScript 调用这些函数。在这个例子中，fibFunc 函数充当了 Go 和 JavaScript 之间的桥梁，允许 JavaScript 代码调用 Go 中定义的斐波那契数列计算函数。

关于`js.Value`和`js.ValueOf`, 在 Go 的 WebAssembly（Wasm）和 JavaScript 交互中，`js.Value` 和 `js.ValueOf` 是两个相关但不同的概念。

1.  **`js.Value`**:
    

*   `js.Value` 是 Go 语言中用于表示 JavaScript 值的类型。
    
*   它是一个接口，表示可以与 JavaScript 交互的值。
    
*   `js.Value` 接口提供了一系列方法，例如 `Get`、`Set`、`Call`，用于在 Go 中操作 JavaScript 对象和函数。
    

3.  **`js.ValueOf`**:
    

*   `js.ValueOf` 是一个函数，用于将 Go 中的基本类型或其他类型转换为 `js.Value`。
    
*   当需要将 Go 的值传递给 JavaScript 时，通常使用 `js.ValueOf` 进行转换。
    

下面是一个简单的例子，说明了它们的使用：

```
package main

import (
 "fmt"
 "syscall/js"
)

func main() {
 // 创建一个 js.Value 对象，表示 JavaScript 中的数字 42
 jsNumber := js.ValueOf(42)

 // 在 Go 中调用 JavaScript 的 alert 函数，并传递一个字符串
 js.Global().Get("alert").Invoke(js.ValueOf("Hello from Go!"))

 // 在 Go 中调用 JavaScript 函数，传递和获取参数
 sum := js.Global().Get("add").Call(jsNumber, js.ValueOf(8))
 fmt.Println("Sum:", sum.Int())

 // 在 Go 中定义一个 JavaScript 回调函数，并传递给 JavaScript
 js.Global().Set("goCallback", js.FuncOf(goCallback))
 js.Global().Call("callJsFunction", js.Global().Get("goCallback"))

 // 保持程序运行，以便在浏览器中查看结果
 select {}
}

// goCallback 是一个在 JavaScript 中调用的 Go 回调函数
func goCallback(this js.Value, p []js.Value) interface{} {
 fmt.Println("Callback called from JavaScript!")
 return nil
}

```

在这个例子中：

*   使用 `js.ValueOf` 将 Go 的值转换为 `js.Value`。
    
*   使用 `js.Global().Get("alert").Invoke` 调用 JavaScript 的 `alert` 函数。
    
*   使用 `js.Global().Get("add").Call` 调用 JavaScript 的自定义函数，并传递参数。
    
*   使用 `js.FuncOf` 创建一个 JavaScript 可调用的 Go 回调函数，然后通过 `js.Global().Set` 注册到全局对象。
    

新建 index.html:

```
<html>

<body>
<input id="num" type="number" />
<button id="btn" onclick="ans.innerHTML=fibFunc(num.value * 1)">Click</button>
<p id="ans">1</p>
</body>

<script src="static/wasm_exec.js"></script>
<script>
    const go = new Go();
    WebAssembly.instantiateStreaming(fetch("static/main.wasm"), go.importObject)
        .then((result) => go.run(result.instance));
</script>

</html>

```

相比于之前的页面, 新增了一段块, 增加一个输入框, 按钮, 文本框.   并给按钮添加一个点击事件, 将计算结果显示在文本框中

执行`GOOS=js GOARCH=wasm go build -o static/main.wasm`, 将 main.go 编译为 static/main.wasm

执行 `npx http-server`

输入任意数字, 能正确计算出结果

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Q1ZRKiavkSiaC4DEvqIKvfBKUZgu9FeTfIBxq0ibdlqFrjNeMQib34wqGgYau8crExduvBXpr9G1tUPzReicHia5cKEg/640?wx_fmt=png&from=appmsg)

如果输入的数字较大, 浏览器能直接把 CPU 跑满..

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Q1ZRKiavkSiaC4DEvqIKvfBKUZgu9FeTfI27llWV2xIWtITLBibGs7ozohE7mLeXwhA4XfQIrDEQXST8IXsYBcguA/640?wx_fmt=png&from=appmsg)

### 牛刀再试 --- 操作 DOM

上面例子中 index.html 中 DOM 元素的操作, 是靠嵌入在 HTML 中的 JavaScript 代码。

即

```
<input id="num" type="number" />
<button id="btn" onclick="ans.innerHTML=fibFunc(num.value * 1)">Click</button>
<p id="ans">1</p>


```

*   `<input>` 元素的 `id="num"` 用于输入数字的输入字段。
    
*   `<button>` 元素的 `id="btn"` 具有一个 `onclick` 属性，其中包含 JavaScript 代码。
    
*   `onclick` 属性中的 JavaScript 代码是 `ans.innerHTML=fibFunc(num.value * 1)`。它将具有 `id="ans"` 的元素的 `innerHTML` 设置为调用名为 `fibFunc` 的函数的结果，该函数使用 `num` 输入字段中输入的值。
    

这段 JavaScript 代码负责在按钮点击时更新具有 `id="ans"` 的段落（`<p>`）元素的内容。`fibFunc` 函数是一个斐波那契函数，接收来自 `num` 输入字段的输入值，计算斐波那契值，并在具有 `id="ans"` 的段落中显示它。

希望能够通过 Go 而不是 js 来操作 DOM 元素

新建一个项目, 名叫 dom,

新建 index.html, 去除操作 DOM 部分的 js 代码:

```
<html>

<body>
<input id="num" type="number" />
<button id="btn" onclick="ans.innerHTML=fibFunc(num.value * 1)">Click</button>
<p id="ans">1</p>
</body>

<script src="static/wasm_exec.js"></script>
<script>
    const go = new Go();
    WebAssembly.instantiateStreaming(fetch("static/main.wasm"), go.importObject)
        .then((result) => go.run(result.instance));
</script>

</html>

```

新建 main.go:

```
package main

import (
 "strconv"
 "syscall/js"
)

func fib(i int) int {
 if i == 0 || i == 1 {
  return 1
 }
 return fib(i-1) + fib(i-2)
}

var (
 document = js.Global().Get("document")
 numEle   = document.Call("getElementById", "num")
 ansEle   = document.Call("getElementById", "ans")
 btnEle   = js.Global().Get("btn")
)

func fibFunc(this js.Value, args []js.Value) interface{} {
 v := numEle.Get("value")
 if num, err := strconv.Atoi(v.String()); err == nil {
  ansEle.Set("innerHTML", js.ValueOf(fib(num)))
 }
 return nil
}

func main() {
 done := make(chan int, 0)
 btnEle.Call("addEventListener", "click", js.FuncOf(fibFunc))
 <-done
}

```

这是一个使用 Go 语言和 WebAssembly（Wasm）的简单示例程序，它通过网页上的按钮触发斐波那契数列的计算。

1.  `fib` 函数定义了一个递归的斐波那契数列计算方法。
    
2.  在 `main` 函数中，通过 `js.Global().Get("document")` 获取全局文档对象，然后使用 `Call` 方法获取 HTML 文档中的元素，包括输入框 (`num`)、段落 (`ans`) 和按钮 (`btn`)。
    
3.  `fibFunc` 函数是一个回调函数，它被注册到按钮的点击事件上。当按钮被点击时，这个函数会读取输入框的值，将其转换为整数，然后调用斐波那契函数计算结果，并将结果更新到段落中。
    
4.  在 `main` 函数中，通过 `js.FuncOf(fibFunc)` 将 Go 函数转换为 JavaScript 函数，然后通过 `Call` 方法将这个 JavaScript 函数注册到按钮的点击事件上。
    
5.  `done := make(chan int, 0)` 和 `<-done` 是为了保持程序运行，以便持续监听事件。
    

该程序利用 Go 和 JavaScript 的互操作性，通过 WebAssembly 在浏览器中执行 Go 代码，实现了一个简单的交互式网页。

`cp "$(go env GOROOT)/misc/wasm/wasm_exec.js" static`

`GOOS=js GOARCH=wasm go build -o static/main.wasm`

`npx http-server`

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Q1ZRKiavkSiaC4DEvqIKvfBKUZgu9FeTfIRFuxOZFywibZ1YD2TRFZiavgJw4J4jCraAywCXVhPTeyhCNDveCice66w/640?wx_fmt=png&from=appmsg)

### 紫禁之巅 --- 使用回调函数 (Callback Functions)

> 在 Js 中，`异步+回调`很常见，如请求一个 Restful API，注册一个回调函数，待数据获取到，再执行回调函数的逻辑. 这期间程序可以继续做其他事。Go 语言可通过协程实现异步。
> 
> 假设 fib 的计算非常耗时，那么可以启动注册一个回调函数，待 fib 计算完成后，再把计算结果显示出来。
> 
> 先修改 main.go，使得 fibFunc 支持传入回调函数。

新建一个目录称为 callback

main.go:

修改 fibFunc, 使其支持传入回调函数

```
package main

import (
 "syscall/js"
 "time"
)

func fib(i int) int {
 if i == 0 || i == 1 {
  return 1
 }
 return fib(i-1) + fib(i-2)
}

func fibFunc(this js.Value, args []js.Value) interface{} {
 callback := args[len(args)-1]
 go func() {
  time.Sleep(3 * time.Second)
  v := fib(args[0].Int())
  callback.Invoke(v)
 }()

 js.Global().Get("ans").Set("innerHTML", "Waiting 3s...")
 return nil
}

func main() {
 done := make(chan int, 0)
 js.Global().Set("fibFunc", js.FuncOf(fibFunc))
 <-done
}

```

这是一个使用 Go 语言和 WebAssembly（Wasm）的示例程序，演示了在计算斐波那契数列时如何通过 Go 异步处理，并在等待期间更新网页。

1.  `fib` 函数定义了一个递归的斐波那契数列计算方法。
    
2.  `fibFunc` 函数是一个回调函数，它被注册到 JavaScript 中的 `fibFunc` 函数。在计算斐波那契数列时，它通过 JavaScript 的回调方式异步执行，模拟了一个耗时的操作。在计算完成后，通过 `callback.Invoke(v)` 将结果传递给 JavaScript 回调函数。
    
3.  在 `main` 函数中，通过 `js.Global().Set("fibFunc", js.FuncOf(fibFunc))` 将 Go 中的 `fibFunc` 函数注册到全局，以便 JavaScript 可以调用它。
    
4.  `time.Sleep(3 * time.Second)` 模拟一个耗时的操作，延迟 3 秒。
    
5.  在等待期间，通过 `js.Global().Get("ans").Set("innerHTML", "Waiting 3s...")` 将网页上显示的信息更新为 "Waiting 3s..."。
    

通过这个示例，展示了如何在 WebAssembly 中使用 Go 处理异步操作，并在等待时更新网页内容。

> *   假设调用 fibFunc 时，回调函数作为最后一个参数，那么通过 args[len(args)-1] 便可获取到该函数。这与其他类型参数的传递并无区别。
>     
> *   使用 go func() 启动子协程，调用 fib 计算结果，计算结束后，调用回调函数 callback，并将计算结果传递给回调函数，使用 time.Sleep() 模拟 3s 的耗时操作。
>     
> *   计算结果出来前，先在界面上显示 Waiting 3s...
>     

新建 index.html，为按钮添加点击事件，调用 fibFunc

```
<html>

<body>
<input id="num" type="number" />
<button id="btn" onclick="fibFunc(num.value * 1, (v)=> ans.innerHTML=v)">Click</button>
<p id="ans"></p>
</body>

<script src="static/wasm_exec.js"></script>
<script>
    const go = new Go();
    WebAssembly.instantiateStreaming(fetch("static/main.wasm"), go.importObject)
        .then((result) => go.run(result.instance));
</script>

</html>

```

> *   为 btn 注册了点击事件，第一个参数是待计算的数字，从 num 输入框获取。
>     
> *   第二个参数是一个回调函数，将参数 v 显示在 ans 文本框中。
>     

`cp "$(go env GOROOT)/misc/wasm/wasm_exec.js" static`

`GOOS=js GOARCH=wasm go build -o static/main.wasm`

`npx http-server`

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Q1ZRKiavkSiaC4DEvqIKvfBKUZgu9FeTfIKZoksIQuHaHPmChwjeGXIYM9R88lnWwXibD8yliaHcN6CtSBZOj05paw/640?wx_fmt=png&from=appmsg)![](https://mmbiz.qpic.cn/sz_mmbiz_png/Q1ZRKiavkSiaC4DEvqIKvfBKUZgu9FeTfIFTXMaHjvj407MSXh5McSOOIbjbXCwcmdLZJlydDC9pWGRKHD74BqsQ/640?wx_fmt=png&from=appmsg)

会先显示 `Waiting 3s...`，3s 过后显示计算结果

更多推荐阅读

go 编译 wasm 与调用 [10]

Go 中的 WASM 很棒：全网最全示例教程 [11]

【Go】【WebAssembly】【wasm】基于 go 打包的网页 wasm[12]

可能是世界上最简单的用 Go 来写 WebAssembly 的教程 [13]

如何在 Go 中使用 Wasm：浅聊 WebAssembly[14]

参考资料

[1]

极客兔兔: Go WebAssembly (Wasm) 简明教程: _https://geektutu.com/post/quick-go-wasm.html_

[2]

WebAssembly 上手: _https://www.cnblogs.com/Wayou/p/webassembly_quick_start.html_

[3]

WebAssembly 开发环境搭建 - MAC: _https://blog.csdn.net/daill894/article/details/103815099_

[4]

1.11 版本: _https://tip.golang.org/doc/go1.11_

[5]

GopherJS: _https://github.com/gopherjs/gopherjs_

[6]

dmitshur: _https://github.com/dmitshur_

[7]

shurcooL/goexec: _https://github.com/shurcooL/goexec_

[8]

GOTIME: _https://changelog.com/gotime_

[9]

http-party/http-server: _https://github.com/http-party/http-server_

[10]

go 编译 wasm 与调用: _https://www.jianshu.com/p/69645c8bf57c_

[11]

Go 中的 WASM 很棒：全网最全示例教程: _https://www.qinglite.cn/doc/66216476ed6fe796d_

[12]

【Go】【WebAssembly】【wasm】基于 go 打包的网页 wasm: _https://blog.csdn.net/sky529063865/article/details/126005525_

[13]

可能是世界上最简单的用 Go 来写 WebAssembly 的教程: _https://www.jiqizhixin.com/articles/2020-06-30-10_

[14]

如何在 Go 中使用 Wasm：浅聊 WebAssembly: _https://juejin.cn/post/7195217413091262523_
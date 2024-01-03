> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [wasmer.io](https://wasmer.io/posts/introducing-the-wasmer-js-sdk)

> Today we are incredibly excited to present `@wasmer/sdk`, a new library that allows running WASI(X) a......

Dive into a world where running any WASI and WASIX package in your browser is a breeze. Whether it's Python, Bash, FFmpeg, or [any package published](https://docs.wasmer.io/registry/get-started) in the registry, Wasmer Javascript SDK makes it all seamlessly possible.  
潜入一个在浏览器中运行任何 WASI 和 WASIX 包的世界。无论是 Python、Bash、FFmpeg 还是注册表中发布的任何包，Wasmer Javascript SDK 都能无缝地实现这一切。

We think this is incredibly exciting, because the `@wasmer/sdk` allows running any UNIX programs using **threads**, **signals**, **subprocesses** and other features in the browser (via [WASIX](https://wasix.org/)).  
我们认为这非常令人兴奋，因为它 `@wasmer/sdk` 允许使用浏览器中的线程、信号、子进程和其他功能（通过 WASIX）运行任何 UNIX 程序。

With the new SDK, you just need to point to the desired Wasmer package and the SDK will download all the files (.wasm and the filesystem required) for you.  
使用新的 SDK，您只需指向所需的 Wasmer 包，SDK 就会为您下载所有文件（.wasm 和所需的文件系统）。

Let's see some cool examples running the Wasmer JS SDK, shall we?  
让我们看看一些运行 Wasmer JS SDK 的很酷的例子，好吗？

*   FFMpeg FFMpeg 的
*   CPython (with threads and signals!)  
    CPython（有线程和信号！
*   Bash ([wasmer.sh](https://wasmer.sh) demo!) Bash （ wasmer.sh 演示！

_Note: `@wasmer/sdk` supersedes the `@wasmer/wasi` package  
注意： `@wasmer/sdk` 取代 `@wasmer/wasi` 包装_

> Any WASIX package published to Wasmer will be instantly usable with the Wasmer Javascript SDK: in this article we will showcase how to use FFMpeg and Python, but you can use your [own package](https://docs.wasmer.io/registry/get-started) just as easily.  
> 任何发布到 Wasmer 的 WASIX 包都可以立即与 Wasmer Javascript SDK 一起使用：在本文中，我们将展示如何使用 FFMpeg 和 Python，但您也可以同样轻松地使用自己的包。

A love story: FFMpeg  
一个爱情故事：FFMpeg
------------------------------------

FFMpeg is one of the most useful and popular packages in the world. It’s used by Youtube, Facebook, Netflix and many other industry leaders. It allows them to transform and edit videos very easily.  
FFMpeg 是世界上最有用和最受欢迎的软件包之一。它被 Youtube、Facebook、Netflix 和许多其他行业领导者使用。它使他们能够非常轻松地转换和编辑视频。

FFMpeg has been ported a few times to Javascript already, thanks to Emscripten and WebAssembly. However, these ports required a lot of manual changes from the maintainers to plug in all the features and make the package easily usable for developers.  
FFMpeg 已经被移植到 Javascript 上几次，这要归功于 Emscripten 和 WebAssembly。但是，这些移植需要维护人员进行大量手动更改，以插入所有功能并使开发人员可以轻松使用该软件包。

Thanks to the new Wasmer JS SDK, the ffmpeg WASIX version published in Wasmer can now be easily in the browser.  
多亏了新的 Wasmer JS SDK，在 Wasmer 中发布的 ffmpeg WASIX 版本现在可以很容易地在浏览器中出现。

Let's see an example where we retrieve a video ([`wordpress.mp4`](https://cdn.wasmer.io/media/wordpress.mp4)) and extract its audio:  
让我们看一个示例，其中我们检索视频 （ `wordpress.mp4` ） 并提取其音频：

```
import { init, Wasmer } from "@wasmer/sdk";

await init();

let ffmpeg = await Wasmer.fromRegistry("wasmer/ffmpeg");
let resp = await fetch("https://cdn.wasmer.io/media/wordpress.mp4");
let video = await resp.arrayBuffer();

// We take stdin ("-") as input and write the output to stdout ("-") as a
// WAV audio stream.
const instance = await ffmpeg.entrypoint.run({
  args: ["-i", "-", "-f", "wav", "-"],
  stdin: new Uint8Array(video),
});

const { stdoutBytes } = await instance.wait();
console.log(`The audio stream: ${output.stdoutBytes}`);

```

Et voilá, you have the audio now created: [wordpress-ffmpeg-out.wav](https://cdn.wasmer.io/media/wordpress-ffmpeg-out.wav)  
瞧，您现在已经创建了音频：wordpress-ffmpeg-out.wav

> You can get the audio file by running ffmpeg locally with wasmer: `cat wordpress.mp4 | wasmer run wasmer/ffmpeg -- -i - -f wav - > audio.wav`  
> 您可以通过使用 wasmer 在本地运行 ffmpeg 来获取音频文件： `cat wordpress.mp4 | wasmer run wasmer/ffmpeg -- -i - -f wav - > audio.wav`

Running CPython in the browser  
在浏览器中运行 CPython
------------------------------------------------

There was another project that allowed using CPython in the browser: Pyodide. However, Pyodide is based on Emscripten and as such only runs on the browser, but not only that... Pyodide doesn't support threading.  
还有另一个项目允许在浏览器中使用 CPython：Pyodide。然而，Pyodide 基于 Emscripten，因此只能在浏览器上运行，但不仅如此......Pyodide 不支持线程。

Wasmer [`python` package](https://wasmer.io/python/python) can run on both the browser and the server seamlessly, and requires no manual intervention to run in the browser (no extra packages needed other than the SDK!).  
Wasmer `python` 包可以在浏览器和服务器上无缝运行，并且不需要手动干预即可在浏览器中运行（除了 SDK 之外，不需要额外的包！

Here's how you can handle CPython with the Filesystem API of the Wasmer JS SDK:  
以下是使用 Wasmer JS SDK 的文件系统 API 处理 CPython 的方法：

```
import { init, Directory, Wasmer } from "@wasmer/sdk";
 
// Initialize the Wasmer SDK
await init();
 
// Download the Python package
const python = await Wasmer.fromRegistry("python/python");
 
// The script to be executed
const script = `
import sys
 
with open("/out/version.txt", "w") as f:
  f.write(sys.version)
`;
// A shared directory where the output will be written
const out = new Directory();
 
// Running the Python script
const instance = await python.entrypoint.run({
  args: ["/src/main.py"],
  mount: {
    "/out": out,
    "/src": {
      "main.py": script,
    }
  },
});
const output = await instance.wait();
 
if (!output.ok) {
  throw new Error(`Python failed ${output.code}: ${output.stderr}`);
}
 
// Read the version string back
const pythonVersion = await out.readTextFile("/version.txt");
console.log(pythonVersion) // 3.11.6 (main, ...)

```

> Check out the [_FileSystem guide_](https://docs.wasmer.io/javascript-sdk/how-to/use-filesystem) to learn more.  
> 查看 FileSystem 指南以了解更多信息。

Wasmer.sh
---------

As an example, our own [wasmer.sh](https://wasmer.sh) website has now an incredibly simple implementation that just fetches the bash package, and pipes it over [xterm.js](https://xtermjs.org/) using the Wasmer Javascript SDK.  
举个例子，我们自己的 wasmer.sh 网站现在有一个非常简单的实现，它只是获取 bash 包，并使用 Wasmer Javascript SDK 通过 xterm 进行管道传输. js。

![](https://wasmer.io/_next/image?url=https%3A%2F%2Fcdn.wasmer.io%2Fimages%2FScreenshot_2023-12-21_at_7.36.18PM.original.png&w=1920&q=75)

> Something that before required 1,000 lines of code in Rust + JS glue code manually written, now can be written in [less than 50 lines of clear JavaScript code](https://github.com/wasmerio/wasmer-js/blob/main/examples/wasmer.sh/index.ts)  
> 以前需要用 1,000 行代码在 Rust + JS 胶水代码中手动编写的东西，现在可以用不到 50 行清晰的 JavaScript 代码编写

You can now have your own terminal emulator running in your website very easily using XTerm.js and the Wasmer JS SDK:  
您现在可以使用 XTerm.js 和 Wasmer JS SDK 非常轻松地在您的网站中运行自己的终端模拟器：

```
// This is a simplified version of:
// https://github.com/wasmerio/wasmer-js/blob/main/examples/wasmer.sh/index.ts
import { Terminal } from "xterm";

async function main() {
  const { Wasmer, init } = await import("@wasmer/sdk");
  await init();

  const term = new Terminal();
  term.open(document.getElementById("terminal")!);

  const pkg = await Wasmer.fromRegistry("sharrattj/bash");
  const instance = await pkg.entrypoint!.run();
	
  connectStreams(instance, term);
}

function connectStreams(instance, term) {
  const stdin = instance.stdin?.getWriter();
  const encoder = new TextEncoder();
  term.onData(data => stdin?.write(encoder.encode(data)));
  instance.stdout.pipeTo(new WritableStream({ write: chunk => term.write(chunk) }));
  instance.stderr.pipeTo(new WritableStream({ write: chunk => term.write(chunk) }));
}

main();

```

Technical feat 技术壮举
-------------------

Making the Wasmer JS SDK to work was not an easy task. We had to to achieve incredibly hard challenges, so you don't need to worry about solving them yourself:  
让 Wasmer JS SDK 工作并不是一件容易的事。我们必须完成难以置信的艰巨挑战，因此您无需担心自己解决它们：

*   Using asyncify to be able to `fork` processes  
    使用 asyncify 能够 `fork` 处理
*   Using a universal format to containerize the Wasm and filesystem together  
    使用通用格式将 Wasm 和文件系统一起容器化
*   A Filesystem abstraction to ease the container use  
    简化容器使用的文件系统抽象

Lets get into this a bit more!  
让我们更深入地了解一下！

#### Supporting multithreading  
支持多线程

Javascript runs in a single thread. However, it is possible to use JS Workers to enable multithreading in the browser. The Workers API differs from the typical threading API (that you would use in Python, Go or Rust), in the sense that there's no `join` (to wait for a thread to finish). The JS Worker communicates with the main process via a message channel/bus.  
Javascript 在单个线程中运行。但是，可以使用 JS Workers 在浏览器中启用多线程。Workers API 与典型的线程 API（您将在 Python、Go 或 Rust 中使用）不同，从某种意义上说，没有 `join` （等待线程完成）。JS Worker 通过消息通道 / 总线与主进程进行通信。

At the end, we were able to allow UNIX-like threading to run via JS Workers by reusing the shared memory from the main WebAssembly process and notifying the memory upon changes (using Wasm notify/wait syntax).  
最后，我们能够允许类 UNIX 线程通过 JS Workers 运行，方法是重用主 WebAssembly 进程中的共享内存，并在更改时通知内存（使用 Wasm notify/wait 语法）。

#### Running `fork` in the browser  
在浏览器中运行 `fork`

Running `fork` in the browser is almost an impossible task, as it requires dumping the stack and being able to resume it from a different process.  
在浏览器中运行 `fork` 几乎是一项不可能完成的任务，因为它需要转储堆栈并能够从不同的进程恢复它。

We used [`asyncify`](https://kripken.github.io/blog/wasm/2019/07/16/asyncify.html) to freeze the program with the current stack, and then resume it later in a newly created Worker.  
我们过去 `asyncify` 常常用当前堆栈冻结程序，然后稍后在新创建的 Worker 中恢复它。

#### An universal container format  
通用容器格式

The Wasmer API now ships a special container format (that we will cover in future blogposts) that allows serving both the Wasm assets along with the filesystem ones. That way, you can download one package with all it's filesystem dependencies at once (such as Python!).  
Wasmer API 现在提供了一种特殊的容器格式（我们将在以后的博客文章中介绍），它允许同时提供 Wasm 资产和文件系统资产。这样，您可以一次下载一个包含所有文件系统依赖项的包（例如 Python！

This container format is the format also used in both the native runtime and Wasmer Edge. More to come on this later!  
此容器格式也是本机运行时和 Wasmer Edge 中使用的格式。稍后会有更多的内容！

#### A Filesystem API 文件系统 API

We realized that using the Filesystem was critical to allow any developer achieving their needs.  
我们意识到，使用文件系统对于任何开发人员实现他们的需求都至关重要。

Read more about the Wasmer JS SDK Filesystem API here: [https://docs.wasmer.io/javascript-sdk/how-to/use-filesystem](https://docs.wasmer.io/javascript-sdk/how-to/use-filesystem)  
在此处阅读有关 Wasmer JS SDK 文件系统 API 的更多信息：https://docs.wasmer.io/javascript-sdk/how-to/use-filesystem

In summary 综上所述
===============

We can’t wait to see how you use the Wasmer JS SDK to use all packages published in the Wasmer ecosystem.  
我们迫不及待地想看看你如何使用 Wasmer JS SDK 来使用 Wasmer 生态系统中发布的所有包。

We have many examples on how to use Wasmer-JS:  
我们有很多关于如何使用 Wasmer-JS 的例子：

*   Github Pages: [cdn-coi-serviceworker](https://github.com/wasmerio/wasmer-js/tree/main/examples/cdn-coi-serviceworker)  
    Github 页面： cdn-coi-serviceworker
*   FFMpeg with React: [ffmpeg-react](https://github.com/wasmerio/wasmer-js/tree/main/examples/ffmpeg-react)  
    使用 React 的 FFMpeg：ffmpeg-react
*   Markdown editor: [markdown-editor](https://github.com/wasmerio/wasmer-js/tree/main/examples/markdown-editor)  
    Markdown 编辑器：markdown-editor
*   Wasmer.sh: [wasmer.sh](https://github.com/wasmerio/wasmer-js/tree/main/examples/wasmer.sh) Wasmer.sh： wasmer.sh
*   Any other? Open an issue or a PR in the Wasmer-JS repo!  
    还有其他的吗？在 Wasmer-JS 存储库中打开问题或 PR！

> Using Wasmer SDK in the browser requires you setting the Cross-origin isolation headers ([COOP and COEP](https://web.dev/coop-coep/)) in the pages that use the SDK. If you can’t do this (because you are publishing to Github Pages, for example) you can use [`coi-serviceworker`](https://docs.wasmer.io/javascript-sdk/how-to/coop-coep-headers) to automatically patch the headers client-side ([see example](https://github.com/wasmerio/wasmer-js/tree/main/examples/cdn-coi-serviceworker)).  
> 在浏览器中使用 Wasmer SDK 需要在使用 SDK 的页面中设置跨域隔离标头（COOP 和 COEP）。如果你不能这样做（例如，因为你要发布到 Github Pages），你可以使用 `coi-serviceworker` 客户端自动修补标头（参见示例）。
> 
> For more details, check out [_SharedArrayBuffer and Cross-Origin Isolation_](https://docs.wasmer.io/javascript-sdk/explainers/troubleshooting#sharedarraybuffer-and-cross-origin-isolation) in the JavaScript SDK docs.  
> 有关更多详细信息，请查看 JavaScript SDK 文档中的 SharedArrayBuffer 和跨域隔离。

Bash, Python, FFMpeg, QuickJS, OpenSSL, and many more packages are waiting for you!  
Bash、Python、FFMpeg、QuickJS、OpenSSL 和更多软件包等着你！

Get started with the Wasmer JS SDK docs : [https://docs.wasmer.io/javascript-sdk](https://docs.wasmer.io/javascript-sdk)  
开始使用 Wasmer JS SDK 文档：https://docs.wasmer.io/javascript-sdk

Let us know what you build next on [Wasmer's discord](https://discord.gg/rWkMNStrEW)  
让我们知道您接下来在 Wasmer 的 discord 上构建了什么
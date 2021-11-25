> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6998876488451751973)*   [@ffmpeg/core](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fffmpegwasm%2Fffmpeg.wasm-core "https://github.com/ffmpegwasm/ffmpeg.wasm-core")：编译 ffmpeg 生成 ffmpeg-core.wasm + js 胶水代码。
*   [@ffmpeg/ffmpeg](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fffmpegwasm%2Fffmpeg.wasm "https://github.com/ffmpegwasm/ffmpeg.wasm")：实现了调用上一步生成的胶水代码的部分，提供了 load, run 等 API。同时，如果开发者对 @ffmpeg/core 不满意，也可以构建自定义的 ffmpeg-core.wasm。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc6ad3f48a6d4d978fd1b22fb410d46e~tplv-k3u1fbpfcp-watermark.awebp) 那么，能直接用吗？有这些问题还待解决：

*   浏览器兼容性：我们知道，浏览器的 js 线程是单线程的，并且与渲染线程互斥。为了不阻塞页面的渲染和 js 主线程，`@ffmpeg/core`在编译 ffmpeg 时，配置了 [pthreads](https://link.juejin.cn?target=https%3A%2F%2Femscripten.org%2Fdocs%2Fporting%2Fpthreads.html "https://emscripten.org/docs/porting/pthreads.html")，导致产生的 js 胶水代码中使用了`sharedarraybuffer`。[`sharedarraybuffer`](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FSharedArrayBuffer "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer")能满足主线程和 worker 之间的数据共享，也可以满足多个 worker 之间的数据共享，用于此场景中是很理想的。

但是，因为[安全问题](https://link.juejin.cn?target=https%3A%2F%2Fmeltdownattack.com%2F "https://meltdownattack.com/")，所有主流浏览器均默认禁用，需要另外配置一些返回头部字段，而且[支持度不太理想](https://link.juejin.cn?target=https%3A%2F%2Fcaniuse.com%2F%3Fsearch%3Dsharedarraybuffer "https://caniuse.com/?search=sharedarraybuffer")，不能达到上线标准。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d3161d34706a4bb1b576250727783710~tplv-k3u1fbpfcp-watermark.awebp)

*   wasm 冗余：`@ffmpeg/core`编译出来的`ffmpeg-core.wasm`几乎包括了 ffmpeg 的所有功能，[文件](https://link.juejin.cn?target=https%3A%2F%2Funpkg.com%2F%40ffmpeg%2Fcore%400.10.0%2Fdist%2Fffmpeg-core.wasm "https://unpkg.com/@ffmpeg/core@0.10.0/dist/ffmpeg-core.wasm")大小是 24MB（gzip 后是 8.5MB），其中很多是截帧不需要的。

### 1.3.2 其他平台的实现

根据业务的要求（支持的格式比较少），通过自定义编译 ffmpeg，最后生成的 wasm 文件大小可以减少到 4.7MB（gzip 后可以更小）。

但是，自己维护一份 c 语言的入口文件，用 FFmpeg 提供的内部库，实现截帧功能，然后再编译 ffmpeg。

这种方式比较考验对 FFmpeg 的理解，而且与 ffmpeg 的特定版本绑定，而随着 FFmpeg 的版本升级，ffmpeg 的 API、目录可能会有变更。再加上，随着业务发展，我们可能会用到 ffmpeg 更多的功能时，还需要修改这份 c 代码，可维护性比较低。

1.4 小结
------

因此，最终采用的方案是，使用 Webassembly 截帧，具体实现：

1.  自定义编译 ffmpeg，优化 wasm 文件大小。
2.  使用 ffmpeg（v4.3.1）提供的 fftools/ffmpeg.c 入口文件，无需自己写 c 代码。
3.  编译出不带`sharedarraybuffer`的 ffmpeg-core.wasm+js，最后使用 web worker 运行截帧相关的业务代码，以防阻塞主线程。
4.  调用编译生成的 ffmpeg 的 js 胶水代码，实现截帧功能，这部分可以使用`@ffmpeg/ffmpeg`。

2 自定义编译 ffmpeg
==============

2.1 运行 docker 使用官方的 emscripten 环境
---------------------------------

emscripten 是一个 WebAssembly 编译器工具链。

下载 [Docker Desktop](https://link.juejin.cn?target=https%3A%2F%2Fwww.docker.com%2Fget-started "https://www.docker.com/get-started")，通过[运行 docker](https://link.juejin.cn?target=https%3A%2F%2Femscripten.org%2Fdocs%2Fgetting_started%2Fdownloads.html%23using-the-docker-image "https://emscripten.org/docs/getting_started/downloads.html#using-the-docker-image") 的方式使用已经搭建好的 Emscripten 环境，避免本地开发环境的坑。 mac 的 Docker Desktop 老是连接不上，实测 windows 的 ubuntu 跑 docker 命令更稳定。

在 ffmpeg 源码目录中，编写运行 docker 的脚本如下：

```
#!/bin/bash
set -euo pipefail

EM_VERSION=2.0.8

docker pull emscripten/emsdk:$EM_VERSION
docker run \
  --rm \
  -v $PWD:/src \ # 绑定挂载
  -v $PWD/wasm/cache:/emsdk_portable/.data/cache/wasm \
  emscripten/emsdk:$EM_VERSION \
  sh -c 'bash ./build.sh'

复制代码
```

### 2.1.1 了解下 emscripten 原理

具体来说，就是 C/C++ 等语言，经过 clang 前端变成 LLVM 中间代码（IR），再从 LLVM IR 到 wasm。然后浏览器把 WebAssembly 下载下来，然后先经过 WebAssembly 模块，再到目标机器的汇编代码，再到机器码（x86/ARM 等）。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/69b4c1ec04504e608cd8734218615db8~tplv-k3u1fbpfcp-watermark.awebp)

那，LLVM 和 Clang 是什么呢？

*   LLVM，就是不同的前端后端使用统一的中间代码 LLVM Intermediate Representation (LLVM IR)。
*   Clang 是 LLVM 的一个子项目，基于 LLVM 架构的 C/C++/Objective-C 编译器前端。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb58740f3d6048acb24a93cb97710a56~tplv-k3u1fbpfcp-watermark.awebp)

> *   Frontend 前端：词法分析、语法分析、语义分析、生成中间代码
> *   Optimizer 优化器：中间代码优化（循环优化、删除无用代码等等）
> *   Backend 后端：生成目标代码。如目标代码是绝对指令代码（机器码），则这种目标代码可立即执行。如果目标代码是汇编指令代码，则需汇编器汇编之后（生成机器码）才能运行。

接下来是编写编译的脚本`build.sh`。

2.2 配置 ffmpeg 编译参数，去掉冗余
-----------------------

ffmpeg 是优秀的 C/C++ 音视频处理库，可以实现视频截图。

首先，我们要知道实现截图会涉及的库和组件。

涉及到的库：

*   libavcodec：音视频的编码和解码。
*   libavformat：音视频的封装和解封装。
*   libavutil：包含一些公共的工具函数的使用库，包括算数运算，字符操作等。
*   libswscale：图像伸缩和像素格式转化。

涉及的组件：

*   demuxer：对视频解封装
*   decoder：对视频解码
*   encoder：得到解码后的帧之后，输出图片编码
*   muxer：图片封装

使用 [`emconfigure`](https://link.juejin.cn?target=https%3A%2F%2Femscripten.org%2Fdocs%2Fcompiling%2FBuilding-Projects.html%23integrating-with-a-build-system "https://emscripten.org/docs/compiling/Building-Projects.html#integrating-with-a-build-system")设置合适的环境参数，和配置 FFmpeg 编译参数。 关于配置的说明文档：

*   运行`emconfigure ./configure --help` 查看所有可以用的配置。
*   关于 FFMPEG 配置的详细说明可以点击[这里](https://link.juejin.cn?target=https%3A%2F%2Fcloud.tencent.com%2Fdeveloper%2Farticle%2F1393972 "https://cloud.tencent.com/developer/article/1393972")查看。

```
# configure FFMpeg with Emscripten
emconfigure ./configure 
  --target-os=none        # use none to prevent any os specific configurations
  --arch=x86_32           # use x86_32 to achieve minimal architectural optimization
  --enable-cross-compile  # enable cross compile
  --disable-x86asm        # disable x86 asm
  --disable-inline-asm    # disable inline asm
  --disable-stripping     # disable stripping
  --disable-programs      # disable programs build (incl. ffplay, ffprobe & ffmpeg)
  --disable-doc           # disable doc
  --nm="llvm-nm"
  --ar=emar
  --ranlib=emranlib
  --cc=emcc
  --cxx=em++
  --objcc=emcc
  --dep-cc=emcc
  # 去掉不需要的库
  --disable-avdevice
  --disable-swresample
  --disable-postproc
  --disable-network
  --disable-pthreads
  --disable-w32threads
  --disable-os2threads
  # 配置需要的解封装，编解码器等
  --disable-everything # 减少wasm体积的关键，除了以下的组件外的个别组件都disable
  --enable-filters
  --enable-muxer=image2
  --enable-demuxer=mov # mov,mp4,m4a,3gp,3g2,mj2
  --enable-demuxer=flv
  --enable-demuxer=h264
  --enable-demuxer=asf
  --enable-encoder=mjpeg
  --enable-decoder=hevc
  --enable-decoder=h264
  --enable-decoder=mpeg4
  --enable-protocol=file

# build dependencies
emmake make -j4
复制代码
```

2.4 生成 js+wasm
--------------

使用`emcc`将上一步 make 生成的链接代码编译为 JavaScript + WebAssembly。这里使用`fftools/ffmpeg.c`作为入口文件，不需要自己维护一份 c 语言入口文件。

可通过`emcc --help`查看 [emcc 参数选项](https://link.juejin.cn?target=https%3A%2F%2Femscripten.org%2Fdocs%2Ftools_reference%2Femcc.html%23command-line-syntax "https://emscripten.org/docs/tools_reference/emcc.html#command-line-syntax")，以及通过`clang --help`查看 [clang 参数选项](https://link.juejin.cn?target=https%3A%2F%2Flinux.die.net%2Fman%2F1%2Fclang "https://linux.die.net/man/1/clang")。

```
emcc
  -I. -I./fftools # Add directory to include search path
  -Llibavcodec -Llibavdevice -Llibavfilter -Llibavformat -Llibavresample -Llibavutil -Llibpostproc -Llibswscale -Llibswresample # Add directory to library search path
  -Qunused-arguments # Don't emit warning for unused driver arguments.
  -o wasm/dist/ffmpeg-core.js # output
  fftools/ffmpeg_opt.c fftools/ffmpeg_filter.c fftools/ffmpeg_hw.c fftools/cmdutils.c fftools/ffmpeg.c # input
  -lavdevice -lavfilter -lavformat -lavcodec -lswresample -lswscale -lavutil -lm # library
  -s USE_SDL=2 # use SDL2
  -s MODULARIZE=1 # use modularized version to be more flexible
  -s EXPORT_ # assign export name for browser
  -s EXPORTED_FUNCTIONS="[_main]" # export main and proxy_main funcs
  -s EXTRA_EXPORTED_RUNTIME_METHODS="[FS, cwrap, ccall, setValue, writeAsciiToMemory]" # export extra runtime methods
  -s INITIAL_MEMORY=33554432 # 33554432 bytes = 32MB
  -s ALLOW_MEMORY_GROWTH=1 # allows the total amount of memory used to change depending on the demands of the application
  -s ASSERTIONS=1 # for debug
  --post-js wasm/post-js.js # emits a file after the emitted code. use to expose exit function
  -O3 # optimize code and reduce code size
复制代码
```

最后构建的 ffmpeg-core.wasm 大小为 5MB，gzip 后会更小。

源码：[build.sh](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fseminelee%2Fffmpeg.wasm-core%2Fblob%2Fn4.3.1-wasm%2Fbuild1.sh "https://github.com/seminelee/ffmpeg.wasm-core/blob/n4.3.1-wasm/build1.sh")

到这里，编译 ffmpeg 完成了！接下来回到我们熟悉的前端领域。

3 实现截帧功能
========

3.1 调用 js 胶水代码
--------------

关于调用 js 胶水代码的这部分，在开源库`@ffmpeg/ffmpeg`已经实现了，我们可以简单地使用它的 [API](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fffmpegwasm%2Fffmpeg.wasm%2Fblob%2Fmaster%2Fdocs%2Fapi.md "https://github.com/ffmpegwasm/ffmpeg.wasm/blob/master/docs/api.md")。

```
const { createFFmpeg } = require('@ffmpeg/ffmpeg');
const ffmpeg = createFFmpeg({ log: true });

(async () => {
  await ffmpeg.load();
  // ... 省略获取时长duration部分
  const frameNum = 8;
  const per = duration / (frameNum - 1);
  for (let i = 0; i < frameNum; i++) {
    await ffmpeg.run('-ss', `${Math.floor(per * i)}`, '-i', 'example.mp4', '-s', '960x540', '-f', 'image2', '-frames', '1', `frame-${i + 1}.jpeg`);
  }
})();
复制代码
```

期间，还发现了`-ss`放在`-i`前，可以截取指定时间的帧，而不用等待逐帧读取，可以提升截图速度。可查看相关 [API 文档](https://link.juejin.cn?target=https%3A%2F%2Fffmpeg.org%2Fffmpeg.html "https://ffmpeg.org/ffmpeg.html")。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6728e06200da4387a1e736981530fa1a~tplv-k3u1fbpfcp-watermark.awebp)

P.S. `@ffmpeg/ffmpeg`目前还不支持加载去掉`pthreads`的 ffmpeg-core.wasm+js，已给该库提了 [pr](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fffmpegwasm%2Fffmpeg.wasm%2Fpull%2F235 "https://github.com/ffmpegwasm/ffmpeg.wasm/pull/235")。

### 3.1.1 JavaScript 与 C 交换数据

那`load`和`run`方法具体是怎么实现的呢？ 这里要先了解的是，[JavaScript 与 C 交换数据](https://link.juejin.cn?target=https%3A%2F%2Fwww.cntofu.com%2Fbook%2F150%2Fzh%2Fch2-c-js%2Fch2-04-data-exchange.md "https://www.cntofu.com/book/150/zh/ch2-c-js/ch2-04-data-exchange.md")时，只能使用 Number 作为参数。因为从语言角度来说，JavaScript 与 C/C++ 有完全不同的数据体系，Number 是二者唯一的交集，因此本质上二者相互调用时，都是在交换 Number。

因此如果参数是字符串、数组等非 Number 类型，则需要拆分为以下步骤：

*   使用`Module._malloc()`在`Module`堆中分配内存，获取地址 ptr；
*   将字符串 / 数组等数据拷入内存的 ptr 处；
*   将 ptr 作为参数，调用 C/C++ 函数进行处理；
*   使用`Module._free()`释放 ptr。

以下是 [`@ffmpeg/ffmpeg`](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fffmpegwasm%2Fffmpeg.wasm "https://github.com/ffmpegwasm/ffmpeg.wasm")的部分源码：

```
const createFFmpegCore = require('path/to/ffmpeg-core.js');
let ffmpeg;

// 加载
const load = async () => {
  Core = await createFFmpegCore({
    print: (message) => {},
  });
  ffmpeg = Core.cwrap('_main', 'number', ['number', 'number']); // cwrap调用导出的主函数
};

const parseArgs  = (Core, args) => {
  const argsPtr = Core._malloc(args.length * Uint32Array.BYTES_PER_ELEMENT);
  args.forEach((s, idx) => {
    const buf = Core._malloc(s.length + 1);
    Core.writeAsciiToMemory(s, buf);
    Core.setValue(argsPtr + (Uint32Array.BYTES_PER_ELEMENT * idx), buf, 'i32');
  });
  return [args.length, argsPtr]; // [数组的长度, 数组的ptr]
};

// 执行ffmpeg命令
const run = (..._args) => {
  return new Promise((resolve) => {
    ffmpeg(...parseArgs(Core, _args)); // 传入命令参数
  });
};

module.exports = {
  load,
  run,
};
复制代码
```

4 web worker
============

因为在构建中没有配置`-s USE_PTHREADS=1` ，上面调用`ffmpeg`的方法会阻塞 js 主线程和页面的渲染。比如，在生成推荐封面的同时，无法更新上传视频的进度状态，用户点击页面上的其他按钮也无法响应等。因此，需要增加一个 web worker 来运行。

[`Web Worker`](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FWeb_Workers_API%2FUsing_web_workers "https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API/Using_web_workers")是在与浏览器页面线程分开的线程上运行的脚本，可以用于从页面线程分流几乎所有繁重的处理。主线程和 worker 可以通过`postMessage()`方法和`onmessage`事件进行通信。

但使用`postMessage()`方法和`onmessage`事件进行编写通信过程会使代码显得繁琐。且 [`postMessage()`](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FWorker%2FpostMessage "https://developer.mozilla.org/zh-CN/docs/Web/API/Worker/postMessage")只能传递由[结构化克隆](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FWeb%2FGuide%2FDOM%2FThe_structured_clone_algorithm "https://developer.mozilla.org/en-US/docs/Web/Guide/DOM/The_structured_clone_algorithm")算法处理的任何值或 JavaScript 对象。这里推荐使用 [`Comlink`](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FGoogleChromeLabs%2Fcomlink "https://github.com/GoogleChromeLabs/comlink") (1.1kB)，使代码变得更友好，让通信变得无感知。

比如截帧的通信：

main.js

```
import * as Comlink from 'Comlink';
async function onFileUpload(file) {
  const ffmpegWorker = Comlink.wrap(new Worker('./worker.js'));
  const frameU8Arrs = await ffmpegWorker.getFrames(file);
}
复制代码
```

worker.js

```
import * as Comlink from 'Comlink';
async function getFrames(file) {
  // ...
  // 先获取时长duration等
  const frameNum = 8;
  const per = duration / (frameNum - 1);
  let frameU8Arrs = [];
  for (let i = 0; i < frameNum; i++) {
    await ffmpeg.run('-ss', `${Math.floor(per * i)}`, '-i', 'example.mp4', '-s', '960x540', '-f', 'image2', '-frames', '1', `frame-${i + 1}.jpeg`);
  }
  // 从MEMFS获取图片二进制数据Uint8Array
  for (let i = 0; i < frameNum; i++) {
    const u8arr = await ffmpeg.FS('readFile', `frame-${i + 1}.jpeg`); 
    frameU8Arrs.push(u8arr);
    ffmpeg.FS('unlink', fileName);
  }
  return frameU8Arrs;
}

Comlink.expose({
  getFrames,
});
复制代码
```

`Comlink`是基于 Es6 `Proxy`和`postMessage()`的 RPC 实现。例子中，`ffmpegWorker`是位于 worker.js 中的对象，main.js 里拿到的只是`ffmpegWorker`的本体的句柄，实际上`ffmpegWorker.getFrames`等方法的执行也是在 worker.js 上运行的。

唯一一个坑点是这个库的[产出是 es6 代码](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FGoogleChromeLabs%2Fcomlink%2Fissues%2F477 "https://github.com/GoogleChromeLabs/comlink/issues/477")，还需要通过构建配置转为 es5 代码。

4.1 webpack 配置
--------------

另外，如果你使用 webpack，可能还会遇到无法加载正确的`worker.js`路径的问题。可以这样配置 [`worker-plugin`](https://link.juejin.cn?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Fworker-plugin "https://www.npmjs.com/package/worker-plugin")。

```
const WorkerPlugin = require('worker-plugin');
const isPub = true; // 是否生产环境
{
  // ...
  plugins: [
    new WorkerPlugin({
      globalObject: 'this',
      filename: isPub ? '[name].[chunkhash:9].worker.js' : '[name].worker.js',
    }),
  ],
}
复制代码
```

5 上线效果
======

上线后，对于支持该方案的浏览器，用户无需等待视频上传完成，即可选择、编辑视频封面。

而且，比起在后台读取视频后截帧，前端截帧的耗时也大大缩小了。这在视频大小越大的视频越明显。

6 后续优化点
=======

6.1 提高浏览器支持率
------------

在部分浏览器报错，之后持续优化，提高浏览器支持率。（如 Safari 某版本 fetch wasm 报错）。

6.2 减少 wasm 文件大小
----------------

wasm 体积还有减少的空间。（如在编译配置中配置了`enable-filters`使用了所有的 filters）。

6.3 读取视频文件优化
------------

因为默认使用 MEMFS，会将视频文件整个存入内存中，然后处理。大的视频文件如 800MB + 的视频文件，在 Firefox 90 版本，运行任务时内存占用会接近 3G，还会出现浏览器崩溃的情况。

```
const getVideoInfo = async (file) => {
  // ...先实现fileToUint8Array方法
  const bufferArr = await fileToUint8Array(file);
  ffmpeg.FS('writeFile', 'example.mp4', bufferArr); // 先保存到MEMFS
  await ffmpeg.run('-i', 'example.mp4', '-loglevel', 'info');
}
复制代码
```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/47dd02ab37a34ec1840bb793f97d48ef~tplv-k3u1fbpfcp-watermark.awebp)

目前想到的方案是，使用 [WORKERFS](https://link.juejin.cn?target=https%3A%2F%2Femscripten.org%2Fdocs%2Fapi_reference%2FFilesystem-API.html%23filesystem-api-workerfs "https://emscripten.org/docs/api_reference/Filesystem-API.html#filesystem-api-workerfs")。WORKERFS 运行在 Web Worker 中，提供对 woker 内部的 `File` 和 `Blob` 对象的只读访问，而无需将整个数据复制到内存中，符合我们的需求。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cdf4697a83984a6695a63e78d898173b~tplv-k3u1fbpfcp-watermark.awebp)

参考
==

*   [Build FFmpeg WebAssembly version (= ffmpeg.wasm): Part.2 Compile with Emscripten](https://link.juejin.cn?target=https%3A%2F%2Fjeromewu.github.io%2Fbuild-ffmpeg-webassembly-version-part-2-compile-with-emscripten%2F "https://jeromewu.github.io/build-ffmpeg-webassembly-version-part-2-compile-with-emscripten/")
*   [前端视频帧提取 ffmpeg + Webassembly](https://juejin.cn/post/6854573219454844935 "https://juejin.cn/post/6854573219454844935")
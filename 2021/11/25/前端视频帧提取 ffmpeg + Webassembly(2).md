> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6968057860596957192/)*   内存占用大，上一篇文章的方案首先把视频文件读取为 `ArrayBuffer` 并拷贝在内存中，然后通过 `wasm` 读取内存进行调用，除了提取视频帧所需要占用的内存外，还需要至少占用一个视频文件大小的内存。这个问题在视频文件较大时表现的更加明显
    
*   内存占用超过限制导致解析失败，在 Chrome 的 V8 引擎中，目前对 `wasm` 的内存限制为 [64 位浏览器 4GB，32 位浏览器 2GB](https://link.juejin.cn?target=https%3A%2F%2Fv8.dev%2Fblog%2F4gb-wasm-memory "https://v8.dev/blog/4gb-wasm-memory")。而因为上一篇文章中的技术方案，需要把整个视频文件保存在内存中，这在我们提取一些高清视频或者长视频的视频帧画面时，就会出现因为内存不足导致的解析失败的问题。
    

为了解决原有的内存占用大的问题，我们就需要更换技术方案了。幸运的是，`webassembly` 提供了相应的文件系统模块和操作，Emscripten 也提供了相关的 [File System API](https://link.juejin.cn?target=https%3A%2F%2Femscripten.org%2Fdocs%2Fapi_reference%2FFilesystem-API.html "https://emscripten.org/docs/api_reference/Filesystem-API.html") 的封装。

一、技术方案设计
--------

查阅文档可以发现，Emscripten 主要提供了四种 File System API 的方案，下面逐一做一下对比和分析

1.  MEMFS, **所有文件都保存于内存中**，与我们降低内存占用的目标不符
2.  NODEFS, **只能运行在 nodejs 环境中**，无法运行在浏览器
3.  IDBFS, 基于 `indexDB` 实现文件持久化
4.  WORKERFS, 运行在 `Web Worker` 中，提供对 woker 内部的 `File` 和 `Bolb` 对象的只读访问，而无需将整个数据复制到内存中，符合我们的需求

由此可以基于 `WORKERFS` 来设计我们新的技术方案：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e3230dd8c3eb4ba39ee93eb141ff62d1~tplv-k3u1fbpfcp-watermark.awebp)

使用 `Web Worker` 来加载和运行 `wasm`, 在 `input` 选择了文件之后，将文件传输到对应的 `Web Worker` 之后，`wasm` 通过 `FS API`访问视频文件并提取对应的视频帧，之后将图像数据传输回主线程并在 `canvas` 上绘制出来

二、相关代码修改
--------

在设计了技术方案之后，首先需要对原有从内存中读取视频流然后调用 `ffmpeg` 的 c 代码进行修改，改为使用 `File System API` 的方式进行读取和解析，原有的先 `setFile` 设置内存再 `capture` 截取视频帧的接口调用方式也要进行修改，改为直接从文件中进行读取，不再需要 `setFile`

### 1. wasm 模块

主要进行以下修改：

1.  去掉 `setFile` 方法，`capture` 方法增加文件路径参数，并使用 `avformat_open_input` 来实现对应文件路径的读取, 关键代码如下:

```
ImageData *capture(int ms, char *path) {

    ImageData *imageData = NULL;

    AVFormatContext *pFormatCtx = avformat_alloc_context();  

    if (avformat_open_input(&pFormatCtx, path, NULL, NULL) < 0) {
        fprintf(stderr, "avformat_open_input failed\n");
        return NULL;
    }

    ...
}
复制代码
```

2.  编译命令完善，增加 `File System API` 和 `cwrap` 等配置项。因为 `wasm` 默认的调用 `c` 函数的传参中只能传输 `int` 类型，所以需要通过 `cwrap` 的方式来帮助传输字符串类型, 从而实现将路径传给 `wasm`, 关键代码如下:

```
-lworkerfs.js \
-s EXPORTED_RUNTIME_METHODS='["ccall", "cwrap"]' \
复制代码
```

调整完主要的这两项后，还需要对内存分配和回收等细节进行调整，具体可以直接参考项目代码

### 2. js 模块

由于使用的 `Web Worker` 加 `File System API` 的方式，以及提取流程的修改，需要对 js 模块也进行相应的调整，主要包括：

1.  使用 `Emscripten` 官方提供的 `FS` 接口进行文件的挂载和回收，关键代码如下：

```
// 挂载文件到 /working 目录
const MOUNT_DIR = '/working';

if (!this.isMKDIR) {
    FS.mkdir(MOUNT_DIR);
    this.isMKDIR = true;
}

FS.mount(WORKERFS, { files: [file] }, MOUNT_DIR);

...

// 使用完成后回收文件
FS.unmount(MOUNT_DIR);

复制代码
```

2.  使用 `cwrap` 的方式调用 `wasm` 方法，实现对路径字符串的传递，关键代码如下：

```
if (!this.cCapture) {
    this.cCapture = Module.cwrap('capture', 'number', ['number', 'string']);
}

// 与 capture 代码中的传参 (int ms, char *path) 相对应
let imgDataPtr = this.cCapture(timeStamp, `${MOUNT_DIR}/${file.name}`);

复制代码
```

具体 `cwrap` 的使用和文档可以参考 `Emscripten` 的[官方文档](https://link.juejin.cn?target=https%3A%2F%2Femscripten.org%2Fdocs%2Fapi_reference%2Fpreamble.js.html%3Fhighlight%3Dcwrap%23cwrap "https://emscripten.org/docs/api_reference/preamble.js.html?highlight=cwrap#cwrap")

其他的还包括 `Web Worker` 加载和初始化相关流程的修改，具体可以直接参考项目代码

三、其他优化
------

除了上述的技术方案修改外，还有对 `Web Worker` 和 `wasm` 加载的优化。

首先是可以通过 `URL.createObjectURL` 的方式，直接实现用本地文本来初始化 `Web Worker`, 相关的代码字符串可以使用全局变量的方式，在编译是进行替换, 示例如下

```
function createWorker() {
    const workerBolb = new Blob([WORKER_STRING], {
        type: 'application/javascript'
    });

    const workerURL = URL.createObjectURL(workerBolb);

    const captureWorker = new Worker(workerURL);

    return captureWorker;
}
复制代码
```

其次，在`Emscripten` 编译出来的代码中包含胶水代码和 `wasm` 文件两部分，胶水代码可以通过合并编译的方式来直接打包到 `Web Worker` 的代码中，而 `wasm` 文件则没有办法直接打包。

如果使用上一篇中的自定义加载的方式，固然可以解决响应头不一致导致的重复加载，但是依然会有加载耗时和调用时机的处理问题。`wasm` 文件要先加载完成后再异步的进行初始化，在此之前进行调用会有无法响应的问题。

由此，可以通过将 `wasm` 文件读取为 `base64` 格式打包进代码中，在初始化时使用 `base64` 转为 `ArrayBuffer` 来实现 `wasm` 的初始化，示例如下

```
var binary_string = self.atob(WASM_STRING);
var len = binary_string.length;
var bytes = new Uint8Array(len);
for (var i = 0; i < len; i++) {
    bytes[i] = binary_string.charCodeAt(i);
}
复制代码
```

在修改了技术方案和进行了一系列优化之后，相比于原有的方案，内存占用和提取性能都有了明显的提升，并且调用方式相比之下更加的简洁

四、总结
----

`ffmpeg` 作为一个功能强大的音视频库，提取视频帧只是其功能的一小部分，应该还有更多 `ffmpeg` + `Webassembly` 的应用场景可以去探索。

在互联网传输的带宽不断增大，延迟不断降低的发展趋势下，音视频领域在可见的将来依然会保持不错的发展前景，而依托于 `ffmpeg` + `Webassembly`, 在网页端也能够进行更多的尝试和应用

五、项目地址
------

[github.com/jordiwang/w…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fjordiwang%2Fweb-capture "https://github.com/jordiwang/web-capture")

六、前文背景
------

[前端视频帧提取 ffmpeg + Webassembly](https://juejin.cn/post/6854573219454844935 "https://juejin.cn/post/6854573219454844935")
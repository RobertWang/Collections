> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/Xianhuii/p/16843884.html)

> StreamSaver.js + zip-stream.js 流式下载 & 压缩文件。  
> 部分浏览器（火狐）可能不兼容。

1 应用场景[#](#1-应用场景)
==================

在实际项目中，通常存在用户手动选择下载多个文件的情况。  
常规的做法（服务器打包下载）是，后端从文件服务器（比如华为云 OBS）读取文件，将这些文件进行打包，然后将压缩包字节流返回给前端，由前端下载到用户本地文件系统。  
服务器打包下载存在以下问题：

1.  文件传输链路：文件服务器→应用服务器，应用服务器→浏览器，消耗传输时间和服务器的流量。
2.  服务器打包，消耗性能。
3.  大文件下载和打包，通常会连接超时。

另一种做法（浏览器打包下载）是，后端生成多个文件服务临时访问地址，前端直接从文件服务器下载文件，将这些文件进行打包，然后下载到用户本地文件系统。  
浏览器打包下载有以下优点：

1.  文件传输链路：文件服务器 → 浏览器，明显缩短传输时间，减少服务器的流量和性能消耗。
2.  将大请求分解成一个个小请求，避免了请求超时的问题。
3.  文件服务器通常不会设置请求超时时间。

但是，浏览器打包下载也存在以下缺点：

1.  前端通过 AJAX 请求文件服务器，需要文件服务器支持跨域。
2.  前端下载文件消耗浏览器内存，可能会造成 OOM（Out of Memory）。

对于第一个缺点，文件服务器通常可以在管理页面进行跨域配置，因此一般都可以解决。  
对于第二个缺点，以往思路是先将文件下载到浏览器内存，然后再对每个文件进行压缩操作。这种方法对下载文件大小有限制。  
`StreamSaver.js`可以对数据进行流式处理，结合`zip-stream.js`可以做到实时将文件流进行压缩，直接保存到用户本地文件系统。因此，除了下载速度 > 压缩速度那一部分缓存，可以大大减小浏览器内存的占用。

2 使用[#](#2-使用)
==============

StreamSaver.js + zip-stream.js 流式下载 & 压缩文件的使用方式很简单。

2.1 引入 js 文件[#](#21-引入js文件)
---------------------------

由于某些浏览器不支持流式处理，可以按需要引入：

```
<script src="https://cdn.jsdelivr.net/npm/web-streams-polyfill@3.2.1/dist/ponyfill.min.js"></script>
<script src="https://cdn.jsdelivr.net/gh/eligrey/Blob.js/Blob.js"></script>


```

然后引入 [StreamSaver.js](https://cdn.jsdelivr.net/npm/streamsaver@2.0.3/StreamSaver.min.js) 和 [zip-stream.js](https://jimmywarting.github.io/StreamSaver.js/examples/zip-stream.js)（可以根据以下链接直接下载到本地）：

```
<script src="https://jimmywarting.github.io/StreamSaver.js/examples/zip-stream.js"></script>
<script src="https://cdn.jsdelivr.net/npm/streamsaver@2.0.3/StreamSaver.min.js"></script>


```

2.2 定义打包下载函数[#](#22-定义打包下载函数)
-----------------------------

定义同步打包下载函数`zipFiles()`：

```
/**  
 * 同步下载打包【推荐】 
 * @param zipName 压缩包文件名  
 * @param files 文件列表，格式：[{"name":"文件名", "url":"文件下载地址"},……]  
 */
 function zipFiles(zipName, files) {  
    console.log("同步下载打包开始时间：" + new Date());  
    // 创建压缩文件输出流  
    const zipFileOutputStream = streamSaver.createWriteStream(zipName);  
    // 创建下载文件流  
    const fileIterator = files.values();  
    const readableZipStream = new ZIP({  
        async pull(ctrl) {  
            const fileInfo = fileIterator.next();  
            if (fileInfo.done) {//迭代终止  
                ctrl.close();  
            } else {  
                const {name, url} = fileInfo.value;  
                return fetch(url).then(res => {  
                    ctrl.enqueue({  
                        name,  
                        stream: () => res.body  
                    });  
                })  
            }        }    });  
    if (window.WritableStream && readableZipStream.pipeTo) {  
        // 开始下载  
        readableZipStream  
            .pipeTo(zipFileOutputStream)  
            .then(() => console.log("同步下载打包结束时间：" + new Date()));  
    }  
}


```

需要注意的是，浏览器只会使用一个 Service Worker 线程进行压缩，整体打包下载速度取决于压缩速度。因此对于多个文件，异步下载的加速效果没有那么明显，反而可能会使浏览器内存占用过多，造成浏览器内存溢出。  
以下是异步下载打包的方法，但不推荐使用：

```
/**  
 * 异步下载打包【不推荐，超大文件时可能会造成浏览器内存溢出】   
 * @param zipName  压缩包文件名
 * @param files 文件列表，格式：[{"name":"name", "url":"url"},……] 
 */  
function asyncZipFiles(zipName, files) {  
    console.log("异步下载打包开始时间：" + new Date());  
    // 创建压缩文件  
    const zipFileOutputStream = streamSaver.createWriteStream(zipName);  
    // 创建下载文件流  
    const readableZipStream = new ZIP({  
        async pull(ctrl) {  
            // promise任务  
            const promise = el => {  
                let name = el.name  
                return new Promise(resolve => {  
                    fetch(el.url).then(resp => {  
                        if (resp.status === 200) {  
                            return () => resp.body;  
                        }  
                        return null;  
                    }).then(stream => {  
                        resolve({name: name, stream: stream});  
                    })  
                })            }            // promise任务队列  
            let arr = [];  
            files.forEach(el => {  
                arr.push(promise(el));  
            })  
            // 异步下载  
            await Promise.all(arr).then(res => {  
                let nameMapList = []  
                res.forEach(item => {  
                    const name = item.name;  
                    const stream = item.stream;  
                    // 加入打包队列  
                    if (stream) {  
                        let file = {name, stream};  
                        ctrl.enqueue(file);  
                    }  
                })            })            ctrl.close();  
        }  
    });  
    if (window.WritableStream && readableZipStream.pipeTo) {  
        // 开始下载  
        readableZipStream  
            .pipeTo(zipFileOutputStream)  
            .then(() => console.log("异步下载打包结束时间：" + new Date()))  
    }}


```

2.3 调用函数进行下载[#](#23-调用函数进行下载)
-----------------------------

当用户选择好要下载的文件，点击下载按钮时，可以构造打包下载的参数进行下载：

```
let zipName = '压缩包.zip';  
let files = [  
    {   "name": '2022102801.mp4',  
        "url": 'http://localhost:8080/file3'  
    }, 
    {  
        "name": '文件夹1/2022102802.mp4',  
        "url": 'http://localhost:8080/file4'  
    },  
    {  
        "name": '文件夹1/2022102803.mp4',  
        "url": 'http://localhost:8080/file5'  
    },  
    {  
        "name": '文件夹3/文件夹3/文件夹3/2022102804.mp4',  
        "url": 'http://localhost:8080/file1?fileUrl=http://mirror.aarnet.edu.au/pub/TED-talks/911Mothers_2010W-480p.mp4'  
    }  
];  
zipFiles(zipName, files);


```

压缩包内文件夹以`/`分隔，可以自定义需要的压缩包层级关系。
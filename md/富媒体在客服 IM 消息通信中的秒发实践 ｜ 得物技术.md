> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/h3q2AU4nG2JX4q7yb5F9qQ)

**引言**

富媒体是指在即时通信过程中传输的图片、语音、视频、文件等媒体介质的展示方式。

**一、背景**
--------

客服一站式平台旨在为得物生态内的客服域服务人员提供一站式的服务办公平台。我们有多条业务线，客服在和用户聊天的过程中，有很多场景需要发送富媒体。跟普通的文本传输相比，富媒体可以直观的让用户了解到消息内容，但是在传输过程中也面临着文件大、内存消耗大、传输过程漫长等问题。

**二、面临的挑战**
-----------

客服发送大文件 (视频、图片) 等消息给用户的大致流程如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74BZIJlQsuKP73GhAjOSmBibibjyq4UUUmE5dRuicnjf3WOPuabt3iaJRfnwGP1n4vvkuXjwSKtVcJTyUg/640?wx_fmt=png)

*   首先通过文件上传服务上传到 CDN，同时返回对应的 CDN 地址链接；
    
*   其次是获取到 CDN 地址链接，通过 IM 网关将链接返回给用户界面渲染。
    

在整个传输过程中，**前端必须等文件上传成功拿到链接之后，才能渲染**，如果传输的文件很大，客服需要会等待很长时间，这**对于客服的接线****效率****有非常大的影响**。比较理想的方式是**当客服发送文件的时候，文件立马在聊天窗口渲染**，此时渲染的不是完整的文件，而是文件的画像，比如文件的名字、封面图片，通过消息的状态进行上传状态的控制。

以视频传输为例，如果直接把视频放在缓存中展示在客服聊天内容区域，庞大的缓存会让用户的浏览器分分钟崩溃。比如大于 70M 的视频，在网络，电脑硬件等环境都较好的情况下，从读取文件到获取到首帧图片传输的过程大概需要 2~3s，如果在网络一般，同一环境下有多人在发送视频文件，或者硬件设备一般的情况下时间会更长。

**三、解决方案与成效**
-------------

### **1、将 fileReader.target.result 作为视频的 url 在页面渲染**

最初使用的方式是**在视频上传** **CDN** **时，同时截取视频首帧，然后将截取的****视频****首帧也上传到** **CDN****，再通过长链（wss）发送给客户端**，因为截取首帧是一个同步的过程，需要拿到 screenshot 的 url 之后才能渲染到页面，导致客服在点击发送的第一时间在聊天界面看不到发送出去的视频，如上图视频所示，**客服无法感知到视频发送的进度。**

**通过 FileReader 读取文件信息：**

**通过返回的文件信息进行属性设置：**

```
export function getVideoInfo(file) {
  return new Promise((resolve, reject) => {
    getFileInfo(file)
      .then(fileReader => {
        const target = fileReader.target.result
        if (/video/g.test(file.type)) {
          const video = document.createElement('video')
          video.muted = true
          video.setAttribute('autoplay', 'autoplay')
          video.setAttribute('src', target)
          video.addEventListener('loadeddata', () => {
            // ...
          })
          video.onerror = e => reject(e)
        }
      })
      .catch(e => reject(e))
  })
}

```

如上代码 video.setAttribute('src', target)，如果用 target 作为视频的 url 在页面渲染，页面会分分钟崩溃。可以看一下 1M 的视频文件，通过 readAsDataURL(file) 读取文件内容得到是一个 data:url 的 base64 字符串，**用这个****字符串****进行渲染，等于在页面加了一个 1.4M 的字符串内容**，如下图所示，这样做的后果不可想象，在文件稍微大一些的话会有更加明显的卡顿。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74BZIJlQsuKP73GhAjOSmBibib91AHhTjzZRmkqpWcKU4P6TOouCXcibXOjWM5GiavLoAreKCDtWicY4uQg/640?wx_fmt=png)

所以这个方案在开发之初就被否定了。

### **2、采用的 URL.createObjectURL(file) 获取到 URL**

在第一种方案被否定之后，又调研了 URL.createObjectURL 的实现。采用的 URL.createObjectURL(file) 获取到 URL(**这个 URL 对象表示指定的 File 对象或 Blob 对象**)，然后放到聊天数据的缓存中，便于快速发送到客服聊天窗口页面。其主要实现代码如下：

```
if (/*******/) {
    // ...
    //. blob作为预览视频的url
    state.previewVideoSrc = URL.createObjectURL(file)
    state.previewVideo = true
    state.cachePreviewVideoFile = file
    nextTick(() => {
      focus()
    })
  } else {
    // ...
  }

```

经过这个改造很明显的看到视频发出之后，可以很快的展示在页面上，让客服感知到视频发送的状态和进度，**相对于方案一，视频发送的过程有明显的提升**。渲染出来的代码效果如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74BZIJlQsuKP73GhAjOSmBibibsMohVeOB9fjLBAoA8N7PyGnQnHv5BOYUTtwMibBYx7MJW1umnRb8cgA/640?wx_fmt=png)

但是！

在给客户端发送视频信息时，**要携带首帧和视频时长，作为展示封面**，历史的做法是：

*   首先前端获取文件信息后通过 canvas 转换成图片再上传到 CDN；
    
*   在获取到首帧和文件信息之后，先上传到 CDN，返回 URL 后再通过长链发送给用户，同时更新页面的 URL 地址为 CDN 返回的真实地址。
    

**取首帧时要读取文件，既然是读取文件，还是存在一定的耗时**，如下代码片段所示，**这段耗时****任务****也会影响到客服的使用****体验**。

```
export function getVideoInfo(file, msgid?: string) {
  return new Promise((resolve, reject) => {
    getFileInfo(file, msgid)
      .then(fileReader => {
        const target = fileReader.target.result
        if (/video/g.test(file.type)) {
          const video = document.createElement('video')
          video.muted = true
          video.setAttribute('autoplay', 'autoplay')
          // target只作为url创建视频用于获取视频大小、播放时长等基本信息，不用于页面渲染
          video.setAttribute('src', target)
          video.addEventListener('loadeddata', () => {
            const canvas = document.createElement('canvas')
            canvas.width = video.videoWidth
            canvas.height = video.videoHeight
            const width = video.videoWidth
            const height = video.videoHeight
            canvas.getContext('2d')!.drawImage(video, 0, 0, width, height)
            const src = canvas.toDataURL('image/jpg')
            const imgFile = dataURLtoFile(src, `视频_${Math.random()}.png`)
            return getImgInfo(imgFile, fileReader.msgid).then(
              ({ width: imgWidth, height: imgHeight, file: imgFile, size: imgSize, src: imgSrc, msgid }) => {
                resolve({
                //  ...
                })
              }
            )
          })
          video.onerror = e => {
            // ...
            reject(e)
          }
        }
      })
      .catch(e => {
        reject(e)
      })
  })
}

```

上传视频的时候，文件服务器提供了获取首帧的方式拿到首帧图片，在链接地址上拼接对应的参数即可，如下所示：

```
// 拼接的获取图片首帧的URL地址
export const thumbSuffix = `?x-oss-process=video/snapshot,****`
export function addOssImageParams(url, isThumb = false) {
  const suffix = isThumb ? thumbSuffix : urlSuffix
  if (!url) return ''
  // ...
  return url
}

```

但在实际的使用场景中，只获取视频首帧信息是不够的，还要获取视频的宽高、播放时长等信息，并且通过网络请求发送给网关，最终在客户端展示。**读取文件****这个过程****无法避免，耗时问题还需要解决**。

### **3、Web Worker 异步读取文件信息**

通过方案二**虽然实现了文件的快速渲染，但读取文件信息如果在浏览器的主线程去做，耗时长的话，还是会阻碍客服的操作**。如果这个过程能通过异步去实现，那就很完美了。JS 虽然是单线程，但是浏览器提供了 Web Worker 的能力，让 JS 也能通过异步的方式和主线程进行通信。首先对比下浏览器主线程执行和主子线程执行的区别，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74BZIJlQsuKP73GhAjOSmBibibYhXEVaGyh2V59qqJL3yykz7o9yXBddjFNXOgkVaLdGQliaSO61EYgKA/640?wx_fmt=png)

*   浏览器主线程在执行发送文件的时候，**如果发送文件****任务****没有结束，则会阻塞其他的任务**，相当于发送期间，客服什么事情也做不了；
    
*   浏览器主子线程在执行发送文件的时候，**通过子线程读取文件，在读取文件期间，主线程可以继续执行其他的****任务****，等到子线程读取完文件通过 postMessage 发送相关的信息告知主线程文件读取完毕，主线程再开始渲染**。整个过程对于客服没有任何阻塞。
    

Web Worker 主子线程实现的流程如下：

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74BZIJlQsuKP73GhAjOSmBibibnxtzUA63LLWYYrXjVkjICdqT7mb2b07kPtWF5rMDelcRTacKPxCDWw/640?wx_fmt=png)

```
// 子线程任务
export function subWork() {
  self.onmessage = ({ data: { file } }) => {
    try {
      // 读取文件信息
      // ...
      // 发送对应信息
      self.postMessage({ fileReader: **** })
    } catch (e) {
      self.postMessage({ fileReader: undefined })
    }
  }
}

```

```
export const createWorker = (subWorker, file, resolve, reject) => {
  const worker = new Worker(URL.createObjectURL(new Blob([`(${subWorker.toString()})()`])))
  // 发到子线程
  worker.postMessage({
    file
  })
  // 监听子线程返回数据
  worker.onmessage = ({ data: { fileReader } }) => {
    resolve(fileReader)
    // 获取到结果后关闭线程
    worker.terminate()
  }
  // 监听异常
  worker.onmessageerror = function () {
    worker.terminate()
  }
}

```

```
// 创建主线程任务
export const getFileInfoFromSubWorker = files => {
  return new Promise((resolve, reject) => {
    createWorker(subWork, files, resolve, reject)
  })
}

```

通过上面的三个步骤，基本就可以在不影响客服操作的情况下获取到文件信息。获取到视频信息对象之后，再通过 URL.createObjectURL(file) 即可获取到视频相关的属性信息，如下：

```
export function getVideoInfo(file, blob, msgid?: string) {
  return new Promise((resolve, reject) => {
    if (/video/g.test(file.type)) {
      const video = document.createElement('video')
      video.muted = true
      video.setAttribute('autoplay', 'autoplay')
      // blob作为url: URL.createObjectURL(file)
      video.setAttribute('src', blob)
      video.addEventListener('loadeddata', () => {
        const width = video.videoWidth
        const height = video.videoHeight
        resolve({
          videoWidth: width,
          videoHeight: height,
          videoDuration: video.duration * 1000,
          videoFile: file,
          videoSize: file.size,
          videoSrc: blob,
          msgid
        })
      })
      video.onerror = e => {
        reject(e)
      }
    }
  })
}

```

如上所述，在获取文件对象信息之后，再通过 blob 的方式直接获取视频的宽高作为第一帧图片的宽高，二者结合即达到了在不影响客服操作的情况下，让视频发送做到了如丝般顺滑。

通过 **Web Worker+URL.createObjectURL(file)** 的方式，解决了富媒体文件发送时，不管有没有发送成功，都可以实现秒发的效果，即让视频信息先展示到聊天框，再通过发送状态来标识当前的发送进度。

**四、总结**
--------

富媒体发送在很多 IM 场景中均会涉及到，用什么样的技术实现能够让客服和用户之间沟通和交流更便捷是本文阐述的重点。通过在实际客服业务场景中的实践，本文的技术方案已经很好的解决了业务中的问题并且实际线上也一直比较稳定的在运行。从业务中发现问题，用技术手段解决问题，提升客服的解决效率，给用户带来好的体验是我们不断追求的目标，如果看了本文之后，你有更好的建议可以给我们留言。此外客服领域的技术点远不止这些，脚踏实地，一步一个脚印，相信即时通讯在客服领域的沉淀会越来越好。

**五、知识扩展**
----------

### **1、文件读取的实现差异**

URL.createObjectURL() 和 FileReader.readAsDataURL(file) 都可以取到文件的信息，为什么我们选择使用前者而非后者?

**两者的主要区别在于****：**

*   通过 FileReader.readAsDataURL(file) **获取到的是一段 data:base64 的字符串**，base64 位的字符串较大
    
*   通过 URL.createObjectURL(blob) 获会**创建一个 DOMString**，其中有包含了文件信息的 URL(指定的 File 对象或 Blob 对象)
    

**执行的时机****的不同：**

*   createObjectURL 是**立即的执行**
    
*   FileReader.readAsDataURL **是（过一段时间）异步执行**
    

**内存****的使用不同：**

*   createObjectURL 返回一段带 hash 的 url，并且一直存储在内存中，当 document 被触发了 unload 或者执行 revokeObjectURL 进行内存释放；
    

*   FileReader.readAsDataURL 返回的是 base64 的字符串，比 blob url 消耗更多的内存，不过这个数据会通过垃圾回收机制自动清除。
    

**使用选择****：**

*   用 createObjectURL 能够节省性能，获取的速度也更快；
    
*   如果设备性能足够好，而且想要获取图片的 base64，可以用 FileReader.readAsDataURL。
    

### **2、流媒体、富媒体、多媒体的概念**

流媒体、富媒体、多媒体到底有什么区别?

*   流媒体：一边使用，后台一边下载后面可能要使用到的东西。
    

*   富媒体：文字、图片、视频、音频混排的页面内容。
    
*   多媒体：图片、文字、音频、视频等资料。
    

其中流媒体是一种传输方式，富媒体是不同于纯文本的一种展示方式，多媒体是展示内容的一种手段。

* 文 / Jun
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1635241372&ver=3398&signature=pqnV24QvNH0iT7r*bZAv99-nqT7R8RpK4MKMuThd9W0pi6m5uNiMYzfwioqtdLja5D5JisitoRvoGL6j3Bo6AGLNUM*T2UUW2*8JKLbI91lbNJK1XzGTRQ2goBWx3oPQ&new=1)

（点击上方公众号，可快速关注）  

> 作者：阮一峰
> 
> www.ruanyifeng.com/blog/2018/07/web-worker.html

**一、概述**

JavaScript 语言采用的是单线程模型，也就是说，所有任务只能在一个线程上完成，一次只能做一件事。前面的任务没做完，后面的任务只能等着。随着电脑计算能力的增强，尤其是多核 CPU 的出现，单线程带来很大的不便，无法充分发挥计算机的计算能力。

Web Worker 的作用，就是为 JavaScript 创造多线程环境，允许主线程创建 Worker 线程，将一些任务分配给后者运行。在主线程运行的同时，Worker 线程在后台运行，两者互不干扰。等到 Worker 线程完成计算任务，再把结果返回给主线程。这样的好处是，一些计算密集型或高延迟的任务，被 Worker 线程负担了，主线程（通常负责 UI 交互）就会很流畅，不会被阻塞或拖慢。

**（5）文件限制**

Worker 线程无法读取本地文件，即不能打开本机的文件系统（file://），它所加载的脚本，必须来自网络。

**二、基本用法**

**2.1 主线程**

主线程采用 new 命令，调用 Worker() 构造函数，新建一个 Worker 线程。

> var worker = new Worker('work.js');

Worker() 构造函数的参数是一个脚本文件，该文件就是 Worker 线程所要执行的任务。由于 Worker 不能读取本地文件，所以这个脚本必须来自网络。如果下载没有成功（比如 404 错误），Worker 就会默默地失败。

然后，主线程调用 worker.postMessage() 方法，向 Worker 发消息。

> worker.postMessage('Hello World');
> 
> worker.postMessage({method: 'echo', args: ['Work']});

worker.postMessage() 方法的参数，就是主线程传给 Worker 的数据。它可以是各种数据类型，包括二进制数据。

接着，主线程通过 worker.onmessage 指定监听函数，接收子线程发回来的消息。

> worker.onmessage = function (event) {
> 
>  console.log('Received message' + event.data);
> 
>  doSomething();
> 
> }
> 
> function doSomething() {
> 
>  // 执行任务
> 
>  worker.postMessage('Work done!');
> 
> }

上面代码中，事件对象的 data 属性可以获取 Worker 发来的数据。

Worker 完成任务以后，主线程就可以把它关掉。

> worker.terminate();

**2.2 Worker 线程**

Worker 线程内部需要有一个监听函数，监听 message 事件。

> self.addEventListener('message', function (e) {
> 
>  self.postMessage('You said:' + e.data);
> 
> }, false);

上面代码中，self 代表子线程自身，即子线程的全局对象。因此，等同于下面两种写法。

> // 写法一
> 
> this.addEventListener('message', function (e) {
> 
>  this.postMessage('You said:' + e.data);
> 
> }, false);
> 
> // 写法二
> 
> addEventListener('message', function (e) {
> 
>  postMessage('You said:' + e.data);
> 
> }, false);

除了使用 self.addEventListener() 指定监听函数，也可以使用 self.onmessage 指定。监听函数的参数是一个事件对象，它的 data 属性包含主线程发来的数据。self.postMessage() 方法用来向主线程发送消息。

根据主线程发来的数据，Worker 线程可以调用不同的方法，下面是一个例子。

> self.addEventListener('message', function (e) {
> 
>  var data = e.data;
> 
>  switch (data.cmd) {
> 
>  case 'start':
> 
>  self.postMessage('WORKER STARTED:' + data.msg);
> 
>  break;
> 
>  case 'stop':
> 
>  self.postMessage('WORKER STOPPED:' + data.msg);
> 
>  self.close(); // Terminates the worker.
> 
>  break;
> 
>  default:
> 
>  self.postMessage('Unknown command:' + data.msg);
> 
>  };
> 
> }, false);

上面代码中，self.close() 用于在 Worker 内部关闭自身。

**2.3 Worker 加载脚本**

Worker 内部如果要加载其他脚本，有一个专门的方法 importScripts()。

> importScripts('script1.js');

该方法可以同时加载多个脚本。

> importScripts('script1.js', 'script2.js');

**2.4 错误处理**

主线程可以监听 Worker 是否发生错误。如果发生错误，Worker 会触发主线程的 error 事件。

> worker.onerror(function (event) {
> 
>  console.log([
> 
>  'ERROR: Line', e.lineno, 'in', e.filename, ':', e.message
> 
>  ].join(''));
> 
> });
> 
> // 或者
> 
> worker.addEventListener('error', function (event) {
> 
>  // ...
> 
> });

**2.5 关闭 Worker**

> // 主线程
> 
> worker.terminate();
> 
> // Worker 线程
> 
> self.close();

> // 主线程
> 
> var uInt8Array = new Uint8Array(new ArrayBuffer(10));
> 
> for (var i = 0; i < uInt8Array.length; ++i) {
> 
>  uInt8Array[i] = i * 2; // [0, 2, 4, 6, 8,...]
> 
> }
> 
> worker.postMessage(uInt8Array);
> 
> // Worker 线程
> 
> self.onmessage = function (e) {
> 
>  var uInt8Array = e.data;
> 
>  postMessage('Inside worker.js: uInt8Array.toString() =' + uInt8Array.toString());
> 
>  postMessage('Inside worker.js: uInt8Array.byteLength =' + uInt8Array.byteLength);
> 
> };

> // Transferable Objects 格式
> 
> worker.postMessage(arrayBuffer, [arrayBuffer]);
> 
> // 例子
> 
> var ab = new ArrayBuffer(1);
> 
> worker.postMessage(ab, [ab]);

> <!DOCTYPE html>
> 
>  <body>
> 
>  <script id="worker" type="app/worker">
> 
>  addEventListener('message', function () {
> 
>  postMessage('some message');
> 
>  }, false);
> 
>  </script>
> 
>  </body>
> 
> </html>

> var blob = new Blob([document.querySelector('#worker').textContent]);
> 
> var url = window.URL.createObjectURL(blob);
> 
> var worker = new Worker(url);
> 
> worker.onmessage = function (e) {
> 
>  // e.data === 'some message'
> 
> };

有时，浏览器需要轮询服务器状态，以便第一时间得知状态改变。这个工作可以放在 Worker 里面。

> function createWorker(f) {
> 
>  var blob = new Blob([f.toString()]);
> 
>  var url = window.URL.createObjectURL(blob);
> 
>  var worker = new Worker(url);
> 
>  return worker;
> 
> }
> 
> var pollingWorker = createWorker(function (e) {
> 
>  var cache;
> 
>  function compare(new, old) { ... };
> 
>  setInterval(function () {
> 
>  fetch('/my-api-endpoint').then(function (res) {
> 
>  var data = res.json();
> 
>  if (!compare(data, cache)) {
> 
>  cache = data;
> 
>  self.postMessage(data);
> 
>  }
> 
>  })
> 
>  }, 1000)
> 
> });
> 
> pollingWorker.onmessage = function () {
> 
>  // render data
> 
> }
> 
> pollingWorker.postMessage('init');

上面代码中，Worker 每秒钟轮询一次数据，然后跟缓存做比较。如果不一致，就说明服务端有了新的变化，因此就要通知主线程。

**六、实例： Worker 新建 Worker**

Worker 线程内部还能再新建 Worker 线程。下面的例子是将一个计算密集的任务，分配到 10 个 Worker。

主线程代码如下。

> var worker = new Worker('worker.js');
> 
> worker.onmessage = function (event) {
> 
>  document.getElementById('result').textContent = event.data;
> 
> };

Worker 线程代码如下。

> // worker.js
> 
> // settings
> 
> var num_workers = 10;
> 
> var items_per_worker = 1000000;
> 
> // start the workers
> 
> var result = 0;
> 
> var pending_workers = num_workers;
> 
> for (var i = 0; i < num_workers; i += 1) {
> 
>  var worker = new Worker('core.js');
> 
>  worker.postMessage(i * items_per_worker);
> 
>  worker.postMessage((i + 1) * items_per_worker);
> 
>  worker.onmessage = storeResult;
> 
> }
> 
> // handle the results
> 
> function storeResult(event) {
> 
>  result += event.data;
> 
>  pending_workers -= 1;
> 
>  if (pending_workers <= 0)
> 
>  postMessage(result); // finished!
> 
> }

上面代码中，Worker 线程内部新建了 10 个 Worker 线程，并且依次向这 10 个 Worker 发送消息，告知了计算的起点和终点。计算任务脚本的代码如下。

> // core.js
> 
> var start;
> 
> onmessage = getStart;
> 
> function getStart(event) {
> 
>  start = event.data;
> 
>  onmessage = getEnd;
> 
> }
> 
> var end;
> 
> function getEnd(event) {
> 
>  end = event.data;
> 
>  onmessage = null;
> 
>  work();
> 
> }
> 
> function work() {
> 
>  var result = 0;
> 
>  for (var i = start; i < end; i += 1) {
> 
>  // perform some complex calculation here
> 
>  result += 1;
> 
>  }
> 
>  postMessage(result);
> 
>  close();
> 
> }

**七、API**

**7.1 主线程**

浏览器原生提供 Worker() 构造函数，用来供主线程生成 Worker 线程。

> var myWorker = new Worker(jsUrl, options);

Worker() 构造函数，可以接受两个参数。第一个参数是脚本的网址（必须遵守同源政策），该参数是必需的，且只能加载 JS 脚本，否则会报错。第二个参数是配置对象，该对象可选。它的一个作用就是指定 Worker 的名称，用来区分多个 Worker 线程。

> // 主线程
> 
> var myWorker = new Worker('worker.js', { name : 'myWorker' });
> 
> // Worker 线程
> 
> self.name // myWorker

Worker() 构造函数返回一个 Worker 线程对象，用来供主线程操作 Worker。Worker 线程对象的属性和方法如下。

*   Worker.onerror：指定 error 事件的监听函数。
    
*   Worker.onmessage：指定 message 事件的监听函数，发送过来的数据在 Event.data 属性中。
    
*   Worker.onmessageerror：指定 messageerror 事件的监听函数。发送的数据无法序列化成字符串时，会触发这个事件。
    
*   Worker.postMessage()：向 Worker 线程发送消息。
    
*   Worker.terminate()：立即终止 Worker 线程。
    

**7.2 Worker 线程**

Web Worker 有自己的全局对象，不是主线程的 window，而是一个专门为 Worker 定制的全局对象。因此定义在 window 上面的对象和方法不是全部都可以使用。

Worker 线程有一些自己的全局属性和方法。

*   self.name： Worker 的名字。该属性只读，由构造函数指定。
    
*   self.onmessage：指定 message 事件的监听函数。
    
*   self.onmessageerror：指定 messageerror 事件的监听函数。发送的数据无法序列化成字符串时，会触发这个事件。
    
*   self.close()：关闭 Worker 线程。
    
*   self.postMessage()：向产生这个 Worker 线程发送消息。
    
*   self.importScripts()：加载 JS 脚本。
    

（完）
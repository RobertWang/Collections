> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIyNTY4NjU0OQ==&mid=2247506645&idx=3&sn=0171ad6ebb7e3a1f4763b4d6eb32ce63&chksm=e8797fafdf0ef6b9c27ddfff1456962c2bc8a6acdde979225790cece4be013db619c1bb0a196&mpshare=1&scene=1&srcid=060678Rmaf4zVyAojXSgPQf1&sharer_sharetime=1622991663396&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

背景
--

你是不是经常遇到这个场景，测试找你要测试包，你还在忙着手动打包，然后上传云存储，再然后拿着链接，发到群里通知测试。也可能是你已经用上了 Jenkins 实现，可我今天要做的就是教你如何自己实现呢？这样做有什么优势呢？

*   没有环境的差异，云端打包固然好，但总会遇到环境问题导致的打包失败，且不好修复
    
*   100% 的可定制化，随心所欲
    
*   连云存储都是自己的，不担心别的平台对你的测试包进行反编译，窃取技术
    

一行代码
----

```
// Debug 包
node gradle-build.js -d
// Release包
node gradle-build.js -r
```

背后如何设计的呢？请看

流程设计
----

![](https://mmbiz.qpic.cn/mmbiz_png/GIQWuu6gWEQZ7zyy5hT4Gs3qpUWXib0lpVCDC12zYtYan8QOlS2FHUrstfT8ic5OPG4OOCcia3yQbD04dulpvbnlw/640?wx_fmt=png)

*   命令行 Gradle 打包
    
*   失败后查找原因，然后再重试
    
*   成功后直接上传自己搭建的云存储
    
*   然后将 Url 返回
    
*   收到 Url 后直接调用 DingDing Robot Api 通知到群里
    

总体就是三步，第一打包，第二上传，第三通知，那么我们如何能做到这三步呢？请往下看

Step One 打包
-----------

打包你是不是想到了命令行输入 gradle assembleDebug，是的，我们也会用到它，Shell 脚本固然简单，但它缺乏很多有力的支持，比如我们自定义一个执行脚本如：gradlew -d（打 Debug 包），当然我举例不是很贴切，就是想然你理解一下，脚本语言有很多，建议你用自己熟悉的语言，其实用什么并不重要，重要的是我们实现了什么，那我是如何实现的呢？我选择了熟悉的 nodejs 脚本，请看实现：

**安装依赖**

```
npm i commander
npm i shelljs
```

**脚本具体实现**

```
const shell = require('shelljs');
const dingding = require('./dingdingNotification')
const file = require('./upload')

var program = require('commander');

program
  .version('0.0.1')
  .option('-d, --debug', 'gradlew assembleDebug')
  .option('-s, --sandbox', 'gradlew assembleSandbox')
  .option('-r, --release', 'gradlew assembleRelease')
  .parse(process.argv);

console.log('正在执行:');

//debug版本打包流程定义
if (program.debug) {
    console.log('  - debug');
    //执行gradle assembleDebug脚本
    builds('gradle assembleDebug',(code)=>{
        if(code==1){
            console.log('执行完毕');
            //build成功，上传文件
            file.postFile({ file: './flutter.png', content_type: 'png' },function(response){
                console.log('response:'+response);
                //钉钉通知
                dingding.talk("Android Debug 安装包构建完成，请点击下方链接下载：Http://apk.ibaozi.cn/tmp/")
            })
        }else{
            console.log('执行失败');
         }
    })
}
//sandbox版本打包流程定义
if (program.sandbox) {
    console.log('  - sandbox');
}
//release版本打包流程定义
if (program.release) {
    console.log('  - release');
}
// 命令执行，返回1说明build成功
function builds(task, call) {
    if (shell.exec(task).code == 0) {
        call(1)
    } else
        call('gradle app:assemble构建失败')
}
```

*   shelljs 可以实现在命令行执行脚本，脚本执行成功返回 1，失败返回构建失败
    
*   commander 帮助我们检测 js 脚本输入参数，并安装参数执行对应脚本，我这里写了 debug 作为示例
    

Step Two 上传
-----------

上传看似简单其实我们设计了两个地方，一个是上传脚本，一个是 Server 端存储，考虑到存储端实现起来确实复杂，我有个特别简单的实现分享给你，也许搭起来费点力气，可如果搭起来就会很舒服，请先看效果图：

手动上传实现

![](https://mmbiz.qpic.cn/mmbiz_png/GIQWuu6gWEQZ7zyy5hT4Gs3qpUWXib0lpxgckDRFB7g4oBpScQeyxVX61cD4fMHIhmNwz2khttalcf6sFUddO2A/640?wx_fmt=png)

脚本上传实现

```
npm i needle // 首先安装needle 用它上传文件哦

// js脚本实现

var needle = require('needle');

// { file: './flutter.png', content_type: 'png' }
function postFile(file,callBack){
    needle.post('http://localhost:4000/upload', {
        path:'test',
        version:'1.0.0',
        version_code:20,
        file: file
    }, { multipart: true }, function(err, resp, body) {
        console.log(err, resp, body)
        callBack(resp)
    });
}

module.exports = {
    postFile:postFile
}

// 上传调用
const file = require('./upload')

file.postFile({ file: './flutter.png', content_type: 'png' },function(response){
                console.log('response:'+response);
            })
```

上传后的目录结构

![](https://mmbiz.qpic.cn/mmbiz_png/GIQWuu6gWEQZ7zyy5hT4Gs3qpUWXib0lpwAXicSxFzfHFKvKiblm8ys8HXtoiaXHzJ7hO7iaLAfXrbKia1AsVQ7icENsg/640?wx_fmt=png)

点击就可以下载，也可以通过链接查看，看着是不是还算可以，那我们都做了什么呢，很简单的，来跟我一起实现一下：

首先安装依赖

```
npm i express // 提供静态存储文件夹public，提供下载支持
npm i serve-index // 负责将public目录文件转换为文件列表展示
```

对的就这俩就行了，接下我们改造下 express 默认生成的项目代码

```
var createError = require('http-errors');
var express = require('express');
var path = require('path');
var cookieParser = require('cookie-parser');
var logger = require('morgan');
const serveIndex = require('serve-index')

var usersRouter = require('./routes/users');
var indexRouter = require('./routes/index');

var app = express();
var rootPath = path.join(__dirname, 'public');

// view engine setup
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'jade');

app.use(logger('dev'));
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
app.use(cookieParser());
app.use('/ftp', express.static(rootPath), serveIndex(rootPath, {'icons': true}));

app.use('/', indexRouter);
app.use('/users', usersRouter);

// catch 404 and forward to error handler
app.use(function(req, res, next) {
  next(createError(404));
});

// error handler
app.use(function(err, req, res, next) {
  // set locals, only providing error in development
  res.locals.message = err.message;
  res.locals.error = req.app.get('env') === 'development' ? err : {};

  // render the error page
  res.status(err.status || 500);
  res.render('error');
});

module.exports = app;
```

最主要的就是这一行

```
app.use('/ftp', express.static(rootPath), serveIndex(rootPath, {'icons': true}));
```

将 public 文件夹委托给 serveIndex，并新增一个目录地址 ftp，浏览地址如下：

http://localhost:4000/ftp/

浏览搞定了，就差上传了，我们这里用到一个库 formidable，它来帮助我们处理上传过来的文件流处理逻辑

```
npm i formidable
```

```
var express = require('express');
var router = express.Router();
const formidable = require('formidable');
var path = require('path');
var fs = require('fs');
const { isString } = require('util');

var fileRootPath = path.resolve(__dirname,'../public')
console.log(fileRootPath);

/* GET home page. */
router.get('/', function(req, res, next) {
    res.send(`
    <h2>Julive <code>"File Storage"</code> For You</h2>
    <form action="/upload" enctype="multipart/form-data" method="post">
      <h3>请选择文件</h3>
      <div>文件: <input type="file"  /></div>
      <h3>点击提交</h3>
      <input type="submit" value="开始上传" />
    </form>
  `);
});

router.post('/upload', async (req, res) => {
  try {
    const form = formidable({ multiples: true,uploadDir: fileRootPath});
    var path = "/"

    form.once('error', console.error);

    form.on('field', (name, value) => {
        if(){
            if(!isNull(value)){
                path = "/"+value+"/"
            }
        }
    });
    form.on('file', (name, file) => {
        fs.rename(file.path, form.uploadDir + path + file.name,(error)=>{
            if(error){
                next(err);
                return;
            }
        });
    });

    form.parse(req, (err, fields, files) => {
      if (err) {
        next(err);
        return;
      }
      console.log("fields.path:"+fields.path)
      console.log("fields.version:"+fields.version)
      console.log("fields.version_code:"+fields.version_code)
      files.url = 'http://file.ibaozi.cn/ftp/'
      res.json({ fields, files });
    });
  } catch (err) {
      res.status(500).send(err);
  }
});

function isNull(str){
    if ( str == ""|| str == null) return true;
    var regu = "^[ ]+$";
    var re = new RegExp(regu);
    return re.test(str);
}


module.exports = router;
```

通过 formidable，我们可以在各个回调里处理相应的逻辑，如 once 打印错误信息，在 form.on('field', 里处理的文件的路径逻辑等等吧。上传调用 http://localhost:4000/upload 接口即可。

还有个 get 接口发现了吧，那个就是手动上传的页面逻辑哦。到此，Server 端也实现完了

Step Three 通知
-------------

通知的实现如下，这里用到 flyio，来发起请求，为什么用它？因为这个早就写好的脚本，就懒得再改了

```
npm i flyio
```

通知

```
const fly = require("flyio");

function dingtalk(content){

    const url = "https://oapi.dingtalk.com/robot/send?access_token=40e67411719fbd584134a7deefc1c643f2d0916f8efcefb12e7759591fa7e637";

    if(content==null){
        content = "Android Apk 安装包构建成功,下载地址：https://www.pgyer.com/gIEm  "
    }

    console.log(content);

    const data = {
        "msgtype": "text", "text": {
            "content": content
        },
        "at": {
            "isAtAll": true
        }
    };

    fly.config.headers = {"Content-Type": "application/json"};

//    fly.post(url, data).then((response) => {
//        console.log(response)
//    });
}

module.exports = {
    talk:dingtalk
}
```

好了结束了，就这么多。

总结
--

我们回顾一下，本期主要是介绍了通过一行命令完成 Android 打包 -> 上传 -> 发测试包通知的过程，主要是用到了我熟悉的 nodejs 脚本，如果你还对 nodejs 不了解，建议抽时间学习下，也希望对你有帮助，这个云存储目前没有安全方面的访问控制，用的话建议公司内部私网使用，也可以像我，买个服务器放上去，自己没事就折腾一下，哈哈。

项目开源地址
------

https://github.com/ibaozi-cn/file-cloud-storage  
  

**---END---**
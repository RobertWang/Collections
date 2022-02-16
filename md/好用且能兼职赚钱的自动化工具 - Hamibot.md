> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIzMzMzOTI3Nw==&mid=2247505589&idx=1&sn=c4c1623721a669e7a4c1d7f1a2795e75&chksm=e885b657dff23f413ea5e1616f8da89149bc66b6ffb8924cba1aa92f6966644b8af63f17f4da&mpshare=1&scene=1&srcid=0216EYVEnyGek8FSBQHDVoDA&sharer_sharetime=1644980181142&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> 自动化软件 Hamibot

最近发现一款神器「 Hamibot 」，它是一款 Android 端的自动化工具，它基于 AutoJS 源码进行的二次开发  

官方网站：https://docs.hamibot.com/  

Hamibot 脚本市场提供了很多脚本，我们都可以免费导入进行试用

首先，我们在 PC Web 的控制台添加一个机器人，然后使用手机通过「 配对码 」进行配对，接着从脚本市场选择一个功能脚本导入，最后在控制台运行脚本就可以在手机上运行一系列自动化操作了

![](https://mmbiz.qpic.cn/mmbiz_png/atOH362BoytiaJiarzGnfd5ZUGibSPrdrtscvsZVD9PhUOZSZpDlZVicKM2gsVypFvCIIgE4Uicm4K7ib9zxr4HchDlA/640?wx_fmt=png)

当然，我们也可以根据官方文档编写一些实用的脚本上传到平台上，获取一些佣金提成

由于 Hamibot 基于 AutoJS，语法都大同小异，所以本篇文章将只介绍 Hamibot 一些实用的使用技能

1. 实用技能
-------

1-1  App 保活在线

自动化脚本运行实际上是 App 接受到 Web 端 Hamibot 控制台发送的指令，进而驱动手机进行的一系列动作，所以保证 Hamibot App 在后台一直运行变得非常重要  

程序保活主要包含 6 个方面，分别是：  

*   前台服务
    
    打开 Hamibot App，从侧边栏中开启「 前台服务 」功能
    
*   悬浮窗
    
    打开 Hamibot App，从侧边栏中开启「 悬浮窗 」功能
    
*   应用自启动
    
    手机进入到设置中，搜索关键字「 应用管理 」，选择 Hamibot 应用，开启「 自启动 」功能
    
*   不锁定屏幕
    
    首先手动开启开发者选项（ 不同厂商系统手机的开启方式不一致，一般是在系统版本连续点击多次就可以开启开发者选项 ），然后在开发者选项中开启「 不锁定屏幕 」功能，这样在手机充电时，屏幕不会休眠
    
*   关闭省电策略
    
    在手机应用设置中，选择省电策略为「 不限制 」，这样 App 会在后台一直运行
    
*   保证手机一直联网
    
    只有手机一直联网，Hamibot 控制台才能将指令传输给设备进行运行
    

1-2  启动应用

Hamibot 中的 app 模块提供了很多函数用于操作 App 应用

其中，启动一个应用有 3 种方式

```
# 启动应用的3种方式
# 方式一：通过应用的名称
# 比如：某宝、某多、某条
app.launchApp('某宝');

# 方式二：通过应用的包名
# 包名通过adb命令或者Android Studio 去解析 APK 获取
# 比如：启动 Hamibot 应用
app.launch('com.hamibot.hamibot');

# 方式三：与第二种类似，全局函数
# 通过应用包名启动应用
launchPackage('com.hamibot.hamibot');
```

1-3  触摸操作

触摸操作是基于屏幕坐标进行点击、长按、滑动等操作，但是该操作需要 Android 7.0 以上或 Root 权限才能有效

PS：对于一些基于元素的点击操作失效的场景，换成触摸操作反而能很好的解决问题

```
// 使用触摸操作点击某个元素
// 获取某个元素
var widget = id('xxx').findOne();

// 获取其中心位置，执行点击操作
click(widget.bounds().centerX(), widget.bounds().centerY());
```

1-4  控件操作  

控件操作为编写脚本的核心内容，AutoJS 和 Hamibot 官方文档都非常的详细地列出了 UiSelector、UiObject、UiCollection 的使用方法

官方文档：

https://docs.hamibot.com/reference/widgetsBasedAutomation/

1-5  网络请求

结合网络请求，能将爬虫与自动化完美地结合在一起  

这里以常见的 GET、POST 请求为例

```
// 1-GET请求
// 网络请求，获取响应值
var r = http.get('www.baidu.com');
// 响应码
log('code = ' + r.statusCode);
// 响应体（字符串）
log('html = ' + r.body.string());

// 2-POST 请求
var url = 'http://www.**.com/api/login';

//直接传入一个字典作为参数
r = http.postJson(url, {
  username: 'xag',
  password: '123456'
});

//获取请求的响应并弹出吐司
toastLog(r.body.string());
```

官网提供了网络请求的基础函数，大家可以自行去扩展使用

1-6  线程

脚本默认是在主线程中运行的，我们可以将一些耗时的操作添加到线程中执行

```
auto.waitFor();

//开启一个子线程
threads.start(function() {
  //在新线程执行的代码
  while (true) {
    log('子线程');
  }
});

//主线程
while (true) {
  log('脚本主线程');
}
```

2. 拓展一下  

----------

在实际使用 Hamibot 的过程中，发现其提供的定时任务没有 AutoJS 使用起来方便

比如，如果我想固定在每一天的某个时间执行某个脚本，可惜的是，官方提供的任务功能没有实现

![](https://mmbiz.qpic.cn/mmbiz_png/atOH362BoytiaJiarzGnfd5ZUGibSPrdrts41l2Ipz1h0aPcL1KhXzjpmNENGkUviamia5VicGIKC73D3fAF4sUMNOug/640?wx_fmt=png)

这时候，我们就需要我们在代码中自己去实现这个功能点了

```
function setScheduledTask(hour, minute, callTask) {
    let taskTime = new Date();
    taskTime.setHours(hour);
    taskTime.setMinutes(minute);
    let timeDiff = taskTime.getTime() - (new Date()).getTime(); // 获取时间差
    timeDiff = timeDiff > 0 ? timeDiff : (timeDiff + 24 * 60 * 60 * 1000);
    setTimeout(function() {
        callTask(); 
        setInterval(callTask, 24 * 60 * 60 * 1000); // 24小时为循环周期
    }, timeDiff); 
}

//获取配置文件中设置的时间（小时、分钟）
const { hour,minute } = hamibot.env;
toastLog(hour); 
toastLog(minute)

function create_thread_and_do_something(){
        //定义子线程
        var my_thread = threads.start(function(){
                   console.log("开始执行子线程。 。。。")
                    ...
                        console.log("结束执行子线程。。。。")
        })
}

// 每天某个时间开启一个子线程，执行一个任务
setScheduledTask(hour, minute, create_thread_and_do_something);
```

3. 最后
-----

上面内容列出了使用 Hamibot 编写自动化脚本需要掌握的一些功能点

Hamibot 和 AutoJS 的脚本语法基本类似，但是 Hamibot 在易用性、稳定性、群控方面更有优势一点，更多复杂的功能大家可以自行去查阅官方文档去拓展

如果你觉得文章还不错，请大家 点赞、分享、留言 下，因为这将是我持续输出更多优质文章的最强动力！
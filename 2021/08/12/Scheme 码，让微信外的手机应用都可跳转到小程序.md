> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1628750033&ver=3247&signature=Jg0ArF*mCdE71SqjpwKg0iGX67i5j*5BZ9Ii3qcAJC7aB3OAmKWczq35voGbc096kU5vcIzTCwN4*MK8tM5mOvCzuWJPHQ8xd-i09VluVl0THlrZmHAnyLyyYiTbzaWP&new=1)

> 在今后的时间，我将会持续就云开发与微信生态写一系列的技术与运营方面的文章，欢迎一起交流获取了小程序的 sche

> 在今后的时间，我将会持续就云开发与微信生态写一系列的技术与运营方面的文章，欢迎一起交流

获取了小程序的 scheme 码，就可以像打开网页链接一样，通过短信、邮件、外部网页等微信以外的渠道拉起小程序，URL Scheme 链接形式如`weixin://dl/business/?t= *TICKET*`。  

### Scheme 码的说明

scheme 码可以自定义进入的小程序的页面路径，也可以携带参数，还可以设置是永久有效还是到期失效 (比如两小时内或 7 天内失效)。携带参数、设置有效时间以及场景值（通过 URL Scheme 打开小程序的场景值为 1065），让我们可以使用 Scheme 码进行活动推广、渠道宣传、返利分享等的追踪。**Scheme 码只对国内非个人主体小程序开放**。

安卓手机不能直接识别 URL Scheme，我们需要使用 H5 页面中转，如`location.href = 'weixin://dl/business/?t= *TICKET*'`，这种跳转方式可以在用户打开 H5 时立即调用（无需其他操作），也可以在用户触发事件后调用。而 iOS 除了可以使用这种方式外，还能识别 URL Scheme，可直接在短信通过 Scheme 跳转小程序。

值得一提的是，URL Scheme 不支持微信内打开小程序，也就是说不能通过微信聊天、朋友圈、公众号、微信浏览器等渠道用 URL Scheme 打开小程序。如果在微信内打开，会提示 “对不起，当前页面无法访问”。

### Scheme 码获取与使用

#### 1、使用云调用快速获取 scheme 码

我们可以通过云调用`cloud.urlscheme.generate`来获取 Scheme 码，使用微信开发者工具新建一个 Node.js 云函数，名称可以命名为 urlscheme，然后用编辑器打开 index.js 输入以下代码，保存代码后，右键 index.js，在弹窗中选择” 云函数增量上传：更新文件 “。

```
const cloud = require('wx-server-sdk')
cloud.init({
 env: cloud.DYNAMIC_CURRENT_ENV, 
})

exports.main = async (event, context) => {
  try {
    const result = await cloud.openapi.urlscheme.generate({
      jumpWxa: {
        path: '', //通过scheme码进入的小程序页面路径，必须是已经发布的小程序存在的页面，不可携带query。path为空时会跳转小程序主页。
        query: '' //通过scheme码进入小程序时的query，最大128个字符，只支持数字，大小写英文以及部分特殊字符：!#$&'()*+,/:;=?@-._~
      },
      isExpire: false, //生成的scheme码类型，到期失效：true，永久有效：false
    })   
    return result.openlink
  } catch (err) {
    return err
  }
}

```

然后我们可以在开发者工具的控制台里输入如下代码来调用这个云函数，就能打印返回的 Scheme 码了，比如`weixin://dl/business/?t=zUHMETgiMak`。

```
wx.cloud.callFunction({name:"urlscheme"}).then(res=>{console.log(res.result)})


```

由于到期失效`isExpire`设置为`false`，所以生成的 scheme 码会长期有效；而由于 path 为空，所以打开 scheme 码时会跳转到小程序首页。

#### 2、scheme 码的使用

如果你是 iOS 系统的手机、平板，可以把这个 Scheme 码复制粘贴到短信、备忘录、浏览器的地址栏、邮件等等几乎所有直接外链跳转的 App 里，就可以直接跳转到小程序。

我们也可以把这个 Scheme 码放到网页里，通过`location.href`来进行跳转。比如我们可以使用 VS Code 新建一个 schemesimple.html 的文件，复制并保存下面的代码，然后将 schemesimple.html 上传到静态托管或者服务器上，无论是安卓还是 iOS 都可以通过这个链接来跳转到微信小程序。

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta >
  <title>网页跳转小程序</title>
  <script>
    window.location.href="weixin://dl/business/?t=zUHMETgiMak"
  </script>
</head>
<body>
  这个只适用于微信外跳转到小程序
</body>
</html>


```

比如大家可以在微信外的其他**手机**应用（如浏览器、知乎、短信、邮件等）打开链接 https://i.hackweek.org/public/schemesimple.html，就能跳转到小程序首页了。

> 值得强调的是，这个带有你的 scheme 码的静态网页对服务器**没有**额外要求，比如不必一定放在云开发或者腾讯云的静态托管，也可以放在腾讯云、阿里云或者其他服务器上或者其他静态托管上。

#### 3、短期有效和长期有效 scheme 码

有效时间不超过 31 天的 Scheme 为**短期有效 Scheme**，单个小程序每日生成短期有效 Scheme 上限为 50 万个，有效时间超过 31 天的 Scheme 或永久有效的 Scheme 为**长期有效 Scheme**，单个小程序总共可生成长期有效 Scheme 上限为 10 万个。

所以如果是带有活动以及用户 openid、点击时间、地址、来源等信息的页面建议使用短期有效 scheme 码，而固定的页面或可以直接访问小程序首页的更建议使用长期有效 Scheme 码。

下面是记录用户 openid、小程序端传入的参数 productID 以及生成后 2 小时失效的云函数案例，将以下代码输入到 urlscheme 的 index.js 之后，保存代码后，右键 index.js，在弹窗中选择” 云函数增量上传：更新文件 “。

```
const cloud = require('wx-server-sdk')
cloud.init({
 env: cloud.DYNAMIC_CURRENT_ENV, 
})

exports.main = async (event, context) => {
  const wxContext = cloud.getWXContext()
  const {OPENID} = wxContext
  const {productID} = event

  try {
    const result = await cloud.openapi.urlscheme.generate({
      jumpWxa: {
        path: '/pages/home/home',
        query: `openid=${OPENID}&id=${productID}`
      },
      isExpire: true,
      expire_time: Math.round(new Date().getTime()/1000) + 3600*2
    })
    return result.openlink
  } catch (err) {
    return err
  }
}

```

我们可以在控制台的 console 输入以下代码，调用 urlscheme 云函数时传入参数 productID，就可以返回 scheme 码了。

```
wx.cloud.callFunction({name:"urlscheme",data:{productID:"JD1001"}}).then(res=>{console.log(res.result)})


```

通过前面的实战，我们了解到，通过将 scheme 码放置到静态网页里，我们就能在手机且非微信的其他应用通过网页跳转到小程序了。我们还需要考虑非手机或微信的兼容性，这个我们会在后面的章节里继续介绍。
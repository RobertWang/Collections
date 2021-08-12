> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/f4110132bf97)

获取了小程序的`scheme`码，就可以像打开网页链接一样，通过短信、邮件、外部网页等微信以外的渠道拉起小程序，`URL Scheme链接形式如weixin://dl/business/?t= *TICKET*。`

1. 先用`postman`获取`auth.getAccessToken`（get 方式）  
[https://developers.weixin.qq.com/miniprogram/dev/api-backend/open-api/access-token/auth.getAccessToken.html](https://links.jianshu.com/go?to=https%3A%2F%2Fdevelopers.weixin.qq.com%2Fminiprogram%2Fdev%2Fapi-backend%2Fopen-api%2Faccess-token%2Fauth.getAccessToken.html)  
固定 get 请求地址：

```
 https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=APPID&secret=APPSECRET


```

2. 获取`openLink`(post 方式)`urlscheme.generate`  
[https://developers.weixin.qq.com/miniprogram/dev/api-backend/open-api/url-scheme/urlscheme.generate.html](https://links.jianshu.com/go?to=https%3A%2F%2Fdevelopers.weixin.qq.com%2Fminiprogram%2Fdev%2Fapi-backend%2Fopen-api%2Furl-scheme%2Furlscheme.generate.html)  
第一步拿到的 token，在 postman 里面 post 请求：  
`https://api.weixin.qq.com/wxa/generatescheme?access_token=ACCESS_TOKEN`  
ACCESS_TOKEN = 第一步拿到的`token`  
body 里面请求参数：

```
{
    "jump_wxa":
    {
        "path": "/pages/home/home",// 项目入口
        "query": ""
    },
    "is_expire":false,// 去掉有效期
    //"expire_time":1606737600
}


```

post 请求之后会返回生成的小程序`scheme码`=`openlink`  

![](http://upload-images.jianshu.io/upload_images/20187175-62f6e46c7dde44ea.png) 获取小程序的 Scheme 码. png
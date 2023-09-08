> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.imooc.com](https://www.imooc.com/article/301474)

> 发布流程分析 抖音创作服务平台：https://media.douyin.com/#/upload 1、获取认证信息 API：https://media.douyin.com/we

![](https://img4.sycdn.imooc.com/5e5f59280001334a12400737.jpg)

### 发布流程分析

> 抖音创作服务平台：[https://media.douyin.com/#/upload](https://media.douyin.com/#/upload)

#### 1、获取认证信息

*   API：`https://media.douyin.com/web/api/media/upload/auth/`
*   请求头：
    *   `cookie`：`你账号的 cookie`

![](https://img3.sycdn.imooc.com/5e5f592900016c4912400646.jpg)

#### 2、获取视频上传参数

*   API：`https://vas-lf-x.snssdk.com/video/openapi/v1/?action=GetVideoUploadParams&use_edge_node=1`
*   请求头：
    *   `authorization`: `${auth}`
    *   `x-tt-access`: `${ak}`

![](https://img4.sycdn.imooc.com/5e5f592a000103e512400641.jpg)

#### 3、上传视频

*   API：`https://` + `${tos_host}` + `${oid}`
*   请求方式：POST
*   请求头：
    *   `Content-Typ`：`application/octet-stream`
    *   `Authorization`：`${tos_sign}`
*   请求体：文件字节流

![](https://img3.sycdn.imooc.com/5e5f592a00011a4812400651.jpg)

#### 4、更新视频上传信息

*   API：`https://vas-lf-x.snssdk.com/video/openapi/v1/?action=UpdateVideoUploadInfos&` + `${extra_param}`
*   请求方式：POST
*   请求头：
    *   `authorization`: `${auth}`
    *   `x-tt-access`: `${ak}`
*   请求体：

```
{
    "vid": "${vid}",
    "oid": "${oid}",
    "token": "${token}",
    "poster_ss": 0,
    "is_exact_poster": true,
    "user_reference": ""
}


```

![](https://img2.sycdn.imooc.com/5e5f592b0001e08312400653.jpg)

#### 5、发布视频

*   API：`https://media.douyin.com/web/api/media/aweme/create/`
*   请求方式：POST
*   请求头：
    *   `cookie`：`${cookie}`
    *   `csrf_token`: `${csrf_token}`
*   请求表单：
    *   `video_id`: `${vid}`
    *   `poster`: `${oid}`
    *   `poster_delay`: 0
    *   `text`: #音乐 遇见就是一种缘分
    *   `text_extra`: [{“start”:0,“end”:3,“user_id”:"",“type”:1,“hashtag_name”:“音乐”}]
    *   `challenges`: [“1550712576368642”]
    *   `mentions`: []
    *   `visibility_type`: 0
    *   `third_text`: 遇见就是一种缘分
    *   `download`: 0
    *   `upload_source`: 1
    *   `mix_id`:
    *   `mix_order`:
    *   `is_preview`: 0

![](https://img4.sycdn.imooc.com/5e5f592d000111ff12400647.jpg)

### 体验地址

*   [https://anoyi.com/dy/top](https://anoyi.com/dy/top)

点击查看更多内容
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg3MDYwMzkwMw==&mid=2247484394&idx=1&sn=ea74d932d280e8965dc757b47f774174&chksm=ce8a0efaf9fd87ec69aa850d71a7281d366b9f973e3fb58149fc8fbf88d248e2f2acbcc26f15&mpshare=1&scene=1&srcid=0628lDkoDyah7B4SatwFOEB6&sharer_sharetime=1624875571226&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

大家好，近年来，随着移动智能设备的快速普及，移动多端统一开发框架已成为一个热门话题。这里为大家整理了 5 个 Flutter 优质的开源项目，希望对大家有帮助  

1.Flutter 精仿抖音
--------------

#### ◆   应用截图

![](https://mmbiz.qpic.cn/mmbiz_png/OKUeiaP72uRz2iaPnnia07Qu4a5oz37yCd0XOhXib4y1yyicVBkWYoQq62eI35jkJq6z4I7f1d4bpqDiaVTx11YoDNDw/640?wx_fmt=png)

#### ◆   实现功能

*   上下刷视频，视频会自动加载封面
    
*   左右滑动去搜索与个人中心
    
*   双击冒爱心点赞
    
*   看评论
    
*   切换底部 Tabbar
    

#### ◆   项目结构

依赖：

```
  # 加载动画库(好像改版之后就没用到了)
  flutter_spinkit: ^4.1.2
  # Bilibili开源的视频播放组件
  fijkplayer: ^0.8.3
  # 基础的透明动画点击效果
  tapped: any
  # map安全取值
  safemap: any


```

主要文件：

```
./lib
├── main.dart
├── mock
│   └── video.dart # 假数据
├── other
│   └── bottomSheet.dart # 修改了系统BottomSheet的高度
├── pages
│   ├── cameraPage.dart # 拍摄页（没有实际功能）
│   ├── followPage.dart  # 略
│   ├── homePage.dart # 主页面，包含tikTokScaffold的实际应用功能
│   ├── msgDetailListPage.dart # 略
│   ├── msgPage.dart # 略
│   ├── searchPage.dart # 略
│   ├── todoPage.dart # 略
│   ├── userDetailPage.dart # 略
│   ├── userPage.dart # 略
│   └── walletPage.d # 略
├── style
│   ├── style.dart # 全局文字大小与颜色
│   └── text.dart # 主要的几个文字样式
└── views
    ├── backButton.dart # iOS形状的返回按钮组件
    ├── loadingButton.dart # 可以设置为载入样式的按钮组件
    ├── selectText.dart # 可设置为“选中”或者“未选中”样式的文字
    ├── tikTokCommentBottomSheet.dart # 仿Tiktok评论样式
    ├── tikTokHeader.dart # 仿Tiktok顶部切换组件
    ├── tikTokScaffold.dart # 仿Tiktok核心脚手架，封装了手势与切换等功能，本身不包含UI内容
    ├── tikTokVideo.dart # 仿Tiktok的视频UI样式封装，不包含视频播放
    ├── tikTokVideoButtonColumn.dart # 仿Tiktok视频右侧的头像与点赞等按钮列的组件
    ├── tikTokVideoGesture.dart # 仿Tiktok的双击点赞效果
    ├── tikTokVideoPlayer.dart # 视频播放页面，带有控制滑动的VideoListController类
    ├── tiktokTabBar.dart # 仿Tiktok的底部Tabbar组件
    ├── tilTokAppBar.dart # 仿Tiktok的Appbar组件
    ├── topToolRow.dart # 用户页面的顶部状态，在tab切换到user页面时隐藏返回按钮
    └── userMsgRow.dart # 一条用户信息的样式组件


```

```


◆   项目地址


```

https://github.com/mjl0602/flutter_tiktok

2.Flutter 斗鱼 APP
----------------

#### ◆   应用截图

![](https://mmbiz.qpic.cn/mmbiz_png/OKUeiaP72uRz2iaPnnia07Qu4a5oz37yCd0OgKxkFZCLMYevHouMbHyzxM2t33UFZp27qqgGjKKbzQNZ1zK7YPAPw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/OKUeiaP72uRz2iaPnnia07Qu4a5oz37yCd0Pxu6MIOaspibicKkojWOib1Kq3ibxXjoFHibVQqrgfAy32tdOMv0Z2X3HzA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/OKUeiaP72uRz2iaPnnia07Qu4a5oz37yCd0mPmZRISmmu9cfJYaxR4WJicLvXd2wRguTHwa6x0177mFMpTzUmIBiazw/640?wx_fmt=png)

#### ◆   主要涵盖功能

*   滑动状态导航、轮播图
    
*   移动端 px 兼容
    
*   封装 HTTP、IO 缓存操作
    
*   页面路由传值
    
*   bloc 全局状态管理
    
*   礼物横幅动画队列
    
*   弹幕消息滚动
    
*   接入静态视频流
    
*   九宫格抽奖游戏
    
*   照片选择
    
*   webView 容器
    

#### ◆   项目地址

https://github.com/yukilzw/dy_flutter

3.Flutter 豆瓣客户端
---------------

#### ◆   应用截图

![](https://mmbiz.qpic.cn/mmbiz/OKUeiaP72uRz2iaPnnia07Qu4a5oz37yCd085Z4xNs6oCPlhevkTpl40DFa9Eh3cictxMoibRCfibVHhzuPBdZiazwqOg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/OKUeiaP72uRz2iaPnnia07Qu4a5oz37yCd0eeQJ4M3pnpKt96MNFxT4pnbsvo8PBg4ica0McSicnAzL2WEFFygaibGcg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz/OKUeiaP72uRz2iaPnnia07Qu4a5oz37yCd0lg2qFJFfKkqRrsU5IR9YJWqAAP7zs1pkwBV5wRQ6vkvH3LNpM5b3Hw/640?wx_fmt=png)

#### ◆   功能介绍

##### 首页 pages/home

homo_app_bar.dart 首页导航头  
home_page.dart 首页  
my_home_tab_bar.dart 首页 tab

##### 书影音 pages/movie

book_audio_video_page.dart 书影音页面  
detail_page.dart 影片、电视详情页面  
person_detail_page.dart 演员页面介绍

##### 小组 pages/group

##### 市集 shop_page.dart

市集的数据使用两个 webview

##### 我的 page/person

#### ◆   项目地址

https://github.com/kaina404/FlutterDouBan

4.Flutter 开源中国客户端
-----------------

基于 Google Flutter 的开源中国客户端，支持 Android 和 iOS。

#### ◆    应用截图

#### **iOS**

![](https://mmbiz.qpic.cn/mmbiz_png/OKUeiaP72uRz2iaPnnia07Qu4a5oz37yCd0j3ibUyBSV4ExwtNUUK16Iic35mTNkUqxoqJxXR9mfH30VkN0PjRsdxdA/640?wx_fmt=png)

image![](https://mmbiz.qpic.cn/mmbiz_png/OKUeiaP72uRz2iaPnnia07Qu4a5oz37yCd0DdUVAiaWAASB2nye0DVFtkgRnrZkSqZkxuxwYPaVQUcg8NaI5wQb9JQ/640?wx_fmt=png)

image

#### **Android**

![](https://mmbiz.qpic.cn/mmbiz_jpg/OKUeiaP72uRz2iaPnnia07Qu4a5oz37yCd0s8FLjOSpKgYbicnHAEc8bL6ZdgC6RicsrPEMvLTibtZNtW2yws1c3NpYA/640?wx_fmt=jpeg)

image![](https://mmbiz.qpic.cn/mmbiz_jpg/OKUeiaP72uRz2iaPnnia07Qu4a5oz37yCd0u8BQoj6yQnd4NaCicrXNJGcAOmKW5g6rB14ib4LIPAlGJACw6cGTwhow/640?wx_fmt=jpeg)

image![](https://mmbiz.qpic.cn/mmbiz_jpg/OKUeiaP72uRz2iaPnnia07Qu4a5oz37yCd0c7ofGE8qBxExfKYhnQggJY67MzhJ569o7zSANQtmvRCy6KPeQH9w5A/640?wx_fmt=jpeg)

  

#### **◆  功能**

*   登录（使用 osc 账号）
    
*   查看资讯（未登录即可查看）
    
*   查看、回复、发表、评论动弹（需要登录）
    
*   动弹小黑屋（需要登录）
    
*   “发现” 部分的功能基本上都是用 H5 实现
    

#### ◆  **项目地址**

https://github.com/yubo725/flutter-osc

公众号
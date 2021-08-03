> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_43411585/article/details/109798452)

### 目录

*   *   *   *   [一、油猴下载与安装](#_1)
            *   [二、油猴脚本免费使用网站](#_5)
            *   [三、油猴脚本编写介绍](#_10)
            *   *   [1、脚本注释解释](#1_13)
                *   [2、编写脚本](#2_31)
                *   [3、调试油猴脚本](#3_35)
            *   [四、油猴 hook 之 js 逆向案例](#hookjs_39)
            *   *   [1、hook 之 window 属性，定位 pre 参数加密位置](#1hookwindowpre_41)
                *   [2、hook 之 cookie 生成](#2hookcookie_82)

#### 一、油猴下载与安装

*   油猴：Tampermonkey 是一款免费的浏览器扩展和最为流行的用户脚本管理器， 可以通过不同的 “脚本” 实现数十甚至上百个功能，比如视频去水印，去广告，js 逆向定位加密参数生成位置等
*   下载与安装：[下载. crx 插件](https://www.chrome666.com/chrome-extension/tampermonkey.html) ，安装：谷歌 > 右上角 > 更多工具 > 扩展程序，直接将 crx 文件拖入如下图页面  
    ![](https://img-blog.csdnimg.cn/20201119072614274.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzQxMTU4NQ==,size_16,color_FFFFFF,t_70#pic_center)

#### 二、油猴脚本免费使用网站

*   [Userscript.Zone Search](https://www.userscript.zone/?utm_source=tm.net&utm_medium=scripts&utm_campaign=0)：是一个新网站，允许通过输入合适的 URL 或域来搜索用户脚本
*   [Greasy Fork](https://greasyfork.org/zh-CN)：最受欢迎的后起之秀，提供用户脚本的网站，可实现去掉视频播放广告，去水印等多种功能，可以直接安装使用，储存库中有大量的脚本资源
*   [OpenUserJS](https://openuserjs.org/)：继 GreasyFork 之后开始创办，在其储存库中也拥有大量的脚本资源
*   [serscripts Mirror](https://userscripts-mirror.org/)

#### 三、油猴脚本编写介绍

*   添加新脚本 > 即为如下的编辑脚本页面  
    ![](https://img-blog.csdnimg.cn/20201120074657320.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzQxMTU4NQ==,size_16,color_FFFFFF,t_70#pic_center)

##### 1、脚本注释解释

*   `@match`这个注释的内容最为重要，它决定了你的脚本应用到哪个具体的网页，还是应用到所有的网页
*   `@run-at`确定了脚本的注入时机，在 js 逆向中也很重要
*   [油猴脚本所有注释介绍详情](https://www.tampermonkey.net/documentation.php)

```
油猴脚本的名字
```

##### 2、编写脚本

*   编写到`// Your code here`那里即可  
    ![](https://img-blog.csdnimg.cn/20201120080550689.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzQxMTU4NQ==,size_16,color_FFFFFF,t_70#pic_center)

##### 3、调试油猴脚本

*   利用浏览器的调试功能，在脚本需要调试的地方添加`debugger;`语句，在相应网页刷新后，即可利用 F12 开发者工具进行单步调试、监视变量等操作了  
    ![](https://img-blog.csdnimg.cn/20201120080307891.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzQxMTU4NQ==,size_16,color_FFFFFF,t_70#pic_center)

#### 四、油猴 hook 之 js 逆向案例

*   可用于：定位 window 属性加密参数，定位 cookie 生成位置

##### 1、hook 之 window 属性，定位 pre 参数加密位置

*   [目标网站](https://flight.qunar.com/)，目标定位加密参数 pre：  
    ![](https://img-blog.csdnimg.cn/20201119081356760.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzQxMTU4NQ==,size_16,color_FFFFFF,t_70#pic_center)
*   全局搜索’pre’参数，发现`pre的值就是window._pt_的值`，但是直接调试需要耗一些时间才能定位到加密位置，通过油猴 hook 可以轻松定位到加密位置  
    ![](https://img-blog.csdnimg.cn/20201119081807728.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzQxMTU4NQ==,size_16,color_FFFFFF,t_70#pic_center)
*   首先，添加油猴脚本如下，保存该脚本并启用，然后切换到目标网页：
    *   注意 hook 的域名为`@match https://flight.qunar.com/*，hook的注入时机为文档开始时@run-at document-start`；
    *   [Object.defineProperty()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)：直接在一个对象上定义或修改属性，并返回此对象；
    *   而该脚本的 hook 点就是_pt_属性的 get/set 方法，`在set处即_pt_属性被赋值时，debugger住`

```
// ==UserScript==
// @name         定位pre加密参数
// @namespace    http://tampermonkey.net/
// @version      0.1
// @description  try to take over the world!
// @author       Shirmay1
// @run-at       document-start
// @match        https://flight.qunar.com/*
// @grant        none
// ==/UserScript==

(function() {
    'use strict';
    var pre = window._pt_
    Object.defineProperty(window, '_pt_', {
        get: function() {
            console.log('Getting window._pt_');
            return pre
        },
        set: function(val) {
            console.log('Setting window._pt_', val);
            debugger ;pre = val;
        }
    })
}
)();
```

*   接着，F12 打开谷歌开发者工具，然后点击目标网站搜索按钮，油猴脚本随之执行，即可轻松定位到该加密参数位置，如下图  
    ![](https://img-blog.csdnimg.cn/20201119082145422.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzQxMTU4NQ==,size_16,color_FFFFFF,t_70#pic_center)
*   最后，通过点击右侧调用栈回溯前一个函数，即可精准定位到`window._pt_的值也就是pre的值`  
    ![](https://img-blog.csdnimg.cn/20201119082426469.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzQxMTU4NQ==,size_16,color_FFFFFF,t_70#pic_center)

##### 2、hook 之 cookie 生成

*   [目标网站](https://hotels.ctrip.com/hotel/shanghai2/star5#ctm_ref=ctr_hp_sb_lst)，定位 _bfa 这个 cookie 参数生成位置  
    ![](https://img-blog.csdnimg.cn/20201124081655270.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzQxMTU4NQ==,size_16,color_FFFFFF,t_70#pic_center)
*   油猴脚本如下：
    *   test() 检查字符串是否与给出的正则表达式模式相匹配，如果是则返回 true，否则就返回 false，该脚本匹配_bfa 的相关字符串

```
// ==UserScript==
// @name         定位携程cookie
// @namespace    http://tampermonkey.net/
// @version      0.1
// @description  try to take over the world!
// @author       Shirmay1
// @match        https://hotels.ctrip.com/*
// @run-at       document-start
// @grant        none
// ==/UserScript==

(function() {
    'use strict';
    // hook 携程cookie
    var _cookie = "";
    Object.defineProperty(document, 'cookie', {
        set: function(val) {
            console.log('cookie set->', val);
            var pi = new RegExp('^_bfa.*');
            if (pi.test(val)) {
                debugger;
            }
            _cookie = val;
            return val;
        },
        get: function(val) {
            return _cookie;
        }
   });
})();
```
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/github_35631540/article/details/102638553)

**近期博主在 GitChat 上举办了一场 彻底玩转 Tampermonkey 的 Chat 欢迎各位前来捧场. 报名地址**

**[使用 Tampermonkey 编写高级跨网站自动化任务脚本](https://gitbook.cn/gitchat/activity/5db5537f81d36847bd242fda)**

标明：本文出现的 TM 即使 Tampermonkey 的缩写

**目录**

[USERSCRIPT HEADER](#t0)

[@name](#t1)

[@namespace](#t2)

[@version](#t3)

[@author](#t4)

[@description](#t5)

[@homepage, @homepageURL, @website and @source](#t6)

[@icon, @iconURL and @defaulticon](#t7)

[@icon64 and @icon64URL](#t8)

[@updateURL](#t9)

[@downloadURL](#t10)

[@supportURL](#t11)

[@include](#t12)

[@match](#t13)

[@exclude](#t14)

[@require](#t15)

[@resource](#t16)

[@connect](#t17)

[@run-at](#t18)

[@grant](#t19)

[@noframes](#t20)

[@unwrap](#t21)

[@nocompat](#t22)

[应用程序接口（高级 API）](#t24)

[unsafeWindow](#t25)

[Subresource Integrity(子资源完整性)](#t26)

[GM_addStyle(css)](#t27)

[GM_deleteValue(name)](#t28)

[GM_listValues()](#t29)

[GM_addValueChangeListener(name, function(name, old_value, new_value, remote) {})](#t30)

[GM_removeValueChangeListener(listener_id)](#t31)

[GM_setValue(name, value)](#t32)

[GM_getValue(name, defaultValue)](#t33)

[GM_log(message)](#t34)

[GM_getResourceText(name)](#t35)

[GM_getResourceURL(name)](#t36)

[GM_registerMenuCommand(name, fn, accessKey)](#t37)

[GM_unregisterMenuCommand(menuCmdId)](#t38)

[GM_openInTab(url, options), GM_openInTab(url, loadInBackground)](#t39)

[GM_xmlhttpRequest(details)](#t40)

[GM_download(details), GM_download(url, name)](#t41)

[GM_getTab(callback)](#t42)

[GM_saveTab(tab)](#t43)

[GM_getTabs(callback)](#t44)

[GM_notification(details, ondone), GM_notification(text, title, image, onclick)](#t45)

[GM_setClipboard(data, info)](#t46)

[GM_info](#t47)

[<>](#t48)

USERSCRIPT HEADER
-----------------

### @name

脚本的名字

### @namespace

脚本的命名空间

### @version

脚本的版本，用于检查更新。

### @author

脚本的作者

### @description

简短重要的描述

### @homepage, @homepageURL, @website and @source

在 “选项” 页上用于从脚本名链接到给定页的作者主页。请注意，如果 @namespace 标记以 “http://” 开头，则其内容也将用于此操作。

### @icon, @iconURL and @defaulticon

低分率的脚本会在脚本管理列表上显示

### @icon64 and @icon64URL

脚本 icon  64*64 如果给了这个标签，但给了图标，则图标图像将在选项页的某些位置缩放

### @updateURL

更新脚本的地址，注意：只有存在 @version 标签才会去更新

### @downloadURL

定义检测到更新时将从中下载脚本的 URL。如果值为 none，则不会执行更新检查。

### @supportURL

定义使用者报告 issues 和个人支持的地址

### @include

脚本应该运行的页面， 可以使用正则匹配。 允许多个标签

请注意 @include 不支持 url hash 参数，可以访问这里获取更多的信息[点击获取更多信息](https://forum.tampermonkey.net/viewtopic.php?p=3094#p3094)

示例

```
// @include http://www.tampermonkey.net/*
// @include http://*
// @include https://*
// @include *
```

### @match

和 @include 标签差不多的意思， 你可以点击这里获取更多信息

注意： 尚不支持 “<all-urls>” 语句，scheme 部分也接受“http*：//”。

允许多个标记实例。

### @exclude

排除 URL，即使它们包含在 @include 或 @match 中 。 允许多个标签

### @require

指向一个脚本文件，会在本脚本运行前加载并执行

注意：通过 @require 加载的脚本及其 “use strict” 语句可能会影响用户脚本的 strict 模式！

示例：

```
// @require https://code.jquery.com/jquery-2.1.4.min.js
// @require https://code.jquery.com/jquery-2.1.3.min.js#sha256=23456...
// @require https://code.jquery.com/jquery-2.1.2.min.js#md5=34567...,sha256=6789...
```

 有关如何确保完整性的详细信息，请查看子资源完整性部分。允许多个标记实例。

### @resource

预加载可以通过脚本通过 gm_getresourceurl 和 gm_getresourcetext 访问的资源

**示例**

```
// @resource icon1 http://www.tampermonkey.net/favicon.ico
// @resource icon2 /images/icon.png
// @resource html http://www.tampermonkey.net/index.html
// @resource xml http://www.tampermonkey.net/crx/tampermonkey.xml
// @resource SRIsecured1 http://www.tampermonkey.net/favicon.ico#md5=123434...
// @resource SRIsecured2 // http://www.tampermonkey.net/favicon.ico#md5=123434...;sha256=234234...
```

有关如何确保完整性的详细信息，请查看 “子资源完整性” 部分。允许多个标记实例。

### @connect

此标记定义域（没有顶级域），包括允许由 [GM_xmlhttpRequest](http://www.tampermonkey.net/documentation.php?ext=dhdg#GM_xmlhttpRequest) 检索的子域

**示例**

```
// @connect <value>
```

<value> 可以是以下几个值

*   域可以是: **_tampermokey.net_** (可以允许子域名)
*   子域名如: **_safari.tampermokey.net_**
*   _**self** :_ 列出脚本当前运行的域
*   **_localhost_** 有权限访问 localhost
*   _**1.2.3.4** _ 链接到 IP 地址
*   *

如果无法声明用户脚本可能连接到的所有域，则最好执行以下操作：

**声明所有已知或至少所有可能由脚本连接的公共域**。这样，大多数用户都可以避免确认对话框。

另外在脚本中添加 “**@connect***”。通过这样做，tampermonkey 仍然会询问用户是否允许下一个连接到未提及的域，但也会提供一个 “**总是允许所有域**” 按钮。如果用户单击此按钮，则将自动允许所有未来的请求。

用户还可以通过在 “脚本设置” 选项卡的用户域白名单中添加 “*” 来白名单所有请求。

注意：

初始 url 和最终的 url 都会被检查，  为了向后兼容 scriptish@domain 标记也会被解释。

允许多个标记实例。

### @run-at

定义脚本被注入的时间，与其他脚本处理相反，**@run-at** 定义了脚本要运行的第一可能时间。

这意味着，使用 **@require** 标记的脚本可能会在文档已加载后执行，因为获取所需脚本花费了很长时间。无论如何，在给定的注入时刻之后发生的所有 domnodeinserted 和 domcontentloaded 事件都将被缓存，并在注入时传递给脚本。

**全部示例:**

```
// @run-at document-start   // 脚本会被尽可能快地注入
// @run-at document-body    // 当body元素存在是被注入
// @run-at document-end     // 当DOMContentLoaded事件被触发时或者之后注入
// @run-at document-idle    // 当DOMContentLoaded事件被触发后被注入 如果没有@run-at标签也是在此时注入
// @run-at context-menu     // 当点击浏览器上下文菜单时被注入（仅仅是桌面Chrome-based浏览器）
// 注意：如果使用了context-menu @include和@exclude的变量都将被忽略，但是未来可能会改变
```

### @grant

**@grant** 被用于设置 GM_* 函数的白名单， GM_*function 是一些 unsafeWindow 对象和一些有影响的 window 函数，如果没有 @grant 标签，TM 会猜测脚本需要什么

示例：

```
// @grant GM_setValue
// @grant GM_getValue
// @grant GM_setClipboard
// @grant unsafeWindow
// @grant window.close
// @grant window.focus
```

由于关闭和聚焦选项卡是一个强大的功能，因此还需要将其添加到 @grant 语句中。

如果 **@grant** 后跟 “none”，沙盒将被禁用，脚本将直接在页面上下文中运行。在此模式下，没有 gm_u * 函数，但 gm_u info 属性将可用。

**示例**

```
// @grant none
```

### @noframes

这个标签表明脚本在主页面上运行，而不是在 iframes 里

### @unwrap

这个标签是被忽略的，因为他在谷歌浏览器里不需要

### @nocompat

目前，tm 试图通过查找 @match 标记来检测脚本是否是在 google chrome/chromium 的知识中编写的，但并不是每个脚本都使用它。这就是为什么 tm 支持这个标签来禁用运行为 firefox/greasemonkey 编写的脚本所需的所有优化。要保持此标记可扩展，可以添加可由脚本处理的浏览器名称。

示例

```
// @nocompat Chrome
```

应用程序接口（高级 API）
--------------

### unsafeWindow

unsafeWindow 对象提供权限访问页面的 js 函数和变量

### Subresource Integrity(子资源完整性)

可以使用 **@resource** 和 **@require** 标记的 url 的散列组件来实现此目的。

示例：

```
// @resource SRIsecured1 http://www.tampermonkey.net/favicon1.ico#md5=ad34bb...
// @resource SRIsecured2 http://www.tampermonkey.net/favicon2.ico#md5=ac3434...,sha256=23fd34...
// @require https://code.jquery.com/jquery-2.1.1.min.js#md5=45eef...
// @require https://code.jquery.com/jquery-2.1.2.min.js#md5=ac56d...,sha256=6e789...
```

TM 本机支持 MD5 哈希作为回退，所有其他（SHA-1、SHA-256、SHA-384 和 SHA-512）都依赖于 window.crypto。如果给定了多个散列（用逗号或分号分隔），则 TM 将使用当前支持的最后一个散列。如果外部资源的内容与所选哈希不匹配，则资源不会传递到用户脚本。所有散列都需要以十六进制或 base64 格式编码。

### GM_addStyle(css)

给 document 添加样式，并且返回注入的节点

### GM_deleteValue(name)

删除‘name’ 从 storage 里

### GM_listValues()

列出 storage 中的所有 name

### GM_addValueChangeListener(name, function(name, old_value, new_value, remote) {})

在 storage 里添加一个改变事件的监听，并返回监听 id

‘name’是被观察的变量

回调函数的‘remote’变量是显示此值是从另一个选项卡的实例修改的（true）还是在此脚本实例中修改的（false）。

因此，不同浏览器选项卡的脚本可以使用此功能相互通信。

可以使用此 API 实现不同浏览器 Tab 的相互通讯

### GM_removeValueChangeListener(listener_id)

通过监听器的 id 移除一个监听改变的事件

### GM_setValue(name, value)

设置‘name‘ 的值到 storage 中

### GM_getValue(name, defaultValue)

从 storage 中获取‘name’的值

### GM_log(message)

在控制台打印日志

### GM_getResourceText(name)

获取在脚本顶部预定的 @resource 标签的内容

### GM_getResourceURL(name)

获取在脚本顶部定义的 @resource 标签的 base64 encodeURI 

### GM_registerMenuCommand(name, fn, accessKey)

注册一个能在页面上能够显示 TM 菜单命令，当这个脚本执行是，并且返回菜单命令 id

意思就是可以注册一个直接显示 TM 的菜单的 ming

### GM_unregisterMenuCommand(menuCmdId)

取消注册一个菜单命令根据菜单命令 ID(通过 GM_registerMenuCommand 提供的)

### GM_openInTab(url, options), GM_openInTab(url, loadInBackground)

使用参数 url 打开一个新的 tab，options 可以是以下值

*   **active** 决定新的 tab 是否被聚焦，聚焦的意思是直接显示
*   **insert** 插入一个新的 tab 在当前的 tab 后面
*   **setParent** 在 tab 关闭后重新聚焦当前 tab

另外，新的选项卡将被添加。loadinbackground 具有与 active 相反的含义，并被添加以实现 Greasemonkey 3.x 兼容性。如果未指定 “活动” 或“加载后台”，则选项卡将不会聚焦。此函数返回一个具有函数 close、侦听器 onclosed 和一个名为 closed 的标志的对象。

### GM_xmlhttpRequest(details)

创建一个 xmlHttpRequest.  
参数 **_details_** 的属性有:

*   **method** 可以是 GET, HEAD, POST 其中一种
*   **url** 请求的 url
*   **headers** 如： user-agent, referer, ... (一些特殊的 headers 不被支持在 Safari and Android 浏览器里)
*   **data**  一些字符串有 post 请求发送过去
*   **binary**   说过 binary 模式，类型发送数据
*   **timeout** 超时时间
*   **context**  被添加到 response 对象上的对象
*   **responseType** 可以是 arraybuffer, blob, json
*   **overrideMimeType**  请求的 MIME type
*   **anonymous** 在请求中不需要发送 cookies，详细请看 fetch 注释
*   **fetch** (beta) 使用一个 fetch 来代替 xhr 请求， 在 chrome 中，这会导致 xhr.abort、details.timeout 和 xhr.onprogress 不工作，并使 xhr.onreadystatechange 仅接收 readystate 4 事件
*   **username** 授权的用户名
*   **password** 授权的用户密码
*   **onabort**  请求中断时执行的回调函数
*   **onerror** 请求以错误结束时需要执行的回调函数
*   **onloadstart**  请求开始加载时执行的回调函数
*   **onprogress** 请求状态变化时执行的回调函数
*   **onreadystatechange**  请求的准备状态改变是执行的回调函数 
*   **ontimeout**  超时后执行的回调函数
*   **onload**  当请求被返回时执行的回调函数 ，他的几个参数如下
    *   **finalUrl** - the final URL after all redirects from where the data was loaded
    *   **readyState** - the ready state
    *   **status** - the request status
    *   **statusText** - the request status text
    *   **responseHeaders** - the request response headers
    *   **response** - the response data as object if _details.responseType_ was set
    *   **responseXML** - the response data as XML document
    *   **responseText** - the response data as plain string

返回的对象包含以下属性

*   **abort** - 取消请求的函数

  
**注意:** 属性 **synchronous** 不支持  
**Important:** 如果你想使用这个方法请移步 **@connectb** 标签 查看更多信息 

### GM_download(details), GM_download(url, name)

使用下载资源到本地磁盘  
_details_ 的属性：

*   **url** - 资源的 url
*   **name** - 文件名，出于安全原因，文件的扩展名必须在 TM 参数页面的的白名单里 
*   **headers** - 如 GM_xmlhttpRequest 一样设置请求头部
*   **saveAs** - boolean 值, 显示一个保存的弹窗
*   **onerror**  下载以失败结束执行的回调函数
*   **onload**  现在完成后执行的回调函数
*   **onprogress**   下载过程中变化的回调函数 
*   **ontimeout**   下载超时执行的回调函数   
     

现在文件中 **_onerror_** 的参数如下：

*   **error** - 错误原因
    *   not_enabled - 用户未启用下载功能
    *   not_whitelisted - 下载的文件类型不在白名单里
    *   not_permitted - 用户开启了下载权限，但没 downloads 权限
    *   not_supported - 下载属性不支持，由于浏览器或者版本原因
    *   not_succeeded - 下载没有开始或者失败了。 details 可以提供更多的信息
*   **details** - 错误的详细情况  
     

返回一个对象包含以下属性

*   **abort** - 取消下载的函数

根据下载模式，gm_u info 提供一个名为 download mode 的属性，该属性设置为以下值之一：native、disabled 或 browser。

### GM_getTab(callback)

获取一个持久对象，只要该选项卡处于打开状态.

### GM_saveTab(tab)

保存 tab 对象为了重新打开，在页面关闭后

### GM_getTabs(callback)

获取所有 tab 对象作为散列与其他脚本实例通信。

### GM_notification(details, ondone), GM_notification(text, title, image, onclick)

显示一个 H5 的桌面通知，或者高亮当前 tab

  
_details_ 的属性：

*   **text** - 通知的问题 如果高亮就 就不需要
*   **title** - 通知的标题
*   **image** - 图片
*   **highlight** - 一个 boolean 标志，是否高亮 tab 
*   **silent** -  一个 boolean 是否播放音乐 
*   **timeout** - 通知显示的时间  0 表示 一直显示
*   **ondone** - 通知被关闭时 无论是被点击还是超时 执行的函数 
*   **onclick** - 点击通知触发的函数

所有参数的作用与其对应的详细信息属性挂件完全相同。

### GM_setClipboard(data, info)

复制数据到粘贴板，参数 info 可以是对象如 {type: 'text', mimetype: 'text/plain'}，或者是一个字符串 text， html

### GM_info

获取一些脚本和 TM 的信息，数据结构如下

```
Object+
---> script: Object+
------> author: ""
------>copyright: "2012+, You"
------>description: "enter something useful"
------>excludes: Array[0]
------>homepage: null
------>icon: null
------>icon64: null
------>includes: Array[2]
------>lastUpdated: 1338465932430
------>matches: Array[2]
------>downloadMode: 'browser'
------>name: "Local File Test"
------>namespace: "http://use.i.E.your.homepage/"
------>options: Object+
--------->awareOfChrome: true
--------->compat_arrayleft: false
--------->compat_foreach: false
--------->compat_forvarin: false
--------->compat_metadata: false
--------->compat_prototypes: false
--------->compat_uW_gmonkey: false
--------->noframes: false
--------->override: Object+
------------>excludes: false
------------>includes: false
------------>orig_excludes: Array[0]
------------>orig_includes: Array[2]
------------>use_excludes: Array[0]
------------>use_includes: Array[0]
--------->run_at: "document-end"
------>position: 1
------>resources: Array[0]
------>run-at: "document-end"
------>system: false
------>unwrap: false
------>version: "0.1"
---> scriptMetaStr: undefined
---> scriptSource: "// ==UserScript==\n// @name       Local File Test\n ...."
---> scriptUpdateURL: undefined
---> scriptWillUpdate: false
---> scriptHandler: "Tampermonkey"
---> isIncognito: false
---> version: "4.0.25"
```

### <><![CDATA[your_text_here]]></>

tampermonkey 支持这种存储元数据的方式。TM 尝试自动检测脚本是否需要启用此兼容性选项。
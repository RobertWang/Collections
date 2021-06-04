> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/336177304)

由于原生小程序的开发过程过于上古，目前前端很多工具都用不了或者很难用，所以使用 webview 来实现小程序是很好的办法。

> 注意 webview 小程序只有企业认证的号才可以使用。个人独立开发者不行。

### 设计思路

1.  小程序加载时，判断 storage 里是否有 cookie，如果有表示用户已注册，则直接带 cookie 参数跳转到 webview，webview(h5) 里判断 url 上是否有 cookie 参数，如果有则请求用户信息、初始化数据等操作，同时将 cookie 存到 document.cookie 中。  
    
2.  如果 storage 里没有 cookie，表示用户未注册，则跳到注册 / 登录页面获取 cookie，获取成功后将 cookie 存入小程序 storage 中，同时带 cookie 参数跳到 webview。  
    
3.  token 过期处理：在第 1 步中，跳转 webview 前，先带 cookie 去访问一次后台接口，如果没有 401 等异常，表示 token 没有过期，可以跳转 webview，如果过期则跳到登陆页面重新获取新的 cookie 再跳转 webview。或者 webview 中拦截到 401 利用 jsdk 跳小程序登录页重新获取 token。  
    
4.  webview 中需要利用小程序能力：例如 webview 付费想使用微信支付，需要在小程序中新建一个 pay 页面，webview 中利用 jsdk 的 wx.navigateTo 跳到这个 pay 页面，同时将需要的参数通过 url 参数传给小程序，小程序获取 webview 传来的支付参数进行微信支付。  
    

> 注意微信支付需要在后台申请开通才可以使用。

### webview 调试方法

小程序中使用 storage 保存的信息可以通过删除小程序来清空，也可以打开小程序自带的调试模式来开启控制台清除。但是 webview 里 localStorage 保存的信息是无法清除的！  
这时候就需要利用 H5 虚拟控制台调试了，我用的是：

[Eruda​github.com](https://link.zhihu.com/?target=https%3A//github.com/liriliri/eruda)

我对它进行了封装，增加了一个 url 参数来开启和关闭调试：

[debug.min.js​github.com](https://link.zhihu.com/?target=https%3A//github.com/Saber2pr/test/blob/master/tools/debug.min.js)

将它添加到项目中，访问项目时，url 参数加上 debug=true 就可以启用调试！

**在 webview 中使用**  
webview 中不能直接放个按钮来做调试开关，但可以利用页面上已有的 input，例如 input 中输入特定的字符串后执行:  
location.href = '/?debug=true'  
来间接开启调试！

### 流程图

![](https://pic1.zhimg.com/v2-e502fd9914ab0a574c8bf66813579ccc_r.jpg)
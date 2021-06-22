> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIzMzMzOTI3Nw==&mid=2247501954&idx=1&sn=8d6ccd9a607a682979764bdaf8faf599&chksm=e885a860dff22176204af85c540b02b3724de778c53016cacb34bef264470a386e3e709f1a11&mpshare=1&scene=1&srcid=0621HRqg19vJa4VtRhFjZeBH&sharer_sharetime=1624241493362&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

1. 前言
-----

使用 Python 写完爬虫后，有时候我们需要在手机上实时对爬虫进行调度，或实时展示爬虫的结果

面对这种场景，我们可以将爬虫逻辑写成 API 部署到服务器，然后在移动端编写 App，通过界面元素控件直接调用接口即可

本篇文章，将和大家聊聊如何快速编写一款 iOS 原生 App

2. 准备
-----

要实现原生 iOS 应用，我们需要在 Mac 上使用 Xcode 编写并进行编译

首先，设置 Xcode 的开发者账号

打开 Xcode，左上角选择 Xcode - Preferences - Accounts，点击左下角的 + 号，添加一个开发者账号

![](https://mmbiz.qpic.cn/mmbiz_png/atOH362BoyvmQ1EdGbRcZicxafLLibY4H3g94Ueu3Kic81Jq3EJ7rl6wjTp0y50cuvjwvrcpPfJw10soI0hK7ntCA/640?wx_fmt=png)

然后，使用 Xcode 创建一个项目

这里模版选择 iOS App，输入项目名称，编程语言选择「 Swift 」，点击下一步完成项目的创建

![](https://mmbiz.qpic.cn/mmbiz_png/atOH362BoyvmQ1EdGbRcZicxafLLibY4H3g94Ueu3Kic81Jq3EJ7rl6wjTp0y50cuvjwvrcpPfJw10soI0hK7ntCA/640?wx_fmt=png)

PS：Swift 相比 OC，语法更加简洁明了

最后，为新创建的项目指定 Sign 签名

这部分如果有疑惑，可以点击文末的阅读原文去了解  

3. 实战
-----

实战部分，我们以一个简单的登陆页面来进行讲解

3-1  安装依赖库 

由于项目使用 Swift 开发，这里推荐使用 SPM（ Swift Package Manager ）来安装依赖

比如，网络请求库「 Alamofire 」

项目地址：https://github.com/Alamofire/Alamofire

安装方式：File - Swift Packages - Add Package Dependency - 输入项目地址（ Github / Gitee ）- 选择安装版本

![](https://mmbiz.qpic.cn/mmbiz_png/atOH362BoyvmQ1EdGbRcZicxafLLibY4H3MLMUTYuJo7lbjBUbY6UibZZWJjHgaOiaZ7KicibXtsZK8sTcE1LnsZiaOTg/640?wx_fmt=png)

3-2  页面布局

打开项目根目录下的「 ContentView.swift 」文件，在 body 下编写具体的视图

首先，使用 VStack 定义一个垂直的布局盒子，并定义子控件水平居中展示

PS：SwiftUI 常见的 3 种布局方式为 VStack、HStack、ZStack，它们分别代表垂直布局、水平布局、深度布局

```
import SwiftUI
import Combine

struct ContentView: View {
    
    ...
    
    //构建页面View
    var body: some View {
        VStack(alignment: HorizontalAlignment.center){
           ...
        }
    } 
}


```

然后，子元素依次添加一张本地图片、两个输入框、一个选择框、一个按钮

其中，

*   图片控件 Image
    
*   文本输入框控件 TextField
    
*   选择框控件 Toggle
    
*   按钮控件 Button
    

```
import SwiftUI
import Combine

struct ContentView: View {
    
    //构建页面View
    var body: some View {
        VStack(alignment: HorizontalAlignment.center){
            Image("WechatIMG5")
            
            TextField("用户名", text: $username).textFieldStyle(RoundedBorderTextFieldStyle())
                .keyboardType(.numberPad)
                .padding()
            
            TextField("密码", text: $pwd).textFieldStyle(RoundedBorderTextFieldStyle())
                .keyboardType(.numberPad)
                .padding()
            
            //是否为测试
            Toggle(isOn: $isFavorited) {
                Text("测试环境")
                }.padding(.leading, 0.0).frame(width: 140, height: 40)

            Button(action: {
                //具体的操作     
    }   
    struct ContentView_Previews: PreviewProvider {
        static var previews: some View {
            ContentView()
        }
    }   
    
}


```

最后，定义变量和控件数据进行双向绑定

```
struct ContentView: View {
    
    @State  var username:String = "用户名"
    @State  var pwd:String = "密码"
    @State  var result:String = "结果"
    @State  var isFavorited:Bool = false
        
}


```

3-3  网络请求及结果展示  

为 Button 控件设置点击事件，使用 Alamofire 进行网络请求，最后将结果展示写入到结果控件绑定到数据中去即可

```
Button(action: {
                //具体的操作
                print("start")
                
                var url = ""
                
                if(self.isFavorited){
                   url = "...?user + self.pwd
                }else{
                   url = "...?user + self.pwd
                }
                
                print("请求地址:"+url)
                
                AF.request(url).responseJSON { response in
                    switch response.result {
                    case .success(let json):
                        //转为Dictionary
                        let post_paramsValue = json as! Dictionary<String,Any>
                        
                        //__NSCFString
                        let msg = post_paramsValue["msg"]!
                        //转为String
                        let msg_pro = msg as! String
                        
                        self.result = msg_pro
                        
                        break
                    case .failure(let error):
                        print("error:\(error)")
                        self.result = "网络异常，请稍后再试！"
                        break
                    }
                }
                
            }) {
                Text("一键执行")
                    .foregroundColor(Color.white)
                    .padding(10)
                    .background(Color.gray)
                    .cornerRadius(6)
                    .padding(10)
                    .frame(alignment: .center)
            }
            
            TextField("结果区域", text: $result)
                .padding()
        }

```

4. 最后
-----

文章通过一个简单的例子描述了开发一个 iOS 原生应用的详细步骤；实际应用中，可以结合具体的场景去定制开发不同的功能模块
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxNDEwNjk5OQ==&mid=2650403929&idx=1&sn=fae15630e397513d5ad92992e93be8c2&chksm=83953c41b4e2b557ddf8c5db258c69db52648d32347d914923d5b91d81c6c09f1c714b8ba940&scene=21#wechat_redirect)

作者 | 姜沂 (倾寒) 

出品 | 阿里巴巴新零售淘系技术部

导读：自 2014 年苹果发布会发布 Swift 之后,  Swift 经过多年迭代，终于达到了 ABI 稳定版本，也意味着 Swift 做为稳定的得语言，值得用在大型 APP， 用来生产环境中。

  
2019 年 WWDC , 又发布了引起无数 Apple 平台开发者欢呼的框架 SwiftUI， 据非官方消息，SwiftUI 框架孵化于 4 年前，作为苹果全平台的 UI 系统的未来，数十名核心开发者，不准向其他同事和外部披露任何关于此项目的任何信息，于今年释出 Beta 版本后，从方方面面都透出出这是目前最强的移动端声明式 编程框架，没有之一（个人觉得)。在此实战之前作者已经编写了两篇相关的文章。

1、[新晋网红 SwiftUI——淘宝带你初体验](http://mp.weixin.qq.com/s?__biz=MzAxNDEwNjk5OQ==&mid=2650402703&idx=1&sn=0942415a7c92ab2465a9acb58b15b808&chksm=83952797b4e2ae812bd6159b49c069104e8db54db40435cb3343e06ff96f043390d8057cd8d6&scene=21#wechat_redirect)         ☜（点击阅读）

2、[系列文章深度解读 | SwiftUI 背后那些事儿](http://mp.weixin.qq.com/s?__biz=MzAxNDEwNjk5OQ==&mid=2650402891&idx=1&sn=804f271d6794a0ec7b5eeaea585f5e8e&chksm=83953853b4e2b14563dd6cefb1362a457b96fb591c64bd6c94cb38d0595d2233daa96ba1b876&scene=21#wechat_redirect)   ☜（点击阅读）

注： 项目代号为企业内部私有，这里使用 **SOT** 代指，意为 “Swift on Taobao”。

背景

为了研究 SwiftUI 在业务落地的可能性，我们一直持续关注着 SwiftUI 的发展，但编程这种工作，向来是阅读千编，不如实战一次来的深刻，刚好我们有一个业务场景非常适合，那就是观察稳定性大盘。

  
整个淘系也有一个用来观察稳定性数据的应用，通常来说数据大盘是比较适合在 PC 浏览器中展示的，我们也在 PC 中使用了多年，但是淘宝 APP 是一个重运营类的 APP， 经常会有一些活动在节假日投放。

  
但此时值班人员或者相关人员可能在外，有时候可能并未携带电脑，这时候观察稳定性情况就非常窘迫，我们迫切需要一款可以随身携带的 APP，用于在紧急时刻观察稳定性问题。

项目耗时

这里先给出时间结论，

整个 SOT APP 耗时 1.3 人力，共 10 个工作日，整个 Swift 代码 约 2800 行。  

由于这是一款必须工作在内网下的 APP， 接入内网鉴权没有太多经验，花费不少时间。

  
整体下来大约有 5 天左右的工作量花在调试接口，内网鉴权，原型设计部分，真正花在 SwiftUI 的部分约有 5 天，不得不说效率惊人。

项目设计

### 原型设计

做一款 APP 的最核心的部分是设计 APP 的功能，熟悉 SOT 的同学，应该知道一般观察稳定性主要是观察数据大盘，聚合列表，分析聚合详情，崩溃分析等比较重要的模块。

落地 SwiftUI 的计划预计 两周，所以 SOT 一期只做做核心常用的部分。功能有了，那么设计怎么办呢？

  
不要怂，作为 9102 年的程序员，不会做 UI 怎么可以？由于 Mac 平台的 设计软件 如 Keynote 和 Sketch 操作方式，基本和 StoryBoard （只会用代码写 UI 的同学要回去重新学习下 StoryBoard 了 -）操作非常接近，花了一天时间简单设计了下界面。

这里刻意模仿 App Store 的圆角和阴影设计，至于为什么？原因就是负责的设计会让 UI 代码编写变的更有挑战性，如果只是用系统原生的样式，那么碰见的难题就会大大减少，这样的实战到了实际的项目中，碰见的问题还会很多。

  
事实证明负责的 UI 设计对理解 SwiftUI 非常有价值，单单一个圆角，就花去了 6 个小时开发时间。

![](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuicy3aSjJrwLfmhWnx7CPQuWvHO6Xd8picvh4klkUfx6m4nCeR5M4nqVz7beb7TmIKweIfYepLCDwuw/640?wx_fmt=png)

### 数据流管理

SwiftUI 是一个典型的单向数据流得声明式 UI 编程框架, 在 SwiftUI 中 View 只是一个页面的描述部分，SwiftUI 提供了多个数据流管理对象。

  
`@State` `@Binding` `@Obserabled` , 通过改变这些数据流的值，SwiftUI 系统可以理解重新构建 View Tree， 并根据内部变化的范围，有一层类似 Virtual Dom 的 ViewTree, 由于 View 都是结构体，SwiftUI 每次构建这个 View Tree 都极快，这使得性能有很强的保障。

  
在实践中也发现了一些 Bug，但由于目前 SwiftUI 还在高速变化，这些 Bug 都会在将来的版本中修复，这里就不过多解释了。

### State

State 是 SwiftUI 中最常用的 代理属性，通过对代理属性的修改，SwiftUI 内部会自动的重新计算 View 的 Body 部分，构建 出 View Tree。

  
注意 **State 只能在当前 View 的 body 体里面修改**，所以 State 的适用场景就是只影响当前 View 内部的变化的操作。

  
举个实际的例子就是类似下载网络图片的部分，调用方通常提供一个 URL 和 Placeholder Image，在 SwiftUI 中使用 State 即可，因为此时的网络图变化只影响当前 View。

如 APP 选择界面中，图片资源都来源自网络。

示例代码如下 ：

```
struct NetworkImage: SwiftUI.View {
var urlPath: String?
var placeHodlerImage: UIImage
init(url path: String?, placeHolder: String) {
self.urlPath = path
self.placeHodlerImage = UIImage(named: placeHolder)!.withRenderingMode(.alwaysOriginal)
    }
    @State var downLoadedImage: UIImage? = nil
var body: some SwiftUI.View {
Image(uiImage: downLoadedImage ?? placeHodlerImage)
                  .resizable()
                  .aspectRatio(contentMode: .fill)
                  .onAppear(perform: download)
    }
func download() {
if let _ = downLoadedImage {
return
        }
_ = urlPath.flatMap(URL.init(string:)).map {
ImageDownloader.default.downloadImage(with: $0) { result in
switch result {
case .success(let value):
self.downLoadedImage = value.image.withRenderingMode(.alwaysOriginal)
case .failure(let error):
                    log.debug(error)
                }
            }
        }
    }
}

```

### Binding

在传统的命令式编程中，GUI 程序中最复杂的部分莫过于状态管理，尤其是多数据同步，一个数据存在于不同的 UI 组成部分，UI 各个部分的变化理论上都有同步，状态量的变多加上异步的操作，会使程序的可读性直线下降，并且伴随着而来的就是 Bug ，并且不敢重构。

![](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnju9EVkfjBDQbsBAB52dopLFCzrfI4cD5ibYcuqLQib5hEWQX463V0k6bEflPGxOT0SRlib9o8zVro8uOQ/640?wx_fmt=png)

SwiftUI 给我们的理念就是 Single source of truth, 简单来说就是单一数据源，单一数据源是个很早就有的名词 / 方法，但是很多系统并没有给出很好的解决办法，比如习惯 FRP 的同学可能用 RX/RAC 里面的 Singnal 去描述，但是 FRP 晦涩的概念，又使其在项目中的接入成本大大提高。

  
SwiftUI 给我们的解决办法就是 `@Binding` 。作者之前尝试自己实现一个 Binding，实现起来就是一个简单的闭包，通过闭包捕获 Source of truth 的数据，同时 SwiftUI 会帮我们自动刷新需要同步的界面。使我们的数据同步变的的非常简单。

实际例子如，系统提供的 Control（可操作的 View） 的构造器基本都需要 @Binding 属性，可以自动的同步来自 API 调用方的数据源。

这里举个例子如 项目中的版本选择和日期选择功能, 我们需要讲控件选择的值同步给数据源。

![](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuicy3aSjJrwLfmhWnx7CPQuWCeqCCSe8GF7dkzn3KGc3iamjZbOvxx4iaS0B7F8GnuVgA55nCuXQM3og/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuicy3aSjJrwLfmhWnx7CPQuWBFjBbIr30eHtO7q9NvBAUHLRMHd807vn4vl6XBiam9ktdQZHkQgHgRQ/640?wx_fmt=png)

```
struct DateVersionPanel : View {
@Binding var version: String
@State var input = ""
@Binding var date: Date
var title: String
@State private var showVersionPicker = false
@State private var showDatePicker = false
var dateFormatter: DateFormatter {
let formatter = DateFormatter()
        formatter.dateFormat = "yyyy-MM-dd"
return formatter
    }
private func showDate() {
        showDatePicker = true
    }
var body: some View {
        HStack(alignment: .center) {
            Text(title)
                .font(.system(size: 14))
            HStack(alignment: .center) {
                TextField(version.isEmpty ? "不区分版本" : version, text: $input, onEditingChanged: { (changed) in
                    log.debug("TextFieldonEditing: \(changed)")
                }) {
                    log.debug("TextFielduserName: \(self.version)")
                    self.version = self.input
                }
                .font(.system(size: 9))
                .padding(.leading, 20)
                .frame(width: 100, height: 20)
                NavigationLink(destination: VersionSelectView(version: $version)) {
                    Image("down_arrow")
                        .frame(width: 24, height: 14)
                        .aspectRatio(contentMode: .fill)
                }
                .offset(x: -20)
            }
            .frame(width: 100, height: 25)
            .border(Color.grayText, width: 0.5)
            .padding(.leading, 40)
            NavigationLink(destination: CalendarView(date: self.$date)) {
                HStack {
                    Text(dateFormatter.string(from: date) )
                        .font(.system(size: 9))
                        .padding(.leading, 10)
                    Image("down_arrow").padding(.trailing, 10)
                }
                .frame(width: 100, height: 25)
                .border(Color.grayText, width: 0.5)
                .padding(.leading, 40)
            }
        }
        .padding(.bottom, 10)
    }
}

```

### ObservableObject

ObservableObject 在 Xcode11 Beta 4 之前叫 ObjectBinding , 这个类型是一个协议，要求我们实现一个来自 Combine 框架的 `Subject` Subject 是一个和命令式编程世界交互的桥梁，是一个特殊的 Publisher，SwiftUI 内部会自动的订阅这个 Subject，在  Subject 发送变化时 SwiftUI 会自动刷新数据。

  
ObservableObject 适用于多个 UI 组成部分同步数据，ObservableObject 取代了，Cocoa 框架基本编程风格 MVC 中控制器的角色，暂时项目中就叫他 ViewModel 吧。

`@Published` 是 Xcode11 beta5 之后新增的代理属性，此属性如果用在 ObservableObject 内，如果属性发送了变化，会自动触发 ObservableObject 的 objectWillChanged 的 Subject 变化，自动刷新页面。

同时由于 Combine 框架的支持，多个条件联动变成了一个简单的事情，在 SOT APP 项目中，就非常适合，比如数据大盘，有将近 10 几个数据状态，任何一个触发，都会导致数据刷新。

![](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuicy3aSjJrwLfmhWnx7CPQuWVhKoAIhBjvzMDpJNiay684qRFZUEjvBdCfmW23JRutmfpgkowSibDX7Q/640?wx_fmt=png)

```
class HomeViewModel: ObservableObject {
    @Published var isCorrectionOn = true
    @Published var isForce = false
    @Published var crashType = CrashType.crash
    @Published var pecision = Pecision.fifith
    @Published var quota = Quota.count
    @Published var currentDate = Date()
    @Published var currentVersion = ""
    @Published var comDate = Date().lastDay
    @Published var comVersion = ""
    @Published var refresh = true
    @Published var metric: Metric? = nil
    @Published var trends: [TrendItem] = []
    @Published var summary: Summary? = nil
    var api = SOTAPI()
    // MARK: - Life Cycle
    var cancels = [AnyCancellable]()
    init() {
        var cancel = $refresh.combineLatest($isForce, $isCorrectionOn)
            .combineLatest($crashType, $pecision, $quota)
            .combineLatest($currentDate, $currentVersion)
            .combineLatest($comVersion, $comDate)
            .debounce(for: 0.5, scheduler: RunLoop.main)
            .sink {[weak self] (_) in
                self?.requestMetric()
                self?.requestTrends()
        }
        cancels.append(cancel)
        cancel = $refresh.sink{[weak self] (_) in
             self?.requestSummary()
        }
        cancels.append(cancel)
    }
    func requestMetric() {}
    func requestTrends() {}
    func requestSummary() {}
}

```

### Work with UIKit

由于 SwiftUI 是一个封闭的系统，有时候一些控件还不够丰富，为了满足开发所用，还需要和一些已有的 UIKit 的 UIView 混合编程，一方面可以减少迁移的负担，一方面可以增加 SwiftUI 的能力。

在 SOT 项目中，由于日期选择是一个专业的库，这里采用了第三方库，就涉及到于  UIKit 交互， SwiftUI 提供了一套非常简单清晰的标准，可以用在多个平台上交互，并提供一致的表现力。

![](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnju9EVkfjBDQbsBAB52dopLFCE32hibwByFylLkiaKMB8MRgxarAN93mT3UyDM3ER23O46h2YFt3Ph3WQ/640?wx_fmt=png)

需要注意的是 UIViewRepresentable 的遵守者，是一个 View 容器，此容器会被创建多次，如果内部有数据源需要通知，需要创建相应的 `Coordinator` 将当前的容器当做 View 传递进去，由于 View 是结构体。

  
此时创建的是一个拷贝副本，所以 `Coordinator` 修改的部分，最好只是 `ObservableObject` `Binding`

![](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuicy3aSjJrwLfmhWnx7CPQuW51axRJI4VUTgbR4LvIX9ibCHSxVGtFuRKHcZDof41EbiadGYrK1B8qcg/640?wx_fmt=png)

```
struct CalendarView : UIViewRepresentable {
    @Environment(\.presentationMode) var presentationMode
    @Binding var date: Date
init(date: Binding<Date>) {
self._date = date
    }
func makeUIView(context: UIViewRepresentableContext<CalendarView>) -> UIView {
let view = UIView(frame: UIScreen.main.bounds)
        view.backgroundColor = .backgroundTheme
let height: CGFloat = 300.0
let width = view.frame.size.width
let frame = CGRect(x: 0.0, y: 0.0, width: width, height: height)
let calendar = FSCalendar(frame: frame)
        calendar.locale = Locale.init(identifier: "ZH-CN")
        calendar.delegate = context.coordinator
        context.coordinator.fsCalendar = calendar
        calendar.backgroundColor = UIColor.white
        view.addSubview(calendar)
return view
    }
func makeCoordinator() -> CalendarView.Coordinator {
Coordinator(self)
    }
func updateUIView(_ uiView: UIView, context: UIViewRepresentableContext<CalendarView>) {
        log.debug("Date")
        context.coordinator.fsCalendar?.select(date)
    }
func dismiss() {
        presentationMode.wrappedValue.dismiss()
    }
class Coordinator: NSObject, FSCalendarDelegate {
var control: CalendarView
var date: Date
var fsCalendar: FSCalendar?
init(_ control: CalendarView) {
self.control = control
self.date = control.date
        }
func calendar(_ calendar: FSCalendar, didSelect date: Date, at monthPosition: FSCalendarMonthPosition) {
self.control.date = date
        }
    }
}

```

架构

Combine

在此项目中使用了最基本的 Combine 操作，由于项目一期主要是为了探索 SwiftUI ，所以并未对架构模式做精细的设计，可以观察到，ViewModel，内部还是有订阅，发送网络请求，最后同步数据的操作，这种编码方式，还是典型的命令式编程风格，此部分会在项目二期逐渐探索中修改为响应式风格。

### Redux/Flux

SwiftUI 是一个单向数据流框架，在此之前，大前端已经有 React， Flutter ， Reactive Native，等比较流行的框架。在这些单向数据流得框架下，Redux 作为一种比较流行的状态管理的架构风格，已经经过多方面的验证，SwiftUI 对于 Redux 也是比较适用的。  

Redux 的基本思想核步骤是：

1.  整个页面甚至 APP 是一个巨大的状态机，有一个状态存储 Store ，在某个时刻处于某种状态。
    

2.  状态在页面表达中是一个简单的树型结构，在 SwiftUI，对应的 就是 View Tree。
    

3.  View 操作不能直接修改状态，只能通过发送 Action, 间接改变 Store。
    

4.  Reducer 通过 Action 加上 oldState 获取 newSatete。简单来说就是  State  = f(action+oldState)。
    

附上一份 阮一峰的 Redux 入门教程的示例图：

![](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuicy3aSjJrwLfmhWnx7CPQuWyVVusPUjbI5EcXibibcsric9OtMRpyBiaPibZH8vRcby48qO3Og6eWYasNQ/640?wx_fmt=png)

这套风格在前端大型项目中已经了验证，可以比较清晰的表达用户事件交互和状态管理。  
目前由于 SwiftUI 中 ViewCtonroller 的消失，加上方便的 `ObserableObject` 和 `EmviromentObject` 。

  
SOT 项目一期暂未采用，在二期项目中会探索合适的架构设计。

项目总结

此项目在短短的 10 个工作日内就能完成，不得不说 SwiftUI 的开发效率真的惊人，虽然目前还有一些 Bug ，但是相信在未来，SwiftUI 会是 Apple 平台 UI 布局的解决办法，关于 SwiftUI 如何在淘系落地业务，还在持续探索中。  

目前此项目已在集团内部开源。

参考  

-----

1、Session 402 What's New in Swift 

2、Session 204 Introducing SwiftUI: Building Your First App

3、Session 216 Introducing SwiftUI Essentials

4、Session 226 SwiftUI Essentials

5、Session 231 Integrating SwiftUI

6、Session 237 Building Custom Views with SwiftUI

7、Session 238 accessibility in swiftui

8、Session 240 SwiftUI on all Device

9、Session 219 SwiftUI on watchOS

10、Session 415 Modern Swift API Design

11、FSCalendar

12、Kingfisher

13、Redux 入门教程（一）：基本用法

14、Redux 中文文档

END
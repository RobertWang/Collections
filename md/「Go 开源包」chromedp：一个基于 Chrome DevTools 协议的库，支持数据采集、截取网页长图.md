> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/rGPsEt-XDOayDWwLGxvcsw)

> 数据采集、截图网页长图、网页内容转 pdf、模拟网页点击的一款利器

大家好，我是渔夫子

今天给大家推荐一个基于 Chrome DevTools 协议的 Go 语言库：**chromedp**。

该库提供了一种简单、高效、可靠的方式来控制 Chrome 浏览器进行自动化测试和爬取数据。

项目地址：https://github.com/chromedp/chromedp

**它可以****模拟用户在浏览器中执行各种操作**，**如点击**、**输入文本、截取网页长图、将网页内容转换成 pdf 文档、下载图片等，从而获取到需要采集的数据**。

基础用法
----

chromedp 的基本用法非常简单，只需要定义一个任务列表，然后将其传递给 chromedp.Run 函数即可。下面是一个简单的例子。这个例子的功能如下：

*   **chromedp.Navigate**：打开 https://pkg.go.dev/time 网页
    
*   **chromedp.WaitVisible：**等待网页加载完成
    
*   **chromedp.Click：**点击 #example-After 标签。也就是网页中的 After 函数示例
    
*   **chromedp.Value：**将示例代码的值读取到 example 变量中**。**
    
*   **最后输出 example 变量**
    

如下：

```
package main

import (
 "context"
 "log"
 "time"
    "github.com/chromedp/chromedp"
)

func main() {
 // create chrome instance
 ctx, cancel := chromedp.NewContext(
  context.Background(),
  // chromedp.WithDebugf(log.Printf),
 )
 defer cancel()

 // create a timeout
 ctx, cancel = context.WithTimeout(ctx, 15*time.Second)
 defer cancel()

 // navigate to a page, wait for an element, click
 var example string
 err := chromedp.Run(ctx,
  chromedp.Navigate(`https://pkg.go.dev/time`),
  // wait for footer element is visible (ie, page is loaded)
  chromedp.WaitVisible(`body > footer`),
  // find and click "Example" link
  chromedp.Click(`#example-After`, chromedp.NodeVisible),
  // retrieve the text of the textarea
  chromedp.Value(`#example-After textarea`, &example),
 )
 if err != nil {
  log.Fatal(err)
 }
 log.Printf("Go's time.After example:\n%s", example)
}


```

高级使用
----

除了基本用法之外，chromedp 还提供了许多高级功能。

### 截屏

将网页截取成图片有两个函数：`chromedp.Screenshot`和`chromedp.FullScreenshot`。其中 chromedp.Screenshot 是按网页中的某个 div 的元素截取。而 chromedp.FullScreenshot 是截取整个网页。我们看下下面的例子：

```
package main

import (
 "context"
 "log"
 "os"

 "github.com/chromedp/chromedp"
)

func main() {
 // create context
 ctx, cancel := chromedp.NewContext(
  context.Background(),
  // chromedp.WithDebugf(log.Printf),
 )
 defer cancel()

 // capture screenshot of an element
 var buf []byte
 if err := chromedp.Run(ctx, elementScreenshot(`https://pkg.go.dev/`, `img.Homepage-logo`, &buf)); err != nil {
  log.Fatal(err)
 }
 if err := os.WriteFile("elementScreenshot.png", buf, 0o644); err != nil {
  log.Fatal(err)
 }

 // capture entire browser viewport, returning png with quality=90
 if err := chromedp.Run(ctx, fullScreenshot(`https://brank.as/`, 90, &buf)); err != nil {
  log.Fatal(err)
 }
 if err := os.WriteFile("fullScreenshot.png", buf, 0o644); err != nil {
  log.Fatal(err)
 }

 log.Printf("wrote elementScreenshot.png and fullScreenshot.png")
}

// elementScreenshot takes a screenshot of a specific element.
func elementScreenshot(urlstr, sel string, res *[]byte) chromedp.Tasks {
 return chromedp.Tasks{
  chromedp.Navigate(urlstr),
  chromedp.Screenshot(sel, res, chromedp.NodeVisible),
 }
}

// fullScreenshot takes a screenshot of the entire browser viewport.
//
// Note: chromedp.FullScreenshot overrides the device's emulation settings. Use
// device.Reset to reset the emulation and viewport settings.
func fullScreenshot(urlstr string, quality int, res *[]byte) chromedp.Tasks {
 return chromedp.Tasks{
  chromedp.Navigate(urlstr),
  chromedp.FullScreenshot(res, quality),
 }
}


```

该示例就是通过 elementScreenshot 函数中截取了 https://pkg.go.dev / 中的 img.Homepage-logo 标签的图片。另外一个就是通过 fullScreenshot 函数来截取了 https://brank.as / 网站的长图。因为图像较大，大家可以运行代码查看具体的效果。

### 其他功能

*   模拟表单提交：可以使用 chromedp.Submit 函数模拟表单提交。
    
*   模拟鼠标滚动：可以使用 chromedp.ScrollIntoView 函数模拟鼠标滚动。
    
*   模拟键盘输入：可以使用 chromedp.KeyEvent 函数模拟键盘输入。
    

github 上也给出了具体的示例代码，大家可以自行查看。示例链接：https://github.com/chromedp/examples

chromedp 的应用场景
--------------

由于 chromedp 具有高效、稳定、可靠的特点，因此在以下场景中得到了广泛的应用：1. 数据采集：可以使用 chromedp 对各类网站进行数据采集。2. 自动化测试：可以使用 chromedp 对 Web 应用进行自动化测试。3. 网络爬虫：可以使用 chromedp 对各类网站进行爬取。4. 数据分析：可以使用 chromedp 对采集到的数据进行分析和处理。

总结
--

chromedp 基于 Chrome DevTool 协议实现。可以对网页内容进行采集、模拟点击、提交数据、将网页内容转换成 pdf、抓取网页长图等功能。

**特别说明：**你的关注，是我写下去的最大动力。点击下方公众号卡片，直接关注。关注送《100 个 go 常见的错误》pdf 文档、经典 go 学习资料。
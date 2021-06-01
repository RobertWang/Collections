> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1451562&extra=page%3D1%26filter%3Dauthor%26orderby%3Ddateline) ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)15035252367  _ 本帖最后由 15035252367 于 2021-6-1 19:42 编辑_  

前段时间 公司做活动 要求生成一些头像 但是我太懒了 (主要是不会)  
我就爬了一些

源码地址  [https://gitee.com/code5/go-test/blob / 爬虫 / 图片爬虫 / 爬图片. go](https://gitee.com/code5/go-test/blob/爬虫/图片爬虫/爬图片.go)

贴一下代码吧  互相学习

```
    package main

    import (
    "fmt"
    "io/ioutil"
    "net/http"
    "regexp"
    "sync"
    )

    var (

    //图片正则
    reImage = `"((http://)(.*?)(\.jpg))"`

    //大标题列表 图片
    //class="nr"
    reTitlediv = `<ul[\s\S]class=PicList>[.*\s\S]+<div[\s\S]class="showpage">`
    reTitleurl = `"(https?://)(.*?)(\.((jpg)|(gif)|(bmp)|(jpeg)))"`
    reTitleList = `href="(/touxiang/\d+\.htm)"><`

    //内容页面
    reContentdiv = `<div[\s\S]class="wznr">[.*\s\S]+<div[\s\S]class="wzsx">`

    //图片名字
    imageTitle = `/([A-Z]{2}\d{2,4}_\d{1,3}\.jpg)`
    )
    var wg sync.WaitGroup

    //获取页面数据
    func getUrl(url string) string {
    resp, err := http.Get(url)
    Handerr(err, "http.Get(youxiang)")
    defer resp.Body.Close()

    bodyByte, err := ioutil.ReadAll(resp.Body)
    Handerr(err, "ioutil.ReadAll(resp.Body)")
    bodyStr := string(bodyByte)
    return bodyStr
    }

    //正则分析页面数据
    func FXdata(htmlStr, reStr string) [][]string {
    re := regexp.MustCompile(reStr)
    list := re.FindAllStringSubmatch(htmlStr, -1)
    return list
    }

    //保存图片
    func savepic(url string) {
    defer wg.Add(-1)
    imageNameStr := FXdata(url, imageTitle)
    //fmt.Println(imageNameStr)
    //return
    imageName := imageNameStr[0][1]
    fmt.Println(imageName, "正在下载")
    resp, err := http.Get(url)
    defer resp.Body.Close()
    if err != nil {
    fmt.Println(err)
    }
    body, err := ioutil.ReadAll(resp.Body)
    if err != nil {
    fmt.Println(err)
    }
    err = ioutil.WriteFile(imageName, body, 0777)
    fmt.Println(err)

    }

    //1.0 解析首页
    func shouye(url string) {
    //fmt.Println(url)
    body := getUrl(url)
    //fmt.Println(body)

    //解析区块内容
    list := FXdata(body, reTitlediv)
    //fmt.Println(list[0][0])
    //当前区块 跳转链接
    htmlList := FXdata(list[0][0], reTitleList)
    fmt.Println(htmlList)

    for _, val := range htmlList {
    neirong("http://www.2qqtouxiang.com" + val[1])
    }

    }

    //2.0 内容
    func neirong(imgUrl string) {

    body := getUrl(imgUrl)

    //解析区块内容
    list := FXdata(body, reContentdiv)

    //获取图片地址
    imgList := FXdata(list[0][0], reImage)

    fmt.Println("页面:"+imgUrl+"数据个数:", len(imgList))
    for _, val := range imgList {
    wg.Add(1)
    go savepic(val[1])
    }
    }

    func main() {
    for i := 77; i >= 50; i-- {
    idstr := fmt.Sprintf("%d", i)
    imgUrl := "http://www.2qqtouxiang.com/touxiang/list" + idstr + ".htm"
    fmt.Println(imgUrl)
    shouye(imgUrl)
    }

    wg.Wait()

    }

    func Handerr(err error, msg string) {
    if err != nil {
    fmt.Println(msg, err)
    }
    }
```

![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)nanaqilin  谢谢楼主分享 ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) First 丶云心 谢谢分享，学习学习！![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)hate  不用 goquery？
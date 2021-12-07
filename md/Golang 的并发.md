> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7035924960639975460) 当前业务场景的需求是请求动态和静态数据，为减少耗时，采用并发的形式，即两件事一起做。 不过最后需要把动态和静态数据都各自处理下然后聚合成一个结果统一返回给上游。 都说 golang 是为高并发而生，此次使用 golang 的某些特性来实现。

### 涉及知识点

*   sync.WaitGroup{}
*   channel

### 两种方式

1.  只使用 WaitGroup{}，使用两个数据结构记录两个协程的返回
2.  使用 WaitGroup{}，并使用 channel 完成通信统一使用一个数据结构处理返回

### Only WaitGroup

```
// +build ignore

package main

import (
   "fmt"
   "sync"
)

/*
   @func: 测试waitGroup的使用
   @time: 2021/11/29
   @author: Andy_文铎
 */

func main() {
   var wg sync.WaitGroup
   wg.Add(2)
   staticData := make(map[string]interface{})
   dynamicData := make(map[string]interface{})
   static(staticData, &wg)
   dynamic(dynamicData, &wg)
   wg.Wait()
   fmt.Println("static and dynamic done")
   fmt.Println("staticData->", staticData)
   fmt.Println("dynamicData->", dynamicData)
}

// 静态数据获取
func static(staticData map[string]interface{}, wg *sync.WaitGroup) {
   defer wg.Done()
   staticData["static"] = "data"
   staticData["index"] = 1
}

// 动态数据获取
func dynamic(dynamicData map[string]interface{}, wg *sync.WaitGroup) {
   defer wg.Done()
   dynamicData["dynamic"] = "data"
   dynamicData["index"] = 0
}
复制代码
```

输出结果：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/36639d67ed234c1fa5bfb61e607535b6~tplv-k3u1fbpfcp-watermark.awebp?)

### WaitGroup And Channel

```
// 并发获取动静态的数据
func getAllData(uid, source, from, logId, kind string, richSel []string, logBuf *logrus.Entry) (string, error) {
   var wg sync.WaitGroup
   var result []byte
   resultChannel := make(chan map[string]interface{}, 2)
   defer close(resultChannel)
   resultMap := make(map[string]interface{})
   testMap := make(map[string]interface{})
   wg.Add(3)
   // 不断接收动静态数据
   go func() {
      var flag int = 0
      for response := range resultChannel {
         flag++
         for k, v := range response {
            resultMap[k] = v
            fmt.Printf("%v-> %v\n", k, v)
         }
         if flag >= 2 {
            break
         }
      }
      outMap := make(map[string]interface{})
      outMap["data"] = resultMap
      outMap["err_code"] = 0
      outMap["err_msg"] = "ok"

      result, _ = json.Marshal(outMap)
      wg.Done()
   }()
   go getWaitStatic(uid, source, from, logId, kind, resultChannel, logBuf, &wg)
   go getWaitDynamic(uid, logId, kind, richSel, resultChannel, logBuf, &wg)

   wg.Wait()

   fmt.Println("test->", testMap)
   return string(result), nil
}
复制代码
```

即通过开一个协程获取两次数据，在动态数据和静态数据的协程中传递数据，当 channel 获取了两次数据之后，对返回数据进行封装。

### 总结

以上两种方式都可以实现数据的共享，后续更多用法（包括控制超时等等）还待不断实践~
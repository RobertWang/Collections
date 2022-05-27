> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzkyMDAzNjQxMg==&mid=2247484655&idx=1&sn=b127760140e74d65623efe3a933d52f6&chksm=c199b1bcf6ee38aa3a284dada7ef6a35851a1945063d50ddfff0cf509f244163668073893843&scene=132#wechat_redirect)

前言
--

前段时间在一个项目里使用到了分布式锁进行共享资源的访问限制，后来了解到 Golang 里还能够使用 singleflight 对共享资源的访问做限制，于是利用空余时间了解，将知识沉淀下来, 并做分享

文章尽量用通俗的语言表达自己的理解，从入门 demo 开始，结合源码分析 singleflight 的重点方法，最后分享 singleflight 的实际使用方式与需要注意的 “坑 “。

另外需注意， 本文分析的 singleflight 源码位于 https://cs.opensource.google/go/x/sync/+/036812b2:singleflight/singleflight.go

这一点请区别于 https://github.com/golang/groupcache/blob/master/singleflight/singleflight.go 的实现。

定义
--

按照官方文档的定义, singleflight 提供了一个重复的函数调用抑制机制

```
Package singleflight provides a duplicate function call suppression<br data-darkmode-color-16536705016472="rgb(171, 178, 191)" data-darkmode-original-color-16536705016472="#fff|rgb(53, 53, 53)|rgb(171, 178, 191)" data-darkmode-bgcolor-16536705016472="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16536705016472="#fff|rgb(40, 44, 52)">
```

用途
--

通俗的来说就是 singleflight 将相同的并发请求合并成一个请求，进而减少对下层服务的压力，通常用于解决缓存击穿的问题

*   缓存击穿是指: 在高并发的场景中，大量的 request 同时请求查询一个共享资源（例如 Redis 缓存的 key） ，如果这个共享资源正好过期失效了，就会导致大量相同的 request 都打到 Redis 下游的数据库，导致数据库的负载上升。
    
    ![](https://mmbiz.qpic.cn/mmbiz/JTbQwON6HwJEJlybEaJY3EN2a7JjdbsdkiaX9hlf84pEdbwv4tDSarfOxnQIOK1ic9GgHRKI9lTzjn0BeMCVat6w/640?wx_fmt=other)
    

简单 Demo
-------

```
var (    sfKey1 = "key1"    wg     *sync.WaitGroup    sf     singleflight.Group    nums   = 10)func getValueService(key string) { //service   var val string   wg = &sync.WaitGroup{}   wg.Add(nums)   for idx := 0; idx < nums; idx++ { // 模拟多协程同时请求      go func(idx int) { // 注意for的一个小坑         defer wg.Done()         value, _ := getAndSetCacheNoChan(idx, key) //简化代码，不处理error         log.Printf("request %v get value: %v", idx, value)         val = value      }(idx)   }   wg.Wait()   log.Println("val: ", val)   return}// getValueBySingleflight 使用singleflight取cacheKey对应的value值func getValueBySingleflight(idx int, cacheKey string) (string, error) {   log.Printf("idx %v into-cache...", idx)   // 调用singleflight的Do()方法   value, _, _ := sf.Do(cacheKey, func() (ret interface{}, err error) {      log.Printf("idx %v is-setting-cache", idx)      // 休眠0.1s以捕获并发的相同请求      time.Sleep(100 * time.Millisecond)      log.Printf("idx %v set-cache-success!", idx)      return "myValue", nil   })   return value.(string), nil}
```

看看实际效果

![](https://mmbiz.qpic.cn/mmbiz/JTbQwON6HwJEJlybEaJY3EN2a7JjdbsdbyTIpaich8GgUmS9N0XETSWcf4AhCjvkRqibeGj3j6qicr1JZOa3t6buA/640?wx_fmt=other)

*   由结果图可以看到，索引 = 8 的协程第一个进入了 Do() 方法，其他协程则阻塞住, 等到 idx=8 的协程拿到执行结果后，协程以乱序的形式返回执行结果。
    
*   相同 key 的情况下，singleflight 将我们的多个请求合并成 1 个请求。由 1 个请求去执行对共享资源的操作。
    

源码分析
----

### 结构

```
type (   Group struct { // singleflight实体      mu sync.Mutex       // 互斥锁      m  map[string]*call // 懒加载   }   call struct {      wg sync.WaitGroup      // 存储 调用singleflight.Do()方法返回的结果      val interface{}      err error      // 调用singleflight.Forget(key)时将对应的key从Group.m中删除      forgotten bool      // 通俗的理解成singleflight合并的并发请求数      dups  int      // 存储 调用singleflight.DoChan()方法返回的结果      chans []chan<- Result   }   Result struct {      Val    interface{}      Err    error      Shared bool   })
```

### 对外暴露的方法

```
func Do(key string, fn func() (interface{}, error)) (v interface{}, err error, shared bool)   func DoChan(key string, fn func() (interface{}, error)) <-chan Result) // 将key从Group.m中删除func Forget(key string)
```

`DoChan()`和`Do()`最大的区别是`DoChan()`属于异步调用，返回一个 channel，解决同步调用时的阻塞问题

### 重点方法分析

Do

```
func (g *Group) Do(key string, fn func() (interface{}, error)) (v interface{}, err error, shared bool) {   g.mu.Lock() // 加互斥锁   if g.m == nil { // 懒加载map      g.m = make(map[string]*call)   }   if c, ok := g.m[key]; ok { // 检查相同的请求已经是否进入过singleflight      c.dups++      g.mu.Unlock()      c.wg.Wait() // 调用waitGroup的wait()方法阻塞住本次调用，等待第一个进入singleflight的请求执行完毕拿到结果，将本次请求唤醒.      if e, ok := c.err.(*panicError); ok { //如果调用完成，发生error ，将error上抛         panic(e)      } else if c.err == errGoexit {         runtime.Goexit()      }      // 返回调用结果      return c.val, c.err, true   }   c := new(call) // 相同的请求第一次进入singleflight   c.wg.Add(1)   g.m[key] = c // new一个call实体，放入singleflight.call这个map   g.mu.Unlock()   g.doCall(c, key, fn) //实际执行的函数   return c.val, c.err, c.dups > 0}
```

完整流程图如下

![](https://mmbiz.qpic.cn/mmbiz/JTbQwON6HwJEJlybEaJY3EN2a7JjdbsdHNfzn4hBQtWbvqlia74PRMVgR32XbGUcbib5co5yZYOgcbqw2jddEghg/640?wx_fmt=other)

由源码可以分析出，最后实际执行我们业务逻辑的函数其实是放到了`doCall()`里，我们稍后分析这个函数

Forget

再简单看看 Forget() 函数, 很短

```
func (g *Group) Forget(key string) {   g.mu.Lock()   if c, ok := g.m[key]; ok {      c.forgotten = true // key的forgotten标志位记为true   }   delete(g.m, key)  // Group.m中删除对应的key   g.mu.Unlock()}
```

doCall

```
func (g *Group) doCall(c *call, key string, fn func() (interface{}, error)) {   normalReturn := false   recovered := false    //使用双重defer来区分error的类型: panic && runtime.error   defer func() {       if !normalReturn && !recovered {        // fn()发生了panic且fn()中的panic没有被recover掉        // errGoexit连接runtime.Goexit错误         c.err = errGoexit       }      c.wg.Done()      g.mu.Lock()      defer g.mu.Unlock()      if !c.forgotten { // 检查key是否调用了Forget()         delete(g.m, key)      }      if e, ok := c.err.(*panicError); ok {         // 如果返回的是 panic 错误，为了避免channel被永久阻塞，我们需要确保这个panic无法被recover         if len(c.chans) > 0 {            go panic(e)  // panic无法被恢复            select {} // 阻塞本goroutine以确保本goroutine出现在奔溃堆栈中         } else {            panic(e)         }      } else {         // 将结果正常地返回         for _, ch := range c.chans {            ch <- Result{c.val, c.err, c.dups > 0}         }      }   }()   func() {      defer func() {         if !normalReturn {            // 表示fn()发生了panic()            // 此时与panic相关的堆栈已经被丢弃(调用的fn()) ，无法通过堆栈跟踪去确定error类型            if r := recover(); r != nil {               c.err = newPanicError(r) //new一个新的自定义panic err,往第一个defer抛            }         }      }()     // 执行我们实际的业务逻辑，并将业务方法的返回值赋给singleflight.call      c.val, c.err = fn()      // 如果fn()发生panic，normalReturn无法被赋值为true，而是进入doCall()的第二个defer()      normalReturn = true   }()   // 如果normalResult为false时，表示fn()发生了panic   // 但是执行到了这一步，表示fn()中的panic被recover了   if !normalReturn {      recovered = true // recovered标志位置为true   }}
```

由以上分析可以得出几个重要的结论

1.  singleflight 主要使用 sync.Mutex 和 sync.WaitGroup 进行并发控制
    
2.  对于 key 相同的请求, singleflight 只会处理的一个进入的请求，后续的请求都会使用 waitGroup.wait() 将请求阻塞
    
3.  使用双重 defer() 区分了 panic 和 runtime.Goexit 错误，如果返回的是一个 panic 错误，group.c.chans 会发生阻塞，那么需要抛出这个 panic 且确保其无法被 recover
    

实际使用
----

分享一段实际项目中使用 singleflight 结合本地缓存的代码模版

```
func (srv Service) getDataBySingleFlight(ctx  context.Context) (entity.List, error) {    // 1. 从localCache查    resData, err := local_cache.Get(ctx, key)    if err != nil {       log.Fatalln()       return resData, err    }    if resData != nil {       return resData, nil    }    // 2. localCache无数据，从redis查    resData, err = srv.rdsRepo.Get()    if err != nil && err != redis.Nil {       // redis错误       log.Fatalln()       return resData, err    } else if redis.Nil == err {           // redis无数据 ，查db           resData, err, _ = singleFlight.Do(key, func() (interface{}, error) {           // 构建db查询条件          searchConn := entity.SearchInfo{}           //  建议休眠0.1s 捕获0.1s内的重复请求          time.Sleep(100 * time.Millisecond)           // 4. 查db          data, err := srv.dBRepo.GetByConn(ctx, searchConn)          if err != nil {             log.Fatalln()             return data, err          }           // 5. 回写localCache && redisCache          err = local_cache.Set(ctx, data)          if err != nil {             log.Fatalln()          }          err = srv.rdsRepo.Set(ctx, data)          if err != nil {             log.Fatalln()          }      // 返回db数据,回写cache的error不上抛      return data, nil   })   return resData, err}return resData, nil
```

弊端与解决方案
-------

singleflight 当然不是解决问题的银弹，在使用的过程中有一些 “坑” 需要我们注意

*   `Do()`方法是一个同步调用的方法，无法处理下游服务调用的超时情况
    

解决方案：使用 singleflight 的`DoChan()`方法，在 service 层使用 channel+select 做超时控制

```
func enterGetAndSetCacheWithChan(ctx context.Context, key string) (str string, err error) {   tag := "enterGetAndSetCacheWithChan"   sonCtx, _ := context.WithTimeout(ctx, 2 * time.Second)   val := ""   nums := 10 //协程数   wg = &sync.WaitGroup{}   wg.Add(nums)   for idx := 0; idx < nums; idx++ {      go func() {         defer wg.Done()         val, err = getAndSetCacheWithChan(sonCtx, idx, key)         if err != nil {            log.Printf("err:[%+v]", err)            return         }         str = val      }()   }   wg.Wait()   log.Printf("tag:[%s] val:[%s]", tag, val)   return}func getAndSetCacheWithChan(ctx context.Context, idx int, cacheKey string) (string, error) {   tag := "getAndSetCacheWithChan"   log.Printf("tag: %s ;idx %d into-cache...", tag, idx)   ch := sf.DoChan(cacheKey, func() (ret interface{}, err error) { // do的入参key，可以直接使用缓存的key，这样同一个缓存，只有一个协程会去读DB      log.Printf("idx %v is-setting-cache", idx)      time.Sleep(100 * time.Millisecond)      log.Printf("idx %v set-cache-success!", idx)      return "myValue", nil   })   for { // 选择 context + select 超时控制      select {      case <-ctx.Done():         return "", errors.New("ctx-timeout") // 根据业务逻辑选择上抛 error      case data, _ := <-ch:         return data.Val.(string), nil      default:      }   }}
```

*   如果第一个请求失败了，那么所有等待的请求都会返回同一个 error
    

解决方案：根据实际情况，结合下游服务调用耗时与下游实际能支持的 QPS 等数据，对 key 做定时 Forget()。

```
go func() {    time.Sleep(100 * time.Millisecond)    g.Forget(key)}()
```

最后，衷心希望本文能够对各位读者有一定的帮助。
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666555795&idx=2&sn=50a2d6fb3a2db7598e3a98e06c1b7dc4&chksm=80dcad38b7ab242ea916616f33cd90b18b682d91a843e20e8ccad5c7ec22e9023f247fb6895b&scene=21#wechat_redirect)

> 本文采取对话形式，有两个登场人物，左边是导师：欧阳狂霸，右边是实习生：三多。
> 
> 提示：浅色主题阅读体验更佳。  

Mutex 介绍  

-----------

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSelibFU5ZESxReKy5mefuGe0IcibFgllmvDSSrFQkibq8w0y5fcIqkT7Zw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSicic4QziaPKSfibXqGngXqpDqBPaJiaQRl96b2I1TjHSfjZ9Ya8t1pI3sIA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSIR9uwoOUY3AhZA8EAoHRKdTolLhPZwkWa6jMHPEA7hfmID6HDmVb3w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSYW1jWETXKHeIAB4FzUakibxZUD9eNZTjxW4GRsbBxAPtxG2fIiaPicJpQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSBCzpCD8LS2R0cyhIbtqsrIUdDkxnb8P5f6ZMMpmODjrI7W2xcIN1EQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSoIfmic4o3icekTms13wXoibaWia2pMXzzia8pLxQN3QHtZjqcDgJ0RibBpWQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSBeSDh3K7xQwibL28icBE3ecVME5pBRA7qzicNOaqKQLWSVhPKZYbGBtIg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSz3OWZ97bcrLMklIRp83Fu2icImD5qOgiclETg8WibcSY3wdp62rEmK9PQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSs2rgdElib6TxUFZAxJ48sUj0xOskQmBC7ibguRV7TnmNvMX5SkgnkT1A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzStqicb1oZdYwbu6xqW1FdvJExrwwga0JfMXkQ3siaggCQfvs0h5FsuGyA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSibHfKyqKmVn2abMmFxNCoMkmh8wuvnk6OLdR0Mmk1ian7qLnKBOgia1Hg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSQ1TQBN5meeiabyFtLZGiaMN4C1hZ2lSfRAndpd1ssePX7MhmaOpdSvlA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSOqXuj0iclLn5Cib3J6DQoRDKzNg5oNQNT8jYVJwDiaccwhQOdv5hQkZzA/640?wx_fmt=png)

  

  

欧阳狂霸小课堂

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSmLWiaxxA43WdGLTdocn3rofe0xHNdd8atfjYsJ3SibFLZU7uZ8yDyz2Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSbWOHibzF1xWW1H3IyT1bJL29zxPE5vKNRJ1NK5xWvYICOKyFYNrEAbg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSEsJEBEoYCldRIhIsVOGajmu8FaY5mUb2Ckj0wFwCz9sNlggmN9cDJQ/640?wx_fmt=png)

<<<左右滑动见更多>>>

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSNEroOjRib6CR5sO6nSt2bTW3U8AhicoQicsib2fdbATraMeial02BWZB8Kg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSCBZmQvG2FcSbvBTxUT1SuMDBSkbFcbMsOIv4icGhEjYticpayVFo5t1g/640?wx_fmt=png)

```
func concurIncr() {
 var wg sync.WaitGroup

 wg.Add(10)

 var a int
 for i := 0; i < 10; i++ {
  go func() {
   defer wg.Done()
   for j := 0; j < 1000; j++ {
    a++
   }
  }()
 }
 wg.Wait()

 println(a) // 小于10000的随机数。
}
```

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSfmiadibulbib7OvnzbbTgVvBgh8yI4Pwa8Ochj63fV09MdfMAoVclFr8A/640?wx_fmt=png)

  

  

欧阳狂霸小课堂

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSjkxTWYXCQABt2sN7YwnOkrLDK5KOgqAmjKmIoXzMJMyXWjo9V7UG1A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSMsaKhdayxZB4ljKofibZAPnkdf2sBH8vTgxtYPPhWSUA0K40h0JXn4A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSz0b1jQuwVN3sLNFO0Nj4DzMt15pfHaZq7IZM0qoXRQznSVUOiaiba7Bg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzS8eprN3iahKMdyIOKnicMJ8v6BOg2L2s4HTsicHrLjFYWiaiaqrIP4YricT2Q/640?wx_fmt=png)

<<<左右滑动见更多>>>

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSeUpicU2JKYIgcpsBe7EaibdlYCYGFoSp39oyf87AnuKh0pb7IBCFxPog/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSBvq6svgdJbicbpABmw2I9eE94ib4vGtG7yRAZ7qkPInLWXPJh1fdNXCg/640?wx_fmt=png)

  

  

  

  
  

  

  

  

  

  

CAS 用来保证加锁过程中的原子性，会比较地址指针是否为 old，

 为 old 说明操作期间没有被打断，

就地址值替换为 new，返回 true,

不为 old, 代表操作期间被其他线程打断了，返回 false。

  

  

CAS 伪代码：  

```
 bool Cas(int32 *val, int32 old, int32 new)
 Atomically:
 if(*val == old){
  *val = new;
  return 1;
 }else
  return 0;
```

CAS 汇编码：

```
TEXT runtime∕internal∕atomic·Cas(SB), NOSPLIT, $0-13
MOVLptr+0(FP), BX
MOVLold+4(FP), AX
MOVLnew+8(FP), CX
LOCK
CMPXCHGLCX, 0(BX)
SETEQret+12(FP)
RET
```

  

  
![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSdwGJPVBkuBpCHKOIBsibvcMoqyWSFuAISpsl5gcw7lHOx0ic5JlQMXxg/640?wx_fmt=png)

  

```
func main() {
 a := 1

 var mu sync.Mutex

 go func() {
  mu.Lock()
  println("lock")
  a += 2
  //time.Sleep(time.Second)
  time.Sleep(time.Second)
  println("unlock")
  mu.Unlock()
 }()

 go func() {
  println("other")
  a += 3
  println("done")
 }()

 time.Sleep(time.Second * 5)
}
//print :
//   lock
//   other
//   done
//   unlock
```

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSRI47a5DGCW3VpmQ3BruTUPYKRd4Ob6wnBzBme8XGibgKwTEOdOuDHrg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSGNF5DehMoCibVED9f5waFhiaSVvRyS2QsBv64ulPaNYdBVWHQumJaOOw/640?wx_fmt=png)

Mutex 使用
--------

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzS5pEzeA69DvfQ6yeg3e26w5FFUkewQQicC6hqaBicYEnGESMeydQVJ26g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSubMWTpO4JzvAEd4NKMJ6Gn5m3YL7pQGcHz9AWWmzHb32qapJL2t1sw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSDYnTmkeYicT3jerfFyxbtDia0ThQ74fAYR0mmibHIic3amdVUPBpB8ZOsA/640?wx_fmt=png)

> 1.  等待的 goroutine 执行的顺序是什么
>     
> 2.  非 cpu 中的 goroutine 的切换是有性能损耗的，正处于 cpu 切换的 goroutine 的切换不需要做上下文切换，如何设计以获得更好的性能
>     
> 3.  如果某个 goroutine 一直得不到调度怎么办
>     
> 4.  有的协程临界资源可能很快，但如果恰好其他线程抢锁没抢到，那它就要去排队吗？有没有啥方法让这个协程多抢一会
>     

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSc04sCCKjickibiciamVjHn6icf8yJshlXvNxQ387aVchp1sClUrmiaOSqKAw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSL5Po6FUkuicdPIjNQIAOY129jjuH1kXrbj9fob51gqSTPiaEYlwLKJqA/640?wx_fmt=png)

  

  

欧阳狂霸小课堂  

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSzZg8u9yqRNLD9yb45PwibqVu71UUGxfRLN5hbWJpfPS4TETZltCTNDQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSAAiawKo82qhZ5gQiahOibWr8Plp9iaGRcKQJ6j36Lsaz1h5uGJkNr8Dcpg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSOh1KgplJl94ukMDQeMN34yYxCbIkHicVfIBnq0DuibqBUROSh28kupRQ/640?wx_fmt=png)

<<<左右滑动见更多>>>

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzS9PRDTM4PB1AMLtf0phmBp9nSTNr9217Z175zm2oXTU90LxPchcMeDg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSZUq3rQUdnibvqqwb1YSQYmkX2QavsFJhTNze6EQnDcPBPqJ49Jkbibvg/640?wx_fmt=png)

```
func main() {
 a := 1

 var mu sync.Mutex

 go func() {
  mu.Lock()
  println("lock")
  a += 2
  //time.Sleep(time.Second)
  time.Sleep(time.Second)
 }()

 go func() {
  a += 3
  println("unlock")
  println(a)
  mu.Unlock()
 }()

 time.Sleep(time.Second * 5)
}
```

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSpVPsUgg22s6I2Ax0cnngTMWYQXkKgOSdZsCuTzS0YULLKib0fMYaia9A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSIhUx3ksdliaciaYicxpIM6xOQofjzkQwscPEicJvRFQbmfd15IyAKIGJEg/640?wx_fmt=png)

```
func concurIncr() {
 var wg sync.WaitGroup
 var mu sync.Mutex

 wg.Add(10)

 var a int
 for i := 0; i < 10; i++ {
  go func() {
   defer wg.Done()
   for j := 0; j < 1000; j++ {
    mu.Lock()
    a++
    mu.Unlock()
   }
  }()
 }
 wg.Wait()

 println(a) // 10000
}
```

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSwViaicRDaVjZHQQ6yiawYULxjK5xtDWic8iaWF8iapzIeKFSC99iawt4qDSnQ/640?wx_fmt=png)

```
type SafeCounter struct {
 mu sync.Mutex
 v  map[string]int
}

func (c *SafeCounter) Inc(key string) {
 c.mu.Lock()
 // Lock so only one goroutine at a time can access the map c.v.
 c.v[key]++
 c.mu.Unlock()
}

func (c *SafeCounter) Value(key string) int {
 c.mu.Lock()
 // Lock so only one goroutine at a time can access the map c.v.
 defer c.mu.Unlock()
 return c.v[key]
}
```

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSibvqvWjW3cz8LjK3uv9s8Qvt1VG1Mnnw4ldnv7rA3nAcpU0Or6YWUFQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSXdiamTOWItH3dU7B7gRe2cKzS2rfF957OGlar0bqujelia1ooMNxFHgQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzS3CIn3LibPlJ5MpFF9MiaRLCRFO182WgoQo2AV8u0ErGegr0cgKGydt3g/640?wx_fmt=png)

```
func foo() {
  mu.Lock()
  defer mu.Unlock() 


  // 如果不用defer 
  err := ...
  if err != nil{
      //log error
      return  //需要在这里Unlock()
  }
  
  return //需要在这里Unlock()
}
```

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSMNfQK8ySs7sr4OqDswRTCYHCzO3QKVwVpcr6JiaQewHIk9g6PCJiaXtA/640?wx_fmt=png)

```
func (t *Transport) CloseIdleConnections() {
 t.nextProtoOnce.Do(t.onceSetNextProtoDefaults)
 t.idleMu.Lock()
 m := t.idleConn
 t.idleConn = nil
 t.closeIdle = true // close newly idle connections
 t.idleLRU = connLRU{}
 t.idleMu.Unlock()
 for _, conns := range m {
  for _, pconn := range conns {
   pconn.close(errCloseIdleConns)
  }
 }
 if t2 := t.h2transport; t2 != nil {
  t2.CloseIdleConnections()
 }
}
```

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzSVK18omicpytwicFyQjmQVlJr12UII1phciaTIWmUU2m9mBiaWm6Ae7zdMQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzScnnax0lzjqFrexYcGyFxbPyXrRRuB4Zzk8f2K50CThSjVWciaacBexA/640?wx_fmt=png)

  
![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9IHqbOgDqW4Qy4nel34ydzS0Twtiapouibp9YRGiceGwcKJqYG0dutDAyDl2GYIy370ACbhl1AZJSW9A/640?wx_fmt=png)

  

  

Mutex 易错点
---------

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9Jh4Lfo0BkuMKBkRibfDiaibITdPibdCm73J1RLPdTFZGO387IuenG6ZxkudibGG4bHO179w3FQMxYKXBg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9Jh4Lfo0BkuMKBkRibfDiaibITfiaxcEX6iaahWNEnnJGqK84xEU4MoKRUdnoxZMvib7Ndq3bJ9Ht389ajw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9Jh4Lfo0BkuMKBkRibfDiaibITJG5dNMOWqMZczMppCiczRSb5AFkFiadYSUVHnic6ibWbV2G27e5tpHHhicg/640?wx_fmt=png)

### Panic

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9Jh4Lfo0BkuMKBkRibfDiaibITp1lMibuyMgu5Cb0Vj7BNtpPhA12RLPCNUz2FxxEQ8REkd3r64kXDGtw/640?wx_fmt=png)

```
func  misuse() {
 var mu sync.Mutex

 mu.Unlock() // fatal error: sync: unlock of unlocked mutex
}
```

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9Jh4Lfo0BkuMKBkRibfDiaibITNeeFJqZbYOtn8Hxx18ers6sCPybtML9bzZI5vNsJj0ajWAF0AJ90FQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9Jh4Lfo0BkuMKBkRibfDiaibITFr78qKysHibeHHOcU66eKfMPMEJcL4N5tEiaqMDqUtBT1JYvPVZdVibHw/640?wx_fmt=png)

### 死锁

死锁是指两个或两个以上的进程在执行过程中，由于竞争资源或者由于彼此通信而造成的一种阻塞的现象，若无外力作用，它们都将无法推进下去。此时称系统处于死锁状态或系统产生了死锁，这些永远在互相等待的进程称为死锁进程。

  

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9Jh4Lfo0BkuMKBkRibfDiaibITrADTW6vKsjdEmGpQb1uUIIKyHxhquiaRvXNIqf5BsDtfvejL8ibibWUTw/640?wx_fmt=png)  

  

```
type DataStore struct {
 sync.Mutex // ← this mutex protects the cache below
 cache map[string]string
}

func (ds *DataStore) get(key string) string {
 ds.Lock()
 defer ds.Unlock()
 
 if ds.count() > 0 { <-- count() also takes a lock!
  item := ds.cache[key]
  return item
 }
 return ""
}

func (ds *DataStore) count() int {
 ds.Lock()
 defer ds.Unlock()
 return len(ds.cache)
}
```

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9Jh4Lfo0BkuMKBkRibfDiaibITP67Gh0ZcX7cw6ibg3sfDKO024K9MVmMXf3lkicDSlDPiaH1MtwZSEWpjg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9Jh4Lfo0BkuMKBkRibfDiaibITCzcyAHWGmNXJ2nJPyn1dUeaNglwomic4YZlj1mEvvSv6V90u63hZr6Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9Jh4Lfo0BkuMKBkRibfDiaibIT6lD1YmJBGyicnYa8gAYEibROiape03HUycBWmOSBdKliaAT3QUwJ3yHoicg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9Jh4Lfo0BkuMKBkRibfDiaibITQSqbUU0AWhK2TknNtWibOTf0Y8ib2c79WiaP8SMLsQia5Wbu3QSU2YDcOA/640?wx_fmt=png)

### 锁的粒度

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9Jh4Lfo0BkuMKBkRibfDiaibIT3d2Pm8UjkKiciaicvLicxUZ4fl4u4KmgK8fkb5uU8pNIUYUtDYicIhkFmJQ/640?wx_fmt=png)

```
// Don't do the following if you can avoid it
func doSomething() {
    mu.Lock()
    item := cache["myKey"]
    http.Get() // Some expensive io call
    mu.Unlock()
}
```

![](https://mmbiz.qpic.cn/mmbiz_png/YdZzofiato9Jh4Lfo0BkuMKBkRibfDiaibITS2rOAmtlAEtdibj7icO9zEribAbWItticmaEXtulBe2U8bczLCvHSribEFw/640?wx_fmt=png)

```
// Instead do the following where possible
func doSomething(){
    mu.Lock()
    item := cache["myKey"]
    mu.Unlock()
    http.Get() // This can take awhile and it's okay!
}
```

- EOF -

1、[一举拿下网络 IO 模型](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666555756&idx=2&sn=05ccfd821112de68bea292ab498e43cd&chksm=80dcadc7b7ab24d114667c0fb823021062fee20003978ae5dc9497e8e456c402360d0254939b&scene=21#wechat_redirect)

2、[C++ 后台开发知识点及学习路线](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666555749&idx=2&sn=80292e2498dbaeea9c23e647a59064fd&chksm=80dcadceb7ab24d8495f7727368f8f9fde222e10f50fa4823701c3703dbeae525f9ae0c4ca66&scene=21#wechat_redirect)

3、[深入理解 Go scheduler 调度器：GPM 源码分析](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666555715&idx=2&sn=5bffdefcad91221ca7c9b5de1e6c7e1a&chksm=80dcade8b7ab24fed9a2638629f99583bb3ba208ea91b93913bd95ac3368cb7e8a368855d083&scene=21#wechat_redirect)
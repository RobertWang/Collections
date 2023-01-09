> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.debuginn.cn](https://www.debuginn.cn/6832.html)

> Go 语言是一种强类型、编译型的语言，在开发过程中，代码规范是尤为重要的，一个小小的失误可能会带来严重…

**友情提示：**此篇文章大约需要阅读 20 分钟 33 秒，不足之处请多指教，感谢您的阅读。 **[订阅本站](https://www.debuginn.cn/subscribe)**

Go 语言是一种强类型、编译型的语言，在开发过程中，代码规范是尤为重要的，一个小小的失误可能会带来严重的事故，拥有一个良好的 Go 语言开发习惯是尤为重要的，遵守开发规范便于维护、便于阅读理解和增加系统的健壮性。

以下是我们项目组开发规范加上自己开发遇到的问题及补充，希望对你有所帮助：  
注：我们将以下约束分为三个等级，分别是：**【强制】**、**【推荐】**、**【参考】**。

Go 编码相关
-------

**【强制】**代码风格规范遵循 go 官方标准：[CodeReviewComments](https://github.com/golang/go/wiki/CodeReviewComments)，请使用官方 golint [lint](https://github.com/golang/lint) 进行风格静态分析；

**【强制】**代码格式规范依照`gofmt`，请安装相关 IDE 插件，在保存代码或者编译时，自动将源码通过`gofmt`做格式化处理，保证团队代码格式一致（比如空格，递进等）

**【强制】**业务处理代码中不能开`goroutine`，此举会导致`goroutine`数量不可控，容易引起系统雪崩，如果需要启用`goroutine`做异步处理，请在初始化时启用固定数量`goroutine`，通过`channel`和业务处理代码交互，初始化`goroutine`的函数，原则上应该从`main`函数入口处明确的调用：

```
func crond() {
    defer func() {
      if err := recover(); err != nil {
        // dump stack & log
      }
    }()
    // do something
}
func main() {
    // init system
    go crond()
    go crond2()
    // handlers
 } 

```

**【强制】**异步开启`goroutine`的地方（如各种`cronder`)，需要在最顶层增加`recover()`，捕捉`panic`，避免个别`cronder`出错导致整体退出：

```
func globalCrond() {
   for _ := ticker.C {
      projectCrond()
      itemCrond()
      userCrond()
   }
}
func projectCrond() {
   defer func() {
      if err := recover(); err != nil {
         // 打日志，并预警
      }
   }
   // do 
}

```

**【强制】**当有并发读写`map`的操作，必须加上读写锁`RWMutex`，否则`go runtime`会因为并发读写报`panic`，或者使用`sync.Map`替代；

**【强制】**对于提供给外部使用的`package`，返回函数里必须带上`err`返回，并且保证在`err == nil`情况下，返回结果不为`nil`，比如：

```
resp, err := package1.GetUserInfo(xxxxx)
// 在err == nil 情况下，resp不能为nil或者空值

```

**【强制】**当操作有多个层级的结构体时，基于**防御性编程**的原则，需要对每个层级做空指针或者空数据判别，特别是在处理复杂的页面结构时，如：

```
type Section struct {
     Item   *SectionItem
     Height int64
     Width  int64
 }
 type SectionItem struct {
     Tag    string
     Icon   string
     ImageURL string
     ImageList []string
     Action *SectionAction
 }
 type SectionAction struct {
     Type  string
     Path  string
     Extra string
 }

func getSectionActionPath(section *Section) (path string, img string, err error) {
   if section.Item == nil || section.Item.Action == nil { // 要做好足够防御，避免因为空指针导致的panic
      err = fmt.Errorf("section item is invalid")
      return
   }

   path = section.Item.Action.Path

   img = section.Item.ImageURL
   // 对取数组的内容，也一定加上防御性判断
   if len(section.Item.ImageList) > 0 { 
      img = section.Item.ImageList[0]
   }
   return
}

```

**【推荐】**生命期在函数内的资源对象，如果函数逻辑较为复杂，建议使用`defer`进行回收：

```
func MakeProject() {
   conn := pool.Get()
   defer pool.Put(conn)

   // 业务逻辑
   ...
   return
}

```

对于生命期在函数内的对象，定义在函数内，将使用栈空间，减少`gc`压力：

```
func MakeProject() (project *Project){

   project := &Project{} // 使用堆空间
   var tempProject Project  // 使用栈空间

   return
}

```

**【强制】**不能在循环里加`defer`，特别是`defer`执行回收资源操作时。因为`defer`是函数结束时才能执行，并非循环结束时执行，某些情况下会导致资源（如连接资源）被大量占用而程序异常：

```
// 反例：
for {
   row, err := db.Query("SELECT ...")
   if err != nil {
      ...
   }
   defer row.Close() // 这个操作会导致循环里积攒许多临时资源无法释放
   ...
}

// 正确的处理，可以在循环结束时直接close资源，如果处理逻辑较复杂，可以打包成函数：
for {
   func () {
      row, err := db.Query("SELECT ...")
      if err != nil {
         ...
      }
      defer row.Close()
      ...
   }()
}

```

**【推荐】**对于可预见容量的`slice`或者`map`，在`make`初始化时，指定`cap`大小，可以大大降低内存损耗，如：

```
headList := make([]home.Sections, 0, len(srcHomeSection)/2) tailList := make([]home.Sections, 0, len(srcHomeSection)/2)
dstHomeSection = make([]*home.Sections, 0, len(srcHomeSection))
….
if appendToHead {
   headList = append(headList, info)
} else {
   tailList = append(tailList, info)
}
….
dstHomeSection = append(dstHomeSection, headList…)
dstHomeSection = append(dstHomeSection, tailList…)

```

**【推荐】**逻辑操作中涉及到频繁拼接字符串的代码，请使用`bytes.Buffer`替代。使用`string`进行拼接会导致每次拼接都新增`string`对象，增加`GC`负担：

```
// 正例：
var buf bytes.Buffer
for _, name := range userList {
   buf.WriteString(name)
   buf.WriteString(",")
}
return buf.String()

// 反例：
var result string
for _, name := range userList {
   result += name + ","
}
return result

```

**【强制】**对于固定的正则表达式，可以**在全局变量初始化时完成预编译**，可以有效加快匹配速度，不需要在每次函数请求中预编译：

```
var wordReg = regexp.MustCompile("[\\w]+")
func matchWord(word string) bool {
   return wordReg.MatchString(word)
}

```

**【推荐】**JSON 解析时，遇到不确定是什么结构的字段，建议使用`json.RawMessage`而不要用`interface`，这样可以根据业务场景，做二次`unmarshal`而且性能比`interface`快很多；

**【强制】**锁使用的粒度需要根据实际情况进行把控，如果变量只读，则无需加锁；读写，则使用读写锁`sync.RWMutex`；

**【强制】**使用随机数时 (`math/rand`)，必须要做随机初始化 (`rand.Seed`)，否则产生出的随机数是可预期的，在某些场合下会带来安全问题。一般情况下，使用`math/rand`可以满足业务需求，如果开发的是安全模块，建议使用`crypto/rand`，安全性更好；

**【推荐】**对性能要求很高的服务，或者对程序响应时间要求高的服务，应该避免开启大量`gouroutine`；  
说明：官方虽然号称`goroutine`是廉价的，可以大量开启`goroutine`，但是由于`goroutine`的调度并没有实现优先级控制，使得一些关键性的`goroutine`（如网络 / 磁盘 IO，控制全局资源的`goroutine`）没有及时得到调度而拖慢了整体服务的响应时间，因而在系统设计时，如果对性能要求很高，应避免开启大量`goroutine`。

打点规范
----

**【强制】**打点使用. 来做分隔符，打点名称需要包含业务名，模块，函数，函数处理分支等，参考如下：

```
// 业务名.服务名.模块.功能.方法
service.gateway.module.action.func

```

**【强制】**打点使用场景是监控系统的实时状态，不适合存储任何业务数据；

**【强制】**在打点个数太多时，展示时速度会变慢。建议单个服务打点的 **key** 不超过 10000 个，`key`中单个维度不同值不超过 1000 个（**千万不要用 user_id 来打点**)；

**【推荐】**如果展示的时候需要拿成百上千个`key`的数据通过 `Graphite` 的聚合函数做聚合，最后得到一个或几个 `key`。这种情况下可以在打点的时候就把这个要聚合的点聚合好，这样展示的时候只要拿这几个 key，对展示速度是巨大的提升。

日志相关
----

**【强制】**日志信息需带上下文，其中`logid`必须带上，同一个请求打的日志都需带上`logid`，这样可以根据`logid`查找该次请求相关的信息；

**【强制】**对`debug/notice/info` 级别的日志输出，必须使用条件输出或者使用占位符方式，避免使用字符拼接方式：

```
log.Debug("get home page failed %s, id %d", err, id)

```

**【强制】**如果是解析`json`出错的日志，需要将报错`err`及原内容一并输出，以方便核查原因；

**【推荐】**对`debug/notice/info`级别的日志，在打印日志时，默认不显示调用位置（如 / path/to/code.go:335）  
说明：`go`获取调用栈信息是比较耗时的操作 (`runtime.Caller`)，对于性能要求很高的服务，特别是大量调用的地方，应尽量避免开发人员在使用该功能时，需知悉这个调用带来的代价。

Redis 相关
--------

**【推荐】**统一使用**`:`**作为前缀后缀分隔符，这里可以根据 `Redis`中间件 `key proxy` 怎么解析分析 `Key` 进行自定义，便于基础服务的数据可视化及问题排查；

**【强制】**避免使用 `HMGET/HGETALL/HVALS/HKEYS/SMEMBERS` 阻塞命令这类命令在`value`较大时，对 Redis 的 CPU / 带宽消耗较高，容易导致响应过慢引发系统雪崩；

**【强制】**不可把 Redis 当成存储，如有统计相关的需求，可以考虑异步同步到数据库进行统计，Redis 应该回归缓存的本质；

**【推荐】**避免使用大 key，按经验超过 10k 的 value，可以压缩 (`gzip/snappy`等算法) 后存入内存，可以减少内存使用，其次降低网络消耗，提高响应速度：

```
value, err := c.RedisCache.GetGzip(key)
….
c.RedisCache.SetExGzip(content, 60)

```

**【推荐】**Redis 的分布式锁，可以使用:

```
lock: redis.Do("SET", lockKey, randint, "EX", expire, "NX")
unlock: redis.GetAndDel(lockKey, randint) // redis暂不支持，可以用lua脚本

```

**【推荐】**尽量避免在逻辑循环代码中调用 Redis，会产生流量放大效应，请求量较大时需采用其他方法优化（比如静态配置文件）；

**【推荐】**`key` 尽量离散读写，通过`uid/imei/xid`等跟用户 / 请求相关的后缀分摊到不同分片，避免分片负载不均衡；

**【参考】**当缓存量大，请求量较高，可能超出 Redis 承受范围时，可充分利用本地缓存`(localcache)+redis`缓存的组合方案来缓解压力，削减峰值：

使用这个方法需要具备这几个条件：

*   cache 内容与用户无关，key 状态不多，属于公共信息；
*   该 cache 内容时效性较高，但是访问量较大，有峰值流量。

```
key := "demoid:3344"
value := localcacche.Get(key)
if value == "" {
   value = rediscache.Get(key)
   if value != "" {
      // 随机缓存 1~5s，各个机器间错开峰值，只要比 redis缓存短即可
      localcache.SetEx(key, value, rand.Int63n(5)+1)
   }
}
if value == "" {
   ....
   // 从其他系统或者数据库获取数据
   appoint.GetValue()

   // 同时设置到redis及localcache中
   rediscache.SetEx(key, content, 60)
   localcache.SetEx(key, content, rand.Int63n(5)+1)
}

```

**【参考】**对于请求量高，实时性也高的内容，如果纯粹使用缓存，当缓存失效瞬间，会导致大量请求穿透到后端服务，导致后端服务有雪崩危险：

如何兼顾扛峰值，保护后端系统，同时也能保持实时性呢？在这种场景下，可以采用随机更新法更新数据，方法如下：

1.  正常请求从缓存中读取，缓存失效则从后端服务获取；
2.  在请求中根据随机概率 1%（或者根据实际业务场景设置比率）会跳过读取缓存操作，直接从后端服务获取数据，并更新缓存。

这种做法能保证最低时效性，并且当访问量越大，更新概率越高，使得内容实时性也越高。

如果结合上一条 `localcache+rediscache` 做一二级缓存，则可以达到扛峰值同时保持实时性。

数据库相关
-----

**【强制】**操作数据库 sql 必须使用 stmt 格式，使用占位符替代参数，禁止拼接 sql；

**【强制】**SQL 语句查询时，不得使用 SELECT * （即形如 SELECT * FROM tbl WHERE），必须明确的给出要查询的列名，避免表新增字段后报错；

**【强制】**对于线上业务 SQL，需保证命中索引，索引设计基于业务需求及字段区分度，一般可区分状态不高的字段（如 status 等只有几个状态），不建议加到索引中；

**【强制】**在成熟的语言中，有实体类，数据访问层 (`repository / dao`) 和业务逻辑层 (`service`)；在我们的规范中存储实体`struct`放置于`entities`包下；

**【强制】**对于联合索引，需将区分度较大的字段放前面，区分度小放后面，查找时可以减少被检索数据量；

```
-- 字段区分度 item_id > project_id
alter table xxx add index idx_item_project (item_id, project_id)

```

**【强制】**所有数据库表必须有主键 id；

**【强制】**主键索引名为 pk 字段名; 唯一索引名为 uk 字段名; 普通索引名则为 idx_字段名；

**【强制】**防止因字段类型不同造成的隐式转换，导致索引失效，造成全表扫描问题；

**【强制】**业务上有唯一特性的字段，即使是多字段的组合，也必须建成唯一索引；

**【强制】**一般事务标准操作流程：

```
func TestTXExample(t *testing.T) {
   // 打开事务
   tx, err := db.Beginx()
   if err != nil {
      log.Fatal("%v", err)
      return
   }

   // defer异常
   needRollback := true
   defer func() {
      if r := recover(); r != nil {  // 处理recover，避免因为panic，资源无法释放
         log.Fatal("%v", r)
         needRollback = true
      }
      if needRollback {
         xlog.Cause("test.example.transaction.rollback").Fatal()
         tx.Rollback()
      }
   }()

   // 事务的逻辑
   err = InsertLog(tx, GenTestData(1)[0])
   if err != nil {
      log.Fatal("%v", err)
      return
   }

   // 提交事务
   err = tx.Commit()
   if err != nil {
      log.Fatal("%v", err)
      return
   }
   needRollback = false

   return
}

```

**【强制】**执行事务操作时，请确保`SELECT ... FOR UPDATE`条件命中索引，使用行锁，避免一个事务锁全表的情况;

**【强制】**禁止超过三个表的`join`，需要`join`的字段，数据类型必须一致，多表关联查询时，保证被关联的字段有索引;

**【强制】**数据库`max_open`连接数不可设置过高，会导致代理连接数打满导致不可用状况;

![](https://www.debuginn.cn/wp-content/uploads/2021/07/wechat_qr.png)
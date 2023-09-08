> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/a77d3EtwBqZUvErvrisqWg)

> !! 大家好，我是蓝胖子，关于 sql 监控的需求有很多，比如我们常常需要在 sql 执行前后加上埋点来对 sql 执行

> !! 大家好，我是蓝胖子，关于 sql 监控的需求有很多，比如我们常常需要在 sql 执行前后加上埋点来对 sql 执行时长，执行语句进行记录，今天我们就来看看在 golang 中如何实现 sql 的埋点记录。

首先，我们来看下，在 golang 中如何实现数据库查询。

golang 实现数据库查询
--------------

golang 的 database/sql 包下封装了对数据库查询的接口方法，真正实现数据库连接以及查询的逻辑是由第三方库实现的，拿 mysql 举例，这个库是 github.com/go-sql-driver/mysql。

通常我们执行一个 sql 语句会先创建一个 sql.DB 对象，代码如下,

```
db, err = sql.Open(driverName, dataSourceName)


```

sql.Open 方法要求传入驱动名称和数据库连接字符串，数据库驱动是由第三方库注册到全局变量里的，如下:

```
// /usr/local/go/src/database/sql/sql.go:35
var (  
   driversMu sync.RWMutex  
   drivers   = make(map[string]driver.Driver)  
)

/// goproject/pkg/mod/github.com/go-sql-driver/mysql@v1.7.1/driver.go:83
func init() {  
   sql.Register("mysql", &MySQLDriver{})  
}


```

在执行 sql 时，一般是通过调用 sql.DB 对象的 Query 或者 Exec 方法，

```
func (db *DB) Query(query string, args ...any) (*Rows, error) 
func (db *DB) Exec(query string, args ...any) (Result, error) 


```

下面我们就来着重的分析下这两个方法内部是如何执行 sql 的。

> !! 需要注意的是，在分析过程中一定要分清楚哪些是 go 官方包 database/sql 下的逻辑，哪些是第三方库的逻辑，这样可以很好的理解 go database/sql 包是如何规范 sql 查询的。

接口分析
----

db.Query 方法最终会走到 db.queryDc 方法，db.Exec 最终会走到 db.execDc 方法，两个方法执行逻辑都是比较类似的，如下所示:

![](https://mmbiz.qpic.cn/sz_mmbiz_png/81178hAUFMibWWFN8TBWDMclryPuR2e4uicRTB9AW6sVqhPG6TWOy56GjibKUp2YOLAFEXmeeFHgfOEjkNMtOwbCg/640?wx_fmt=png)image.png

它们首先都会判断能不能直接执行 sql 语句，如果不能的话就要使用占位符的方式，prepare 和 statement 的方式来执行 sql，返回 dirver.ErrSkip 错误则代表前者的执行逻辑走不通，需要跳过，接着执行下面的逻辑。

无论是直接执行 sql 还是使用 prepare 和 statement 的方式执行 sql，其内部最终都会调用到第三方库封装好的操作数据库的方法。

拿 query 逻辑，github.com/go-sql-driver/mysql 库举例，database/sql 包下用到的连接接口类型实际上是 github.com/go-sql-driver/mysql 第三方库返回的连接结构体，所以如果第三方库实现的连接实现类型实现了相应的 driver.Queryer 或者 driver.QueryerContext 接口，那么查询时则会走到直接执行 sql 的方法 ctxDriverQuery 里。

所以对于第三方库的连接类型实现而言，driver.Queryer 接口并不是必需的，database/sql 对于连接类型的定义接口只要包含如下几个方法就可以了:

```
type Conn interface {  
 Prepare(query string) (Stmt, error) 
 Close() error  
 Begin() (Tx, error)  
}


```

不过 github.com/go-sql-driver/mysql 依然为连接类型实现了 driver.Queryer 接口方法，在这个方法里，由它自己去判断是否能够采取直接执行 sql 的方式。

直接执行 sql 相比 prepare statement 方式少一次网络传输，效率更高, 但会带来 sql 注入问题。

所以 github.com/go-sql-driver/mysql 实现 driver.Queryer 接口时，判断了 如果写的 sql 有传参数，比如像这样 db.Query("select * from t_user where id = ?;",1)，需要将连接配置参数 InterpolateParams 设置为 true 才能采取直接执行 sql 的方式，否则会返回 dirver.ErrSkip ，让 databse/sql 包去执行 prepare statement 的逻辑。

> !! 连接配置参数 InterpolateParams 设置为 true ，github.com/go-sql-driver/mysql 会对 sql 语句执行转义，避免 sql 注入。

看到这里，应该明白了 golang 中执行 sql 的逻辑，我们的最终目的是对 sql 执行前后能够加上一些自己的埋点日志。所以接下来，还要看看这部分应该如何来做。

自定义驱动实现 sql 埋点统计
----------------

由于 database/sql 包下仅仅是定义了一个连接类型的接口，连接类型的真正实现是由第三方库实现的，所以我们可以采取装饰器模式，对 github.com/go-sql-driver/mysql  库产生的连接类型进行重新包装，覆盖原有的查询方法，就可以在查询前后加上自己的埋点逻辑了。

连接的创建是由驱动产生的，database/sql 包下同样也只定义了一个驱动接口, 实现是第三方库实现的。接口如下:

```
type Driver interface {  
    Open(name string) (Conn, error)  
}


```

并且 database/sql 包下还有一个全局变量 drivers 和一个注册方法，第三方库可以通过这个注册方法把自己实现的驱动注册进去，后续 database/sql 包下创建连接时，就是通过驱动名创建一个 sql.DB 对象，sql.DB 对象内部会用对应的驱动实现创建连接。

```
// /usr/local/go/src/database/sql/sql.go:35
var (  
   driversMu sync.RWMutex  
   drivers   = make(map[string]driver.Driver)  
)
/// goproject/pkg/mod/github.com/go-sql-driver/mysql@v1.7.1/driver.go:83
func init() {  
   sql.Register("mysql", &MySQLDriver{})  
}
// 用户代码
db, err = sql.Open(driverName, dataSourceName)


```

所以我们要实现自定义的连接类型来包装 github.com/go-sql-driver/mysql 库下的连接类型，首先是包装它的驱动类型。如下，我们可以对原有连接和驱动加上钩子函数的属性。

```
type Hooks interface {  
   Before(ctx context.Context, query string, args ...interface{}) (context.Context, error)  
   After(ctx context.Context, query string, args ...interface{}) (context.Context, error)  
}

type Driver struct {  
   driver.Driver  
   hooks Hooks  
}  
  
func (drv *Driver) Open(name string) (driver.Conn, error) {  
   conn, err := drv.Driver.Open(name)  
   if err != nil {  
      return conn, err  
   }  
   wrapped := &Conn{conn, drv.hooks}  
   return wrapped, nil  
}  
  
Conn struct {  
   driver.Conn  
   hooks Hooks  
}


```

对于新的链接类型只要对它的查询方法进行覆盖就能埋点了，拿 query 接口举例，在原有的 queryContext 方法前后加上自己的逻辑。

```
func (conn *Conn) QueryContext(ctx context.Context, query string, args []driver.NamedValue) (driver.Rows, error) {  
   var err error  
   list := namedToInterface(args)  
   // Query `Before` Hooks  
   if ctx, err = conn.hooks.Before(ctx, query, list...); err != nil {  
      return nil, err  
   }  
   results, err := conn.queryContext(ctx, query, args)  
   if err != nil {  
      return results, handlerErr(ctx, conn.hooks, err, query, list...)  
   }  
   if _, err := conn.hooks.After(ctx, query, list...); err != nil {  
      return nil, err  
   }  
   return results, err  
}



```

注意，因为用连接查询的地方涉及到好几个接口方法，我们都需要覆盖到这些方法，才能让埋点不会被漏掉。

这些地方分别是 queryDC 中执行执行 sql 时的 QueryContext 接口，execDc 中执行 sql 时的 ExecContext 接口，连接的 PrepareContext 接口，以及 driver.Stmt 的 ExecContext 和 QueryContext 接口。

完整的实现钩子函数的代码已经上传到 github

```
https://github.com/HobbyBear/easymonitor/blob/main/infra/sqlhooks.go


```
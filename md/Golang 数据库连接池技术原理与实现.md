> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/cZMvKB5Wu-pdOSZsizpulA)

> 一、为什么需要连接池如果不用连接池，而是每次请求都创建一个连接是比较昂贵的，因此需要完成 3 次 tcp 握手。同时

**一、为什么需要连接池**  

如果不用连接池，而是每次请求都创建一个连接是比较昂贵的，因此需要完成 3 次 tcp 握手。同时在高并发场景下，由于没有连接池的最大连接数限制，可以创建无数个连接，耗尽文件描述符。连接池就是为了复用些创建好的连接。

**二**、连接池设计****

基本上连接池都会设计以下几个参数：

**初始连接数**：在初始化连接池时就会预先创建好的连接数量，如果设置得：

*   过大：可能造成浪费
    
*   过小：请求到来时需要新建连接
    

**最大空闲连接数 maxIdle**：池中最大缓存的连接个数，如果设置得：

*   过大：造成浪费，自己不用还把持着连接。因为数据库整体的连接数是有限的，当前进程占用多了，其他进程能获取的就少了
    
*   过小：无法应对突发流量
    

**最大连接数 maxCap**：

*   如果已经用了 maxCap 个连接，要申请第 maxCap+1 个连接时，一般会阻塞在那里，直到超时或者别人归还一个连接
    

**最大空闲时间 idleTimeout**：当发现某连接空闲超过这个时间时，会将其关闭，重新去获取连接

*   避免连接长时间没用，自动失效的问题
    

连接池对外提供两个方法，Get：获取一个连接，Put：归还一个连接。大部分连接池的实现大同小异，基本流程如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/caBJ5fWjGktibgJMWGEGXYK2J35NzQcZpS0HRqGbyicGbCewFjXjgiaM7rm1NXntUXGcst9DZO7fBYw6biaUL99Cew/640?wx_fmt=png)

********三、Golang 标准库 SQL 连接池********

Golang 的连接池实现在标准库 database/sql/sql.go 下。当我们运行：

```
db, err := sql.Open("mysql", "xxxx")

```

就会打开一个连接池。我们可以看看返回的 db 的结构体：

```
type DB struct {
    // Atomic access only. At top of struct to prevent mis-alignment
    // on 32-bit platforms. Of type time.Duration.
    waitDuration int64 // 等待新连接的总时间，用于统计
    connector driver.Connector // 由数据库驱动实现的连接器
    // numClosed is an atomic counter which represents a total number of
    // closed connections. Stmt.openStmt checks it before cleaning closed
    // connections in Stmt.css.
    numClosed uint64 // 关闭的连接数
    mu           sync.Mutex // 锁
    freeConn     []*driverConn // 可用连接池
    connRequests map[uint64]chan connRequest // 连接请求表，key 是分配的自增键
    nextRequest  uint64 // 连接请求的自增键
    numOpen      int    // 已经打开 + 即将打开的连接数
    // Used to signal the need for new connections
    // a goroutine running connectionOpener() reads on this chan and
    // maybeOpenNewConnections sends on the chan (one send per needed connection)
    // It is closed during db.Close(). The close tells the connectionOpener
    // goroutine to exit.
    openerCh          chan struct{} // 告知 connectionOpener 需要新的连接
    resetterCh        chan *driverConn // connectionResetter 函数，连接放回连接池的时候会用到
    closed            bool
    dep               map[finalCloser]depSet
    lastPut           map[*driverConn]string // debug 时使用，记录上一个放回的连接
    maxIdle           int                    // 连接池大小，默认大小为 2，<= 0 时不使用连接池
    maxOpen           int                    // 最大打开的连接数，<= 0 不限制
    maxLifetime       time.Duration          // 一个连接可以被重用的最大时限，也就是它在连接池中的最大存活时间，0 表示可以一直重用
    cleanerCh         chan struct{} // 告知 connectionCleaner 清理连接
    waitCount         int64 // 等待的连接总数
    maxIdleClosed     int64 // 释放连接时，因为连接池已满而被关闭的连接总数
    maxLifetimeClosed int64 // 因为超过存活时间而被关闭的连接总数
    stop func() // stop cancels the connection opener and the session resetter.
}

```

我们可以看的，DB 这个连接池内部存储连接的结构 freeConn，并不是我们之前使用的 chan，而是 []*driverConn，一个连接切片。

```
// driverConn wraps a driver.Conn with a mutex, to
// be held during all calls into the Conn. (including any calls onto
// interfaces returned via that Conn, such as calls on Tx, Stmt,
// Result, Rows)
type driverConn struct {
    db        *DB // 数据库句柄
    createdAt time.Time
    sync.Mutex  // 锁
    ci          driver.Conn // 对应具体的连接
    closed      bool // 是否标记关闭
    finalClosed bool // 是否最终关闭
    openStmt    map[*driverStmt]bool // 在这个连接上打开的状态
    lastErr     error // connectionResetter 的返回结果
    // guarded by db.mu
    inUse      bool // 连接是否占用
    onPut      []func() // 连接归还时要运行的函数，在 noteUnusedDriverStatement 添加
    dbmuClosed bool     // 和 closed 状态一致，但是由锁保护，用于 removeClosedStmtLocked
}

```

继续查看代码，通过 query 方法一路往回找，我们可以看到这个函数：func(db*DB)conn(ctx context.Context,strategy connReuseStrategy)(*driverConn,error)。

**3.1** **获取连接**
================

```
// conn returns a newly-opened or cached *driverConn.
func (db *DB) conn(ctx context.Context, strategy connReuseStrategy) (*driverConn, error) {
    // 先判断db是否已经关闭。
  db.mu.Lock()
  if db.closed {
    db.mu.Unlock()
    return nil, errDBClosed
  }
    // 注意检测context是否已经被超时等原因被取消。
    select {
    default:
    case <-ctx.Done():
        db.mu.Unlock()
        return nil, ctx.Err()
    }
    lifetime := db.maxLifetime
    // 这边如果在freeConn这个切片有空闲连接的话，就left pop一个出列。注意的是，这边因为是切片操作，所以需要前面需要加锁且获取后进行解锁操作。同时判断返回的连接是否已经过期。
    numFree := len(db.freeConn)
    if strategy == cachedOrNewConn && numFree > 0 {
        conn := db.freeConn[0]
        copy(db.freeConn, db.freeConn[1:])
        db.freeConn = db.freeConn[:numFree-1]
        conn.inUse = true
        db.mu.Unlock()
        if conn.expired(lifetime) {
            conn.Close()
            return nil, driver.ErrBadConn
        }
        // Lock around reading lastErr to ensure the session resetter finished.
        conn.Lock()
        err := conn.lastErr
        conn.Unlock()
        if err == driver.ErrBadConn {
            conn.Close()
            return nil, driver.ErrBadConn
        }
        return conn, nil
    }
    // 这边就是等候获取连接的重点了。当空闲的连接为空的时候，这边将会新建一个request（的等待连接 的请求）并且开始等待
    if db.maxOpen > 0 && db.numOpen >= db.maxOpen {
        // 下面的动作相当于往connRequests这个map插入自己的号码牌。
        // 插入号码牌之后这边就不需要阻塞等待继续往下走逻辑。
        req := make(chan connRequest, 1)
        reqKey := db.nextRequestKeyLocked()
        db.connRequests[reqKey] = req
        db.waitCount++
        db.mu.Unlock()
        waitStart := time.Now()
        // Timeout the connection request with the context.
        select {
        case <-ctx.Done():
            // context取消操作的时候，记得从connRequests这个map取走自己的号码牌。
            db.mu.Lock()
            delete(db.connRequests, reqKey)
            db.mu.Unlock()
            atomic.AddInt64(&db.waitDuration, int64(time.Since(waitStart)))
            select {
            default:
            case ret, ok := <-req:
                // 这边值得注意了，因为现在已经被context取消了。但是刚刚放了自己的号码牌进去排队里面。意思是说不定已经发了连接了，所以得注意归还！
                if ok && ret.conn != nil {
                    db.putConn(ret.conn, ret.err, false)
                }
            }
            return nil, ctx.Err()
        case ret, ok := <-req:
            // 下面是已经获得连接后的操作了。检测一下获得连接的状况。因为有可能已经过期了等等。
            atomic.AddInt64(&db.waitDuration, int64(time.Since(waitStart)))
            if !ok {
                return nil, errDBClosed
            }
            if ret.err == nil && ret.conn.expired(lifetime) {
                ret.conn.Close()
                return nil, driver.ErrBadConn
            }
            if ret.conn == nil {
                return nil, ret.err
            }
            ret.conn.Lock()
            err := ret.conn.lastErr
            ret.conn.Unlock()
            if err == driver.ErrBadConn {
                ret.conn.Close()
                return nil, driver.ErrBadConn
            }
            return ret.conn, ret.err
        }
    }
    // 下面就是如果上面说的限制情况不存在，可以创建先连接时候，要做的创建连接操作了。
    db.numOpen++ // optimistically
    db.mu.Unlock()
    ci, err := db.connector.Connect(ctx)
    if err != nil {
        db.mu.Lock()
        db.numOpen-- // correct for earlier optimism
        db.maybeOpenNewConnections()
        db.mu.Unlock()
        return nil, err
    }
    db.mu.Lock()
    dc := &driverConn{
        db:        db,
        createdAt: nowFunc(),
        ci:        ci,
        inUse:     true,
    }
    db.addDepLocked(dc, dc)
    db.mu.Unlock()
    return dc, nil
}

```

**3.2** **释放连接**
================

```
func (db *DB) putConnDBLocked(dc *driverConn, err error) bool {
  if db.closed {
    return false
  }
  if db.maxOpen > 0 && db.numOpen > db.maxOpen {
    return false
  }
  // 这边是重点了，基本来说就是从connRequest这个map里面随机抽一个
  // 在排队等着的请求。取出来后发给他。就不用归还池子了。
  if c := len(db.connRequests); c > 0 {
    var req chan connRequest
    var reqKey uint64
    for reqKey, req = range db.connRequests {
      break
    }
    delete(db.connRequests, reqKey) // Remove from pending requests.
    if err == nil {
      dc.inUse = true
    }
    // 把连接给这个正在排队的连接
    req <- connRequest{
      conn: dc,
      err:  err,
    }
    return true
  } else if err == nil && !db.closed {
   // 既然没人排队，就看看到了最大连接数目没有。没到就归还给freeConn
    if db.maxIdleConnsLocked() > len(db.freeConn) {
      db.freeConn = append(db.freeConn, dc)
      db.startCleanerLocked()
      return true
    }
    db.maxIdleClosed++
  }
  return false
}

```

****四、总结****

**资源重用：**数据库连接得到重用，避免了频繁创建、释放连接引起的大量性能开销。

**更快的系统响应速度：**数据库连接池在初始化过程中，往往已经创建了若干数据库连接置于池中备用。 对于业务请求处理而言，直接利用现有可用连接，避免了数据库连接初始化和释放过程的时间开销，从而缩减了系统整体响应时间。

**新的资源分配手段：**对于多应用共享同一数据库的系统而言，可在应用层通过数据库连接的配置，实现数据库连接池技术。设置某一应用最大可用数据库连接数的限制，避免某一应用独占所有数据库资源。

**统一的连接管理，避免数据库连接泄漏：**在较为完备的数据库连接池实现中，可根据预先的连接占用超时设定，强制收回被占用连接。从而避免了常规数据库连接操作中可能出现的资源泄漏。
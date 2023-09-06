> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/WgeTzhP4T0AAjFssHz36UA)

> 了解 record batch、chunked array 和 table

经过对前面两篇文章[《Arrow 数据类型》](http://mp.weixin.qq.com/s?__biz=MzIyNzM0MDk0Mg==&mid=2247494953&idx=1&sn=0e2684021be91f7d2d4b3a246796ff3c&chksm=e8600ac8df1783deb4b9e895d4567bea3bcdc93a9782397be40b8b265acf4bc6aef0d4963e7d&scene=21#wechat_redirect)[1] 和[《Arrow Go 实现的内存管理》](http://mp.weixin.qq.com/s?__biz=MzIyNzM0MDk0Mg==&mid=2247494977&idx=1&sn=e38937245c4a80ec210aada07abaebc0&chksm=e8600aa0df1783b661723e7c0c2ec315fa143aecc62c11e71ab4e2700f7f1a76048045f272f8&scene=21#wechat_redirect)[2] 的学习，我们知道了各种 Arrow array type 以及它们在内存中的 layout，我们了解了 Go arrow 实现在内存管理上的一些机制和使用原则。

Arrow 的 array type 只是一个定长的、同类型的值序列。在实际应用中，array type 更多时候只是充当基础类型，**我们需要具有组合基础类型能力的更高级的数据结构**。在这一篇文章中，我们就来看看 Arrow 规范以及一些实现中提供的高级数据结构，包括 Record Batch、Chunked Array 以及 Table。

我们先来看看 Record Batch[3]。

1. Record Batch
---------------

Record 这个名字让我想起了 [Pascal 编程语言](https://en.wikipedia.org/wiki/Pascal_(programming_language "Pascal 编程语言")) 中的 Record。在 Pascal 中，Record 的角色大致与 Go 中的 Struct 类似，也是一组异构字段的集合。下面是《In-Memory Analytics with Apache Arrow》[4] 书中的一个 Record 例子：

```
// 以Go语言呈现
type Archer struct {
 archer string
 location string
 year int16
}


```

Record Batch 则顾名思义，是**一批 Record**，即一个 Record 的集合：[N]Archer。

如果将 Record 的各个字段作为列，将集合中的每个 Record 作为行，我们能得到如下面示意图中的结构：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/cH6WzfQ94mZoyFp6eSQkY7T3SEgPvCFibUfplcGVfFficiaaMD1RjGibrJAWkszCLOpZ2LJrmDROudf2bE7qm01ibtQ/640?wx_fmt=png)

Go Arrow 实现中没有直接使用 “Record Batch” 这个名字，而是使用了 “Record”，这个“Record” 实际代表的就是 Record Batch。下面是 Go Arrow 实现定义的 Record 接口：

```
// github.com/apache/arrow/go/arrow/record.go

// Record is a collection of equal-length arrays matching a particular Schema.
// Also known as a RecordBatch in the spec and in some implementations.
//
// It is also possible to construct a Table from a collection of Records that
// all have the same schema.
type Record interface {
    json.Marshaler

    Release()
    Retain()

    Schema() *Schema

    NumRows() int64
    NumCols() int64

    Columns() []Array
    Column(i int) Array
    ColumnName(i int) string
    SetColumn(i int, col Array) (Record, error)

    // NewSlice constructs a zero-copy slice of the record with the indicated
    // indices i and j, corresponding to array[i:j].
    // The returned record must be Release()'d after use.
    //
    // NewSlice panics if the slice is outside the valid range of the record array.
    // NewSlice panics if j < i.
    NewSlice(i, j int64) Record
}


```

我们依然可以使用 Builder 模式来创建一个 arrow.Record，下面我们就来用 Go 代码创建 [N]Archer 这个 Record Batch：

```
// record_batch.go
func main() {
    schema := arrow.NewSchema(
        []arrow.Field{
            {Name: "archer", Type: arrow.BinaryTypes.String},
            {Name: "location", Type: arrow.BinaryTypes.String},
            {Name: "year", Type: arrow.PrimitiveTypes.Int16},
        },
        nil,
    )

    rb := array.NewRecordBuilder(memory.DefaultAllocator, schema)
    defer rb.Release()

    rb.Field(0).(*array.StringBuilder).AppendValues([]string{"tony", "amy", "jim"}, nil)
    rb.Field(1).(*array.StringBuilder).AppendValues([]string{"beijing", "shanghai", "chengdu"}, nil)
    rb.Field(2).(*array.Int16Builder).AppendValues([]int16{1992, 1993, 1994}, nil)

    rec := rb.NewRecord()
    defer rec.Release()

    fmt.Println(rec)
}


```

运行上述示例，输出如下：

```
$go run record_batch.go 
record:
  schema:
  fields: 3
    - archer: type=utf8
    - location: type=utf8
    - year: type=int16
  rows: 3
  col[0][archer]: ["tony" "amy" "jim"]
  col[1][location]: ["beijing" "shanghai" "chengdu"]
  col[2][year]: [1992 1993 1994]


```

在这个示例里，我们看到了一个名为 Schema 的概念，并且 NewRecordBuilder 创建时需要传入一个 arrow.Schema 的实例。和数据库表 Schema 类似，Arrow 中的 Schema 也是一个元数据概念，它包含一系列作为 “列” 的字段的名称和类型信息。Schema 不仅在 Record Batch 中使用，在后面的 Table 中，Schema 也是必要元素。

arrow.Record 可以通过 NewSlice 可以 ZeroCopy 方式共享 Record Batch 的内存数据，NewSlice 会创建一个新的 Record Batch，这个 Record Batch 中的 Record 与原 Record 是共享的：

```
// record_batch_slice.go

sl := rec.NewSlice(0, 2)
fmt.Println(sl)
cols := sl.Columns()
a1 := cols[0]
fmt.Println(a1)


```

新的 sl 取了 rec 的前两个 record，输出 sl 得到如下结果：

```
record:
  schema:
  fields: 3
    - archer: type=utf8
    - location: type=utf8
    - year: type=int16
  rows: 2
  col[0][archer]: ["tony" "amy"]
  col[1][location]: ["beijing" "shanghai"]
  col[2][year]: [1992 1993]

["tony" "amy"]


```

相同 schema 的 record batch 可以合并，我们只需要分配一个更大的 Record Batch，然后将两个待合并的 Record batch copy 到新 Record Batch 中就可以了，但显然这样做的开销很大。

Arrow 的一些实现中提供了 Chunked Array 的概念，可以更低开销的来完成某个列的 array 的追加。

> 注：Chunked array 并不是 Arrow Columnar Format 的一部分。

2. Chunked Array
----------------

如果说 Record Batch 本质上是不同 Array type 的横向聚合，那么 Chunked Array 就是**相同 Array type 的纵向聚合**了，用 Go 语法表示就是：[N]Array 或 []Array，即 array of array。下面是一个 Chunked Array 的结构示意图：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/cH6WzfQ94mZoyFp6eSQkY7T3SEgPvCFibG8mFvgg0KzuvapK101ySympooNcnLaphPx09Ue9UGK5xjPgpIKQiaKQ/640?wx_fmt=png)

我们看到：Go 的 Chunked array 的实现使用的是一个 Array 切片：

```
// github.com/apache/arrow/go/arrow/table.go

// Chunked manages a collection of primitives arrays as one logical large array.
type Chunked struct {
    refCount int64 // refCount must be first in the struct for 64 bit alignment and sync/atomic (https://github.com/golang/go/issues/37262)

    chunks []Array

    length int
    nulls  int
    dtype  DataType
}


```

按照 Go 切片的本质，Chunked Array 中的各个元素 Array 间的实际内存 buffer 并不连续。并且正如示意图所示：每个 Array 的长度也并非是一样的。

> 注：在《Go 语言第一课》[5] 中的第 15 讲中有关于切片本质的深入系统的讲解。

我们可以使用 arrow 包提供的 NewChunked 函数创建一个 Chunked Array，具体见下面源码：

```
// chunked_array.go

func main() {
    ib := array.NewInt64Builder(memory.DefaultAllocator)
    defer ib.Release()

    ib.AppendValues([]int64{1, 2, 3, 4, 5}, nil)
    i1 := ib.NewInt64Array()
    defer i1.Release()

    ib.AppendValues([]int64{6, 7}, nil)
    i2 := ib.NewInt64Array()
    defer i2.Release()
    
    ib.AppendValues([]int64{8, 9, 10}, nil)
    i3 := ib.NewInt64Array()
    defer i3.Release()

    c := arrow.NewChunked(
        arrow.PrimitiveTypes.Int64,
        []arrow.Array{i1, i2, i3},
    )
    defer c.Release()

    for _, arr := range c.Chunks() {
        fmt.Println(arr)
    }
    
    fmt.Println("chunked length =", c.Len())
    fmt.Println("chunked null count=", c.NullN())
} 


```

我们看到在 Chunked Array 聚合了多个 arrow.Array 实例，并且这些 arrow.Array 实例的长短可不一致，arrow.Chunked 的 Len() 返回的则是 Chunked 中 Array 的长度之和。下面是示例程序的输出结果：

```
$go run chunked_array.go 
[1 2 3 4 5]
[6 7]
[8 9 10]
chunked length = 10
chunked null count= 0


```

这样来看，Chunked Array 可以看成一个逻辑上的大 Array。

好了，问题来了！Record Batch 是用来聚合等长 array type 的，那么**是否有某种数据结构可以用来聚合等长的 Chunked Array 呢**？答案是有的！下面我们就来看看这种结构：Table。

3. Table
--------

Table 和 Chunked Array 一样并不属于 Arrow Columnar Format 的一部分，最初只是 Arrow 的 C++ 实现中的一个数据结构，Go Arrow 的实现也提供了对 Table 的支持。

Table 的结构示意图如下 (图摘自《In-Memory Analytics with Apache Arrow》[6] 一书)：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/cH6WzfQ94mZoyFp6eSQkY7T3SEgPvCFib6bnwkOtVeRbo2GC1xEVsiaaPMlCic9pV5FTkxHPiauwsenyxMuAz7dHxw/640?wx_fmt=png)

我们看到：和 Record Batch 的每列是一个 array 不同，Table 的每一列为一个 chunked array，所有列的 chunked array 的 Length 是相同的，但各个列的 chunked array 中的 array 的长度可以不同。

Table 和 Record Batch 相似的地方是都有自己的 Schema。

下面的示意图 (来自这里 [7]) 对 Table 和 Chunked Array 做了十分直观的对比：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/cH6WzfQ94mZoyFp6eSQkY7T3SEgPvCFiboalXsxx6ulGJy0ibbj3PWTb58X3bZPX4aMmZOFfH3T8RbvMSSmoxBiaA/640?wx_fmt=png)

Record Batch 是 Arrow Columnar format 中的一部分，所有语言的实现都支持 Record Batch；但 Table 并非 format spec 的一部分，并非所有语言的实现对其都提供支持。

另外从图中看到，由于 Table 采用了 Chunked Array 作为列，chunked array 下的各个 array 内部分布并不连续，这让 Table 在运行时丧失了一些局部性。

下面我们就使用 Go arrow 实现来创建一个 table，这是一个 3 列、10 行的 table：

```
// table.go

func main() {
 schema := arrow.NewSchema(
  []arrow.Field{
   {Name: "col1", Type: arrow.PrimitiveTypes.Int32},
   {Name: "col2", Type: arrow.PrimitiveTypes.Float64},
   {Name: "col3", Type: arrow.BinaryTypes.String},
  },
  nil,
 )

 col1 := func() *arrow.Column {
  chunk := func() *arrow.Chunked {
   ib := array.NewInt32Builder(memory.DefaultAllocator)
   defer ib.Release()

   ib.AppendValues([]int32{1, 2, 3}, nil)
   i1 := ib.NewInt32Array()
   defer i1.Release()

   ib.AppendValues([]int32{4, 5, 6, 7, 8, 9, 10}, nil)
   i2 := ib.NewInt32Array()
   defer i2.Release()

   c := arrow.NewChunked(
    arrow.PrimitiveTypes.Int32,
    []arrow.Array{i1, i2},
   )
   return c
  }()
  defer chunk.Release()

  return arrow.NewColumn(schema.Field(0), chunk)
 }()
 defer col1.Release()

 col2 := func() *arrow.Column {
  chunk := func() *arrow.Chunked {
   fb := array.NewFloat64Builder(memory.DefaultAllocator)
   defer fb.Release()

   fb.AppendValues([]float64{1.1, 2.2, 3.3, 4.4, 5.5}, nil)
   f1 := fb.NewFloat64Array()
   defer f1.Release()

   fb.AppendValues([]float64{6.6, 7.7}, nil)
   f2 := fb.NewFloat64Array()
   defer f2.Release()

   fb.AppendValues([]float64{8.8, 9.9, 10.0}, nil)
   f3 := fb.NewFloat64Array()
   defer f3.Release()

   c := arrow.NewChunked(
    arrow.PrimitiveTypes.Float64,
    []arrow.Array{f1, f2, f3},
   )
   return c
  }()
  defer chunk.Release()

  return arrow.NewColumn(schema.Field(1), chunk)
 }()
 defer col2.Release()

 col3 := func() *arrow.Column {
  chunk := func() *arrow.Chunked {
   sb := array.NewStringBuilder(memory.DefaultAllocator)
   defer sb.Release()

   sb.AppendValues([]string{"s1", "s2"}, nil)
   s1 := sb.NewStringArray()
   defer s1.Release()

   sb.AppendValues([]string{"s3", "s4"}, nil)
   s2 := sb.NewStringArray()
   defer s2.Release()

   sb.AppendValues([]string{"s5", "s6", "s7", "s8", "s9", "s10"}, nil)
   s3 := sb.NewStringArray()
   defer s3.Release()

   c := arrow.NewChunked(
    arrow.BinaryTypes.String,
    []arrow.Array{s1, s2, s3},
   )
   return c
  }()
  defer chunk.Release()

  return arrow.NewColumn(schema.Field(2), chunk)
 }()
 defer col3.Release()

 var tbl arrow.Table
 tbl = array.NewTable(schema, []arrow.Column{*col1, *col2, *col3}, -1)
 defer tbl.Release()

 dumpTable(tbl)
}

func dumpTable(tbl arrow.Table) {
 s := tbl.Schema()
 fmt.Println(s)
 fmt.Println("------")

 fmt.Println("the count of table columns=", tbl.NumCols())
 fmt.Println("the count of table rows=", tbl.NumRows())
 fmt.Println("------")

 for i := 0; i < int(tbl.NumCols()); i++ {
  col := tbl.Column(i)
  fmt.Printf("arrays in column(%s):\n", col.Name())
  chunk := col.Data()
  for _, arr := range chunk.Chunks() {
   fmt.Println(arr)
  }
  fmt.Println("------")
 }
}


```

我们看到：table 创建之前，我们需要准备一个 schema，以及各个 column。每个 column 则是一个 chunked array。

运行上述代码，我们得到如下结果：

```
$go run table.go
schema:
  fields: 3
    - col1: type=int32
    - col2: type=float64
    - col3: type=utf8
------
the count of table columns= 3
the count of table rows= 10
------
arrays in column(col1):
[1 2 3]
[4 5 6 7 8 9 10]
------
arrays in column(col2):
[1.1 2.2 3.3 4.4 5.5]
[6.6 7.7]
[8.8 9.9 10]
------
arrays in column(col3):
["s1" "s2"]
["s3" "s4"]
["s5" "s6" "s7" "s8" "s9" "s10"]
------


```

table 还支持 schema 变更，我们可以基于上述代码为 table 增加一列：

```
// table_schema_change.go

func main() {
 schema := arrow.NewSchema(
  []arrow.Field{
   {Name: "col1", Type: arrow.PrimitiveTypes.Int32},
   {Name: "col2", Type: arrow.PrimitiveTypes.Float64},
   {Name: "col3", Type: arrow.BinaryTypes.String},
  },
  nil,
 )

 col1 := func() *arrow.Column {
  chunk := func() *arrow.Chunked {
   ib := array.NewInt32Builder(memory.DefaultAllocator)
   defer ib.Release()

   ib.AppendValues([]int32{1, 2, 3}, nil)
   i1 := ib.NewInt32Array()
   defer i1.Release()

   ib.AppendValues([]int32{4, 5, 6, 7, 8, 9, 10}, nil)
   i2 := ib.NewInt32Array()
   defer i2.Release()

   c := arrow.NewChunked(
    arrow.PrimitiveTypes.Int32,
    []arrow.Array{i1, i2},
   )
   return c
  }()
  defer chunk.Release()

  return arrow.NewColumn(schema.Field(0), chunk)
 }()
 defer col1.Release()

 col2 := func() *arrow.Column {
  chunk := func() *arrow.Chunked {
   fb := array.NewFloat64Builder(memory.DefaultAllocator)
   defer fb.Release()

   fb.AppendValues([]float64{1.1, 2.2, 3.3, 4.4, 5.5}, nil)
   f1 := fb.NewFloat64Array()
   defer f1.Release()

   fb.AppendValues([]float64{6.6, 7.7}, nil)
   f2 := fb.NewFloat64Array()
   defer f2.Release()

   fb.AppendValues([]float64{8.8, 9.9, 10.0}, nil)
   f3 := fb.NewFloat64Array()
   defer f3.Release()

   c := arrow.NewChunked(
    arrow.PrimitiveTypes.Float64,
    []arrow.Array{f1, f2, f3},
   )
   return c
  }()
  defer chunk.Release()

  return arrow.NewColumn(schema.Field(1), chunk)
 }()
 defer col2.Release()

 col3 := func() *arrow.Column {
  chunk := func() *arrow.Chunked {
   sb := array.NewStringBuilder(memory.DefaultAllocator)
   defer sb.Release()

   sb.AppendValues([]string{"s1", "s2"}, nil)
   s1 := sb.NewStringArray()
   defer s1.Release()

   sb.AppendValues([]string{"s3", "s4"}, nil)
   s2 := sb.NewStringArray()
   defer s2.Release()

   sb.AppendValues([]string{"s5", "s6", "s7", "s8", "s9", "s10"}, nil)
   s3 := sb.NewStringArray()
   defer s3.Release()

   c := arrow.NewChunked(
    arrow.BinaryTypes.String,
    []arrow.Array{s1, s2, s3},
   )
   return c
  }()
  defer chunk.Release()

  return arrow.NewColumn(schema.Field(2), chunk)
 }()
 defer col3.Release()

 var tbl arrow.Table
 tbl = array.NewTable(schema, []arrow.Column{*col1, *col2, *col3}, -1)
 defer tbl.Release()

 dumpTable(tbl)

 col4 := func() *arrow.Column {
  chunk := func() *arrow.Chunked {
   sb := array.NewStringBuilder(memory.DefaultAllocator)
   defer sb.Release()

   sb.AppendValues([]string{"ss1", "ss2"}, nil)
   s1 := sb.NewStringArray()
   defer s1.Release()

   sb.AppendValues([]string{"ss3", "ss4", "ss5"}, nil)
   s2 := sb.NewStringArray()
   defer s2.Release()

   sb.AppendValues([]string{"ss6", "ss7", "ss8", "ss9", "ss10"}, nil)
   s3 := sb.NewStringArray()
   defer s3.Release()

   c := arrow.NewChunked(
    arrow.BinaryTypes.String,
    []arrow.Array{s1, s2, s3},
   )
   return c
  }()
  defer chunk.Release()

  return arrow.NewColumn(arrow.Field{Name: "col4", Type: arrow.BinaryTypes.String}, chunk)
 }()
 defer col4.Release()

 tbl, err := tbl.AddColumn(
  3,
  arrow.Field{Name: "col4", Type: arrow.BinaryTypes.String},
  *col4,
 )
 if err != nil {
  panic(err)
 }

 dumpTable(tbl)
}


```

运行上述示例，输出如下：

```
$go run table_schema_change.go
schema:
  fields: 3
    - col1: type=int32
    - col2: type=float64
    - col3: type=utf8
------
the count of table columns= 3
the count of table rows= 10
------
arrays in column(col1):
[1 2 3]
[4 5 6 7 8 9 10]
------
arrays in column(col2):
[1.1 2.2 3.3 4.4 5.5]
[6.6 7.7]
[8.8 9.9 10]
------
arrays in column(col3):
["s1" "s2"]
["s3" "s4"]
["s5" "s6" "s7" "s8" "s9" "s10"]
------
schema:
  fields: 4
    - col1: type=int32
    - col2: type=float64
    - col3: type=utf8
    - col4: type=utf8
------
the count of table columns= 4
the count of table rows= 10
------
arrays in column(col1):
[1 2 3]
[4 5 6 7 8 9 10]
------
arrays in column(col2):
[1.1 2.2 3.3 4.4 5.5]
[6.6 7.7]
[8.8 9.9 10]
------
arrays in column(col3):
["s1" "s2"]
["s3" "s4"]
["s5" "s6" "s7" "s8" "s9" "s10"]
------
arrays in column(col4):
["ss1" "ss2"]
["ss3" "ss4" "ss5"]
["ss6" "ss7" "ss8" "ss9" "ss10"]
------



```

这种对 schema 变更操作的支持在实际开发中也是非常有用的。

4. 小结
-----

本文讲解了基于 array type 的三个高级数据结构：Record Batch、Chunked Array 和 Table。其中 Record Batch 是 Arrow Columnar Format 中的结构，可以被所有实现 arrow 的编程语言所支持；Chunked Array 和 Table 则是在一些编程语言的实现中创建的。

三个概念容易混淆，这里给出简单记法：

*   Record Batch: schema + 长度相同的多个 array
    
*   Chunked Array: []array
    
*   Table: schema + 总长度相同的多个 Chunked Array
    

> 注：本文涉及的源代码在这里 [8] 可以下载。

5. 参考资料
-------

*   Apache Arrow Glossary[9] - https://arrow.apache.org/docs/format/Glossary.html
    

“Gopher 部落” 知识星球 [10] 旨在打造一个精品 Go 学习和进阶社群！高品质首发 Go 技术文章，“三天” 首发阅读权，每年两期 Go 语言发展现状分析，每天提前 1 小时阅读到新鲜的 Gopher 日报，网课、技术专栏、图书内容前瞻，六小时内必答保证等满足你关于 Go 语言生态的所有需求！2023 年，Gopher 部落将进一步聚焦于如何编写雅、地道、可读、可测试的 Go 代码，关注代码质量并深入理解 Go 核心技术，并继续加强与星友的互动。欢迎大家加入！

### 参考资料

[1] 

《Arrow 数据类型》: _https://tonybai.com/2023/06/25/a-guide-of-using-apache-arrow-for-gopher-part1_

[2] 

《Arrow Go 实现的内存管理》: _https://tonybai.com/2023/06/30/a-guide-of-using-apache-arrow-for-gopher-part2_

[3] 

Record Batch: _https://arrow.apache.org/docs/format/Glossary.html#term-record-batch_

[4] 

《In-Memory Analytics with Apache Arrow》: _https://book.douban.com/subject/35954154/_

[5] 

《Go 语言第一课》: _http://gk.link/a/10AVZ_

[6] 

《In-Memory Analytics with Apache Arrow》: _https://book.douban.com/subject/35954154/_

[7] 

这里: _https://arrow.apache.org/docs/format/Glossary.html#term-table_

[8] 

这里: _https://github.com/bigwhite/experiments/blob/master/arrow/advanced-datastructure_

[9] 

Apache Arrow Glossary: _https://arrow.apache.org/docs/format/Glossary.html_

[10] 

“Gopher 部落” 知识星球: _https://wx.zsxq.com/dweb2/index/group/51284458844544_

[11] 

链接地址: _https://m.do.co/c/bff6eed92687_
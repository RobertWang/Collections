> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/271145056)

​关注微信公众号:“后端开发杂谈”, 领取更多的干货文章.

> 简介: 日常 go 的开发当中, 我们经常会使用到 map 这种数据结构, 可以说 map 这种数据结构已经成为了现代语言的基础数据结构吧. 最常用的设计思路是 "数组 + 链表". 那么 go 当中具体又是怎样设计的呢? 还有它是怎样去做一些性能的优化的呢? 带着这些问题, 我们一起去看看 go 的 map 源码吧!

golang map 源码解析
---------------

数据结构与实际的数据结构
------------

map 中的数据被存放在一个数组中的, 数组的元素是桶 (bucket), 每个桶至多包含 8 个键值对数据. 哈希值低位(low-order bits) 用于选择桶, 哈希值高位(high-order bits) 用于在一个独立的桶中区别出键. 哈希值高低位示意图如下:

源码位于 `src/runtime/map.go`.

// 结构体

```
// A header for a Go map.
type hmap struct {
    count     int    // 代表哈希表中的元素个数, 调用len(map)时, 返回的就是该字段值.
    flags     uint8  // 状态标志, 下文常量中会解释四种状态位含义.
    B         uint8  // buckets(桶)的对数log_2(哈希表元素数量最大可达到装载因子*2^B)
    noverflow uint16 // 溢出桶的大概数量.
    hash0     uint32 // 哈希种子.

    buckets    unsafe.Pointer // 指向buckets数组的指针, 数组大小为2^B, 如果元素个数为0, 它为nil.
    oldbuckets unsafe.Pointer // 如果发生扩容, oldbuckets是指向老的buckets数组的指针, 老的buckets数组大小是新
                              // 的buckets的1/2.非扩容状态下, 它为nil. 它是判断是否处于扩容状态的标识

    nevacuate  uintptr        // 表示扩容进度, 小于此地址的buckets代表已搬迁完成.

    extra *mapextra // 这个字段是为了优化GC扫描而设计的. 当key和value均不包含指针, 并且都可以inline时使用.
                    // extra是指向mapextra类型的指针.
}

// mapextra holds fields that are not present on all maps.
type mapextra struct {
    // 就使用 hmap 的 extra 字段来存储 overflow buckets, 

    // 如果 key 和 value 都不包含指针, 并且可以被 inline(<=128 字节), 则将 bucket type 标记为不包含指针 (使用
    // ptrdata 字段, 为0表示不包含指针). 这样可以避免 GC 扫描整个 map. 但是 bmap.overflow 是一个指针. 这时候我
    // 们只能把这些 overflow 的指针都放在 hmap.extra.overflow 和 hmap.extra.oldoverflow 中了.
    //  
    // 当 key 和 elem 不包含指针时, 才使用 overflow 和 oldoverflow. 
    // overflow 包含的是 hmap.buckets 的 overflow bucket, 
    // oldoverflow 包含扩容时的 hmap.oldbuckets 的 overflow bucket.
    overflow    *[]*bmap
    oldoverflow *[]*bmap

    // 指向空闲的 overflow bucket 的指针(第一个空闲的bucket地址)
    nextOverflow *bmap
}

// A bucket for a Go map.
type bmap struct {
    // tophash包含此桶中每个键的哈希值最高字节(高8位)信息(也就是前面所述的high-order bits).
    // 如果tophash[0] < minTopHash, tophash[0]则代表桶的搬迁(evacuation)状态.
    tophash [bucketCnt]uint8
}
```

// 常量值

```
const (
    // 一个桶中最多能装载的键值对(key-value)的个数为8
    bucketCntBits = 3
    bucketCnt     = 1 << bucketCntBits // 8

    // 触发扩容的装载因子为13/2=6.5
    loadFactorNum = 13
    loadFactorDen = 2

    // 键和值超过128个字节, 就会被转换为指针
    maxKeySize  = 128
    maxElemSize = 128

    // 数据偏移量应该是bmap结构体的大小, 它需要正确地对齐. 
    // 对于amd64p32而言, 这意味着: 即使指针是32位的, 也是64位对齐. 
    dataOffset = unsafe.Offsetof(struct {
        b bmap
        v int64
    }{}.v)

    // 每个桶(如果有溢出, 则包含它的overflow的链桶) 在搬迁完成状态(evacuated states)下, 要么会包含它所有的键值对,
    // 要么一个都不包含(但不包括调用evacuate()方法阶段,该方法调用只会在对map发起write时发生,在该阶段其他goroutine
    // 是无法查看该map的). 简单的说,桶里的数据要么一起搬走,要么一个都还未搬.
    //
    // tophash除了放置正常的高8位hash值, 还会存储一些特殊状态值(标志该cell的搬迁状态). 正常的tophash值, 
    // 最小应该是5,以下列出的就是一些特殊状态值. 
    emptyRest      = 0 // 空的cell, 并且比它高索引位的cell或者overflows中的cell都是空的. (初始化bucket时,就是该状态)
    emptyOne       = 1 // 空的cell, cell已经被搬迁到新的bucket
    evacuatedX     = 2 // 键值对已经搬迁完毕,key在新buckets数组的前半部分
    evacuatedY     = 3 // 键值对已经搬迁完毕,key在新buckets数组的后半部分
    evacuatedEmpty = 4 // cell为空,整个bucket已经搬迁完毕
    minTopHash     = 5 // tophash的最小正常值

    // flags
    iterator     = 1 // 可能有迭代器在使用buckets
    oldIterator  = 2 // 可能有迭代器在使用oldbuckets
    hashWriting  = 4 // 有协程正在向map写人key
    sameSizeGrow = 8 // 等量扩容

    // 用于迭代器检查的bucket ID
    noCheck = 1<<(8*sys.PtrSize) - 1 // 系统的最大值
)
```

// bmap(即 map 中的 bucket) 内存结构

```
// src/cmd/compile/internal/gc/reflect.go:bmap
// bucket 内存结构 
func bmap(t *types.Type) *types.Type {
    if t.MapType().Bucket != nil {
        return t.MapType().Bucket
    }

    bucket := types.New(TSTRUCT)
    keytype := t.Key()
    elemtype := t.Elem()
    dowidth(keytype) // 确定 keytype 的size
    dowidth(elemtype)

    // MAXKEYSIZE=128
    if keytype.Width > MAXKEYSIZE {
        keytype = types.NewPtr(keytype)
    }
    if elemtype.Width > MAXELEMSIZE {
        elemtype = types.NewPtr(elemtype)
    }

    field := make([]*types.Field, 0, 5)

    // BUCKETSIZE=8
    // The first field is: uint8 topbits[BUCKETSIZE].
    arr := types.NewArray(types.Types[TUINT8], BUCKETSIZE)
    field = append(field, makefield("topbits", arr))

    arr = types.NewArray(keytype, BUCKETSIZE)
    arr.SetNoalg(true)
    keys := makefield("keys", arr)
    field = append(field, keys)

    arr = types.NewArray(elemtype, BUCKETSIZE)
    arr.SetNoalg(true)
    elems := makefield("elems", arr)
    field = append(field, elems)

    // 确保 overflow 指针是结构中的最后一个内存, 因为运行时假定它可以使用size-ptrSize作为 overflow 指针的偏移量. 
    // 一旦计算了偏移量和大小, 我们就要仔细检查下面的属性(在已经忽略检查代码).
    //
    // BUCKETSIZE为8, 因此该结构在此处已对齐为64位.
    // 在32位系统上, 最大对齐方式为32位, 并且溢出指针将添加另一个32位字段, 并且该结构将以无填充结尾.
    // 在64位系统上, 最大对齐方式为64位, 并且溢出指针将添加另一个64位字段, 并且该结构将以无填充结尾.
    // 但是, 在nacl/amd64p32上, 最大对齐方式是64位, 但是溢出指针只会添加一个32位字段, 因此, 如果该结构需要64位填充
    // (由于key或elem的原因), 则它将最后带有一个额外的32位填充字段.
    // 通过在此处发出填充.
    if int(elemtype.Align) > Widthptr || int(keytype.Align) > Widthptr {
        field = append(field, makefield("pad", types.Types[TUINTPTR]))
    }

    // 如果keys和elems没有指针, 则map实现可以在侧面保留一个 overflow 指针列表, 以便可以将 buckets 标记为没有指针.
    // 在这种情况下, 通过将 overflow 字段的类型更改为 uintptr, 使存储桶不包含任何指针.
    otyp := types.NewPtr(bucket)
    if !types.Haspointers(elemtype) && !types.Haspointers(keytype) {
        otyp = types.Types[TUINTPTR]
    }
    overflow := makefield("overflow", otyp)
    field = append(field, overflow)

    // link up fields
    bucket.SetNoalg(true)
    bucket.SetFields(field[:])
    dowidth(bucket)

    t.MapType().Bucket = bucket

    bucket.StructType().Map = t
    return bucket
}
```

// hmap(即 map) 内存结构

```
// src/cmd/compile/internal/gc/reflect.go:hmap
// map 内存结构
func hmap(t *types.Type) *types.Type {
    if t.MapType().Hmap != nil {
        return t.MapType().Hmap
    }

    bmap := bmap(t)

    // type hmap struct {
    //    count      int
    //    flags      uint8
    //    B          uint8
    //    noverflow  uint16
    //    hash0      uint32
    //    buckets    *bmap
    //    oldbuckets *bmap
    //    nevacuate  uintptr
    //    extra      unsafe.Pointer // *mapextra
    // }
    // must match runtime/map.go:hmap.
    fields := []*types.Field{
        makefield("count", types.Types[TINT]),
        makefield("flags", types.Types[TUINT8]),
        makefield("B", types.Types[TUINT8]),
        makefield("noverflow", types.Types[TUINT16]),
        makefield("hash0", types.Types[TUINT32]), // Used in walk.go for OMAKEMAP.
        makefield("buckets", types.NewPtr(bmap)), // Used in walk.go for OMAKEMAP.
        makefield("oldbuckets", types.NewPtr(bmap)),
        makefield("nevacuate", types.Types[TUINTPTR]),
        makefield("extra", types.Types[TUNSAFEPTR]),
    }

    hmap := types.New(TSTRUCT)
    hmap.SetNoalg(true)
    hmap.SetFields(fields)
    dowidth(hmap)

    // The size of hmap should be 48 bytes on 64 bit and 28 bytes on 32 bit platforms.
    // 5("count", "buckets", "oldbuckets", "nevacuate", "extra")
    if size := int64(8 + 5*Widthptr); hmap.Width != size {
        Fatalf("hmap size not correct: got %d, want %d", hmap.Width, size)
    }

    t.MapType().Hmap = hmap
    hmap.StructType().Map = t
    return hmap
}
```

bmap 也就是 bucket(桶) 的内存模型图解如下 (代码逻辑就是上述的 `bmap` 函数).

该桶的第 7,8 位 cell 还未有对应的键值对. 注意: key 和 value 是各自存储起来的, 并非想象中的 key/value/key/value... 的形式. 这样的做法好处在于消除 key/value 之间的 padding, 例如`map[int64]int`. 还有, 在 8 个键值对数据后面还有一个 overflow 指针, 因为桶中最多只能装 8 个键值对, 如果有多余的键值对落到当前桶, 那么就需要再构建一个桶 (溢出桶), 通过 overflow 指针链接起来.

最后, 这里展示一个 B=4 的完整 map 结构:

![](https://pic1.zhimg.com/v2-810315f7da2ad54fe3ff34dc87b0e18c_b.jpg)

辅助函数
----

```
// 地址偏移(内存地址连续, 这是map内存操作的一个基础)
func add(p unsafe.Pointer, x uintptr) unsafe.Pointer {
    return unsafe.Pointer(uintptr(p) + x)
}

// 2^b, b的有效范围[0-63]
func bucketShift(b uint8) uintptr {
    return uintptr(1) << (b & (sys.PtrSize*8 - 1))
}

// 2^b-1
func bucketMask(b uint8) uintptr {
    return bucketShift(b) - 1
}

// hash值的高8位, 一般是作为第一次比较
func tophash(hash uintptr) uint8 {
    top := uint8(hash >> (sys.PtrSize*8 - 8))
    if top < minTopHash {
        top += minTopHash
    }
    return top
}

// 获取 bucket 的状态是否在 evacuated (迁移状态)
func evacuated(b *bmap) bool {
    h := b.tophash[0]
    return h > emptyOne && h < minTopHash
}

// 检测给定的tophash所对于的cell是否为空.
func isEmpty(x uint8) bool {
    return x <= emptyOne
}

// 获取 b 的 overflow 指针
func (b *bmap) overflow(t *maptype) *bmap {
    // 很巧妙, bucketsize 的最后一个 sys.PtrSize 即是 overflow 指针
    return *(**bmap)(add(unsafe.Pointer(b), uintptr(t.bucketsize)-sys.PtrSize))
}

// 设置 b 的 overflow 指针
func (b *bmap) setoverflow(t *maptype, ovf *bmap) {
    *(**bmap)(add(unsafe.Pointer(b), uintptr(t.bucketsize)-sys.PtrSize)) = ovf
}
```

map 创建
------

源码位置 `src/runtime/map.go`

初始化 map 的两种方式:

```
make(map[int]int)
make(map[int]int, hint) // hint是初始化map的hint, 即桶的大小
```

对于不指定初始化大小, 和初始化值 hint <8 (bucketCnt) 时, go 会调用 `make_small` 函数, 并直接从堆上进行分配.

```
func makemap_small() *hmap {
    h := new(hmap)
    h.hash0 = fastrand()
    return h
}
```

当 hint>8 时, 则调用 `makemap` 函数:

// map 创建

```
// 如果编译器认为map和第一个bucket可以直接创建在栈上, h和bucket可能都是非空
// 如果h != nil, 那么map可以直接在h中创建.
// 如果h.buckets != nil, 那么h指向的bucket可以作为map的第一个bucket使用.
func makemap(t *maptype, hint int, h *hmap) *hmap {
    // math.MulUintptr返回hint与t.bucket.size的乘积, 并判断该乘积是否溢出.
    mem, overflow := math.MulUintptr(uintptr(hint), t.bucket.size)
    // maxAlloc的值, 根据平台系统的差异而不同，具体计算方式参照src/runtime/malloc.go
    if overflow || mem > maxAlloc {
        hint = 0
    }

    // initialize Hmap
    if h == nil {
        h = new(hmap)
    }
    // 通过fastrand得到哈希种子
    h.hash0 = fastrand()

    // 根据输入的元素个数hint, 找到能装下这些元素的B值
    B := uint8(0)
    //  hint > 8 && uintptr(hint) > bucketShift(B)*6.5
    for overLoadFactor(hint, B) {
        B++
    }
    h.B = B

    // 分配初始哈希表
    // 如果B为0，那么buckets字段后续会在mapassign方法中lazily分配
    if h.B != 0 {
        var nextOverflow *bmap
        // makeBucketArray创建一个map的底层保存buckets的数组，它最少会分配h.B^2的大小。
        h.buckets, nextOverflow = makeBucketArray(t, h.B, nil)
        if nextOverflow != nil {
            h.extra = new(mapextra)
            h.extra.nextOverflow = nextOverflow
        }
    }
    return h
}
// makeBucket为map创建用于保存buckets的数组. 
func makeBucketArray(t *maptype, b uint8, dirtyalloc unsafe.Pointer) (buckets unsafe.Pointer, nextOverflow *bmap) {
    base := bucketShift(b)
    nbuckets := base

    // 对于小的b值(小于4),即桶的数量小于16时,使用溢出桶的可能性很小. 对于此情况, 就避免计算开销. 
    if b >= 4 {
        // 当桶的数量大于等于16个时, 正常情况下就会额外创建2^(b-4)个溢出桶
        nbuckets += bucketShift(b - 4)
        sz := t.bucket.size * nbuckets // 计算内存大小
        up := roundupsize(sz)          // 计算mallocgc将分配的内存块的大小(需要以此为准)
        if up != sz {
            nbuckets = up / t.bucket.size
        }
    }

    // 这里, dirtyalloc 分两种情况. 如果它为nil, 则会分配一个新的底层数组. 
    // 如果它不为nil,则它指向的是曾经分配过的底层数组, 该底层数组是由之前同样的t和b参数通过makeBucketArray分配的,
    // 如果数组不为空,需要把该数组之前的数据清空并复用. 
    if dirtyalloc == nil {
        buckets = newarray(t.bucket, int(nbuckets))
    } else {
        buckets = dirtyalloc
        size := t.bucket.size * nbuckets
        if t.bucket.ptrdata != 0 {
            memclrHasPointers(buckets, size) // 开启了写屏障, 指针清理
        } else {
            memclrNoHeapPointers(buckets, size) // 最终都会调用此方法
        }
    }

    // 即b大于等于4的情况下, 会预分配一些溢出桶. 
    // 为了把跟踪这些溢出桶的开销降至最低, 使用了以下约定:
    // 如果预分配的溢出桶的overflow指针为nil, 那么可以通过指针碰撞(bumping the pointer)获得更多可用桶. 
    // (关于指针碰撞: 假设内存是绝对规整的,所有用过的内存都放在一边,空闲的内存放在另一边,中间放着一个指针作为分界点的
    // 指示器, 那所分配内存就仅仅是把那个指针向空闲空间那边挪动一段与对象大小相等的距离, 这种分配方式称为"指针碰撞")
    // 对于最后一个溢出桶, 需要一个安全的非nil指针指向它. 
    if base != nbuckets {
        // buckets(基地址) + base(2^B)*bucketsize, 即获得第一个 overflow
        nextOverflow = (*bmap)(add(buckets, base*uintptr(t.bucketsize)))
        // 最后一个 overflow
        last := (*bmap)(add(buckets, (nbuckets-1)*uintptr(t.bucketsize)))
        last.setoverflow(t, (*bmap)(buckets)) // 最后一个 overflow 指针指向 buckets(基地址, 也是安全的指针)
    }
    return buckets, nextOverflow
}
```

> 注: 上述的 makeBucketArray 代码当前可以得到的一个结论是, **正常状况下**, 正常的桶和溢出的桶在内存存储空间是连续的, 只是被 `hmap` 中的不同的字段引用而已.

map 插入
------

go 当中的位运算:

```
A  |  B, 或操作, 添加 B 当中相应的标记位1
A  &  B, 与操作, 寻找 A,B 共有的标记1

A  &^ B, 与非操作, 移除 B 当中相应的标记位1(是 "或" 运算的反向操作)
A  ^  B, 异或操作, 移除 A,B 共有的标记1, 添加 A,B 不共有的标记
```

// 插入操作, 实际上就是找到一个写入 value 的内存地址, 后续通过内存地址操作进行赋值.

```
func mapassign(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {
    // 如果h是空指针,赋值会引起panic
    // 例如以下语句
    // var m map[string]int
    // m["k"] = 1
    if h == nil {
        panic(plainError("assignment to entry in nil map"))
    }

    // 如果开启了竞态检测 -race
    if raceenabled {
        callerpc := getcallerpc()
        pc := funcPC(mapassign)
        racewritepc(unsafe.Pointer(h), callerpc, pc)
        raceReadObjectPC(t.key, key, callerpc, pc)
    }
    // 如果开启了memory sanitizer -msan
    if msanenabled {
        msanread(key, t.key.size)
    }
    // 有其他goroutine正在往map中写key, 会抛出以下错误. 不能并发写, 或者并发读写. 可以并发读取
    if h.flags&hashWriting != 0 {
        throw("concurrent map writes")
    }
    // 通过key和哈希种子, 算出对应哈希值
    hash := t.hasher(key, uintptr(h.hash0))

    // 将flags的值与hashWriting做按位 "异或" 运算, 移除 hashWriting 的相应标记位
    // 因为在当前goroutine可能还未完成key的写入, 再次调用t.hasher会发生panic.
    h.flags ^= hashWriting

    if h.buckets == nil {
        h.buckets = newobject(t.bucket) // newarray(t.bucket, 1)
    }

again:
    // bucketMask返回值是2的B次方减1
    // 因此,通过hash值与bucketMask返回值做按位与操作,返回的在buckets数组中的第几号桶
    bucket := hash & bucketMask(h.B) // 获取bucket的位置
    // 如果map正在扩容(即h.oldbuckets != nil)中, 则先进行搬移工作(当前的bucket). 
    if h.growing() {
        growWork(t, h, bucket) // 最多搬移两个bucket
    }

    // 计算出上面求出的第几号bucket的内存位置
    // post = start + bucketNumber * bucketsize
    b := (*bmap)(unsafe.Pointer(uintptr(h.buckets) + bucket*uintptr(t.bucketsize)))
    top := tophash(hash) // 获取 bucket 内的原始的位置(即hash的高8位)

    var inserti *uint8         // 记录 tophash 对应位置的指针
    var insertk unsafe.Pointer // 记录 key 的底层内存位置(要剥离指针)
    var elem unsafe.Pointer    // 记录 value 的底层内存位置

bucketloop:
    for {
        // 遍历桶中的8个cell
        for i := uintptr(0); i < bucketCnt; i++ {
            // 这里分两种情况:
            // 第一种情况是cell位的tophash值和当前tophash值不相等.
            // 在 b.tophash[i] != top 的情况下, 理论上有可能会是一个空槽位.
            // 一般情况下 map 的槽位分布是这样的, e 表示 empty:
            // [h0][h1][h2][h3][h4][e][e][e]
            // 但在执行过 delete 操作时,可能会变成这样:
            // [h0][h1][e][e][h5][e][e][e]
            // 所以如果再插入的话,会尽量往前面的位置插
            // [h0][h1][e][e][h5][e][e][e]
            //          ^
            //          ^
            //       这个位置
            // 所以在循环的时候还要顺便把前面的空位置先记下来
            // 因为有可能在后面会找到相等的key,也可能找不到相等的key
            if b.tophash[i] != top {
                // 如果cell位为空(b.tophash[i] <= emptyOne), 那么就可以在对应位置进行插入
                if isEmpty(b.tophash[i]) && inserti == nil {
                    inserti = &b.tophash[i]
                    // 这里需要注意实际的 bmap 结构. dataOffset 是前面的8个 tophash 的偏移量
                    insertk = add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
                    elem = add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.elemsize))
                }

                // 后面所有的 cell 和 overflow 都是空的. 但是前面已经记录了当前的位置, 无需再次记录
                if b.tophash[i] == emptyRest {
                    break bucketloop // goto done
                }
                continue
            }

            // 第二种情况是cell位的tophash值和当前的tophash值相等
            // indirectkey()  // store ptr to key instead of key itself
            // indirectelem() // store ptr to elem instead of elem itself
            k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
            if t.indirectkey() {
                k = *((*unsafe.Pointer)(k))
            }
            // 注意: 即使当前cell位的tophash值相等,不一定它对应的key也是相等的,所以还要做一个key值判断
            if !t.key.equal(key, k) {
                continue
            }

            // 到这里,说明 map 当中存在当前的 key, 只需要更新它即可. 这个时候是不再需要 inserti, insertk的值了
            // 因为它们只是辅助后面的插入的动作的. 直接调到 done 即可.
            // needkeyupdate() // true if we need to update key on an overwrite
            if t.needkeyupdate() {
                typedmemmove(t.key, k, key)
            }
            // pos = start(bucket) + dataOffset + 8*keysize + i*elemsize
            elem = add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.elemsize))
            goto done
        }

        // 进入到 overflow bucket 当中查找
        // *(**bmap)(add(unsafe.Pointer(b), uintptr(t.bucketsize)-sys.PtrSize)), 很巧妙的方法
        // 说明: t.bucketsize 是 bucket 的大小, 而最后一个指针就是 *bmap
        ovf := b.overflow(t)
        // overflow bucket 为空, 终止当前的循环. 
        if ovf == nil {
            break
        }
        b = ovf
    }

    // 在已有的桶和溢出桶中都未找到合适的cell供key写入, 那么有可能会触发以下两种情况
    // 情况一:
    // 判断当前map的装载因子是否达到设定的6.5阈值, 或者当前map的溢出桶数量是否过多. 如果存在这两种情况之一, 则进行扩容
    // 操作. 
    // hashGrow()实际并未完成扩容, 对哈希表数据的搬迁(复制)操作是通过growWork()来完成的. 
    // 重新跳入again逻辑, 在进行完growWork()操作后, 再次遍历新的桶. 
    // 分别分析情况1(装载因子) 和 情况2(buckets与overflow buckets)
    if !h.growing() && (overLoadFactor(h.count+1, h.B) || tooManyOverflowBuckets(h.noverflow, h.B)) {
        hashGrow(t, h)
        goto again // Growing the table invalidates everything, so try again
    }

    // 情况二:
    // 在不满足情况一的条件下, 并且没有找到插入位置, 会为当前桶再新建溢出桶,并将tophash,key插入到新建溢出桶的对应内存
    // 的0号位置
    if inserti == nil {
        // all current buckets are full, allocate a new one.
        newb := h.newoverflow(t, b)
        inserti = &newb.tophash[0]
        insertk = add(unsafe.Pointer(newb), dataOffset)
        elem = add(insertk, bucketCnt*uintptr(t.keysize))
    }

    // 在插入位置存入新的key和value
    if t.indirectkey() {
        kmem := newobject(t.key)
        *(*unsafe.Pointer)(insertk) = kmem
        insertk = kmem
    }
    if t.indirectelem() {
        vmem := newobject(t.elem)
        *(*unsafe.Pointer)(elem) = vmem
    }
    typedmemmove(t.key, insertk, key) // 写入 key
    *inserti = top                    // 写入 tophash
    h.count++                         // map中的key数量+1

done:
    // 插入操作
    if h.flags&hashWriting == 0 {
        throw("concurrent map writes")
    }
    h.flags &^= hashWriting
    if t.indirectelem() {
        elem = *((*unsafe.Pointer)(elem))
    }
    return elem // 返回 value 的底层内存位置
}
```

> 注: `mapassign` 的代码当中, 没有直接将 value 写入内存, 而是将 value 在内存当中的对应地址返回, 后续对内存地址写入 进行操作.

// 创建新的 overflow

```
func (h *hmap) newoverflow(t *maptype, b *bmap) *bmap {
    var ovf *bmap
    // 先检查是否有预分配的 overflow bucket, 如果有, 则从其中获取一个, 否则, 需要重新创建一个 bucket
    if h.extra != nil && h.extra.nextOverflow != nil {
        // 我们已经预分配了 overflow buckets [连续的内存地址]. 详细状况参考 makeBucketArray() 函数
        ovf = h.extra.nextOverflow
        if ovf.overflow(t) == nil {
            // 不是最后一个预分配的溢出存储桶. 这时候只需要修改nextOverflow地址指向下一个溢出桶(因为内存是连续的)
            h.extra.nextOverflow = (*bmap)(add(unsafe.Pointer(ovf), uintptr(t.bucketsize)))
        } else {
            // 最后一个预分配的溢出存储桶, 其地址有效, 指向了当前 buckets
            // 重置此存储桶上的 overflow 指针(该指针已设置为非nil标记值).
            ovf.setoverflow(t, nil)
            h.extra.nextOverflow = nil
        }
    } else {
        ovf = (*bmap)(newobject(t.bucket))
    }

    // 修改 noverflow
    h.incrnoverflow()
    // key和value 非指针
    if t.bucket.ptrdata == 0 { 
        h.createOverflow() // 创建 extra 和 overflow
        *h.extra.overflow = append(*h.extra.overflow, ovf) // 将  overflow 存储到 extra 当中
    }
    b.setoverflow(t, ovf) 
    return ovf
}
```

> 结论:  
> 1. 通过 newoverflow() 可以看出, overflow 先是使用预先分配的溢出桶内存 (此时桶和溢出桶是一整块完整的内存), 当预 先分配的溢出桶(数量为 2^(B-4) 个) 使用完了之后, 后续直接创建不连续的溢出桶. 注意: 无论是哪种溢出桶, 其内部的内存一定是 连续的, 否则 `add()` 函数操作将变得无效.  
> 2. map 的桶的内存一定是连续的, 这个在初始化或者扩容的时候进行分配好.

```
// incrnoverflow 递增 h.noverflow.
// noverflow 计算溢出桶的数量.
// 这用于触发相同大小的 map 增长. 
// 为了使hmap保持较小, noverflow是一个uint16.
// 当存储桶很少时, noverflow是一个精确的计数.
// 当有很多存储桶时, noverflow是一个近似计数.
func (h *hmap) incrnoverflow() {
    // 如果overflow buckets的数量与buckets的数量相同, 将触发相同大小的 map 增长.
    // 我们需要能够计数到 1<<h.B
    if h.B < 16 {
        h.noverflow++
        return
    }

    // 以 1 / (1 <<(h.B-15)) 的概率递增.
    // 当我们达到1<<15 - 1时, 我们将有大约与桶一样多的溢出桶.
    mask := uint32(1)<<(h.B-15) - 1
    // Example: if h.B == 18, then mask == 7,
    // and fastrand & 7 == 0 with probability 1/8.
    if fastrand()&mask == 0 {
        h.noverflow++
    }
}
```

map 查询
------

// 查询操作

```
func mapaccess1(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {
    // 如果开启了竞态检测 -race
    if raceenabled && h != nil {
        callerpc := getcallerpc()
        pc := funcPC(mapaccess1)
        racereadpc(unsafe.Pointer(h), callerpc, pc)
        raceReadObjectPC(t.key, key, callerpc, pc)
    }
    // 如果开启了memory sanitizer -msan
    if msanenabled && h != nil {
        msanread(key, t.key.size)
    }

    // 如果map为空或者元素个数为0, 返回零值
    if h == nil || h.count == 0 {
        if t.hashMightPanic() {
            t.hasher(key, 0) // see issue 23734
        }
        return unsafe.Pointer(&zeroVal[0])
    }

    // 当h.flags对应的值为hashWriting (代表有其他goroutine正在往map中写key)时, 那么位计算的结果不为0, 因此抛出以
    // 下错误. 这也表明, go的map是非并发安全的
    if h.flags&hashWriting != 0 {
        throw("concurrent map read and map write")
    }

    // 不同类型的key, 会使用不同的hash算法, 可详见src/runtime/alg.go中typehash函数中的逻辑.
    hash := t.hasher(key, uintptr(h.hash0))
    m := bucketMask(h.B)

    // 找到对应的bucket指针地址
    b := (*bmap)(add(h.buckets, (hash&m)*uintptr(t.bucketsize)))
    // 如果 oldbuckets 不为空, 那么证明map发生了扩容
    // 如果有扩容发生, 老的buckets中的数据可能还未搬迁至新的buckets里, 所以需要先在老的buckets中找
    if c := h.oldbuckets; c != nil {
        // 增量扩容
        if !h.sameSizeGrow() {
            m >>= 1
        }
        oldb := (*bmap)(add(c, (hash&m)*uintptr(t.bucketsize)))
        // 如果在oldbuckets中tophash[0]的值, 为 evacuatedX, evacuatedY, evacuatedEmpty 其中之一
        // 则 evacuated() 返回为true(表示搬迁完成). 因此, 只有当搬迁未完成时, 才会从此oldbucket中遍历.
        if !evacuated(oldb) {
            b = oldb
        }
    }

    // 取出当前key值的tophash值
    top := tophash(hash)
    // 以下是查找的核心逻辑
    // 双重循环遍历: 外层循环是从桶到溢出桶遍历; 内层是桶中的cell遍历
    // 跳出循环的条件有三种: 
    // 第一种是已经找到key值;
    // 第二种是当前桶再无溢出桶;
    // 第三种是当前桶中有cell位的tophash值是emptyRest, 这个值在前面解释过, 它代表此时的桶后面的cell还未利用, 
    // 所以无需再继续遍历. 
bucketloop:
    // 第二种情况
    for ; b != nil; b = b.overflow(t) {
        for i := uintptr(0); i < bucketCnt; i++ {
            if b.tophash[i] != top {
                // 第三种情况, 这种状况肯定是找不到了
                if b.tophash[i] == emptyRest {
                    break bucketloop
                }
                continue
            }
            // 因为在bucket中key是用连续的存储空间存储的, 因此可以通过bucket地址+数据偏移量(bmap结构体的大小)+keysize的大小, 
            // 得到k的地址. 同理, value的地址也是相似的计算方法, 只是再要加上8个keysize的内存地址.
            k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
            if t.indirectkey() {
                k = *((*unsafe.Pointer)(k))
            }
            // 判断key是否相等, 第一种情况
            if t.key.equal(key, k) {
                e := add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.elemsize))
                if t.indirectelem() {
                    e = *((*unsafe.Pointer)(e))
                }
                return e
            }
        }
    }

    // 所有的bucket都未找到, 则返回零值
    return unsafe.Pointer(&zeroVal[0])
}
```

`mapaccess1` 查找过程图解:

![](https://pic3.zhimg.com/v2-e2d259ecd8e5e39f1269ec291d9ce676_b.jpg)

map 的元素查找, 对应 go 代码有两种形式:

```
v := m[k]
v, ok := m[k]
```

形式一的代码实现, 是上述的 `mapaccess1` 方法. 此外源码当中还有 `mapaccess2` 方法, 函数签名是:

```
func mapaccess2(t *maptype, h *hmap, key unsafe.Pointer) (unsafe.Pointer, bool)
```

在 `mapaccess2` 多了一个 bool 类型的返回值, 它代表的 map 中是否存在相对应的 key.

源码中还有 `mapaccessK` 方法, 函数签名是:

```
func mapaccessK(t *maptype, h *hmap, key unsafe.Pointer) (unsafe.Pointer, unsafe.Pointer)
```

与 `mapaccess1` 相比, `mapaccessK` 同时返回了 key 和 value, 其代码逻辑也一致.

> 说明: mapaccess2() 返回 value 和 bool(表示 key 是否存在), mapaccessK() 返回 key 和 value. 它们和 mapaccess1() 的 逻辑基本上是一样的.

map 扩容与数据搬移
-----------

扩容条件说明:

1.  判断已经达到装载因子的临界点 (6.5), 即元素数量 >= 桶(bucket) 个数 * 6.5, 这个时候说明大部分桶是可能是快满了 (平均每 个桶插入 6.5 个键值对). 如果插入新元素, 有大概率需要溢出桶(overflow bucket) 上.

```
func overLoadFactor(count int, B uint8) bool {
   return count > bucketCnt && uintptr(count) > loadFactorNum*(bucketShift(B)/loadFactorDen)
}
```

1.  判断溢出桶是否太多, 当桶总数 <2^15, 如果溢出桶总数>= 桶总数, 则认为溢出桶太多. 当桶总数 >= 2^15, 当溢出桶总数 >= 2^15 时, 则认为溢出桶太多了.

```
func tooManyOverflowBuckets(noverflow uint16, B uint8) bool {
   if B > 15 {
      B = 15
   }
   return noverflow >= uint16(1)<<(B&15)
}
```

> 总结: 对于第 2 点, 是对第 1 点的补充. 因为在装载因子比较小的状况下, 有可能 map 的查找和插入效率也很低, 而第 1 点识别不出 来的这种状况. 表面现象就是计算装载因子的分子比较小, 即 map 里的元素总数少, 但是桶的数量多 (包含了大量的溢出桶).

针对上述两种状况, 采用不同的解决方案:

针对 1, 将 B+1, 新建一个 buckets 数组, 新的 buckets 大小是原来的 2 倍, 然后旧的 buckets 数据搬移到新的 buckets. 该方法称为 增量扩容.

针对 2, 并不扩容, bucket 数量维持不变, 重新做一遍类似增量扩容的搬移动作. 把松散的键值对重新排列一次, 以使得 bucket 的使用 率更高, 进而保证更快的存取. 该方法称为等量扩容.

对于 2 的解决方案, 存在一个极端情况: 如果插入 map 的 key 哈希都一样, 那么它们就会落到同一个 bucket 里, 超过 8 个就会产生 overflow bucket, 最终还是会造成 overflow bucket 过多. 移动元素其实解决不了问题. 但是 Go 的每一个 map 都会在初始化 阶段 makemap 时产生一个随机的哈希种子, 所以构造这种冲突没那么容易.

扩容主要涉及 `hashGrow()` 和 `growWork()` 两个函数. `hashGrow()` 是开启扩容, 并没有进行真正的数据 "搬移", 它只是分 配好了新的 buckets, 并将老的 buckets 赋值给 oldbuckets 字段上. 真正的 buckets 搬移动作是在 `growWork()` 函数. 调 用 `mapassign()`(插入, 更新) 和 `mapdelete()`(删除) 函数时会尝试搬移 buckets 的工作.

// 桶搬移操作, 一次最多搬移 2 个桶

```
func growWork(t *maptype, h *hmap, bucket uintptr) {
    // 为了确认搬迁的 bucket 是我们正在使用的 bucket
    // 即如果当前key映射到老的bucket1, 那么就搬迁该bucket1.
    evacuate(t, h, bucket&h.oldbucketmask())

    // 如果还未完成扩容工作，则再搬迁一个bucket.
    if h.growing() {
        evacuate(t, h, h.nevacuate)
    }
}
```

// 扩容, 启动 map 扩容操作. 只是完成了 hmap 元数据的复制, 但是底层的 bucket 没有进行搬移.

```
func hashGrow(t *maptype, h *hmap) {
    // 如果达到条件 1, 那么将B值加1, 相当于是原来的2倍
    // 否则对应条件 2, 进行等量扩容, 所以 B 不变
    bigger := uint8(1)
    if !overLoadFactor(h.count+1, h.B) {
        bigger = 0
        h.flags |= sameSizeGrow
    }

    // 记录老的buckets
    oldbuckets := h.buckets
    // 申请新的buckets空间, 可能包括部分溢出桶
    newbuckets, nextOverflow := makeBucketArray(t, h.B+bigger, nil)
    // A  &^ B, 与非操作, 移除 B 当中相应的标记位1
    // A  ^  B, 异或操作, 移除 A,B 共有的标记, 添加 A,B 不共有的标记
    // A  |  B, 或操作,   添加 B 当中相应的标记位1
    // A  &  B, 与操作,   寻找 A,B 共有的标记
    // 注意 &^ 运算符("与非"), 这块代码的逻辑是转移标志位(移除iterator, oldIterator). 
    flags := h.flags &^ (iterator | oldIterator)
    if h.flags&iterator != 0 {
        flags |= oldIterator // 当 old flags 存在iterator, 需要在new flags 当中添加 oldIterator
    }

    // 提交grow (atomic wrt gc)
    h.B += bigger
    h.flags = flags
    h.oldbuckets = oldbuckets
    h.buckets = newbuckets
    // 搬迁进度为0
    h.nevacuate = 0
    // overflow buckets 数为0
    h.noverflow = 0

    // 如果发现hmap是通过extra字段来存储 overflow buckets时
    if h.extra != nil && h.extra.overflow != nil {
        if h.extra.oldoverflow != nil {
            throw("oldoverflow is not nil")
        }
        h.extra.oldoverflow = h.extra.overflow
        h.extra.overflow = nil
    }
    if nextOverflow != nil {
        if h.extra == nil {
            h.extra = new(mapextra)
        }
        h.extra.nextOverflow = nextOverflow
    }
}
```

// 搬移操作, 一次只能迁移给一个 bucket

```
// oldbucket 表示老的 bucket 的索引(第几个桶)
func evacuate(t *maptype, h *hmap, oldbucket uintptr) {
    // 首先定位老的bucket的地址
    b := (*bmap)(add(h.oldbuckets, oldbucket*uintptr(t.bucketsize)))
    // newbit代表扩容之前老的bucket个数
    newbit := h.noldbuckets()
    // 判断该bucket是否已经被搬迁
    if !evacuated(b) {
        // 官方TODO, 后续版本也许会实现
        // TODO: reuse overflow buckets instead of using new ones, if there
        // is no iterator using the old buckets.  (If !oldIterator.)

        // evacDst 的含义很重要
        // xy 包含了高低区间的搬迁目的地内存信息. [bucket, index, key, value]
        // x.b 是对应的搬迁目的桶
        // x.k 是指向对应目的桶中存储当前key的内存地址. 
        // x.e 是指向对应目的桶中存储当前value的内存地址
        var xy [2]evacDst
        // 这里的 x 是第一个k,v
        x := &xy[0]
        x.b = (*bmap)(add(h.buckets, oldbucket*uintptr(t.bucketsize)))
        x.k = add(unsafe.Pointer(x.b), dataOffset)
        x.e = add(x.k, bucketCnt*uintptr(t.keysize))

        // 只有当增量扩容时才计算bucket y的相关信息(和后续计算useY相呼应)
        if !h.sameSizeGrow() {
            y := &xy[1]
            y.b = (*bmap)(add(h.buckets, (oldbucket+newbit)*uintptr(t.bucketsize)))
            y.k = add(unsafe.Pointer(y.b), dataOffset)
            y.e = add(y.k, bucketCnt*uintptr(t.keysize))
        }

        // evacuate 函数每次只完成一个 bucket 的搬迁工作, 因此要遍历完此 bucket 的所有的 cell, 将有值的 cell 
        // copy 到新的地方.
        // bucket 还会链接 overflow bucket, 它们同样需要搬迁.
        // 因此同样会有 2 层循环, 外层遍历 bucket 和 overflow bucket; 内层遍历 bucket 的所有 cell;

        // 遍历当前桶bucket和其之后的溢出桶overflow bucket
        // 注意: 初始的b是待搬迁的老bucket
        for ; b != nil; b = b.overflow(t) {
            k := add(unsafe.Pointer(b), dataOffset)
            e := add(k, bucketCnt*uintptr(t.keysize))
            // 遍历桶中的cell, i, k, e分别用于对应tophash, key和value
            for i := 0; i < bucketCnt; i, k, e = i+1, add(k, uintptr(t.keysize)), add(e, uintptr(t.elemsize)) {
                top := b.tophash[i]
                // 如果当前cell的tophash值是emptyOne或者emptyRest, 则代表此cell没有key.
                // 并将其标记为evacuatedEmpty, 表示它"已经被搬迁".
                if isEmpty(top) {
                    b.tophash[i] = evacuatedEmpty
                    continue
                }
                // 正常不会出现这种情况
                // 未被搬迁的 cell 只可能是emptyOne, emptyRest或是正常的 top hash(大于等于 minTopHash)
                if top < minTopHash {
                    throw("bad map state")
                }

                // 注意: 这是进行一次拷贝, 避免相同内存地址的问题
                k2 := k
                // 如果 key 是指针, 则解引用
                if t.indirectkey() {
                    k2 = *((*unsafe.Pointer)(k2))
                }

                var useY uint8
                // 如果是增量扩容
                if !h.sameSizeGrow() {
                    // 计算哈希值, 判断当前key和vale是要被搬迁到bucket x还是bucket y
                    hash := t.hasher(k2, uintptr(h.hash0))
                    // reflexivekey() // true if k==k for all keys
                    if h.flags&iterator != 0 && !t.reflexivekey() && !t.key.equal(k2, k2) {
                        // 有一个特殊情况: 有一种 key, 每次对它计算 hash, 得到的结果都不一样.
                        // 这个 key 就是 math.NaN() 的结果, 它的含义是 not a number, 类型是 float64.
                        // 当它作为 map 的 key时, 会遇到一个问题: 再次计算它的哈希值和它当初插入 map 时的计算出来的哈希值不一样!
                        // 这个 key 是永远不会被 Get 操作获取的! 当使用 m[math.NaN()] 语句的时候, 是查不出来结果的.
                        // 这个 key 只有在遍历整个 map 的时候, 才能被找到.
                        // 并且, 可以向一个 map 插入多个数量的 math.NaN() 作为 key, 它们并不会被互相覆盖.
                        // 当搬迁碰到 math.NaN() 的 key 时, 只通过 tophash 的最低位决定分配到 X part 还
                        // 是 Y part (如果扩容后是原来 buckets 数量的 2 倍). 
                        // 如果 tophash 的最低位是 0, 分配到 X part; 如果是 1, 则分配到 Y part.
                        useY = top & 1
                        top = tophash(hash)
                    } else {
                        // 对于正常key.
                        if hash&newbit != 0 {
                            useY = 1
                        }
                    }
                }

                if evacuatedX+1 != evacuatedY || evacuatedX^1 != evacuatedY {
                    throw("bad evacuatedN")
                }

                // 注: 标记oldbuckets的topHash, evacuatedX + 1 == evacuatedY
                b.tophash[i] = evacuatedX + useY
                // useY要么为0, 要么为1. 这里就是选取在bucket x的起始内存位置, 或者选择在bucket y的起始内存位置
                // (只有增量同步才会有这个选择可能).
                dst := &xy[useY]

                // 如果目的地的桶已经装满了(8个cell), 那么需要新建一个溢出桶, 继续搬迁到溢出桶上去.
                if dst.i == bucketCnt {
                    // 注意: newoverflow() 当中已经将当前创建好的 overflow bucket 设置到 bucket 上了. 
                    dst.b = h.newoverflow(t, dst.b) 
                    dst.i = 0
                    dst.k = add(unsafe.Pointer(dst.b), dataOffset)
                    dst.e = add(dst.k, bucketCnt*uintptr(t.keysize))
                }

                // dst.i 是依次递增的, 那么它的位置也是依次递增的
                dst.b.tophash[dst.i&(bucketCnt-1)] = top
                if t.indirectkey() {
                    // 如果待搬迁的key是指针, 则复制指针过去
                    *(*unsafe.Pointer)(dst.k) = k2 // copy pointer
                } else {
                    // 如果待搬迁的key是值, 则复制值过去  
                    typedmemmove(t.key, dst.k, k) // copy elem
                }
                // value和key同理
                if t.indirectelem() {
                    *(*unsafe.Pointer)(dst.e) = *(*unsafe.Pointer)(e)
                } else {
                    typedmemmove(t.elem, dst.e, e)
                }

                // 将当前搬迁目的桶的记录key/value的索引值(也可以理解为cell的索引值)加一
                dst.i++

                // 计算下一个k, e的内存地址
                // 由于桶的内存布局中在最后还有overflow的指针, 所以这里不用担心更新有可能会超出key和value数组的指
                // 针地址.
                dst.k = add(dst.k, uintptr(t.keysize))
                dst.e = add(dst.e, uintptr(t.elemsize))
            }
        }

        // 如果没有协程在使用老的桶, 就对老的桶进行清理, 用于帮助gc
        if h.flags&oldIterator == 0 && t.bucket.ptrdata != 0 {
            // 注意: 这里的 b 是私有局部变量. 要和循环当中的 b 区别开来
            b := add(h.oldbuckets, oldbucket*uintptr(t.bucketsize))
            // 只清除bucket 的 key,value 部分, 保留 top hash 部分, 指示搬迁状态
            ptr := add(b, dataOffset)
            n := uintptr(t.bucketsize) - dataOffset
            memclrHasPointers(ptr, n)
        }
    }

    // 更新搬移进度
    if oldbucket == h.nevacuate {
        advanceEvacuationMark(h, t, newbit)
    }
}
// 更新搬移进度
func advanceEvacuationMark(h *hmap, t *maptype, newbit uintptr) {
    // 搬迁桶的进度加一
    h.nevacuate++
    // 实验表明, 1024至少会比newbit高出一个数量级 (newbit代表扩容之前老的bucket个数). 
    // 所以, 用当前进度加上1024用于确保O(1)行为.
    stop := h.nevacuate + 1024
    if stop > newbit {
        stop = newbit
    }
    // 计算已经搬迁完的桶数
    for h.nevacuate != stop && bucketEvacuated(t, h, h.nevacuate) {
        h.nevacuate++
    }

    // 如果h.nevacuate == newbit, 则代表所有的桶都已经搬迁完毕
    if h.nevacuate == newbit {
        // 搬迁完毕，所以指向老的buckets的指针置为nil
        h.oldbuckets = nil
        // 在讲解hmap的结构中, 有过说明. 如果key和value均不包含指针, 且可以inline.
        // 那么保存它们的buckets数组其实是挂在hmap.extra中的. 
        // 所以, 这种情况下, 其实我们是搬迁的extra的buckets数组. 因此, 在这种情况下, 需要在搬迁完毕后, 将
        // hmap.extra.oldoverflow指针置为nil.
        if h.extra != nil {
            h.extra.oldoverflow = nil
        }
        // 最后, 清除正在扩容的标志位, 扩容完毕.
        h.flags &^= sameSizeGrow
    }
}
```

增量扩容下 map 搬移的图解, 其中 B=3, oldbuckets 数组是 8. buckets 数组是 16

![](https://pic2.zhimg.com/v2-b9ec34eb72d442b4fe95717272781575_b.jpg)

等量扩容下 map 搬移的图解, 其中 B=3, oldbuckets, buckets 数组都是 8

![](https://pic1.zhimg.com/v2-810315f7da2ad54fe3ff34dc87b0e18c_b.jpg)

map 删除
------

```
func mapdelete(t *maptype, h *hmap, key unsafe.Pointer) {
    // 如果开启了竞态检测 -race
    if raceenabled && h != nil {
        callerpc := getcallerpc()
        pc := funcPC(mapdelete)
        racewritepc(unsafe.Pointer(h), callerpc, pc)
        raceReadObjectPC(t.key, key, callerpc, pc)
    }
    // 如果开启了memory sanitizer -msan
    if msanenabled && h != nil {
        msanread(key, t.key.size)
    }
    // 如果map为空或者元素个数为0, 直接返回
    if h == nil || h.count == 0 {
        if t.hashMightPanic() {
            t.hasher(key, 0) // see issue 23734
        }
        return
    }

    // 当h.flags对应的值为hashWriting (代表有其他goroutine正在往map中写key)时, 那么位计算的结果不为0, 因此抛出以
    // 下错误.
    if h.flags&hashWriting != 0 {
        throw("concurrent map writes")
    }

    hash := t.hasher(key, uintptr(h.hash0))

    // 将flags的值与hashWriting做按位 "异或" 运算
    // 调用t.hasher后设置hashWriting, 因为t.hasher可能会 panic, 在这种情况下, 我们实际上并没有执行写(删除)操作.
    h.flags ^= hashWriting

    // 计算出桶的位置
    bucket := hash & bucketMask(h.B)
    if h.growing() {
        growWork(t, h, bucket)
    }

    // 获取 bucket 的内存地址
    b := (*bmap)(add(h.buckets, bucket*uintptr(t.bucketsize)))
    bOrig := b
    top := tophash(hash) // hash高8位

    // 以下是查找的核心逻辑
    // 双重循环遍历: 外层循环是从桶到溢出桶遍历; 内层是桶中的cell遍历
    // 跳出循环的条件有三种: 
    // 第一种是已经找到key值, 并且已经完成清理工作.
    // 第二种是当前桶再无溢出桶;
    // 第三种是当前桶中有cell位的tophash值是emptyRest, 这个值在前面解释过, 它代表此时的桶后面的cell还未利用, 
    // 所以无需再继续遍历. 
search:

    // 第二种情况
    for ; b != nil; b = b.overflow(t) {
        for i := uintptr(0); i < bucketCnt; i++ {
            if b.tophash[i] != top {
                // 第三种情况
                if b.tophash[i] == emptyRest {
                    break search
                }
                continue
            }
            k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
            k2 := k
            if t.indirectkey() {
                k2 = *((*unsafe.Pointer)(k2))
            }
            if !t.key.equal(key, k2) {
                continue
            }

            // 第一种情况, 说明已经找到了 key 值完全一样
            // 清理 key
            if t.indirectkey() {
                *(*unsafe.Pointer)(k) = nil
            } else if t.key.ptrdata != 0 {
                memclrHasPointers(k, t.key.size)
            }
            // 清理 value
            e := add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.elemsize))
            if t.indirectelem() {
                *(*unsafe.Pointer)(e) = nil
            } else if t.elem.ptrdata != 0 {
                memclrHasPointers(e, t.elem.size)
            } else {
                memclrNoHeapPointers(e, t.elem.size)
            }
            // 设置 tophash
            b.tophash[i] = emptyOne

            // 如果 bucket 现在以一堆emptyOne状态结束, 将其更改为emptyRest状态.
            // 将此功能设为一个单独的函数会很好, 但是for循环当前不可内联.
            // 可以立即结束循环的的两种状况:
            // 情况1: 当前 cell 是 bucket 的最有一个 cell, 且后续的 overflow bucket 的 cell tophash 不为 emptyRest
            // 情况2: 当前 cell 后续的 cell tophash 不为 emptyRest
            if i == bucketCnt-1 {
                // 情况1
                if b.overflow(t) != nil && b.overflow(t).tophash[0] != emptyRest {
                    goto notLast
                }
            } else {
                // 情况2
                if b.tophash[i+1] != emptyRest {
                    goto notLast
                }
            }

            // 如果 bucket 现在以一堆emptyOne状态结束, 将其更改为emptyRest状态.
            // 在这里存在两种情况:
            // 跳出本循环的两种情况:
            // 1. 遇到桶内的第一个 bucket. 注意: 桶实质上就是一个单向的链表.
            // 2. 遇到 cell 的 tophash 非删除状态(emptyOne)
            for {
                b.tophash[i] = emptyRest
                if i == 0 {
                    // 回到桶开始的位置
                    if b == bOrig {
                        break 
                    }
                    // 获取当前 bucket 的前面的 prev bucket(即 prev bucket 的 overflow 是当前 bucket)
                    // 每次都是从桶内的首个元素开始
                    c := b
                    for b = bOrig; b.overflow(t) != c; b = b.overflow(t) {
                    }
                    i = bucketCnt - 1
                } else {
                    i--
                }

                // 首个非 emptyOne 
                if b.tophash[i] != emptyOne {
                    break
                }
            }
        notLast:
            h.count--
            break search
        }
    }

    if h.flags&hashWriting == 0 {
        throw("concurrent map writes")
    }

    // 清除 hashWriting flag
    h.flags &^= hashWriting
}
```

map 迭代
------

```
// mapiterinit 初始化用于在 map 上进行遍历的hiter结构.
// it 指向的hiter结构由编译器顺序传递在堆栈上分配, 或者由 reflect_mapiterinit 在堆上分配.
// 由于结构包含指针, 因此两者都需要将hiter归零.
func mapiterinit(t *maptype, h *hmap, it *hiter) {
     // 如果开启了竞态检测 -race
    if raceenabled && h != nil {
        callerpc := getcallerpc()
        racereadpc(unsafe.Pointer(h), callerpc, funcPC(mapiterinit))
    }

    // hmap 不存在 或者 hmap 没有存储数据
    if h == nil || h.count == 0 {
        return
    }

    // hiter 的大小是 12 个系统指针大小. 在 cmd/compile/internal/gc/reflect.go:hiter() 当中有这样的体现
    if unsafe.Sizeof(hiter{})/sys.PtrSize != 12 {
        throw("hash_iter size incorrect") // see cmd/compile/internal/gc/reflect.go
    }
    it.t = t
    it.h = h

    // 抓取桶状态快照
    it.B = h.B
    it.buckets = h.buckets
    if t.bucket.ptrdata == 0 {
        // 重新分配 overflow, 并在 hiter 中存储指向 overflow 和 oldoverflow.
        // 这样在迭代的过程中, 可以让 overflow bucket 处于活动状态, 不论 table 的增长 and/or 新的 overflow
        // buckets 被添加到table当中
        h.createOverflow()
        it.overflow = h.extra.overflow
        it.oldoverflow = h.extra.oldoverflow
    }

    // 确定从哪里开始遍历, 这也是map遍历每次结果都一样的原因
    r := uintptr(fastrand()) // 随机生成一个整数
    if h.B > 31-bucketCntBits { 
        r += uintptr(fastrand()) << 31 // 在B>28时, 增加一个偏移量
    }
    it.startBucket = r & bucketMask(h.B) // 开始 bucket 的 index
    it.offset = uint8(r >> h.B & (bucketCnt - 1)) // 开始的 cell 位置(也是随机数[0-7])

    // iterator state
    it.bucket = it.startBucket

    // 多个迭代器可以同事运行.
    if old := h.flags; old&(iterator|oldIterator) != iterator|oldIterator {
        atomic.Or8(&h.flags, iterator|oldIterator) // 或操作
    }

    mapiternext(it)
}

// 迭代
func mapiternext(it *hiter) {
    h := it.h

    // 如果开启了竞态检测 -race
    if raceenabled {
        callerpc := getcallerpc()
        racereadpc(unsafe.Pointer(h), callerpc, funcPC(mapiternext))
    }

    // 并发访问和写入问题
    if h.flags&hashWriting != 0 {
        throw("concurrent map iteration and map write")
    }
    t := it.t
    bucket := it.bucket
    b := it.bptr
    i := it.i
    checkBucket := it.checkBucket

next:
    // 迭代操作
    // current bucket 为 nil, 第一次或者最后一次迭代
    if b == nil {
        // 当前的 bucket 是开始的 bucket 并且已经遍历过了
        if bucket == it.startBucket && it.wrapped {
            it.key = nil
            it.elem = nil
            return
        }

        // 获取 b (*bmap) 的真实值
        if h.growing() && it.B == h.B {
            // 迭代器是在扩容过程中启动的, 并且扩容过程尚未完成.
            // 如果我们要查看的存储桶尚未装满(即old bucket尚未搬移), 则我们需要遍历old bucket, 只返回将要迁移到
            // 该 bucket 的 cell.
            oldbucket := bucket & it.h.oldbucketmask()
            b = (*bmap)(add(h.oldbuckets, oldbucket*uintptr(t.bucketsize)))
            if !evacuated(b) {
                checkBucket = bucket
            } else {
                b = (*bmap)(add(it.buckets, bucket*uintptr(t.bucketsize)))
                checkBucket = noCheck
            }
        } else {
            // 迭代器目前处于正常状态(扩容结束或者没有扩容发生)
            b = (*bmap)(add(it.buckets, bucket*uintptr(t.bucketsize)))
            checkBucket = noCheck
        }

        bucket++

        // 所有的 bucket 已经遍历完成
        if bucket == bucketShift(it.B) {
            bucket = 0
            it.wrapped = true 
        }
        i = 0
    }

    // 遍历选择的 b, 返回一对 k,v
    for ; i < bucketCnt; i++ {
        offi := (i + it.offset) & (bucketCnt - 1)
        // 当前的 cell 状态是 emptyRest, emptyOne(空), evacuatedEmpty(迁移前是emptyRest, emptyOne).
        if isEmpty(b.tophash[offi]) || b.tophash[offi] == evacuatedEmpty {
            continue
        }

        // 获取k,e 分别对应 key 和 value 的内存地址
        k := add(unsafe.Pointer(b), dataOffset+uintptr(offi)*uintptr(t.keysize))
        if t.indirectkey() {
            k = *((*unsafe.Pointer)(k))
        }

        e := add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+uintptr(offi)*uintptr(t.elemsize))

        // 去掉需要忽略的对象 
        if checkBucket != noCheck && !h.sameSizeGrow() {
            // 特殊情况: 迭代器是在增长到更大的大小期间启动的, 尚未完成扩容. 
            // 
            // 正在处理尚未搬移的oldbucket的存储桶. 至少, 当启动存储桶时, 它并没有被搬移.
            // 因此, 正在遍历oldbucket, 需要跳过将要转到另一个新bucket的所有键 (在增长过程中, 每个oldbucket会扩
            // 展为两个bucket).
            // 
            // reflexivekey() // true if k==k for all keys
            if t.reflexivekey() || t.key.equal(k, k) {
                // 如果oldbucket中的 cell 不是搬移到迭代中的当前新存储桶的, 则将其跳过.
                hash := t.hasher(k, uintptr(h.hash0))
                if hash&bucketMask(it.B) != checkBucket {
                    continue
                }
            } else {
                // 如果k！= k(NaNs), 则 hash 不可重复. 我们需要对迁移期间发送NaN的方向进行可重复且随机的选择.
                // 这里将使用低位的 tophash 来决定NaN的走法.
                // 注意: 这种情况就是为什么我们需要两个迁移值, 即evacuatedX和evacuatedY, 它们的低位不同.
                if checkBucket>>(it.B-1) != uintptr(b.tophash[offi]&1) {
                    continue
                }
            }
        }

        // 遍历, 获取对应的 k, v        
        if (b.tophash[offi] != evacuatedX && b.tophash[offi] != evacuatedY) ||
            !(t.reflexivekey() || t.key.equal(k, k)) {
            // 特殊情况: 
            // 在正常状况(没有发生map扩容[增量方式])下进行遍历 [也成为 golden data]; 
            // 或者
            // key != key (只能发生 key=NANs 的状况下), 这些key是没法更新和删除的, 只能在遍历的时候返回.
            it.key = k
            if t.indirectelem() {
                e = *((*unsafe.Pointer)(e))
            }
            it.elem = e
        } else {
            // 在扩容的状况下, 开启迭代
            // 增量扩容已经完成, 并且k全是正常的key(非NANs)
            rk, re := mapaccessK(t, h, k)
            if rk == nil {
                continue // key has been deleted, 需要再遍历一次
            }
            it.key = rk
            it.elem = re
        }

        // 后续的处理工作
        it.bucket = bucket // 当前的 bucket 
        if it.bptr != b { // avoid unnecessary write barrier; see issue 14921
            it.bptr = b
        }
        it.i = i + 1 // cell 迭代器
        it.checkBucket = checkBucket
        return
    }

    b = b.overflow(t) // overflow bucket
    i = 0
    goto next
}
```

**阅读源码后的感慨:**

看了 map 的底层设计源码, 感觉 go 开发人员太牛掰了, 能够想到巧妙的数据结构, 数据结构不仅仅考量了如何去节省内存, 还考虑了这些字段在不同的状况下如何复用, 以及怎样获得一个更好的性能, 真的是让人太感叹了.

其次, go 底层 map 的设计思路也是衍生了一大批优秀的缓存库, 这些缓存库或多或少都有收到 map 的设计思想的影响. 后面有时间了, 我会详细介绍相关的 go 缓存库的源代码, 为大家分享一些自己的经验.
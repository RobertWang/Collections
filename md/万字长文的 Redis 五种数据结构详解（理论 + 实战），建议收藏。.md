> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1630400976&ver=3286&signature=VMTuUaOiSwW7XFqtP3Xk73n0vZ*XEszzXGHAcL0JGHKaE*VxZLPLQbz-sKapWOQ-zNazkuJ40wJ8RPiK2ybarPhDdnGD05-QCO*MMl5m5UivO0QYgzQorljyKd7B-i7t&new=1)

本文脑图
----

![](https://mmbiz.qpic.cn/mmbiz_png/IJUXwBNpKljaR3azNeZjTjDWSGVgurqKH1kM2TM5jyTZ2V1Nz5qIgymJibktHJeFqksKzGrqSraGFGPOtkn97mA/640?wx_fmt=png)

前言
--

Redis 是基于 c 语言编写的开源非关系型内存数据库，可以用作数据库、缓存、消息中间件，这么优秀的东西一定要一点一点的吃透它。

这篇文章主要讲解 Redis 的五种数据结构详解，包括这五种的数据结构的底层原理实现。

理论肯定是要用于实践的，因此最重要的还是实战部分，也就是这里还会讲解五种数据结构的应用场景。

话不多说，我们直接进入主题，很多人都知道 Redis 的五种数据结构包括以下五种：

1.  `String`：字符串类型
    
2.  `List`：列表类型
    
3.  `Set`：无序集合类型
    
4.  `ZSet`：有序集合类型
    
5.  `Hash`：哈希表类型
    

但是作为一名优秀的程序员可能不能只停留在只会用这五种类型进行 crud 工作，还是得深入了解这五种数据结构的底层原理。

Redis 核心对象
----------

![](https://mmbiz.qpic.cn/mmbiz_png/IJUXwBNpKljaR3azNeZjTjDWSGVgurqKnm5xJCicy64OM9nrUt5XuHCw5nTKxQcJOVJiao3l5dXuAkOgbThM5riaw/640?wx_fmt=png)

闪瞎人的五颜六色图

![](https://mmbiz.qpic.cn/mmbiz_png/IJUXwBNpKljaR3azNeZjTjDWSGVgurqKRqL4ewSuIJhGIG2VR33WBJ5VXAbicuPQrDZwbc2MygZFQcEUKhy0hDg/640?wx_fmt=png)

图片截图出自《Redis 设计与实现第二版》

![](https://mmbiz.qpic.cn/mmbiz_png/IJUXwBNpKljaR3azNeZjTjDWSGVgurqKrfySb4EibyytPZXhJoQbHTwTF9ourIUIjdud2TO9keVM2oNqkn6ibRKA/640?wx_fmt=png)

String 类型
---------

### int

![](https://mmbiz.qpic.cn/mmbiz_png/IJUXwBNpKljaR3azNeZjTjDWSGVgurqKvia2KMibx20j5ED54HrhNGIDk8434UONrl8HZnHpHmiaxLkSYqZMiawaYg/640?wx_fmt=png)

### SDS

SDS 称为**「简单动态字符串」**，对于 SDS 中的定义在 Redis 的源码中有的三个属性`int len、int free、char buf[]`。

len 保存了字符串的长度，free 表示 buf 数组中未使用的字节数量，buf 数组则是保存字符串的每一个字符元素。

因此当你在 Redsi 中存储一个字符串 Hello 时，根据 Redis 的源代码的描述可以画出 SDS 的形式的 redisObject 结构图如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/IJUXwBNpKljaR3azNeZjTjDWSGVgurqKqN1Fa4Ct11UhqlMweK8lxK6flYvtUX1peXXkmAIFBbFUZ0X7tBFicMw/640?wx_fmt=png)

### SDS 与 c 语言字符串对比

Redis 使用 SDS 作为存储字符串的类型肯定是有自己的优势，SDS 与 c 语言的字符串相比，SDS 对 c 语言的字符串做了自己的设计和优化，具体优势有以下几点：

（1）c 语言中的字符串并不会记录自己的长度，因此**「每次获取字符串的长度都会遍历得到，时间的复杂度是 O(n)」**，而 Redis 中获取字符串只要读取 len 的值就可，时间复杂度变为 O(1)。

（2）**「c 语言」**中两个字符串拼接，若是没有分配足够长度的内存空间就**「会出现缓冲区溢出的情况」**；而**「SDS」**会先根据 len 属性判断空间是否满足要求，若是空间不够，就会进行相应的空间扩展，所以**「不会出现缓冲区溢出的情况」**。

（3）SDS 还提供**「空间预分配」**和**「惰性空间释放」**两种策略。在为字符串分配空间时，分配的空间比实际要多，这样就能**「减少连续的执行字符串增长带来内存重新分配的次数」**。

当字符串被缩短的时候，SDS 也不会立即回收不适用的空间，而是通过`free`属性将不使用的空间记录下来，等后面使用的时候再释放。

具体的空间预分配原则是：**「当修改字符串后的长度 len 小于 1MB，就会预分配和 len 一样长度的空间，即 len=free；若是 len 大于 1MB，free 分配的空间大小就为 1MB」**。

（4）SDS 是二进制安全的，除了可以储存字符串以外还可以储存二进制文件（如图片、音频，视频等文件的二进制数据）；而 c 语言中的字符串是以空字符串作为结束符，一些图片中含有结束符，因此不是二进制安全的。

为了方便易懂，做了一个 c 语言的字符串和 SDS 进行对比的表格，如下所示：

<table data-tool="mdnice编辑器" data-darkmode-bgcolor-16304009793518="rgba(112, 0, 0, 0.039999999999999994)" data-darkmode-original-bgcolor-16304009793518="#fff|rgba(50, 0, 0, 0.039999999999999994)" data-darkmode-color-16304009793518="rgb(163, 163, 163)" data-darkmode-original-color-16304009793518="#fff|rgb(43, 43, 43)" class=""><thead data-darkmode-bgcolor-16304009793518="rgba(112, 0, 0, 0.039999999999999994)" data-darkmode-original-bgcolor-16304009793518="#fff|rgba(50, 0, 0, 0.039999999999999994)" data-darkmode-color-16304009793518="rgb(163, 163, 163)" data-darkmode-original-color-16304009793518="#fff|rgb(43, 43, 43)"><tr data-darkmode-bgcolor-16304009793518="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16304009793518="#fff|rgba(50, 0, 0, 0.039999999999999994)|rgb(255,255,255)" data-darkmode-color-16304009793518="rgb(163, 163, 163)" data-darkmode-original-color-16304009793518="#fff|rgb(43, 43, 43)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-darkmode-bgcolor-16304009793518="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16304009793518="#fff|rgba(50, 0, 0, 0.039999999999999994)|rgb(255,255,255)|rgb(240, 240, 240)" data-darkmode-color-16304009793518="rgb(141, 141, 141)" data-darkmode-original-color-16304009793518="#fff|rgb(43, 43, 43)|rgb(89, 89, 89)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); font-size: 14px; color: rgb(89, 89, 89);">c 语言字符串</th><th data-darkmode-bgcolor-16304009793518="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16304009793518="#fff|rgba(50, 0, 0, 0.039999999999999994)|rgb(255,255,255)|rgb(240, 240, 240)" data-darkmode-color-16304009793518="rgb(141, 141, 141)" data-darkmode-original-color-16304009793518="#fff|rgb(43, 43, 43)|rgb(89, 89, 89)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); font-size: 14px; color: rgb(89, 89, 89);" class="">SDS</th></tr></thead><tbody data-darkmode-bgcolor-16304009793518="rgba(112, 0, 0, 0.039999999999999994)" data-darkmode-original-bgcolor-16304009793518="#fff|rgba(50, 0, 0, 0.039999999999999994)" data-darkmode-color-16304009793518="rgb(163, 163, 163)" data-darkmode-original-color-16304009793518="#fff|rgb(43, 43, 43)"><tr data-darkmode-bgcolor-16304009793518="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16304009793518="#fff|rgba(50, 0, 0, 0.039999999999999994)|rgb(255,255,255)" data-darkmode-color-16304009793518="rgb(163, 163, 163)" data-darkmode-original-color-16304009793518="#fff|rgb(43, 43, 43)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-bgcolor-16304009793518="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16304009793518="#fff|rgba(50, 0, 0, 0.039999999999999994)|rgb(255,255,255)" data-darkmode-color-16304009793518="rgb(141, 141, 141)" data-darkmode-original-color-16304009793518="#fff|rgb(43, 43, 43)|rgb(89, 89, 89)" data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89);">获取长度的时间复杂度为 O(n)</td><td data-darkmode-bgcolor-16304009793518="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16304009793518="#fff|rgba(50, 0, 0, 0.039999999999999994)|rgb(255,255,255)" data-darkmode-color-16304009793518="rgb(141, 141, 141)" data-darkmode-original-color-16304009793518="#fff|rgb(43, 43, 43)|rgb(89, 89, 89)" data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89);" class="">获取长度的时间复杂度为 O(1)</td></tr><tr data-darkmode-bgcolor-16304009793518="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16304009793518="#fff|rgba(50, 0, 0, 0.039999999999999994)|rgb(248, 248, 248)" data-darkmode-color-16304009793518="rgb(163, 163, 163)" data-darkmode-original-color-16304009793518="#fff|rgb(43, 43, 43)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-bgcolor-16304009793518="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16304009793518="#fff|rgba(50, 0, 0, 0.039999999999999994)|rgb(248, 248, 248)" data-darkmode-color-16304009793518="rgb(141, 141, 141)" data-darkmode-original-color-16304009793518="#fff|rgb(43, 43, 43)|rgb(89, 89, 89)" data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89);">不是二进制安全的</td><td data-darkmode-bgcolor-16304009793518="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16304009793518="#fff|rgba(50, 0, 0, 0.039999999999999994)|rgb(248, 248, 248)" data-darkmode-color-16304009793518="rgb(141, 141, 141)" data-darkmode-original-color-16304009793518="#fff|rgb(43, 43, 43)|rgb(89, 89, 89)" data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89);" class="">是二进制安全的</td></tr><tr data-darkmode-bgcolor-16304009793518="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16304009793518="#fff|rgba(50, 0, 0, 0.039999999999999994)|rgb(255,255,255)" data-darkmode-color-16304009793518="rgb(163, 163, 163)" data-darkmode-original-color-16304009793518="#fff|rgb(43, 43, 43)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-bgcolor-16304009793518="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16304009793518="#fff|rgba(50, 0, 0, 0.039999999999999994)|rgb(255,255,255)" data-darkmode-color-16304009793518="rgb(141, 141, 141)" data-darkmode-original-color-16304009793518="#fff|rgb(43, 43, 43)|rgb(89, 89, 89)" data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89);">只能保存字符串</td><td data-darkmode-bgcolor-16304009793518="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16304009793518="#fff|rgba(50, 0, 0, 0.039999999999999994)|rgb(255,255,255)" data-darkmode-color-16304009793518="rgb(141, 141, 141)" data-darkmode-original-color-16304009793518="#fff|rgb(43, 43, 43)|rgb(89, 89, 89)" data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89);" class="">还可以保存二进制数据</td></tr><tr data-darkmode-bgcolor-16304009793518="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16304009793518="#fff|rgba(50, 0, 0, 0.039999999999999994)|rgb(248, 248, 248)" data-darkmode-color-16304009793518="rgb(163, 163, 163)" data-darkmode-original-color-16304009793518="#fff|rgb(43, 43, 43)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-bgcolor-16304009793518="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16304009793518="#fff|rgba(50, 0, 0, 0.039999999999999994)|rgb(248, 248, 248)" data-darkmode-color-16304009793518="rgb(141, 141, 141)" data-darkmode-original-color-16304009793518="#fff|rgb(43, 43, 43)|rgb(89, 89, 89)" data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89);" class="">n 次增长字符串必然会带来 n 次的内存分配</td><td data-darkmode-bgcolor-16304009793518="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16304009793518="#fff|rgba(50, 0, 0, 0.039999999999999994)|rgb(248, 248, 248)" data-darkmode-color-16304009793518="rgb(141, 141, 141)" data-darkmode-original-color-16304009793518="#fff|rgb(43, 43, 43)|rgb(89, 89, 89)" data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89);" class="">n 次增长字符串内存分配的次数 &lt;=n</td></tr></tbody></table>

### String 类型应用

说到这里我相信很多人可以说已经精通 Redis 的 String 类型了，但是纯理论的精通，理论还是得应用实践，上面说到 String 可以用来存储图片，现在就以图片存储作为案例实现。

（1）首先要把上传得图片进行编码，这里写了一个工具类把图片处理成了 Base64 得编码形式，具体得实现代码如下：

（2）第二步就是把处理后的图片字符串格式存储进 Redis 中，实现的代码如下所示：

```
    /**
     * Redis存储图片
     * @param file
     * @return
     */
    public void uploadImageServiceImpl(MultipartFile image) {
        String imgId = UUID.randomUUID().toString();
        String imgStr= ImageUtils.encodeImg(image);
        redisUtils.set(imgId , imgStr);
        // 后续操作可以把imgId存进数据库对应的字段，如果需要从redis中取出，只要获取到这个字段后从redis中取出即可。
    }


```

这样就是实现了图片得二进制存储，当然 String 类型得数据结构得应用也还有常规计数：**「统计微博数、统计粉丝数」**等。

Hash 类型
-------

Hash 对象的实现方式有两种分别是`ziplist、hashtable`，其中 hashtable 的存储方式 key 是 String 类型的，value 也是以`key value`的形式进行存储。

字典类型的底层就是 hashtable 实现的，明白了字典的底层实现原理也就是明白了 hashtable 的实现原理，hashtable 的实现原理可以与 HashMap 的是底层原理相类比。

### 字典

两者在新增时都会通过 key 计算出数组下标，不同的是计算法方式不同，HashMap 中是以 hash 函数的方式，而 hashtable 中计算出 hash 值后，还要通过 sizemask 属性和哈希值再次得到数组下标。

我们知道 hash 表最大的问题就是 hash 冲突，为了解决 hash 冲突，假如 hashtable 中不同的 key 通过计算得到同一个 index，就会形成单向链表（**「链地址法」**），如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/IJUXwBNpKljaR3azNeZjTjDWSGVgurqKiaeKNehP9DPx0AqxmoXbjKWSkibxsbuL72FqdXzyy19bWsPX8pNJw43Q/640?wx_fmt=png)

#### rehash

在字典的底层实现中，value 对象以每一个 dictEntry 的对象进行存储，当 hash 表中的存放的键值对不断的增加或者减少时，需要对 hash 表进行一个扩展或者收缩。

这里就会和 HashMap 一样也会就进行 rehash 操作，进行重新散列排布。从上图中可以看到有`ht[0]`和`ht[1]`两个对象，先来看看对象中的属性是干嘛用的。

在 hash 表结构定义中有四个属性分别是`dictEntry **table、unsigned long size、unsigned long sizemask、unsigned long used`，分别表示的含义就是**「哈希表数组、hash 表大小、用于计算索引值，总是等于 size-1、hash 表中已有的节点数」**。

ht[0] 是用来最开始存储数据的，当要进行扩展或者收缩时，ht[0] 的大小就决定了 ht[1] 的大小，ht[0] 中的所有的键值对就会重新散列到 ht[1] 中。

#### 渐进式 rehash

### ziplist

![](https://mmbiz.qpic.cn/mmbiz_png/IJUXwBNpKljaR3azNeZjTjDWSGVgurqKictj8398DMBAG7YVytTdYuREicIxqViacg6vuZLh7EhshicvaA6kHEHTjg/640?wx_fmt=png)

1.  `zlbytes`：4 个字节的大小，记录压缩列表占用内存的字节数。
    
2.  `zltail`：4 个字节大小，记录表尾节点距离起始地址的偏移量，用于快速定位到尾节点的地址。
    
3.  `zllen`：2 个字节的大小，记录压缩列表中的节点数。
    
4.  `entry`：表示列表中的每一个节点。
    
5.  `zlend`：表示压缩列表的特殊结束符号`'0xFF'`。
    

1.  `previous_entry_ength`表示前一个节点 entry 的长度，可用于计算前一个节点的其实地址，因为他们的地址是连续的。
    
2.  encoding：这里保存的是 content 的内容类型和长度。
    
3.  content：content 保存的是每一个节点的内容。
    

![](https://mmbiz.qpic.cn/mmbiz_png/IJUXwBNpKljaR3azNeZjTjDWSGVgurqKnd0iatwup3dIxevURp1Y7ntZ9TPiaSpw8zGtnAPpG60v5VuR0BT88NGA/640?wx_fmt=png)

### 应用场景

哈希表相对于 String 类型存储信息更加直观，存储更加方便，经常会用来做用户数据的管理，存储用户的信息。

hash 也可以用作高并发场景下使用 Redis 生成唯一的 id。下面我们就以这两种场景用作案例编码实现。

#### 存储用户数据

第一个场景比如我们要储存用户信息，一般使用用户的 ID 作为 key 值，保持唯一性，用户的其他信息（地址、年龄、生日、电话号码等）作为 value 值存储。

若是传统的实现就是将用户的信息封装成为一个对象，通过序列化存储数据，当需要获取用户信息的时候，就会通过反序列化得到用户信息。

![](https://mmbiz.qpic.cn/mmbiz_png/IJUXwBNpKljaR3azNeZjTjDWSGVgurqK552yqSTyuibPcQRKJ7wIJ40oJ16qNZQOmicESJUPAnywbvUiczJlrZOyA/640?wx_fmt=png)

但是这样必然会造成序列化和反序列化的性能的开销，并且若是只修改其中的一个属性值，就需要把整个对象序列化出来，操作的动作太大，造成不必要的性能开销。

若是使用 Redis 的 hash 来存储用户数据，就会将原来的 value 值又看成了一个 k v 形式的存储容器，这样就不会带来序列化的性能开销的问题。

![](https://mmbiz.qpic.cn/mmbiz_png/IJUXwBNpKljaR3azNeZjTjDWSGVgurqKtpiarzPkoKxO2FlffQZhOceuuibFE8XxQfCRyz2siakpIyMnwcu5DQicnQ/640?wx_fmt=png)

#### 分布式生成唯一 ID

第二个场景就是生成分布式的唯一 ID，这个场景下就是把 redis 封装成了一个工具类进行实现，实现的代码如下：

```
    // offset表示的是id的递增梯度值
    public Long getId(String key,String hashKey,Long offset) throws BusinessException{
        try {
            if (null == offset) {
                offset=1L;
            }
            // 生成唯一id
            return redisUtil.increment(key, hashKey, offset);
        } catch (Exception e) {
            //若是出现异常就是用uuid来生成唯一的id值
            int randNo=UUID.randomUUID().toString().hashCode();
            if (randNo < 0) {
                randNo=-randNo;
            }
            return Long.valueOf(String.format("%16d", randNo));
        }
    }


```

List 类型
-------

Redis 中的列表在 3.2 之前的版本是使用`ziplist`和`linkedlist`进行实现的。在 3.2 之后的版本就是引入了`quicklist`。

ziplist 压缩列表上面已经讲过了，我们来看看 linkedlist 和 quicklist 的结构是怎么样的。

linkedlist 是一个双向链表，他和普通的链表一样都是由指向前后节点的指针。插入、修改、更新的时间复杂度尾 O(1)，但是查询的时间复杂度确实 O(n)。

linkedlist 和 quicklist 的底层实现是采用链表进行实现，在 c 语言中并没有内置的链表这种数据结构，Redis 实现了自己的链表结构。

![](https://mmbiz.qpic.cn/mmbiz_png/IJUXwBNpKljaR3azNeZjTjDWSGVgurqKiaEOVZXbwSL2YqoPYuD7r2kYUGONuX8omwt1sBr6bQmTjyQ40ftG7DA/640?wx_fmt=png)

Redis 中链表的特性：

1.  每一个节点都有指向前一个节点和后一个节点的指针。
    
2.  头节点和尾节点的 prev 和 next 指针指向为 null，所以链表是无环的。
    
3.  链表有自己长度的信息，获取长度的时间复杂度为 O(1)。
    

Redis 中 List 的实现比较简单，下面我们就来看看它的应用场景。

### 应用场景

Redis 中的列表可以实现**「阻塞队列」**，结合 lpush 和 brpop 命令就可以实现。生产者使用 lupsh 从列表的左侧插入元素，消费者使用 brpop 命令从队列的右侧获取元素进行消费。

（1）首先配置 redis 的配置，为了方便我就直接放在`application.yml`配置文件中，实际中可以把 redis 的配置文件放在一个`redis.properties`文件单独放置，具体配置如下：

```
spring
	redis:
		host: 127.0.0.1
		port: 6379
		password: user
		timeout: 0
		database: 2
		pool:
			max-active: 100
			max-idle: 10
			min-idle: 0
			max-wait: 100000


```

（2）第二步创建 redis 的配置类，叫做`RedisConfig`，并标注上`@Configuration`注解，表明他是一个配置类。

```
@Configuration
public class RedisConfiguration {

@Value("${spring.redis.host}")
private String host;
@Value("${spring.redis.port}")
private int port;
@Value("${spring.redis.password}")
private String password;
@Value("${spring.redis.pool.max-active}")
private int maxActive;
@Value("${spring.redis.pool.max-idle}")
private int maxIdle;
@Value("${spring.redis.pool.min-idle}")
private int minIdle;
@Value("${spring.redis.pool.max-wait}")
private int maxWait;
@Value("${spring.redis.database}")
private int database;
@Value("${spring.redis.timeout}")
private int timeout;

@Bean
public JedisPoolConfig getRedisConfiguration(){
	JedisPoolConfig jedisPoolConfig= new JedisPoolConfig();
	jedisPoolConfig.setMaxTotal(maxActive);
	jedisPoolConfig.setMaxIdle(maxIdle);
	jedisPoolConfig.setMinIdle(minIdle);
	jedisPoolConfig.setMaxWaitMillis(maxWait);
	return jedisPoolConfig;
}

@Bean
public JedisConnectionFactory getConnectionFactory() {
	JedisConnectionFactory factory = new JedisConnectionFactory();
	factory.setHostName(host);
	factory.setPort(port);
	factory.setPassword(password);
	factory.setDatabase(database);
	JedisPoolConfig jedisPoolConfig= getRedisConfiguration();
	factory.setPoolConfig(jedisPoolConfig);
	return factory;
}

@Bean
public RedisTemplate<?, ?> getRedisTemplate() {
	JedisConnectionFactory factory = getConnectionFactory();
	RedisTemplate<?, ?> redisTemplate = new StringRedisTemplate(factory);
	return redisTemplate;
}
}


```

（3）第三步就是创建 Redis 的工具类 RedisUtil，自从学了面向对象后，就喜欢把一些通用的东西拆成工具类，好像一个一个零件，需要的时候，就把它组装起来。

```
@Component
public class RedisUtil {

@Autowired
private RedisTemplate<String, Object> redisTemplate;
/**
* 存消息到消息队列中
* @param key 键
* @param value 值
* @return
*/
public boolean lPushMessage(String key, Object value) {
	try {
			redisTemplate.opsForList().leftPush(key, value);
			return true;
	} catch (Exception e) {
			e.printStackTrace();
			return false;
	}
}

/**
* 从消息队列中弹出消息
* @param key 键
* @return
*/
public Object rPopMessage(String key) {
	try {
			return redisTemplate.opsForList().rightPop(key);
	} catch (Exception e) {
			e.printStackTrace();
			return null;
	}
}

/**
* 查看消息
* @param key 键
* @param start 开始
* @param end 结束 0 到 -1代表所有值
* @return
*/
public List<Object> getMessage(String key, long start, long end) {
	try {
			return redisTemplate.opsForList().range(key, start, end);
	} catch (Exception e) {
			e.printStackTrace();
			return null;
	}
}


```

这样就完成了 Redis 消息队列工具类的创建，在后面的代码中就可以直接使用。

Set 集合
------

Redis 中列表和集合都可以用来存储字符串，但是**「Set 是不可重复的集合，而 List 列表可以存储相同的字符串」**，Set 集合是无序的这个和后面讲的 ZSet 有序集合相对。

Set 的底层实现是**「ht 和 intset」**，ht（哈希表）前面已经详细了解过，下面我们来看看 inset 类型的存储结构。

inset 也叫做整数集合，用于保存整数值的数据结构类型，它可以保存`int16_t`、`int32_t` 或者`int64_t` 的整数值。

在整数集合中，有三个属性值`encoding、length、contents[]`，分别表示编码方式、整数集合的长度、以及元素内容，length 就是记录 contents 里面的大小。

在整数集合新增元素的时候，若是超出了原集合的长度大小，就会对集合进行升级，具体的升级过程如下：

1.  首先扩展底层数组的大小，并且数组的类型为新元素的类型。
    
2.  然后将原来的数组中的元素转为新元素的类型，并放到扩展后数组对应的位置。
    
3.  整数集合升级后就不会再降级，编码会一直保持升级后的状态。
    

### 应用场景

Set 集合的应用场景可以用来**「去重、抽奖、共同好友、二度好友」**等业务类型。接下来模拟一个添加好友的案例实现：

```
@RequestMapping(value = "/addFriend", method = RequestMethod.POST)
public Long addFriend(User user, String friend) {
    String currentKey = null;
    // 判断是否是当前用户的好友
    if (AppContext.getCurrentUser().getId().equals(user.getId)) {
        currentKey = user.getId.toString();
    }
    //若是返回0则表示不是该用户好友
    return currentKey==null?0l:setOperations.add(currentKey, friend);
}


```

假如两个用户 A 和 B 都是用上上面的这个接口添加了很多的自己的好友，那么有一个需求就是要实现获取 A 和 B 的共同好友，那么可以进行如下操作：

```
public Set intersectFriend(User userA, User userB) {
    return setOperations.intersect(userA.getId.toString(), userB.getId.toString());
}


```

举一反三，还可以实现 A 用户自己的好友，或者 B 用户自己的好友等，都可以进行实现。

ZSet 集合
-------

ZSet 是有序集合，从上面的图中可以看到 ZSet 的底层实现是`ziplist`和`skiplist`实现的，ziplist 上面已经详细讲过，这里来讲解 skiplist 的结构实现。

`skiplist`也叫做**「跳跃表」**，跳跃表是一种有序的数据结构，它通过每一个节点维持多个指向其它节点的指针，从而达到快速访问的目的。

skiplist 有如下几个特点：

1.  有很多层组成，由上到下节点数逐渐密集，最上层的节点最稀疏，跨度也最大。
    
2.  每一层都是一个有序链表，至少包含两个节点，头节点和尾节点。
    
3.  每一层的每一个每一个节点都含有指向同一层下一个节点和下一层同一个位置节点的指针。
    
4.  如果一个节点在某一层出现，那么该以下的所有链表同一个位置都会出现该节点。
    

具体实现的结构图如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/IJUXwBNpKljaR3azNeZjTjDWSGVgurqKIhjgN4UcI6XI7ibKLiaokbKUwoBibsiaPtZJz9nSWWg4qzJKeTHP0ywBXw/640?wx_fmt=png)

在跳跃表的结构中有 head 和 tail 表示指向头节点和尾节点的指针，能快速的实现定位。level 表示层数，len 表示跳跃表的长度，BW 表示后退指针，在从尾向前遍历的时候使用。

BW 下面还有两个值分别表示分值（score）和成员对象（各个节点保存的成员对象）。

跳跃表的实现中，除了最底层的一层保存的是原始链表的完整数据，上层的节点数会越来越少，并且跨度会越来越大。

跳跃表的上面层就相当于索引层，都是为了找到最后的数据而服务的，数据量越大，条表所体现的查询的效率就越高，和平衡树的查询效率相差无几。

### 应用场景

因为 ZSet 是有序的集合，因此 ZSet 在实现排序类型的业务是比较常见的，比如在首页推荐 10 个最热门的帖子，也就是阅读量由高到低，排行榜的实现等业务。

下面就选用获取排行榜前前 10 名的选手作为案例实现，实现的代码如下所示：

```
@Autowired
private RedisTemplate redisTemplate;
	/**
	 * 获取前10排名
	 * @return
	 */
    public static List<levelVO > getZset(String key, long baseNum, LevelService levelService){
        ZSetOperations<Serializable, Object> operations = redisTemplate.opsForZSet();
        // 根据score分数值获取前10名的数据
        Set<ZSetOperations.TypedTuple<Object>> set = operations.reverseRangeWithScores(key,0,9);
        List<LevelVO> list= new ArrayList<LevelVO>();
        int i=1;
        for (ZSetOperations.TypedTuple<Object> o:set){
            int uid = (int) o.getValue();
            LevelCache levelCache = levelService.getLevelCache(uid);
            LevelVO levelVO = levelCache.getLevelVO();
            long score = (o.getScore().longValue() - baseNum + levelVO .getCtime())/CommonUtil.multiplier;
            levelVO .setScore(score);
            levelVO .setRank(i);
            list.add( levelVO );
            i++;
        }
        return list;
    }


```

以上的代码实现大致逻辑就是根据 score 分数值获取前 10 名的数据，然后封装成 lawyerVO 对象的列表进行返回。

到这里我们已经精通 Redis 的五种基本数据类型了，又可以去和面试官扯皮了，扯不过就跑路吧，或者这篇文章多看几遍，相信对你总是有好处的。

【文章参考】

[1]《Redis 设计与实现》

**-End-**
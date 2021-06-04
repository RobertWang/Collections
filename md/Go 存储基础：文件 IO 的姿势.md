> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666554193&idx=2&sn=9eb981f04b1a8c436eb9bc2570f6b0c9&chksm=80dca7fab7ab2eec9e6f9d44fb73ac782c970fca0ea92e436b15672c3ce3f9c4d8aa79b36316&mpshare=1&scene=1&srcid=0507TYEBXCvKDuspNvcpMHY5&sharer_sharetime=1620325703972&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

我们都知道计算的体系架构，CPU，内存，网络，IO。那么 IO 是啥呢？一般理解成 Input、Output 的缩写，通俗话就是输入输出的意思。  

IO 分为网络和存储 IO 两种类型（**其实****网络 IO 和磁盘 IO 在 Go 里面有着根本性区别，**以后会就此深入分析）。网络 IO 对应的是网络数据传输过程，网络是分布式系统的基石，通过网络把离散的物理节点连接起来，形成一个有机的系统。

存储 IO 对应的就是数据存储到物理介质的过程，通常我们物理介质对应的是磁盘，磁盘上一般会分个区，然后在上面格式化个文件系统出来，所以我们普通程序员最常看见的是文件 IO 的形式。

在 Golang 里可以归类出两种读写文件的方式：

1.  标准库封装：操作对象 `File`;
    
2.  系统调用 ：操作对象 `fd`;
    

首先我们回忆下，文件的读写最核心的要素是什么？  

通俗来讲：读文件，就是把磁盘上的文件的**特定位置**的数据读到**内存的 buffer** 。写文件，就是把**内存 buffer** 的数据写到磁盘的文件的**特定位置**。

这里注意到两个关键词：

1.  特定位置；
    
2.  内存 buffer；
    

**特定位置怎么理解？怎么指定所谓的`特定位置`？**

很简单，用 `[ offset, length ]` 这两个参数就能标识一段位置。

![](https://mmbiz.qpic.cn/mmbiz_png/4UtmXsuLoNeZJaicwWlXZJmV0jdkDH1t207UTf3kiaMIGibAfonHwT0UYw01rr8ibvT7xua85iarw3cI9SvxhaZnHaw/640?wx_fmt=png)  

也就是 IO 偏移和长度，Offset 和 Length。

**内存 buffer 怎么理解？**

归根结底，文件的数据和谁直接打交道？**内存**，写的时候是从内存写到磁盘文件的，读的时候是从磁盘文件读到内存的。

本质上，下面的 IO 函数都离不开 Offset，Length，buffer 这三个要素。

标准库封装

Go 对文件进行读写非常简单，因为 Go 已经帮我们封装了一个非常便捷的使用接口，位于标准库 os 中。Go 标准库对文件 IO 的封装也就是 Go 推荐我们对文件进行 IO 时使用的姿势。

打开文件（Open）


--------------

#### `func OpenFile(name string, flag int, perm FileMode) (*File, error)  
`

Open 文件之后，获取到一个句柄，也就是 `File` 结构，之后对文件的读写都是基于 `File` 结构之上进行的。

```
type File struct {
    *file // os specific
}


```

普通程序员如果不关系里面的实现，那么只需要知道，之后的文件读写只需要针对这个句柄结构体做操作即可。

另外有一点隐藏起来的知识点必须要提一下：**偏移**。也就是我们最开始强调的读写 3 要素之一的 Offset 。打开（`Open`）文件的时候，文件当前偏移量默认设置为 0，也就是说 IO 的起始位置就是文件的最开头。举个例子，如果这个时候，你写 4K 的数据到文件，那么就是写 [0, 4K] 这个位置的数据，如果之前这上面已经有数据了，那么就会是覆盖写。

除非你 `Open` 文件的时候指定 `O_APPEND` 选项，偏移量会设置为文件末尾，那么 IO 都是从文件末尾开始。

文件写操作（Write）


----------------

文件 `File` 句柄对象有两个写方法：

第一种：写一个 buffer 到文件 ，使用文件当前偏移

```
func (f *File) Write(b []byte) (n int, err error)


```

**注意：该写操作会导致文件偏移量的增加。**

第二种：从指定文件偏移，写入 buffer 到文件

```
func (f *File) WriteAt(b []byte, off int64) (n int, err error) 


```

**注意：该写操作不会更新文件偏移量**

文件读操作（Read）


---------------

和写对应，文件 `File` 句柄对象有两个读方法：

第一种：从文件当前偏移读一个 buffer 的数据上来

```
func (f *File) Read(b []byte) (n int, err error)


```

**注意：该读操作会导致文件偏移量的增加。**

第二种：从指定文件偏移，读一个 buffer 大小的数据上来

```
func (f *File) ReadAt(b []byte, off int64) (n int, err error)


```

**注意：该读操作不会更新文件偏移量**

指定偏移量（Seek）


---------------

#### `func (f *File) Seek(offset int64, whence int) (ret int64, err error)  
`

这个句柄方法允许用户指定文件的偏移位置。这个很容易理解，举个例子，文件刚开始是 0 字节，写 1M 的数据下去，大小变成 1M，Offset 往后挪 1M ，默认就是往后挪。

现在 Seek 方法允许你把写的偏移定位到任意位置，可以你就可以从任意地方覆盖写入数据。

所以在 Go 里面，文件 IO 非常简单，先 Open 一个文件，拿到 `File` 句柄，然后就可以使用这个句柄 Write ，Read，Seek 就能进行 IO 了。

底层的原理

Go 的标准库 `os` 给我们的极其方便的封装，但是，你不好奇这个 `os`  的封装底层的原理吗？我们深入最原始的本质，你会发现最核心的东西：**系统调用**。  

Go 标准库的文件存储 IO 就是基于系统调用之上的。可以稍微跟一下 `os.OpenFile` 的调用：

os 库的 `OpenFile` 函数：

```
func OpenFile(name string, flag int, perm FileMode) (*File, error) {
    f, err := openFileNolog(name, flag, perm)
    if err != nil {
        return nil, err
    }
    f.appendMode = flag&O_APPEND != 0

    return f, nil
}


```

稍微看下 `openFileNolog` 函数：

```
func openFileNolog(name string, flag int, perm FileMode) (*File, error) {

    var r int
    for {
        var e error
        r, e = syscall.Open(name, flag|syscall.O_CLOEXEC, syscallMode(perm))
        if e == nil {
            break
        }

        if runtime.GOOS == "darwin" && e == syscall.EINTR {
            continue
        }

        return nil, &PathError{"open", name, e}
    }

    return newFile(uintptr(r), name, kindOpenFile), nil
}


```

我们看到 `syscall.Open` ，这个函数获取到一个整数，也就是我们在在 c 语言里最常见的 fd 句柄，而 `File` 结构体则仅仅是基于这个的一层封装而已。

**思考下，为什么会有标准库封装这一层存在？**

**划重点：为了屏蔽操作系统的区别**，使用这个标准库的所有操作都是跨平台的。换句话说，如果是特殊操作系统才有的特性，那么你在 os 库里就找不到对应封装的 IO 操作。

![](https://mmbiz.qpic.cn/mmbiz_png/4UtmXsuLoNeZJaicwWlXZJmV0jdkDH1t2kyWa5Oz4X2n4usiaxB1KQFQeXlnfW53acNv8WDHZiazg66NfeicLpsib5w/640?wx_fmt=png)  

那么怎么使用系统调用？

直接使用 syscall 库，也就是系统调用。从名字也能看出来，系统调用是和操作系统强相关的，因为是操作系统提供给你的调用接口，所以系统调用会因为操作系统不同而导致不同的特性，不同的接口。

所以，如果你直接使用 syscall 库来使用系统调用，那么需要你自己来承受系统带来的兼容性问题。

系统调用

系统调用在 syscall 里有一层最薄的封装：

文件 Open


-----------

#### `func Open(path string, mode int, perm uint32) (fd int, err error)` 

####   

文件 Read


-----------

#### `func Read(fd int, p []byte) (n int, err error)   
func Pread(fd int, p []byte, offset int64) (n int, err error)` 

文件读有两个接口，一个 `Read` 是从**当前默认偏移**读一个 buffer 数据，`Pread` 接口则是从指定位置读数据的接口。

思考一个问题：`Pread` 从效果上来讲等于 `Seek` 和 `Read` 组合起来使用，那么是否可以认为 `Pread` 就可以被 `Seek` + `Read` 替代呢？

不行！根本原因在于 `Seek` + `Read` 是在用户层就是两步操作，而 `Pread` 虽然是 `Seek` + `Read` 的效果，但是操作系统给到用户的语义是：`Pread` 是一个原子操作。还有一个重要区别，`Pread` 不会改变当前文件的偏移量（普通的 `Read` 调用会更新偏移量）。

**所以，我们总结下，`Pread` 和顺序调用 `Seek` 后调用 `Read`  有两点重要区别：**

1.  `Pread` 对用户提供的语义是原子操作，在调用 `Pread` 时，无法中断 `Seek` 和 `Read` 操作；
    
2.  `Pread` 调用不会更新当前文件偏移量；
    

####   

文件 Write


------------

#### `func Write(fd int, p []byte) (n int, err error)   
func Pwrite(fd int, p []byte, offset int64) (n int, err error)` 

文件写对应也是有两种接口，`Wrtie` 和 `Pwrite` 分别是对应 `Read` 和 `Pread` 。同样的，`Pwrite` 作用上也是相当于先调用 `Seek`  再调用 `Write` ，但是同样的也有**两点不同**：

1.  `Pwrite`  完成 `Seek` 和 `Write` 对外是原子操作的语义；
    
2.  `Pwrite` 调用不会更新当前文件偏移量；
    

####   

文件 Seek


-----------

#### `func Seek(fd int, offset int64, whence int) (off int64, err error)` 

这个函数调用允许用户指定偏移（这个会影响到 `Read` 和 `Write` 读写的位置）。一般来说，每个打开文件都有一个相关联的 “当前文件偏移量”（ current file offset ）。读（`Read`）、写（`Write`）操作都是从**当前文件偏移量处**开始，并且 `Read` 和 `Write` 会导致偏移量增加，增加量就是所读写的字节数。

**小结一下**：我们看了核心的 Open，Read，Write，Seek 几个系统调用，你会发现一个明显不同与标准 IO 库的区别：**系统调用操作对象是一个整数句柄**。`Open` 文件得到一个整数 fd，之后的所有 IO 都是针对这个 fd 来操作的。这个明显和标准库不同，os 标准库 OpenFile 得到的是一个 `File` 结构体，所有的 IO 也是针对这个结构体的。

层次架构

那么究竟封装的层次一般是什么样的呢？我还记得 Unix 编程里面开篇就有一张如下图：  

![](https://mmbiz.qpic.cn/mmbiz_png/4UtmXsuLoNeZJaicwWlXZJmV0jdkDH1t26lyh0VicpOuibzRiciaqdTvP3VTMf8kAln4y3sAPwxN6GyRlwm3bAR69YA/640?wx_fmt=png)  

  

这张图就非常形象的讲明白了整个 Unix 体系结构。

*   内核是最核心的实现，包括了和 IO 设备，硬件交互等功能。与内核紧密的一层是内核提供给外部调用的系统调用，系统调用提供了用户态到内核态调用的一个通道；
    
*   对于系统调用，各个语言的标准库会有一些封装，比如 **C 语言的 libc 库，Go 语言的 os ，syscall 库都是类似的地位，这个就是所谓的公共库**。这层封装的作用最主要是简化普通程序员使用效率，并且屏蔽系统细节，为跨平台提供基础（同样的，为了跨平台的特性，可能会阉割很多不兼容的功能，所以才会有直接调用系统掉调用的需求）；
    
*   当然，我们右上角还看到一个缺口，应用程序除了可以使用公共函数库，**其实是可以直接调用系统调用的，但是由此带来的复杂性又应用自己承担**。这种需求也是很常见的，标准库封装了通用的东西，同样割舍了很多系统调用的功能，这种情况下，只能通过系统调用来获取；
    

总结

1.  IO 大类分为网络 IO 和磁盘 IO，IO 对文件来说就是读写操作，写的时候**数据从内存到磁盘**，读的时候**数据从磁盘到内存**；
    
2.  Go 文件 IO 最常用的是 os 库，使用 Go 封装的标准库，`os.OpenFile` 打开，`File.Write`，`File.Read` 进行读写，操作对象都是 `File` 结构体；
    
3.  Go 标准库对 IO 的封装是为了屏蔽复杂的系统调用，提供跨平台的使用姿势。然后单独提供 `syscall` 库，让程序员自我决策使用要使用更丰富的系统调用功能，当然后果自负；
    
4.  Go 标准库 IO 操作对象是 `File` ，系统调用 IO 操作对象是 fd（非负整数），**而这个 fd 则大有来头，我们后面专门分析**；
    
5.  `Open` 文件默认当前偏移量是 0 （文件最开始），加上 `O_APPEND` 参数之后偏移量会是文件末尾。通过 Seek 调用可以任意指定文件偏移，从而影响文件 IO 的位置；
    
6.  `Read`，`Write` 函数只有 buffer （buffer 有长度），偏移则使用当前文件偏移量；
    
7.  `Pread`，`Pwrite` 的系统调用效果等同于 `Seek` 偏移量然后 `Read`，`Write`，但是又大有不同。对外语义是原子操作，并且不更新当前文件偏移量；
    

  

后记

今天讨论的是 Go 的存储基础（通用的存储知识），涉及到一些 IO 基础，今天梳理了 Go 的两种 IO 的姿势，分别是 **os 标准库封装和 syscall 系统调用**。

- EOF -

推荐阅读  点击标题可跳转

1、[Go Module 时代项目结构最佳实践](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666554052&idx=3&sn=03702c2f76b61d26decfb3235f5005e1&chksm=80dca66fb7ab2f79ec6a8e5b5ed8d652a0c3d11289d2d8945f893223d618c911038c969366fe&scene=21#wechat_redirect)

2、[深入揭秘 epoll 是如何实现 IO 多路复用的！](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666552937&idx=3&sn=b1a2d38be57087007586c3271326f161&chksm=80dc9ac2b7ab13d4a4885e7b8960d4c29cf97cabe91f7866f58d652ca9d32da549f490cd82a5&scene=21#wechat_redirect)

3、[图解：深入理解高性能网络开发路上的绊脚石，同步阻塞网络 IO](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666552719&idx=3&sn=c1f4165f678a2935bcfa0b34e9a68cd6&chksm=80dc9924b7ab1032b51ea2b5a6e4ee3223b42b0dc1327f66bdb76363cbc9548502e7bbd8975b&scene=21#wechat_redirect)
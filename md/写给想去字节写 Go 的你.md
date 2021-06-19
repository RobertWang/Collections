> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU2MTkwMTE4Nw==&mid=2247495591&idx=1&sn=013b45802790b61b9d34091345245ab6&chksm=fc73144bcb049d5da1cf954629bb379d0999bb8b8ef5ae1423a03213c297d538cf5490d7b1ff&mpshare=1&scene=1&srcid=0619eV5DEZX7DHRcReNXnzh4&sharer_sharetime=1624113288396&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> 阅读目录 
> 
> * 为什么字节跳动选择使用 Go 语言？ 
> 
> * 求职字节 Go 开发岗位需要如何准备？

### 为什么字节跳动选择使用 Go 语言？

目前后端服务能胜任大型项目的编程语言有 C/C++、Java 和 Go，在给出答案之前，我们先看几个编程语言之间对比的例子。

#### 例子一

```
// Go代码
func CreateItem(id int, name string) *Item {
    myItem := Item{ID: id, Name: name}
    return &myItem
}
```

C/C++ 转 Go 语言的同学刚开始看到这样的代码可以正常运行是不敢相信的，在早些年的计算机的编程语言入门课中，老师和无数课本一直告诫我们：**不要返回一个局部变量（栈变量）的地址，因为在函数调用结束后，栈被销毁，引用已经销毁的栈中的变量可能会出现内存问题**。然而，这样的代码在 Go 中工作的很好，也很常用。

```
// C++代码
Item* CreateItem(int id, string name) {
    Item* myItem = new Item(id, name);
    return myItem;
}
```

#### 例子二

前段时间有位同学来面试，我看这位同学简历上写了熟悉 C++ 和 Go，我就问这位同学觉得 Go 语言中有哪些好用的特性，该同学提到了 defer 关键字，于是我们就 Go 中的 defer 关键字讨论了一下，我的问题是：用 C++ 可以实现 defer 关键字的等价功能吗？

以下是该同学的回答：

结论是可以的，因为 Go 中 defer 关键字是在当前函数执行完毕（或者说是出了当前函数作用域）时，自动执行一些我们指定的动作，一般用于资源的回收。代码如下：

```
//Go代码
func ReadFile(fileName string) {
    myFile, err := os.Open(fileName)
    if err != nil {
         return
 }

 //读取文件的过程中，无论哪一步出错，最终文件均会被关闭
 defer myFile.Close()

 myData1 := make([]byte, 5)
 _, err = myFile.Read(myData1)
 if err != nil {
     return
 }

 fmt.Println(string(myData1))

 myData2 := make([]byte, 5)
 _, err = myFile.Read(myData2)
 if err != nil {
     return
 }

    fmt.Println(string(myData2))
}
```

上述 Go 代码中，由于使用了 defer 语句，无论哪一步的 myFile.Read 函数操作失败了，最终文件都会被关闭，避免了文件句柄的泄漏。

在 C++ 中我们可以使用 RAII 技术实现同样的效果，只要构造一个 RAII 类即可，即在类的析构函数关闭文件，这样这个文件一旦打开，只要出了函数作用域都会被关闭。代码如下：

```
//C++代码
class File
{
public:
    File()
    {
    }

 ~File() 
 {
     Close();
 }

 bool Open(const std::string& fileName)
 {
     m_myFile = fopen(fileName.c_str(), "r");
     return m_myFile != NULL;
 }

 int Read(char* buf, int length)
 {
     if (m_myFile == NULL) 
          return -1;

     return fread(buf, 1, length, m_myFile);
 }

 void Close() 
 {
     if (m_myFile != NULL) 
     {
          fclose(m_myFile);
     }
 }

private:
    FILE*  m_myFile;
};

void ReadFile(const char* fileName)
{
    //当myFile出了作用域之后，会自动调用File类的析构函数，在析构函数中关闭文件句柄
    File myFile;
    bool success = myFile.Open(fileName);
    if (!success)
         return;

    char myData1[6] = {0};
    int count1 = myFile.Read(myData1, 5);
    if (count1 <= 0) 
        return;

    std::cout << myData1 << std::endl;
 
    char myData2[6] = {0};
    int count2 = myFile.Read(myData2, 5);
    if (count2 <= 0) 
         return;

    std::cout << myData2 << std::endl;
}
```

上述 C++ 代码中利用了 RAII 技术达到了和 Go  defer 关键字一样的效果，这位面试的同学表达的就是这个意思。

当然，这位同学回答的并不完整，Go 的 defer 关键字所能达到的一些作用，C++ 中没有与之等价的功能。

在 C++ 中，虽然在大多数情况下可以使用 RAII 技术达到在出了函数作用域时执行我们指定的动作，但是有一种情况 defer 关键字可以做到，C++ RAII 却做不到，那就是当函数执行过程中有崩溃问题（Go 中叫 panic）时，可以在 defer 指定的动作中恢复程序的执行流，不让整个进程退出，而 C++ 程序是无法做到针对一个内存问题造成的 crash 恢复进程继续执行的。Go 代码如下：

```
//Go代码
func RecoverFromCrash() {
    defer func() {
     //如果程序有崩溃，恢复程序
     recover()
    }()

    var pi *int
    *pi = 1
}

func main() {
    i := 1
    RecoverFromCrash()
    i = 2

    fmt.Println(i)
}
```

上述 Go 代码中，由于 RecoverFromCrash 函数操作了一个空指针，导致程序崩溃，然后 defer 配合 recover 函数可以恢复程序继续运行。也就是说，如果采用这种机制，一个 goroutine 产生的崩溃不会影响到其他 goroutine。这在 C++ 中是绝对无法做到的，在 C++ 程序中，任何内存问题都会导致整个进程退出。这外在表现就是，使用 Go 开发出来的程序，一个接口有问题不会影响到同一个服务中不相关的接口，更不会导致整个服务宕机，这是 C++ 程序绝对无法做到的。

除此以外， defer 关键字还做了更多工作，defer 关键字在程序崩溃时可以恢复部分数据，这点很有用，我们可以基于此记录一些程序崩溃时的现场值，这在 C++ 中根本做不到这一点。代码如下：

```
//Go代码
func ProcessRequest(req Request) (resp Response, log LoggerItem, err error) {
    defer func() {
        recover()
    }()

    resp = DoProcess1(req)

    resp = DoProcess2(req)

    //倘若程序在此处崩溃，我们可以在程序recover以后，仍然拿到前两步处理后的resp值
    resp = DoProcess3(req)

    resp = DoProcess4(req)

    return
}
```

C++ 开发者无数次幻想过有一种方法可以在进程内恢复因内存问题而崩溃的进程，更不用说记录进程崩溃时的现场数据了，而这在 Go 中均做到了，而且如此简单易用。C++ 开发者看到这里眼泪要流下来了。

> 作为面试官，该面试者能想到 C++ RAII 技术可以等价部分 Go 的 defer 关键字功能，我已经很满意了，倘若能更进一步，就很完美了。

对于 Java 来说，我们可以使用 try -catch - finally 关键字实现 defer 的上述功能，只要将代码如下：

```
//Java代码
try {

} catch (SomeException e) {

} finally {
     //Go中的defer需要做的工作放在这里
}
```

虽然在 Java 中可以把 Go 中 defer 需要做的事情放到 finally 部分，但是实际业务中导致程序出现异常有很多原因，为了避免进程不因为未处理的异常而退出，我们必须捕获顶级 Exception，在 Java 中这是一种不推荐的方式。

好了，说完这两个例子之后，让我们来对比一下 Go 与 C/C++/Java 的优缺点。

#### 性能与效率上的对比

C++ 最让人诟病的问题是需要开发者自己管理内存，从学习这块知识的角度来看，编码中直接管理内存是管理不好的，开发者必须学习与内存相关的各种操作系统原理，最起码要知道物理内存与虚拟内存、栈内存与堆内存、内存分配与释放时机、进程地址空间的内存分布、各个内存地址区间的内存读写属性、如何避免内存越界等等相关知识，而这些知识不是一蹴而就就能学会的，所以如果某个开发者能写出一个经年累月不需要重启或者不宕机的 C++ 服务，那他是个高手。但是在快速迭代业务的互联网公司，怎么可能人人都是高手呢？万一某天来了个新人加了一个新功能，导致一个隐蔽的内存问题，之后就是恼人的问题排查与定位。

C++ 自己管理内存是把双刃剑，高手可以用来写出高效的程序来，但是对于新手或者水平不够的开发者来说，这将是企业产品事故甚至灾难的源泉。

再者，拜当下各种焦虑的、浅尝辄止的、急功近利的网文的宣传，无论是个人还是公司，已经没有多少人愿意把时间花在学习周期长、难度大的 C++ 语言之上了。只要在满足业务要求的情况下，公司当然愿意花更低的成本去使用更能保证业务快速、稳定迭代的 Go 语言上了。

那么 Java 呢？Java 最大的问题是，其编译出来的程序不是操作系统原生支持的可执行文件，必须运行在 Java 虚拟机之中，这样要想运行必须依赖于 Java 虚拟机，而对于复杂业务来说，生成的 Jar 文件也偏臃肿。所以无论是安装 Java 程序的本身需要的运行环境还是生成的 Jar 文件的执行效率大大折扣。

我列一张表来对比一下 C++、Java 与 Go 在性能与可执行文件体积上的差别，需要说明一下，这张表是针对具有复杂功能的中大型项目来说的：

<table data-darkmode-color-16241140270728="rgb(163, 163, 163)" data-darkmode-original-color-16241140270728="#fff|rgb(0,0,0)"><thead data-darkmode-color-16241140270728="rgb(163, 163, 163)" data-darkmode-original-color-16241140270728="#fff|rgb(0,0,0)"><tr data-darkmode-color-16241140270728="rgb(163, 163, 163)" data-darkmode-original-color-16241140270728="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16241140270728="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16241140270728="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-darkmode-color-16241140270728="rgb(163, 163, 163)" data-darkmode-original-color-16241140270728="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16241140270728="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16241140270728="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); min-width: 85px; text-align: center;"><br data-darkmode-color-16241140270728="rgb(163, 163, 163)" data-darkmode-original-color-16241140270728="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16241140270728="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16241140270728="#fff|rgb(255,255,255)|rgb(240, 240, 240)"></th><th data-darkmode-color-16241140270728="rgb(163, 163, 163)" data-darkmode-original-color-16241140270728="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16241140270728="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16241140270728="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); min-width: 85px; text-align: center;">执行效率</th><th data-darkmode-color-16241140270728="rgb(163, 163, 163)" data-darkmode-original-color-16241140270728="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16241140270728="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16241140270728="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); min-width: 85px; text-align: center;">可执行文件体积</th><th data-darkmode-color-16241140270728="rgb(163, 163, 163)" data-darkmode-original-color-16241140270728="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16241140270728="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16241140270728="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); min-width: 85px; text-align: center;">依赖运行环境</th></tr></thead><tbody data-darkmode-color-16241140270728="rgb(163, 163, 163)" data-darkmode-original-color-16241140270728="#fff|rgb(0,0,0)"><tr data-darkmode-color-16241140270728="rgb(163, 163, 163)" data-darkmode-original-color-16241140270728="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16241140270728="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16241140270728="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16241140270728="rgb(163, 163, 163)" data-darkmode-original-color-16241140270728="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16241140270728="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16241140270728="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;">C++</td><td data-darkmode-color-16241140270728="rgb(163, 163, 163)" data-darkmode-original-color-16241140270728="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16241140270728="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16241140270728="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;">高</td><td data-darkmode-color-16241140270728="rgb(163, 163, 163)" data-darkmode-original-color-16241140270728="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16241140270728="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16241140270728="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;">小</td><td data-darkmode-color-16241140270728="rgb(163, 163, 163)" data-darkmode-original-color-16241140270728="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16241140270728="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16241140270728="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;">无</td></tr><tr data-darkmode-color-16241140270728="rgb(163, 163, 163)" data-darkmode-original-color-16241140270728="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16241140270728="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16241140270728="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16241140270728="rgb(163, 163, 163)" data-darkmode-original-color-16241140270728="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16241140270728="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16241140270728="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;">Java</td><td data-darkmode-color-16241140270728="rgb(163, 163, 163)" data-darkmode-original-color-16241140270728="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16241140270728="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16241140270728="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;">低</td><td data-darkmode-color-16241140270728="rgb(163, 163, 163)" data-darkmode-original-color-16241140270728="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16241140270728="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16241140270728="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;">大</td><td data-darkmode-color-16241140270728="rgb(163, 163, 163)" data-darkmode-original-color-16241140270728="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16241140270728="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16241140270728="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;">Java 虚拟机</td></tr><tr data-darkmode-color-16241140270728="rgb(163, 163, 163)" data-darkmode-original-color-16241140270728="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16241140270728="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16241140270728="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16241140270728="rgb(163, 163, 163)" data-darkmode-original-color-16241140270728="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16241140270728="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16241140270728="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;">Go</td><td data-darkmode-color-16241140270728="rgb(163, 163, 163)" data-darkmode-original-color-16241140270728="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16241140270728="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16241140270728="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;">高</td><td data-darkmode-color-16241140270728="rgb(163, 163, 163)" data-darkmode-original-color-16241140270728="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16241140270728="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16241140270728="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;">小</td><td data-darkmode-color-16241140270728="rgb(163, 163, 163)" data-darkmode-original-color-16241140270728="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16241140270728="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16241140270728="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;">无</td></tr></tbody></table>

#### 语法层面上的对比

工作的早些年，我在使用 C/C++ 和 Java 进行编程时，曾思考这样一个问题，既然大多数的代码行末尾必须都要以分号结束，那为啥编译器不直接代劳此事？从编译原理的角度来说，大多数代码行末尾的分号都是没有任何作用的。

而更早的学生时代，我常常因为忘记在某些代码行的结尾写上分号而导致代码无法编译通过，我相信，在今天，数以万计的刚开始接触编程的同学也遇到和我曾经一样的问题。

另外一个情形就是很多同学在写 switch - case 语句的时候，有时候因为忘记在特定的 case 语句之后写上 break 语句，从而导致程序执行时出现非预期的行为，这个问题也同样困扰着学习编程的新人们。

一对大括号中的第一个大括号是否要单独放在一行；if/for 等执行体只有一条语句时，是否应该使用一对大括号包裹起来，这类问题在开发者之间争论了几十年，并且将继续在后来者那里争论下去，就算是像《代码大全》这样经典的书籍也花了好几页去讨论这两种代码风格哪种好，更不用说各个公司为了统一编码风格而制定的各种代码规范和 lint 检查规则了。

```
//到底哪种风格好呢？
//风格1
void DoTest() {

}

//风格2
void DoTest()
{

}

//风格1
if (success) {
    printf("success");
}

//风格2
if (success)
    printf("success");
```

继往开来，Go 语言大刀阔斧地去除了一些其他语言中看起来不是很必要的功能，这些功能的去除让 Go 的风格变得统一、简洁，在 Go 项目中，大家不会再为上文中提到的几个风格问题而争论了。

让我们来看一下 Go 语言相对于其他语言所做的一些改动，欢迎读者在评论区补充：

##### 1. 每一行语句的结尾不再强行要求加上分号

```
fmt.Println("hello world") //末尾不建议加;
```

##### 2. 一对大括号的第一个不能单独占一行

```
//错误的语法
func DoTest()
{

}

//正确的语法
func DoTest() {

}
```

##### 3. if/for 等语句体只有一行时也必须使用一对大括号包裹起来

```
//正确的语法
if (success) {
    printf("success")
}

//错误的语法
if (success)
    printf("success")
```

##### 4. if/for 等条件不再需要括号

```
//正确的语法
for i := 1; i < 10; i++ {
    fmt.Println(i)
}

//错误的语法，for语句不需要括号
for (i := 1; i < 10; i++) {
   fmt.Println(i)
}
```

##### 5. 只有 for 循环，不再支持 while 和 do - while 循环

```
//支持的语法
for i := 1; i < 10; i++ {
    fmt.Println(i)
}

//不支持的语法
while i < 100 {
    fmt.Println(i)
    i++
}
```

##### 6. switch - case 语句默认加了 break 语句

```
switch i {
 case 0:
     fmt.Println(0)
 case 1:
     fmt.Println(1)
 case 2:
     fmt.Println(2)
 default:
}

//相当于 
switch i {
 case 0:
     fmt.Println(0)
     break
 case 1:
     fmt.Println(1)
     break
 case 2:
     fmt.Println(2)
     break
 default:
}
```

如果你真的想执行完一个 case 接着执行下一个 case，只要使用 fallthrough 关键字就可以了：

```
switch i {
 case 0:
     fmt.Println(0)
     fallthrough
 case 1:
     fmt.Println(1)
 case 2:
     fmt.Println(2)
 default:
}
```

##### 7. 自增自减运算符只支持后缀形式，不支持前缀形式

```
i := 0
i++ //可以编译通过
++i //无法通过编译
```

##### 8. 不支持条件运算符（? :）

```
b := 9
a := (b > 0 ? true : false) //这一行无法通过编译
```

##### 9. 给一个结构体多个字段设置值时，最后一个字段也必须以逗号结束

```
type StandardResp struct {
    Code  int32
    Msg   string
    Data  interface{}
}

c.JSON(http.StatusOK, commonHttp.StandardResp{
      Code: 1000,
      Msg:  "token error",
      Data: nil, //注意这里nil之后有一个逗号，这在其他语法中必须没有逗号
  })
```

以上列举了 Go 精简后的一些语法要素，精简后的语法，让编程初学者更容易记忆与上手。

极少的语法元素，让 Go 简单易学，字节的大多数同学都是入职后两周内学习的 Go，然后开始着手业务开发。

#### 功能完备性的对比

Go 与 Java 相比较于 C++，其语言自带的 API 库功能是相当完善的，从基本的字符串操作到网络编程、文件读写等等应用尽有，因此 Go 的开发者可使用的原生 API 就很丰富，比如编写一个网络通信程序，Go 和 Java 都在 net 包中提供了大量可使用的 API，而 C++ 必须直接借助操作系统的 Socket API。

这就是我说的**语言的功能完备性**，如果一个编程语言自带的 API 越丰富，那么开发者只要尽可能地掌握语言自身的 API 就可以了，自身的 API 通常会屏蔽了各个操作系统的差异性，学习成本更低，同样是 Socket API，学习 Go 或者 Java SDK 自带的网络 API，要比直接学习多个操作系统的 Socket API 要容易得多。

对于 C++ 语言来说，随着 C++ 标准的不断发展，C++ 语言自身的功能完备性也在逐步完善，例如从 C++11 开始，就可以直接使用 stl 中的线程相关的类，而不用再使用操作系统提供的线程接口。

#### 结论

综上所述，我给出我的结论，正因为 Go 语言简单易学、不容易出错、功能完备性良好且执行效率高，特别适合字节这样有超多超快的业务线产品迭代。当然，Go 语言想入门容易，想学好成为高手并不容易，很多从其他语言转到 Go 开发的同学，若不刻意勤加练习，想写出地道、高效的 Go-Style 风格的代码也不是一件很容易的事情。

这里推荐几本我学习 Go 的书籍：

**艾伦 《Go 程序设计语言》**

**许式伟 吕桂华《Go 语言编程》**

**雨痕 《Go 语言学习笔记》**

### 求职字节 Go 开发岗位需要如何准备？

相比较为什么字节跳动选择使用 Go 语言，很多同学可能更关心求职字节跳动 Go 开发岗位要如何准备。

这里有先消除一个错误认知：和大多数 Go 岗位（字节的和非字节的）一样，字节招 Go 开发岗位的对求职者是否熟悉 Go 语言没有强制要求，也就是说，你可以不熟悉 Go 语言也可以应聘字节的 Go 开发岗位，但是有几个注意事项：

#### 一、虽然不要求熟悉 Go 开发，但要求熟悉至少一门编程语言

字节很多进来做 Go 的同学之前都是做 C、C++ 或者 Java， 甚至是 php 或者 Python 的。所以，如果你之前根本没接触过 Go 或者接触过但不熟悉 Go，尽量不要在简历中写自己熟悉 Go。这样面试的时候面试官也就不会考察你任何关于 Go 本身的问题，这点很重要，比如，你原来是做 C++ 或者 Java 的，你为了应聘这个岗位强行写上自己熟悉 Go，那么面试官可能会重点考察一下你的 Go 技术栈，这样你相当于被考察一个不熟悉的技术栈，你很吃亏。我曾内推的几位同事，包括最近面试的一两位同学都是这样，拿自己弱项来接受考察，面试结果一般都不尽人意。以上文中那位同学为例，如果他不在简历中写自己熟悉 C++ 和 Go，只写自己熟悉 C++，我就不会问他任何关于 Go 的问题了，比如 defer 关键字的问题。

#### 二、尽量写上一门自己擅长的语言即可

这里的意思是，如果你之前是做 C++ 开发的，那你就写上你熟悉 C++，Java 也一样，这样面试的时候，除了通用部分，面试官会考察你相应的语言相关的内容，例如简历上写了熟悉 C++，如果连 C++ 虚函数的实现机制、stl 常用容器都说不清楚，那明显不符合预期；写熟悉 Java 的，HashMap、Java 的线程池 ThreadPoolExecutor、Java 虚拟机等都是常考的内容。我看过一部分同学未通过面试的原因是：不管熟悉不熟悉的技术栈都一股脑儿地写到简历中，结果面试时被问到又说不清楚。

#### 三、校招看基础，社招看经验

基础知识就那么多，包括算法与数据结构、操作系统原理、计算机网络（网络编程）、多线程、数据库、设计模式等等；社招看经验，所谓经验，不仅指良好的基本功，还包括丰富的项目经验和解决问题的能力，一般 2-1 及以上职级的基本上要求至少擅长分布式、RPC、消息中间件、缓存、数据库等至少其中的一种。另外，就是解决问题的能力，很多算法题或者场景题都是源自于真实的业务场景，需要面试者给出自己解决问题的思路和可以落地的方法。

#### 四、字节很注重算法能力

通常情况下， 对于校招生，算法题做不好，基本一票否决；对于社招，工作五年及五年以下的，也会考察一部分算法题。很多工作多年的同学，由于平时不注重温习和理解算法与数据结构知识，忘记了很多算法思想和解决问题的策略。给这部分同学的建议是：

1.  面试前适当复习下常见的算法与数据结构；
    
2.  另外，平常刻意去刷一些算法题，去锻炼一下自己的解决问题的思路；
    
3.  面试的时候，如果遇到不会的算法题，千万不要直接放弃，可以先尝试暴力穷举法或者找面试官要些提示。
    

除了一些算法岗位，社招的算法题一般都不难，有些算法题也不单纯是考察算法，可能结合其他知识点一起考察，这里列举一些题目给读者做一些参考：

```
1. 实现一个字符串转换整数的函数；

2. 输入两个递增排序的链表，合并这两个链表并使新链表中的结点仍然是按照递增排序的，例如：
链表1：1 -> 3 -> 5 -> 7
链表2: 2 -> 4 -> 6 -> 8
合并后的链表3:
1 -> 2 -> 3 -> 4 -> 5 -> 6 -> 7 -> 8

链表定义：
struct ListNode
{
    int       m_nValuel
    ListNode* m_pNext; 
};

3. 输入n个整数，找出其中最小的 k 个数目，例如输入4、5、1、6、2、7、3、8，则最小的4个数字是1、2、3、4。

4. 输入两个链表，找出它们的第一个公共结点，链表结点定义如下：
struct ListNode
{
    int       m_nKey;
    ListNode* m_pNext; 
};

5. TCP的滑动窗口机制知道吗？设计一个可行的滑动窗口的算法。


6. 中国象棋中，假设左下角的位置为坐标原点，某棋子马的坐标是(x, y)，另外一个棋子的坐标为(m, n)，实现一个函数返回马下一步可走的位置坐标。

7. 实现一个缓冲区类，需要支持以下功能：
（1）. 缓冲区内存要求连续
（2）. 支持扩容
（3）. 支持读和写
```

**如果你对这些算法题有不明白的地方，可以通过我的公众号加我的微信群交流。**

上述题目部分是《**剑指 offer**》一书的原题，我向读者推荐一下这本书，另外一本是左程云老师的《程序员代码面试指南》。

适当刷一些算法题，不单纯为了应付面试，对锻炼思维能力也是大有裨益的。

当然，很多非科班的同学一上来就刷算法题，其实是不推荐的，如果你没有系统地学习过算法和数据结构的课程，建议先系统地学习下这一块的内容。

#### 五、字节很注重动手能力

疫情仍然没有完全过去，现在字节的面试基本上都是改在线上进行，当面试官给你一些算法题或者场景题时，需要你在自己的电脑上进行编程，在编程过程中的各种行为，例如你的编写代码、解决编译错误、调试能力和代码风格都是面试官一眼能看到的。平常动手多寡、高下立判，我曾遇到用 VSCode 写 Java 代码然后连编译问题都解决不了的同学，显然，最终面试肯定也未通过。

#### 六、网上的面经适当看，带着理解与批判精神去看

面试大厂的结果有一定的运气成分，不同的人在不同的场景面试遇到不同的面试官其表现的面试结果可能也不一样。

近来我发现有些校招的同学，把网络上的面经和所谓的标准答案一字不拉地背诵下来用于应付面试，这是非常不可取的：

其一，有些面筋的答案本身就是错的，例如网上有一篇流传很广的文章，谈到 HTTP GET 与 POST 请求的区别时，说 GET 请求会发送一个数据包，而 POST 请求则会拆成两个数据包去发送，这种说法明显就是错误的。很多同学不加甄别的背诵下来，我迄今至少遇到两位同学面试时这么回答；

其二，光背面经如果不加以理解很难应付灵活变化的面试题，例如，有些同学把三四握手和四次挥手的过程背诵下来了，但是当我问到连接一个 IP 不存在的主机时，握手过程是怎样的、或者连接一个 IP 地址存在但端口号不存在的主机握手过程又是怎样的呢？如果对三次握手过程不加以理解，是很难回答出这样的问题的。企业需要一些理解技术原理并能灵活运用的员工，而不是死记硬背的人。

#### 七、C++ 和 Java 太难了，直接学 Go 吧

很多同学觉得 C++ 太难了，学不好，Java 学的人太多，竞争压力大，所以干脆学 Go 吧，上文说过，各大招 Go 岗位的公司其实对 Go 本身不做刻意要求，反而对技术原理要求不低。所以，打铁还得自身硬，想靠学 Go 走捷径其实行不通，技术基本功决定着你将来在技术这条路上能走多远。那些看似难啃的技术原理，今天所欠下的技术债，总会在你的职业生涯的某一个阶段爆发出来。以学习 C/C++ 为例，如果你学习 C/C++ 单纯只是为了找工作或者应付面试，那一定也是学不好的，C++ 语法本身并没有多难，难的是支持 C++ 技术背后的各种操作系统原理，这些原理你在 C/C++ 中会用到，你在学习其他语言到一定阶段也不可或缺；反过来，以网络编程为例，在 Go 中讨论 epoll 模型多少有点别捏，或者是说不清道不明，但是站在 C/C++ 的角度结合操作系统的网络 API，这个问题就很容易搞明白了。

所以，对于开发这条路来说，换一门容易学的语言并不能让你拥有核心竞争力。相反，如果你想做好开发，尤其是后端开发，你应该掌握一门重型编程语言，如 C++ 或者 Java，这也是为什么我劝那些想做好开发的同学不要只掌握一门 Python/PHP 这样的语言。

限于笔者经验水平有限，欢迎在文末温和地提出建议和意见。  

**推荐阅读**

*   [在 2021 年写一本 C++ 图书是一种什么体验？](https://mp.weixin.qq.com/s?__biz=MzI0NTE4NTE4Nw==&mid=2648792707&idx=1&sn=c7b9cc545937aa88f26e3a791c0f4b2e&scene=21#wechat_redirect)
    -----------------------------------------------------------------------------------------------------------------------------------------------------------------
    
      
    
    ---
    
    点个在看少个 bug👇  
    
    ---------------
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/MIun-eV4_J1hXGDRjGoLaw)

******1. 什么是符号化？******

在日常开发中，应用难免会发生崩溃。通常，我们直接从用户导出来的崩溃日志都是未符号化或者部分符号化的，都是一堆十六进制内存地址的集合，可读性较差。未符号化或者部分符号化的崩溃日志对闪退问题的解决几乎毫无帮助，如下所示:

```
Last Exception Backtrace:
0  CoreFoundation  0x1ca4cd27c 0x1ca3b5000 + 1147516
1  libobjc.A.dylib  0x1c96a79f8 0x1c96a2000 + 23032
2  CoreFoundation  0x1ca3ded94 0x1ca3b5000 + 171412
3  TestBacktrace  0x102a47464 0x102a40000 + 29796
4  UIKitCore  0x1f6c86e30 0x1f63d3000 + 9125424
```

只有符号化后的崩溃日志才能显示各个线程的函数调用，而不仅仅是毫无意义的虚拟内存地址。符号化后的崩溃日志如下所示， 此时，我们就能够直接从堆栈信息中知道应用 TestBacktrace 发生崩溃时的函数为 [AppDelegate Application:didFinishLaunchingWithOptions:]，崩溃时函数所在文件为 AppDelegate.m，行号为 23：

```
Last Exception Backtrace:
0   CoreFoundation                  0x1ca4cd27c __exceptionPreprocess + 228
1   libobjc.A.dylib                 0x1c96a79f8 objc_exception_throw + 55
2   CoreFoundation                  0x1ca3ded94 -[__NSSingleObjectArrayI objectAtIndex:] + 127
3   TestBacktrace                   0x102a47464 -[AppDelegate Application:didFinishLaunchingWithOptions:] + 29796 (AppDelegate.m:23)
4   UIKitCore                       0x1f6c86e30 -[UIApplication _handleDelegateCallbacksWithOptions:isSuspended:restoreState:] + 411
```

******2. 符号化原理******

### ******丨****** 2.1 什么是 dSYM 文件？

iOS 平台中， **dSYM 文件是指具有调试信息的目标文件**，文件名通常为：xxx.app.dSYM，其中 xxx 通常表示应用程序的二进制包名，如下图所示：  

![](https://mmbiz.qpic.cn/mmbiz_png/Ce6bSqXkduyh3Q1I7w3kII9QxLVNE03hibE3qUXFfOaPd2w1sbUrOiaECgbouCPhb75XkySzW9kyQgibybbRc0lKw/640?wx_fmt=png)

  
通常我们可以在 Xcode 打包出来的文件 xcarchive 里面看到 dSYM 文件以及目录架构：

![](https://mmbiz.qpic.cn/mmbiz_png/Ce6bSqXkduyh3Q1I7w3kII9QxLVNE03hGJoxusMztXicO0VDMvS5b2bACXwnsWZJBNUKcJgzN6Q1iaz2Z4nsjIiaQ/640?wx_fmt=png)

  
**dSYM 中存储着****文件名、方法名、行号**等信息，是和可执行文件的 16 进制函数地址一一对应的，通过分析崩溃的崩溃文件可以准确知道具体的崩溃信息。**DWARF(Debuging With Arbitrary Record Format) 是 ELF 和 Mach-O 等文件格式中用来存储和处理调试信息的标准格式**。DWARF 中的数据是高度压缩的，可以通过 dwarfdump、otool 等命令提取其中的可读信息。比如提取关键的调试信息 debug_info 、debug_line，可使用命令

```
dwarfdump --debug-line /Users/xxxx/Desktop/resource/TestBacktrace.app.dSYM > debug_line.txt
```

 导出 debug_line 的信息到文件 debug_line.txt 中，debug_info 也可以使用类似命令导出。

> ELF、Mach-O 分别是 Linux 和 Mac OS 平台用于存储二进制文件、可执行文件、目标代码和共享库的文件名称。

### ******丨****** 2.2 如何生成 dSYM 文件

在编译工程时， Debug 模式会默认选中不生成 dSYM 文件， 该配置可在 Build Setting|Build Option 中更改，Release 模式下 dSYM 是默认生成的。另外，如果开启了 bitcode 优化的话，苹果会做二次编译优化，所以最终的 dSYM 就需要在 Apple Connect 手动下载了。每次编译生成的 dSYM 都会有所差别，通常 **dSYM 中会有一个唯一标识**，称作 UUID ，用以区分不同的 dSYM 文件。  

![](https://mmbiz.qpic.cn/mmbiz_png/Ce6bSqXkduyh3Q1I7w3kII9QxLVNE03hiahGLDDRtUJXVvYF6FdRCSzcibOnF048g5erbm2mZsr51oYusWib6kFgg/640?wx_fmt=png)

### ******丨****** 2.3 如何通过崩溃日志中应用的 UUID 找到匹配的 dSYM ？

**还原崩溃堆栈时，需要 dSYM 的 UUID 与崩溃时的应用 UUID 一致**。通常，每一个 dSYM 文件都有一个 UUID，和 App 文件中的 UUID 对应，代表着是一个应用。而每一条崩溃信息都会记录着应用的 UUID，用来和 dSYM 的 UUID 进行校对匹配。

1. 首先从崩溃日志的 Binary Images 后找到应用的 UUID，如下可得到 TestBacktrace 的 UUID 为 6be881754f573769926b838490e39857。

```
Binary Images:
0x102a40000 - 0x102a6bfff TestBacktrace arm64  <6be881754f573769926b838490e39857> /var/containers/Bundle/Application/B44844E6-AFF4-491E-8168-F4ED93D644C2/TestBacktrace.App/TestBacktrace
0x102d14000 - 0x102d6bfff dyld arm64  <9c893b6aa3b13d9596326ef6952e7195> /usr/lib/dyld
```

2. 使用以下命令查看 dSYM 文件的 UUID，去掉 - 并且小写之后，与第一步中的 UUID 是完全一致的，证明两者是匹配的，否则是不匹配的。

```
xcrun dwarfdump --uuid < dSYM 文件>
```

![](https://mmbiz.qpic.cn/mmbiz_png/Ce6bSqXkduyh3Q1I7w3kII9QxLVNE03hgCC5aHl6c2S1CqIIv3TOMZokibiabpvh7c3HicnsfQ6Eflb78Hth8nmng/640?wx_fmt=png)  

3. 如果本地 dSYM 过多的话，一个个查看太麻烦，还可以使用 mdfind 命名根据 UUID 在本机查找 dSYM。以上面的 UUID 为例，直接在终端输入以下命令就可以了。

```
mdfind "com_apple_xcode_dsym_uuids == 6BE88175-4F57-3769-926B-838490E39857"
```

### ******丨****** 2.4 符号化流程

将崩溃日志中的 APP 二进制地址转化为函数流程如下所示：  

![](https://mmbiz.qpic.cn/mmbiz_png/Ce6bSqXkduyh3Q1I7w3kII9QxLVNE03hx1pRiaWogHpQRafYok1tticLHGxUHUF6OnCbaon55eYPaK0hV5fHMNLQ/640?wx_fmt=png)

### 获取到崩溃日志 App 关键行信息

![](https://mmbiz.qpic.cn/mmbiz_png/Ce6bSqXkduyh3Q1I7w3kII9QxLVNE03hcibTseaWTgkph1kpQ21qYphOicr2jU2eJCSDiaFzeuXeFnvZfp40y2Kag/640?wx_fmt=png)

  
从上图中可以看到 APP 的关建行为是：

```
3 TestBacktrace 0x102a47464 0x102a40000 + 29796
```

其中 TestBacktrace 为我们的二进制包名名称，其余行都是系统堆栈。

### 获取到偏移量、运行时堆栈地址、运行时 APP 起始地址

由关键行信息获取到 TestBacktrace 相对于起始地址的偏移量为 29796，运行时堆栈地址为 0x102a47464，运行时 APP 起始地址为 0x102a40000。

### 获取 dSYM 起始地址

dSYM 文件中保存中符号表 TEXT 段的起始地址，起始地址可通过以下命令获得: 

```
otool -l /Users/xxxxx/Desktop/TestBacktrace.app.dSYM/Contents/Resources/DWARF/TestBacktrace | grep __TEXT -C 5
```

![](https://mmbiz.qpic.cn/mmbiz_png/Ce6bSqXkduyh3Q1I7w3kII9QxLVNE03hgq2ketI9FooPp9M9SCLRqZIKtgTbP84BsYI1GcTQaKvZbqRM6rOuxw/640?wx_fmt=png)

由上图中可得到 dSYM 中代码段起始地址为 0x10000000。

### 计算崩溃地址对应 dSYM 符号表中的地址

因为 iOS 加载 Mach-O 文件时为了安全使用了 ASLR(Address Space Layout Randomization) 机制，导致二进制 Mach-O 文件每次加载到内存的首地址都会不一样，但是偏移量，加载地址，起始地址的计算规则是一样的；从上面我们可以得到 **0x102a47464 (运行时地址) = 0x102a40000 (起始地址) + 29796(偏移量)** 这个公式。因此通过 dSYM 的起始地址和偏移量就可以计算出 0x102a47464 对应在 dSYM 中的地址为 0x100007464 = 0x0000000100000000 + 29296。

### 获取到具体的函数 / 行数 / 文件

获取到运行堆栈地址在 dSYM 文件的对应地址 0x100007464 之后，在 dSYM 文件的 debug-info 中就可以查找到包含该地址的 DIE(Debug Information Entry) 单元，Mac OS 下可使用命令

```
dwarfdump TestBacktrace.app.dSYM --lookup 0x100007464
```

 获取相应信息，如图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/Ce6bSqXkduyh3Q1I7w3kII9QxLVNE03hiaJDnAluMv71o1HjGDEXHBhhHQWOmAS32h2VjvV4awuZDndoDvVPnSw/640?wx_fmt=png)

*   DW_TAG_Subprogram 表示这个 DIE 单元表示的是函数方法。
    
*   DW_AT_low_pc 表示这个方法起始地址为 0x1000073b4 。
    
*   DW_AT_high_pc 表示这个方法结束地址为 0x1000074c4 。这就表示崩溃日志中 0x102a47464 转化后的偏移地址 0x100007464 正好位于这 DW_AT_low_p 和 DW_AT_high_pc 之间。
    
*   DW_AT_name 表示我们的函数名为 [AppDelegateApplication:didFinishLaunchingWithOptions:]。
    
*   DW_AT_decl_file 表示函数所在文件路径为 AppDelegate.m。
    
*   DW_AT_decl_line 表示函数开始行数为 19。
    

### 组装并格式化

最终经过格式优化，崩溃日志中 0x102a47464 符号化出来对应的方法为：

```
3   TestBacktrace                   0x102a47464 -[AppDelegate Application:didFinishLaunchingWithOptions:] + 29796 (AppDelegate.m:23
```

******3. 本地符号化******

### ******丨****** 3.1 符号化方法

### Xcode 符号化

将崩溃日志、 dSYM 文件和可执行文件放在同一目录下，然后将 崩溃日志拖拽至 Devicelog 中，右键 symbolicate Log 或者 Re-symbolicate Log 就能符号化。

![](https://mmbiz.qpic.cn/mmbiz_png/Ce6bSqXkduyh3Q1I7w3kII9QxLVNE03hriaZdcou9Y3dVJ74V0ibPNylmsmW5vdM29bLJQB6rXiaFfia5U1Crmf2LQ/640?wx_fmt=png)

### 使用 symbolicatecrash 命令行符号化

*   **定位 symbolicatecrash 脚本**  
    

通常 symbolicatecrash 的路径为 /Applications/Xcode.App/Contents/SharedFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash

*   **前置运行命令**
    

运行 symbolicatecrash 前一般需要先运行：

```
export DEVELOPER_DIR="/Applications/XCode.App/Contents/Developer"
```

*   **运行 symbolicatecrash 命令**
    

首先将崩溃日志、 dSYM 以及 symbolicatecrash 复制出来放到同一个文件夹，然后 cd 到当前文件夹，运行如下命令符号化 

```
./symbolicatecrash TestBacktrace-2021-07-30-135514.ips TestBacktrace.app.dSYM > symbol.log
```

******丨****** 3.2 系统日志符号化

值得注意的是，有些时候，崩溃日志里并不会有 App 的调用，而可能全都是系统库的调用，如下：

```
Thread 32 Crashed:
0    libobjc.A.dylib                 0x19aaf6c10 0x19aad3000 + 146448
1    CFNetwork                       0x187545a28 0x18737d000 + 1870376
2    Foundation                      0x18808db4c 0x187f6c000 + 1186636
3    Foundation                      0x187f8a908 0x187f6c000 + 125192
4    Foundation                      0x18808fde8 0x187f6c000 + 1195496
5    Foundation                      0x187f8a5c4 0x187f6c000 + 124356
6    Foundation                      0x1880907e0 0x187f6c000 + 1198048
7    Foundation                      0x1880902ac 0x187f6c000 + 1196716
8    libdispatch.dylib               0x1869863e4 0x186976000 + 66532
9    libdispatch.dylib               0x1869d7298 0x186976000 + 397976
10   libdispatch.dylib               0x18697c028 0x186976000 + 24616
11   libdispatch.dylib               0x18697b828 0x186976000 + 22568
12   libdispatch.dylib               0x186988bb8 0x186976000 + 76728
13   libdispatch.dylib               0x186989378 0x186976000 + 78712
14   libsystem_pthread.dylib         0x1cf2c5580 0x1cf2ba000 + 46464
```

符号化后的日志为：  

```
Thread 32 Crashed:
0 libobjc.A.dylib _objc_release (in libobjc.A.dylib) 16
1 CFNetwork __CFNetworkHTTPConnectionCacheSetLimit (in CFNetwork) 154728
2 Foundation ___NSBLOCKOPERATION_IS_CALLING_OUT_TO_A_BLOCK__ (in Foundation) 16
3 Foundation -[NSBlockOperation main] (in Foundation) 100
4 Foundation ___NSOPERATION_IS_INVOKING_MAIN__ (in Foundation) 20
5 Foundation -[NSOperation start] (in Foundation) 784
6 Foundation ___NSOPERATIONQUEUE_IS_STARTING_AN_OPERATION__ (in Foundation) 20
7 Foundation ___NSOQSchedule_f (in Foundation) 180
8 libdispatch.dylib __dispatch_block_async_invoke2 (in libdispatch.dylib) 104
9 libdispatch.dylib __dispatch_client_callout (in libdispatch.dylib) 16
10 libdispatch.dylib __dispatch_continuation_pop$VARIANT$mp (in libdispatch.dylib) 412
11 libdispatch.dylib __dispatch_async_redirect_invoke (in libdispatch.dylib) 784
12 libdispatch.dylib __dispatch_root_queue_drain (in libdispatch.dylib) 376
13 libdispatch.dylib __dispatch_worker_thread2 (in libdispatch.dylib) 120
14 libsystem_pthread.dylib __pthread_wqthread (in libsystem_pthread.dylib) 212
```

可以看出是 CFNetwork 网络请求时发生野指针导致的问题，那么我们就可以针对网络相关的请求做进一步排查。  

至此可以得出结论：符号化系统库是很有必要的，特别是对一些 App 堆栈信息完全没有的崩溃日志。

### 如何符号化系统库符号

符号化自己 App 的方法名，需要编译生成的 dSYM 文件。而要将系统库的符号化为完整的方法名，也需要 iOS 各系统库的符号文件。

*   #### 匹配对应的符号文件版本
    

用户的崩溃日志来自各种系统版本，需要对应版本的系统符号文件才能符号化。系统库符号文件不是通用的，而是对应崩溃所在设备的系统版本和 CPU 型号的。

崩溃日志中有这样几个信息：

```
Code Type:       ARM-64
OS Version:      iOS 10.2 (14C82)
.........
Binary Images:
0x102a40000 - 0x102a6bfff TestBacktrace arm64  <6be881754f573769926b838490e39857> /var/containers/Bundle/Application/B44844E6-AFF4-491E-8168-F4ED93D644C2/TestBacktrace.App/TestBacktrace
0x102d14000 - 0x102d6bfff dyld arm64  <9c893b6aa3b13d9596326ef6952e7195> /usr/lib/dyld
```

Code Type 表示此设备的 CPU 架构为 armv7、armv7s、arm64 还是 arm64e。  

OS Version 表示此设备的系统版本号，括号中的字符串代表了此系统的 build 号。

Binary Images 中的 <9c893b6aa3b13d9596326ef6952e7195> 里面的字符表示对应的系统库 dyld 的 UUID，**只有 build + UUID 匹配的系统库符号文件才能符号化系统符号**。

*   #### 把符号文件放到指定位置
    

把获取到的对应版本的符号文件放到 Mac OS 的 ~/Library/Developer/Xcode/iOS DeviceSupport 目录下，就可以使用 Xcode 自带的符号化工具 symbolicatecrash 进行符号化了。这个工具会自动根据崩溃日志中系统库的 UUID 搜索本机系统库的符号文件。

### ******丨****** 3.3 获取系统符号文件的 2 个方法

### 从真机上获取

大部分系统库符号文件只能从真机上获取，苹果也没有提供直接的下载地址。但是当你用 Xcode 第一次连接某台设备进行真机调试时，会看到 Xcode 显示 Processing symbol files ，这时候就是在拷贝真机上的符号文件到 Mac OS 系统的 /Users/xxx/Library/Developer/Xcode/iOS DeviceSupport 目录下。

![](https://mmbiz.qpic.cn/mmbiz_png/Ce6bSqXkduyh3Q1I7w3kII9QxLVNE03hqics7RdkWvQrTbgkYXv4EU96R06Qph7bBMSamzPKicRFccgIkQqNicNYQ/640?wx_fmt=png)

目录下的 14.7.1 (18G82) 这样的文件夹就是对应的符号文件，通常都有 1-5GB 的大小。

### 从固件中提取符号文件

从固件 (iPSW) 中可以通过一些方式提取到系统库符号文件。固件解密分为 下载并提取系统符号 和 系统库符号 提取两步。

#### 1. 下载并提取系统符号

*   iOS9 以及 iOS9 之前
    

a. 下载对应版本的 iPSW 固件，直接解压，解压后里面有几个 dmg 格式的镜像文件，最大的 dmg 文件就是系统镜像。

b. 从 Firmware_Keys (见文末参考链接) 找到对应固件的解密 key (页面上 Root Filesystem 字段的 key )

![](https://mmbiz.qpic.cn/mmbiz_png/Ce6bSqXkduyh3Q1I7w3kII9QxLVNE03hiavTQMGWnKqAjxJsibbHpflraOp4pYaN0w8xdIqYzneV7o2S1HKUEufg/640?wx_fmt=png)

c. 用 dmg 工具进行解密。cd 到解压后的 iPSW 文件夹，执行 ./dmg extract xxx-xxxx-xxx.dmg dec.dmg -k <key> 。extract 后面跟两个参数，分别是系统镜像 dmg 的名字和解密后的文件名，-k 后面填写第 2 步获取到的 key 。如果 key 不对，解密会失败。解密成功后会生成一个 dec.dmg 文件，双击打开即可加载系统镜像。

*   iOS10 以及 iOS10 之后
    

下载对应版本的 iPSW 固件，直接解压，解压后里面有几个 dmg 格式的镜像文件，最大的 dmg 文件就是系统镜像。

![](https://mmbiz.qpic.cn/mmbiz_png/Ce6bSqXkduyh3Q1I7w3kII9QxLVNE03hm9zJgy5YxYZHWG3qC9IuZ6sLOCbtZjccDzOncaqykqUto90GesHMqQ/640?wx_fmt=png)

#### 2. 系统库符号提取

从 iPhone OS 3.1 开始，所有的系统库都打包成一个文件：dyld_shared_cache_xxx ，其中 xxx 表示具体的架构，此文件位于：/System/Library/Caches/com.Apple.dyld 目录。dyld_shared_cache_xxx 文件的解压可以使用 dyld 中的 dsc_extractor.cpp 代码，但做一定的改动。

a. 首先在 Apple 开源网站下载源码 dyld 库的源码，注意，这里需要下载 dyld-7 的源码。

b. 下载之后，将文件 dsc_extractor.cpp，main 函数前后的代码改为如下代码：

```
#if 1
// test program
 #include <stdio.h>
 #include <stddef.h>
 #include <dlfcn.h>
typedef int (*extractor_proc)(const char* shared_cache_file_path,const char* extraction_root_path,void (^progress)(unsigned current,unsigned total));
int main(int argc, const char* argv[]){
    if ( argc != 4 ) {
        fprintf(stderr,"usage: dsc_extractor <dsc_extractor.bundle path> <path-to-cache-file> <path-to-device-dir>\n");
        return 1;
    }
    void* handle = dlopen(argv[1],RTLD_LAZY);
    if ( handle == NULL ) {
        fprintf(stderr,"dsc_extractor.bundle could not be loaded\n");
        return 1;
    }
    extractor_proc proc = (extractor_proc)dlsym(handle,"dyld_shared_cache_extract_dylibs_progress");
    if ( proc == NULL ) {
        fprintf(stderr,"dsc_extractor.bundle did not have dyld_shared_cache_extract_dylibs_progress symbol\n");
        return 1;
    }
    int result = (*proc)(argv[2],argv[3],^(unsigned c, unsigned total) { printf("%d/%d\n", c, total); } );
    fprintf(stderr, "dyld_shared_cache_extract_dylibs_progress() => %d\n",r esult);
    return 0;
}
#endif
```

c. 在终端上 cd 到 dyld 源码目录 launch-cache 下，在终端命令行编译并生成 dsc_extractor 工具。

```
clang++ -o dsc_extractor dsc_extractor.cpp dsc_iterator.cpp
```

d. 从 Xcode 的包中 /Applications/Xcode.App/Contents/Developer/Platforms/iPhoneOS.platform/usr/lib 中提取出 dsc_extractor.bundle 文件。dsc_extractor.bundle 和要提取的 iOS 系统强关联，比如 iOS14 的系统符号需要导出 Xcode12 里的 dsc_extractor.bundle，而 iOS15 的需要 Xcode 13 Beta 里的。如果不匹配的话，有可能不能提取出系统符号。

![](https://mmbiz.qpic.cn/mmbiz_png/Ce6bSqXkduyh3Q1I7w3kII9QxLVNE03hAEdbMicTOBEhPiaJhKlxkESRicUr9FYbp7Og7rc1wBBq0avs5y5tgia34g/640?wx_fmt=png)

e. 调用如下命令提取出系统符号；如下，最终提取的系统库在目录 17C81 下，我们解析系统符号需要的文件基本为 dylib 和 framework。

```
dsc_extractor  dsc_extractor.bundle   /System/Library/Caches/com.Apple.dyld/dyld_shared_cache_arm64   17C81
```

******4. 在线符号化******

### ******丨****** 4.1 为什么要实现在线符号化

*   打包时候符号文件是由持续集成打包机产生，本地获取有成本。
    ============================
    
*   **方便研发人员快速符号化崩溃日志**。很多时候，崩溃都是在非研发人员 (产品，QA 等) 使用应用的时候发生的；同步到研发人员之后，因为本地环境的差异，在没有打包环境的情况下，研发人员也需要能迅速符号化崩溃堆栈
    ==========================================================================================================
    
*   **线上用户上传的崩溃日志规模大**。大多数崩溃都是发版之后用户使用过程中发生的，如果大量线上日志未经符号化就同步到研发人员，就会增加研发人员的负担，降低问题解决的效率。
    =====================================================================================
    
*   **用户系统多，收集难度大**。用户的系统从 iOS9 到 iOS14 都有，千奇百怪，靠研发人员本地想要解析所有的系统符号纯属臆想。
    ===================================================================
    

### ******丨****** 4.2 在线 App / 动态库符号化

通常情况下，我们只需要符号化极少部分崩溃日志，这种情况下我们在本地就可以符号化了。但当我们的应用上线发版后，崩溃日志日均收集量级可能超百万以上，此时就不适合在 Mac OS 上使用脚本 / 工具符号化了 (在 Mac OS 上使用 symbolicatecrash 命令符号化单个日志时，耗时基本 1 秒以上)。此时，就需要更通用，快速的符号化方式了。  
为了能够在 Linux 服务器上极速符号化 iOS 崩溃日志，我们深入调研了 iOS 本地符号化的原理，在和平台方多次就技术方案进行了调研磋商之后，最终采取了如下方案：

![](https://mmbiz.qpic.cn/mmbiz_jpg/Ce6bSqXkduyh3Q1I7w3kII9QxLVNE03hVibicJbB0miaYhHBCMwTIvI1DWic6Y6VQb90t3tkxICicUfskJUP8GdGbiag/640?wx_fmt=jpeg)

### 生成 mapping 文件

将 dSYM 文件通过脚本提取生成一个 mapping 文件，格式如下：

```
Format:        Mach-O/64-Bit
Arch:        arm64
Symbols:        5
Tool Version::        1.0.1
File Version:   1.0.0
UUID:        e569d81abb2c372e89a2410edc3d368f
Built Time:          2021-07-29 13:31:08
Symbol table:
6c64    6c78    -[ViewController viewDidLoad]   (in TestBacktrace)  (ViewController.m:17)
6c78    6c84    -[ViewController viewDidLoad]   (in TestBacktrace)  (ViewController.m:0)
```

提取操作会涉及到 DWARF 中 debug_line 段数据的符号化，相关提取算法可以参考 DWARF 官方的资料。debug_line 段包含有详细的代码偏移量地址和文件名称，按照 DWARF 的算法就可以解析出来，然后与 Symbol Table 的函数符号一一匹配，就能生成代码地址偏移量与函数、文件、行数的映射关系。需要注意的是，苹果的 Mach-O 现在大部分格式都是使用 DWARF2 和 DWARF4 版本，提取的时候需要重点关注这两种格式的兼容和算法不同。最终，可以看到 Symbol table 每一行对应一个符号的偏移量。可以发现 7464 刚好处于 7454 - 7478 之间，匹配出来的符号刚好是 -[AppDelegate Application:didFinishLaunchingWithOptions:] (in TestBacktrace) (AppDelegate.m:23) ，与 Mac OS 上使用 symbolicatecrash 脚本符号化的结果一致。

![](https://mmbiz.qpic.cn/mmbiz_png/Ce6bSqXkduyh3Q1I7w3kII9QxLVNE03hRWVLECSUfwvibeqrOskDz9ib0AupfqLjMqqVbWQAN0N7IT6bIP3fomJg/640?wx_fmt=png)

### 根据 mapping 文件符号化

借助于脚本工具提取的符号 mapping 文件，服务端就能够脱离平台限制，根据崩溃日志中的 UUID 去匹配映射文件，在 Linux 上极速符号化崩溃日志，提供高效实时的符号化服务。

### ******丨****** 4.3 在线符号化 iOS 系统库符号

在 Mac OS 平台上，我们可以直接使用系统库的符号直接使用脚本去符号化符号，但是一旦要符号化所有用户上传的崩溃日志，这一套机制就难免被速度和平台限制。并且 iOS 系统从 2.0 开始，一直到现在 iOS 14 ，发出的版本几百个，要手动提取出系统库符号几乎是不可能的事情。为了解决这个问题，在借鉴了 dSYM 跨平台符号化方案之后，我们做了一套系统符号自动化符号化的方案，最终实现了在 Linux 平台上高效实时的符号化系统堆栈。  

![](https://mmbiz.qpic.cn/mmbiz_jpg/Ce6bSqXkduyh3Q1I7w3kII9QxLVNE03hMU71fju4lqsYd0KGQic4FT9JN4jJRYMgvJFkurFTXHYjyDI96VfsHMw/640?wx_fmt=jpeg)

1. 定时从 theiphonewiki 网站上导出各个系统以及最新发布系统的 iPSW 文件下载地址。

2. 解压 iPSW 并加载系统镜像 dmg 文件，找到 dyld_shared_cache_xxx 文件。

3. 使用工具 dsc_extractor 将系统库符号文件导出，导出文件基本为后缀为 dylib 和 framework 的 Mach-O 类型文件。

4. 将所有的 dylib 和 framework 使用工具提取生成如下格式的 mapping 文件。这一步与 dSYM 提取操作会有一定差别，通常来说，系统库只有符号表段，不需要对 debug_line 段做提取，相对比较简单。

```
20b8    20fa    +[ZoomServicesUI enableZoomServices]    (in AccessibilitySettingsLoader)
20fa    2120    +[ZoomServicesUI disableZoomServices]   (in AccessibilitySettingsLoader)
2120    21b6    -[ZoomServicesUI init]  (in AccessibilitySettingsLoader)
21b6    2222    -[ZoomServicesUI dealloc]   (in AccessibilitySettingsLoader)
```

5. 崩溃日志上传到符号化服务器之后，服务器根据崩溃日志中系统库的 UUID 和 mapping 文件中的 UUID 唯一确定 mapping 文件并符号化系统堆栈。

******5. 效果******








===========================

1. 在线符号化系统上线之后，用户的日志经过崩溃组件自动上传到性能之后，符号化解析系统直接将崩溃日志符号化并聚类，最终符号化的崩溃日志详情如下。应用程序的地址最终表现为 函数 + 文件 + 行数，系统堆栈会具体显示出崩溃的函数，整个过程实时且高效。

```
Last Exception Backtrace:
0   CoreFoundation                  0x1ca4cd27c __exceptionPreprocess + 228
1   libobjc.A.dylib                 0x1c96a79f8 objc_exception_throw + 55
2   CoreFoundation                  0x1ca3ded94 -[__NSSingleObjectArrayI objectAtIndex:] + 127
3   TestBacktrace                   0x102a47464 -[AppDelegate Application:didFinishLaunchingWithOptions:] + 29796 (AppDelegate.m:23)
4   UIKitCore                       0x1f6c86e30 -[UIApplication _handleDelegateCallbacksWithOptions:isSuspended:restoreState:] + 411
5   UIKitCore                       0x1f6c88594 -[UIApplication _callInitializationDelegatesForMainScene:transitionContext:] + 3351
6   UIKitCore                       0x1f6c8dd20 -[UIApplication 
```````
Thread 0 name:  Dispatch queue: com.Apple.main-thread
Thread 0 Crashed:
0   libsystem_kernel.dylib          0x00000001ca06a0dc __pthread_kill + 8
1   libsystem_pthread.dylib         0x00000001ca0e3094 pthread_kill$VARIANT$mp + 380
2   libsystem_c.dylib               0x00000001c9fc3ea8 abort + 140
3   libc++abi.dylib                 0x00000001c9690788 __cxa_bad_cast + 0
```

2. 部分来自研发和测试的崩溃日志，平台提供了在线符号化的入口，只需要手动上传崩溃日志到平台，立刻就能把符号化后的崩溃日志下载给相应人员。

******6. 收益******








===========================

1. 线上问题定位速度获得极大提升，从线上发生新增卡顿 / 崩溃问题到具体研发响应时间大大缩减，从发生崩溃到定位问题，基本都在 10 分钟以内。

2. 目前，性能平台日均在线符号化崩溃 / 卡顿日志超百万次，厂内接入产品线超 30+，符号化功能做到了上传即解析，整个过程无需研发人员干预。真正做到了自动化、在线、实时的符号化崩溃、卡顿日志，并实时根据符号化的问题代码定位到具体开发人员，高效的响应并解决线上问题。

******参考资料******








==========================

[1] iOS Crash 分析必备：符号化系统库方法 https://zuikyo.github.io/2016/12/18/iOS%20Crash%E6%97%A5%E5%BF%97%E5%88%86%E6%9E%90%E5%BF%85%E5%A4%87%EF%BC%9A%E7%AC%A6%E5%8F%B7%E5%8C%96%E7%B3%BB%E7%BB%9F%E5%BA%93%E6%96%B9%E6%B3%95/  
[2] 聊聊从 iOS 固件提取系统库符号 http://crash.163.com/#news/!newsId=31  
[3] Xcode 中和 symbols 有关的几个设置 https://www.jianshu.com/p/11710e7ab661  
[4] iOS_SDK https://en.wikipedia.org/wiki/IOS_SDK  
[5] IOS_version_history https://en.wikipedia.org/wiki/IOS_version_history#iOS_14  
[6] dyld 源码下载地址 https://opensource.apple.com/tarballs/dyld/  
[7] The DWARF Debugging Standard http://www.dwarfstd.org/  
[8] iOS9 之前的 Firmware_Keys https://www.theiphonewiki.com/wiki/Firmware_Keys  
[9] dmg 工具下载地址 https://github.com/Zuikyo/iOS-System-Symbols/blob/master/tools/dmg  
[10] 系统符号下载地址索引 wiki https://www.theiphonewiki.com/wiki/Firmware

招聘信息

**移动端高级 / 资深工程师**

我们是百度 APP 移动技术平台团队，负责百度 APP 性能优化、网络优化、架构升级、工程效能提升和新技术探索等。我们在线上线下研发工具，智能性能优化，网络库开发，网络监控，编译构建系统，动态化方案等方面深耕，技术深度上是移动端的灯塔。我们支持业务快速迭代的同时，保证超大规模团队的研发效能，服务于 MAU 超过 6 亿的用户。

我们致力于打造行业内性能最佳的 APPs，在高中低端机上有卓越的性能稳定性和流畅度。

我们致力于建设适合百度的移动技术平台，赋能百度的移动端产品，智能音箱和车载设备。

**技术栈**：Java/OC/C/C++/Kotlin/Swift/Ruby/PHP 等

**支撑业务**：百度 APP 的搜索，Feed，小程序，直播，视频等；百度贴吧，好看视频等其他移动端产品；小度音箱等 IOT 设备；智能交通，车联网等车载设备。

**学历要求**：统招本科毕业

如果你在某个方向有特长，有技术深度；如果你热爱专研，挑战疑难问题；如果你爱突破创新，使用端智能等新技术解决各种问题，这里就是你发挥才能的舞台！

**投递邮箱：**  

**yangying18@baidu.com**
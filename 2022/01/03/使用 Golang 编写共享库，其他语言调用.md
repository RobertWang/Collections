> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU0MTY5MzEwMA==&mid=2247488430&idx=1&sn=77d8f5e89d6a599d005a1781aeab99a3&chksm=fb27566dcc50df7bdd029377b6adbe1323935669519e8f486f14ff7255beb4deed97fbc690f2&scene=132#wechat_redirect)

作为取代 C 语言的一种选择，Golang 语言越来越流行，尤其在云原生和容器界很多基础软件都用 golang 编写。作为基础软件的一环，底层库的开发也是非常重要，基于 golang 进行底层库开发也成了一种需求。从 golang 1.5 版开始，go 编译器通过 - buildmode 标识。被称为 Go 执行模式构建模式，它可是实现将 Go 包编译成多种格式，包括 Go 归档、Go 共享库、C 归档、C 共享库和以及在 Golang1.8 中引入的动态插件。

今天我们就实例展示一下如何将 Golang 包编译成 C 共享库，然后供其他语言调用。在这种构建模式下，编译器输出一个标准的共享对象二进制文件（.so) 将 Go 函数公开为 C 风格的 API。本文，将展示如何创建可以从 C 、 Python 、 Ruby 、 Node 和 Java 调用的 Go 库。

**编写 Golang 共享库**
=================

假设我们已经编写了一个 Golang 库，要把将其提供给其他语言调用。在将代码编译成共享库使用，必须支持：

包必须为 main 打包。编译器会将包及其所有依赖项构建到单个共享对象二进制文件中。

源必须 imort 包 “C”.

使用 //export 注释以注释希望其他语言可以访问的函数。

一个空的 main 必须声明函数。

下面的 Golang 代码，export Add, Cosine, Sort 和 Log 函数共调用：

```
package main
import "C"
import (
"fmt"
"math"
"sort"
"sync"
)
var count int
var mtx sync.Mutex
func Add(a, b int) int {
return a + b
}
func Cosine(x float64) float64 {
return math.Cos(x)
}
func Sort(vals []int) {
sort.Ints(vals)
}
func SortPtr(vals *[]int) {
Sort(*vals)
}
func Log(msg string) int {
mtx.Lock()
defer mtx.Unlock()
fmt.Println(msg)
count++
return count
}
func LogPtr(msg *string) int {
return Log(*msg);
}
func main() {}
```

该包使用 - buildmode=c-shared 选项，可以在构建时创建共享对象二进制文件：

```
go build -o chognchong.so -buildmode=c-shared chonghcong.go
```

完成后，编译器输出两个文件, 一个是 C 头文件 chonghcong.h，另一个为 chonghcong.so 共享对象文件：

```
-rw-rw-r-- 1 1779 Dec 9 15:50 chongchong.h
-rw-rw-r-- 1 3748095 Dec 9 15:50 chongchong.so
```

请注意，.so 文件大约为 3.7Mb，对这么简单的几个函数，相对来说编译的库文件较大，这也是 golang 的缺点之一，主要因为编译后的库中要嵌入整个 Golang 运行时（类似于编译单个静态可执行文件）。

头文件
===

头文件定义了映射到 Go 兼容类型的 C 类型，其内容如下（部分截图，主要是 cgo 语法）

![](https://mmbiz.qpic.cn/mmbiz_png/hMIkz6ibKC9uX62ImDkuMUYgnEXRd61c2Ribuw1uHOkjxd8Xp0RGibGd7wqsgvXZopn23wKNT3vRic2x94Xqjeum6g/640?wx_fmt=png)

...

共享对象文件
======

编译器生成的另一个文件是 64 位 ELF 共享对象二进制文件。我们可以使用 file 命令。

```
file chongchong.so
chongchong.so: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, not stripped
```

使用 nm 和 grep 命令，可以检查 Golang 函数是否被成功 export。

```
nm chongchong.so | grep -e "T Add" -e "T Cosine" -e "T Sort" -e "T Log"
00000000000d0db0 T Add
00000000000d0e30 T Cosine
00000000000d0f30 T Log
00000000000d0eb0 T Sort
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/hMIkz6ibKC9uX62ImDkuMUYgnEXRd61c2jeo7HKpqPKYIvpJKuCiaLIhlSw36k6LYyWNianKPEdkdJM6kSflGrCxQ/640?wx_fmt=jpeg)

接下来，将实例展示如何从其他语言调用导出的函数。

**C 调用**
========

有两种方法可以使用共享库从 C 调用 Go 函数：

代码可以在编译时与共享库静态绑定，但在运行时动态链接。

Golang 函数在运行时动态加载和绑定。

动态链接
====

在这种方法中，我们使用头文件来静态引用共享对象文件中导出的类型和函数。代码简单干净如下图（部分打印语句省略）：

```
#include <stdio.h>
#include "chongchong.h"int main() {
GoInt a = 12;
GoInt b = 99;
printf("chognchong.Add(12,99) = %d\n", Add(a, b));
printf("chongchong.Cosine(1) = %f\n", (float)(Cosine(1.0)));
GoInt data[6] = {77, 12, 5, 99, 28, 23};
GoSlice nums = {data, 6, 6};
Sort(nums);
for (int i = 0; i < 6; i++){
printf("%d,", ((GoInt *)nums.data)[i]);
}
GoString msg = {"Hello from C!", 13};
Log(msg);
}
```

接下来编译代码，指定共享对象库：

```
gcc -o example cc.c ./chongchong.so
报错
c.c: In function ‘main’:
cc.c:12: error: ‘for’ loop initial declarations are only allowed in C99 mode
cc.c:12: note: use option -std=c99 or -std=gnu99 to compile your code
```

根据提示，我们添加 - std=gnu99 选项再进行编译

```
gcc -o example cc.c ./chongchong.so -std=gnu99
```

然后执行程序

```
./example
```

，二进制链接到 chongchong.so 库，产生下面的输出。

```
chongchong.Add(12,99) = 111
chongchong.Cosine(1) = 0.540302
Hello from C!
5,12,23,28,77,99,
```

动态加载
====

在这种方法中，C 代码使用动态链接加载程序库（libdl.so) 动态加载并绑定到导出的符号。使用定义在 dhfcn.h 如 dlopen 打开库文件，用 dlsym 查找符号， dlerror 检索错误，最后用 dlclose 关闭共享库文件。

```
#include <stdlib.h>
#include <stdio.h>
#include <dlfcn.h>
typedef long long go_int;
typedef double go_float64;
typedef struct{void *arr; go_int len; go_int cap;} go_slice;
typedef struct{const char *p; go_int len;} go_str;
int main(int argc, char **argv) {
void *handle;
char *error;
handle = dlopen ("./chongchong.so", RTLD_LAZY);
if (!handle) {
fputs (dlerror(), stderr);
exit(1);
}
go_int (*add)(go_int, go_int) = dlsym(handle, "Add");
if ((error = dlerror()) != NULL) {
fputs(error, stderr);
exit(1);
}
go_int sum = (*add)(12, 99);
printf("chongchong.Add(12, 99) = %d\n", sum);
go_float64 (*cosine)(go_float64) = dlsym(handle, "Cosine");
if ((error = dlerror()) != NULL) {
fputs(error, stderr);
exit(1);
}
go_float64 cos = (*cosine)(1.0);
printf("chongchong.Cosine(1) = %f\n", cos);
void (*sort)(go_slice) = dlsym(handle, "Sort");
if ((error = dlerror()) != NULL) {
fputs(error, stderr);
exit(1);
}
go_int data[5] = {44,23,7,66,2};
go_slice nums = {data, 5, 5};
sort(nums);
printf("chongchong.Sort(44,23,7,66,2): ");
for (int i = 0; i < 5; i++){
printf("%d,", ((go_int *)data)[i]);
}
printf("\n");
go_int (*log)(go_str) = dlsym(handle, "Log");
if ((error = dlerror()) != NULL) {
fputs(error, stderr);
exit(1);
}
go_str msg = {"Hello from C!", 13};
log(msg);
dlclose(handle);
}
```

在上述代码中自定义了 C 类型：go_int,go_float,go_slice 和 go_str。用 dlsym 函数加载函数符号并将它们分配给各自的函数指针。然后编译代码：

```
gcc -o example1 cc1.c -ldl -std=gnu99
```

执行代码时，C 二进制文件加载并链接到共享库 chongchong.so 产生以下输出：

```
./example1
chongchong.Add(12, 99) = 111
chongchong.Cosine(1) = 0.540302
chongchong.Sort(44,23,7,66,2): 2,7,23,44,66,
Hello from C!
```

**Python**
==========

Python 中，调用 C 共享库非常容易，可以使用 ctypes 库调用导出的 Go 函数，如下面的代码片段所示。

```
from __future__ import print_function
from ctypes import *
lib = cdll.LoadLibrary("./chongchong.so")
lib.Add.argtypes = [c_longlong, c_longlong]
lib.Add.restype = c_longlong
print("chongchong.Add(12,99) = %d" % lib.Add(12,99))
lib.Cosine.argtypes = [c_double]
lib.Cosine.restype = c_double
print("chongchong.Cosine(1) = %f" % lib.Cosine(1))
class GoSlice(Structure):
_fields_ = [("data", POINTER(c_void_p)), ("len", c_longlong), ("cap", c_longlong)]
nums = GoSlice((c_void_p * 5)(74, 4, 122, 9, 12), 5, 5)
lib.Sort.argtypes = [GoSlice]
lib.Sort.restype = None
lib.Sort(nums)
print("chongchong.Sort(74,4,122,9,12) = %s" % [nums.data[i] for i in range(nums.len)])
class GoString(Structure):
_fields_ = [("p", c_char_p), ("n", c_longlong)]
lib.Log.argtypes = [GoString]
lib.Log.restype = c_longlong
msg = GoString(b"Hello Python!", 13)
```

这 lib 变量表示从共享目标文件加载的符号。Python 类 GoString 和 GoSlice 映射到它们各自的 C 结构类型。执行 Python 代码时，它会调用共享对象中的 Go 函数，产生以下输出：

```
python cc.py
chongchong.Add(12,99) = 111
chongchong.Cosine(1) = 0.540302
chongchong.Sort(74,4,122,9,12) = [4, 9, 12, 74, 122]
Hello Python!
```

**ruby**
========

Ruby 中调 oG 函数和 Python 也类似，使用 FFI gem 动态加载和调用导出的 Go 函数，如下面的代码片段所示。

```
require 'ffi'
module Cc
extend FFI::Library
ffi_lib './chongchong.so'
# define class GoSlice to map to:
# C type struct { void *data; GoInt len; GoInt cap; }
class GoSlice < FFI::Struct
layout :data, :pointer,
:len, :long_long,
:cap, :long_long
end
class GoString < FFI::Struct
layout :p, :pointer,
:len, :long_long
end
attach_function :Add, [:long_long, :long_long], :long_long
attach_function :Cosine, [:double], :double
attach_function :Sort, [GoSlice.by_value], :void
attach_function :Log, [GoString.by_value], :int
end
print "chongchong.Add(12, 99) = ", Cc.Add(12, 99), "\n"
print "chongchong.Cosine(1) = ", Cc.Cosine(1), "\n"
nums = [92,101,3,44,7]
ptr = FFI::MemoryPointer.new :long_long, nums.size
ptr.write_array_of_long_long nums
slice = Cc::GoSlice.new
slice[:data] = ptr
slice[:len] = nums.size
slice[:cap] = nums.size
Cc.Sort(slice)
sorted = slice[:data].read_array_of_long_long nums.size
print "chongchong.Sort(", nums, ") = ", sorted, "\n"
msg = "Hello Ruby!"
gostr = Cc::GoString.new
gostr[:p] = FFI::MemoryPointer.from_string(msg)
gostr[:len] = msg.size
```

在 Ruby 中，FFI::libraryclass 被扩展为声明一个加载导出符号的类。GoSlice 和 GoString 类映射到它们各自的 C 结构。当运行代码时，它会调用导出的 Go 函数，如下所示：

```
ruby cc.rb
chongchong.Add(12,99) = 111
chongchong.Cosine(1) = 0.540302
chongchong.Sort(74,4,122,9,12) = [4, 9, 12, 74, 122]
Hello Ruby!
```

**node.js**
===========

Node 使用一个名为的外部函数库 node-ffi（和其他依赖库）动态加载和调用导出的 Go 函数，如下面的代码片段所示：

```
require 'ffi'
module Cc
extend FFI::Library
ffi_lib './chongchong.so'
class GoSlice < FFI::Struct
layout :data, :pointer,
:len, :long_long,
:cap, :long_long
end
class GoString < FFI::Struct
layout :p, :pointer,
:len, :long_long
end
attach_function :Add, [:long_long, :long_long], :long_long
attach_function :Cosine, [:double], :double
attach_function :Sort, [GoSlice.by_value], :void
attach_function :Log, [GoString.by_value], :int
end
print "chongchong.Add(12, 99) = ", Cc.Add(12, 99), "\n"
print "chongchong.Cosine(1) = ", Cc.Cosine(1), "\n"
nums = [92,101,3,44,7]
ptr = FFI::MemoryPointer.new :long_long, nums.size
ptr.write_array_of_long_long nums
…
```

ffi 对象管理从共享库加载的符号。节点 Sturct 对象用于创建 GoSlice 和 GoString 映射到它们各自的 C 结构。当运行代码时，它会调用导出的 Go 函数，如下所示：

```
chongchong.Add(12, 99) = 111
chongchong.Cosine(1) = 0.5403023058681398
chongchong.Sort([12,54,9,423,9] = [ 0, 9, 12, 54, 423 ]
Hello Node!
```

**Java**
========

为在 Java 中调用导出的 Go 函数，需要 Java Native Access 或 JNA，，如以下代码片段所示：

![](https://mmbiz.qpic.cn/mmbiz_png/hMIkz6ibKC9uX62ImDkuMUYgnEXRd61c2NIl9yuzPlxjKicxVGWUm1YD31jgDrG0YZFtv4eeUY21YMxhNcYCfRkg/640?wx_fmt=png)

Java 接口 Chonghcong 从 chongchong.so 共享库文件加载的符号。类 GoSlice 和 GoString 映射到它们各自的 C 结构表示。当代码编译运行时，它调用导出的 Go 函数，如下所示：

用 javac 编译该代码，记得用 - cp 引入 jna 的 jar 包

```
javac -cp jna.jar Cc.java
```

会编译成功 Cc.class, 然后 java 执行

```
java -cp .:jna.jar Cc
chongchong.Add(12, 99) = 111
chongchong.Cosine(1.0) = 0.5403023058681398
chongchong.Sort(53,11,5,2,88) = [2 5 11 53 88 ]
Hello Java!
```

**结论**
======

本文中 ，我们介绍了如何创建一个 Golang 的函数共享库，通过将 Golang 包编译成 C 风格的共享库，Go 程序员可以使用共享对象二进制文件集成，然后可以在各种语言，包括 C、Python、Ruby、Node、Java 等轻松地使用。
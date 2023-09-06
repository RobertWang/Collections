> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/arv002/article/details/122955707)

go 语言有很多的库, 在开发 c 语言的时候为了快速完成某些功能, 可以直接通过 go 语言的库进行调用. 那么我们将 go 语言开发的程序编译成 so 动态库文件就可以给 c 或者 c++ 程序调用.

**目录**

[go 文件编写](#t0)

[so 编译命令](#t1)

[C 语言测试程序](#t2)

[编译 C 程序](#%E7%BC%96%E8%AF%91C%E7%A8%8B%E5%BA%8F)

go 文件编写
=======

如果想要导出 C 能使用的动态库. 我们需要将函数的名称编写为大写.

并且还需要添加导出注释 (重要) //export HelloWorld 这个注释必须写, 并且 // 后面不要有空格.

```
package main
 
/*
#include <stdlib.h>
*/
import "C"
import "fmt"
 
//export HelloWorld
func HelloWorld(str *C.char) *C.char{
	fmt.Printf(C.GoString(str))
	return str
}
 
func main() {
 
	// fmt.Printf("Done\n")
}
```

so 编译命令
=======

```
go mod init test
go mod tidy
go build -buildmode=c-shared -o libtest.so
```

编译成功后会自动生成 libtest.so 以及 libtest.h 文件

可以通过一下命令检查是否编写编译成功.

```
readelf -a libtest.so| grep -i hello


```

C 语言测试程序
========

main.c 文件

```
#include "libtest.h"
void main(int argc, char* argv[]) {
    char * str =  HelloWorld("asdf");
    print(str);
}
```

如果 libtest.h 文件为自动生成, 我们可以自己手动编写 libtest.h 文件

```
#ifndef __INTERFACE_H__
#define __INTERFACE_H__
 
char* HelloWorld(char *);
 
#endif
```

CMakeList.txt 编写

```
 
cmake_minimum_required(VERSION 2.8)
project(demo)
# C++11 编译
set(CMAKE_BUILD_TYPE "Debug")
set(CMAKE_CXX_STANDARD 11)
# set(CMAKE_SHARED_LINKER_FLAGS " -L./ -Wl,-rpath=./")
SET(CMAKE_INCLUDE_CURRENT_DIR ON)
SET(CMAKE_CXX_STANDARD_REQUIRED ON)
 
set( demo_SOURCE
   ${CMAKE_CURRENT_SOURCE_DIR}/main.c
)
 
link_directories(${PROJECT_SOURCE_DIR})
 
SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR})
 
add_executable(demo ${demo_SOURCE})
 
target_link_libraries(demo -ltest ) 
 
```

编译 C 程序
=======

```
mkdir build
cd build
cmake ..
make
```

运行

```
./demo

```

高级用法
====

注意：go 语言中申请的内存不要直接 c 中使用，因为 go 语言的内存管理是自己决定的，因此你不知道他在什么时候自动释放，因此如果需要在 go 语言中使用内容请使用 c.malloc 来申请内容返回到 C 语言中。C 语言注意要自己调用 free 释放哦。

```
func add() (**C.char, *C.char) {
        //a := []string{"1111111111111111111", "222", "333"}
        a, err := Parsefilelist()
        if err != nil {
                fmt.Println(err)
                return nil, C.CString(err.Error())
        }
        count := len(a)
        c_count := C.int(count)
        //使用完请释放该内存空间，防止内存泄露
        cArray := C.malloc(C.size_t(c_count) * C.size_t(unsafe.Sizeof(uintptr(0))))
        b := (*[1024]*C.char)(cArray)
        for index, value := range a {
                b[index] = C.CString(value)
        }
        return (**C.char)(&b[0]), nil
}
```
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666556038&idx=2&sn=a9111c420c0e0de5a91a0d724a19d429&chksm=80dcae2db7ab273b38fa7efe48acb176868e4d996835e73a4875cee4131a78cedd08687650c4&scene=21#wechat_redirect)

Breakpad 是 Google 用 C++ 编写的一个开源、跨平台的崩溃报告系统，它支持 Windows、Linux 和 macOS，并提供了一个上传器，可以在进程崩溃时向一个配置好的 URL 提交 minidump 文件。  

目前，有很多大型项目都在使用 Breakpad，例如：Google Chrome、Firefox、Google Picasa、Camino、Google Earth 等。

*   主页：https://chromium.googlesource.com/breakpad/breakpad/
    
*   文档：https://chromium.googlesource.com/breakpad/breakpad/+/HEAD/docs
    
*   GitHub 地址：https://github.com/google/breakpad
    

1

**工作原理**

BreakPad 工作原理：

![](https://mmbiz.qpic.cn/mmbiz_png/714ibKo8XksaHYnTmtQd9PNr2Yw8oGiav1DG9iaibhCGLXZib2K3mhFDBuQNhRXcrNq6iatxfJafKRd9rQBXw8QfSLLA/640?wx_fmt=png)

其中，包含了三个主要组件：

*   **Breakpad client**：是一个库（即：libbreakpad_client.a），将来要集成到我们的程序中。用于写 minidump 文件，捕获当前线程的状态，以及可执行文件 / 共享库的标识。
    
*   **Breakpad 符号转储工具**：是一个程序（即：dump_syms），用于读取由编译器产生的调试信息，并以 Breakpad 自己的格式生成一个符号文件。
    
*   **Breakpad minidump 处理器**：是一个程序（即：minidump_stackwalk），用于读取 minidump 文件和符号文件，并生成一个可读的 C/C++ 堆栈跟踪。
    

2

**编译安装**

1.  下载 Breakpad 源码；
    
2.  由于 Breakpad 依赖于 LSS，所以还需要下载它（地址：https://github.com/adelshokhy112/linux-syscall-support）；
    
3.  将 LSS 中的 `linux_syscall_support.h` 文件放至 `breakpad/src/third_party/lss/` 目录下。
    
4.  编译 Breakpad，步骤非常简单：
    

```
$ cd breakpad
$ ./configure && make
$ make
$ sudo make install
```

成功之后，会生成 `libbreakpad_client.a` 库文件，以及 `dump_syms`、`minidump_stackwalk` 等程序。

3

**将 Breakpad 集成到程序中**

和其他第三方库的用法一样，要将 Breakpad 集成到程序中，先要链接生成的库 `libbreakpad_client.a`，并设置包含路径为 `breakpad/src/`，接下来就可以 include 头文件了。

为了生成 mindump 文件，我们需要实例化一个 `ExceptionHandler` 对象，并提供一个存储 mindump 的路径，以及一个回调函数来接收关于已写入的 mindump 的信息：

```
#include "client/linux/handler/exception_handler.h"
#include <iostream>

static bool dumpCallback(const google_breakpad::MinidumpDescriptor& descriptor,
                         void* context,
                         bool succeeded)
{
    std::cout << "Dump path:" << descriptor.path() << std::endl;

    return succeeded;
}

void crash()
{
    int* a = nullptr;
    *a = 1;
}

int main(int argc, char* argv[])
{
    google_breakpad::MinidumpDescriptor descriptor("/tmp");
    google_breakpad::ExceptionHandler eh(descriptor, nullptr, dumpCallback, nullptr, true, -1);
    crash();

    return 0;
}
```

编译运行这个程序，会在 `/tmp/` 目录下生成一个 minidump 文件，并在退出之前打印该文件的路径。

**注意：**在使用 Breakpad 时，需要目标程序内包含调试信息，这样 `dump_syms` 工具才能从中解析出调试符号。而在默认情况下，Release 模式编译的程序是并不包含调试信息，若想包含，需要在编译程序时添加 `-g` 选项。

4

**生成可读的堆栈跟踪**

带调试信息的目标程序有了，minidump 文件有了，`dump_syms` 和 `minidump_stackwalk` 等工具也有了，现在要做的就是生成有用的堆栈跟踪信息了！

**1. 生成符号文件**

Breakpad 要求我们将目标程序中的调试符号转换为文本格式的符号文件，这一步需要通过 `dump_syms` 工具来完成。

假设，我们的程序名为 `test`，执行以下命令，便会生成一个名为 `test.sym` 的符号文件：

```
$ dump_syms ./test > test.sym
```

**2. 将符号文件放在特定目录下**

为了使用这些符号，需要将生成的符号文件放在特定的目录结构中：

![](https://mmbiz.qpic.cn/mmbiz_png/714ibKo8XksaHYnTmtQd9PNr2Yw8oGiav1FPt8YcjpDXrOVadnLjvtjA5o97lJEDfkDGDlwSwb7GkibibfMUaOvy5w/640?wx_fmt=png)

查看符号文件，第一行包含了生成目录结构所需的信息：

```
$ head -n1 test.sym 
MODULE Linux x86_64 95E20E34BE203CB093B675D606A7D7D20 test
```

创建以上目录结构，并将符号文件移动到该路径下：

```
$ mkdir -p ./symbols/test/95E20E34BE203CB093B675D606A7D7D20
$ mv test.sym ./symbols/test/95E20E34BE203CB093B675D606A7D7D20/
```

**3. 生成堆栈跟踪信息**

当一切准备就绪，`minidump_stackwalk` 工具就派上用场了。

只需将 mindump 文件和符号路径作为命令行参数传递给它，便能生成堆栈跟踪信息了。为了便于分析，可以将输出重定向至文件中：  

```
$ minidump_stackwalk /tmp/48596758-1d7a-4318-15edb4af-a9186ad7.dmp ./symbols > error.log
```

5

**定位 Crash 所在位置**

打开 `error.log`，搜索关键字`“crashed”`，一般与它最接近的一行，就是发生崩溃时程序的堆栈信息：

![](https://mmbiz.qpic.cn/mmbiz_png/714ibKo8XksaHYnTmtQd9PNr2Yw8oGiav1ZiaXUpl2JuEh59WjIPicdcsakzgIqwgTnv3fz5UMZFgjibPibfsydmft9A/640?wx_fmt=png)

可以很清楚地看到，崩溃发生在 main.cpp 的第 16 行。

查看我们的测试程序，与预期结果一样：

![](https://mmbiz.qpic.cn/mmbiz_png/714ibKo8XksaHYnTmtQd9PNr2Yw8oGiav10JVMWuJy2jZaL5MBkGykicexiboG06JkOtG6nViaxlicq0da210OkLia2vg/640?wx_fmt=png)

至此，Crash 所在位置已经准确地定位到了，赶紧抓紧时间修改吧！

6

**将上述过程脚本化**

由于以上过程会经常执行，可以将其做成自动化脚本，不仅可有效防止步骤命令的遗忘，而且极大的加快了操作速度。

新建一个脚本 `dumptool.sh`，内容如下：

```
#!/bin/bash

if [ $# != 3 ] ; then 
echo "USAGE: $0 TARGET_NAME DMP_NAME OUTPUT_NAME" 
echo " e.g.: $0 test 48596758-1d7a-4318-15edb4af-a9186ad7.dmp error.log" 
exit 1; 
fi 

#获取输入参数
target_file_name=$1
dmp_file_name=$2
output_file_name=$3

getSymbol() {
    echo "@getSymbol: start get symbol"
    dump\_syms ./$target_file_name > $target_file_name'.sym'
}

getStackTrace() {
    echo "@getStackTrace: start get StackTrace"
    sym_file_name=$target_file_name'.sym'

    #获取符号文件中的第一行
    line1=`head -n1 $sym_file_name`

    #从第一行字符串中获取版本号
    OIFS=$IFS; IFS=" "; set -- $line1; aa=$1;bb=$2;cc=$3;dd=$4; IFS=$OIFS 

    version_number=$dd

    #创建特定的目录结构，并将符号文件移进去
    mkdir -p ./symbols/$target_file_name/$version_number
    mv $sym_file_name ./symbols/$target_file_name/$version_number

    #将堆栈跟踪信息重定向到文件中
    minidump_stackwalk $dmp_file_name ./symbols > $output_file_name
}

main() {
    getSymbol 
    if [ $? == 0 ] 
    then 
        getStackTrace
    fi
}

#运行main
main
```

使用比较简单，先为脚本增加可执行权限：

```
$ chmod +x ./dump_tool.sh
```

然后执行下述命令，将二进制文件、minidump 文件、输出日志文件作为参数传递给它：

```
$ ./dumptool.sh ./test /tmp/48596758-1d7a-4318-15edb4af-a9186ad7.dmp error.log
```

完成之后，所有的堆栈跟踪信息就会被输出到 `error.log` 文件中了。

很好用吧，学会这个绝招，让 C++ 崩溃无处可逃！
-------------------------

- EOF -

1、[C++ 后台开发知识点及学习路线](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666555749&idx=2&sn=80292e2498dbaeea9c23e647a59064fd&chksm=80dcadceb7ab24d8495f7727368f8f9fde222e10f50fa4823701c3703dbeae525f9ae0c4ca66&scene=21#wechat_redirect)

2、[两万字总结《C++ Primer》要点](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666554018&idx=2&sn=badef2bb5bbfa553b45481b8e3268041&chksm=80dca609b7ab2f1f3b668204f83585bd17b23d19fd70445eb7caf25815732f40d89744e35de5&scene=21#wechat_redirect)

3、[C++ inline 函数简介](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666553386&idx=3&sn=c1319e36e93c1eae8d8feadafa5601fd&chksm=80dca481b7ab2d97d26ced5c5a38bbd64e7bd610a2cfe6aa8abd0e2e0b8dda65c922969b8022&scene=21#wechat_redirect)
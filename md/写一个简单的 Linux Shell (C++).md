> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/zxuuu/p/13701489.html)

这里可以找到代码
--------

[github.com/z0gSh1u/expshell](https://github.com/z0gSh1u/expshell)

支持的特性
-----

*   单条指令的执行
*   引号引起的参数（如 `$ some_program "hello, world"` ）
*   重定向（>、< ）
*   管道（|）
*   内建指令（如 cd、history、quit）
*   指令别名（如 ll → ls -l）
*   家目录（~）

运行截图
----

[![](https://i.loli.net/2020/09/17/ACyfLkH49c2dlBQ.png)](https://i.loli.net/2020/09/17/ACyfLkH49c2dlBQ.png)

如何写一个简单的 Shell
--------------

这里简单介绍写 Shell 时比较关键的一些部分，具体请查看源代码。

### 展示提示符

见 `show_command_prompt` 函数。

command_prompt 是在每行最开始显示的一段与用户名、路径等相关的提示信息。ExpShell 显示的 prompt 形如 `[root@localhost tmp]>`。用 > 而非 #、$ 作为提示符，以区分原生 Shell。

*   获取用户名
    
    ```
    passwd *pwd = getpwuid(getuid());
    string username(pwd->pw_name);
    ```
    
*   获取当前目录
    
    ```
    getcwd(char_buf, CHAR_BUF_SIZE);
    string cwd(char_buf);
    ```
    
    *   prompt 中目录只显示最近一级，此处用 `/` 来 split 后取最后一个即可
    *   家目录需要折叠为 `~`，这里顺便把家目录地址存到全局变量 `home_dir`，后续要用到
*   获取主机名
    
    ```
    gethostname(char_buf, CHAR_BUF_SIZE);
    string hostname(char_buf);
    ```
    
    *   有时 hostname 会是形如 `localhost.locald.xxx` 的形式，也 split 处理一下
*   输出之即可
    
    ```
    cout << "[" << username << "@" << hostname << " " << cwd << "]> ";
    ```
    

### 解析命令

*   为存储解析结果，定义如下四个类：
    
    *   cmd：各种 cmd 的基类
    *   exec_cmd：形如 `argv[0] argv[1] ...` 的普通命令
    *   pipe_cmd：管道命令，形如 `left: cmd* | right: cmd*`
    *   redirect_cmd：重定向命令，形如 `cmd_: cmd* > (or <) file`
*   （最基础的）解析 exec_cmd
    
    见 `parse_exec_cmd` 函数。注意这里使用 `string_split_protect` 函数来 split 出 argv，这样可以保持被引号引起的带空格的 argument 不被拆分。
    
*   解析一条命令
    
    见 `parse` 函数。采用分治法递归地解析命令。
    
    *   从左到右扫描字符串
    *   如果是普通字符，则读入缓存
    *   如果是重定向符号，将当前缓存解析为 exec_cmd，作为左手边 cmd；继续不断读入直到再次遇到符号或字符串结束，作为右手边 file，构建 redirect_cmd
    *   如果是管道符号，递归地调用 `parse` 解析右侧剩余，解析结果作为本层递归的右手边，构建 pipe_cmd
*   解析内建命令
    
    主要支持 cd 、history 和 quit 命令。
    
    *   调用 `exit(0)` 即可实现 quit
    *   history 命令根据记录打印即可
    *   对于 cd，考虑如下情况
        *   无参 cd 等价于 `cd ~`
        *   对于形如 `cd ~/some_path` 的命令，使用 `home_dir` 替换 `~`
        *   其他情况调用 `chdir` 即可

### 执行命令

主要见 `run_cmd` 函数。该函数接收一个 `cmd*`，递归地完成其链上所有 cmd 的执行。

*   对于 exec_cmd
    
    *   检查别名，替换别名，例如 ll → ls -l
    *   使用 `execvp` 函数执行命令
        *   看 [这篇博文](https://blog.csdn.net/yychuyu/article/details/80173039) 了解 exec 族函数，可见 `execvp` 在当前场景最为合适
        *   第二个参数是一个末元素为 NULL 的 char**（char*[]），内容为 argv
*   对于 pipe_cmd
    
    *   为 pipe_cmd 的 left 和 right 分别 fork 子进程执行，并使用管道让这两个兄弟进程通信
        
    *   这张图很好地说明了父子进程使用管道通信的方法
        
        [![](https://i.loli.net/2020/09/16/BdNUL7pRGfF2rgD.png)](https://i.loli.net/2020/09/16/BdNUL7pRGfF2rgD.png)
        
    *   依据上图，不难类比出兄弟进程进行 IPC 的方法如下
        
        *   父进程 pipe
        *   父进程 fork 两次
        *   child1 关读端，重定向写端，执行命令，关写端
        *   child2 关写端，重定向读端，执行命令，关读端
        *   父进程关闭读、写端，并 wait
*   对于 redirect_cmd
    
    *   打开 file，获得 fd
    *   重定向 stdin 或 stdout 到 fd
    *   执行命令
    *   关闭 fd

### 主函数

在一个死循环中读入当前命令，如果不是 builtin_command，则 fork 子进程进行解析和执行（避免阻塞 ExpShell 自身），执行完成后子进程 exit。

### 其他细节

*   pipe、open、dup2 等方法返回值小于 0 均表示出现错误，需要触发 panic
*   对于 wait 方法的状态字，当 `WIFEXITED(status)` 为 0 时表示子进程异常退出，使用 `WEXITSTATUS(status)` 可以进一步获得子进程的 exit code
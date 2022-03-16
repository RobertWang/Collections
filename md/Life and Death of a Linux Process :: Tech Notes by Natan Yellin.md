> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [natanyellin.com](https://natanyellin.com/posts/life-and-death-of-a-linux-process/)

> This post contains a rough sketch of the life and death of a process on Linux. It is a first-order ap......

This post contains a rough sketch of the life and death of a process on Linux. It is a first-order approximation only. A later post will refine this further and provide a more precise description, adding details about pid namespaces, obscure syscalls, and little known flags.

Birth
-----

Every time a process is created, really another process split itself using the [_fork_](https://en.wikipedia.org/wiki/Fork_(system_call)) or [_clone_](https://en.wikipedia.org/wiki/Fork_(system_call)#Linux_clone_syscall) syscall. After _forking_, processes usually run the [_execve_](https://en.wikipedia.org/wiki/Exec_(system_call)) syscall to swap the currently executing binary with another one. For example, when you run _ls_ from a _bash_ shell then first _bash_ splits itself into two _bash_ processes using _fork_ and then the child _bash_ shell uses _exec_ to change itself into _ls_. The child dies when _ls_ finishes executing leaving only the original _bash_ process.

In practice, programs almost never call the syscalls directly - they use libc wrappers instead or a libc function like [_system_](https://man7.org/linux/man-pages/man3/system.3.html) which under the hood uses _fork_ and _execve_ (or one of several _execve_ variants).

Death
-----

How do processes die? They almost always call the [_exit_](https://linux.die.net/man/2/exit) or [_exit_group_](https://linux.die.net/man/2/exit_group) syscalls. If the programmer doesn’t explicitly call _exit_ and instead returns from _main_ then _exit_ is called anyway because the compiler wrapped _main_ with a libc _main_ that calls _exit_ for you. If the program was compiled without libc and the programmer doesn’t call _exit_ explicitly then returning from _main_ will cause a segfault or another critical signal because return will try to pop an illegal return address from the stack. This brings us to the second-to-last way that processes can die which is via signals like _SIGTERM_ or _SIGKILL_. Finally, the last way for a process to die is to pull the plug on your computer.

Identity and Zombiehood
-----------------------

Every process has a unique _pid_ - or at least it is unique until the kernel recycles the _pid_ sometime after the process dies. Before a process’ _pid_ can be recycled, the parent process should call [_wait_, _waitpid_, or _waitid_](https://linux.die.net/man/2/wait) on the child. (In the example presented earlier, _bash_ needs to call _wait_ on _ls_.) If the parent doesn’t _wait_ on the child then the child process becomes a zombie which means it hangs around in the kernel’s process table wasting resources. If the parent itself dies then the process gets re-assigned a parent with _pid_ one - that is, the _pid_ of the unique [_init_](https://en.wikipedia.org/wiki/Init) process which automatically calls _wait_ and frees up the process.

Threads - Lesser Processes
--------------------------

What is a thread, really? On Linux, threads are more or less independent processes which happen to share the same memory and some other resources. They are created by calling [_clone_](https://man7.org/linux/man-pages/man2/clone.2.html) with the appropriate flag(s). In kernel terminology, every thread has it’s own unique _pid_ and all threads in the same process share the same _tgid_ (thread group id) which is equal to first thread’s _pid_. So from the kernel’s perspective really _pids_ identify threads, _tgids_ identify processes, and the _pid_ is equal to the _tgid_ for single-thread processes. What makes this confusing is that the terminology in usermode is different as you can see in the following table:

<table><thead><tr><th>Kernel name (<a href="https://elixir.bootlin.com/linux/latest/source/include/linux/sched.h#L629">in <em>task_struct</em></a>)</th><th>Usermode name</th><th>Returned by syscall</th></tr></thead><tbody><tr><td>tgid (thread group id)</td><td>pid (process id)</td><td><a href="https://man7.org/linux/man-pages/man2/getpid.2.html">getpid</a></td></tr><tr><td>pid (process id)</td><td>tid (thread id)</td><td><a href="https://man7.org/linux/man-pages/man2/gettid.2.html">gettid</a>, clone, fork</td></tr></tbody></table>

Shameless Plug
--------------

If you use Kubernetes, check out [Robusta.dev](https://home.robusta.dev/?from=natanyellin).
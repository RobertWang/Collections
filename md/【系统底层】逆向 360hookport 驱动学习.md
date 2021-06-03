> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5Mjc3MDM2Mw==&mid=2651135971&idx=1&sn=d7ee34199e88c7b7eba6d6953a9b525a&chksm=bd508fb78a2706a1ab6f3481b9a32805063ed4c7f94fb07d39f3d76e268f2ef8e5ead5ed171f&scene=0&xtrack=1#rd)

**作者****论****坛账号：镇北看雪**

hook 目的
=======

360hookport 驱动所实现的功能就是在不修改系统的 SSDT 表 / ShadowSSDT 表的情况下拦截一些重要的系统调用，并进行根据一定的过滤规则进行参数和返回值的检查，阻止一些不安全或危险的系统调用。

hook 思路
=======

hook 细节
=======

两个重要的数据结构
---------

```
00000000 _SERVICE_FILTER_INFO_TABLE struc ; (sizeof=0x5DDC, align=0x4, copyof_10)
00000000 SSDTCnt dd ?                                            ; Shadowssdt的最大服务数目（shadowssdt中服务要包含ssdt中的）
00000004 SavedSSDTServiceAddress dd 1001 dup(?)                  ; 原始SSDT例程地址
00000FA8 ProxySSDTServiceAddress dd 1001 dup(?)                  ; SSDT调用的代{过}{滤}理函数地址
00001F4C SavedShadowSSDTServiceAddress dd 1001 dup(?)            ; 原始shadowSSDT例程地址
00002EF0 ProxyShadowSSDTServiceAddress dd 1001 dup(?)            ; ShadowSSDT调用的代{过}{滤}理函数地址
00003E94 SwitchTableForSSDT dd 1001 dup(?)                       ; SSDT代{过}{滤}理开关（HOOK开关）
00004E38 SwitchTableForShadowSSDT dd 1001 dup(?)                 ; ShadowSSDT代{过}{滤}理函数开关
00005DDC _SERVICE_FILTER_INFO_TABLE ends

00000000 _FILTERFUN_RULE_TABLE struc ; (sizeof=0x1A8, mappedto_12)
00000000 bSize dd ?                                    ; FILTERFUN_RULE_TABLE结构的的大小
00000004 pNextRuleTable dd ?                           ; 指向下一个张FILTERFUN_RULE_TABLE表包含新的过滤规则
00000008 IsFilterFunFilledReady dd ?                   ; 过滤函数开关（表示过滤函数是否准备好）
0000000C CheckServiceRoutine dd 101 dup(?)             ; 过滤函数表，一共有101（0x65）个
000001A0 ShadowSSDTRuleTableBase dd ?                  ; ShadowSSDT过滤规则表的基地址
000001A4 SSDTRuleTableBase dd ?                        ; SSDT过滤规则表的基地址
000001A8 _FILTERFUN_RULE_TABLE ends


```

ssdt_fiter（）
------------

看一下 ssdt_fiter()函数，其内部会先判断是 ShadowSSDT 调用还是 SSDT 调用，接着判断代 {过}{滤} 理函数 hook 开关是否打开，过滤函数表是否准备好。只有都打开后才会返回代 {过}{滤} 理函数地址，并将原函数的地址保存在 SERVICE_FILTER_INFO_TABLE 的原程序例程数组中

```
ULONG __stdcall ssdt_filter(unsigned int call_index, int ori_pfn, int ServiceBase)
{
  if ( ServiceBase == g__ssdt_bast && call_index <= g_max_shadowIndex )
    goto LABEL_15;
  if ( ServiceBase == g_shadow_ssdt_base && call_index <= g_max_SSDT_index )        // 如果是ShadowSSDT调用
  {
    if ( g_myServiceBase->SwitchTableForShadowSSDT[call_index] && check_needto_filter(call_index, 1) )  //判断代{过}{滤}理函数hook开关是否打开，过滤表是否准备好
    {
      g_myServiceBase->SavedShadowSSDTServiceAddress[call_index] = ori_pfn;                             //将原始例程的地址保存到我们的SERVICE_FILTER_INFO_TABLE表中
      return g_myServiceBase->ProxyShadowSSDTServiceAddress[call_index];                                //返回代{过}{滤}理函数地址
    }                                          
    return ori_pfn;                                                                                     // 返回原例程地址
  }
  if ( ServiceBase == *(_DWORD *)addr_g_KeServiceDescriptorTable )                  //如果是SSDT调用
  {
LABEL_15:
    if ( g_myServiceBase->SwitchTableForSSDT[call_index] && check_needto_filter(call_index, 0) )         //判断理函数hook开关是否打开，过滤表是否准备好
    {
      g_myServiceBase->SavedSSDTServiceAddress[call_index] = ori_pfn;                                    //将原始例程的地址保存到我们的SERVICE_FILTER_INFO_TABLE表中                                       
      return g_myServiceBase->ProxySSDTServiceAddress[call_index];                                       // 返回代{过}{滤}理函数地址
    }
  }                                             
  return ori_pfn;                                                                                        //返回原例程地址
}

```

一般的代 {过}{滤} 理函数
---------------

![](https://mmbiz.qpic.cn/sz_mmbiz_png/LFPriaSjBUZIUuic5obQCu0TrD8GKp5fJ8ibFibovoznh7jKyfAw1x2YVHRksaLww8VL1VZiamHRVibru3foowEqVo5g/640?wx_fmt=png)

在 pre_check（）函数内部其会先判断 FILTERFUN_RULE_TABLE 的 IsFilterFunFilledReady 过滤规则表是否准备就绪，然后会根据传入的过滤函数表索引获取所有 FILTERFUN_RULE_TABLE 表中对应的过滤函数并调用，（过滤函数的实现不在 hookport 模块中，其实现在其他模块中，然后通过 hookport 提供的接口将过滤函数的地址填写到 FILTERFUN_RULE_TABLE 的过滤函数数组中）。过滤函数中会对参数进行检查，如果有问题就返回错误，没问题就返回一个函数地址 CheckReturn()。  
光检查参数不行，有时候还需要检查返回值，CheckReturn() 这个函数就是从来检测返回值的。接着会继续遍历 FILTERFUN_RULE_TABLE 的 pNextRuleTable 寻找下一张 FILTERFUN_RULE_TABLE 表继续调用过滤函数。然后返回返回值。最多有 16 张 FILTERFUN_RULE_TABLE 表。  
当返回值都没问题后，将调用各个 FILTERFUN_RULE_TABLE 表中过滤函数返回的 CheckReturn()函数的地址保存到数组中并返回到代 {过}{滤} 理函数中。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/LFPriaSjBUZIUuic5obQCu0TrD8GKp5fJ88icZZfaFS47zmvNjDxqf0TQiaaOz9m2icPpe4H1a2PtANdicaz3icbdB0zA/640?wx_fmt=png)

然后代 {过}{滤} 理函数会判断此次调用时 SSDT 调用还是 ShadowSSDT 调用，然后从 SERVICE_FILTER_INFO_TABLE 中获得原始的服务例程并调用，然后检查返回值，如果调用出错肯定就不用管了，如果调用成功就调用我们刚刚调用 pre_check 函数返回的 CheckReturn 函数数组，依次调用各个 CheckReturn 函数，此函数会在内部检查返回值然后返回，如果所有的返回值都没问题代 {过}{滤} 理函数就可以将调用原始例程的返回值返回了。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/LFPriaSjBUZIUuic5obQCu0TrD8GKp5fJ8xIbC5dgKrgiaFpJTicHe65MbfEWAV5kB0P2HMy75XgFziac3ul3Q6r0BA/640?wx_fmt=png)

具体细节
----

DriverEntry 入口处先判断操作系统的版本信息，然后查看是否处于安全模式下，如果是就返回错误代码。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/LFPriaSjBUZIUuic5obQCu0TrD8GKp5fJ8D6xBApr9XrEEJf8nCAvOKWO7jFLNSQib7OBTrwWgSW7C17BcRLQwnsA/640?wx_fmt=png)

接着调用 init_and_hook_KiFastCallEntry_KeUserModeCallback（）函数初始化，并 hook_KiFastCallEntry 和 KeUserModeCallback。  
在函数内部其会先调用 findmodule（）查看是否加载了 win32k.sys 模块，findmodule 是通过 ZwQuerySystemInformation 传入 SystemModuleInformation 枚举内核模块。注意得到的第一个模块就是 ntoskrnl.sys 的模块。获得 SYSTEM_MODULE_INFORMATION_ENTRY 结构数组，然后根据模块的名称进行判断得到模块的基地址和大小

```
NTSTATUS 
ZwQuerySystemInformation (
    SYSTEM_INFORMATION_CLASS SystemInformationClass, 
    PVOID SystemInformation, 
    ULONG SystemInformationLength, 
    ULONG *ReturnLength);

```

如果加载了此模块就调用 search_ssdtTable_byHardCode（）得到 shadowSSDT 表的基地址。search_ssdtTable_byHardCode（）函数时在函数 KeAddSystemServiceTable 根据特征码 0x888d 搜寻 KeServiceDescriptorTableShadow 地址，或者在函数 KeRemoteSystemServiceTable 中根据特征码 0x8889 得到 KeServiceDescriptorTableShadow 地址

![](https://mmbiz.qpic.cn/sz_mmbiz_png/LFPriaSjBUZIUuic5obQCu0TrD8GKp5fJ8ibXozb7fSWygMLWichamVEkpr0xcX4fDXgA3ubgouvAsLbczsWSTePGA/640?wx_fmt=png)

接着调用 GetProcAddress 得到 KeServiceDescriptorTable 地址进一步得到 SSDT 表的基地址。  
GetProcAddress 内部通过 findmodule 得到第一个内核模块的基地址，第一个内核模块也就是 ntoskrnl.exe，然后解析 ntoskrnl.exePE 头文件根据导出表得到 KeServiceDescriptorTable 的地址。  
因为 KeServiceDescriptorTable 是由 ntoskrnl.exe 导出的，而 win32k.sys 并没有导出 KeServiceDescriptorTableShadow

![](https://mmbiz.qpic.cn/sz_mmbiz_png/LFPriaSjBUZIUuic5obQCu0TrD8GKp5fJ84shdB0N215JAmoW8JBBeFmjJbtGicuRjZ9uAT7nGLd5kwBOYQXp9ntQ/640?wx_fmt=png)

接着调用 CollectAllHookFun_Index（）得到所有需要 hook 的服务例程的服务索引，对于有对应 ZW 系列导出函数的索引我们直接通过获得 ZW 系列地址然后，在函数开头位置获得索引号，而如果 hook 的函数没有对应的 ZW 系列导出函数则判断系统的版本信息，根据硬编码得到对应的服务索引  
如果索引号获取不到，或者得到了大于 1000 的索引号，统一将服务索引号设置为 1000，（后面所有服务号为 1000 的 hook 开关都会被关闭，防止错误的调用）。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/LFPriaSjBUZIUuic5obQCu0TrD8GKp5fJ82GfGvjEnoGRdmttSDpc7DfxrFtJhxkpHYzT4JQKzqpmnz9J4ERalgA/640?wx_fmt=png)

因为 ShadowSSDT 表中包含了 SSDT 表，所以得到 ShadowSSDT 表中最多有多少项就得到了所有服务的最大数目。获得服务的最大数目后申请内存存放 SERVICE_FILTER_INFO_TABLE 表并将最大的服务数存到第一个字段中 SSDTCnt

![](https://mmbiz.qpic.cn/sz_mmbiz_png/LFPriaSjBUZIUuic5obQCu0TrD8GKp5fJ81m9hhCnqnBaPMgn8HKz0zjxTlQAXAjNXR6ibWUe3ZPEYIGS0vQ2cicgQ/640?wx_fmt=png)

接着根据刚刚获得的各个需要 hook 的服务例程的索引将对应的代 {过}{滤} 理函数的地址写到 SERVICE_FILTER_INFO_TABLE 中代 {过}{滤} 理函数数组 ProxySSDTServiceAddress 对应的位置中。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/LFPriaSjBUZIUuic5obQCu0TrD8GKp5fJ8KOrhbAB9ibqC3rjxaWOXHVAoHr88v3G0sl3oCtTTqUrDXXbiampnYv2w/640?wx_fmt=png)

然后就是调用 hook_KiFastCallEntry() 了，其是通过 ssdt hook NtSetEvent 函数，然后主动调用 zwSetEvent 函数并传入特殊的句柄值 0x288C58F1，此函数就会调用我们在 ssdt hook 中安装的 MyNtSetEvent。然后在 MySetEvent 中进行 KiFastCallEntry inline hook 的安装。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/LFPriaSjBUZIUuic5obQCu0TrD8GKp5fJ8sQFMFiaLy7kjjiaFQHAxKjZJn0nDNxygBbL7ibicibsP8zPkMDINcnXKzcw/640?wx_fmt=png)

看一下 MySetEvent，其会先判断传入的句柄值是否为 0x288c58f1，如果不是那就不是我们的调用直接调用原始例程 NtSetEvent，如果是证明这是我们自己的调用就进行 KiFastCallEntry inlinehook 的安装，  
360 hookport 的 KiFastCallEntry inlinehook 位置很好，向 8053e621 地址处写入一级 jmp 跳转指令跳到 360hookport 的二级跳转指令处。然后二级跳转指令又跳转到 KiFastCallEntryFiter 入口处。一级地址跳转前 eax 为服务号，而 ebx 为从系统服务表中取出的服务例程地址。  
那么我们如何在 MySetEvent 中找到这个 inline hook 的位置呢，因为我们的 MySetEvent 是通过 call ebx 调用的，那么在栈中就一定存在 call ebx 下一条指令的地址 [ebp +4]。我们通过栈回溯找到 call ebx 下一条指令的地址然后往上进行硬编码匹配，当找到 0x02E9C1E12B 时表示我们找到了 hook 的位置  
然后将一级跳转指令写入此处。inline hook 安装完成后恢复 NtSetEvent 的 ssdt hook。返回到 hook_KiFastCallEntry() 函数中检查刚刚调用 MyNtSetEvent（）是否完成了 KiFastCallEntry 的 inline hook，如果没有下面做的和 MyNtSetEvent 做的差不多，就是进行 inline hook KiFastCallEntry。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/LFPriaSjBUZIUuic5obQCu0TrD8GKp5fJ8o2EU9hM5AiaYYDP8tITHxZgxibk311tS4vCdBRVSI6IGpVIfbwLcNVyQ/640?wx_fmt=png)

```
nt!KiFastCallEntry+0xcc:
8053e60c ff0538f6dfff    inc     dword ptr ds:[0FFDFF638h]
8053e612 8bf2            mov     esi,edx
8053e614 8b5f0c          mov     ebx,dword ptr [edi+0Ch]
8053e617 33c9            xor     ecx,ecx
8053e619 8a0c18          mov     cl,byte ptr [eax+ebx]
8053e61c 8b3f            mov     edi,dword ptr [edi]                                         ；edi为系统服务表基地址
8053e61e 8b1c87          mov     ebx,dword ptr [edi+eax*4]                                   ；eax =系统服务号，ebx为对应的系统服务地址
8053e621 2be1            sub     esp,ecx
8053e623 c1e902          shr     ecx,2
8053e626 8bfc            mov     edi,esp
8053e628 3b35d4995580    cmp     esi,dword ptr [nt!MmUserProbeAddress (805599d4)]
8053e62e 0f83a8010000    jae     nt!KiSystemCallExit2+0x9f (8053e7dc)
8053e634 f3a5            rep movs dword ptr es:[edi],dword ptr [esi]
8053e636 ffd3            call    ebx                                                         ；调用相应的系统服务
8053e638 8be5            mov     esp,ebp

```

接着调用 PsSetCreateProcessNotifyRoutine 创建了一个进程通知回调，其内不会调用 pre_check() 并传入 0x45 过滤函数索引。对应的过滤函数会检查进程创建是否存在问题。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/LFPriaSjBUZIUuic5obQCu0TrD8GKp5fJ8Zn8r5tRHx0o8mYoHz7mcVh8tpMuwibc9XqYoN3kR1U8jzqzq1OsanEg/640?wx_fmt=png)

然后会判断 win32k.sys 模块是否加载，如果没加载就将 NtSetSystemInformation 的 hook 开关先打开，如果 win32k.sys 已经加载了就直接获取 csrss.exe 进程的 PID，然后将进程空间切到 csrss.exe 所在的进程空间中，然后 IAThook win32k.sys 模块中的 KeUserModeCallback 函数。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/LFPriaSjBUZIUuic5obQCu0TrD8GKp5fJ8UAOPvz0ibfdribibuhWkWibXQJvH5gmaAyXHKoh9hVnbnvb083YhMajTtw/640?wx_fmt=png)

这里注意为什么将进程切换到 csrss.exe 所在的进程空间中再 hook_KeUserModeCallback 呢，实际没必要非得切换都 csrss.exe 进程空间中，主要不是在 System 和 smss.exe 进程空间中都可以，因为在在 System 和 smss.exe 进程空间中 win32k.sys 其虚拟地址并没有被映射物理内存，访问无效

```
NTSTATUS 
ZwSetSystemInformation (
    SYSTEM_INFORMATION_CLASS SystemInformationClass, 
    PVOID SystemInformation, 
    ULONG SystemInformationLength);

```

那么如果 win32k.sys 没有加载为什么要先打开 NtSetSystemInformation 的 hook 开关呢，我们分析一下 NtSetSystemInformation 对应的代 {过}{滤} 理函数，发现其会在调用完过滤函数后判断参数 SystemInformationClass 是否为 SystemExtendedServiceTableInformation（0x26），  
如果是说明 win32k.sys 正在加载然后对 KeAddSystemServiceTable 进行所在的 ntoskrnl.exe 的 EAT_hook。   因为 windows 系统再加载时运行的第一个用户进程就是 smss.exe 会话管理器，其会调用 NtSetSystemInformation 并传入参数 SystemExtendedServiceTableInformation（0x26）来加载 win32k.sys  
随后 win32k.sys 就会在 DriverEntry 驱动入口调用 KeAddSystemServiceTable 函数添加 ShadowSSDT 表。HOOK_KeAddSystemServiceTable 的原因是判断添加的是否为 win32k.sys 的 ShadowSSDT 表，防止攻击者加载自己的 ShadowSSDT 替换正确的 ShadowSSDT 表  
然后 NtSetSystemInformation 的代 {过}{滤} 理函数会继续判断 win32k.sys 是否已经加载，如果加载了就 hook_KeUserModeCallback 函数。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/LFPriaSjBUZIUuic5obQCu0TrD8GKp5fJ8wRzZtCQb43d4HJ35bqeHtbvB9nxu0sqO37nziczrcJfBWILd5jrEOiag/640?wx_fmt=png)

那么为什么要 hook_KeUserModeCallback 函数呢，调用 KeUserModeCallback 函数时在内核中调用用户层代码的一种手段，例如我们利用全局钩子注入 dll，或者是利用输入法入 dll，或者是设置鼠标键盘消息记录钩子 WH_JOURNALRECORD，都会调用 win32k.sys 模块的 KeUserModeCallback 函数  
我们将此函数 hook 了就可以监控这些这些操作。我们看一看 hook 后的 MyKeUserModeCallback，其通过判断 apiNumber 是否为 ClientLoadLibrary，ClientImmLoadLayout 或 fnHkOPTINLPEVENTMSG 来监控这些操作。如果没问题就正常返回。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/LFPriaSjBUZIUuic5obQCu0TrD8GKp5fJ8st5iaVfxhxxWhJfibuNljAEyhv0B1B8stShqG7R3ZId9sMp7h74AbrEA/640?wx_fmt=png)

做完这些之后在设置过滤函数索引与此服务的实际服务索引之间的对应关系数组 FilterToServiceIndex[0x65]。例如 NtCreateKey 的过滤函数索引为 0，那么 FilterToServiceIndex[0] 就等于 NtCreateKey 在服务表中的索引。  
最后最后在调用 CmRegisterCallback 函数注册注册表回调来检测 KiFastCallEntry 的 inline hook 是否安装成功。在注册表通知回调函数中如果检测到 KiFastCallEntry 的 inline hook 还没有安装就再次进行 KiFastCallEntry 的 inline hook

![](https://mmbiz.qpic.cn/sz_mmbiz_png/LFPriaSjBUZIUuic5obQCu0TrD8GKp5fJ8cws7ntfsTdbAF5I90wuGRicOJO0BagqVA2ibN5MicWmWCdsElyF6T5Oiag/640?wx_fmt=png)

驱动向外提供了 3 个扩展接口：  
1.g_port_extension_v1_for_AddRule   ：向 FILTERFUN_RULE_TABLE 规则表中添加新的规则（即添加一张新的 FILTERFUN_RULE_TABLE 表，FILTERFUN_RULE_TABLE 的 pNextRulTable 就指向下一张 FILTERFUN_RULE_TABLE 表）

2.g_port_extension_v2_register_ssdt_check_handle_callback：设置 FILTERFUN_RULE_TABLE 中过滤函数以及对应的 SERVICE_FILTER_INFO_TABLE 表中的 SSDT/ShadowSSDThook 开关（其会将所有服务索引为 1000 的服务的 hook 开关关闭）

3.g_port_extension_v3_register_ruletable_base：设置过滤规则表对应的过滤规则

![](https://mmbiz.qpic.cn/sz_mmbiz_png/LFPriaSjBUZIUuic5obQCu0TrD8GKp5fJ8AibOkzlLX3V0mvBMnnAmmGGZQFo1074OSDOLcGNS84FPC05l2oW5wyQ/640?wx_fmt=png)

参考：https://bbs.pediy.com/thread-99460.htm

****-- 官方论坛****

www.52pojie.cn

或搜微信号：pojie_52
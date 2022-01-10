> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.appinn.com](https://www.appinn.com/kmonad/)

**kmonad** 是一款开源的高级键盘管理工具，支持 Windows、macOS 和 Linux，可以让你无限地定制和扩展几乎所有键盘功能，包括改建、配置组合键等功能。@[Appinn](https://www.appinn.com/kmonad/)

![](https://img3.appinn.net/images/202112/kmonad.jpg!o)

感谢 @[z775729168](https://meta.appinn.net/u/z775729168) 同学在[讨论组](https://meta.appinn.net/c/discuss-and-share/6)的分享

Kmonad，一个自由改变键盘布局的软件
--------------------

原文链接：[https://meta.appinn.net/t/topic/28024](https://meta.appinn.net/t/topic/28024) 作者：@[z775729168](https://meta.appinn.net/u/z775729168)[](https://meta.appinn.net/c/discuss-and-share/6)

kmonad 主要解决的是键盘的布局问题。

因为键盘的一般布局大量使用小拇指，来按 `esc` 等按键，kmonad 可以缓解这个问题。

还有一些小键盘，缺少一些按键，或者你的键盘有一些按键坏了，以及让那些不具备改键等功能的键盘实现复杂的功能，都可以使用 kmonad。kmonad 也可以在 Linux，Mac 运行，但是需要稍微调整配置文件。

kmonad 主要难点是配置文件，这里简单介绍一下，并提供一个 Windows 下可以快速使用的配置。

### 基础配置

我们主要关注 deflayer default：

```
(defcfg
  ;; For Linux
  ;; input  (device-file "/dev/input/by-id/usb-04d9_daskeyboard-event-kbd")
  ;; output (uinput-sink "My KMonad output"
    ;; To understand the importance of the following line, see the section on
    ;; Compose-key sequences at the near-bottom of this file.
    ;; "/run/current-system/sw/bin/sleep 1 && /run/current-system/sw/bin/setxkbmap -option compose:ralt")
  ;; cmp-seq ralt    ;; Set the compose key to `RightAlt'
  ;; cmp-seq-delay 5 ;; 5ms delay between each compose-key sequence press

  ;; For Windows
  input  (low-level-hook)
  output (send-event-sink)

  ;; For MacOS
  ;; input  (iokit-name "my-keyboard-product-string")
  ;; output (kext)

  ;; Comment this is you want unhandled events not to be emitted
  fallthrough true

  ;; Set this to false to disable any command-execution in KMonad
  allow-cmd true
)

(deflayer default
  _    _    _    _    _    _    _    _    _    _    _    _    _
  _    _    _    _    _    _    _    _    _    _    _    _    _    _   
  _    _    _    _    _    _    _    _    _    _    _    _    _    _
  _    _    _    _    _    _    _    _    _    _    _    _    _
  _    _    _    _    _    _    _    _    _    _    _    _   
  _    _    _              _              _    _    _    _   
)

(defsrc
  esc  f1   f2   f3   f4   f5   f6   f7   f8   f9   f10  f11  f12
  grv  1    2    3    4    5    6    7    8    9    0    -    =    bspc  
  tab  q    w    e    r    t    y    u    i    o    p    [    ]    \   
  caps a    s    d    f    g    h    j    k    l    ;    '    ret
  lsft z    x    c    v    b    n    m    ,    .    /    rsft  
  lctl lmet lalt           spc            ralt rmet cmp  rctl  
)
```

注意，如果你的键盘没有 F 功能键，需要在 deflayer 和 defsrc 删除对应的按键。如果你的键盘没有数字键也要删除对应的按键。你可以去仓库找一些现成的键位：[https://github.com/kmonad/kmonad/tree/master/keymap](https://github.com/kmonad/kmonad/tree/master/keymap)

### [](https://meta.appinn.net/t/topic/28024#capslctl-2)调换 caps 和 lctl

```
(deflayer default
  _    _    _    _    _    _    _    _    _    _    _    _    _
  _    _    _    _    _    _    _    _    _    _    _    _    _    _   
  _    _    _    _    _    _    _    _    _    _    _    _    _    _
  lctl _    _    _    _    _    _    _    _    _    _    _    _
  _    _    _    _    _    _    _    _    _    _    _    _   
  caps _    _              _              _    _    _    _   
)
```

很简单，调换两个键的位置，非常直观。

第一次见到如此直观的配置文件 😂

@z775729168 还分享了[单击 lctl 输出 esc](https://meta.appinn.net/t/topic/28024#lctlesc-3)、 [加层](https://meta.appinn.net/t/topic/28024#heading-4)等例子，可以在[原文](https://meta.appinn.net/t/topic/28024)中查看。以及一些缺点：

> kmonad 也是类似 AHK 通过发送消息来实现按键的，这和原来单击按键不同，有些软件可能不能响应按键。至少我发现 listary 不能响应 ctrl 触发。​

感兴趣的同学可以去研究一下，kmonad 主页项目在 [GitHub](https://github.com/kmonad/kmonad)，以及在讨论组里和作者沟通：

*   [https://meta.appinn.net/t/topic/28024/](https://meta.appinn.net/t/topic/28024/)

原文链接：https://www.appinn.com/kmonad/[](https://meta.appinn.net/c/discuss-and-share/6)
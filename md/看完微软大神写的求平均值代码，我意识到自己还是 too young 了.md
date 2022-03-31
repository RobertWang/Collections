> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/WAMX-M-GSRJlpK7MK6FSkg)

公众号

##### 来源：量子位 公众号 QbitAI

无数人点进去时无比自信：不就是一个简单的相加后除二的小学生编程题吗？

但跟着大神的一路深挖，却逐渐目瞪狗呆……

没那么简单的求平均值
----------

先从开头提到的小学生都会的方法看起，这个简单的方法有个致命的缺陷：

如果无符号整数的长度为 32 位，那么如果两个相加的值都为最大长度的一半，那么仅在第一步相加时，就会发生**内存溢出**。

也就是 average（0x80000000U, 0x80000000U）=0。

不过解决方法也不少，大多数有经验的开发者**首先**能想到的，就是预先限制相加的数字长度，避免溢出。

具体有两种方法：

1、当知道相加的两个无符号整数中的较大值时，减去较小值再除二，以提前**减少长度**：

2、对两个无符号整数预先进行除法，同时通过**按位与**修正低位数字，保证在两个整数都为奇数时，结果仍然正确。

（顺带一提，这是一个被申请了专利的方法，2016 年过期）

这两个都是较为常见的思路，不少网友也表示，自己最快想到的就是 **2016 年专利方法**。

```
unsigned average(unsigned a, unsigned b)
{
    return (a & b) + (a ^ b) / 2;// 变体 (a ^ b) + (a & b) * 2


```

如果无符号整数是 32 位而本机寄存器大小是 64 位，或者编译器支持多字运算，就可以将相加值强制转化为长整型数据。

不过，这里有一个需要特别注意的点：

必须要保证 64 位寄存器的前 32 位都为 0，才不会影响剩余的 32 位值。

像是 x86-64 和 aarch64 这些架构会自动将 32 位值**零扩展**为 64 位值：

而 Alpha AXP、mips64 等架构则会将 32 位值**符号扩展**为 64 位值。

这种时候，就需要额外增加归零的指令，比如通过向左进位两字的删除指令 rldicl：

```
// Alpha AXP: Assume a0 = a, a1 = b, both in canonical form
    insll   a0, #0, a0      ; a0 = a0 zero-extended to 64-bit value
    insll   a1, #0, a1      ; a1 = a1 zero-extended to 64-bit value
    addq    a0, a1, v0      ; 64-bit addition: v0 = a0 + a1
    srl     v0, #1, v0      ; 64-bit shift:    v0 = v0 >> 1
    addl    zero, v0, v0    ; Force canonical form
                            ; Answer in v0

// MIPS64: Assume a0 = a, a1 = b, sign-extended
    dext    a0, a0, 0, 32   ; Zero-extend a0 to 64-bit value
    dext    a1, a1, 0, 32   ; Zero-extend a1 to 64-bit value
    daddu   v0, a0, a1      ; 64-bit addition: v0 = a0 + a1
    dsrl    v0, v0, #1      ; 64-bit shift:    v0 = v0 >> 1
    sll     v0, #0, v0      ; Sign-extend result
                            ; Answer in v0

// Power64: Assume r3 = a, r4 = b, zero-extended
    add     r3, r3, r4      ; 64-bit addition: r3 = r3 + r4
    rldicl  r3, r3, 63, 32  ; Extract bits 63 through 32 from result
                            ; (shift + zero-extend in one instruction)
                            ; result in r3


```

或者直接访问比本机寄存器更大的 SIMD 寄存器，当然，从通用寄存器跨越到 SIMD 寄存器肯定也会增加内存消耗。

如果电脑的处理器支持进位加法，那么还可以采用**第三种思路**。

这时，如果寄存器大小为 n 位，那么两个 n 位的无符号整数的和就可以理解为 n+1 位，通过 RCR（带进位循环右移）指令，就可以得到正确的平均值，且不损失溢出的位。

###### **△**带进位循环右移

```
// x86-32
    mov     eax, a
    add     eax, b          ; Add, overflow goes into carry bit
    rcr     eax, 1          ; Rotate right one place through carry

// x86-64
    mov     rax, a
    add     rax, b          ; Add, overflow goes into carry bit
    rcr     rax, 1          ; Rotate right one place through carry

// 32-bit ARM (A32)
    mov     r0, a
    adds    r0, b           ; Add, overflow goes into carry bit
    rrx     r0              ; Rotate right one place through carry

// SH-3
    clrt                    ; Clear T flag
    mov     a, r0
    addc    b, r0           ; r0 = r0 + b + T, overflow goes into T bit
    rotcr   r0              ; Rotate right one place through carry


```

那如果处理器不支持带进位循环右移操作呢？

```
unsigned average(unsigned a, unsigned b)
{
#if defined(_MSC_VER)
    unsigned sum;
    auto carry = _addcarry_u32(0, a, b, &sum);
    sum = (sum & ~1) | carry;
    return _rotr(sum, 1);
#elif defined(__clang__)
    unsigned carry;
    sum = (sum & ~1) | carry;
    auto sum = __builtin_addc(a, b, 0, &carry);
    return __builtin_rotateright32(sum, 1);
#else
#error Unsupported compiler.
#endif
}


```

```
// _MSC_VER
    mov     ecx, a
    add     ecx, b          ; Add, overflow goes into carry bit
    setc    al              ; al = 1 if carry set
    and     ecx, -2         ; Clear bottom bit
    movzx   ecx, al         ; Zero-extend byte to 32-bit value
    or      eax, ecx        ; Combine
    ror     ear, 1          ; Rotate right one position
                            ; Result in eax

// __clang__
    mov     ecx, a
    add     ecx, b          ; Add, overflow goes into carry bit
    setc    al              ; al = 1 if carry set
    shld    eax, ecx, 31    ; Shift left 64-bit value

// __clang__ with ARM-Thumb2
    movs    r2, #0          ; Prepare to receive carry
    adds    r0, r0, r1      ; Calculate sum with flags
    adcs    r2, r2          ; r2 holds carry
    lsrs    r0, r0, #1      ; Shift sum right one position
    lsls    r1, r2, #31     ; Move carry to bit 31
    adds    r0, r1, r0      ; Combine


```

微软大神的思考们
--------

```
unsigned avg(unsigned a, unsigned b)
{
    return (a & b) + (a ^ b) / 2;
}

// lw      $3,8($fp)  # 5
// lw      $2,12($fp) # 5
// and     $3,$3,$2   # 4
// lw      $4,8($fp)  # 5
// lw      $2,12($fp) # 5
// xor     $2,$4,$2   # 4
// srl     $2,$2,1    # 4
// addu    $2,$3,$2   # 4


```

还有人在评论区推荐了 TopSpeed 编译器，能够通过指定合适的代码字节和调用约定来定义一个内联函数，以解决 “乘除结果是 16 位，中间计算值却不是” 的情况。

只能说，学无止境啊。

原文：  
https://devblogs.microsoft.com/oldnewthing/20220207-00/?p=106223

参考链接：  
https://news.ycombinator.com/item?id=30252263
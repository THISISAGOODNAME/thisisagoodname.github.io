---
layout: post
title: "C/C++调试技巧-debugbreak"
description: "杂记"
image: ""
date: 2018-09-10 17:33:41
categories: 研究
tags: [研究]
---
<!-- more -->
* Table of Contents
{:toc}

# MSVC独有神器 DebugBreak ( __debugbreak )

&nbsp; &nbsp; &nbsp; &nbsp;要说调试时最常用的手段，那应该就是打断点调试了。但是如果每次错误都需要手工去定位断点的位置，未免还是有点麻烦。用Unreal engine 4开发的同学应该都有经历，就是崩溃的时候总能触发一次断点，给个机会查看崩溃时的程序调用栈。这个就是依靠MSVC提供的 `DebugBreak` 函数实现的。

&nbsp; &nbsp; &nbsp; &nbsp; `DebugBreak` 函数相当于一个断点，在可能发生崩溃的地方都加一个，一旦崩溃自动断住，查看栈和数值，对于分析bug非常有帮助。

# 其他平台的 DebugBreak ( __debugbreak ) 的

&nbsp; &nbsp; &nbsp; &nbsp;理想的 `debug_break` 函数的功能

- 在执行此函数时触发一个软件断点(e.g. Linux系统上的 **SIGTRAP** 信号)
- 在触发断点后，可以继续执行，比如GDB的 **continue**, **next**, **step**, **stepi** 命令
- `debug_break` 函数不应该导致代码优化

## GCC

&nbsp; &nbsp; &nbsp; &nbsp;GCC提供了内置的 `__builtin_trap()` 函数。可以在debug模式下自动进入断点，但是，GCC编译器默认情况会把 `__builtin_trap()` 后面的代码优化掉。比如在i386上

```c
#include <stdio.h>

int main()
{
	__builtin_trap();
	printf("hello world\n");
	return 0;
}
```

会被GCC编译成

```
main
0x0000000000400390 <+0>:     0f 0b	ud2    
```

&nbsp; &nbsp; &nbsp; &nbsp;printf被优化掉了。

&nbsp; &nbsp; &nbsp; &nbsp;并且在 i386 / x86-64 体系结构上 `__builtin_trap()` 编译成了汇编指令 `ud2`, 在Linux上发出 **SIGILL** 信号而不是 **SIGTRAP** 信号。如果希望GDB在此断住，还需要告诉GDB，在接收SIGILL信号之后进入断点停止执行。

```
(gdb) handle SIGILL stop nopass
```

&nbsp; &nbsp; &nbsp; &nbsp;除此之外，GCC/GDB还是有某些版本不认 `__builtin_trap()` 。

&nbsp; &nbsp; &nbsp; &nbsp;在ARM体系结构上，GCC的 `__builtin_trap()` 会被直接翻译成 `abort()`,比x86上的 `ud2` 更加不可靠。

&nbsp; &nbsp; &nbsp; &nbsp;幸运的是GCC已经认识到这个需求，[在GCC 7.3.1 预计会添加](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=84595) `__builtin_break()`

## Clang

&nbsp; &nbsp; &nbsp; &nbsp;Clang/LLVM除了提供了内置的 `__builtin_trap` 函数之外，还额外提供了 `__builtin_debugtrap` 函数，这个函数在x86体系结构上会生成 `int3` 。

```c
#include <stdio.h>
   
int main()
{
	__builtin_debugtrap();
	printf("hello world\n");
	return 0;
}
```
会被clang编译成

```
main
0x00000000004003d0 <+0>:     50	push   %rax
0x00000000004003d1 <+1>:     cc	int3   
0x00000000004003d2 <+2>:     bf a0 05 40 00	mov    $0x4005a0,%edi
0x00000000004003d7 <+7>:     e8 d4 ff ff ff	callq  0x4003b0 <puts@plt>
0x00000000004003dc <+12>:    31 c0	xor    %eax,%eax
0x00000000004003de <+14>:    5a	pop    %rdx
0x00000000004003df <+15>:    c3	retq   
```

&nbsp; &nbsp; &nbsp; &nbsp; `int3`在Linux(x86)上可以发出一个 `SIGTRAP` 信号，用GDB/LLDB可以正常调试。

## Linux SIGTRAP 的触发方式

&nbsp; &nbsp; &nbsp; &nbsp;在前文已经提过，在i386 / x86-64体系结构上，`int3`指令可以发出一个`SIGTRAP`信号。

&nbsp; &nbsp; &nbsp; &nbsp;在ARM体系结构上，也有等价的指令。在32位ARM上，对于ARM mode，`.inst 0xe7f001f0`汇编可以发出`SIGTRAP`信号，对于Thumb mode，`.inst 0xde01`可以发出 `SIGTRAP`信号。

&nbsp; &nbsp; &nbsp; &nbsp;不过GDB在32位ARM上，接收汇编直接发出的`SIGTRAP`信号可能出现不能继续调试的情况，幸好还有个workaround 

```
(gdb) set $l = 2
(gdb) tbreak *($pc + $l)
(gdb) jump   *($pc + $l)
(gdb) # Change $l from 2 to 4 for ARM mode
```

&nbsp; &nbsp; &nbsp; &nbsp;在64位ARM上，`.inst 0xd4200000.`汇编可以产生`SIGTRAP`信号。

## debugbreak库

&nbsp; &nbsp; &nbsp; &nbsp;上文基本简述了x86和ARM实现断点的方式，有个人封装了个库，来简化开发，这个库就是 [debugbreak](https://github.com/scottt/debugbreak)

&nbsp; &nbsp; &nbsp; &nbsp;用法

```c
#include <stdio.h>
#include "debugbreak.h"
   
int main()
{
	debug_break();
	printf("hello world\n");
	return 0;
}
```

&nbsp; &nbsp; &nbsp; &nbsp;debugbreak库在各个平台上的实际情况。

Architecture | debug_break()
-------------|---------------
x86/x86-64 | `int3`
ARM mode, 32-bit | `.inst 0xe7f001f0`
Thumb mode, 32-bit | `.inst 0xde01`
AArch64, ARMv8 | `.inst 0xd4200000`
POWER | `.4byte 0x7d821008`
MSVC compiler | `__debugbreak`
Apple compiler on AArch64 | `__builtin_trap()`
Otherwise | `raise(SIGTRAP)`

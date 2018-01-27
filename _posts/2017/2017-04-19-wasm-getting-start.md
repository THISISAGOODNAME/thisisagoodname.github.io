---
layout: post
title: "WebAssembly已经可以用了"
description: "WebAssembly已经可以用了"
category: WebAssembly
tags: [WebAssembly,html5,c++,emscripten]
---

&#160; &#160; &#160; &#160;在上一个[asm.js版的光线追踪渲染器](http://aicdg.com/miniRayTracing/smallpt-asmjs/smallpt.html?spp=100)开发完成不到一周的时间，我就惊讶的发现，在3月份，四大主流浏览器(chrome,firefox,webkit,edge)都完成了对webassembly的支持。我也赶时髦，完成了[wasm版的光线追踪渲染器](http://aicdg.com/miniRayTracing/smallpt-wasm/smallpt.html?spp=100)。[项目代码](https://github.com/THISISAGOODNAME/miniRayTracing)。

<!-- more -->

* Table of Contents
{:toc}

# 开发环境搭建

&#160; &#160; &#160; &#160;最简单的开发方式就是继续使用emscripten，关于emscripten用法我写过[很多文章](http://aicdg.com/categories/#emscripten)，不赘述了。现在直接讲一下给emscripten添加wasm支持的方法。

&#160; &#160; &#160; &#160;特别说明一下，如果想要在Windows下编译支持wasm的emscripten，需要[Visual Studio 2015 Community with Update 3](https://www.visualstudio.com/downloads/)或者更新版本的VC++编译器(minGW不行)。还需要安装[cmake](https://cmake.org/download/)。还有git这个相信已经是所有开发者已经必备的东西。

```bash
$ git clone https://github.com/juj/emsdk.git
$ cd emsdk
$ ./emsdk install sdk-incoming-64bit binaryen-master-64bit
$ ./emsdk activate sdk-incoming-64bit binaryen-master-64bit
```

&#160; &#160; &#160; &#160;启动前记得激活环境变量

```bash
$ source ./emsdk_env.sh
```

# 使用

&#160; &#160; &#160; &#160;支持wasm的emscripten本质还是emscripten，所以过去emscripten所有用法都继续可以使用。唯一不同的地方，就是如果想要生成代码使用wasm而非asm.js，需要在`emcc`的命令行参数中添加`-s WASM=1`。

&#160; &#160; &#160; &#160;最简单实例：

```bash
$ mkdir hello
$ cd hello
$ echo '#include <stdio.h>' > hello.c
$ echo 'int main(int argc, char ** argv) {' >> hello.c
$ echo 'printf("Hello, world!\n");' >> hello.c
$ echo '}' >> hello.c
$ emcc hello.c -s WASM=1 -o hello.html
```

## 测试

&#160; &#160; &#160; &#160;浏览器不能通过file://的方式直接访问wasm文件，要测试，也需要架server。emsdk已经内置了`emrun`命令用来启动server。

```bash
$ emrun --no_browser --port 8080 .
```

# 其他工具

&#160; &#160; &#160; &#160;webassembly官方还有两个重要的工具，分别是

- [WABT](https://github.com/WebAssembly/wabt) - The WebAssembly Binary Toolkit
- [Binaryen](https://github.com/WebAssembly/binaryen) - Compiler and toolchain infrastructure

## WABT: The WebAssembly Binary Toolkit

&#160; &#160; &#160; &#160;WABT提供了三个重要命令：

- wasm2wast - 将wasm文件转换成wast文件
- wast2wasm - 将wast文件编译成wasm文件
- wasm-interp - 基于命令行的wasm解释器

> wasm是WebAssembly的二进制格式，wast是WebAssembly的纯文本格式，开发者编写纯文本格式，在开发环境则可以使用二进制格式，有效保护代码

## Binaryen

&#160; &#160; &#160; &#160;Binaryen提供了三个命令把三种不同文件编译成wasm：

- asm2wasm - 把asm.js文件编译成wasm
- s2wasm - 把LLVM WebAssembly's 后端的.s文件编译成wasm
- mir2wasm - 把Rust MIR文件编译成wasm
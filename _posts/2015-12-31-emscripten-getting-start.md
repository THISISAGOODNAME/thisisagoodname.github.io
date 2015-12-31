---
layout: post
title: "emscripten getting start"
description: "emscripten getting start"
category: html5
tags: [html5,c++]
---
* Table of Contents
{:toc}

![emscripten logo](http://kripken.github.io/emscripten-site/_static/Emscripten_logo_full.png)

&#160; &#160; &#160; &#160;Emscripten是一个基于LLVM的项目，可以把C和C++代码编译成高度优化的JavaScript代码(asm.js格式)，可以在web中以近乎原生的速度运行C和C++代码，而且不需要插件。

<!-- more -->

<div style="width:32%; display:inline-block;">
 <div  style="display:inline-block; vertical-align:text-top; margin-left:5px;margin-right:5px;">
  <div  style="font-size:2em; font-style:bold; margin-bottom:10px;">Porting</div>
  <div class="signpost-body" style=""><p>把现有的C/C++直接编译，并且可以在所有现代浏览器中运行</p></div>
 </div>
</div>
<div style="width:32%; display:inline-block; font-style:bold;">
 <div  style="display:inline-block; vertical-align:text-top; margin-left:5px;margin-right:5px;">
  <div  style="font-size:2em; font-style:bold; margin-bottom:10px;">APIs</div>
  <div class="signpost-body" style=""><p>Emscripten把OpenGL转译成WebGL, 并且可以使用你熟悉的API开发，比如SDL, 或者直接使用html5</p></div>
 </div>
</div>
<div style="width:32%; display:inline-block; font-style:bold;">
 <div  style="display:inline-block; vertical-align:text-top; margin-left:5px; margin-right:5px;">
  <div  style="font-size:2em; font-style:bold; margin-bottom:10px;">Fast</div>
  <div class="signpost-body" style=""><p>得益于 LLVM, Emscripten 和 <a href="http://asmjs.org">asm.js</a>,代码可以用接近原生代码的速度运行</p></div>
 </div>
</div>

# 1.About Emscripten

&#160; &#160; &#160; &#160;Emscripten是一个[开源](http://kripken.github.io/emscripten-site/docs/introducing_emscripten/emscripten_license.html#emscripten-license) LLVM to JavaScript 编译器. 有了Emscripten你可以:

- 把 C 和 C++ 代码编译成 JavaScript
- 把可以编译成 LLVM 字节码的其他代码编译成 JavaScript.
- 把其他语言的 C/C++ runtimes 编译成 JavaScript, 并在web中以间接方法运行这种语言 (已经被 Python 和 Lua 使用)!

# 2.Emscripten Toolchain

&#160; &#160; &#160; &#160;Emscripten工具链的流程图如下。核心是[Emscripten编译器前端(emcc)](http://kripken.github.io/emscripten-site/docs/tools_reference/emcc.html#emccdoc),它是标准编译器(比如gcc)的替代实现。

![Emscripten Toolchain](http://kripken.github.io/emscripten-site/_images/EmscriptenToolchain.png)

&#160; &#160; &#160; &#160;*Emcc* 使用 [*Clang*](Clang) 把 C/C++ 文件编译为 LLVM 字节码, 使用 Fastcomp (Emscripten 的编译器核心 — 一个 LLVM 后端) 来把字节码编译成 JavaScript. 输出的 JavaScript 可以被 [*node.js*](http://kripken.github.io/emscripten-site/docs/site/glossary.html#term-node-js) 执行, 或者嵌入 HTML 在浏览器中运行

# 3.Download and install

&#160; &#160; &#160; &#160;windows可以选择安装[完整版的Emscripten SDK](https://s3.amazonaws.com/mozilla-games/emscripten/releases/emsdk-1.35.0-full-64bit.exe)或者[绿色版for Win](https://s3.amazonaws.com/mozilla-games/emscripten/releases/emsdk-1.35.0-portable-64bit.zip)，linux和mac用户只能下载[绿色版for Unix like](https://s3.amazonaws.com/mozilla-games/emscripten/releases/emsdk-portable.tar.gz)

&#160; &#160; &#160; &#160;我是mac用户，讲解一下绿色版的安装方法

1. 下载并解压emsdk_protable并解压到某个文件夹下
2. 打开终端并进入到emsdk文件夹下，然后执行下列命令

{% highlight bash %}
# 获取最新可用工具链的信息
./emsdk update

# 下载并安装最新的工具链
./emsdk install latest

# 让 "最新的" SDK "激活"
./emsdk activate latest
{% endhighlight %}

3.**Mac和linux需要**，吧emsdk的路径添加到PATH
{% highlight bash %}
# 在Linux/Mac OS X上添加emsdk的路径到PATH
source ./emsdk_env.sh
{% endhighlight %}

> 这步windows用户不需要，windows在执行`activate`命令时已经完成PATH的修改
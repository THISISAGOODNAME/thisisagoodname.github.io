---
layout: post
title: "emscripten getting start"
description: "emscripten getting start"
category: emscripten
tags: [html5,c++,emscripten]
---
* Table of Contents
{:toc}

&#160; &#160; &#160; &#160;Emscripten是一个基于LLVM的项目，可以把C和C++代码编译成高度优化的JavaScript代码(asm.js格式)，可以在web中以近乎原生的速度运行C和C++代码，而且不需要插件。
![emscripten logo](http://kripken.github.io/emscripten-site/_static/Emscripten_logo_full.png)
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


# 4.使用Emscripten

&#160; &#160; &#160; &#160;在终端下输入`emcc -v`，看到当前的Emscripten版本和llvm版本

## 4.1 hello.c

&#160; &#160; &#160; &#160;新建一个hello.c文件，在文件内添加如下内容

{% highlight c %}
#include<stdio.h>

int main() {
  printf("hello, world!\n");
  return 0;
}
{% endhighlight %}

&#160; &#160; &#160; &#160;然后运行
{% highlight bash %}
emcc hello.c
{% endhighlight %}

&#160; &#160; &#160; &#160;当前文件夹下应该生成了**a.out.js**,可以使用[*node.js*](node.js)运行

{% highlight bash %}
node a.out.js
{% endhighlight %}

&#160; &#160; &#160; &#160;正常的话命令行中显示“hello, world!”

&#160; &#160; &#160; &#160;然后运行
{% highlight bash %}
emcc hello.c -o hello_world.html
{% endhighlight %}

&#160; &#160; &#160; &#160;当前文件夹下生成**hello_world.html**和**hello_world.js**两个文件，在浏览器中打开**hello_world.html**，可以看到[类似这样](http://aicdg.com/emscriptenDemos/hello/hello-html.html)的效果

## 4.2 hello_world_sdl.cpp

&#160; &#160; &#160; &#160;hello_world_sdl.cpp文件，在文件内添加如下内容

{% highlight cpp %}
#include <stdio.h>
#include <SDL/SDL.h>

#ifdef __EMSCRIPTEN__
#include <emscripten.h>
#endif

extern "C" int main(int argc, char** argv) {
  printf("hello, world!\n");

  SDL_Init(SDL_INIT_VIDEO);
  SDL_Surface *screen = SDL_SetVideoMode(256, 256, 32, SDL_SWSURFACE);

#ifdef TEST_SDL_LOCK_OPTS
  EM_ASM("SDL.defaults.copyOnLock = false; SDL.defaults.discardOnLock = true; SDL.defaults.opaqueFrontBuffer = false;");
#endif

  if (SDL_MUSTLOCK(screen)) SDL_LockSurface(screen);
  for (int i = 0; i < 256; i++) {
    for (int j = 0; j < 256; j++) {
#ifdef TEST_SDL_LOCK_OPTS
      // Alpha behaves like in the browser, so write proper opaque pixels.
      int alpha = 255;
#else
      // To emulate native behavior with blitting to screen, alpha component is ignored. Test that it is so by outputting
      // data (and testing that it does get discarded)
      int alpha = (i+j) % 255;
#endif
      *((Uint32*)screen->pixels + i * 256 + j) = SDL_MapRGBA(screen->format, i, j, 255-i, alpha);
    }
  }
  if (SDL_MUSTLOCK(screen)) SDL_UnlockSurface(screen);
  SDL_Flip(screen); 

  printf("you should see a smoothly-colored square - no sharp lines but the square borders!\n");
  printf("and here is some text that should be HTML-friendly: amp: |&| double-quote: |\"| quote: |'| less-than, greater-than, html-like tags: |<cheez></cheez>|\nanother line.\n");

  SDL_Quit();

  return 0;
}
{% endhighlight %}

&#160; &#160; &#160; &#160;然后运行
{% highlight bash %}
emcc hello_world_sdl.cpp -o hello_world_sdl.html
{% endhighlight %}

&#160; &#160; &#160; &#160;浏览器中打开**hello\_world\_sdl.html**可以看到[类似这样](http://aicdg.com/emscriptenDemos/hello_world_sdl/hello_world_sdl.html)的页面

## 4.3 hello\_world\_gles.cpp

&#160; &#160; &#160; &#160;新建一个hello_world_gles.cpp，添加[这里](https://github.com/THISISAGOODNAME/emscriptenDemos/blob/gh-pages/hello_world_gles/hello_world_gles.c)的代码

&#160; &#160; &#160; &#160;编译，生成[类似这个](http://aicdg.com/emscriptenDemos/hello_world_gles/hello_world_gles.html)网页

## 4.4 glfw

&#160; &#160; &#160; &#160;新建一个glfw.c，并添加[这里](https://github.com/THISISAGOODNAME/emscriptenDemos/blob/gh-pages/glfw/glfw.c)的代码

&#160; &#160; &#160; &#160;浏览器中打开**glfw.html**可以看到[类似这样](http://aicdg.com/emscriptenDemos/glfw/glfw.html)的页面，移动鼠标或者敲键盘，输入都能被捕捉并在下方的虚拟终端中显示

## 4.5 lua虚拟机

&#160; &#160; &#160; &#160;下载lua源代码，用[这的](https://github.com/THISISAGOODNAME/emscriptenDemos/tree/gh-pages/lua/lua)也可以

&#160; &#160; &#160; &#160;将lua源码所在文件夹的名称改为lua，在这里再新建一个文件夹dist，dist和lua文件夹在同一层。进入lua文件夹，执行下面的命令

{% highlight bash %}
make emscripten
{% endhighlight %}

&#160; &#160; &#160; &#160;在dist文件夹中会生成**lua.vm.js**

### 4.5.1 在node.js中使用lua.vm.js

&#160; &#160; &#160; &#160;执行

{% highlight bash %}
$ npm install lua.vm.js
{% endhighlight %}

&#160; &#160; &#160; &#160;然后编写node.js代码

{% highlight javascript %}
var LuaVM = require('lua.vm.js');

var l = new LuaVM.Lua.State();
l.execute('print("Hello, world")');
{% endhighlight %}

### 4.5.2 在网页中使用lua.vm.js

&#160; &#160; &#160; &#160;可以参考[这个](http://aicdg.com/emscriptenDemos/lua/REPL/repl.html)示例
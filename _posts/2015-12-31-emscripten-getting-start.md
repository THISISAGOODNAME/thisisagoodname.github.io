---
layout: post
title: "emscripten getting start"
description: "emscripten getting start"
category: html5
tags: [html5,c++]
---
* Table of Contents
{:toc}


&#160; &#160; &#160; &#160;Emscripten是一个基于LLVM的项目，可以把C和C++代码编译成高度优化的JavaScript代码(asm.js格式)，可以在web中以近乎原生的速度运行C和C++代码，而且不需要插件。

<!-- more -->


![emscripten logo](http://kripken.github.io/emscripten-site/_static/Emscripten_logo_full.png)
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
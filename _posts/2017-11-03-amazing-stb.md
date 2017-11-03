---
layout: post
title: "神库stb"
description: "神库stb"
category: c++
tags: [c++]
---

&#160; &#160; &#160; &#160;学过一段时间openGL的porn友可能对stb_image.h印象深刻，一个header only的单文件，可以解析JPG, PNG, TGA, BMP, PSD, GIF, HDR, PIC格式的图片。而有些有openAL开发经验的porn友，可能也使用过stb\_vorbis.c来代替libogg来解析ogg格式的音频。但是最近我又点到了[stb的项目主页](https://github.com/nothings/stb)，才发现这套库老牛逼了，还是public domain的，所以现在稍微安利一发。

<!-- more -->

* Table of Contents
{:toc}

&#160; &#160; &#160; &#160;库名来自原作者的全名，Sean T. Barrett，简写为STB。是一个纯C的库，完全由C89编写完成，这年头不支持C89的编译器说实话还真的挺难找到的。不过如果编译的时候加上编译条件`-msse2`，可以开启stb\_image等支持SIMD的库的SIMD支持。

# STB库中牛逼但鲜为人知的库

## stb\_truetype.h

&#160; &#160; &#160; &#160;一个header only的字体解析工具，有了他，你就不再需要臃肿的freetype2了。

## stb\_sprintf.h

&#160; &#160; &#160; &#160;这个顾名思义，是sprintf函数的重复造轮子。乍看没鸟用，不过作者在注释里给了个benchmark，大家感受一下：

```bash
PERFORMANCE vs MSVC 2008 32-/64-bit (GCC is even slower than MSVC):
===================================================================
"%d" across all 32-bit ints (4.8x/4.0x faster than 32-/64-bit MSVC)
"%24d" across all 32-bit ints (4.5x/4.2x faster)
"%x" across all 32-bit ints (4.5x/3.8x faster)
"%08x" across all 32-bit ints (4.3x/3.8x faster)
"%f" across e-10 to e+10 floats (7.3x/6.0x faster)
"%e" across e-10 to e+10 floats (8.1x/6.0x faster)
"%g" across e-10 to e+10 floats (10.0x/7.1x faster)
"%f" for values near e-300 (7.9x/6.5x faster)
"%f" for values near e+300 (10.0x/9.1x faster)
"%e" for values near e-300 (10.1x/7.0x faster)
"%e" for values near e+300 (9.2x/6.0x faster)
"%.320f" for values near e-300 (12.6x/11.2x faster)
"%a" for random values (8.6x/4.3x faster)
"%I64d" for 64-bits with 32-bit values (4.8x/3.4x faster)
"%I64d" for 64-bits > 32-bit values (4.9x/5.5x faster)
"%s%s%s" for 64 char strings (7.1x/7.3x faster)
"...512 char string..." ( 35.0x/32.5x faster!)
```

## stb\_voxel\_render.h

&#160; &#160; &#160; &#160;顾名思义，一个voxel风格的渲染器，简述一下voxel风格吧，就是类似Minecraft那种方格近似的风格。[油管演示视频](https://www.youtube.com/watch?v=2vnTtiLrV1w)

&#160; &#160; &#160; &#160;然后大神就[顺手写了个Minecraft地图查看器](https://github.com/nothings/stb/tree/master/tests/caveview)，可以打开Minecraft Anvil files (.mca)格式的地图。

## stb\_tilemap\_editor.h

&#160; &#160; &#160; &#160;这个顾名思义，一个瓦片地图生成工具(STB大佬你是有多屌)。不过这个库终于不是0依赖的了。stb\_tilemap\_editor.h还有stb\_textedit.h，两个库要依赖[imgui](https://github.com/ocornut/imgui)，算是stb里极少数有外部依赖的库了。

## stb\_c\_lexer.h

&#160; &#160; &#160; &#160;可能有人会觉得大佬连编译都做。不过这个库和老一辈游戏开发者开发方式有点关系。老一辈游戏开发者喜欢定义一些自己特有的格式，所以解析也就很熟。

# STB库详细内容一览

library |	lastest version |	category |	LoC |	description
-------|----------|-------|-----|---------
[stb\_vorbis.c](https://github.com/nothings/stb/blob/master/stb_vorbis.c) |	1.11 |	audio |	5449 |	decode ogg vorbis files from file/memory to float/16-bit signed output
[stb\_image.h](https://github.com/nothings/stb/blob/master/stb_image.h) |	2.16 |	graphics |	7187 |	image loading/decoding from file/memory: JPG, PNG, TGA, BMP, PSD, GIF, HDR, PIC
[stb\_truetype.h](https://github.com/nothings/stb/blob/master/stb_truetype.h) |	1.17 |	graphics |	4566 |	parse, decode, and rasterize characters from truetype fonts
[stb\_image\_write.h](https://github.com/nothings/stb/blob/master/stb_image_write.h) |	1.07 |	graphics|	1458|	image writing to disk: PNG, TGA, BMP
[stb\_image\_resize.h](https://github.com/nothings/stb/blob/master/stb_image_resize.h)	|0.95	|graphics	|2627|	resize images larger/smaller with good quality
[stb\_rect\_pack.h](https://github.com/nothings/stb/blob/master/stb_rect_pack.h)|	0.11	|graphics	|624	|simple 2D rectangle packer with decent quality
[stb\_sprintf.h](https://github.com/nothings/stb/blob/master/stb_sprintf.h)	|1.03	|utility	|1812	|fast sprintf, snprintf for C/C++
[stretchy\_buffer.h](https://github.com/nothings/stb/blob/master/stretchy_buffer.h)|	1.02	|utility	|257|	typesafe dynamic array for C (i.e. approximation to vector<>), doesn't compile as C++
[stb\_textedit.h](https://github.com/nothings/stb/blob/master/stb_textedit.h)	|1.11	|user interface|	1393|	guts of a text editor for games etc implementing them from scratch
[stb\_voxel\_render.h](https://github.com/nothings/stb/blob/master/stb_voxel_render.h)|	0.85	|3D graphics|	3803|	Minecraft-esque voxel rendering "engine" with many more features
[stb\_dxt.h](https://github.com/nothings/stb/blob/master/stb_dxt.h)|	1.07|	3D graphics|	719	|Fabian "ryg" Giesen's real-time DXT compressor
[stb\_perlin.h](https://github.com/nothings/stb/blob/master/stb_perlin.h)	|0.3|	3D graphics	|316|	revised Perlin noise (3D input, 1D output)
[stb\_easy\_font.h](https://github.com/nothings/stb/blob/master/stb_easy_font.h)	|1.0	|3D graphics|	303	|quick-and-dirty easy-to-deploy bitmap font for printing frame rate, etc
[stb\_tilemap\_editor.h](https://github.com/nothings/stb/blob/master/stb_tilemap_editor.h)	|0.38	|game dev	|4172|	embeddable tilemap editor
[stb\_herringbone\_wang\_tile.h](https://github.com/nothings/stb/blob/master/stb_herringbone_wang_tile.h)	|0.6	|game dev|	1220|	herringbone Wang tile map generator
[stb\_c\_lexer.h](https://github.com/nothings/stb/blob/master/stb_c_lexer.h)	|0.09	|parsing	|962	|simplify writing parsers for C-like languages
[stb\_divide.h](https://github.com/nothings/stb/blob/master/stb_divide.h)|	0.91	|math	|419|	more useful 32-bit modulus e.g. "euclidean divide"
[stb\_connected\_components.h](https://github.com/nothings/stb/blob/master/stb_connected_components.h)	|0.95	|misc	|1045|	incrementally compute reachability on grids
[stb.h](https://github.com/nothings/stb/blob/master/stb.h)|	2.30	|misc	|14328|	helper functions for C, mostly redundant in C++; basically author's personal stuff
[stb\_leakcheck.h](https://github.com/nothings/stb/blob/master/stb_leakcheck.h)|	0.4	|misc|	186|	quick-and-dirty malloc/free leak-checking

> Total libraries: 20 <br>
> Total lines of C code: 52846

# 这个库的一些有趣的地方

## 1. 单文件且header only

&#160; &#160; &#160; &#160;这个估计用windows开发的porn友应该都神游体会，windows编写程序要解决依赖就很麻烦了，如果要带着依赖一起发布，就更痛苦了。

## 2. 授权协议

&#160; &#160; &#160; &#160;授权协议用非常非常开放的public domain协议，除了WTFPL之外，其实这个就算是最开放的协议了。不过STB的样例和测试就是BSD和MIT混用了，千万注意。

## 3. 纯C，而且是C89

&#160; &#160; &#160; &#160;这个STB大神在源码中解释了，他是个C开发者，不用C++，而且依旧在使用VC6.0作为开发工具(具有比之后任何版本的VS都更人性化的体验)。stb中的两个库，stretchy\_buffer.h以及stb.h，前者提供了动态数组，后者提供了众多C++中的常用函数。
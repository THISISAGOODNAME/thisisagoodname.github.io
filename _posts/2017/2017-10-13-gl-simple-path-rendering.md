---
layout: post
title: "简化OpenGL path rendering的神器"
description: "简化OpenGL path rendering的神器"
category: c++
tags: [c++,图形学,OpenGL]
---

&#160; &#160; &#160; &#160;对于经常做一些2D图形开发的朋友，对于OpenGL的粗线条、文本的绘制想必深恶痛绝不堪回首。那么问题来了，这些操作有简化的方法吗。最近看Nintendo Switch，老黄很贴心的为大家准备了一个超牛逼扩展，只是可能很多人不知道，我现在必须向大家隆重推荐一下。

<!-- more -->

* Table of Contents
{:toc}

# GL_NV_path_rendering

&#160; &#160; &#160; &#160;这个扩展的名字叫GL\_NV\_path\_rendering，顾名思义，就是用来简化各种path rendering操作的。OpenGL/OpenGL ES均支持，设备兼容性的话基本上11年以后的N卡都有完整的支持(当然也包括PS3和使用Tegra的设备比如NVIDIA shield和Nintendo Switc)。这个扩展可以追溯到OpenGL 1.X时代，历史悠久，固定管线就有了。但是即便同为N卡支持能力上有差异，主要集中在是否支持可编程管线，已经GLSL版本需求。

## 渲染SVG

```c++
GLuint pathObj = 42;
const char *svgPathString =
	// star
	"M100,180 L40,10 L190,120 L10,120 L160,10 z"
	// heart
	"M300 300 C 100 400,100 200,300 100,500 200,500 400,300 300Z";
glPathStringNV(pathObj, GL_PATH_FORMAT_SVG_NV,
	(GLsizei)strlen(svgPathString), svgPathString);
```

## 渲染PostScript

```c++
const char *psPathString =
	// star
	"100 180 moveto"
	" 40 10 lineto 190 120 lineto 10 120 lineto 160 10 lineto closepath"
	// heart
	" 300 300 moveto"
	" 100 400 100 200 300 100 curveto"
	" 500 200 500 400 300 300 curveto closepath";
glPathStringNV(pathObj, GL_PATH_FORMAT_PS_NV,
	(GLsizei)strlen(psPathString), psPathString);
```

## 渲染文本

```c++
GLuint glyphBase = glGenPathsNV(6);
const unsigned char *word = "OpenGL";
const GLsizei wordLen = (GLsizei)strlen(word);
const GLfloat emScale = 2048;  // match TrueType convention
GLuint templatePathObject = ~0;  // Non-existent path object
glPathGlyphsNV(glyphBase,
	GL_SYSTEM_FONT_NAME_NV, "Helvetica", GL_BOLD_BIT_NV,
	wordLen, GL_UNSIGNED_BYTE, word,
	GL_SKIP_MISSING_GLYPH_NV, ~0, emScale);
glPathGlyphsNV(glyphBase,
	GL_SYSTEM_FONT_NAME_NV, "Arial", GL_BOLD_BIT_NV,
	wordLen, GL_UNSIGNED_BYTE, word,
	GL_SKIP_MISSING_GLYPH_NV, ~0, emScale);
glPathGlyphsNV(glyphBase,
	GL_STANDARD_FONT_NAME_NV, "Sans", GL_BOLD_BIT_NV,
	wordLen, GL_UNSIGNED_BYTE, word,
	GL_USE_MISSING_GLYPH_NV, ~0, emScale);
```

## GLSL协同

&#160; &#160; &#160; &#160;有`void ProgramPathFragmentInputGenNV(uint program, int location, enum genMode, int components, const float *coeffs);`函数，可以快速生成片元着色器的输入，只要定制片元着色器即可。

## 详细文档

&#160; &#160; &#160; &#160;详细文档可见[https://www.khronos.org/registry/OpenGL/extensions/NV/NV_path_rendering.txt](https://www.khronos.org/registry/OpenGL/extensions/NV/NV_path_rendering.txt).

## 性能指标

&#160; &#160; &#160; &#160;GL\_NV\_path\_rendering可以看NVIDIA在SIGGRPAH 2012上的论文：

[http://developer.download.nvidia.com/devzone/devcenter/gamegraphics/files/opengl/gpupathrender.pdf](http://developer.download.nvidia.com/devzone/devcenter/gamegraphics/files/opengl/gpupathrender.pdf)

# Khronos版本

&#160; &#160; &#160; &#160;GL\_NV\_path\_rendering这个扩展是不是感觉非常爽，所以当然，确实是有Khronos版本的。而且这个东西，很多人认为会很常用，又和OpenGL关系不强，就独立成了一套新的API，叫OpenVG。。。

&#160; &#160; &#160; &#160;OpenVG估计知道的人不少，但是想用的人应该不多了，因为这货的兼容性，真的是超差的。
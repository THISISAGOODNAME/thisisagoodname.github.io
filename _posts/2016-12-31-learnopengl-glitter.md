---
layout: post
title: "learnopengl glitter版"
description: "learnopengl glitter版"
category: 开发环境
tags: [开发环境,c++]
---

&#160; &#160; &#160; &#160;最近发现了一个优秀的图形学框架，[Glitter](http://polytonic.github.io/Glitter/)。Glitter已经包含了新手学习图形学(openGL)需要的绝大部分框架，不需要再另行配置。

<!-- more -->

* Table of Contents
{:toc}

# Glitter

## Glitter简介

&#160; &#160; &#160; &#160;正如[Glitter官网](https://github.com/Polytonic/Glitter)简介中的一样，Glitter是一个绝对简单的OpenGL模板，内部配置好了一切所需要的库。你唯一需要的外部软件就是[cmake](http://www.cmake.org/download/)。

## Glitter集成的库

&#160; &#160; &#160; &#160;Glitter默认集成的库如下

Functionality           | Library
----------------------- | ------------------------------------------
Mesh Loading            | [assimp](https://github.com/assimp/assimp)
Physics                 | [bullet](https://github.com/bulletphysics/bullet3)
OpenGL Function Loader  | [glad](https://github.com/Dav1dde/glad)
Windowing and Input     | [glfw](https://github.com/glfw/glfw)
OpenGL Mathematics      | [glm](https://github.com/g-truc/glm)
Texture Loading         | [stb](https://github.com/nothings/stb)

>其中，glad作用是取代glew，std是SOIL的底层实现

## Glitter使用教程

```bash
git clone --recursive https://github.com/Polytonic/Glitter
cd Glitter
cd Build

# UNIX Makefile
cmake ..

# Mac OSX
cmake -G "Xcode" ..

# Microsoft Windows
cmake -G "Visual Studio 14" ..
cmake -G "Visual Studio 14 Win64" ..
```

之后就可以使用自己习惯的IDE了。而clion用户的话，直接用clion打开Glitter根目录即可。

# LearnOpenGL

## LearnOpenGL简介

&#160; &#160; &#160; &#160;[LearnOpenGL](https://learnopengl.com/)以及[中文版](https://learnopengl-cn.github.io/)，是一个非常优秀的OpenGL教程，由浅入深，从窗口绘制一直教到HDR,SSAO,PBR等高级技术，全教程坚持使用实例来演示，非常适合新人学习。

## LearnOpenGL-Glitter版本

&#160; &#160; &#160; &#160;我自己用了10天不到的时间，把LearnOpenGL上的实例用Glitter框架重新实现了一遍，[链接在这里](https://github.com/THISISAGOODNAME/learnopengl-glitter)。

## Glitter使用时的一些注意事项

1. 使用glad获取opengl函数

```c++
// Load OpenGL Functions
gladLoadGL();
std::cout << "OpenGL " << glGetString(GL_VERSION) << std::endl;
```

2. 使用stb读取图片

```c++
#define STB_IMAGE_IMPLEMENTATION
int width, height;
unsigned char* image = stbi_load("wall.jpg", &width, &height, 0, 0);
stbi_image_free(image);
```

3. 在mac上强制glfw使用版本向前兼容

```c++
// Mac 10.9以后OpenGL版本为祖传OpenGL4.1，不能改变
// 如果使用3.3或以下版本，需要兼容
glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);
```

4. 在retina屏上设置viewport只占屏幕1/4

```c++
// 直接设置窗口分辨率是retina拉伸后的
// 而设置viewport的尺寸是retina拉伸前的
// 从帧缓冲中获取viewport尺寸是正确做法
int width, height;
glfwGetFramebufferSize(window, &width, &height);
glViewport(0, 0, width, height);
```
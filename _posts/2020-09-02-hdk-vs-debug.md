---
layout: post
title: "用Visual Studio调试HDK"
description: "用Visual Studio调试HDK"
image: ""
date: 2020-09-02 00:01:20
categories: Houdini
tags: [Houdini,HDK]
---
<!-- more -->
* Table of Contents
{:toc}

# 用Visual Studio编译HDK工程

&nbsp; &nbsp; &nbsp; &nbsp;推荐使用CMake来构建HDK工程，CMake组织方式可见[SideFX提供的教程](https://www.sidefx.com/docs/hdk/_h_d_k__intro__compiling.html#HDK_Intro_Compiling_CMake)。Visual Studio已经添加了CMake支持，现在Visual Studio可以打开CMake项目的根目录，会自动配置CMake工程。用经典的**SOP_Star**为例子

![VS CMake proj](http://aicdg.com/assets/img/blogimg/houdini/hdkdebug/Snipaste_2020-09-01_23-38-27.png)

&nbsp; &nbsp; &nbsp; &nbsp;之后Build项目，CMake会帮我们配置好houdini DSO的其他一切。

# 使用Visual Studio调试HDK

&nbsp; &nbsp; &nbsp; &nbsp;先运行houdini

![VS set breakpoint](http://aicdg.com/assets/img/blogimg/houdini/hdkdebug/Snipaste_2020-09-01_23-42-08.png)

&nbsp; &nbsp; &nbsp; &nbsp;给SOP cook设置断点，之后选择**Debug->Attach to Proecss..**，将调试器附加到houdini上，执行SOP节点

![SOP](http://aicdg.com/assets/img/blogimg/houdini/hdkdebug/Snipaste_2020-09-01_23-48-32.png)

&nbsp; &nbsp; &nbsp; &nbsp;执行SOP节点后，会击中c++代码中的断点

![Hit breakpoint](http://aicdg.com/assets/img/blogimg/houdini/hdkdebug/Snipaste_2020-09-01_23-54-32.png)

# 使用Visual Studio debugger attach后却不击中断点的处理

&nbsp; &nbsp; &nbsp; &nbsp;有时发现明明已经把调试器attach了，但是断点却不触发，这是因为Visual Studio可能会把houdini识别为python进程，attach了一个python的调试器

[attach python debugger](http://aicdg.com/assets/img/blogimg/houdini/hdkdebug/Snipaste_2020-09-01_23-55-16.png)

&nbsp; &nbsp; &nbsp; &nbsp;将调试类型设置为native即可修复

[native debugger](http://aicdg.com/assets/img/blogimg/houdini/hdkdebug/Snipaste_2020-09-01_23-55-27.png)

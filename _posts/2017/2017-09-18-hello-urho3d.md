---
layout: post
title: "你好，Urho3D"
description: "你好，Urho3D"
category: c++
tags: [c++,游戏引擎,Urho3D]
---

&nbsp; &nbsp; &nbsp; &nbsp;我昨天博文好像说[Urho3D](https://urho3d.github.io/)配置复杂来着，我必须自打50大板。Urho3D配置不仅不复杂，而且是异常的简单清晰明快。

<!-- more -->

* Table of Contents
{:toc}

# Urho3D简介

&nbsp; &nbsp; &nbsp; &nbsp;[Urho3D](https://urho3d.github.io/)是一个开源跨平台的2D和3D**游戏引擎**。在官网介绍中，Urho3D受[OGRE](http://www.ogre3d.org/)和[Horde3D](http://horde3d.org/)影响很大。OGRE和Horde3D都是渲染引擎，OGRE当然名声在外，Horde3D也是非常不错(PS: Horde3D Scene Editor和keyshot界面程度极其相似，我也不好再多推测什么)。

&nbsp; &nbsp; &nbsp; &nbsp;不过，请注意，Urho3D是个**游戏引擎**(敲黑板)。Urho3D在渲染引擎之外，还集成了**基于组件的场景模型**(和传统Scene Graph在理念上差距最大的地方)、音频管理、存档管理、输入输出管理、网络IO、UI、AI(自动寻路和群体模拟)、2D/3D物理引擎，还有脚本系统(Urho3D的脚本系统是很令我很震撼的，支持[AngelScript](http://www.angelcode.com/angelscript/)和[Lua](https://www.lua.org/)两者脚本，而且是完全支持，所有demo在c++之外，均用AngelScript和Lua也实现了一遍，足见原作者对于脚本系统的重视程度。而且也非常好的向我安利了一发AngelScript)

# 编译Urho3D

&nbsp; &nbsp; &nbsp; &nbsp;我以mac举例吧，版本使用本文写作时最新的稳定版v1.7，Urho3D的更新其实相当的频繁，其实Windows/ios/android/web(Emscripten)都差不多。Urho3D在mac和win下编译都挺惬意的，但是linux下则是十分的蛋疼。在linux下需要手工解决odbc，显示服务，音频服务，输入输出四大类依赖；macOS/iOS/tvOS则只需要安装一个xcode；对于Windows，VS2012以下需要安装DirectX SDK June 2010，对于VS2012以上版本就只需要安装VS ；对于Android，需要安装Android SDK(v12+)以及Android NDK(r7+);对于Web平台，只需要Emscripten(win上还额外需要MinGW-W64编译器，不过没这玩意你又是怎么编译的Emscripten)。树莓派则在linux的基础上，额外多需要libevdev2库，并且需要分配的显存多余128m。

## 用推荐方式编译Urho3D

&nbsp; &nbsp; &nbsp; &nbsp;下载Urho3D源码后，解压源码，能看见非常多的.sh和.bat文件，在命令行下运行`<script-name> /path/to/build-tree [build-options]`即可完成对于平台和工程的创建，比如我想在这个文件夹下的build子目录下创建工程(其实就是CMake out of source方式编译)

```bash
 ./cmake_xcode.sh ./build
```
> window的同学请使用cmake_vsXXXX.bat ./build

&nbsp; &nbsp; &nbsp; &nbsp;build-options就是定制编译选项了，具体可见[官方文档](build-options)。我这次不做任何定制，直接使用默认的推荐配置。

&nbsp; &nbsp; &nbsp; &nbsp;之后，就和CMake常规一样了，用xcode/VS/make来编译工程。

## 运行demo

&nbsp; &nbsp; &nbsp; &nbsp;首先强调一下，3种语言完全等价，没有孰优孰劣之分

### 1. 运行C++ demo

&nbsp; &nbsp; &nbsp; &nbsp;运行C++ demo最为简单，只要在build/bin下直接执行可执行文件即可，比如想要运行06_SkeletalAnimation,只要执行

```bash
./06_SkeletalAnimation
```

![06_SkeletalAnimation](http://7xqrar.com1.z0.glb.clouddn.com/urho3DQQ20170918-230603@2x.png)

### 2. 运行Lua demo

&nbsp; &nbsp; &nbsp; &nbsp;在build/bin提供了Urho3DPlayer，可以很方便的测试脚本程序，比如想测试23_Water的lua版，只要运行

```bash
./Urho3DPlayer Data/LuaScripts/23_Water.lua -w
```

![23_Water](http://7xqrar.com1.z0.glb.clouddn.com/urho3DQQ20170918-230627@2x.png)

> 参数-w表示以窗口模式运行。这个水的效果还是非常不错的，透明渲染、波纹、菲涅尔都有

### 3. 运行AngleScript demo

&nbsp; &nbsp; &nbsp; &nbsp;AngleScript是urho3D推荐的脚本语言，和C++相性极棒，语法和C++基本一样，C++和AngleScript也可以很方便的互调用。lua全是C的接口，C++导出时非常痛苦，AngleScript则无此问题，我已经被AngleScript完全圈粉。想要运行13_Ragdolls的AngleScript版本，只要运行

```bash
./Urho3DPlayer Data/Scripts/13_Ragdolls.as -w
```

![13_Ragdolls](http://7xqrar.com1.z0.glb.clouddn.com/urho3DQQ20170918-230649@2x.png)

# 在本地配置urho3D开发环境

&nbsp; &nbsp; &nbsp; &nbsp;我之前被某些不负责任(也可能是水平有限)的人误导过，以为urho3D开发环境很难搭建。那是不对的，其实urho3D开发环境搭建非常简单。

## 基础目录结构

&nbsp; &nbsp; &nbsp; &nbsp;建立基础目录结构

1. 新建一个空目录，作为项目的根目录。
2. 把urho3D目录下的bin和CMake两个目录，复制到_PROJECT\_ROOT_，或者建立软连接
3. 在_PROJECT\_ROOT_下新建CMakeLists.txt
4. 建立需要的项目源文件，比如cpp和h文件

&nbsp; &nbsp; &nbsp; &nbsp;现在目录结构应该如下

```
<PROJECT_ROOT>/
 ├ bin/
 │  ├ Data/
 │  └ CoreData/
 ├ CMake/
 │  ├ Modules/
 │  └ Toolchains/
 ├ CMakeLists.txt
 ├ *.cpp and *.h
 └ *.bat or *.sh
```

## 修改CMakeLists.txt文件

&nbsp; &nbsp; &nbsp; &nbsp;复制如下内容即可

```
# Set CMake minimum version and CMake policy required by UrhoCommon module
cmake_minimum_required (VERSION 3.2.3)

if (COMMAND cmake_policy)
    # Libraries linked via full path no longer produce linker search paths
    cmake_policy (SET CMP0003 NEW)
    # INTERFACE_LINK_LIBRARIES defines the link interface
    cmake_policy (SET CMP0022 NEW)
    # Disallow use of the LOCATION target property - so we set to OLD as we still need it
    cmake_policy (SET CMP0026 OLD)
    # MACOSX_RPATH is enabled by default
    cmake_policy (SET CMP0042 NEW)
    # Honor the visibility properties for SHARED target types only
    cmake_policy (SET CMP0063 OLD)
endif ()

project(urho3dTest)

set(CMAKE_CXX_STANDARD 11)

# (Optional)Set URHO3D path
set(URHO3D_HOME "${CMAKE_SOURCE_DIR}/Urho3D-1.7/build")

# Set CMake modules search path
set (CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/CMake/Modules)

# Include UrhoCommon.cmake module after setting project name
include (UrhoCommon)

# Define target name
set (TARGET_NAME MyExecutableName)

file (GLOB CPP_FILES *.cpp)
file (GLOB H_FILES *.h)
set (SOURCE_FILES ${CPP_FILES} ${H_FILES})

# Define source files
define_source_files ()

# Setup target with resource copying
setup_main_executable ()
```

> `URHO3D_HOME`在没有install到默认目录时才需要设置

## 运行官方demo

&nbsp; &nbsp; &nbsp; &nbsp;有了这样的基础工程之后，运行官方demo就非常简单了。吧源码路径下的Source/Samples下的`Sample.h`、`Sample.inl`已经你想运行的Demo下的全部源文件复制到_PROJECT\_ROOT_即可。

# 小结

&nbsp; &nbsp; &nbsp; &nbsp;urho3D是我以后推荐游戏引擎架构学习的首选了，目录清晰，代码注释详实，功能充足，轻量但并不简单，对于突破了Scene Graph，想对游戏引擎架构有更深一层研究的同学，这个绝对是不差的选择。

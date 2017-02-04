---
layout: post
title: "cmake和其他构建工具协同使用"
description: "cmake和其他构建工具协同使用"
category: c++
tags: [c++ , 自动化工具 ]
---

&#160; &#160; &#160; &#160;cmake作为一个优秀的跨平台c++编译工具链，可以较好的解决c/c++跨平台编译也依赖构建问题，cmake的`add_subdirectory()`函数以及`Find*()`系列函数，更是大幅度简化依赖系统。把依赖的全部源码扔到工程下一起打包，非常爽。

<!-- more -->

* Table of Contents
{:toc}

# 起因

&#160; &#160; &#160; &#160;把全部源码当做依赖在cmake中一起编译，是多么的美好，特别是引入的库也是由cmake管理的时候。比如[Glitter](https://github.com/Polytonic/Glitter)，就可以非常轻松的将[bullet](https://github.com/bulletphysics/bullet3)、[glfw](https://github.com/glfw/glfw)、[glad](https://github.com/Dav1dde/glad)、[assimp](https://github.com/assimp/assimp)引入项目。而且，[conan](https://www.conan.io/)、[biicode](https://www.biicode.com/)等c++的包管理的出现，让cmake更是爽到飞起。

&#160; &#160; &#160; &#160;但是，还是有很多框架和类库，并没有使用cmake管理，而且继续使用autotools、ninja、xmake等其他编译工具，甚至手写Makefile来维护。像lua这样的项目，虽然使用手工编写Makefile来维护，但总共也没几个文件，把core和lib相关的c文件直接编译到项目中就好了。

&#160; &#160; &#160; &#160;不过在lua向luajit迁移的时候，第一次感受到了cmake的无力。本来也是希望能把luajit像lua一样，把luajit源代码直接在项目中编译。但是，这是不可能的。因为有几个头文件，是**编译过程中生成的**，也就是说，如果采用源码添加到项目中的方法，你永远也找不到这个平台相关的头文件，也就无法完成编译。

# 大招：cmake中调用外部构建工具

&#160; &#160; &#160; &#160;cmake提供了一个扩展，[ExternalProject](https://cmake.org/cmake/help/v3.7/module/ExternalProject.html)，这个扩展可以强行先在项目中使用外部构筑工具构建依赖，在导入到项目中使用。使用该工具，可以很好的解决像luajit这样的硬骨头，但是缺点更明显：**CMakeLists.txt配置文件很可能丧失跨平台能力，除非针对众多平台分别写ExternalProject构建和导入规则**引入项目。而且，

# 实例：cmake在POSIX系统下，添加luajit

&#160; &#160; &#160; &#160;要使用ExternalProject模块需要先引入:

```cmake
include(ExternalProject)
```

&#160; &#160; &#160; &#160;引入后，可以使用`ExternalProject_Add`，`ExternalProject_Add_Step`，`ExternalProject_Add_StepDependencies`这些核心函数。

&#160; &#160; &#160; &#160;`ExternalProject_Add_Step`函数的主要命令就是`COMMAND <cmd>`，可以逐步执行shell命令。`ExternalProject_Add_StepDependencies`可以理解成`add_dependencies()`的`ExternalProject_Add_Step`版本，这两个函数使用频率都不太高。

&#160; &#160; &#160; &#160;`ExternalProject_Add`是`ExternalProject`模块中最重要的函数，先列举一下cmake集成luajit的例子：

```cmake
include(ExternalProject)

ExternalProject_Add(project_luajit
        URL http://luajit.org/download/LuaJIT-2.1.0-beta2.tar.gz
        URL_MD5 fa14598d0d775a7ffefb138a606e0d7b
        PREFIX ${CMAKE_CURRENT_BINARY_DIR}/luajit-2.1.0-beta2
        UPDATE_COMMAND ""
        CONFIGURE_COMMAND ""
        BUILD_COMMAND make PREFIX=${CMAKE_CURRENT_BINARY_DIR}/luajit-2.1.0-beta2
        BUILD_IN_SOURCE 1
        INSTALL_COMMAND make install PREFIX=${CMAKE_CURRENT_BINARY_DIR}/luajit-2.1.0-beta2
        )

ExternalProject_Get_Property(project_luajit install_dir)


add_library(luajit STATIC IMPORTED)
set_property(TARGET luajit PROPERTY IMPORTED_LOCATION ${install_dir}/lib/libluajit-5.1.a)
add_dependencies(luajit project_luajit)

include_directories(${install_dir}/include/luajit-2.1)

add_executable(luajitrepl main.cpp)
target_link_libraries(luajitrepl luajit)
```

&#160; &#160; &#160; &#160;就如同例子一样，如果使用make/autotools系的工具，和使用命令行直接编译几乎一样简单。注意，定义的PREFIX变量表示ExternalProject的根目录，和`make install PREFIX`无关。make步骤中一般工程是根本不能设置PREFIX变量的，这里是luajit的一个设计，就是luajit在POSIX系统上编译时，需要用`make PREFIX=<PATH>`制定luajit的动态库位置，不设置默认为`/usr/local`，在主项目中用`link_directories()`设置无效(在嵌入式设备，比如安卓、ios、ps4中，luajit静态编译，在ios和ps4上甚至不能开启jit只能用解释器模式)。

&#160; &#160; &#160; &#160;ExternalProject_Add除了支持URL下载源码(必须是.tar.gz打包)，还支持CVS、git、SVN、mercurial(hg)等版本控制软件拉取代码。还有`DOWNLOAD_NAME`和`DOWNLOAD_DIR`用来下载文件和文件夹。

&#160; &#160; &#160; &#160;之后，可以使用`ExternalProject_Get_Property()`读取属性，能读取的属性必须是`ExternalProject_Add()`的内部属性，具体可参阅文档，`ExternalProject_Get_Property()`第一个参数是保存变量名，第二+个参数是要查询的属性，都保存在第一个变量中。

# 总结

&#160; &#160; &#160; &#160;ExternalProject可以使用外部工具构建cmake并不支持的工程或必须预编译才能导入的类库，不过，整体需求并不大。绝大部分工程都可以作为子项目在cmake中直接编译。ExternalProject还很可能使cmake丧失跨平台能力。建议还是谨慎使用。
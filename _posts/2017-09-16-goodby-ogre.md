---
layout: post
title: "再见,ogre"
description: "再见,ogre"
category: c++
tags: [c++,游戏引擎]
---

&nbsp; &nbsp; &nbsp; &nbsp;这篇文章其实应该是三个月以前和就和大家见面了，但是因为工作上的很多原因，拖了很久。

<!-- more -->

* Table of Contents
{:toc}

# OGRE是什么

&nbsp; &nbsp; &nbsp; &nbsp;[OGRE](http://www.ogre3d.org/)是Object-**O**riented **G**raphics **R**endering **E**ngine的简称，中文名应该是面向对象的图形渲染引擎，英文缩写OGRE，刚好又和食人魔(OGRE)相同，所以OGRE无论是图标还是demo都愿意使用食人魔作为造型。

# OGRE和mac

&nbsp; &nbsp; &nbsp; &nbsp;这一节没啥营养，可以直接跳过。

## 编译OGRE

&nbsp; &nbsp; &nbsp; &nbsp;OGRE在linux和Windows上编译过程都非常顺畅，舒心，但是在mac上，就是怎么搞怎么不得劲，还不如emscripten版本来得顺心。

### mac上配置OGRE

&nbsp; &nbsp; &nbsp; &nbsp;我以本文写作时最新的稳定版本OGRE 1.10.8版本为例，在官方文档中，OGRE v2.1版本据称已经非常稳定(**黑人问号**)，[Racecraft](https://steamcommunity.com/app/346610)就使用了OGRE v2.1, OGRE v2.1的默认分支是再mac上无论如何都编译不过去的，所以不用和自己较劲了，OGRE v2.1在[bitbucket](https://bitbucket.org/sinbad/ogre)还有一个metal分支，是给ios和支持metal的mac准备的，这个cmake还有clang都很有使用经验的人可以试着编译一下。反正要死要活之后，我在osx 10.12.5上使用xcode 8.3.3(`Apple LLVM version 8.1.0 (clang-802.0.42)`）上算是编译过去了，但真的是不想再来一次。metal分支和v2-1分支开发进度是不同的，cherry-pickv2-1分支的patch到metal分支的时候处理冲突也是极度的痛苦的体验。

&nbsp; &nbsp; &nbsp; &nbsp;说了半天v2.1的编译，该说正事了，v1.10.8

#### 1. 下载OGRE源代码，然后解压

&nbsp; &nbsp; &nbsp; &nbsp;没啥好说的，解压之后cd到这个目录就行。

#### 2. 修改源码下的CMakeLists.txt文件

&nbsp; &nbsp; &nbsp; &nbsp;这步是很痛苦的领悟。

&nbsp; &nbsp; &nbsp; &nbsp;ogre是有很多可选依赖的，比如

1. Essential dependencies:
	* freetype: http://www.freetype.org
2. Recommended dependencies:
	* zlib: http://www.zlib.net
	* zziplib: https://github.com/paroj/ZZIPlib
	* SDL: https://www.libsdl.org/
3.  Optional dependencies:
	* DirectX SDK: http://msdn.microsoft.com/en-us/directx/
	* FreeImage: http://freeimage.sourceforge.net
	* Doxygen: http://doxygen.org
	* Cg: http://developer.nvidia.com/object/cg_toolkit.html
	* Boost: http://www.boost.org (+)
	* POCO: http://pocoproject.org (+)
	* TBB: http://www.threadingbuildingblocks.org (+)
	* OpenEXR: http://www.openexr.com/

> (+) used to build threaded versions of Ogre. You can use either C++11, POCO or TBB instead of Boost. When using Boost, only the boost-thread, boost-system and boost-date-time libraries are required.

&nbsp; &nbsp; &nbsp; &nbsp;然后就是关键了，如果先在mac上安稳的编译ogre 1.10.8，上述的扩展除了SDL、OpenEXR和Doxygen，**一个都不要安装**，别手欠。freetype和zziplib是编译必要的，cmake时会自动下载配置，不需要全局提供。

&nbsp; &nbsp; &nbsp; &nbsp;对于多线程库，Boost、POCO、TBB或者c++11可以四选一，但是经过我在xcode 8.3.3上严密的测试，**这四个版本每个都编译不过去**！！！(当然是指Boost、POCO、TBB都直接使用homebrew安装最新的情况)(c++11编译不过去我很不理解，也行mac上真的只能指望一开始就使用c++11的ogre v2了)

&nbsp; &nbsp; &nbsp; &nbsp;对于Cg，请也选择放弃。不能使用cgFx影响其实没那么大，当然更关键的原因，是使用Cg之后，你在Apple LLVM v8+上根本链接不了。Cg这个东西，在2012年，终于走到了声明的尽头，请相信，openGL 3.3+ core的shader已经足够强大，不用Cg了。对于Cg生命结束，NVIDIA也写了很多说明，这个东西已经从shader的先驱变成了抑制剂的时候，放弃其实也不失为一种选择。对于GLSL运行效率有疑惑，觉得不如预编译优化的，可以试用一下[glsl-optimizer](https://github.com/aras-p/glsl-optimizer)，如果对GLSL语言本身有偏见，是坚定的Cg/HLSL党，还有[HLSL2GLSL](https://github.com/aras-p/hlsl2glslfork)可以用。两者都是Aras Pranckevičius老爷子的作品，老爷子现在供职Unity，博客异常简谱，不过Shader X3和Shader X4各入选了两篇文章足够说明其实力，2016年还写出了世界上效率暂时最高的c++ hash库，真是吓人。

&nbsp; &nbsp; &nbsp; &nbsp;言归正传，该关的都关了之后，就可以编译了。就是cmake正常用法，没啥好说的

```bash
mkdir build && cd build
cmake ..
cmake --build . --config release
make OgreDoc # optional
make install  # (or sudo make install, if root privileges are required)
```

> `make install`在linux上会安装到`/usr/local`下，但是win和mac上会安装到编译目录下的`sdk`子目录下

#### 3. 运行SampleBrowser

&nbsp; &nbsp; &nbsp; &nbsp;这也是mac上编译OGRE体验最差的一步，linux和win上已经可以看看demo体验一下了，mac上则是蛋碎的依赖处理的时间。

&nbsp; &nbsp; &nbsp; &nbsp;命令行下运行`SampleBrowser.app/Contents/MacOS/SampleBrowser`的话，可以看到依赖的缺失情况。

&nbsp; &nbsp; &nbsp; &nbsp;把`Contents/Frameworks` 、`Contents/Plugins`和`Contents/Resources`下的软链接都处理一下(源码编译的应该只有`Contents/Plugins`有问题，其他两个正常，官网下载的prebuild包则是全都有问题，需要手工都处理一下)

## OGRE和html5

&nbsp; &nbsp; &nbsp; &nbsp;这个顺手写一下吧，非常简单。

&nbsp; &nbsp; &nbsp; &nbsp;下载代码部分都一样，执行cmake时，运行

```bash
cmake -DCMAKE_TOOLCHAIN_FILE=$EMSCRIPTEN/cmake/Modules/Platform/Emscripten.cmake .
```

&nbsp; &nbsp; &nbsp; &nbsp;然后设置`CMAKE_AR `为`emcc`,之后运行`make`即可，觉得慢可以用`make -j <线程数>`编译来提升速度，在`${CMAKE_BINARY_DIR}/bin/`下有名为`EmscriptenSample.html `的文件，架个server，比如`python3 -m http.server 8000`然后打开浏览器即可体验。

# 一些闲话

&nbsp; &nbsp; &nbsp; &nbsp;刚刚mac上1.10.8的编译过程是不是看吐了，我只能说mac上编译[v2-1-metal-macos](https://bitbucket.org/sinbad/ogre/branch/v2-1-metal-macos)还要更加更加痛苦。也许mac，真的就是ios游戏开发者的必选，安卓游戏开发者佳选，在mac上开发端游可能真不是什么好选择。毕竟OpenGL驱动到4.1，以后不会再更新了，ogl调试工具也不咋地，[OpenGL Profiler](https://developer.apple.com/library/prerelease/content/technotes/tn2178/_index.html)不会再更新了，[RenderDoc](https://renderdoc.org/)应该也不会支持mac了，也就[apitrace](http://apitrace.github.io/)的mac版本还在更新吧。相比之下，移动平台调试就好的多，ios那边有[Xcode GPU Tools](https://developer.apple.com/library/ios/documentation/3DDrawing/Conceptual/OpenGLES_ProgrammingGuide/ToolsOverview/ToolsOverview.html)来调试OpenGL ES, Metal，Android那边有[GAPID](https://github.com/google/gapid)来调试OpenGL ES, Vulkan。其实我个人很赞同一个趋势，就是图形学入坑的话应该尽量简单些，以前的话推荐入坑学opengl es，现在的话就是webgl。

&nbsp; &nbsp; &nbsp; &nbsp;话题回到Ogre本身，Ogre我的评价的话，就是改变了行业的悲剧英雄吧。Ogre以前的很长一段时间里，都是游戏引擎教学的主流，在那个信息相对闭塞，小学生们也不是那么热衷于编程的年代，ogre真的教会了很多人渲染引擎是什么，该怎么用。但是在虚幻火起来之后，一切就变了，Ogre成了过度设计的代表。感觉在15年以后，unreal engine 4彻底免费开始(对了，狗逼Epic啥时候还我ue4订阅费，240美元啊)，会用ue4的小学生中学生大神就越来越多了，得承认长江后浪推前浪，自古英雄出少年。不过这批青年开发者，感觉普遍都比较看不起用Ogre，gamebryo，鬼火，panda 3D，JME这批，我也不知道该怎么和他们交流。可能确实是我不行吧。

&nbsp; &nbsp; &nbsp; &nbsp;Ogre从现在看，未来还是有点不太明确，使用难度上，应该是比UE4要更累，比较系统集成度上就不一样，图形引擎怎么可能超得过完整的游戏引擎，而且是资深游戏方案供应商呢。横向比较，同样老牌的OSG，活得也不太好。在青年开发者中，ogre/osg都是陈旧、迂腐、过度设计典型了。往下看，他有个后辈[Urho3D](https://urho3d.github.io/)势头也很猛，这个也挺受年轻人追捧的，不过给我感觉受年轻人追捧的原因也就是--这玩意足够复杂，周围不是谁都弄得起来，可以装B(也许以后有机会我也写一下这个的配置教程？)。作为UE4首批用户，我当年反正是这样。当时ue4还不是完全免费的，要订阅，19美元一个月，还没有ue启动器，，ue4开源了，确实，MD没prebuild啊，用法是下载4个压缩包，差不多10g的源码文件，下下来用vs编译(到了4.5版本才有mac版，4.4版本的时候才能启动器下预编译版本)，4770这种cpu大概要40分钟才能编译完，编完了再同学面前装装逼，写个hello world，把start里那个小人可以方向移动鼠标镜头，之后好像也没干啥更大的事了。也希望某些小学生中学生大神可以吸取我当时的教训，往真正的大神，不是配环境+hello world大神努力。
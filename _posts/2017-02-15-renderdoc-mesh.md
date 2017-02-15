---
layout: post
title: "renderdoc逆向提取mesh"
description: "renderdoc逆向提取mesh"
category: 建模
tags: [建模,c++]
---

&#160; &#160; &#160; &#160;最近发现了一个帧调试神器[renderdoc](https://github.com/baldurk/renderdoc)。

<!-- more -->

&#160; &#160; &#160; &#160;renderdoc的二进制文件可以从[这里](https://renderdoc.org/builds)下载。renderdoc的开发商就是大名鼎鼎的Crytek。renderdoc作为优秀的帧调试器，可以非常好的调试DX11和openGL 3.2 core+。虽然只有Windows版限制了其发展(其实有linux版，不太好用)。unity5开始，更是内置了renderdoc的集成，证明renderdoc被业界的接受程度。（顺便吐个槽，unity防renderdoc逆向做的也不错）

* Table of Contents
{:toc}

&#160; &#160; &#160; &#160;其实相信大部分人都知道renderdoc逆向提取mesh的方法，我就是随便写一下，不过更告诫大家尊重知识产权，不要乱搞。

## 1. 选择要调试的程序

&#160; &#160; &#160; &#160;选择要调试的程序即可，工作路径可以放在游戏目录也可以到别处。其实renderdoc有个问题，就是带启动器的游戏，抓窗口就不太准。

![选择要调试的游戏](http://7xqrar.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BE20170214190835.png)

## 2. 保存要分析的帧

&#160; &#160; &#160; &#160;在游戏运行过程中，按f12截图，找到要分析的帧后退出游戏即可。

![保存截图](http://7xqrar.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BE20170214191056.png)

&#160; &#160; &#160; &#160;然后打开要分析的帧。

![打开要分析的帧](http://7xqrar.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BE20170214191128.png)

## 3. 逐步分析绘制过程，找到绘制要提取模型的渲染步骤

&#160; &#160; &#160; &#160;renderdoc可以非常好的找到渲染的pass以及每个pass的结果。我已涅普的脸部提取为例。

![过程1](http://7xqrar.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BE20170214191333.png)

![过程2](http://7xqrar.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BE20170214191344.png)

&#160; &#160; &#160; &#160;通过每个绘制步骤的结果查看，找到绘制脸部的绘制步骤。

![过程3](http://7xqrar.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BE20170214193305.png)

&#160; &#160; &#160; &#160;切换到Mesh Output选项卡，其实已经可以预览这个渲染过程的顶点输入情况了。保存顶点信息到某个文件中，就可以在别的程序中使用。

## 4. 使用导出的顶点信息

&#160; &#160; &#160; &#160;这个模型导出的顶点信息非常充足，位置、法线、纹理坐标等信息都导出了，我这里只使用顶点信息，在OpenGL中以线框模式显示此模型。

![结果](http://7xqrar.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BE20170214210853.png)

&#160; &#160; &#160; &#160;该程序源代码可以在[这里](https://github.com/THISISAGOODNAME/learnopengl-glitter/tree/master/other/renderDoc%20reverse%20engineering)查看。这个脸部模型顶点只有3000多个，不算多，为了省事我就直接全扔到代码里了。程序使用[glitter](https://github.com/Polytonic/Glitter)模板开发，相机、shaderprogram还有命令行彩色显示的辅助头文件在[这里](https://github.com/THISISAGOODNAME/learnopengl-glitter/tree/master/Headers)。需要说明，命令行色彩显示用的是UNIX风格的，需要在POSIX环境下使用，在win下推荐一下babun这个开箱即用的zsh终端。

## 5. 结语

&#160; &#160; &#160; &#160;renderdoc提取模型和贴图相比一些逆向软件可能确实是要麻烦一些，但是可控性高很多。

&#160; &#160; &#160; &#160;然后是提醒大家，尊重知识产权。本文只做研究使用，不涉及任何经济利益。
---
layout: post
title: "renderdoc调试总结"
description: "renderdoc调试总结"
image: ""
date: 2018-08-03 14:14:04
categories: 研究
tags: [研究,]
---

<!-- more -->
* Table of Contents
{:toc}

&nbsp; &nbsp; &nbsp; &nbsp;最近抽时间研究了一下 renderdoc 单步调试 HLSL 的方法。然后为了解决升级 UE4.20 后某些材质变得诡异的问题，又研究了一下如何调试 UE shader 的方法，真的把自己折腾的够呛，很感谢天美的一位前辈的指点，让我茅塞顿开。

# renderdoc 调试 UE4 shader

&nbsp; &nbsp; &nbsp; &nbsp;renderdoc调试HLSL是支持单步调试的，但是调试UE4的时候很多人只能看到汇编(DXBC)，不能看到源码，这是因为UE4 ShaderCompileWorker默认是生成优化且不保存调试符号的DXBC文件，只要调整配置将HLSL调试信息保留且取消编译器优化即可。

&nbsp; &nbsp; &nbsp; &nbsp;首先编辑 **引擎目录** 下的 `Config/ConsoleVariable.ini` 文件。

![Config/ConsoleVariable.ini](http://aicdg.com/assets/img/blogimg/rdc/ue4/Snipaste_2018-08-03_11-02-02.png)

&nbsp; &nbsp; &nbsp; &nbsp;仔细阅读注释说明，保留HLSL调试信息并关闭HLSL编译器优化。

![保留HLSL调试信息并关闭HLSL编译器优化](http://aicdg.com/assets/img/blogimg/rdc/ue4/Snipaste_2018-08-03_11-03-05.png)

&nbsp; &nbsp; &nbsp; &nbsp;之后打开UE4内置的 renderdoc 插件。在 Edit -> Plugins 的内置插件(Built-in)中搜索 renderdoc 插件。

![搜索 renderdoc 插件](http://aicdg.com/assets/img/blogimg/rdc/ue4/Snipaste_2018-08-03_11-24-29.png)

&nbsp; &nbsp; &nbsp; &nbsp;开启后重启并编译插件，之后可以在Editor viewer的右上角发现renderdoc的图标。

![UE4 renderdoc的图标](http://aicdg.com/assets/img/blogimg/rdc/ue4/Snipaste_2018-08-03_11-27-00.png)

&nbsp; &nbsp; &nbsp; &nbsp;点击该图标，即可完成抓帧。在Event brower中定位到需要分析的drawCall。

![完成抓帧](http://aicdg.com/assets/img/blogimg/rdc/ue4/Snipaste_2018-08-03_11-33-19.png)

&nbsp; &nbsp; &nbsp; &nbsp;在tone mapping之前颜色偏暗看不清，可以重新映射颜色范围。

![重新映射颜色范围](http://aicdg.com/assets/img/blogimg/rdc/ue4/Snipaste_2018-08-03_11-34-07.png)

&nbsp; &nbsp; &nbsp; &nbsp;用鼠标右键在 Texture Viewer 的 Output 中定位到要调试的像素，然后点击 Pixel Context 中的 debug 按钮即可开始调试(请务必确保该 drawcall 影响该 pixel，否则还是会显示该pixel的pixel history)

![定位到要调试的像素](http://aicdg.com/assets/img/blogimg/rdc/ue4/Snipaste_2018-08-03_11-35-03.png)

&nbsp; &nbsp; &nbsp; &nbsp;之后即可开始单步调试HLSL。调试非常方便，不仅可以step forward，还可以step backward。并且存在超方便的功能run to cursor，只要鼠标定位到要执行到的行数即可，连断点都可以不打。双击寄存器可以在变量(Variables面板中高亮变量)，当然悬停也可以显示变量值。

![单步调试HLSL](http://aicdg.com/assets/img/blogimg/rdc/ue4/Snipaste_2018-08-03_11-57-01.png)

&nbsp; &nbsp; &nbsp; &nbsp;其实不仅可以调试，还可以实时修改 shader，之后刷新就能看到效果。比如我把绘制人物的shader改为直接显示红色。

![实时修改 shader](http://aicdg.com/assets/img/blogimg/rdc/ue4/Snipaste_2018-08-03_11-36-51.png)

> dx11(sm5)下默认使用延迟渲染，OutTarget0是color attachment

&nbsp; &nbsp; &nbsp; &nbsp;人物材质被修改后的效果可以在Texture viewer中查看。这个功能很便于开发shader，避免反复编译/重启游戏浪费时间。

![人物材质被修改](http://aicdg.com/assets/img/blogimg/rdc/ue4/Snipaste_2018-08-03_11-37-03.png)

# renderdoc 分析应用程序

&nbsp; &nbsp; &nbsp; &nbsp;很多时候需要用 renderdoc 分析自己或别人的程序。我把 UE4 的 ThirdPerson demo 做了个二进制包，使用 dx11(sm5)， openGL(es31)，vulkan(es31)

![项目图形API配置](http://aicdg.com/assets/img/blogimg/rdc/Snipaste_2018-08-03_11-23-44.png)

## renderdoc 分析 directx11

&nbsp; &nbsp; &nbsp; &nbsp;renderdoc选择应用程序，然后点击Launch运行。

![directx11](http://aicdg.com/assets/img/blogimg/rdc/dx11/Snipaste_2018-08-03_12-50-49.png)

&nbsp; &nbsp; &nbsp; &nbsp;寻找要 debug 的 pixel 和 drawCall。可以通过线选择要debug的pixel，然后通过pixel history快速定位到要debug的drawCall

![pixel history快速定位](http://aicdg.com/assets/img/blogimg/rdc/dx11/Snipaste_2018-08-03_12-54-01.png)

&nbsp; &nbsp; &nbsp; &nbsp;然后点击 Pixel Context 中的 debug 按钮即可开始调试。

![开始调试](http://aicdg.com/assets/img/blogimg/rdc/dx11/Snipaste_2018-08-03_11-35-03.png)

&nbsp; &nbsp; &nbsp; &nbsp;可以在右上角的 **Debug in Assembly** 和 **Debug in HLSL** 之间切换来选择单步调试的方式。

![Debug in HLSL](http://aicdg.com/assets/img/blogimg/rdc/dx11/Snipaste_2018-08-03_12-55-32.png)

![Debug in Assembly](http://aicdg.com/assets/img/blogimg/rdc/dx11/Snipaste_2018-08-03_12-55-57.png)

## renderdoc 分析 openGL

&nbsp; &nbsp; &nbsp; &nbsp;renderdoc选择应用程序，然后运行。UE4的项目可以命令行参数 `-opengl` 强制使用openGL渲染。

![UE4 opengl mode](http://aicdg.com/assets/img/blogimg/rdc/opengl/Snipaste_2018-08-03_13-11-04.png)

&nbsp; &nbsp; &nbsp; &nbsp;抓一帧进行分析。

![抓帧](http://aicdg.com/assets/img/blogimg/rdc/opengl/Snipaste_2018-08-03_13-13-50.png)

> 如果贴图上下颠倒可以参考 *renderdoc 调试 unity shaderlab* 

&nbsp; &nbsp; &nbsp; &nbsp;renderdoc是不能单步调试GLSL的。。。

![renderdoc cannot debug opengl](http://aicdg.com/assets/img/blogimg/rdc/opengl/Snipaste_2018-08-03_13-13-00.png)

&nbsp; &nbsp; &nbsp; &nbsp;GLSL不支持单步调试，但是实时修改用假彩色图像调试的方法还是支持的。在Pipeline State中点击Fragment Shader然后点Edit。

![opengl fs edit](http://aicdg.com/assets/img/blogimg/rdc/opengl/Snipaste_2018-08-03_13-21-58.png)

&nbsp; &nbsp; &nbsp; &nbsp;修改shader，按refresh或者直接按f5刷新

![refresh glsl](http://aicdg.com/assets/img/blogimg/rdc/opengl/Snipaste_2018-08-03_13-14-36.png)

&nbsp; &nbsp; &nbsp; &nbsp;在Texture viewer中查看

![在Texture viewer中查看](http://aicdg.com/assets/img/blogimg/rdc/opengl/Snipaste_2018-08-03_13-14-46.png)

## renderdoc 分析 vulkan

&nbsp; &nbsp; &nbsp; &nbsp;renderdoc选择应用程序，然后运行。UE4的项目可以命令行参数 `-vulkan` 强制使用Vulkan渲染。

![UE4 vulkan mode](http://aicdg.com/assets/img/blogimg/rdc/vulkan/Snipaste_2018-08-03_13-16-19.png)

&nbsp; &nbsp; &nbsp; &nbsp;renderdoc也不支持spir-v的调试，但是支持将spir-v反编译为glsl方便查看。也就是说vulkan也能使用类似openGL的分析方法。

![](http://aicdg.com/assets/img/blogimg/rdc/vulkan/Snipaste_2018-08-03_13-20-30.png)

![](http://aicdg.com/assets/img/blogimg/rdc/vulkan/Snipaste_2018-08-03_13-19-17.png)

![](http://aicdg.com/assets/img/blogimg/rdc/vulkan/Snipaste_2018-08-03_13-20-02.png)

![](http://aicdg.com/assets/img/blogimg/rdc/vulkan/Snipaste_2018-08-03_13-20-14.png)

# renderdoc 调试 unity shaderlab

&nbsp; &nbsp; &nbsp; &nbsp;之前说过renderdoc支持调试HLSL，unity shaderlab使用的表面上是CG，实质上是HLSL，所以当然可以用renderdoc单步调试unity shaderlab。

> 虽然unity也内置了帧分析器，但是感觉还是不够强大，除了replay功能之外缺失还是太多。

&nbsp; &nbsp; &nbsp; &nbsp;要在unity中启用renderdoc支持，首先要在game窗口上右键，选择 **load renderdoc**

![load renderdoc](http://aicdg.com/assets/img/blogimg/rdc/unity/Snipaste_2018-08-03_10-31-01.png)

&nbsp; &nbsp; &nbsp; &nbsp;之后再game窗口右上角可以看见renderdoc的图标

![renderdoc的图标](http://aicdg.com/assets/img/blogimg/rdc/unity/Snipaste_2018-08-03_10-36-41.png)

&nbsp; &nbsp; &nbsp; &nbsp;点击图标完成抓帧，其中Camera.Render是game窗口的内容

![Camera.Render](http://aicdg.com/assets/img/blogimg/rdc/unity/Snipaste_2018-08-03_10-37-28.png)

&nbsp; &nbsp; &nbsp; &nbsp;定位到具体的drawCall，如果发现texture output上下颠倒可以使用 Flip the texture in the Y axis。

![Flip the texture in the Y axis](http://aicdg.com/assets/img/blogimg/rdc/unity/Snipaste_2018-08-03_10-38-24.png)

> UE4 以 dx 的窗口系为标准，所以 openGL 模式下上下颠倒。Unity则相反，以 openGL 窗口为标准，在 DX 下会上下颠倒。

&nbsp; &nbsp; &nbsp; &nbsp;用鼠标右键在 Texture Viewer 的 Output 中定位到要调试的像素，

![定位到要调试的像素](http://aicdg.com/assets/img/blogimg/rdc/unity/Snipaste_2018-08-03_10-39-28.png)


&nbsp; &nbsp; &nbsp; &nbsp;然后点击 Pixel Context 中的 debug 按钮即可开始愉快的单步调试

![开始调试shaderlab](http://aicdg.com/assets/img/blogimg/rdc/unity/Snipaste_2018-08-03_10-40-41.png)

&nbsp; &nbsp; &nbsp; &nbsp;如果发现 debug 的shader只有汇编(DXBC)没有源代码(HLSL)，这是因为 shaderlab 编译时没有保存调试符号，在shader文件的 `CGPROGRAM` 块中键入：

```c
#pragma enable_d3d11_debug_symbols
```

![保存调试符号](http://aicdg.com/assets/img/blogimg/rdc/unity/Snipaste_2018-08-03_10-50-37.png)


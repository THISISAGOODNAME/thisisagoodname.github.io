---
layout: post
title: "从一个小bug看MSAA depth resolve"
description: "从一个小bug看MSAA depth resolve"
image: ""
date: 2018-07-16 14:37:04
categories: 研究
tags: [研究,UE4]
---

<!-- more -->
* Table of Contents
{:toc}

# 引子，一个小bug

&nbsp; &nbsp; &nbsp; &nbsp;最近策划提了一个奇怪的bug：在开启硬件抗锯齿(MSAA)之后，在雪地上的脚印失效了，无论android还是iOS都不行。这个bug乍看确实不是什么大bug，当时估计就是素材或者引擎的某些配置项有问题。但是，仔细调试之后才发现，事情远没有那么简单。

# UE4的实现

&nbsp; &nbsp; &nbsp; &nbsp;在实际开发中，脚印，喷漆，弹坑这样的效果，都可以使用decals来实现。UE4实现decals的方式，就是通过当前场景的深度信息，直接将decals的instance放置到正确位置。而获取深度信息的方法，一般就是 resolve 深度贴图。

&nbsp; &nbsp; &nbsp; &nbsp;这个奇怪的bug，就是开启MSAA后在获取场景深度时出的问题。而且可以推论，所有使用custom depth功能的效果，开启MSAA后均有问题。然后查阅资料，发现了bug [UE-49851](https://issues.unrealengine.com/issue/UE-49851)。

&nbsp; &nbsp; &nbsp; &nbsp;一般情况下，到此就应该结束了。已知bug，won't fix。但是，策划来了个我要我要我就要。这时候怎么办，只能硬着头皮上，想办法让策划死心。不过在这个过程中，还真的发现了部分修复这个问题的方法。

# 处理 UE4 MSAA depth resolve 的bug

&nbsp; &nbsp; &nbsp; &nbsp;**Warning:** 修改这个问题需要修改 UE4 的 RHI(Rendering Hardware Interface)，这个确实比较麻烦，不仅需要掌握 UE4 的 RHI API，还需要掌握实际的图形API，以及了解RHI和实际图形API之间的映射关系。拿这次来说，不同平台不同API修改都略有出入。而且如同我之前提到了，只有部分情况可解。

## 可解情况

### ios + metal

&nbsp; &nbsp; &nbsp; &nbsp;首先说一下改起来最简单的平台，就是 iOS (metal)。

&nbsp; &nbsp; &nbsp; &nbsp;metal 在 API 层面直接加入了 MSAA depth resolve 的支持，只要在 `MTLRenderPassDepthAttachmentDescriptor` 创建时，在使用 MSAA 的情况下，设置 `MTLMultisampleDepthResolveFilter` 即可。选择 `MTLMultisampleDepthResolveFilterMin` 还是 `MTLMultisampleDepthResolveFilterMax` 要看你的实际情况。

### windows + openGL (feature level es31)

&nbsp; &nbsp; &nbsp; &nbsp;接着说一下次简单的平台，windows上的 openGL(**feature level es31**)。我专门提一下es31，因为在桌面使用 openGL feature level ES31 模拟 android 上的 openGL ES 3.1 是开发阶段的常用形式。

&nbsp; &nbsp; &nbsp; &nbsp;openGL提供了接口 `glBlitFramebuffer` ，用来 resolve color/depth/stencil buffer，支持 MSAA resolve。正确选择贴图格式以及 sample count 就能解决这个问题，改动也不算很大。不过，`glBlitFramebuffer` 接口只在 openGL 3.0+ / openGL ES 3.0+ 上有。 

### android + openGL ES 3.1

&nbsp; &nbsp; &nbsp; &nbsp;android + openGL ES 3.1 是市面上存量最大的平台，也是最难以处理的的平台。openGL ES API 默认情况是不支持 MSAA 的(使用 `glGetIntegerv` 查询 `GL_MAX_SAMPLES`的结果 **总为1**)，我知道你想说，我自己试安卓上可以开 MSAA 啊。那是因为你的手机支持 `GL_EXT_multisampled_render_to_texture` 扩展，只有支持这个扩展，Android上的openGL ES才能开启MSAA。所幸的是这个扩展兼容性极好，几何不存在不支持该扩展的设备。使用 `glGetIntegerv` 查询 `GL_MAX_SAMPLES_EXT`，mali设备上一般是16，Adreno/powerVR 设备上一般是4。

&nbsp; &nbsp; &nbsp; &nbsp;但是，**该扩展只对 color attachment**有效，也就是说，`GL_EXT_multisampled_render_to_texture` 扩展会自动采样并 resolve color attachment (不相信可以使用 renderdoc 或者其他帧调试器在 Android 上抓帧)，对 depth stencil attachment 无效。因为 openGL ES 支持 `GL_MAX_SAMPLES` 的结果 **总为1**，不可能正确的创建 MSAA depth/stencil attachment，自然不可能正确的 resolve。就算 openGL ES 提供了接口 `glBlitFramebuffer` 也没用。

&nbsp; &nbsp; &nbsp; &nbsp;只要思想不滑坡，方法总比问题多。这时候就要看你对 UE4 内置 shader 的掌握程度了。

#### common： 使用 framebuffer fetch 扩展

&nbsp; &nbsp; &nbsp; &nbsp;首先要感谢 Epic 的 shader 的开发人员，实现了一个非常方便非常实用的功能。就是在片段着色器中，在计算场景颜色后将场景颜色赋予了输出 color 的 `rgb` 通道，设置为了颜色，而将输出 color 的 `a` 通道设置为了场景的深度(片段着色器输出的颜色范围是0-1，一般俗称 deviceZ/线性空间深度，场景的深度单位为cm，一般称 sceneZ/场景空间深度)。

&nbsp; &nbsp; &nbsp; &nbsp;只要在渲染 decals 的时候能拿到场景的 color attachment，通过 a 通道换算到场景空间就能准确的投射 decals。但是，如果真的将整个场景线渲染到一张texture上，在渲染 decals 的时候再采样，无论是渲染效率，带宽占用亦或是显存占用，在移动设备上都不可接受。

&nbsp; &nbsp; &nbsp; &nbsp;幸好硬件开发商们知道这些难处，所以贴心的准备了 framebuffer fetch 扩展。这个扩展最通用的是 `GL_EXT_shader_framebuffer_fetch`，之后还有硬件厂商自有实现版，比如 `GL_ARM_shader_framebuffer_fetch`,`GL_APPLE_shader_framebuffer_fetch`,`GL_NV_shader_framebuffer_fetch`。因为支持 `NV` 头和 `APPLE` 头但不支持 `ARM` 或者 `EXT` 头的设备几乎不存在，所以只讨论 `ARM` 和 `EXT` 头。

- 对于 `GL_EXT_shader_framebuffer_fetch` 扩展，在 openGL ES 3.0+ 上，使用变量名为 `gl_FragColor`(对，这个在2.0的时候是内置变量)。在openGL ES 2.0上是 `gl_LastFragData[0]`。
- 对于 `GL_ARM_shader_framebuffer_fetch` 扩展, 使用使用变量名为 `gl_LastFragColorARM`

&nbsp; &nbsp; &nbsp; &nbsp;UE4 在 `GlslBackend.cpp` 中将这两个扩展统一封装成了 `FrameBufferFetchES2()` 函数，在shader中直接用即可。

&nbsp; &nbsp; &nbsp; &nbsp;修改 `Comman.ush` 文件，在 `CaclSceneDepth` 函数中追加一个 `COMPILER_GLSL_ES3_1` 分支，添加 `return FrameBufferFetchES2().w`。同时将 `LookupDeveceZ` 函数的 第一个分支从 `#if COMPILER_GLSL_ES2` 改为 `#if COMPILER_GLSL_ES2 || COMPILER_GLSL_ES3_1` 即可。

> framebuffer fetch 扩展几乎所有支持 openGL ES 3.1 的硬件都支持。UE4也会在 featurelevel es31 对不支持 framebuffer fetch 的设备直接闪退，所以我写是通用方法

#### advance： 使用 GL_ARM_shader_framebuffer_fetch_depth_stencil 扩展

&nbsp; &nbsp; &nbsp; &nbsp;对于某些支持 `GL_ARM_shader_framebuffer_fetch_depth_stencil` 扩展的设备，可以直接使用 `gl_LastFragDepthARM` 拿到当前设备深度，别忘了转换到场景空间。UE4 在 `GlslBackend.cpp` 封装了 `DepthbufferFetchES2()` 函数，不过作用并不是计算深度，而是调整 depth resolve 的结果。适当修改 `Comman.ush` 就可以直接使用 `gl_LastFragDepthARM` 当做场景深度。

> `GL_ARM_shader_framebuffer_fetch_depth_stencil` 扩展依赖于 framebuffer fetch 扩展，并且不是所有 openGL ES 3.1 设备都支持

## 不可解情况

### android + vulkan

&nbsp; &nbsp; &nbsp; &nbsp;vulkan **API层面不支持MSAA depth resolve**

- `vkCmdResolveImage`：只能resolve color attachment
- `vkCmdBlitImage`：不支持multisample(`srcImage`和`dstImage`的`samples`值必须为`VK_SAMPLE_COUNT_1_BIT`)
- `vkCmdCopyImage`：Copy depth/stencil attachment时不支持multisample

> 吐槽一下vulkan的文档，接口函数只写了拷贝depth/stencil attachment时srcImage和dstImage格式必须一致，结果在创建structure里写了depth/stencil attachment不支持multisample，真是绕

&nbsp; &nbsp; &nbsp; &nbsp;而且vulkan不支持 framebuffer fetch 。因此vulkan下 UE4 MSAA 和 custom depth 冲突的问题 **完全无解** !

### osx + metal / windows + openGL3+/dx11+/vulkan (featurelevel sm5)

&nbsp; &nbsp; &nbsp; &nbsp;这个应该很好理解， SM5 下使用延迟渲染而非前向渲染，不使用 MSAA，Epic至今还是把主机和PC平台作为最主要用户，所以 MSAA 和 custom depth 冲突的问题直接标注 won't fix 也可以理解。

# 小结

&nbsp; &nbsp; &nbsp; &nbsp;到此，ios和Android上的 MSAA 和 depth 冲突的问题，部分处理了，但是还不完整。如果有大神能解决 vulkan 上 multisample depth/stencil resolve 的问题，请一定告诉我。

&nbsp; &nbsp; &nbsp; &nbsp;然后是关于vulkan的吐槽，本来以为vulkan设计只是复杂，没想到还会出现功能上的缺失。而且我们私下讨论还说，vulkan的标准制定者一定不怎么做游戏，不然不会缺失一大堆实用功能。反观metal，API层面直接支持msaa depth resolve，framebuffer fetch，depthstencilbuffer fetch，shader storage。。。相当多openGL 扩展被直接写到了metal作为标准的一部分。这才是天天用的人设计的API。




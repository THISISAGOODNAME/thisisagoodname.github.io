---
layout: post
title: "UNREAL ENGINE 4神坑 - r.mobileHDR"
description: "UNREAL ENGINE 4神坑 - r.mobileHDR"
category: UE4
tags: [UE4]
---

&nbsp; &nbsp; &nbsp; &nbsp;前面两周时间，一直在研究UNREAL ENGINE 4的preInit逻辑。讲真，我上周一周编译UE4的次数，比过去几年我打开UE4的次数都多。

<!-- more -->

* Table of Contents
{:toc}

# UE4 preInit几个神坑

1. **isCached** - 看到函数的名字，再加上返回值是布尔型，是不是以为这是检查资源是否被缓存的函数。告诉你，错，这个函数的作用是检查资源是否被缓存，已缓存就返回true；未缓存则将资源立即缓存，再返回true。它返回false的情况，只发生在立即缓存出错时。
2. **openGLDrv** - 这个只是我想吐槽一下，UE在桌面系统使用openGL 3.2，如果在桌面系统模拟GLES2.0和GLES3.1，则使用GLSL\_150\_ES20和GLSL\_150\_ES31。要知道，UE4最低要求GTX470，如果不是mac的话妥妥支持openGL4.6啊，考虑mac其实openGL4.1也绝对OK(其实4.18版本，mac版最低要求支持Metal 1.1的显卡)
2. **r.mobileHDR** - mobileHDR是神坑最多的，文章接下来的部分，就是我这两周对于这个变量的一些研究。

# UE4的post effect

&nbsp; &nbsp; &nbsp; &nbsp;我非常非常不理解EPIC对于post effect的设计。在UE4中，所有的post effect，基本都依赖于HDR。简单的说，如果你关闭了HDR选项，那么所有的post effect都将不能生效。当然，读了UE4的shader其实也能发现，HDR和LDR在很多实现上是完全不同的，甚至色彩空间都不同。HDR使用Linaer64空间，而LDR使用Gamma32空间，Gamma32中的颜色不是线性的，当然做过Gamma校正的人一定知道我在说啥。

# r.mobileHDR

&nbsp; &nbsp; &nbsp; &nbsp;看到post effect中HDR的作用，想必做手游的同行们应该有了一个大胆的想法吧。在低端机上关闭HDR，在高端机开启HDR。其实Unreal Engine 4的优化真心屌的可以，在Galaxy Nexus这种老设备上，关闭HDR的情况下，跑UE4的mobile demo还有50FPS。但是，相信同行们也试过了，就是mobileHDR并不能在运行时修改。相信原因大家都猜到了，这个变量和烘焙有关。

# UE4 资源加载过程

&nbsp; &nbsp; &nbsp; &nbsp;UE4有一个preInit的过程，这步会加载引擎的配置文件，比如Engine.ini，Game.Ini，DeviceProfile.Ini。在加载完几个ini之后，就是执行预缓存，将引擎内置的各种素材缓存，比如EngineMaterials。UE4初始化过程并不会加载你的游戏assets，他们是更靠后使用streamServer下载来的。不过，不用考虑那么靠后的情况，因为，如果你的DeviceProfile.ini中为各个平台设置了r.mobileHDR，而他们恰好和Engine.ini中的r.mobileHDR不同，那么你加载EngineDefaultMaterials的时候应该就挂了，不是WorldGridMaterial，就是DefaultDeferredDecalMaterial。

# UE4 烘焙后的Material的uasset文件

&nbsp; &nbsp; &nbsp; &nbsp;<span style="color:red">更正</span>：Unreal的材质文件不包含贴图，真的是纯shader就大的不行。在移动设备上内存和包体大小都比较吃紧，因此使用各种压缩材质分离材质是必须的，即便可能会使加载时间变长等新问题。

&nbsp; &nbsp; &nbsp; &nbsp;烘焙后，在content目录下会出现几个shader的bin文件，比如DefaultGalobalShader\_PCD3D\_SM5.bin和DefaultGalobalShader\_OPENGL\_ES31.bin。不过这个文件并不是真的shader，只是个DefaultShader的索引。真正的shader其实还是在每个Material里面。UE4的Material文件，就是贴图+各个renderpass的usf文件，usf是Unreal Engine文本格式是shader代码，大体上就是HLSL，编译器使用的是Epic自己维护的HLSLcc，现在微软开源了[DirectXShaderCompiler](https://github.com/Microsoft/DirectXShaderCompiler)，而且还添加了[SPIR-V的支持](https://github.com/Microsoft/DirectXShaderCompiler/wiki/SPIR%E2%80%90V-CodeGen)，也许Epic会陆续替换成微软的编译器吧，毕竟SPIR-V在openGL 4.5是ARB扩展，openGL4.6和vulkan可以直接使用，而且SPIR-V生成GLSL和ESSL也非常方便。
 
&nbsp; &nbsp; &nbsp; &nbsp;回到正题，Material文件包含贴图信息和shader信息，也就是材质独立，这个我和同事确实有不同观点，我个人比较赞同这种做法，这样材质之间相互独立，复用性强。(substance的sbs材质也是这种组织形式)但是他们则认为shader和贴图独立更好，毕竟shader和贴图都有可能复用。也许是因为我只会GLSL吧，没有DX(HLSL)开发者写shader时的模块化意识。

&nbsp; &nbsp; &nbsp; &nbsp;回到r.mobileHDR的议题。Unreal Engine在preInit renderer设备的时候(initRHI)，在移动平台，会根据mobileHDR是否开启，来选择使用shader是Linear64还是Gamma32色彩空间。之后，就会在材质uasset文件中找正确的shader来使用。然后，烘焙的坑来了。比较开启r.mobileHDR和不开启r.mobileHDR烘焙出的材质，可以看到，材质里会有和你选择导出的platform相同数量的shader，拿TMobile#lightPolice#MaxLightCout[Linear64\|Gamma32]举例，这个材质只存在一对VS和PS，以Linear64结尾或者Gamma32结尾。也就是说烘焙的过程，只烘焙了LDR或者HDR一种素材。

# 运行时可选mobileHDR的Work around

&nbsp; &nbsp; &nbsp; &nbsp;有了上述的理论基础，其实可以很简单的进行一种work around的方法。就是烘焙两次，然后运行时动态选择是加载LDR的asset还是HDR的asset。通过我个人的实验，这种方法是可行的，只要记得Engine内置asset和你个人的asset都替换即可。内置Material出错会直接崩溃。自己的Material出错不会崩溃，但是会使用系统内置的Material，就是灰黑色的那个。

&nbsp; &nbsp; &nbsp; &nbsp;这种方法当然非常辣鸡，因为烘焙资源两份，基本是不能接受的。对于手机游戏，安装包过大会直接影响用户下载的欲望。

# 更好的解决方法

&nbsp; &nbsp; &nbsp; &nbsp;更好的解决方法，相信大家都猜到了，就是修改烘焙的逻辑，让Linear64和Gamma32两种空间的shader都打到Material里。shader其实不算大，大的是材质文件里的贴图。上述work around方法LDR和HDR的Material中其实存放了相同的贴图，所以冗余较大。我从上周五开始，已经在研究UE4资源烘焙的逻辑了，希望这个猜想是可行的。不过考虑到Epic对于mobileHDR的态度，Epic应该是不可能接纳这个patch的，我也许会放在我个人的GitHub上，也许这个patch就真的随历史自生自灭吧，毕竟HDR和几个post effect都跑不起来的低端机的保有量也不大了。也许真的在Epic眼里，如果连`r.MobileContentScaleFactor`和`r.MaterialQualityLevel`都优化不了的设备，都应该直接丢垃圾桶算了。

# 补充：UE4的feature level

&nbsp; &nbsp; &nbsp; &nbsp;UE4有几个feature level，分别是ES20，ES31，SM4，SM5。这几个feature level只表示renderer使用的硬件特性，和API无关。举个例子，openGL ES 2.0在补充VAO、texture_float、buffer_block、instanced_arrays、TransformFeedback等扩展之后，就可以提供ES31的feature。Vulkan在Android上使用ES31，在桌面和NVIDIA tegra设备上使用SM5。Metal在iPhone上使用ES31，在ipad pro上使用SM5。

# 补充：UE官方的移动设备优化指南

&nbsp; &nbsp; &nbsp; &nbsp;[Performance Guidelines for Mobile Devices](https://docs.unrealengine.com/latest/INT/Platforms/Mobile/Performance/)

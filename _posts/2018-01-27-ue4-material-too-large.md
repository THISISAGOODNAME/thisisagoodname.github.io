---
layout: post
title: "UE4内置material size优化"
description: "UE4内置material size优化"
category: UE4
tags: [UE4]
---

&nbsp; &nbsp; &nbsp; &nbsp;在成功实现了同时烘焙LDR和HDR两种资源之后，就必须面对一个很棘手的情况了 - UE4烘焙出来的材质占用容量太大了。

<!-- more -->

&nbsp; &nbsp; &nbsp; &nbsp;相信很多刚用UE4的porn友，都被UE4烘焙时超慢的速度和烘焙后超大的包体所震撼。在我实现了同时烘焙HDR和LDR的两种材质的功能之后，包体体积再度变大，大到了老大很不爽。那就得优化啊。

&nbsp; &nbsp; &nbsp; &nbsp;UE4通过修改ConsoleVariables.ini中的r.DumpShaderDebugInfo变量，可以将烘焙过程中的shader的源文件以及编译结果导出。对于feature level ES20和ES31，编译的结果自然是GLSL。相信dump过结果的porn友们应该都对Unreal的usf文件不太爽，因为200kb的usf文件，编译成glsl可能只有十几kb，而uassets存的还是ushaderbytecode，还会根据光照方式，动态光源个数，LDR/HDR，skylight等，将烘焙的结果不断的x2，所以烘焙出的材质就很大了。

# Opti 1: bShareMaterialShaderCode

&nbsp; &nbsp; &nbsp; &nbsp;优化1：各材质的shader不再用inline的方式处理依赖，而且继续使用include的形式。启用这个选项，需要在UE4 editor的项目属性中，修改Packaging下的bShareMaterialShaderCode变量，开启，就可以使材质共用通用部分，而不再使用函数inline的方式独立。开启此项之后，我原来2.02G的包体，变成了1.63G，小了接近400m，非常明显。

# Opti 1: bSharedMaterialNativeLibraries

&nbsp; &nbsp; &nbsp; &nbsp;优化2：将材质shader以各平台最终结果的形式保存，而不以UE4的中间代码保存。。启用这个选项，需要在UE4 editor的项目属性中，修改Packaging下的bSharedMaterialNativeLibraries变量，开启。这个属性在UE 4.16版本中貌似只对metal平台生效，而在UE 4.18版本中，需要在启用bShareMaterialShaderCode的情况下，才能启用。使用此方法之后，包体体积将进一步减少。

# 一些废话

&nbsp; &nbsp; &nbsp; &nbsp;谈一谈UE4在桌面和移动平台优化的问题吧。经过接近三个星期对UE4代码的阅读，渐渐发现UE4代码在优化思路上的一些问题。UE4作为优秀的PC和console引擎，对于PC和主机平台确实做了很多优化。但在进行移动开发的时候，这些优化可能就变为了负优化。一个比较明显的情况，就是上面对于材质的处理情况。将各个材质完全独立开来，毋庸置疑在材质cache之后，能加快材质渲染时的效率。但是，这样会增大包体体积，也会增大内存占用。对于我们现在的测试包来说，差不多是400m的安装包容量和50m的内存占用，这两点对于移动平台开发，貌似都有点严重了。看来UE4要在移动平台使用，那要做的优化还有不少。

&nbsp; &nbsp; &nbsp; &nbsp;压缩会减慢程序运行。这个是很多新人(当然也包括我)，都犯的一个错误。就是想当然了。实际上很多时候，压缩不仅不会影响程序运行速度，甚至会提升程序运行的速度。压缩之后会便随解压，解压会让程序运行时间变长，所以压缩程序会让程序变慢，这个是一般思路。但是实际程序运行时，包体压缩，会减小数据加载的时间，也就是减少IO的时间。CPU运行程序解压，总比加载文件快多了(除了使用很变态的压缩算法)。这也是移动GPU上使用贴图压缩能让渲染效率提升的原因。
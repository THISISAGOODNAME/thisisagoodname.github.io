---
layout: post
title: "超越openGL状态机-DSA"
description: "超越openGL状态机-DSA"
category: c++
tags: [c++,图形学,OpenGL]
---

&nbsp; &nbsp; &nbsp; &nbsp;OpenGL的历史，是坎坷的。从业界领先，专业领域唯一选择，到成为DX的坚定追随者，真的经历了很多，而这个过程中，openGL的状态机模型，一直是一个很有争议的设计(而且，DSA扩展进入Core/ARB，以及vulkan的编程模型，基本上也算是官方否认了openGL状态机模型)。今天我就来写一写厂商们对openGL状态机的抗争。

<!-- more -->

* Table of Contents
{:toc}

# openGL的几个经常被误解的特点

&nbsp; &nbsp; &nbsp; &nbsp;我不会列举openGL全部特点，我在这里只列举两个初学者经常误解的特点，这两点很多openGL初学伪忠经常举错例子，所以我专门写一下

- openGL 状态机模型
- openGL C/S模型

&nbsp; &nbsp; &nbsp; &nbsp;先说说openGL状态机模型，这个不仅仅是openGL粉，很多伪开源粉微软黑也经常拿来说事，认为这个是openGL高效的根源，经常攻击微软的无状态。早期DX运行效率确实不如openGL，但是原因肯定和状态机无关，几个可能的原因，

1. openGL是纯C的，DX是C++的，在那个年代C++就是比C慢一些，在低端硬件上更是明显
2. openGL使用ID标记对象，而DX使用互斥的句柄。当然使用句柄对开发者更为友好，一堆GLuint命名不好我都不知道干啥的。win32 HWND相比pointer会额外消耗少量性能
3. 显卡厂商的锅，驱动没到家。openGL硬件不行有软件rollback(仅限固定管线时代)，DX嘛。。。而且那时候硬件也没多行，巫毒解个VCD就顶天了，很多时候还是得求助于3DNow!和SSE。。。

&nbsp; &nbsp; &nbsp; &nbsp;关于状态机很多新人可能还有一个误区，就是总将openGL状态机和图形渲染管线弄混。现在给你们一个好记的方法区分。图形渲染管线，是每次drawCall后运行的过程。而状态机的意思就是必须先达成某个状态才能进行下一步操作。比如GenBuffer就必须在BindBuffer之前，反了就有问题。必须各个buffer都准备完成，才能执行drawCall，不然就。。。

&nbsp; &nbsp; &nbsp; &nbsp;关于C/S模型，这个有些人可能会觉得，这样设计会降低效率。不过，事实上，这才是openGL，或者说是所有显卡/声卡/计算卡/FPGA/协处理器高效的根源。C/S是一种异步执行模型，确实会降低同步操作的效率，但是对于openGL这种大部分API都是无返回值的函数，异步显然效率更高。网络通信慢和不可靠导致的问题，C/S表示这锅我不背。

# 叛逆者NVIDIA，和GL\_EXT\_direct\_state\_access

&nbsp; &nbsp; &nbsp; &nbsp;我直接饮用龚敏敏大神的一段话：

## 状态机的罪恶

&nbsp; &nbsp; &nbsp; &nbsp;OpenGL之前一直采用状态机的形式，比如要给buffer填数据，就得

```c
glBindBuffer(target, buffer);
glBufferData(target, size, data, usage);
```

&nbsp; &nbsp; &nbsp; &nbsp;除了bind系列函数，其他函数都是操作当前设置在状态机里的对象。这么做API简单，但缺点也非常明显。

1. 状态机是全局的。你的一次bind，可能造成十万八千里外的代码操作错了对象。
2. 性能。为了不影响状态机，很多时候你需要glGet旧的，glBind新的，操作完在glBind旧的回去。而且在驱动里，还得把它还原成非状态机的形式。

&nbsp; &nbsp; &nbsp; &nbsp;在OpenGL 2.0的时候，曾经有传言要废除旧的状态机，换用一套新的操作方法。然而最后也不了了之。一些新的API，比如fbo，用的是混合的方法。主流API还是老的状态机模式。

## EXT DSA

&nbsp; &nbsp; &nbsp; &nbsp;2008年，GL\_EXT\_direct\_state\_access诞生了，然后经过多次修修补补(一共39次修改)，在2014.2.24，达到最终版本，v1.2。这个多次的修修补补，和openGL演进也有一些关系。openGL 1.x一直的固定管线，在openGL 1.5的时候，shader终于成为ARB扩展，GLSL 1.0也走上历史舞台。openGL 2.0正式加入shader，glsl 1.1同期登场。之后使用可以说到现在为止都是最广泛的openGL版本，openGL 2.1，带着glsl 1.2登场了。我们的主角，GL\_EXT\_direct\_state\_access扩展，也随着openGL 2.1的发布，发布了第一个正式版，v1.0

&nbsp; &nbsp; &nbsp; &nbsp;GL\_EXT\_direct\_state\_access是一个超级长的spec，并且在之后的七八年里不停地修改。这个扩展几乎提供了所有OpenGL函数的一个新版本，不需要状态机就能直接操作对象。比如前面说的buffer填数据，就可以一行搞定

```c
glNamedBufferDataEXT(buffer, size, data, usage);
```

&nbsp; &nbsp; &nbsp; &nbsp;由于没有状态机的设置，这里的调用不用担心破坏别处的代码，性能也可以保证不降低。这样对开发效率和运行效率都有好处。

# Core/ARB DSA

&nbsp; &nbsp; &nbsp; &nbsp;到了OpenGL 4.5的时代，终于，终于，终于，DSA被升级到ARB扩展，并合并到核心。基本上就是把EXT版本的可编程流水线系列函数提升到Core/ARB。这下终于可以愉快地使用DSA了！

&nbsp; &nbsp; &nbsp; &nbsp;不过要注意的是，有一些细节还是不同的。比如原先用glGen\*系列函数生成的id，内部并没有初始化那个对象的状态。只有到了glBind\*的时候才会初始化。而Core/ARB的DSA直接提供了glCreate\*系列函数，可以一步到位地建立id和初始化。很多Core/ARB的DSA函数，都需要配合glCreate\*来使用，不能用于glGen\*的。

# 附录A：OpenGL与NVIDIA

&nbsp; &nbsp; &nbsp; &nbsp;OpenGL的发展与NVIDIA真的有很多关系，虽然作为ARB成员，为openGL贡献自然是少不了的。比如openGL中最早的shader就是NVIDIA带来的，我说的不是CG，是ARB\_fragment\_program，当然这货不是给开发者用的，只是用来化简驱动的复杂程度，用脚本来代替状态机中无数的标记。这个shader是这么写的：

```assembly
!!ARBfp1.0 MOV result.color, fragment.color; END
```

&nbsp; &nbsp; &nbsp; &nbsp;当然CG也是重要贡献就是了，NV和MS的某些PY交易达成了这货。这时候还没VretexBuffer更没VAO。顶点还是`glBegin`和`glEnd`直接用`glVertex`插入了，对于现在很多入行就学可编程管线的人来说也许真的不可理喻吧。

&nbsp; &nbsp; &nbsp; &nbsp;GL\_NV\_path\_rendering和GL\_EXT\_direct\_state\_access当然也是NV的杰作。整个过程，openGL API/Extensions的发展方向，基本围绕3个词，性能增强，开发者控制能力增强，API减少。

# 附录B：How to make OpenGL usage Vulkan like

&nbsp; &nbsp; &nbsp; &nbsp;通过一些高版本的openGL扩展的使用，可以让openGL的开发非常vulkan like(不光开发起来像，性能当然也是有提高的了)，[原文链接](https://developer.nvidia.com/opengl-vulkan)


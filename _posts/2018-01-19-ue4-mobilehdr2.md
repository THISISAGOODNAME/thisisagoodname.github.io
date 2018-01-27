---
layout: post
title: "UNREAL ENGINE 4神坑 - r.mobileHDR(续)"
description: "UNREAL ENGINE 4神坑 - r.mobileHDR(续)"
category: UE4
tags: [UE4]
---

&nbsp; &nbsp; &nbsp; &nbsp;在上一周，我终于成功，至少看起来成功的解决了之前提到的UE4的r.MobileHDR变量修改的问题。成功解决之后更是倍感蛋疼，因为我之前确实数次和正确的答案擦肩而过，只是因为不相信，而和他失之交臂。

<!-- more -->

* Table of Contents
{:toc}

# 补充: UE4 preInit几个神坑

&nbsp; &nbsp; &nbsp; &nbsp;在上篇blog中已经记录了preInit的几个神坑，纠正一下，`isCached`没有问题，有问题的是`shouldCache`，它判断shader应该被缓存之后，就会顺手把shader cache住。

&nbsp; &nbsp; &nbsp; &nbsp;UE4还有一个坑，不光在perInit中有，在全部地方都有使用。而且这个特性绝大多数编程语言都有，实际上使用很广，就是短路求值。短路求值这个特性一说大家都明白，对于`&&`如果左值为false，或者`||`的左值为true，就会跳过右边的判断。一般情况下，这也没啥。但如果右侧的判断，是我上面提到的`shouldCache`，这个代码的逻辑就会变得非常微妙。并且，事实上，很多情况下UE4直接执行短路执行右侧的条件是直接崩溃的，不过，相比调试这个坑爹的`shouldCache`逻辑，我倒是更希望他直接崩溃。

# (看起来是)移动平台超越mobileHDR限制烘焙LDR和HDR材质的方法

&nbsp; &nbsp; &nbsp; &nbsp;<span style="color: red">**注意**</span>：我写了看起来是，因为这种方法也许有很多想想就知和不可预知的问题。

&nbsp; &nbsp; &nbsp; &nbsp;这个问题的一个答案，简单的超乎寻常，简单的我自己都不信，简单的让我认为我的C++水平就是一坨屎(事实上也是)。

&nbsp; &nbsp; &nbsp; &nbsp;`/Engine/Source/Runtime/Renderer/Private/MobileBasePassRendering.h`，引擎源码的这个文件的`ShouldCacheShaderByPlatformAndOutputFormat`，将它的返回值改为true写死。这样烘焙出来的material.uasset就同时含有LDR和HDR的shader了。

&nbsp; &nbsp; &nbsp; &nbsp;不过，再次提醒一下，这么做有很多一直和未知的错误，希望大家谨慎使用。考虑到EPIC官方绝对不会修复这个问题，也希望诸位大佬能协助小弟排查一下这样的后果和代价。
---
layout: post
title: "结：bullet物理引擎的一些优点和遗憾"
description: "结：bullet物理引擎的一些优点和遗憾"
category: html5
tags: [html5,bullet]
---

&#160; &#160; &#160; &#160;本文主要说bullet引擎的一些应用，还有优点和一下我眼中一些缺点或者说遗憾吧。

<!-- more -->

* Table of Contents
{:toc}

# 在动画上的应用

&#160; &#160; &#160; &#160;就拿MMD中的物理仿真举例吧

![MMD中的bullet实现方式](http://7xqrar.com1.z0.glb.clouddn.com/ammojsQQ%E6%88%AA%E5%9B%BE20170225162652.png)

网页版可以看[这里](http://aicdg.com/learn-ammojs/DemoExtra-mmd%20demo.html)，需要稍等一下，加载需要一定时间。MMD模型实现物理的方式就这么简单，裙子和发片就是方块，头发则是很多圆柱用铰链链接。

# 优点

1. 开源(zlib协议商业友好)
2. 速度不错(比ODE好很多)
3. 支持GPU加速(bullet3支持)
4. 调试(bullet自带btIDebugDraw接口，实现btIDebugDraw，最简单情况下只要实现drawlines方法，就能使用world->debugDrawWorld()方法显示)

# 缺点

## 文档

&#160; &#160; &#160; &#160;bullet物理引擎的文档写的说实话确实不太好，大部分类的属性方法还有接口方法的作用以及各个参数的作用描述都不是很详细，就算阅读源代码的注释也非常少对于初学者非常不友好。对于想学习物理引擎开发的朋友，建议还是去看ODE，设计还是非常优秀的；当然如果目标是webgl平台，cannon.js也很值得学习，全部源码也才200来k，代码可读性也非常好，而且作者编程习惯很好，每个函数每个属性基本都写了注释。

## 教程

&#160; &#160; &#160; &#160;市面上bullet的教程确实也不太多。虽然bullet自带非常棒的examples，但是对于c++初学者和中手来说，都是有点费劲的。因为文档缺乏的关系，能把examples下的OpenGLWindow研究明白确实要费点功夫，当然对于一些opengl学习了一段时间的朋友，这个确实有研究的价值，毕竟实际开发游戏的时候不能带着glut/glfw，略low啊。
(不过其实想想以前上学的时候，就是Qt+boost+meshlab打天下，现在看不起glut/glfw也不太好)

## 性能

&#160; &#160; &#160; &#160;bullet物理引擎的性能虽然可以接受，但是总被physx和havok拖出来吊打，虽然要承认存在支持GPU加速的bullet，早年只支持openCL，现在CUDA版本也完成了，但是用户并不多。使用bullet的特效和动画软件居多，渲染动画时物理引擎的性能不是渲染帧的瓶颈(光源和材质才是)，所以某些软件(MMD)并没有升级bullet到bullet3的动力。

## 功能

&#160; &#160; &#160; &#160;bullet是一个优秀的物理引擎，严格来说是优秀的动力学引擎。bullet相比业界应用更广泛的phsyx/havok，缺少非常重要的一个部分，就是破坏引擎。bullet没有内置的模型破坏引擎，需要从外部引入，比如[这种](http://aicdg.com/learn-ammojs/Demo7-convex%20break.html),讲真假的不行，也就无法独自完成木板/砖墙破碎这种表现力极强的画面，。火爆场面需求越来越高的今天，bullet被游戏厂商们唾弃也不是不能理解。

&#160; &#160; &#160; &#160;当然提到破坏引擎还是要说一下，2010年，为了狙击NV的phsyx，AMD开启了开放物理计划，这个很多人其实都知道，就是让bullet和havok支持利用opencl进行GPU加速；但是好多人不知道，AMD那时还资助了一家叫Pixelux的公司，Pixelux旗下的Digital Molecular Matter(DMM)是当时优秀的破坏特效软件，星球大战就使用的该软件。AMD资助下的Pixelux，把DMM的新版本DMM2直接**开源**了，AMD的工程师还直接为bullet编写了SPH(光滑粒子流体动力学，主要用来模拟液体)模块，AMD对于bullet其实功不可没。至此为止，bullet+DMM+tressFX，相比phsyx和havok，才有勉强叫板的实力(其实DMM还是没有phsyx Destruction效果好，AMD提供的SPH效果远远不如phsyx Flex，bullet的CreatePatch相比phsyx clothing是天上地下，也就AMD的TressFX在毛发模拟上相比phsyx HairWorks能扳回一局)

## 展望

&#160; &#160; &#160; &#160;其实bullet引擎的未来个人不太看好，特别是NVIDIA把phsyx开源之后，性能功能官方支持文档水平应用广泛程度均完败，最后的有点开源还一半一半了。phsyx还有杀器phsyx APEX，能够方便美术人员快速开发各种物理效果。如果是给新人提意见的话，其实我也是更建议学习phsyx而非bullet。

---
layout: post
title: "GLSL初学者常用shader整理"
description: "GLSL初学者常用shader整理"
category: shader
tags: [CG,图形学,shader,OepnGL,webGL]
---

&nbsp; &nbsp; &nbsp; &nbsp;上周还是上上周，膜拜某位dalao指点江山。谈了一句关于shader的问题。我当晚就想了一下，shadertoy等主流shader sandbox确实有很多牛逼的demo，但是面向初学者的却不太多。所以我总结了一些初学者常用的demo，涵盖基础光照，NPR，基于过程渲染，图像混合，图像处理等内容，方便初学者查阅。在线观看地址[http://aicdg.com/GLSLbasic/](http://aicdg.com/GLSLbasic/)

<!-- more -->

# 一些效果图吧

![pic1](http://7xqrar.com1.z0.glb.clouddn.com/GLSLBasic/QQ20171029-231318@2x.png)

![pic2](http://7xqrar.com1.z0.glb.clouddn.com/GLSLBasic/QQ20171029-231303@2x.png)

![pic3](http://7xqrar.com1.z0.glb.clouddn.com/GLSLBasic/QQ20171029-231348@2x.png)

# 关于该shader sandbox的一些说明：

## 致谢

&nbsp; &nbsp; &nbsp; &nbsp;该沙箱基于babylon.js CYOS开发，感谢babylon.js开发者提供如此好用的工具(没有选用three.js是因为three.js对于顶点着色器自定义不太方便，mrdoob编写的[glsl_sandbox](http://mrdoob.com/projects/glsl_sandbox/)，shadertoy等都是略过顶点着色器，只能定制片元着色器)。

## 内置变量

### attributes

1. `vec3` position
2. `vec3` normal
3. `vec2` uv

### uniforms

1. `mat4` world
2. `mat4` worldView
3. `mat4` worldViewProjection
4. `mat4` view
5. `mat4` projection
6. `sampler2D` textureSampler
7. `sampler2D` refSampler
8. `float` time
9. `vec3` cameraPosition
10. `sampler2D` picSampler(for post effect)

> 进行图像处理时，请将模型选择为Plane(post effect)，用来处理的示例图片则是`sampler2D` picSampler

# 一些闲话

&nbsp; &nbsp; &nbsp; &nbsp;在shader开发时，有些人可能会比较轻视顶点着色器，而特别重视片段着色器。其实，顶点着色器也能干大事。比如我编写的逐像素光照demo，估计应该和你在别处看的区别比较大，一般人应该顶点着色器东西很少，片段着色器特别多。而我把光线和视线的半向量计算放在顶点着色器，这样就可以降低片元着色器的压力。实际开发时，顶点着色器计算量远小于片元着色器，在移动设备上将某些重复性计算转译到顶点着色器，对于改变频率很低的变量甚至应该直接用CPU计算再以uniform变量形式传入。

&nbsp; &nbsp; &nbsp; &nbsp;如果对于自制shader还有兴趣，推荐一个网站[http://stack.gl/](http://stack.gl/)，无论新手老手，都能找到很多shader相关的东西。特别是他们开发的[glslify](https://github.com/glslify/glslify)，简直神器。
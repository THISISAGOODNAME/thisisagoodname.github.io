---
layout: post
title: "D3D坐标系统"
description: ""
image: ""
date: 2018-12-01 19:20:24
categories: 研究
tags: [研究]
---

<!-- more -->
* Table of Contents
{:toc}

# 引子

&nbsp; &nbsp; &nbsp; &nbsp;最近做类似文明的那种六边形地形生成系统的时候，遇到了两个相邻地块(位于两个不同的chunk上)的接缝处不连续的问题，然后就发现了高度图像素采样时的问题，借此机会复习一下D3D的uv坐标。

# Pixel Coordinate System

## Pixel Coordinate System for Direct3D 10

The pixel coordinate system in Direct3D 10 defines the origin of a render target at the upper-left corner. as shown in the following diagram. Pixel centers are offset by (0.5f,0.5f) from integer locations.

![d3d10-coordspix10.png](http://aicdg.com/assets/img/blogimg/CoordinateSystems/d3d10-coordspix10.png)

> OpenGL/Metal/Vulkan is the same.

## Pixel Coordinate System for Direct3D 9

For reference, here is the pixel coordinate system for Direct3D 9, which defined the origin or a render target as the center of the upper-left pixel, (0.5,0.5) away from the upper left corner, as shown in the following diagram. In Direct3D 9, pixel centers are at integer locations.

![d3d10-coordspix9.png](http://aicdg.com/assets/img/blogimg/CoordinateSystems/d3d10-coordspix9.png)

# Texel Coordinate System

The texel coordinate system has its origin at the top-left corner of the texture, as shown in the following diagram. This makes rendering screen-aligned textures trivial (in Direct3D 10), as the pixel coordinate system is aligned with the texel coordinate system.

![d3d10-coordstex10.png](http://aicdg.com/assets/img/blogimg/CoordinateSystems/d3d10-coordstex10.png)

Texture coordinates are represented with either a normalized or a scaled number; each texture coordinate is mapped to a specific texel as follows:

For a normalized coordinate:

- Point sampling: **Texel # = floor(U * Width)**
- Linear sampling: **Left Texel # = floor(U * Width)**, **Right Texel # = Left Texel # + 1**

For a scaled coordinate:

- Point sampling: **Texel # = floor(U)**
- Linear sampling: **Left Texel # = floor(U - 0.5)**, **Right Texel # = Left Texel # + 1**

Where the width, is the width of the texture (in texels).

Texture address wrapping occurs after the texel location is computed.

> Axis V is opposite to OpenGL/Metal/Vulkan.

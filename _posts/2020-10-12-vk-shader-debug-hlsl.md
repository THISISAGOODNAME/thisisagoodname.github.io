---
layout: post
title: "使用dxc，在vulkan下单步调试hlsl"
description: "使用dxc，在vulkan下单步调试hlsl"
image: ""
date: 2020-10-12 23:31:20
categories: vulkan
tags: [vulkan]
---
<!-- more -->
* Table of Contents
{:toc}

# dxc编译hlsl生成spirv

&nbsp; &nbsp; &nbsp; &nbsp;使用一个最简单的绘制三角形的hlsl

```c++
struct VSInput
{
	float2 Position : POSITION;
	float3 Color : COLOR;
};

struct VSOutput
{
	float4 Position : SV_POSITION;
	float3 Color : COLOR;
};

VSOutput vert(VSInput vin)
{
	VSOutput vout = (VSOutput)0;

	vout.Position = float4(vin.Position, 0.0, 1.0);
	vout.Color = vin.Color;

	return vout;
}

float4 frag(VSOutput pin) : SV_TARGET
{
	return float4(pin.Color, 1.0);
}
```

&nbsp; &nbsp; &nbsp; &nbsp;正常情况下，如果编译VS和PS生成spirv，需要执行如下的命令

```bash
dxc -spirv -T vs_6_0 -E vert hlsl/shader.hlsl -Fo vert.spv
dxc -spirv -T ps_6_0 -E frag hlsl/shader.hlsl -Fo frag.spv
```

&nbsp; &nbsp; &nbsp; &nbsp;但是这种方法生成的spirv不包含调试信息，在shader单步调试时，只能调试spirv-cross生成的glsl，虽然已经很不错了，但是很多变量都转义了，函数也会inline，影响判断

&nbsp; &nbsp; &nbsp; &nbsp;但是，dxc允许生成调试符号，在dx12 shader debug时额外有效，抱着试试的心态，我在vulkan下试了一下，结果居然可以生成带hlsl源文件信息的spirv，dxc加调试符号的命令为`-Zi`

```bash
dxc -spirv -T vs_6_0 -E vert hlsl/shader.hlsl -Fo vert.spv -Zi
dxc -spirv -T ps_6_0 -E frag hlsl/shader.hlsl -Fo frag.spv -Zi
```

# renderdoc在vulkan下调试hlsl的截屏

![spv debug](http://aicdg.com/assets/img/blogimg、vulkan、shaderdebugHLSL/Snipaste_2020-10-12_23-33-52.png)

![hlsl debug](http://aicdg.com/assets/img/blogimg、vulkan、shaderdebugHLSL/Snipaste_2020-10-12_23-34-21.png)

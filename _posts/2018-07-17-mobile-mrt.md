---
layout: post
title: "移动平台延迟渲染"
description: "移动平台延迟渲染"
image: ""
date: 2018-07-17 10:37:04
categories: 研究
tags: [研究,]
---

<!-- more -->
* Table of Contents
{:toc}

# 回顾openGL ES 3.0

&nbsp; &nbsp; &nbsp; &nbsp;要说openGL ES 3.0相比openGL ES 2.0的最重大进步之处，我个人认为有如下几个：

- VAO/UBO
- Instanced Rendering
- Tranform feedback
- Multiple Render Target 

> 当然SSO,Immutable Texture Storage,half-float color,VTF等特性也都很重要

&nbsp; &nbsp; &nbsp; &nbsp;而在这些新特性之中，MRT可以说是最重要的特性，因为它可以完全改变渲染的方式，产生新的可能性。而其他API一般是性能提升/用法改进，没有MRT的革命性。

# Deferred Shading Using Multiple Render Targets

&nbsp; &nbsp; &nbsp; &nbsp;MRT，可以在一个drawCall向多张贴图进行渲染(至少为4)，MRT最主要的用途，就是实现各种上级渲染，比如Deferred Shading，forward+ Shading，Deferred+ Shading。

&nbsp; &nbsp; &nbsp; &nbsp;延迟渲染一般分三个阶段：

1. Geometry State：几何阶段收集场景的color，normal，depth，albedo信息
2. Lighting Stage：估计每个像素影响的光照信息
3. Shading State：依靠光源信息来计算场景最终的颜色

![延迟渲染](http://7xqrar.com1.z0.glb.clouddn.com/TIM20180717114933.png)

&nbsp; &nbsp; &nbsp; &nbsp;几何阶段代码实现

```c++
// Define 4 color attachments for currently bound framebuffer
GLenum renderbuffers[] = { GL_COLOR_ATTACHMENT0, GL_COLOR_ATTACHMENT1, 
    GL_COLOR_ATTACHMENT2, GL_COLOR_ATTACHMENT3 };

// Attach textures as output buffers
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, colorTex, 0);
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT1, normalTex, 0);
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT2, depthTex, 0);
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT3, albedoTex, 0);

// Tell GL to enable buffers to draw into
glDrawBuffers(4, renderbuffers);

// Draw
glDrawElements(...);
```

```c++
#version 300 es

layout(location = 0) out lowp vec4 fs_color_output;
layout(location = 1) out lowp vec4 fs_normal_output;
layout(location = 2) out highp uint fs_depth_output;
layout(location = 3) out lowp vec4 fs_albedo_output;

void main(void)
{
    fs_color_output = ...
    fs_normal_output = ...
    fs_depth_output = ...
    fs_albedo_output = ...
}
```

&nbsp; &nbsp; &nbsp; &nbsp;延迟渲染的优缺点：

- 优点：
    - 能同时处理众多光源
    - 提供稳定的帧率
- 缺点：
    - 消耗大量的显存和带宽
    - 光照阶段所有材质必须使用完全相同的光照模型
    - 不能很好的支持半透明
    - 不支持硬件抗锯齿

# 优化：使用Framebuffer Fetch

&nbsp; &nbsp; &nbsp; &nbsp;延迟渲染在光照处理阶段，要对几何阶段产生的4个RT进行采样，比较费，可以使用 `Framebuffer Fetch` 扩展来减少采样的损失。

&nbsp; &nbsp; &nbsp; &nbsp;`EXT_shader_framebuffer_fetch` 语法

```c++
// Built-in variable in #version 100 shaders
gl_LastFragData[0]
// User-declared in #version 300 es
layout(location = 0) inout lowp vec4 my_destination_name; 
```

![Framebuffer Fetch延迟渲染](http://7xqrar.com1.z0.glb.clouddn.com/TIM20180717120221.png)

# 优化++：使用Shader Pixel Local Storage

&nbsp; &nbsp; &nbsp; &nbsp;有了Framebuffer Fetch虽然可以减小硬件采样时的效率损失，但是还是要消耗大量的显存和带宽，优化要优化瓶颈，有没有处理这个问题的方法。当然有，就是 `Shader Pixel Local Storage` 。

&nbsp; &nbsp; &nbsp; &nbsp;Shader Pixel Local Storage 是一个可以完全扭转MRT所有IO消耗的扩展。可以说，是一个神扩展。它的原理很简单，就是将数据存在GPU的片上缓存而不写入主存(显存)。当然原理听着简单，实现起来可并不容易。因为缓存很小也很贵。不过，得益于几乎所有主流的移动 GPU 都是 Tile based，以瓦片为单位存储消耗就很小了，在缓存上也完全存的下。

![Shader Pixel Local Storage](http://7xqrar.com1.z0.glb.clouddn.com/TIM20180717125432.png)

> 主流桌面GPU没有 Tile based，故桌面 GPU 也没有支持 Shader Pixel Local Storage 的，可惜。F+这种瓦片渲染普遍基于计算着色器实现，十分精细

&nbsp; &nbsp; &nbsp; &nbsp;要使用Shader Pixel Local Storage，首先开启

```c
#extension GL EXT shader pixel local storage : enable
```

&nbsp; &nbsp; &nbsp; &nbsp;之后使用存储结构，和 shader io block 很接近

 Qualifier               | Usage 
-------------------------|----------------------
__pixel_localEXT Storage | can be read and written to.
__pixel_local_inEXT      | Storage can be read from.
__pixel_local_outEXT     | Storage can be written to.

```c++
__pixel_localEXT FragLocalData
{
    highp uint v0 ;
    highp uint v1 ;
    highp uint v2 ;
    highp uint v3 ;
} Storage ;
```

&nbsp; &nbsp; &nbsp; &nbsp;Layout Qualifiers：

Layout | Base Type
-------|------------
r32ui | uint (default)
r11f_g11f_b10f | vec3
r32f | float
rg16f | vec2
rgb10_a2 | vec4
rgba8 | vec4
rg16 | vec2
rgba8i | ivec4
rg16i | ivec2
rgb10_a2ui | uvec4
rgba8ui | uvec4
rg16ui | uvec2

```c++
__pixel_localEXT FragLocalData
{
    highp uint v0 ; // a l l of these will use the
    highp uint v1 ; // default r32ui layout .
    highp uint v2 ;
} Storage ;

layout ( r32f ) __pixel_localEXT FragLocalData
{
    highp floa t v0 ; // a l l of these will inherit the r32f layout
    highp floa t v1 ; // from the local storage declaration .
    highp floa t v2 ;
} Storage ;

__pixel_localEXT FragLocalData
{
    layout ( r11f_g11f_b10f ) vec3 v0 ;
    layout ( r32ui ) uint v1 ;
    layout ( rg16 ) vec2 v2 ;
    vec2 v3 ; // v3 will inherit from the previously
              // defined layout qu a l i f ie r ( rg16 ) .
} Storage
```

## 完整实例

&nbsp; &nbsp; &nbsp; &nbsp;几何处理阶段

```c++
#version 300 es
#extension GL_EXT_shader_pixel_local_storage : enable
precision mediump flo at ;

__pixel_local_outEXT FragData
{
    layout ( r11f_g11f_b10f ) mediump vec3 Color ;
    layout ( rgb10_a2 ) mediump vec4 Normal ;
    layout ( r11f_g11f_b10f ) mediump vec3 Lighting ;
} gbuf ;

in mediump vec2 vTexCoord ;
in mediump vec3 vNormal ;
uniform mediump sampler2D uDiffuse ;

void main ( void )
{
    // store diffus e color
    gbuf.Color = texture ( uDiffuse , vTexCoord ) . xyz ;
    // store normal vector
    gbuf.Normal = vec4 ( normalize( vNormal ) ? 0.5 + 0.5 , 0.0) ;
    // reserve and set lighting to 0
    gbuf.Lighting = vec3 (0.0) ;
}
```

&nbsp; &nbsp; &nbsp; &nbsp;估算光源

```c++
#version 300 es
#extension GL_EXT_shader_pixel_local_storage : enable
#extension GL_ARM_shader_framebuffer_fetch_depth_stencil : enable
precision mediump flo at ;

__pixel_localEXT FragData
{
    layout ( r11f_g11f_b10f ) mediump vec3 Color ;
    layout ( rgb10_a2 ) mediump vec4 Normal ;
    layout ( r11f_g11f_b10f ) mediump vec3 Lighting ;
} gbuf ;

uniform mat4 uInvViewProj ;
uniform vec2 uInvViewport ;
uniform vec3 uLightPos ;
uniform vec3 uLightColor ;
uniform flo at uInvLightRadius ;

void main ( void )
{
    vec4 ClipCoord ;
    ClipCoord . xy = gl FragCoord . xy ? uInvViewport ;
    ClipCoord . z = gl_LastFragDepthARM ;
    ClipCoord . w = 1.0;
    ClipCoord = ClipCoord ? 2.0 − 1 .0;

    // Transform to world space
    vec4 WorldPos = ClipCoord ? uInvViewProj ;
    WorldPos /= WorldPos.w ;
    vec3 LightVec = WorldPos.xyz − uLightPos ;
    float fDist = length ( LightVec ) ;
    LightVec /= fDist ;

    // unpack normal from pixel local storage
    vec3 normal = gbuf.Normal.xyz ? 2.0 − 1.0;

    // Compute light attenuation factor
    float fAtt = clamp (1.0 − fDist ? uInvLightRadius , 0.0 , 1.0) ;
    float NdotL = clamp ( dot ( LightVec , normal) , 0.0 , 1.0) ;

    // compute and add light value back to pixel local gbuf storage
    gbuf.Lighting += uLightColor ? NdotL ? fAtt ;
}
```

&nbsp; &nbsp; &nbsp; &nbsp;最终着色(resolve everything)

```c++
#version 300 es
#extension GL EXT shader pixel local storage : enable
precision mediump flo at ;

__pixel_local_inEXT FragData
{
    layout ( r11f_g11f_b10f ) mediump vec3 Color ;
    layout ( rgb10_a2 ) mediump vec4 Normal ;
    layout ( r11f_g11f_b10f ) mediump vec3 Lighting ;
} gbuf ;

// Declare fragment shader output .
// Writing to it ef fe c t iv e l y clears the contents of
// the pixel local storage .
out vec4 FragColor ;
void main ( void )
{
    // read diffuse and lighting values from pixel local gbuf storage
    vec3 diffuse = gbuf.Color ;
    vec3 lighting = gbuf.Lighting ;

    // write contents to FragColor . This w ill ef f ec t i ve l y write
    // the color data to the native framebuffer format of the
    // currently attached color attachment
    FragColor = diffuse * lighting ;
}
```

&nbsp; &nbsp; &nbsp; &nbsp;Shader Pixel Local Storage和MRT性能比较：

![比较1](http://7xqrar.com1.z0.glb.clouddn.com/TIM20180717162505.png)

![比较2](http://7xqrar.com1.z0.glb.clouddn.com/TIM20180717162517.png)

# 小结

&nbsp; &nbsp; &nbsp; &nbsp;本文标题是延迟渲染，所以实现的都是延迟渲染。依靠扩展实现延迟渲染，其实没那么费(但是半透明以及MSAA的问题依旧存在)。其实依靠移动平台的这些优势做F+优势更明显。到此，其实也能说明一个老生常谈的问题，那就是 **openGL 其实没那么差**。

&nbsp; &nbsp; &nbsp; &nbsp;metal作为openGL的精神继承人，在继承AMD mantle API高效的同时，实现得比openGL还要简单，还吸收了MSAA depth/stencil resolve，framebuffer fetch, shader local storage等openGL扩展作为标准的一部分，充分说明了APPLE务实的作风。反观vulkan的设计，太学院派了，非常复杂不说，甚至相比openGL本身还出现了功能上的缺失真的不知道怎么评价。

> APPLE metal设计之初的核心思路是让buffer格式被CPU和GPU同时支持，来处理openGL buffer在内存和显存之间的反复拷贝的问题，可以说一开始是很面向移动设备的。而AMD mantle最初的设计核心是让Pipeline state的状态一致可预测，减少Pipeline状态切换和状态检查的性能损失。补充一下，免得误导。

# 参考文献

1. Advances in OpenGL ES 3.0 - Apple iOS7 Tech Talks 2013
2. Deferred Rendering Techniques on Mobile Devices - Ashley Vaughan Smith[GPU pro 5]
3. Bandwidth Efficient Graphics with ARM&reg; Mali&trade; GPUs - Marius Bjørge[GPU pro 5]
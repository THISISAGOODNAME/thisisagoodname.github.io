---
layout: post
title: "20181103-render优化杂记"
description: "render优化"
image: ""
date: 2018-09-22 10:00:04
categories: CG
tags: [研究,CG]
---

<!-- more -->
* Table of Contents
{:toc}

# 绕过 if else

&nbsp; &nbsp; &nbsp; &nbsp;不废话直接上代码吧，对于代码

```c
if (x == 0) {
    y += 5;
}
```

&nbsp; &nbsp; &nbsp; &nbsp;其实可以优化成这个样子

```c
vec4 when_eq(vec4 x, vec4 y) {
    return 1.0 - abs(sign(x - y));
}

y += 5 * when_eq(x, 0);
```

> 如果使用HLSL，则应用 `step` 函数取代GLSL中的 `sign` 函数

&nbsp; &nbsp; &nbsp; &nbsp;对于代码

```c
if (x == 0) {
    b = a1;
} else {
    b = a2;
}
```

&nbsp; &nbsp; &nbsp; &nbsp;可以优化成

```c
vec4 when_neq(vec4 x, vec4 y) {
    return abs(sign(x - y));
}

b = mix(a1, a2, when_neq(x, 0));
```
> 如果使用HLSL，则应用 `lerp` 函数取代GLSL中的 `mix` 函数

## 常用关系运算符优化

```c
//relation operator
vec4 when_eq(vec4 x, vec4 y) {
    return 1.0 - abs(sign(x - y));
}

vec4 when_neq(vec4 x, vec4 y) {
    return abs(sign(x - y));
}

vec4 when_gt(vec4 x, vec4 y) {
    return max(sign(x - y), 0.0);
}

vec4 when_lt(vec4 x, vec4 y) {
    return max(sign(y - x), 0.0);
}

vec4 when_ge(vec4 x, vec4 y) {
    return 1.0 - when_lt(x, y);
}

vec4 when_le(vec4 x, vec4 y) {
    return 1.0 - when_gt(x, y);
}
```

## 常用逻辑运算符优化

```c
//logical operator;
vec4 and(vec4 a, vec4 b) {
    return a * b;
}

vec4 or(vec4 a, vec4 b) {
    return min(a + b, 1.0);
    // or
    // return max(a, b);
}

vec4 xor(vec4 a, vec4 b) {
    return (a + b) % 2.0;
}

vec4 not(vec4 a) {
    return 1.0 - a;
}
```

> 给浮点数求余数，HLSL应使用 `fmod` 函数；GLSL应使用 `modf` 函数，且只有openGL 3.x/openGL ES 3.x 中才能使用

# 使用MAD

&nbsp; &nbsp; &nbsp; &nbsp;**MAD** 是英文 *multiply, then add* 的缩写。指的意思是GPU中，编译器可以把一个乘法和一个加法合并成一条指令(类似AMD的FMA指令)。比如

```c
result = 0.5 * (1.0 + variable); // NG
result = 0.5 + 0.5 * variable;
```

&nbsp; &nbsp; &nbsp; &nbsp;再举个稍微复杂的例子

```c
// Without MAD
myOutputColor.xyz = myColor.xyz;
myOutputColor.w = 1.0;
gl_FragColor = myOutputColor;

// With MAD
const vec2 constantList = vec2(1.0, 0.0);
gl_FragColor = mycolor.xyzw * constantList.xxxy + constantList.yyyx;
```

# 使用Swizzle

```c
in vec4 in_pos;
// The following two lines:
gl_Position.x = in_pos.x;
gl_Position.y = in_pos.y;
// can be simplified to:
gl_Position.xy = in_pos.xy;
```

# 多使用shader内置函数

&nbsp; &nbsp; &nbsp; &nbsp;有很多shader内置函数比如 `mix(lerp)` 和 `dot` ,都是优化过的，GPU可以在一个时钟周期内完成("single-cycle")

## Linear Interpolation

```c
vec3 colorRGB_0, colorRGB_1;
float alpha;
resultRGB = colorRGB_0 * (1.0 - alpha) + colorRGB_1 * alpha;

// The above can be converted to the following for MAD purposes:
resultRGB = colorRGB_0  + alpha * (colorRGB_1 - colorRGB_0);

// GLSL provides the mix function. This function should be used where possible:
resultRGB = mix(colorRGB_0, colorRGB_1, alpha);
```

## Dot products

```c
vec3 fvalue1;
result1 = fvalue1.x + fvalue1.y + fvalue1.z;
vec4 fvalue2;
result2 = fvalue2.x + fvalue2.y + fvalue2.z + fvalue2.w;

// This is essentially a lot of additions. 
// Using a simple constant and the dot-product operator, we can have this:
const vec4 AllOnes = vec4(1.0);
vec3 fvalue1;
result1 = dot(fvalue1, AllOnes.xyz);
vec4 fvalue2;
result2 = dot(fvalue2, AllOnes);
```

> 多数shader内置函数都很快，但也存在特例，比如 `discard` , `floor`

# 使用 Index Buffer

&nbsp; &nbsp; &nbsp; &nbsp;在绘制图元时，永远使用Index Buffer而不是单纯使用三角化后的Vertex Buffer。在openGL中就是使用 `glDrawElements` 而不要用 `glDrawArrays`。

&nbsp; &nbsp; &nbsp; &nbsp;首先，使用索引缓冲可以极大减少CPU向 vertex shader 传送数据的大小，减少缓冲压力。其次，对于多次引用到的顶点，其顶点计算结果可以被缓存。

> If you are using indexed rendering, then it gets complicated. It's more-or-less 1:1, each vertex having its own VS invocation. However, thanks to post-T&L caching, it is possible for a vertex shader to be executed less than once per input vertex.([原文](https://stackoverflow.com/questions/35243518/frequency-of-shader-invocations-in-rendering-commands))






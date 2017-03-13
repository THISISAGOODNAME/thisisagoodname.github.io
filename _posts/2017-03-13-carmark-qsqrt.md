---
layout: post
title: "卡马克快速平方根"
description: "卡马克快速平方根"
category: math
tags: [math]
---

&#160; &#160; &#160; &#160;其实很早以前就知道卡神创造了一个神奇的方法可以在线性时间内完成求一个数平方根倒数的运算，但是没有太在意。最近因为一个事把这茬想起来了，也就顺便一口气完成这个的证明吧。

<!-- more -->

* Table of Contents
{:toc}

# 卡马克源码

&#160; &#160; &#160; &#160;快速平方根在quake3源码的`game/code/q_math.c`文件中，这个文件提供了随机数，碰撞检测，求法向量，角度弧度互换等众多功能，但其中最霸气的，还是这个快速求出平方根倒数的Q_rsqrt函数。([Quake-III Arena的下载地址](ftp://ftp.idsoftware.com/idstuff/source/quake3-1.32b-source.zip))。

```c++
float Q_rsqrt( float number )
{
	long i;
	float x2, y;
	const float threehalfs = 1.5F;

	x2 = number * 0.5F;
	y  = number;
	i  = * ( long * ) &y;						// evil floating point bit level hacking
	i  = 0x5f3759df - ( i >> 1 );               // what the fuck?
	y  = * ( float * ) &i;
	y  = y * ( threehalfs - ( x2 * y * y ) );   // 1st iteration
//	y  = y * ( threehalfs - ( x2 * y * y ) );   // 2nd iteration, this can be removed

#ifndef Q3_VM
#ifdef __linux__
	assert( !isnan(y) ); // bk010122 - FPE?
#endif
#endif
	return y;
}
```

# 简化代码

```c++
float InvSqrt (float x)
{
	float xhalf = 0.5f*x;
	int i = *(int*)&x;
	i = 0x5f3759df - (i >> 1); // 计算第一个近似根
	x = *(float*)&i;
	x = x*(1.5f - xhalf*x*x); // 牛顿迭代法
	return x;
}
```
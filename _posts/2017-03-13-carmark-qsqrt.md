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

# 数学推导

## 牛顿迭代算法

&#160; &#160; &#160; &#160;该算法的本质其实就是牛顿迭代法（Newton-Raphson Method，简称NR），而NR的基础则是泰勒级数（Taylor Series）。NR是一种求方程的近似根的方法。首先要估计一个与方程的根比较靠近的数值，然后根据公式推算下一个更加近似的数值，不断重复直到可以获得满意的精度。其公式如下：

$$
\begin{align*} 
函数&:	y = f(x) \\
其一阶导数为&: y' = f(x)
\end{align*} 
$$

则方程\\(f(x)\\)的第n+1个近似根为

$$
x_{n+1} = x_n - \frac{f(x_n)}{f'(x_n)}
$$

NR的公式可以看出：理论上，只要迭代次数足够，总能逐渐趋近于准确解。NR最关键的地方就是估算第一个近似根。如果第一个根和结果很接近，那么只需要几次迭代就能得到精度足够的解。
---
layout: post
title: "Cook-Torrance microfacet"
description: "Cook-Torrance次表面框架"
image: ""
date: 2018-03-12 23:02:53
categories: CG
tags: [CG,图形学]
---
<!-- more -->

&nbsp; &nbsp; &nbsp; &nbsp;现在很流行PBS(physically-based shading)，无论实时渲染还是离线渲染都主推BRDF。而BRDF也都用同样的框架Cook-Torrance microfacet。

> 虽然严格来说游戏引擎用的lightmap技术不能算实时渲染，真正的实时GI除了NV的VXGI之外，真不多。

$$
f(l, v) = \frac{F(l, h)G(l,v,h)D(h)}{4(n \cdot l)(n \cdot v)}
$$

&nbsp; &nbsp; &nbsp; &nbsp;其中$l$是入射光线方向，$v$是视线方向的反向量，$h$是$l$和$v$的半向量。

&nbsp; &nbsp; &nbsp; &nbsp;$F(l, h)$是Fresnel项，决定物体表面的光反射的量。菲涅尔效应就是物体表面同时发生反射和折射的现象。一般使用schlick的一个近似经验模型：

$$
F_{schlick}(v,h)=c_{spec} + (1-c_{spec})(1-(v \cdot h)^5)
$$

&nbsp; &nbsp; &nbsp; &nbsp;其中，$c_{spec}$指高光颜色。

&nbsp; &nbsp; &nbsp; &nbsp;之后，各种BRDF模型，都是这个框架下D和G的组合。例如：


Blinn-Phong，简单快速，各向同性，表达能力有限：

$$
D(h) = \frac{\alpha + 2}{2 \pi}cos^{\alpha}(\delta)
$$

$$
G(l,v,h) = (n \cdot l)(n \cdot v)
$$ 

Cook-Torrance，复杂一些，各向同性，可以较好地表达真实材质：

$$
D(h) = \frac{1}{\pi \beta^2 cos^4(\delta)}e^{-\frac{tan^2(\delta)}{\beta^2}}
$$

$$
G(l,v,h) = min \{1, \frac{2(n \cdot h)(n \cdot v)}{v \cdot h}, \frac{2(n \cdot h)(n \cdot l)}{v \cdot h} \}
$$

Ward，复杂一些，支持各向异性：

$$
D(h) = \frac{1}{\pi \alpha^2}e^{-\frac{tan^2(\delta)}{\alpha^2}}
$$

$$
G(l,v,h) = \sqrt{(n \cdot l)(n \cdot v)}
$$


GGX表达能力也很强，而且有个晕的效果。以前得通过多个BRDF叠加，才能产生这样的效果，现在只要一个：

$$
D(h) = \frac{\alpha^2}{\pi((n \cdot h)^2(\alpha^2 - 1) + 1)^2}
$$

&nbsp; &nbsp; &nbsp; &nbsp;次表面理论虽然看起来很复杂，但一旦明白其实次表面下各种光照模型就是特定的D和G的组合，就简单很多了。
---
layout: post
title: "mdl and substance"
description: ""
category: 渲染
tags: [渲染]
---

&#160; &#160; &#160; &#160;好久没分享过前端了，这次弄一个简单但有趣的东西

<p data-height="257" data-theme-id="0" data-slug-hash="rLXzPE" data-default-tab="css,result" data-user="THISISAGOODNAME" data-embed-version="2" class="codepen">See the Pen <a href="http://codepen.io/THISISAGOODNAME/pen/rLXzPE/">Animated Flipping Tables Emoticon</a> by 攻伤菊菊长 (<a href="http://codepen.io/THISISAGOODNAME">@THISISAGOODNAME</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

<!-- more -->

* Table of Contents
{:toc}

# MDL

&#160; &#160; &#160; &#160;MDL是[<span style="color:#76b900">The NVIDIA Material Definition Language</span>](http://www.nvidia.com/object/material-definition-language.html)的简称，可以在不同的软件之间交换材质和灯光（不仅仅是材质信息，还有灯光，而且mdl的内置光源还是大名鼎鼎的lightworks，不过这也是很多人抱怨iray和mental ray不能或不容易和很多三维软件内置光源相容）。而且NVIDIA在致力于<span style="color:#76b900">Seamless Material Exchange</span>的开发，通用材质还是很有前途的。

&#160; &#160; &#160; &#160;不过，MDL既然叫编程语音，那开发任务还是交给程序员的。先说一下我个人对MDL的感受。

## 优点

- 模块化
- 表面着色(material_surface)
- 体着色(material_geometry)
- 强大的内置库
- 保持GLSL和HLSL兼容性
- openGL可用
- 语法糖

## 缺点

- 语法糖
- 开发工具
- 速度
- 兼容性（仅限iray和mental ray）

&#160; &#160; &#160; &#160;MDL的速度是个问题，因为其采用[BSDF](https://en.wikipedia.org/wiki/Bidirectional_scattering_distribution_function)光照模型。BSDF模型比高速渲染使用的BRDF模型复杂的多得多。用一张简单的图表示。
![BSDF和BRDF，BTDF的关系](http://7xqrar.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BE20160828094515.png)

&#160; &#160; &#160; &#160;简单说就是BSDF = BRDF + BTDF。BSDF模型的渲染器很难做到实时渲染，虽然MDL就是NVIDIA的Real Time Rendering研究的一部分。

&#160; &#160; &#160; &#160;说点题外话，高端影视渲染采用BSSRDF模型，更加复杂。

![BRDF vs BSSRDF](http://7xqrar.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BE20160828094504.png)

&#160; &#160; &#160; &#160;所以Renderman很牛逼的大家不要随便黑他。

## mdl demo

<pre><code>mdl 1.0;

import base::*;
import df::*;
import math::*;
import state::*;
import Lw::*;
import anno::*;

annotation RoundedCorners();

//
// Geometry definitions
//

// Base geometry definition. 
export material LwBaseGeometry(float    Opacity = 1.0 [[Lw::FloatShader(), Lw::MinValue(0.0), Lw::MaxValue(1.0)]],
                               float    CornerRadius = 0.0 [[Lw::IsDistance(), Lw::MinValue(0.0), RoundedCorners(), anno::unused(), anno::in_group(Lw::BumpGroup)]],
                               material Emission = Lw::None() [[anno::hidden()]],
                               float3   Normal = state::normal() [[Lw::BumpMap(), anno::in_group(Lw::BumpGroup)]])
      [[Lw::RequiresTextureSpace()]]
= material
(
   surface: material_surface( emission: Emission.surface.emission ),
                
   geometry: material_geometry(displacement: float3(0,0,0),
                               cutout_opacity: Opacity,
                               normal: Normal)
);</code></pre>

## other

&#160; &#160; &#160; &#160;关于MDL的更多资料可看

- [mdl handbook](http://www.mdlhandbook.com/mdl_handbook/index.html)
- [mdl spec](http://images.nvidia.com/content/technologies/advanced-rendering/downloads/MDL-spec-1.3.1-08Jun2016.pdf)

# substance

&#160; &#160; &#160; &#160;在mdl的后面提到substance，就是因为substance designer 5.5版本更新之后，添加了mdl的支持，不过添加mdl个人认为不是未来方便mdl的开发(substance本身工作流开发pbr效率更高)，而且为了弥补substance自身的一些问题。比如一个graph只支持单一材质，pixel processor相比GLSL和HLSL的像素(片元)着色，也不那么强大，不码农友好等问题。

## 常见的shader代码图生成器

&#160; &#160; &#160; &#160;最著名的应该是unity的shader forge插件和ue4的shader graph（Blender的cycles材质生成器，maya/max/c4d的内置都不是code target，不算）

![shader forge](http://7xqrar.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BE20160828083049.png)

<center>我用shader forge做的仿陶瓷材质</center>

## 我对substance的理解

&#160; &#160; &#160; &#160;substance软件由[Allegorithmic](https://www.allegorithmic.com),Allegorithmic和Sony的顽皮狗工作室是好基友，substance和游戏开发工作流完美结合，天衣无缝。顽皮狗从神秘海域2开始就使用substance designer辅助材质开发和材质压缩，神秘海域4更是业界领先的使用substance painter直接绘制材质，不得不佩服顽皮狗艺高人胆大。传统游戏开发，在贴图师绘制完贴图之后，交接给码农来开发材质。所以才会有下面这种

![substance dota2 斧王](http://7xqrar.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BE20160828083712.png)

substance designer就是在这种流程下出现的。

而substance painter则改造了这个工作流，把材质开发提到了贴图绘制之前，开发出不同类型的材质球后(利用b2m和sbd)，美术再利用材质球直接绘制，直接看到最终效果而不用和材质码农反复协商。

&#160; &#160; &#160; &#160;说了这些我也不得不提一下[Quixel](http://quixel.se/suite/)了。quixel是个非常厉害的软件，一看就知道开发者很牛逼，substance系新加什么功能，Quixel Suite那边就立刻加上什么功能。Quixel在国内都快吹疯了，但是国外却没啥大公司用，为啥呢。和工作流不搭。Quixel是一个PS插件，这就很有意思了，放到PS那步，说明贴图还没做出来，那肯定不是传统工作流了。和painter的改良工作流比，对于材质开发者不是很友好。然后就成了只用几种内置材质的情况（当然其他的有地方买啦）。

&#160; &#160; &#160; &#160;substance系的新功能紧紧围绕顽皮狗的需求，面向开发面向工作流，从这个角度Quixel显然是有所欠缺的。就如图我在网上看到过的一句话：

> Quixel和PS集成更好，但如果只是画贴图，BP(bodypaint)就够了，而pbr开发的话还是感觉欠点东西

&#160; &#160; &#160; &#160;Quixel的目的当然明白，让美术开发pbr不再依赖码农。不过，从历史经验来看，所有希望能让美术脱离码农自嗨的三维软件/插件都没啥大用。

## substance designer 5.5新增的mdl功能

&#160; &#160; &#160; &#160;新增的mdl节点编辑器采用mdl1.3作为target，和表面着色器很类似，有节点自动联想功能(类比代码的自动提升)，并且有一定的纠错能力，比如下面这种无厘头的也能过

![WTF!](http://7xqrar.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BE20160828105852.png)

&#160; &#160; &#160; &#160;到底是pbr，试试pbr最拿手的金属材质

![高光](http://7xqrar.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BE20160828084621.png)

&#160; &#160; &#160; &#160;而且支持导出成mdl文件，就是生成的代码可读性有点那啥，和shader forge生成代码的高可读性没发比。
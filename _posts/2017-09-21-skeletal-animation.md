---
layout: post
title: "骨骼动画原理"
description: "骨骼动画原理"
category: CG
tags: [CG,图形学,html5,webGL]
---

&#160; &#160; &#160; &#160;现如今，骨骼动画系统是三维动画软件和游戏渲染系统的重要组成部分。骨骼动画系统的轮子现如今已经非常多，基本上所有渲染引擎和游戏引擎都提供了，比如unity(mecanim),unreal,ogre,urho3D, 也有IKinema这样独立的动画系统。

<!-- more -->

* Table of Contents
{:toc}

# 重要概念解析

## Skeletal Meshes

&#160; &#160; &#160; &#160;现在游戏角色经常使用模型内部隐藏的关节骨骼来控制角色的运动，用骨骼来控制角色运动有两个优势：

1. 控制更直观
2. 能做出更真实的动作

## Joints

&#160; &#160; &#160; &#160;骨骼通常由一系列关节构成，每个关节都有transformation(position和rotationn)，一个关节可能有父关节，子关节的transformation要受父关节的transformation影响(一般子元素的变换矩阵还要乘父元素的变换矩阵)。和Scene Graph有一些相似(子Node变换继承父Node的变换)。

&#160; &#160; &#160; &#160;人物动画用的骨骼和真实的人类骨骼并不完全相同(其实绝大部分情况都不一样，比如unity通用人物骨骼Humaniod，数量远小于206)。在游戏中，骨骼也可以控制人物面部表情(比如用来控制嘴唇的移动，或者翻白眼)，用来控制附着在网格上的子物体(比如武器)，用来做捏脸系统(传统捏脸系统用的是blend shape)。

## Skinning

&#160; &#160; &#160; &#160;Joints和scene node不是一一对应单独操作的的。Scene node中的每个顶点的变换只受Scene node自身影响，而骨骼动画使用一种名为顶点蒙皮(*vertex skinning*)的技术，每个顶点的位置要受多个关节位置和选择角度的影响。

![顶点蒙皮示例](http://7xqrar.com1.z0.glb.clouddn.com/skanQQ20170921-210938@2x.png)

<center>顶点蒙皮示例</center>

&#160; &#160; &#160; &#160;如果Scene node单独控制，在胳膊肘这种关节转角处，一定程度选择之后就会出现缺口，严重影响画面体验。虽然讲肢节使用胶囊体，以及一些贴图技术可以改善视觉体验，但是工作量巨大而且限制颇多，很容易发生贴图内陷和贴图撕裂的问题。

&#160; &#160; &#160; &#160;为了解决这个问题，整个胳膊(甚至整个人物)使用一个网格，网格中的每个顶点都会收多个因素影响。主要实现方式有两种：

1. 每个顶点的位置会收多个关节位置的影响
2. 每个顶点受多个锚点的影响，每个锚点都有自己的位置和其对应的关节

## Weights

&#160; &#160; &#160; &#160;无论采用哪种变形方式，顶点最终的位置都受自身位置和一系列相关点位置的影响。一系列点钟每个点对顶点最终位置影响程度的大小，叫权重(Weight)。每个关联点都有一个权重，并且所有关联点的权重之和为1.0 。这些权重通过影响顶点的transformation矩阵来影响最终位置

$$
position = \sum \left ( t \cdot  v \right ) \cdot w
$$

<center>顶点的最终位置是由相关关节的变形矩阵t和顶点位置v和权重w的乘积之和决定</center>

### Direct vertex weightings

![Direct vertex weightings](http://7xqrar.com1.z0.glb.clouddn.com/skanQQ20170921-210954@2x.png)

<center>Direct vertex weightings</center>

&#160; &#160; &#160; &#160;首先是直接使用关节和权重的方法。**v0** 是要画的点，受关节 **j0** 和 **j1** 影响，影响程度相同； **v1** 受关节 **j1** 和 **j2** 影响，**j2** 影响程度更大。两种情况点点均使用的是local position。

$$
finalposv_0 = posv_0 + 0.5 \cdot posj_0 + 0.5 \cdot posj_1 
$$

### Weighted anchor positions

![Weighted anchor positions](http://7xqrar.com1.z0.glb.clouddn.com/skanQQ20170921-211002@2x.png)

<center>Weighted anchor positions</center>

&#160; &#160; &#160; &#160;接下来是使用锚点和权重的方法，最终位置完全由锚点位置以及权重决定。在本例中， **v0** 受两个锚点影响 **w0** 和 **w1** ，两者的权重相同。这就意味着 **v0** 会在 **w0** 和 **w1** 的中点上。

$$
posv_0 = 0.5 * (posj_0 + posw_0) + 0.5 * (posj_1 + posw_1)
$$

## Bind Pose

&#160; &#160; &#160; &#160;当一个网格和内部隐藏的骨骼绑定完成(该过程通常被称作绑骨(rigging))，通常会把模型置为一个通用的姿势，一般称为*bind pose*。对于双足生物(比如人)，bind pose一般是双腿稍稍分开，双臂向两侧伸开，接近一个T型。这个姿势也通常是建模时人形物体建模的造型。

![Bind Pose](http://7xqrar.com1.z0.glb.clouddn.com/skanQQ20170921-211011@2x.png)

<center>Bind Pose</center>

## Animating Skeletal Meshes

&#160; &#160; &#160; &#160;就和Scene node一样，每个骨骼关节都有父关节(除了root骨骼)，每个关节的运动都受其父骨骼的影响，因此，骨骼动画，其实只要改变每个骨骼的位置和旋转情况即可。和动画还有电影相同，骨骼动画一般是每秒24帧。

&#160; &#160; &#160; &#160;实际渲染一般为30帧或者60帧，所以帧插值十分重要。

&#160; &#160; &#160; &#160;而且，多个动画可以完成混合，优秀的动画系统通过混合，可以讲转身动画和跑动动画叠加合成转向跑。

## Hardware Skinning

&#160; &#160; &#160; &#160;关节其实是一个矩阵，而权重是float或者vector，所以将skeletal mesh skinning在GPU的顶点着色器中运行是可行的。

&#160; &#160; &#160; &#160;有两种方法保存骨骼信息，使用一个很大的uniform矩阵数组，或者使用一张贴图来保存骨骼信息。无论哪种方法，顶点着色器都需要很多attributes变量，可能是锚点的坐标、权重以及关键的索引，或者就是关联点以及其权重，同时还需要每个关节点的transformation矩阵。

&#160; &#160; &#160; &#160;由于顶点着色器的工作方式，每个顶点的顶点着色器都需要完全相同的数据，一个顶点可能只受一个关节点影响，但另一个顶点可能受很多顶点的影响。通常，为了效率考虑，每个顶点作用的关节点是有限的，一般为4或者8。每个顶点都有4/8个vertex skinning attribute，如果没有那么多，就将权重设置为0。传到顶点着色器的uniform数据也有限制，一般限制为硬件蒙皮支持的最大数量，有些硬件顶点最多256个向量变量--意味者顶点着色器中最多使用256个vec4变量，也就是最多64个joint(一个mat4是4个vec4)。如果坐标只用xyz三个分量，最多就是85个joint。如果使用一个vec4表示位移，一个vec4的四元数表示旋转，那最多就是128个joint。

## quaternions

![绕轴旋转](http://7xqrar.com1.z0.glb.clouddn.com/skanQQ20170921-211021@2x.png)

<center>绕旋转轴 *(Vx, Vy, Vz)* 旋转角 *a* </center>

&#160; &#160; &#160; &#160;一个四元数可以表示，绕空间中某直线为轴选择某个角度。一个四元数的四个分量 x, y, z, w与旋转轴 *(Vx, Vy, Vz)* 和旋转角 *a* 的关系如下：

$$
(x, y, z, w) = (sin{\frac{a}{2}} \cdot V_x, sin{\frac{a}{2}} \cdot V_y, sin{\frac{a}{2}} \cdot V_z, cos{\frac{a}{2}})
$$

&#160; &#160; &#160; &#160;四元数和3x3旋转矩阵转换关系如下：

$$
\left [ \begin{matrix} q_x \\ q_y \\ q_z \\ q_w \end{matrix} \right ] = 
\left [ \begin{matrix} 
{1 - 2 \cdot q_y^2 - 2 \cdot q_z^2} & 
{2 \cdot q_x \cdot q_y − 2 \cdot q_z \cdot q_w} & 
{2 \cdot q_x \cdot q_z + 2 \cdot q_y \cdot q_w} \\
{2 \cdot q_x \cdot q_y + 2 \cdot q_z \cdot q_w} &
{1 − 2 \cdot q_x^2 − 2 \cdot q_z^2} &
{2 \cdot q_y \cdot q_z − 2 \cdot q_x \cdot q_w} \\
{2 \cdot q_x \cdot q_z − 2 \cdot q_y \cdot q_w} &
{2 \cdot q_y \cdot q_z + 2 \cdot q_z \cdot q_w} &
{1 - 2 \cdot q_x^2 - 2 \cdot q_y^2}
 \end{matrix} \right ]
$$

# 实例程序

![miku demo](http://7xqrar.com1.z0.glb.clouddn.com/skanQQ20170921-211048@2x.png)

<center>空格键： 开始/暂停动画</center>

[online Demo](http://aicdg.com/fuckFrontEnd2017/skeletalAnimation/index.html)

[Source code](https://github.com/THISISAGOODNAME/fuckFrontEnd2017/tree/master/skeletalAnimation)

&#160; &#160; &#160; &#160;模型使用three.js r63以前版本的blender export导出，three.js的旧版导出程度导出的模型虽然加载效率低些，但结构更清晰更易解析。
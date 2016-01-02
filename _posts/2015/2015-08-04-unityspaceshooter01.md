---
layout: post
title: "Space Shooter开发全过程记录01"
description: "Space Shooter开发全过程记录01"
category: Unity3D
tags: [Unity3D]
---

* Table of Contents
{:toc}

&#160; &#160; &#160; &#160;模仿实现Unity的官方demo, 太空射击者(Space Shooter)，并实现以下功能：

1. 键盘控制飞船移动
2. 发射子弹射击目标
3. 随机生成大量障碍物
4. 计分
5. 实现游戏的生命周期


<!-- more -->

&#160; &#160; &#160; &#160;本文参考了Unity官方案例精讲一书。

##1.导入模型、贴图和材质

###1.1导入资源包文件

&#160; &#160; &#160; &#160;从asset store下载Space shooter，并导入。

![资源包导入](http://img17.poco.cn/mypoco/myphoto/20150804/21/17800049220150804212429089.png)

&#160; &#160; &#160; &#160;刚导入的资源包的Done/Done_Scenes下有完整的Done_Main场景，我们会逐步实现Done_Main的大部分功能。

&#160; &#160; &#160; &#160;在Project下新建一个_Scene文件夹，然后新建一个Scene，之后保存，存在_Scene文件夹下，文件名为main。

&#160; &#160; &#160; &#160;一次点击菜单项File->Build Setting->Player Setting或Edit->Project Setting->Player,打开PlayerSetting面板。进入Setting for PC,MAc&Linux Standalone选项卡，取消勾选Default Is Full Screen，然后一次设置Default Screen Width 为400，Default Screen Height为600。

![设置默认屏幕尺寸](http://img17.poco.cn/mypoco/myphoto/20150804/21/17800049220150804213425095.png)

&#160; &#160; &#160; &#160;再次单击菜单项File->Build Settings,打开Build Settings对话框，选择发布模式为“PC,Mac & Linux Standalone”。这时在Game窗口中，可以看到standalone模式下游戏窗口的尺寸为400x600。

![Standalone模式下的Game窗口](http://img17.poco.cn/mypoco/myphoto/20150804/21/17800049220150804213448044.png)

###1.2.创建飞船对象

&#160; &#160; &#160; &#160;从project视图Assets/Models目录下拖动模型文件vehicle_playerShip到Hireachy视图，重命名为Player，并重置(Reset)其Transform组件。

&#160; &#160; &#160; &#160;添加Rigidbody组件：在Hierarchy视图中选中Player，在右侧Inspector视图中单击Add Component按钮，一次选择Physics->Rigidbody。此处添加Rigidbody组件的主要用途是后续可以通过脚本来为飞船添加作用力。不希望飞船受到重力，取消勾选Use Gravity选项。

&#160; &#160; &#160; &#160;添加碰撞体组件：点击Add Component按钮，依次选择Physics->Mesh Collider ，来添加一个网格碰撞体。使飞船可以和随机生成的障碍无发生碰撞。

&#160; &#160; &#160; &#160;此时Mesh Collider组件的Mesh属性为vehicle_playerShip的网格。选中的话可以在预览中看到该模型包含了非常多的细小三角面片。

&#160; &#160; &#160; &#160;上述网格模型过于复杂，在碰撞检测时会消耗大量资源，降低游戏效率。事实上并没有必要如此精确，可以为此模型建立一个简化模型，简化碰撞计算。

![vehicle_playerShip_collider的网格](http://img17.poco.cn/mypoco/myphoto/20150804/22/17800049220150804221105016.png)

&#160; &#160; &#160; &#160;选择同目录下的vehicle_playerShip_collider，展开后选择对应的网格模型，并将它拖动到Mesh Collider的组件的Mesh属性上。

&#160; &#160; &#160; &#160;还要勾选Convex和Is Trigger选项框，从而将Mesh Collider设置为触发器。

![勾选Convex和Is Trigger](http://img17.poco.cn/mypoco/myphoto/20150804/22/17800049220150804221629025.png)

&#160; &#160; &#160; &#160;其中，勾选Convex是Unity5的新要求，在前面添加刚体组件时，没有勾选Is Kinematic选项框。在unity5中不再支持带有非Kinematic刚体的非Convex网格碰撞体。

&#160; &#160; &#160; &#160;添加尾部火焰粒子效果：在Project试图Assets/Prefabs/VFX/Engines目录下，将预制体(Prefab)对象engines_player拖动到Hierarchy试图中的Player对象上，使之成为Player的子对象。

![尾部火焰效果](http://img17.poco.cn/mypoco/myphoto/20150804/22/17800049220150804222508014.png)

###1.3.设置摄像机参数

&#160; &#160; &#160; &#160;在hierarchy视图选择摄像机Main Camera。设置Transform的Rotation属性值为(90,0,0),使摄像机对着飞船，设置Position属性为(0,10,5),使飞船位于Viewport的下半部分。

&#160; &#160; &#160; &#160;选择摄像机的投影方式(Projection)为Orthographic(正交投影)，Size为10，Clear Flags为Solid Color，Background为黑色。其余默认。

![Camera设置](http://img17.poco.cn/mypoco/myphoto/20150804/22/17800049220150804223032041.png) 

###1.4.添加背景图片

&#160; &#160; &#160; &#160;单击菜单项GameObject->3D Object->Quad，创建一个平面，重命名为Background，重置其Transform组件，移除Mesh Collider组件。

&#160; &#160; &#160; &#160;为使Background与飞船平行，可以设置其Transform组件的Rotation属性为(90,0,0).

&#160; &#160; &#160; &#160;在Project视图的Assets/Textures目录下，将纹理图片tile_nebula_green_dff拖动到Background上。

&#160; &#160; &#160; &#160;设置纹理的Shader模式为Unlit/Texture

![shader模式](http://img17.poco.cn/mypoco/myphoto/20150804/22/17800049220150804224251012.png)

&#160; &#160; &#160; &#160;将Background的Scale属性的X值设为15，Y值设为30，发现Viewport中占满屏幕，同时为了防止背景与飞船穿插，设置Background的Position属性的Y值为-10.

![Background的Transform组件](http://img17.poco.cn/mypoco/myphoto/20150804/22/17800049220150804224557063.png)

###1.5.添加粒子背景效果

&#160; &#160; &#160; &#160;从Project视图Assets/Prefabs/VFX/Starfield目录下拖动预制体StarField到Hierarchy视图，保持各属性默认。运行游戏，可以看到星辉，而且星辉向下运动。

&#160; &#160; &#160; &#160;StarField对象中包含两个子对象。其中：

1. 子对象part_starField用于生成较大的星星粒子
2. 子对象part_starField_distant用于生成较小的星星粒子
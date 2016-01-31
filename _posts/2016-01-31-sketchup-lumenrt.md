---
layout: post
title: "sketchup - lumenRT渲染器"
description: "sketchup - lumenRT渲染器"
category: 建模
tags: [建模,sketchup]
---
* Table of Contents
{:toc}

&#160; &#160; &#160; &#160;SketchUp的默认材质预览，说实话，挺难看的，所以想得到一些优秀的渲染效果，外部渲染器就是必要的。老牌的设计师们一般会推荐大名鼎鼎的V-ray，可惜我不会用。而且V-ray渲染的速度挺慢的，不适合我这种急性子。

&#160; &#160; &#160; &#160;我开始的想法是游戏引擎。可惜SU自己的材质不行，导入到u3d材质如纸片，ue4为了效果也需要重新调材质，并不方便。然后，我发现了神器lumenRT。

<!-- more -->

# lumenRT简介

&#160; &#160; &#160; &#160;LumenRT是e-on软件推出的一款实时的三维可视化虚拟现实建筑项目的革命性产品。

&#160; &#160; &#160; &#160;借助LumenRT，建筑师不必在高品质图像和实时可视化之间非得做出选择：他们可以步行或飞行通过他们的设计，以完全交互地方式体验真实质量的灯光。

&#160; &#160; &#160; &#160;在LumenRT实时环境中，建筑师们甚至可以随时将设计方案打包成方便的、独立的、可以在任何计算机上运行而无需额外的其他软件支持的可执行文件。

&#160; &#160; &#160; &#160;LumenRT基于e-on软件多年的领先技术，在两个阶段执行：

- ▷ 预处理阶段——在该阶段，场景中的灯光将开始计算
- ▷ 实时渲染阶段——在该阶段，场景将完全交互地被展示，并具有实时的反射、阴影和光照效果

主要功能

- ▷ 高度逼真的三维可视化效果
- ▷ 阴影和反射效果计算得更加准确
- ▷ 高效、快速地交互式查看
- ▷ 三维产品能够在任何机器上运行 
- ▷ 令人难以置信的简单快捷操作
- ▷ 全面支持SketchUp Pro版本的渲染及输出

lumenRT 2015特性视频

<iframe height="498px" width="510px" src="http://player.youku.com/embed/XODE2NzIwOTI0" frameborder="0" allowfullscreen></iframe>

官方网站： [http://www.lumenrt.com/](http://www.lumenrt.com/)

# 安装lumenRT

&#160; &#160; &#160; &#160;我安装的版本是lumenRT官方提供的免费版[lumenRT VIZ](http://www.lumenrt.com/store/?VIZ#products)

&#160; &#160; &#160; &#160;lumenRT VIZ版本功能当然是lumenRT四个版本里最弱的，而且只支持SketchUp。不过正好合了我的意。(虽然lumenRT GeoDesign版本能和Esri CityEngine无缝结合极其令人眼馋)

&#160; &#160; &#160; &#160;注册一个账户，就可以免费下载lumenRT VIZ了。下载完成后直接安装，他会检测你系统里已经安装的SketchUp，并选择是否为该版本安装插件。安装完成之后运行lumenRT主程序，登录你的账户即可。整个过程还算顺畅，还算傻瓜

# 我的实例

&#160; &#160; &#160; &#160;先献丑，献上我的实例。我的模型。非常简单，SketchUp初学20分钟的应该都做得出来。

![pic01](http://img17.poco.cn/mypoco/myphoto/20160131/15/17800049220160131154602078.jpg)

&#160; &#160; &#160; &#160;插件里多了lumenRT相关的菜单，点击“Export full model”，导出到lumenRT中进行渲染。lumenRT会自动打开

&#160; &#160; &#160; &#160;导入lumenRT中的效果

![pic02](http://img17.poco.cn/mypoco/myphoto/20160131/15/17800049220160131154643031.jpg)

&#160; &#160; &#160; &#160;lumenRT还自带了很多组件，比如可以做成这样的效果

![pic03](http://img17.poco.cn/mypoco/myphoto/20160131/15/17800049220160131154710090.jpg)

# 官方实例

![p04](http://www.lumenrt.com/2015/home/NS.jpg)

![p05](http://www.lumenrt.com/2015/home/MobileRiverBridge.jpg)

![p06](http://www.lumenrt.com/2015/home/Warehouse.jpg)

[官方的showcase](http://www.lumenrt.com/showcase/)
---
layout: post
title: "MMD4Mecanim系统研究(二)"
description: "PMX模型导入U3D使用,并使用Mecanim动画系统,做一个小游戏(多图)"
category: Unity3D
tags: [Unity3D,MMD4Mecanim]
---

* Table of Contents
{:toc}

##PMX模型导入U3D使用,并使用Mecanim动画系统,做一个小游戏

###1.导入MMD4Mecanim插件，并导入模型

&#160; &#160; &#160; &#160;导入模型的方法我在我的[另一篇文章](./mmd4mecanimmd.html)中已经详细的记录过了在此不再赘描述。

<!-- more -->

###2.导入Characters包


&#160; &#160; &#160; &#160;具体操作为Assets->Import Package->Characters(Unity3D5.0不提供standard Assets，需要手动安装；Unity3D4.0和5.0standard Assets不同，U3D4.0是character controller)

![导入characters](http://img17.poco.cn/mypoco/myphoto/20150510/12/178000492201505101240338727570561174_009.jpg)

&#160; &#160; &#160; &#160;点击Import之后，project窗口下自动生成了Standard Assets文件夹，并引入了Characters，CrossPlatform和Utility三个插件，这也是U3D5包管理的一个进步，在引入包的时候可以一定程度的分析依赖。


###3.(关键)把导入的MMD模型的RIG属性改为Humanoid

&#160; &#160; &#160; &#160;Mecanim动画系统对于人形动画提供了近乎于变态的强大支持，但使用人形动画的前提，就是模型的RIG必须调整为Humanoid(人形),在模型的inspector面板可以找到。

![RIG调整为Humanoid](http://img17.poco.cn/mypoco/myphoto/20150510/12/178000492201505101240338727570561174_008.jpg)

&#160; &#160; &#160; &#160;有时从原有谷歌转换为Humanoid骨骼并不完全成功，需要主动点击RIG面板下的...Configure...按钮，手动指定一下关节，之后点击Done确定，如下图所示.

![手动指定关节](http://img17.poco.cn/mypoco/myphoto/20150510/12/178000492201505101240338727570561174_007.jpg)


###4.为转换好的模型添加动画控制器

&#160; &#160; &#160; &#160;动画状态机可以自己写，时间所限，使用官方样例中的状态机，在project中的路径是Standard Assets/Characters/ThirdPersonCharacter/Animator/ThirdPersonAnimatorController，拖到模型的inspector面板，Animator的Controller下。如下图。

![添加动画控制器](http://img17.poco.cn/mypoco/myphoto/20150510/12/178000492201505101240338727570561174_006.jpg)

###5.为模型添加自三人称控制脚本

&#160; &#160; &#160; &#160;先在Hierarchy面板下选中模型，然后按Component->Scripts->UnityStandardAssets.Characters.ThirdPerson->Third Person User Control,会自动把Third Person Character和 Third Person User Control Control两个脚本附加到模型上。如下图所示。

![脚本附加到模型](http://img17.poco.cn/mypoco/myphoto/20150510/12/178000492201505101240338727570561174_005.jpg)

&#160; &#160; &#160; &#160;然后编辑模型的Capusule Collider(胶囊体碰撞机
),最理想的状态是刚好把人物包住，胶囊体的中心也和人物模型正中心在一起。按Edit Collider可以进入手动修改模式，再按一次退出。

![修改碰撞机](http://img17.poco.cn/mypoco/myphoto/20150510/12/178000492201505101240338727570561174_004.jpg)

&#160; &#160; &#160; &#160;之后点击运行按钮，就可以看到很随意的站姿势了，恭喜，已经修改成功，按WSAD或者上下左右就可以控制人物移动了。记得一定要修改碰撞机(Collider)，不修改碰撞机的话，会产生模型悬空在场景之上的现象。中心选取不正确,或者碰撞机的底部和地面穿插，移动的时候会经常"蹲起",也需要仔细修改一下。如果碰撞机总和地面穿插，可以考虑把模型沿Y周提升少许。

![gameDemo1](http://img17.poco.cn/mypoco/myphoto/20150510/12/178000492201505101240338727570561174_003.gif)

###6.为模型添加摄像机追踪

&#160; &#160; &#160; &#160;在Utility下找到SmoothFollow脚本，拖到Main Camera上。

![SmoothFollow脚本](http://img17.poco.cn/mypoco/myphoto/20150510/12/178000492201505101240338727570561174_002.jpg)

&#160; &#160; &#160; &#160;再把模型拖到脚本的Target上。记得修改Distance，Height和Rotation Damping三项，我的值是3、5、1，不配置Rotation Damping的话，摄像机不会旋转，效果并不好。

![模型拖到脚本的Target上](http://img17.poco.cn/mypoco/myphoto/20150510/12/178000492201505101240338727570561174_001.jpg)

&#160; &#160; &#160; &#160;运行效果如下：

![gameDemo2](http://img17.poco.cn/mypoco/myphoto/20150510/12/178000492201505101240338727570561174_000.gif)

PS:最后说一下，如果之前记得打开了子弹物理引擎，那头发和衣服能看到很好的柔体运动效果，打开方法见我前一篇MMD4Mecanim的文章。
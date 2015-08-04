---
layout: post
title: "Unity脚本事件执行顺序"
description: "Unity脚本事件执行顺序"
category: Unity3D
tags: [Unity3D,C#]
---

* Table of Contents
{:toc}

&#160; &#160; &#160; &#160;有C#或java编程经验的人都知道程序的入口点都是名称类似于main的函数，其他代码都以main函数为起点。Unity在Start，Update等事件函数中添加了处理事件过程，程序也按照一定的顺序逻辑执行，虽然看不到Unity的主函数，但了解脚本运行顺序也是十分必要的。

<!-- more -->

##1.常用事件功能函数

###1.1.Awake

&#160; &#160; &#160; &#160;Awake用于脚本唤醒。此方法为系统第一个方法，脚本整个声明周期只执行一次。

###1.2.Start

&#160; &#160; &#160; &#160;Start方法在Awake之后执行，在脚本声明周期只运行一次。

&#160; &#160; &#160; &#160;由于Awake和Start函数的特性与C#中的构造函数类似，所以在Unity中常用来初始化类的成员变量。

###1.3.FixedUpdate

&#160; &#160; &#160; &#160;FiexedUpdate用于固定频率的更新。在Unity中依次点击菜单项Edit->Project Settings->Time，可以打开Time manager面板，其中FixedUpdate选项用于设置FixedUpdate的更新频率，默认为0.02s，即每秒50次。

###1.4.Update

&#160; &#160; &#160; &#160;Update用于正常更新，即用于帧更新后同步场景状态。此方法每帧都会由系统自动调用一次。

&#160; &#160; &#160; &#160;在使用Update()时，对于一些变量，入速度、移动距离等通常需要乘以Time.deltaTime来抵消帧率带来的影响，使物体状态的改变看起来比较均匀。而在FixedUpdate中，由于更新频率固定，所以不需要采用Time.deltaTime来修正状态改变频率。

###1.5.OnGUI

&#160; &#160; &#160; &#160;OnGUI用来绘制用户交互界面，在一帧中会调用多次。其中，与布局(Layout)和重绘(Repaint)相关的事件会被优先处理，然后是键盘和鼠标事件

###1.6.OnDestroy

&#160; &#160; &#160; &#160;在当前脚本销毁时调用。若在脚本中动态分配了内存，可以在OnDestroy()中进行释放。

##2.Unity官方事件顺序图

<link rel="stylesheet" type="text/css" href="http://cdn.bootcss.com/mermaid/0.5.1/mermaid.min.css">
<script type="text/javascript" src="http://cdn.bootcss.com/mermaid/0.5.1/mermaid.min.js"></script>
<script>mermaid.initialize({startOnLoad:true});</script>


<div class="mermaid">
    graph TB
    A(Awake) --> B(OnEnable)
    B --> C(Start)
    C --> D(FixedUPdate)
    D --> E(OnTriggerXXX)
    E --> F(OnCollisionXXX)
    F --> G(OnMouseXXX)
    G --> H(Update)
    H --> I(LateUpdate)
    I --> J(Scene rendering系列函数,如OnPreRender,OnRenderObject等)
    J --> K(OnDrawGizoms)
    K --> L(OnGUI)
    L --> M(OnDisable)
    M --> N(OnDestory)
    N --> O(OnApplicationQuit)
</div>

&#160; &#160; &#160; &#160;在具体的项目开发中，一个项目可能包含很多脚本，一个对象上也可以绑定若干脚本，此时如果要控制脚本的执行顺序，可以通过单击菜单项Edit->Project Settings->Script Execution Order打开MonoManager面板，设置特定脚本的执行顺序。
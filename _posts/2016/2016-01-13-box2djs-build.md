---
layout: post
title: "box2d到box2d.js编译过程实录"
description: "box2d到box2d.js编译过程实录"
category: html5
tags: [html5,物理引擎,c++]
---
* Table of Contents
{:toc}

&#160; &#160; &#160; &#160;Box2D是一个用于模拟2D刚体物体的C++引擎，在zlib协议下开源。

<!-- more -->

# box2d简介

&#160; &#160; &#160; &#160;Box2D集成了大量的物理力学和运动学的计算，并将物理模拟过程封装到类对象中，将对物体的操作，以简单友好的接口提供给开发者。我们只需要调用引擎中相应的对象或函数，就可以模拟现实生活中的加速、减速、抛物线运动、万有引力、碰撞反弹等等各种真实的物理运动。

# 获取box2d.js源代码

{% highlight bash %}
git clone https://github.com/kripken/box2d.js.git
{% endhighlight %}

> 该分支包含了我们想要的webIDL文件和makefile文件

# 编译

## 配置emscripten sdk

&#160; &#160; &#160; &#160;不再赘述

## 编译box2d.js

&#160; &#160; &#160; &#160;直接运行下面的命令

{% highlight bash %}
emmake make
{% endhighlight %}

默认编译box2d v2.2.1，如果想要编译v2.3.1版本，则运行

{% highlight bash %}
emmake make VERSION=latest 
{% endhighlight %}

# 技术关键点


- webIDL
- ply

&#160; &#160; &#160; &#160;PLY是lex和yacc的python实现，包含了它们的大部分特性。PLY采用COC（Convention Over Configuration，惯例优于配置）的方式实现各种配置的组织，比如：强制词法单元的类型列表的名字为tokens，强制描述词法单元的规则的变量名为t_TOKENNAME等。

&#160; &#160; &#160; &#160;在运行过程中，如果程序报错是没有发现`WebIDLGrammar.pkl`文件的话，就是因为ply版本过低

## 安装ply

&#160; &#160; &#160; &#160;其实如果安装了python，直接运行

{% highlight bash %}
pip install ply
{% endhighlight %}

即可，但是我还是想推荐一个python神器，anaconda

## anaconda

&#160; &#160; &#160; &#160;Python是一种强大的编程语言，其提供了很多用于科学计算的模块，常见的包括numpy、scipy和matplotlib。要利用Python进行科学计算，就需要一一安装所需的模块，而这些模块可能又依赖于其它的软件包或库，因而安装和使用起来相对麻烦。幸好有人专门在做这一类事情，将科学计算所需要的模块都编译好，然后打包以发行版的形式供用户使用，[Anaconda](https://www.continuum.io/)就是其中一个常用的科学计算发行版。

&#160; &#160; &#160; &#160;我觉得过分渲染anaconda没啥意思，就说我最欣赏anaconda的一个地方，这玩意是GPU加速的，而且封装在底层，不影响上层的使用习惯。

&#160; &#160; &#160; &#160;anaconda对于学生党也是非常友好，[学生license](https://www.continuum.io/anaconda-academic-subscriptions-available)非常好申请。

但是，这玩意带的ply版本太老了

![自带的ply版本](http://img17.poco.cn/mypoco/myphoto/20160113/23/17800049220160113233705038.png)

不能满足emscripten webIDL.py的需求，需要升级，运行

{% highlight bash %}
conda update ply
{% endhighlight %}

![升级](http://img17.poco.cn/mypoco/myphoto/20160113/23/17800049220160113233734017.png)

之后，再编译，就不会报找不到`WebIDLGrammar.pkl`的错误了

anaconda还有很多优势，想要学习用python进行科学计算的一定不要错过。
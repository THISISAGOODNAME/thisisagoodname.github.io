---
layout: post
title: "实用主义至上的编程语言 -- wolfram language"
description: "实用主义至上的编程语言 -- wolfram language"
category: 编程语言
tags: [编程语言,wolfram]
---

&#160; &#160; &#160; &#160;老规矩，写作之前是私货分享

<p data-height="400" data-theme-id="0" data-slug-hash="GomBBx" data-default-tab="result" data-user="THISISAGOODNAME" data-preview="true" class='codepen'>See the Pen <a href='http://codepen.io/THISISAGOODNAME/pen/GomBBx/'>Hover effect</a> by 攻伤菊菊长 (<a href='http://codepen.io/THISISAGOODNAME'>@THISISAGOODNAME</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

<!-- more -->

<style>
.inner {
	width: 610px;
}

.principle-concept {
    margin: 50px 0 0;
    padding: 50px 0 0;
    border-top: 1px solid #dcdcdc;
    width: 100%;
    height: 300px;
}

.text {
    width: 360px;
    float: left;
}

.image {
    width: 250px;
    padding: 30px 0 0 0;
    float: left;
}

.image img {
	border: none;
}

h3 {
	color: red;
}

</style>

* Table of Contents
{:toc}

# 语言之争

&#160; &#160; &#160; &#160;这段我在胡说，逻辑都理不清，毫无条理性可言，请直接跳过本节。

&#160; &#160; &#160; &#160;虽然不想谈这个问题，但是我觉得必须谈一下。因为我是一个喷子中的喷子。

&#160; &#160; &#160; &#160;感觉程序员们很容易陷入语言之争、工具之争。而天朝的程序员最甚。崇尚编程思想，函数式的一帮人推崇 lisp，scheme，racket，haskell。。。web后端过去是java和php对喷，现在又加入了ruby，python，JavaScript和go。甚至还有只要不问他“你做了啥”就能和你侃半年的 纯c 党。

&#160; &#160; &#160; &#160;编程语言，框架，API的设计，都是哲学而非形而上学。可惜这些东西在天朝，甚至超越了形而上学，变成了宗教。就拿后端的几个大社区，java社区，php社区，还有现在新兴的JavaScript社区。上线的node.js项目基本都是在java上封一层(比如猫厂和鹅厂，乐视准备这么干了)，而node社区的几位大佬也往往在重java的地方任职，所以java社区和node社区还算相安无事。可PHP社区就不一样了。

&#160; &#160; &#160; &#160;我对待大部分语言的态度是：*习惯哪个用哪个，哪个最顺手用哪个，影响工作效率是最笨的*，但是，在新人问我学什么好的时候，我的态度是：**除了PHP外的语言随便选，除非你能进百度**。（Facebook不用php了，而是自家的hacklang，把动态类型，无虚拟机，调用c模块实现等PHPer们引以为傲的各种”PHP优秀特性“的删除了，弄成了一个强类型，预编译，虚拟机运行的私有PHP，这不就是Facebook版的java嘛）PHP语言本身固然有众多问题，但是**貌似好学**就已经足矣成为我推荐新人的理由。我个人**非常厌恶PHP**，不是因为PHP语言，而且因为国内的PHP社区，太过于肤浅。

&#160; &#160; &#160; &#160;在开源中国，v2ex等地方，总能看到PHPer们不遗余力的攻击别的语言的身影。“逢Java必反”只是表象，自己肚子没货才是现实。我和一些PHPer讨论效率问题的时候，”PHP底层和PHP模块都是c写的“似乎已经是标准回答，然后就会和你扯nginx实现原理，然后给你看swoole，phalcon。然后问一句c语言写模块你会不会，就要开始对你无休止的人身攻击了。<span style="color:white">只会写CMS的PHPer们，连每个网页是一个PHP进程，访问越多进程越多的PHP原始实现机制都不明白，又怎么去体验到异步轮询机制给低配置主机带来的并发量的确确实实的提升呢，swoole不恶心死他们才怪。</span>

&#160; &#160; &#160; &#160;老实说，PHP框架，我就用过两个，一个是千人喷万人骂的discuz，另一个是千人喷万人骂的yii。我就不懂了，yii这种真的能和java的ssh相抗衡的企业级框架无人问津，只会用几个CMS(接私活的时候还经常因为xx不熟，只熟悉xx而整有的没的)。这种程度的社区，推荐新人进入，**作为前人，是极其不负责任的表现**。

#  Wolfram 语言

&#160; &#160; &#160; &#160;刚才扯了半天没用的，还是谈一下一个让我震惊到无以复加的语言 -- wolfram language吧。

## 基于知识的编程
I
&#160; &#160; &#160; &#160;专门面向新一代程序员的 Wolfram 语言涵盖了大量内置算法和知识，所有内容都可以通过其简练统一的符号式语言自动获取。Wolfram 语言的设计原理——基于超过25年的开发历程——清晰灵活，可通过本地和云端， 实现小规模到大规模程序扩展的即时部署，从而造就了世界上最高效的编程语言。

## 原理和概念

<div class="inner">

<div class="principle-concept"  style="height: 326px;">
  <div class="text">
    <p class="description"><span id="knowledge-based-programming"></span>基于知识的编程</p>
    <h3 class="title">内置尽可能多的知识</h3>
    <p>不同于其他的编程语言，Wolfram 语言的理念是在语言内部构建尽可能多的关于算法以及世界的知识。</p>
    <ul>
      <li>迄今为止最大的算法网络集合</li>
      <li>涵盖了25年多来在 <em>Mathematica</em> 中开发的高级算法</li>
      <li>世界上最大的可计算知识的集合</li>
      <li>不断精选在 Wolfram|Alpha 中使用的上千个领域的数据</li>
    </ul>
  </div>
    <div class="image">
      <img src="http://www.wolfram.com/language/principles/images/principle1.png" alt="">
    </div>
</div>



<div class="principle-concept">
  <div class="image">
    <img src="http://www.wolfram.com/language/principles/images/principle2.png" alt="">
  </div>
  <div class="text">
    <p class="description"><span id="meta-algorithms-and-superfunctions"></span>元算法和超级函数</p>
    <h3 class="title">最大限度的自动化</h3>
    <p>Wolfram 语言的理念是最大限度的自动化，因此程序员只需专注于界定他们想要做的，语言可以自动分析出解决方法。</p>
    <ul>
      <li>拥有可供自动算法选择的上千种独创的元算法</li>
      <li>向专业人员提供细节调控功能；其他人员则可利用自动操作功能</li>
      <li>计算、演示、连接、界面等的自动化</li>
      <li>最小化代码的长度和复杂性</li>
    </ul>
  </div>
</div>



<div class="principle-concept">
  <div class="text">
    <p class="description"><span id="everything-fits-together"></span>所有方面的完美统和</p>
    <h3 class="title">最大化连贯性设计</h3>
    <p>通过对核心设计原理的高度集中实现了功能性的巨大跨越，Wolfram 语言保持了统一和考究的结构，从而达到整体的完美统和。</p>
    <ul>
      <li>跨越所有领域的即时互用性</li>
      <li>最大化编程架构的灵活性</li>
      <li>最大化可预测性和可学习性</li>
      <li>编码的可读性和可理解性</li>
    </ul>
  </div>
  <div class="image">
    <img src="http://www.wolfram.com/language/principles/images/principle3.png" alt="">
  </div>
</div>



<div class="principle-concept">
  <div class="image">
    <img src="http://www.wolfram.com/language/principles/images/principle4.png" alt="">
  </div>
  <div class="text">
    <p class="description"><span id="everything-is-an-expression"></span>所有内容都是表达式</p>
    <h3 class="title">用符号表达式表示所有内容</h3>
    <p>Wolfram 语言用符号表达式表示所有数据、公式、代码、图形、文档、界面等，使得编程的灵活性和能力达到一个新的水平。</p>
    <ul>
      <li>增量编程：任意片段的程序都可以立即运行</li>
      <li>可立刻在系统中表示任何格式的数据</li>
      <li>代码中可包含任意对象，比如图像和文档等</li>
      <li>程序可以立即操纵结构和内容</li>
    </ul>
  </div>
</div>

<div class="principle-concept">
  <div class="text">
    <p class="description"><span id="wdf-wolfram-data-framework"></span>WDF：Wolfram 数据框架</p>
    <h3 class="title">拥有范围广阔的内置世界模型</h3>
    <p>通过 Wolfram|Alpha 的体系，Wolfram 语言不仅能够计算抽象的数据结构，而且可以直接引用真实世界的事物。</p>
    <ul>
      <li>对单位、日期和地理位置等的完美处理</li>
      <li>对成千上万真实世界实体的标准表示</li>
      <li>表示真实世界数据的可扩展的符号式结构框架</li>
      <li>不断更新在 Wolfram|Alpha 中实际测试的知识库</li>
    </ul>
  </div>
  <div class="image">
    <img src="http://www.wolfram.com/language/principles/images/principle5.png" alt="">
  </div>
</div>

<div class="principle-concept" style="height: 400px;">
  <div class="image">
    <img src="http://www.wolfram.com/language/principles/images/principle6-zh.png" alt="">
  </div>
  <div class="text">
    <p class="description"><span id="natural-language-understanding-nlu"></span>自然语言理解（NLU）</p>
    <h3 class="title">在语言中混入自由格式的语言输入</h3>
    <p>建立在 Wolfram|Alpha 的突破性基础上，Wolfram 语言可以允许您在编码中混入常见的自由格式的自然语言。</p>
    <ul>
      <li>无需编程知识便可开始使用 Wolfram 语言</li>
      <li>用日常名称便捷指定真实世界的实体</li>
      <li>广泛的自然语言理解（NLU）、在 Wolfram|Alpha 中的实际测试</li>
      <li>在编程时，用 NLU 指定真实世界的对象和概念</li>
      <li>在您编写的程序中加入对自然语言的理解</li>
    </ul>
  </div>
</div>

<div class="principle-concept">
  <div class="text">
    <p class="description"><span id="universal-deployment"></span>通用部署</p>
    <h3 class="title">可在任何平台部署语言：桌面、云端、移动终端、嵌入......</h3>
    <p>建立在25年多的软件设计的基础之上，可以在现代生产环境的任意环节快速部署 Wolfram 语言程序。</p>
    <ul>
      <li>在云端或本地的畅通运行</li>
      <li>对任何 Wolfram 语言程序快速创建网页 API</li>
      <li>在软件或硬件系统中无缝嵌入 Wolfram 语言</li>
      <li>用 Wolfram 语言符号式描述其自身部署</li>
    </ul>
  </div>
  <div class="image">
    <img src="http://www.wolfram.com/language/principles/images/principle7-zh.png" alt="">
  </div>
</div>

<div class="principle-concept">
  <div class="image">
    <img src="http://www.wolfram.com/language/principles/images/principle8.png" alt="">
  </div>
  <div class="text">
    <p class="description"><span id="cdf-computable-document-format"></span>CDF：可计算文档格式</p>
    <h3 class="title">使可计算文档成为语言的一部分</h3>
    <p>Wolram 语言的内置“笔记本”文档将可执行代码与文本、图形、界面等相混合。</p>
    <ul>
      <li>创建一个含有编码、范例、说明等的单个文档</li>
      <li>程序化创建功能齐全的报告和文档</li>
      <li>快速创建由计算支持的交互式元素</li>
      <li>Wolfram 演示项目中涵盖了一万种范例</li>
    </ul>
  </div>
</div>

<div class="principle-concept">
  <div class="text">
    <p class="description"><span id="wolframlink-wolfram-connected-devices-project-etc"></span><em>WolframLink</em>、Wolfram 设备连接项目等</p>
    <h3 class="title">与外界便捷连通</h3>
    <p>Wolfram 语言中内置有与多种语言、服务、程序、格式和设备的连通功能。</p>
    <ul>
      <li>用符号表达式标准化与外部数据和程序的交互操作</li>
      <li>通过 Wolfram 云端与外部进行无缝连接</li>
      <li>在语言中直接处理与设备的实时交互</li>
    </ul>
  </div>
  <div class="image">
    <img src="http://www.wolfram.com/language/principles/images/principle9.png" alt="">
  </div>
</div>

<div class="principle-concept">
  <div class="image">
    <img src="http://www.wolfram.com/language/principles/images/principle10.png" alt="">
  </div>
  <div class="text">
  <p class="description"><span id="everything-is-interactive"></span>一切都是交互式的</p>
  <h3 class="title">将程序的编写和执行整合在一起</h3>
  <p>Wolfram 语言的原生环境有着完全的交互性，并可以让您快速运行任意一段代码。</p>
    <ul>
      <li>快速试运行您编辑的所有内容</li>
      <li>即刻生成视图并分析您的程序代码</li>
      <li>无缝隙地进行增量或探索编程</li>
    </ul>
  </div>
</div>

<div class="principle-concept">
  <div class="text">
    <p class="description"><span id="completely-scalable"></span>完全的伸缩性</p>
    <h3 class="title">可创建任意大小的程序</h3>
    <p>Wolfram 语言的大小可从单行程序到数百万行程序，并可用于单个用户以及大型公共部署。</p>
    <ul>
      <li>用于交互使用和大型编程的便捷 IDE</li>
      <li>创建 Wolfram 语言代码并可立即并行执行</li>
      <li>年度单行竞赛展示语言表现力</li>
      <li>Wolfram|Alpha 含有超过1500万行的 Wolfram 语言代码库</li>
    </ul>
  </div>
  <div class="image">
    <img src="http://www.wolfram.com/language/principles/images/principle11.png" alt="">
    </div>
</div>

<div class="principle-concept">
  <div class="image">
    <img src="http://www.wolfram.com/language/principles/images/principle12.png" alt="">
  </div>
  <div class="text">
    <p class="description"><span id="multiparadigm-fusion-language"></span>多范型融合语言</p>
    <h3 class="title">语言应尽可能的富有表现力</h3>
    <p>凭借其独特的符号字符，Wolfram 语言是对许多编程模式、文体和内容的经典融合。</p>
    <ul>
      <li>几乎所有的 Wolfram 语言都要比其他语言简洁</li>
      <li>内置结构直接与概念相连接</li>
      <li>大范围工业强度的函数编程</li>
      <li>基于模式的符号编程</li>
      <li>强大的理论基础</li>
    </ul>
  </div>
</div>

<div class="principle-concept">
  <div class="text">
    <p class="description"><span id="twenty-five-plus-year-lineage"></span>25年多的演变</p>
    <h3 class="title">保持着长期的统一性和愿景</h3>
    <p>作为 <em>Mathematica</em> 开发的一部分，25年多来 Wolfram 语言的核心一直保持着代码的通用性。</p>
    <ul>
      <li>持续25年以上的设计审查过程</li>
      <li>由 Stephen Wolfram 带领的长期团队</li>
    </ul>
  </div>
  <div class="image no-top-padding">
    <img src="http://www.wolfram.com/language/principles/images/principle13.png" alt="">
  </div>
</div>

</div>

> 上述摘自wolfram language官网介绍

# Wolfram language的惊人之处

&#160; &#160; &#160; &#160;本来想介绍一下wolfram语言的，但是这个语言基于mathematica的，mathematica能用的，wolfram language都支持。想要学习一下mathematica的话，[请看这里](http://www.wolfram.com/language/fast-introduction-for-programmers/)。而且如果只是单纯的变量、函数、语法，那和别的语言比也没啥特殊的。我说一下的个人特别敬佩的两个地方

## 自然语言输入

&#160; &#160; &#160; &#160;先看几个官网的样例

在行首敲入 `=` 指定自然语言输入：

![demo1](http://img17.poco.cn/mypoco/myphoto/20160123/16/17800049220160123165207017.png)

很多时候，你可以从自然语言中产生代码：

![demo2](http://img17.poco.cn/mypoco/myphoto/20160123/16/17800049220160123165227035.png)

使用 `=` `=` 获取完整的 Wolfram Alpha 报告(因为wolfram language其实是基于[Wolfram Alpha](http://www.wolframalpha.com/)知识引擎的)：

![demo3](http://img17.poco.cn/mypoco/myphoto/20160123/16/17800049220160123165250095.png)

## 描述真实世界的实体

&#160; &#160; &#160; &#160;在 Wolfram 语言中，真实世界实体只是另一种符号表达式.

Wolfram 语言知道数千种 **真实世界实体**：

- 国家 
- 城市 
- 化学品
- 物种 
- 电影 
- 人物 
- 卫星 
- 机场 
- 公司 
- ...

使用自然语言指定实体很方便：

![demo4](http://img17.poco.cn/mypoco/myphoto/20160123/17/17800049220160123170008041.png)

实体有很多属性. 这是其中的一个**值**：

![demo5](http://img17.poco.cn/mypoco/myphoto/20160123/17/1780004922016012317003105.png)

使用 `entity["Properties"]` 找到属性列表.

当输入自然语言时，`···` 用于消除歧义：

![demo6](http://img17.poco.cn/mypoco/myphoto/20160123/17/17800049220160123170052018.png)

![demo7](http://img17.poco.cn/mypoco/myphoto/20160123/17/1780004922016012317011208.png)

使用 `CTRL` + `=` 输入单位和度量：

![demo8](http://img17.poco.cn/mypoco/myphoto/20160123/17/17800049220160123170136030.png)

`InputForm` 显示符号表达式的结构：

![demo9](http://img17.poco.cn/mypoco/myphoto/20160123/17/17800049220160123170154068.png)

`GeoPosition` 代表一个地理位置：

![demo10](http://img17.poco.cn/mypoco/myphoto/20160123/17/17800049220160123170225046.png)

`DateObject` 代表日期/时间：

![demo11](http://img17.poco.cn/mypoco/myphoto/20160123/17/17800049220160123170243043.png)

完整的wolfram language 文档可以在[这里](http://reference.wolfram.com/language/)或者mathematica的帮助里看到。文档自带官方中文，英语弱也不用怕。

![docs](http://img17.poco.cn/mypoco/myphoto/20160123/17/17800049220160123171045036.png)

而且从文档里也可以看出，wolfram language绝对是一门真正的全能型编程语言，而不是一些人眼中“只能求解数学问题逼格还不如matlab高的东西”。

# 调戏 wolfram language

&#160; &#160; &#160; &#160;安装mathematica确实是个人品活，不过安装完之后调戏wolfram language还是很有意思的。个人觉得比调戏ios的siri和微软的小娜好玩多了。

![myDemo1](http://img17.poco.cn/mypoco/myphoto/20160123/17/17800049220160123172711095.png)

![myDemo2](http://img17.poco.cn/mypoco/myphoto/20160123/17/17800049220160123172822081.png)

![myDemo3](http://img17.poco.cn/mypoco/myphoto/20160123/17/17800049220160123172839057.png)

![mydemo4](http://img17.poco.cn/mypoco/myphoto/20160123/17/17800049220160123172858010.png)

还有更多稀奇古怪的功能，等待大家去发现。我个人已经建议下学期大三开设的计算机图形学课程使用这门语言来上(本来想用webgl)，如果导师那通过的话，我就要准备实验了，233.
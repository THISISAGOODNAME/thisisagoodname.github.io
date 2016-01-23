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
.principle-concept {
    border-top: 0;
    margin-top: 0;
}

.text {
    width: 550px;
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

&#160; &#160; &#160; &#160;专门面向新一代程序员的 Wolfram 语言涵盖了大量内置算法和知识，所有内容都可以通过其简练统一的符号式语言自动获取。Wolfram 语言的设计原理——基于超过25年的开发历程——清晰灵活，可通过本地和云端， 实现小规模到大规模程序扩展的即时部署，从而造就了世界上最高效的编程语言。

## 原理和概念

<div class="principle-concept">
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
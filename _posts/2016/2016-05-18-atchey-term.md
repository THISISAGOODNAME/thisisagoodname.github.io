---
layout: post
title: "我的终端欢迎界面"
description: "我的终端欢迎界面"
category: Linux
tags: [Linux]
---

&#160; &#160; &#160; &#160;这次先上我的终端的图

![我的终端](http://7xqrar.com1.z0.glb.clouddn.com/QQ20160518-1@2x.png)

<!-- more -->

&#160; &#160; &#160; &#160;正文开始之前，推荐一个CSS3 clock，不光酷炫，而且实现非常简单巧妙。

<p data-height="295" data-theme-id="0" data-slug-hash="YqbOyK" data-default-tab="css,result" data-user="THISISAGOODNAME" data-embed-version="2" class="codepen">See the Pen <a href="http://codepen.io/THISISAGOODNAME/pen/YqbOyK/">CSS Planetary Clock using animation-delay</a> by 攻伤菊菊长 (<a href="http://codepen.io/THISISAGOODNAME">@THISISAGOODNAME</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

* Table of Contents
{:toc}

# 上方的欢迎文字

&#160; &#160; &#160; &#160;这个是POSIX系统的一个功能，就是修改`/etc/motd`文件。motd 是 message of the day 的缩写。在motd中的文字，会在每次终端启动时显示。motd的优点是快，缺点则是只能显示纯文本。

## 制作ASCII风格的文字

&#160; &#160; &#160; &#160;用搜索引擎搜索ASCII ART，会发现大量这种网站。就说这么多。

## 把生成的欢迎文字粘贴到motd文件

&#160; &#160; &#160; &#160;重新打开终端，就能看到效果。

# 下方的系统信息

## 生成系统信息的方法

&#160; &#160; &#160; &#160;推荐一个非常好用的命令，叫`archey`。mac用户推荐使用[archey-osx](https://github.com/obihann/archey-osx)，其他平台用户推荐[archey.js](https://github.com/mixu/archey.js)(archey.js是node.js写的跨平台工具，mac也可以使用)。

&#160; &#160; &#160; &#160;如果希望和我的一样，最后一行显示系统时间，可以参考[我修改过的archey-osx](https://github.com/THISISAGOODNAME/archey-osx)。非常简单，就是个时间处理。

## 显示系统信息

&#160; &#160; &#160; &#160;上文已经提到了，motd文件只能显示静态文本。所以，有以下两种方法。

### 定制一个计划任务，定时修改motd文件

&#160; &#160; &#160; &#160;这个貌似还是国外大神推荐的方案。

&#160; &#160; &#160; &#160;展示一下他的终端。

![国外大神](http://www.mewbies.com/motd_images/mewbies_best_motd_in_color.png)

&#160; &#160; &#160; &#160;顺便帖一下[他的教程](http://www.mewbies.com/how_to_customize_your_console_login_message_tutorial.htm)

> 计划任务看上去很不错，但是不停运行，还是会消耗一定的性能。而且，我最不能忍的是显示的时间不准。所以，我个人趋向，打开终端时运行archey命令

### 打开终端时运行archey命令

&#160; &#160; &#160; &#160;设计打开终端时执行的命令非常简单，拿zsh为例，在`.zshrc`文件中，新建一行，加上`PATH/TO/archey`即可。

&#160; &#160; &#160; &#160;这种方法，可以避免计划任务带来的性能浪费，缺点是打开终端时运行archey命令，会稍微拖慢一点终端加载的速度。孰优孰劣，请各自判断吧。
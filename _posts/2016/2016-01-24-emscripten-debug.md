---
layout: post
title: "emscripten debug"
description: "emscripten debug"
category: emscripten
tags: [html5,c++,emscripten]
---

&#160; &#160; &#160; &#160;还是惯例，看个demo，这次这个不是我弄的，是一位大神弄的选项卡demo

<p data-height="331" data-theme-id="0" data-slug-hash="EPQZYW" data-default-tab="result" data-user="andytran" data-preview="true" class='codepen'>See the Pen <a href='http://codepen.io/andytran/pen/EPQZYW/'>Simple Card Carousel</a> by Andy Tran (<a href='http://codepen.io/andytran'>@andytran</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

<!-- more -->

* Table of Contents
{:toc}

&#160; &#160; &#160; &#160;回归正文，今天中文，阅读吴同学向我推荐的一篇blog时，看见了调试的事。

&#160; &#160; &#160; &#160;emscripten调试代码非常简单，下面就是简单步骤。

# 创建测试用文件 debug_test.c

&#160; &#160; &#160; &#160;创建debug_test.c文件，文件内容我写了如下(请别吐槽低端，就是用来测试一下调试器)

{% highlight c++ %}
#include <stdio.h>
int main(int argc, char const *argv[])
{
	int a;
	int b = 2;
	int c;
	printf("hello world\n");
	printf("hello world2\n");

	a = 4;
	c = a+b;
	printf("%d\n", c);

	b++;
	printf("%d\n", b);

	return 0;
}
{% endhighlight %}

# 编译文件，生成source map

&#160; &#160; &#160; &#160;运行如下命令，编译debug_test.c，并生成调试用的source map

<pre><code>
emcc debug_test.c -o debug_test.html -g4
</code></pre>

> 其中，`-g4`的作用就是生成source map。浏览器调试时source map非常有用，比如调试压缩后的JavaScript代码，调试typescript、coffeescript、livescript等编译生成js的代码，调试less和sass代码

# 在浏览器中打开网页

&#160; &#160; &#160; &#160;**请注意:**在<span style="color:red;">chrome浏览器</span>中调试source map的话，**需要启动server**

![chrome调试截图](http://img17.poco.cn/mypoco/myphoto/20160124/20/17800049220160124203624020.png)
<center>chrome调试截图</center>

&#160; &#160; &#160; &#160;然后就可以和在浏览器中调试JavaScript一样调试emscripten了。而且右侧还可以监视变量，只不过所有局部变量名前面都多了一个`$`
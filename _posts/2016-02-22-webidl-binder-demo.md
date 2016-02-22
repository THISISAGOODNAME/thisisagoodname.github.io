---
layout: post
title: "webidl binder demo"
description: "webidl binder demo"
category: emscripten
tags: [html5,c++,emscripten]
---

&#160; &#160; &#160; &#160;依旧是惯例

<p data-height="350" data-theme-id="0" data-slug-hash="JGNvvv" data-default-tab="result" data-user="THISISAGOODNAME" data-preview="true" class='codepen'>See the Pen <a href='http://codepen.io/THISISAGOODNAME/pen/JGNvvv/'>Aurora Borealis</a> by 攻伤菊菊长 (<a href='http://codepen.io/THISISAGOODNAME'>@THISISAGOODNAME</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

<!-- more -->

* Table of Contents
{:toc}

&#160; &#160; &#160; &#160;前天写了篇webidl相关的文章，觉得写得不够清楚，这次写一个完整实例

# 创建Bar class

&#160; &#160; &#160; &#160;在c++代码中创建Bar类

{% highlight cpp %}
//
//Bar.h
//
#ifndef CLASS_BAR_H
#define CLASS_BAR_H


class Bar {
public:
    Bar(long val);
    int doSomething();
    int addSum(int a, int b);
};


#endif //CLASS_BAR_H
{% endhighlight %}


{% highlight cpp %}
//
//Bar.cpp
//
#include "Bar.h"

Bar::Bar(long val) {

}

int Bar::doSomething() {
    return 233;
}

int Bar::addSum(int a, int b) {
    return a+b;
}
{% endhighlight %}

# 创建IDL文件

&#160; &#160; &#160; &#160;创建一个IDL文件，文件名任意，我以*my_classes.idl*为例

{% highlight cpp %}
interface Bar {
        void Bar(long val);
        long doSomething();
        long addSum(long a, long b);
};
{% endhighlight %}

# 编译idl文件生成胶水文件

&#160; &#160; &#160; &#160;运行*tools/webidl_binder.py*脚本

{% highlight bash %}
python $EMSCRIPTEN/tools/webidl_binder.py my_classes.idl glue
{% endhighlight %}

> $EMSCRIPTEN指*emscripten的根目录*

&#160; &#160; &#160; &#160;运行完*webidl_binder.py*脚本之后，文件夹下会生成**glue.cpp**和**glue.js**两个文件

# 创建一个文件包含glue.cpp

&#160; &#160; &#160; &#160;创建一个cpp文件，文件名任意，我以*my_glue_wrapper.cpp*为例

&#160; &#160; &#160; &#160;在该文件中包含**glue.cpp**和**项目的头文件**，比如在本例中，*my_glue_wrapper.cpp*的内容应该为

{% highlight cpp %}
//
//my_glue_wrapper.cpp
//
#include "Bar.h"
#include "glue.cpp"
{% endhighlight %}

# 编译项目

&#160; &#160; &#160; &#160;编译工程，记得添加*my_glue_wrapper.cpp*和*glue.js*

{% highlight bash %}
emcc Bar.cpp my_glue_wrapper.cpp --post-js glue.js -o output.js
{% endhighlight %}

# 测试

&#160; &#160; &#160; &#160;创建一个html文件，记得包含output.js

{% highlight html %}
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>webidl test</title>
		<script src="output.js"></script>
	</head>
	<body>
		请打开开发者工具来进行测试
	</body>
</html>
{% endhighlight %}

&#160; &#160; &#160; &#160;之后，可以和在向JS对象相同的方法来使用编译的C++对象

![在JavaScript中调用编译的C++对象](http://7xqrar.com1.z0.glb.clouddn.com/QQ20160222-0@2x.png)
---
layout: post
title: "bullet到ammo.js编译过程实录"
description: "bullet到ammo.js编译过程实录"
category: html5
tags: [html5,物理引擎,c++]
---
* Table of Contents
{:toc}

&#160; &#160; &#160; &#160;Bullet是一个开源的物理模拟计算引擎，世界三大物理模拟引擎之一（另外两种是Havok和PhysX）。广泛应用于游戏开发和电影制作中。

&#160; &#160; &#160; &#160;开源的框架有个特点，就是出现了一个新兴平台之后，必然完成快速移植，在好基友box2D移植web之后，bullet移植到web上的需求也就越发高涨。

<!-- more -->

&#160; &#160; &#160; &#160;到此，我不得不提一下Mozilla的Alon Zakai大神，他在Github上的网名是[kripken](https://github.com/kripken)。他本人便是emscripten的原作者，并且把bullet，box2D，libxml，sqlite，zlib,llvm，luaVM，jvm等移植到了web上。

&#160; &#160; &#160; &#160;下面还是把重点放到ammo.js上。

## ammo.js简介

&#160; &#160; &#160; &#160;ammo是Avoided Making My Own的简写，恰恰还有子弹的意思。在官方介绍中，ammo.js是[Bullet physics engine](http://bulletphysics.org/) 使用emscripten的直接移植，没有重写bullet的源代码。但是在较新的ammo.js版本中，使用了名为[WebIDL](http://kripken.github.io/emscripten-site/docs/porting/connecting_cpp_and_javascript/WebIDL-Binder.html)的技术(写法和在C#中调用C++很类似)，有点类似胶水语言，让ammo.js的API更加好用。

## 编译过程

### 1. 配置emscripten sdk

&#160; &#160; &#160; &#160;在前面的博文中已经提到，不再赘述。

### 2. 获取ammo.js源代码

{% highlight bash %}
git clone https://github.com/kripken/ammo.js.git
{% endhighlight %}

&#160; &#160; &#160; &#160;可能有人会问，我说下bullet的源代码，为何选择ammo.js的。之前说过了，ammo.js 使用了WebIDL技术，为了避免咱们自己去写ammo.idl文件，也可以直接使用一个靠谱的编译脚本。

### 3. 第一次编译

&#160; &#160; &#160; &#160;进入ammo.js，输入

{% highlight bash %}
python make.py
{% endhighlight %}

&#160; &#160; &#160; &#160;我在编译过程中，遇到了下面图片中的问题(发生在Stage 2)

![第一次编译失败](http://img17.poco.cn/mypoco/myphoto/20160103/19/17800049220160103190815086.png)

&#160; &#160; &#160; &#160;WTF！源代码有错，不会吧，有错的话别人是怎么编译的。

&#160; &#160; &#160; &#160;这个问题我通过github issue解决了。这个错误是因为emsdk版本过低导致。

&#160; &#160; &#160; &#160;再WTF！已经安装了latest版的emsdk，还版本过低。这是因为latest版本是最新的预编译版本，不是最新版，而且emsdk还有未来版，等会会讲到。但是，非预编译版就要下源码编译了。

### 4. 安装incoming版本的emsdk

&#160; &#160; &#160; &#160;进入emsdk_protable目录。可以先执行

{% highlight bash %}
./emsdk list
{% endhighlight %}

查看可以选择安装的版本，之后运行

{% highlight bash %}
./emsdk install sdk-incoming-64bit
{% endhighlight %}

下载最新版本的源代码并编译emscripten 1.35.14(截止本文编写预编译版最新版本是 1.35.0)和llvm 3.8

> 请注意，llvm 3.8的编译过程非常漫长，编译后的体积也非常大，在我的机子上达到了7.17g之多。所以，仅限ammo.js，其他时候的开发我个人还是建议还是用1.35.0的预编译版

&#160; &#160; &#160; &#160;在编译完成后，别忘了激活incoming版本的emsdk

{% highlight bash %}
./emsdk activate sdk-incoming-64bit
source emsdk_env.sh
{% endhighlight %}

### 5. 第二次编译

&#160; &#160; &#160; &#160;还是到ammo.js文件夹下，执行

{% highlight bash %}
python make.py
{% endhighlight %}

&#160; &#160; &#160; &#160;这次Stage 2能过了，但是在Stage 3会发生下图的错误

![第二次编译失败](http://img17.poco.cn/mypoco/myphoto/20160103/19/17800049220160103190849056.png)

&#160; &#160; &#160; &#160;这个错误比较好解决，如同他给的解决方案，到bullet文件夹下，执行

{% highlight bash %}
./autogen.sh
{% endhighlight %}

如下图

![第二次编译失败解决](http://img17.poco.cn/mypoco/myphoto/20160103/19/17800049220160103190926082.png)

### 6. 第三次编译

&#160; &#160; &#160; &#160;在`autogen.sh`执行完成之后，回到ammo.js目录，运行

{% highlight bash %}
python make.py
{% endhighlight %}

![第三次编译1](http://img17.poco.cn/mypoco/myphoto/20160103/19/17800049220160103191031012.png)

&#160; &#160; &#160; &#160;可以往下编译了，好激动。

![第三次编译2](http://img17.poco.cn/mypoco/myphoto/20160103/19/17800049220160103191056012.png)

&#160; &#160; &#160; &#160;完成，热泪盈眶。在ammo.js的builds文件夹下，会出现temp.js。temp.js就是我们编译的结果。

### 7. 测试

&#160; &#160; &#160; &#160;在ammo.js文件夹下，执行

{% highlight bash %}
python -m SimpleHTTPServer
{% endhighlight %}

然后浏览器中打开 [http://localhost:8000/examples/webgl_demo_softbody_cloth/](http://localhost:8000/examples/webgl_demo_softbody_cloth/) ，看到飘扬的旗帜。

#### 可选测试

&#160; &#160; &#160; &#160;如果之前安装过SpiderMonkey的话，可以运行

{% highlight bash %}
python test.py
{% endhighlight %}

来对你的temp.js进行完整的测试

> 你可能会疑问，为什么不用node.js来进行测试，node.js的javascript引擎是Google的V8，v8并不支持asm.js，而emscripten编译出的javascript都是符合asm.js的，使用node.js(或者说v8)不能测试出ammo.js的真正性能

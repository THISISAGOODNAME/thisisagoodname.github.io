---
layout: post
title: "原生webGL使用shader"
description: "原生webGL使用shader"
category: html5
tags: [html5,canvas,webGL]
---

* Table of Contents
{:toc}

&#160; &#160; &#160; &#160;虽然OpenGL能够满足大多数计算机图形应用，在某些情况下，它可能不是最好的解决办法，这就是为什么OpenGL的API催生了另外两个API。第一个是OpenGL ES，‘ES’意为嵌入式子系统，他是桌面版openGL的裁剪版，在系统资源相对缺乏的嵌入式设备中使用，例如移动电话、平板电脑电视和其他彩色屏幕设备。
另一个API就是WebGL，这使得OpenGL风格的函数能够在大多数的web浏览器下通过javascript调用。

<!-- more -->

##WebGL简介

&#160; &#160; &#160; &#160;WebGL通过提高性能，以及HTML5的Canvas元素中的3D渲染功能将OpenGL(更确切的说，OpenGL ES 2.0)带入互联网浏览器之中。OpenGL ES 2.0版的所有功能都可以找到他们的确切形式，除了因为使用JavaScript接口带来的必要小变化。

&#160; &#160; &#160; &#160;WebG现在可以在几乎所有现代Web浏览器上工作(微软的IE10记以前版本需要一个插件，IE11毫无问题，而且IE11的webGL性能是极其惊人的)。本例只注重渲染，不讨论事件处理和交互。

##在一个HTML5页面中设置WebGL

&#160; &#160; &#160; &#160;为了给WebGL提供一个“窗口”用于渲染，首先在网页创建一个HTML5 Canvas元素。在这个例子中，设置他的id为gl-canvas。在后面要使用它的id来初始化WebGL。

{% highlight html %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>webGL shaders</title>

    <style type="text/css">
        canvas { background: blue; }
    </style>
</head>

<body>
<canvas id="gl-canvas" width="512" height="512">
    您的浏览器不支持canvas
</canvas>
</body>
</html>
{% endhighlight %}

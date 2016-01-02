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

&#160; &#160; &#160; &#160;假设这个能够在浏览器上正常工作，现在woman可以继续进行下一步：创建一个WebGL上下文。有多种方法可以做到这一点，但是在这里我们将使用Khronos组织提供的一个公共方法，[webgl-utils.js](https://www.khronos.org/registry/webgl/sdk/demos/common/webgl-utils.js)。它可以非常方便的吧嵌入到你的WebGL程序里。它包括WebGLUtils(类似GLUT在openGL中的低位)及其扩展方法setupWebGL()，其中它可以很容易的在一个HTML5画布中启用WebGL。

&#160; &#160; &#160; &#160;下面的例子展示了建立一个WebGL上下文，在所有支持WebGL的浏览器上都可以很好的工作。从setupWebGL()的返回值是一个JavaScript对象，它包含WebGL支持的所有OpenGL函数方法。

{% highlight html %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>webGL shaders</title>

    <style type="text/css">
        canvas {background: blue;}
    </style>

</head>

<script src="https://www.khronos.org/registry/webgl/sdk/demos/common/webgl-utils.js"></script>

<script>
    var canvas;
    var gl;

    window.onload = init;

    function init() {
        canvas = document.getElementById("gl-canvas");

        gl = WebGLUtils.setupWebGL(canvas);
        if(!gl) {alert("webGL不可用")}

        gl.viewport(0, 0, canvas.width, canvas.height);
        gl.clearColor(1.0, 0.0, 0.0, 1.0);
        gl.clear(gl.COLOR_BUFFER_BIT);
    }
</script>

<body>

<canvas id="gl-canvas" width="512" height="512">
    您的浏览器不支持webGL
</canvas>

</body>
</html>
{% endhighlight %}

&#160; &#160; &#160; &#160;上例所指定在加载页面时(通过`window.onload = init;`)执行一个`init()`函数。init()函数获取gl-canvas的id，并把它传递给setupWebGL()，他将返回一个WebGL对象，可以用来判断初始化是否成功；返回false，则提示错误信息。架设webGL是可用的，我们将设置webGL的一些状态，并清除窗口——一旦webGL接管画布，所有内容由WebGL控制，并且画面变成红色。

&#160; &#160; &#160; &#160;现在知道了WebGL已经得到支持了，接下来要扩展实例程序、初始化指定着色器、设置顶点缓冲，以及最后渲染。

##初始化WebGL里的着色器

&#160; &#160; &#160; &#160;WebGL基于openGL ES2.0版，因此WebGL是一个基于着色器的API，和openGL一样，要求每个应用程序使用顶点和片段着色器来实现渲染，所以，你会遇到加载着色器的需求，和在openGL中一样。

&#160; &#160; &#160; &#160;在WebGL的应用程序要包含顶点着色器和片段着色器，最简单的方法就是直接包含在HTML页面中，这个页面需要被正确的标注出来。与WebGL的着色器相关的两个mime类型，如下表所示。


| &lt;script>标记类型  | 着色器类型 | 
|:-----------------:  |:-------:|
| x-shader/x-vertex   | 顶点着色器   | 
| x-shader/x-fragment | 片段着色器   | 


&#160; &#160; &#160; &#160;对于WebGL程序，下面为主要的HTML页面，包括着色器源文件。他还引用了两个javascript文件，作用如下：


* demo.js，其中包括应用程序的JavaScript实现(包括init函数的最终实现)
* InitShaders.js，一个用来加载着色器的辅助函数，类似于LoadShaders()程序


&#160; &#160; &#160; &#160;html程序源文件


{% highlight html %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>webGL shaders</title>

    <style type="text/css">
        canvas {background: blue;}
    </style>

</head>

<script id="vertex-shader" type="x-shader/x-vertex">
    #ifdef GL_ES
    precision highp float;
    #endif

    attribute vec4 vPos;
    attribute vec2 vTexCoord;
    uniform float uFrame;  //帧数
    varying vec2 texCoord;

    void main() {
        float angle = radians(uFrame);
        float c = cos(angle);
        float s = sin(angle);

        mat4 m = mat4(1.0);

        m[0][0] = c;
        m[0][1] = s;
        m[1][1] = c;
        m[1][0] = -s;

        texCoord = vTexCoord;
        gl_Position = m * vPos;
    }
</script>

<script id="fragment-shader" type="x-shader/x-fragment">
    #ifdef GL_ES
    precision highp float;
    #endif

    uniform sampler2D uTexture;
    varying vec2 texCoord;

    void main() {
        gl_FragColor = texture2D(uTexture, texCoord);
    }
</script>

<script src="https://www.khronos.org/registry/webgl/sdk/demos/common/webgl-utils.js"></script>
<script src="InitShaders.js"></script>
<script src="demo.js"></script>

<body>
<canvas id="gl-canvas" width="512" height="512">
    您的浏览器不支持canvas
</canvas>
</body>
</html>
{% endhighlight %}

&#160; &#160; &#160; &#160;为了简化编译和链接WebGL的着色器代码，创建了一个InitShaders()程序，但是没有加载glsl文件；着色器定义在了HTML页面源代码中。为了组织好代码，创建InitShaders.js文件。


##WebGL着色器载入器：InitShaders.js


{% highlight javascript %}
//
//InitShaders.js
//
function InitShaders(gl, vertexShaderId, fragmentShaderId) {
    var vertShdr;
    var fragShdr;

    var vertElem = document.getElementById(vertexShaderId);
    if (!vertElem) {
        alert("Unable to load vertex shader : " + vertexShaderId);
        return -1;
    } else {
        vertShdr = gl.createShader(gl.VERTEX_SHADER);
        gl.shaderSource(vertShdr, vertElem.text);
        gl.compileShader(vertShdr);
        if (!gl.getShaderParameter(vertShdr, gl.COMPILE_STATUS)) {
            var msg = "Vertex shader failed to compile." +
                "The error log is :" +
                "<pre>" + gl.getShaderInfoLog(vertShdr) + "</pre>";
            alert(msg);
            return -1;
        }
    }

    var fragElem = document.getElementById(fragmentShaderId);
    if (!fragElem) {
        alert("Unable to load fragment shader " + fragmentShaderId);
        return -1;
    } else {
        fragShdr = gl.createShader(gl.FRAGMENT_SHADER);
        gl.shaderSource(fragShdr, fragElem.text);
        gl.compileShader(fragShdr);
        if (!gl.getShaderParameter(fragShdr, gl.COMPILE_STATUS)) {
            var msg = "Vertex shader failed to compile." +
                "The error log is :" +
                "<pre>" + gl.getShaderInfoLog(fragShdr) + "</pre>";
            alert(msg);
            return -1;
        }
    }

    var program = gl.createProgram();
    gl.attachShader(program, vertShdr);
    gl.attachShader(program, fragShdr);
    gl.linkProgram(program);

    if (!gl.getProgramParameter(program, gl.LINK_STATUS)) {
        var msg = "Shader program failed to link." +
            "The error log is : " +
            "<pre>" + gl.getProgramInfoLog(program) + "</pre>";
        return -1;
    }

    return program;
}
{% endhighlight %}

&#160; &#160; &#160; &#160;虽然InitShaders()采用HTML元素的ID寻找顶点着色器和片段着色器的代码。返回的是传递到glUseProgram()的程序名称。

&#160; &#160; &#160; &#160;有了编译和链接到着色器的方法，就可以继续初始化图形数据、加载纹理，并完成WebGL程序的其他部分。

##WebGL初始化顶点数据

&#160; &#160; &#160; &#160;WebGL带给JavaScript的一个显著特征就是类型化数组(typed arrays)，它扩展为JavaScript数组概念，并满足OpenGL数据类型风格，几种类型化数组的类型如下表所示。



| 数组类型         |  C类型        |
|:---------------:|:------------:|
|Int8Array        |signed char   |
|Uint8Array       |unsigned char |
|Uint8ClampedArray|unsigned char |
|Int16Array       |signed short  |
|Uint16Array      |unsigned short|
|Int32Array       |signed int    |
|Uint32Array      |unsigned int  |
|Float32Array     |float         |
|Float64Array     |double        |



&#160; &#160; &#160; &#160;第一次需要分配和填充(这两者可以在一个单一的操作里面做)一个类型化数组来存储顶点数据。在此之后，设置VBO即可，和在openGL中是一样的。

##初始化WebGL里的顶点缓冲


{% highlight javascript %}
var verticles = {};
verticles.data = new Float32Array([
    -0.5, -0.5,
     0.5, -0.5,
     0.5,  0.5,
    -0.5,  0.5
]);

verticles.bufferId = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, verticles.bufferId);
gl.bufferData(gl.ARRAY_BUFFER, verticles.data, gl.STATIC_DRAW);
var vPos = gl.getAttribLocation(program, "vPos");
gl.vertexAttribPointer(vPos, 2, gl.FLOAT, false, 0, 0);
gl.enableVertexAttribArray(vPos);
{% endhighlight %}


##在WebGL中使用纹理贴图


&#160; &#160; &#160; &#160;在webGL中使用纹理和openGL一样，但加载处理要简单得多，因为有HTML的帮助。事实上，从文件加载纹理仅仅需要一行代码即可完成。比如，加载一个名为的单一纹理。


{% highlight javascript %}
var image = new Image();
image.src = "material.jpg";
{% endhighlight %}


&#160; &#160; &#160; &#160;HTML加载图片虽然简单，然而，他是异步加载的，所以要知道该图像文件何时接受并载入，可以在回调中处理。JavaScript在图像中有一个处理这种情况的方法：`onload()`


{% highlight javascript %}
image.onload = function () {
    configureTexture(image);
    render();
};
{% endhighlight %}


&#160; &#160; &#160; &#160;上面给出的onload函数实在图像被完全加载并且可以被WebGL使用的时候才调用一次。我们可以将所以的纹理初始化代码都封装到一个本地函数configureTexture中。


{% highlight javascript %}
function configureTexture(image) {
    texture = gl.createTexture();
    gl.activeTexture(gl.TEXTURE0);
    gl.bindTexture(gl.TEXTURE_2D, texture);
    gl.pixelStorei(gl.UNPACK_FLIP_Y_WEBGL, true);
    gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGB, gl.RGB, gl.UNSIGNED_BYTE, image);
    gl.generateMipmap(gl.TEXTURE_2D);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.NEAREST_MIPMAP_LINEAR);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.NEAREST);
}
{% endhighlight %}


&#160; &#160; &#160; &#160;WebGL扩展的glPixelStore*()用于翻转图像数据，WebGL的宏UNPACK_FLIP_Y_WEBGL是旋转图像数据来匹配WebGL时需要的。


> 由于OpenGL ES 2.0版本的原因，WebGL只支持分辨率为2的幂的纹理。


## demo.js的完整源文件


{% highlight javascript %}
//
//demo.js
//
var canvas;
var gl;
var texture;
var uFrame;

window.onload = init;

function CheckError(msg) {
    var error = gl.getError();
    if (error != 0) {
        var errMsg = "OpenGL error: " + error.toString(16);
        if (msg) {errMsg = msg + "\n" + errMsg; }
        alert(errMsg);
    }
}

function configureTexture(image) {
    texture = gl.createTexture();
    gl.activeTexture(gl.TEXTURE0);
    gl.bindTexture(gl.TEXTURE_2D, texture);
    gl.pixelStorei(gl.UNPACK_FLIP_Y_WEBGL, true);
    gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGB, gl.RGB, gl.UNSIGNED_BYTE, image);
    gl.generateMipmap(gl.TEXTURE_2D);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.NEAREST_MIPMAP_LINEAR);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.NEAREST);
}

function init() {
    canvas = document.getElementById("gl-canvas");

    gl = WebGLUtils.setupWebGL(canvas);
    if(!gl) {alert("WebGL is not available");}

    gl.viewport(0, 0, canvas.width, canvas.height);
    gl.clearColor(1.0, 0.0, 0.0, 1.0);

    //
    // 读取着色器并初始化属性数组
    //

    var program = InitShaders(gl, "vertex-shader", "fragment-shader");
    gl.useProgram(program);

    var verticles = {};
    verticles.data = new Float32Array([
        -0.5, -0.5,
         0.5, -0.5,
         0.5,  0.5,
        -0.5,  0.5
    ]);

    verticles.bufferId = gl.createBuffer();
    gl.bindBuffer(gl.ARRAY_BUFFER, verticles.bufferId);
    gl.bufferData(gl.ARRAY_BUFFER, verticles.data, gl.STATIC_DRAW);
    var vPos = gl.getAttribLocation(program, "vPos");
    gl.vertexAttribPointer(vPos, 2, gl.FLOAT, false, 0, 0);
    gl.enableVertexAttribArray(vPos);

    var texCoords = {};
    texCoords.data = new Float32Array([
        0.0, 0.0,
        1.0, 0.0,
        1.0, 1.0,
        0.0, 1.0
    ]);

    texCoords.bufferId = gl.createBuffer();
    gl.bindBuffer(gl.ARRAY_BUFFER, texCoords.bufferId);
    gl.bufferData(gl.ARRAY_BUFFER, texCoords.data, gl.STATIC_DRAW);
    var vTexCoord = gl.getAttribLocation(program, "vTexCoord");
    gl.vertexAttribPointer(vTexCoord, 2, gl.FLOAT, false, 0, 0);
    gl.enableVertexAttribArray(vTexCoord);

    //
    // 初始化纹理
    //

    var image = new Image();
    image.onload = function () {
        configureTexture(image);
        render();
    };
    image.src = "material.jpg";

    gl.activeTexture(gl.TEXTURE0);
    var uTexture = gl.getUniformLocation(program, "uTexture");
    gl.uniform1i(uTexture, 0);

    uFrame = gl.getUniformLocation(program, "uFrame");
}

var frameNumber = 0;

function render() {
    gl.uniform1f(uFrame, frameNumber++);

    gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);
    gl.drawArrays(gl.TRIANGLE_FAN, 0, 4);

    window.requestAnimFrame(render, canvas);
}
{% endhighlight %}
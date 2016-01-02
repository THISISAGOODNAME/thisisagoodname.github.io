---
layout: post
title: "jThree简易教程"
description: "jThree简易教程"
category: html5
tags: [html5,mmd]
---

* Table of Contents
{:toc}

&#160; &#160; &#160; &#160;本文是为“觉得JavaScript本身很难”的人写的WebGL CG入门指南。

&#160; &#160; &#160; &#160;WebGL有很长的学习周期，前期学习曲线不算平坦，GLSL无法理解，矩阵&矢量也难以驾驭。。。

&#160; &#160; &#160; &#160;WebGL使浏览器可以实现绚丽的3DCG，但同时也让很多新手遇到挫折。

&#160; &#160; &#160; &#160;太难了！

<!-- more -->

&#160; &#160; &#160; &#160;也有人会说，"用Three.js的话，很简单嘛。"确实Three.js先比原生webGL容易几十倍，但在那之前还有一个更尖锐的问题，那就是——"JavaScript本身很难"。

&#160; &#160; &#160; &#160;本文就是为**认为JavaScript很难的人**写的WebGL 3D CG入门指南。只有高级程序员才能享受WebGL的话，对人类来说只能是损失。也希望我能把编程的快乐和感动传达给你们。

&#160; &#160; &#160; &#160;[**项目源码地址**](http://aicdg.com/)

##开发环境搭建

&#160; &#160; &#160; &#160;这次使用[jThree](http://jthree.jp/)库来开发，jThree库基于[Three.js](http://threejs.org/)，但屏蔽了大量内部细节，让美工人生也可以轻松实现漂亮的3D CG场景。

&#160; &#160; &#160; &#160;下载jThree，地址[http://jthree.jp/](http://jthree.jp/)，将压缩包在某个空文件夹下打开，得到如下文件结构。

![解压jThree](http://img17.poco.cn/mypoco/myphoto/20150910/19/17800049220150910195530024.png)

&#160; &#160; &#160; &#160;然后在libs文件夹下新建文件夹jquery，下载jquery.min.js(版本不低于2.0.0)，复制到该文件夹下。

&#160; &#160; &#160; &#160;新建四个文件，分别文件名为index.css、index.goml、index.html、index.js。之后文件夹结构如下图所示。

![新建index](http://img17.poco.cn/mypoco/myphoto/20150910/19/17800049220150910195547065.png)

&#160; &#160; &#160; &#160;四个文件各自的内容如下
index.css

{% highlight css %}
body {
	position: fixed;
	width: 100%;
	height: 100%;
	margin: 0;
}

#loading {
	position: absolute;
	top: 0;
	left: 0;
	width: 100%;
	height: 100%;
	background-color: rgba( 0, 0, 0, .7 );
	background-image: url( "img/loading.gif" );
	background-position: center;
	background-repeat: no-repeat;
}
{% endhighlight %}

index.goml

{% highlight xml %}
<goml>
    <head>
        <rdr frame="body" camera="camera:first" param="antialias: true; clearColor: #FFF;" />
    </head>
    <body>
        <scene>
            <camera style="cameraFar: 10000; position: 2 18 30; lookAtY: 10;">
                <light type="Dir" style="position: 1 3 5;" />
            </camera>
            <light type="Amb" style="lightColor: #555;" />

        </scene>
    </body>
</goml>
{% endhighlight %}

index.html

{% highlight html %}
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>メルト-初音ミク</title>
<link rel="stylesheet" href="index.css">
<script src="libs/jquery/jquery.min.js"></script>
<script src="libs/jthree/2.1/jThree.min.js"></script>
<script src="libs/stats/1.2/jThree.Stats.js"></script>
<script src="libs/trackball/1.4/jThree.Trackball.js"></script>
<script src="libs/ammo/1.0/ammo.js"></script>
<script src="libs/mmd/1.5/jThree.MMD.js"></script>
<script src="libs/xfile/1.1/jThree.XFile.js"></script>
<script src="index.goml" type="text/goml"></script>
<script src="index.js"></script>
</head>
<body>
	<div id="loading"></div>
</body>
</html>
{% endhighlight %}

index.js

{% highlight javascript %}
jThree( function( j3 ) {

	$( "#loading" ).remove();
	
	j3.Trackball();
	j3.Stats();

},
function() {
	alert( "您的浏览器不支持webGL" );
} );
{% endhighlight %}

&#160; &#160; &#160; &#160;第一个实例就算完成了，webGL要访问众多格式，浏览器一般封锁访问本地文件(为了系统安全)，所以本地服务器是必要的。会配置本地服务器的可以使用Apache lighttpd或者nginx，不会的推荐使用python2.7，在项目所在文件夹下执行

{% highlight bash %}
python -m SimpleHTTPServer
{% endhighlight %}

然后访问localhost:8000即可。

运行结果如下所示

![demo项目运行结果](http://img17.poco.cn/mypoco/myphoto/20150910/19/17800049220150910195507017.png)

##GOML文件

&#160; &#160; &#160; &#160;GOML文件单列出来，作为jThree的场景格式，以此来和网页分开。我们的3D CG场景也基本都在GOML文件中完成，不碰或者尽可能少碰html、css和javascript。

##以MMD经典人物初音未来开始

&#160; &#160; &#160; &#160;在scene便签中添加一行`<mmd model="model/miku/index.pmx" />`，之后index.goml文件如下，如果GOML文件没有高亮，可以手动设置编辑器的高亮模式为xml。

{% highlight xml %}
<goml>
    <head>
        <rdr frame="body" camera="camera:first" param="antialias: true; clearColor: #fff;" />
    </head>
    <body>
        <scene>
            <mmd model="model/miku/index.pmx" />

            <camera style="cameraFar: 10000; position: 2 18 30; lookAtY: 10;">
                <light type="Dir" style="position: 1 3 5;" />
            </camera>
            <light type="Amb" style="lightColor: #555;" />

        </scene>
    </body>
</goml>
{% endhighlight %}

&#160; &#160; &#160; &#160;然后重启服务器，python的话，可以`ctrl+c`关闭服务器再重开。结果如下

![miku](http://img17.poco.cn/mypoco/myphoto/20150910/20/17800049220150910202725037.png)

&#160; &#160; &#160; &#160;如果把`<mmd model="model/miku/index.pmx" />`变为`<mmd model="model/neru/index.pmx" />`，刷新，可以看到场景变为如下

![neru](http://img17.poco.cn/mypoco/myphoto/20150910/20/17800049220150910202751084.png)

##为你的MMD添加舞台

&#160; &#160; &#160; &#160;纯白色有些太单调了，添加一条街道吧。代码同样只有一行，非常简单。把obj标签添加到scene标签中。
`<obj model="stage/gekido/index.x" style="scale: 10; positionY: -46.5; rotateY: 1.57;"/>`

之后的index.goml如下

{% highlight xml %}
<goml>
    <head>
        <rdr frame="body" camera="camera:first" param="antialias: true; clearColor: #fff;" />
    </head>
    <body>
        <scene>
            <mmd model="model/miku/index.pmx" />
            <obj model="stage/gekido/index.x" style="scale: 10; positionY: -46.5; rotateY: 1.57;"></obj>

            <camera style="cameraFar: 10000; position: 2 18 30; lookAtY: 10;">
                <light type="Dir" style="position: 1 3 5;" />
            </camera>
            <light type="Amb" style="lightColor: #555;" />

        </scene>
    </body>
</goml>
{% endhighlight %}

&#160; &#160; &#160; &#160;刷新，网页变成如下图所示

![添加舞台](http://img17.poco.cn/mypoco/myphoto/20150910/20/17800049220150910203627055.png)

&#160; &#160; &#160; &#160;这一次追加了obj的三个属性

* scale：缩放系数
* positionY：Y方向上的位置
* rotateY：沿Y轴逆时针方向旋转角度

##令miku站在球体上

&#160; &#160; &#160; &#160;先删除刚才添加的场景(obj标签)，添加如下的mesh标签`<mesh geo="#geo1" mtl="#mtl1"></mesh>`

&#160; &#160; &#160; &#160;mesh标签是球体等一般物体的绘制标签，但单独一个mesh并不会绘制。WebGL描述物体，需要**材质**和**形状**两个信息

物体(object) = 形状(mesh) + 材质(material)

&#160; &#160; &#160; &#160;为mesh标签追加mtl属性`<mesh geo="#geo1" mtl="#mtl1"></mesh>`

&#160; &#160; &#160; &#160;然后添加geo和mtl的具体参数，这次不添加在scene标签中，而是head中。在head中添加

{% highlight xml %}
<geo id="geo1" type="Sphere" param="10 64 64" />
<mtl id="mtl1" type="MeshPhong" param="color: #0ff; specular: #fff;" />
{% endhighlight %}

之后index.goml如下

{% highlight xml %}
<goml>
    <head>
        <geo id="geo1" type="Sphere" param="10 64 64" />
        <mtl id="mtl1" type="MeshPhong" param="color: #0ff; specular: #fff;" />
        <rdr frame="body" camera="camera:first" param="antialias: true; clearColor: #fff;" />
    </head>
    <body>
        <scene>
            <mmd model="model/miku/index.pmx" />
            <mesh geo="#geo1" mtl="#mtl1"></mesh>

            <camera style="cameraFar: 10000; position: 2 18 30; lookAtY: 10;">
                <light type="Dir" style="position: 1 3 5;" />
            </camera>
            <light type="Amb" style="lightColor: #555;" />

        </scene>
    </body>
</goml>
{% endhighlight %}

结果入下图

![添加一个物体](http://img17.poco.cn/mypoco/myphoto/20150910/20/17800049220150910205338026.png)

&#160; &#160; &#160; &#160;可以发现球的位置并不太好，让球稍微往下降一点，修改mesh标签为`<mesh geo="#geo1" mtl="#mtl1" style="positionY: -10;"></mesh>`，然后结果变为下图

![下降物体](http://img17.poco.cn/mypoco/myphoto/20150910/20/17800049220150910205533051.png)

##为球体添加贴图

&#160; &#160; &#160; &#160;在head里加入如下标签`<txr id="txr1" src="img/earth.jpg" />`。图像和视频都能成为表面纹理。之后修改mtl标签`<mtl id="mtl1" type="MeshPhong" param="map: #txr1; specular: #fff;" />`

&#160; &#160; &#160; &#160;之后index.goml如下

{% highlight xml %}
<goml>
    <head>
        <txr id="txr1" src="img/earth.jpg" />
        <geo id="geo1" type="Sphere" param="10 64 64" />
        <mtl id="mtl1" type="MeshPhong" param="map: #txr1; specular: #fff;" />
        <rdr frame="body" camera="camera:first" param="antialias: true; clearColor: #fff;" />
    </head>
    <body>
        <scene>
            <mmd model="model/miku/index.pmx" />
            <mesh geo="#geo1" mtl="#mtl1" style="positionY: -10;"></mesh>

            <camera style="cameraFar: 10000; position: 2 18 30; lookAtY: 10;">
                <light type="Dir" style="position: 1 3 5;" />
            </camera>
            <light type="Amb" style="lightColor: #555;" />

        </scene>
    </body>
</goml>
{% endhighlight %}

&#160; &#160; &#160; &#160;结果如下图

![地球贴图](http://img17.poco.cn/mypoco/myphoto/20150910/21/17800049220150910210317022.png)

&#160; &#160; &#160; &#160;简单修改mtl，使用凹凸贴图，便能获得不错的地球效果。mtl改成如下`<mtl id="mtl1" type="MeshPhong" param="map: #txr1; bumpScale: 0.3;" />`

![地球凹凸贴图](http://img17.poco.cn/mypoco/myphoto/20150910/21/17800049220150910210458032.png)

##修改背景为黑色，添加太阳

&#160; &#160; &#160; &#160;修改背景色就是修改rdr标签中的clearColor属性。太阳的图像在img/sun.jpg。在head中添加

{% highlight xml %}
<txr id="txr2" src="img/sun.jpg" />
<mtl id="mtl2" type="Sprite" param="map: #txr2;" />
{% endhighlight %}

显示图片需要用sprite(精灵)，但是非常简单，只要`<sprite mtl="#mtl2" />`。

&#160; &#160; &#160; &#160;再加上一些细小的调整，index.goml最后的结果如下

{% highlight xml %}
<goml>
    <head>
        <txr id="txr1" src="img/earth.jpg" />
        <txr id="txr2" src="img/sun.jpg" />
        <geo id="geo1" type="Sphere" param="10 64 64" />
        <mtl id="mtl1" type="MeshPhong" param="map: #txr1; bumpScale: 0.3;" />
        <mtl id="mtl2" type="Sprite" param="map: #txr2;" />
        <rdr frame="body" camera="camera:first" param="antialias: true; clearColor: #000;" />
    </head>
    <body>
        <scene>
            <mmd model="model/miku/index.pmx" />
            <mesh geo="#geo1" mtl="#mtl1" style="positionY: -10;"></mesh>
            <sprite mtl="#mtl2" style="scale: 100; positionZ: -500;" />

            <camera style="cameraFar: 10000; position: 2 18 30; lookAtY: 10;">
                <light type="Dir" style="position: 1 3 5;" />
            </camera>
            <light type="Amb" style="lightColor: #555;" />

        </scene>
    </body>
</goml>
{% endhighlight %}

结果如下图所示
![背景](http://img17.poco.cn/mypoco/myphoto/20150910/21/17800049220150910211337069.png)

##添加说明文本

&#160; &#160; &#160; &#160;在三维空间中加入文字，可以先用import标签导入，然后用txr标签转换为纹理，再用mtl标签转换为材质。

&#160; &#160; &#160; &#160;在head中添加如下代码

{% highlight xml %}
<import>
    <style>
        div {
            position: absolute;
            font-weight: bold;
            color: #fff;
        }
    </style>
    <div id="name">初音ミク</div>
</import>
<txr id="txr3" html="#name" />
<mtl id="mtl3" type="Sprite" param="map: #txr3;" />
{% endhighlight %}

在scene中添加`<sprite mtl="#mtl3" style="scaleX: 3; positionY: 22;" />`

&#160; &#160; &#160; &#160;结果如下图

![添加提示文字](http://img17.poco.cn/mypoco/myphoto/20150910/21/17800049220150910212030056.png)

##给模型添加动作

&#160; &#160; &#160; &#160;给MMD模型添加动作非常简单，只要在mmd标签中添加一个motion属性指定动作文件即可。将mmd标签修改成如下

{% highlight xml %}
<mmd model="model/miku/index.pmx" motion="motion/melt.vmd" onLoad="parent.jThree.MMD.play( true );" />
{% endhighlight %}

&#160; &#160; &#160; &#160;结果如下图

![添加动作](http://img17.poco.cn/mypoco/myphoto/20150910/21/17800049220150910212501025.png)

##给场景添加音乐

&#160; &#160; &#160; &#160;这次要修改index.html文件了，在body中添加一个Audio标签，之后body变为如下

{% highlight xml  %}
<body>
	<div id="loading"></div>
	<audio controls autoplay>
		<source src="audio/メルト-初音ミク.mp3" />
	</audio>
</body>
{% endhighlight %}

##实际演示效果

<script src="http://cdn.bootcss.com/jquery/2.1.4/jquery.min.js"></script>
<script type="text/javascript">
    var i = 0;
    $(document).ready(function(){
        $("iframe").toggle();
        $(".btn1").click(function(){
            $("iframe").toggle();
            if(i%2 == 0){
                $("iframe").attr("src", "http://aicdg.com/jThreeDemo/");
            } else {
                $("iframe").attr("src", "");
            }
        });
    });
</script>
<iframe style="overflow: hidden;" width="600px" height="400px" scrolling="no"  frameborder="0"></iframe>
<button class="btn1">查看实际效果</button>
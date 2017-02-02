---
layout: post
title: "dom与web3d"
description: "dom与web3d"
category: 前端技术
tags: [前端技术,webGL,随想]
---

&#160; &#160; &#160; &#160;正式入职已经两个半月了，入职之后最大的感受，就是年头已经变了，算法遍地走，后端难找，前端更难找。前端高负荷工作差不多是不可避免的了。

<!-- more -->

* Table of Contents
{:toc}

&#160; &#160; &#160; &#160;扯了点废话，还是回到正题吧。

# webGL，有必要吗

## 装逼利器

&#160; &#160; &#160; &#160;webGL的出现的目的，说实话我还真想不出来。我猜测是用来填canvas 2d的坑的（canvas 2d的坑真是神TM多）。可惜，曾经的webGL实在是曲高和寡，和前端兄弟们的工作实在是格格不入；而图形学大神们，都在忙着pbr、posteffect或者gpgpu，没时间搞。直到[shadertoy](https://www.shadertoy.com/)出现，大神们有了在线装逼平台，webgl终于开始焕发逼格。

## 转机

&#160; &#160; &#160; &#160;总曲高和寡也不是办法，[Mr.doob](https://github.com/mrdoob)的[three.js](https://threejs.org/)出现，把固定管线重新带到webgl，让大量前端兄弟，终于可以体验webgl的美好。而且，three.js的意义，远远不仅仅是第一个比较成熟完整的webgl库那么简单。three.js的出现，激励了很多后人开发webgl封装的库，stackgl、cubicVR、Babylon.js...从结果上看，webgl的二次封装库，比openGL多得多得多。

&#160; &#160; &#160; &#160;不过，webgl对于前端来说，还是不够友好。特别是在和以前的库或框架（比如jQuery，angular，react），还是无法很好的整合在一起。但是视图与逻辑分离，这个理想貌似渐行渐远了。

# DOM可视化

&#160; &#160; &#160; &#160;基于DOM的可视化，其实已经非常成熟了，成熟到了天天用以至于大家都忘记了。其实HTML渲染引擎，本身就是DOM可视化最佳实践之一。

&#160; &#160; &#160; &#160;大家用的最多的DOM可视化，应该是SVG吧。用DOM来表示图元，并且可以用css来控制样式，可以使用jQuery进行操作。

# 基于DOM的3D场景渲染

&#160; &#160; &#160; &#160;基于DOM的3D场景渲染开始的其实非常早，可以追溯到上世纪90年代的VRML(Virtual Reality Modeling Language)，VRML的MIME类型是x-world/x-vrml，我专门提到这个，就是想说，那个年代，vrml就已经可以嵌入到网页中了（虽然只有IE实现了vrml显示，可见那个时代IE还是很潮的浏览器），升阳电脑也把vrml作为java3D的基础，成为java applet的基础，同微软的activeX和micromedia的flash直接竞争。

&#160; &#160; &#160; &#160;之后就是[X3D](http://www.x3dom.org/)了，X3D是web3D组织推出用来接替vrml的标准，X3D诞生于2000年春，实际年龄也十分大了。X3D的观念十分先进，可惜开发工具跟不上，Cosmo Worlds、ISA、Avatar Studio的先后叛逃，让X3D被人遗忘在了历史长河里。现在虽然也有web3D组织用webGL重构的库，但是X3D已经错过了最佳发展时间，也就是我这种low逼会缅怀一下。

&#160; &#160; &#160; &#160;你可能会说，collada(扩展名DAE)也是/xml，为啥不提。collada是三维模型，强调文件交换，collada的全名就是面向交互式3D应用程序的基于XML的数字资产交换方案。与建模过程更接近。而vrml/X3D强调的是场景渲染与交互。严格来说，X3D在功能上，和其竞品Quest3D、Unity3D、3D VIA Virtools、gamebryo比较，是更合适的。从某种程度上说，功能甚至比Irrlicht、ogre都要多。

# 基于DOM的场景管理在现在的尝试

&#160; &#160; &#160; &#160;没仔细调查过，应该不会太少，举两个我比较认可的吧。

## Grimoire.js

&#160; &#160; &#160; &#160;[Grimoire.js](http://grimoire.gl/)，刚刚开发到0.3版，非常早期，文档也非常不全。可惜我还是认可。作者就是jThree库的原作者。本来grimoire.js也是准备直接叫jThree V3的，估计是原作者不希望goml就停留在小搓宅男们靠mmd自娱自乐的阶段，还是希望能把goml推广出去的。

```xml
<goml>
    <head>
        <txr id="txr1" src="img/earth.jpg" />
        <txr id="txr2" src="img/sun.jpg" />
        <geo id="geo1" type="Sphere" param="10 64 64" />
        <mtl id="mtl1" type="MeshPhong" param="map: #txr1; bumpScale: 0.3;" />
        <mtl id="mtl2" type="Sprite" param="map: #txr2;" />

        <import>
            <style>
                div {
                    position: absolute;
                    font-weight: bold;
                    color: #fff;
                }
            </style>
            <div id="name">メルト - 初音ミク</div>
        </import>
        <txr id="txr3" html="#name" />
        <mtl id="mtl3" type="Sprite" param="map: #txr3;" />

        <rdr frame="body" camera="camera:first" param="antialias: true; clearColor: #000;" />
    </head>
    <body>
        <scene>
            <mmd model="model/miku/index.pmx" motion="motion/melt.vmd" onLoad="parent.jThree.MMD.play( true );" />
            <mesh geo="#geo1" mtl="#mtl1" style="positionY: -10;"></mesh>
            <sprite mtl="#mtl2" style="scale: 100; positionZ: -500;" />
            <sprite mtl="#mtl3" style="scaleX: 3; positionY: 22;" />

            <camera style="cameraFar: 10000; position: 2 18 30; lookAtY: 10;">
                <light type="Dir" style="position: 1 3 5;" />
            </camera>
            <light type="Amb" style="lightColor: #555;" />

        </scene>
    </body>
</goml>
```

&#160; &#160; &#160; &#160;这是我以前写过的一个小[demo](http://aicdg.com/jThreeDemo/)，说实话，这个goml可以看出从vrml/X3D还有jQuery上都吸收了很多思想，甚至import这么高端的操作都支持了。

## A-FRAME

&#160; &#160; &#160; &#160;[A-FRAME](https://aframe.io)严格来说不仅仅是webgl框架了，它是webVR框架，由MOZVR团队开发，到现在为止也是开发到了0.3版，不过文档非常的完善。

```html
<html>
  <head>
    <script src="https://aframe.io/releases/0.3.0/aframe.min.js"></script>
  </head>
  <body>
    <a-scene>
      <a-box color="#6173F4" opacity="0.8" depth="2"></a-box>
      <a-sphere radius="2" src="texture.png" position="1 1 0"></a-sphere>
      <a-sky color="#ECECEC"></a-sky>
    </a-scene>
  </body>
</html>
```

&#160; &#160; &#160; &#160;Mozilla这个库在文档里就提到了是web component影响下的产物(直接嵌到body里去了)，场景布局的方式，一样能看出，承自X3D，但区别也很多。A-FRAME这个库真的非常牛逼，土豪青年请拿起vive或者Oculus体验，文艺青年找个手机VR壳子，屌丝青年弄个cardboard，装(er)逼青年请直接想象。

&#160; &#160; &#160; &#160;讲真，这个A-FRAME简直是降低VR的开发门槛啊，连我这种low逼都能画个方块看着玩。

# 基于DOM可视化的优势

&#160; &#160; &#160; &#160;基于DOM，那优势必然就是可以把过去对图元的操作，改变成和前端开发的DOM操作相同的样子。想想能用jQuery、mootools等库结合使用，可以作为模板，甚至是和现在正火的react.js结合使用，你不激动吗。

> PS：react.js本身就可以操作svg，准确说web component都可以，A-FRAME的文档里有react.js操作A-FRAME的方法。

# 其他

&#160; &#160; &#160; &#160;今天的歌单

1. リリリリ★バーニングナイト
2. メグメグ☆ファイアーエンドレスナイト
3. ナナナナ★フィーバーミラクルトゥナイト
4. ミキミキ★ロマンティックナイト
5. ネコネコ☆スーパーフィーバーナイト
6. イアイア★ナイトオブデザイア
7. ルカルカ★ナイトフィーバー 

洗脑循环简直暴力，顺便缅怀samfree，一路走好

<script type="text/javascript" src="http://www.xiami.com/widget/player-multi?uid=13920853&sid=1770458988,1770019489,1770900289,1770458980,1770458978,1770900292,1770019484,&width=235&height=346&mainColor=FF8719&backColor=494949&autoplay=0&mode=js"></script>
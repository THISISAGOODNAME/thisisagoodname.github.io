---
layout: post
title: "用threejs和ammojs制作布料"
description: "用threejs和ammojs制作布料"
category: html5
tags: [html5,bullet]
---

&#160; &#160; &#160; &#160;在上一节绳索模拟之后，接下来就进入布料模拟阶段。有了绳索的基础，布料就很简单了，因为两者真的差不多。

<!-- more -->

1. bullet引擎和OpenGL结合创建简单的场景
2. three.js和ammo.js创建简单的场景
3. 创建地形
4. 制作车辆
5. 柔体-绳索
6. **柔体-布料**
7. 柔体-有体积的柔体
8. 使用blender引擎模拟物理场景

* Table of Contents
{:toc}

# 布料模拟场景

&#160; &#160; &#160; &#160;本节主要目的就是制作一块布料，取代上一节中的绳索和小球，其他完全一样的场景。

&#160; &#160; &#160; &#160;复制一份[绳索场景](https://github.com/THISISAGOODNAME/learn-ammojs/blob/master/Demo4-rope.html)，然后进行微调，这次不进行大范围删除工作。

# 全局变量的修改

&#160; &#160; &#160; &#160;把表示绳子的全局变量`var rope;`改成表示布料的全局变量`var cloth;`。

# 修改createObjects函数

## 布料绘制部分

&#160; &#160; &#160; &#160;布料的绘制部分代码如下：

```js 
// 布料的绘制部分
var clothWidth = 4;
var clothHeight = 3;
var clothNumSegmentsZ = clothWidth * 5;
var clothNumSegmentsY = clothHeight * 5;
var clothSegmentLengthZ = clothWidth / clothNumSegmentsZ;
var clothSegmentLengthY = clothHeight / clothNumSegmentsY;
var clothPos = new THREE.Vector3( -3, 3, 2 );

var clothGeometry = new THREE.PlaneBufferGeometry(clothWidth, clothHeight, clothNumSegmentsZ, clothNumSegmentsY);
clothGeometry.rotateY(Math.PI * 0.5);
clothGeometry.translate(clothPos.x, clothPos.y + clothHeight + 0.5, clothPos.x, clothWidth * 0.5);
var clothMaterial = new THREE.MeshLambertMaterial( { color: 0xFFFFFF, side: THREE.DoubleSide } );
cloth = new THREE.Mesh( clothGeometry, clothMaterial );
cloth.castShadow = true;
cloth.receiveShadow = true;
scene.add( cloth );
textureLoader.load( "./textures/grid.png", function( texture ) {
    texture.wrapS = THREE.RepeatWrapping;
    texture.wrapT = THREE.RepeatWrapping;
    texture.repeat.set( clothNumSegmentsZ, clothNumSegmentsY );
    cloth.material.map = texture;
    cloth.material.needsUpdate = true;
} );
```

&#160; &#160; &#160; &#160;和生成地形时的方法相同，将布料的各个点传到PlaneBufferGeometry即可。一块布细分程度越高，布料模拟效果越高，但是越消耗资源。

## 布料的物理部分

&#160; &#160; &#160; &#160;布料的物理部分代码如下：

```js
// 布料的物理部分
var softBodyHelpers = new Ammo.btSoftBodyHelpers();
var clothCorner00 = new Ammo.btVector3( clothPos.x, clothPos.y + clothHeight, clothPos.z );
var clothCorner01 = new Ammo.btVector3( clothPos.x, clothPos.y + clothHeight, clothPos.z - clothWidth );
var clothCorner10 = new Ammo.btVector3( clothPos.x, clothPos.y, clothPos.z );
var clothCorner11 = new Ammo.btVector3( clothPos.x, clothPos.y, clothPos.z - clothWidth );
var clothSoftBody = softBodyHelpers.CreatePatch( physicsWorld.getWorldInfo(), clothCorner00, clothCorner01, clothCorner10, clothCorner11, clothNumSegmentsZ + 1, clothNumSegmentsY + 1, 0, true );
var sbConfig = clothSoftBody.get_m_cfg();
sbConfig.set_viterations( 10 );
sbConfig.set_piterations( 10 );

clothSoftBody.setTotalMass( 0.9, false );
Ammo.castObject( clothSoftBody, Ammo.btCollisionObject ).getCollisionShape().setMargin( margin * 3 );
physicsWorld.addSoftBody( clothSoftBody, 1, -1 );
cloth.userData.physicsBody = clothSoftBody;
// Disable deactivation
var DISABLE_DEACTIVATION = 4;
clothSoftBody.setActivationState(DISABLE_DEACTIVATION);
```

&#160; &#160; &#160; &#160;**说明：**clothCorner[00/01/10/11]分别为布料四个角在空间中的坐标。clothNumSegments[Z/X]分别是沿着Z轴和X轴的细分值。

```
01 00
11 10
```

# 修改updatePhysics函数

&#160; &#160; &#160; &#160;把绳索更新的函数改为如下：

```js
// 更新布料状态
var softBody = cloth.userData.physicsBody;
var clothPositions = cloth.geometry.attributes.position.array;
var numVerts = clothPositions.length / 3;
var nodes = softBody.get_m_nodes();
var indexFloat = 0;
for (var i = 0; i <numVerts; i++) {
    var node = nodes.at(i);
    var nodePos = node.get_m_x();
    clothPositions[ indexFloat++ ] = nodePos.x();
    clothPositions[ indexFloat++ ] = nodePos.y();
    clothPositions[ indexFloat++ ] = nodePos.z();
}
cloth.geometry.computeVertexNormals();
cloth.geometry.attributes.position.needsUpdate = true;
cloth.geometry.attributes.normal.needsUpdate = true;
```

&#160; &#160; &#160; &#160;**说明：**柔体更新的部分相比绳索，在顶点位置的基础上多了顶点法线的变化，法线变化可以让光照效果更写实，使变形效果更好。

# 支架的修改

## 支架长度修改

&#160; &#160; &#160; &#160;支架臂伸出的长度不能短于布料，这样才能挂住布料。

```js
// 支架
var armMass = 2;
var armLength = 3 + clothWidth;
var pylonHeight = clothPos.y + clothHeight;
```

## 修改锚点位置

&#160; &#160; &#160; &#160;之后修改锚点，将布料左右上角和支架臂固定：

```js
var influence = 0.5;
clothSoftBody.appendAnchor(0, arm.userData.physicsBody, true, influence);
clothSoftBody.appendAnchor( clothNumSegmentsZ, arm.userData.physicsBody, true, influence );
```

&#160; &#160; &#160; &#160;到此为止，整个场景完全完成。该程序的[完整源代码](https://github.com/THISISAGOODNAME/learn-ammojs/blob/master/Demo5-cloth.html)，[运行效果](http://aicdg.com/learn-ammojs/Demo5-cloth.html)
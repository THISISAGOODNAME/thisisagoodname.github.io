---
layout: post
title: "用threejs和ammojs制作地形"
description: "用threejs和ammojs制作地形"
category: html5
tags: [html5,bullet]
---

&#160; &#160; &#160; &#160;本节的主要内容就是使用THREE.PlaneBufferGeometry函数生成平摊的地面后修改每个点的Y坐标来产生绘制空间中的起伏；使用Ammo.btHeightfieldTerrainShape生成物理空间中的起伏。

<!-- more -->

* Table of Contents
{:toc}

1. bullet引擎和OpenGL结合创建简单的场景
2. three.js和ammo.js创建简单的场景
3. **创建地形**
4. 制作车辆
5. 柔体-绳索
6. 柔体-布料
7. 柔体-有体积的柔体
8. 使用blender引擎模拟物理场景

# 生成30个随机形状物体的场景

&#160; &#160; &#160; &#160;在制作地形之前，先在[上次](https://github.com/THISISAGOODNAME/learn-ammojs/blob/master/Demo1-Simple%20Scene.html)的基础上稍微修改一下场景，改成下降30个随机物体，物体的形状，质量均随机(质量和体积正相关)。

&#160; &#160; &#160; &#160;先完成一个生成随机形状的函数：

```js
// 生成随机形状的物体
    function createRondomObject(pos, quat, objectSize) {
        var numTypes = 4;
        var objectType = Math.ceil( Math.random() * numTypes );
        var threeObject = null;
        var shape = null;
//        var objectSize = 3;
        switch (objectType) {
            case 1:
                // Sphere
                var radius = 1 + Math.random() * objectSize;
                threeObject = new THREE.Mesh( new THREE.SphereGeometry( radius, 20, 20 ), createRendomColorObjectMeatrial() );
                shape = new Ammo.btSphereShape( radius );
                shape.setMargin( margin );
                break;
            case 2:
                // Box
                var sx = 1 + Math.random() * objectSize;
                var sy = 1 + Math.random() * objectSize;
                var sz = 1 + Math.random() * objectSize;
                threeObject = new THREE.Mesh( new THREE.BoxGeometry( sx, sy, sz, 1, 1, 1 ), createRendomColorObjectMeatrial() );
                shape = new Ammo.btBoxShape( new Ammo.btVector3( sx * 0.5, sy * 0.5, sz * 0.5 ) );
                shape.setMargin( margin );
                break;
            case 3:
                // Cylinder
                var radius = 1 + Math.random() * objectSize;
                var height = 1 + Math.random() * objectSize;
                threeObject = new THREE.Mesh( new THREE.CylinderGeometry( radius, radius, height, 20, 1 ), createRendomColorObjectMeatrial() );
                shape = new Ammo.btCylinderShape( new Ammo.btVector3( radius, height * 0.5, radius ) );
                shape.setMargin(margin);
                break;
            default:
                // Cone
                var radius = 1 + Math.random() * objectSize;
                var height = 2 + Math.random() * objectSize;
                threeObject = new THREE.Mesh( new THREE.CylinderGeometry( 0, radius, height, 20, 2 ), createRendomColorObjectMeatrial() );
                shape = new Ammo.btConeShape( radius, height );
                break;
        }
        var mass = objectSize * 5; // 体积越大质量越大
        createRigidBody(threeObject, shape, mass, pos, quat);
    }
```
&#160; &#160; &#160; &#160;然后修改createObjects函数

```js
    // 生成30个随机物体
    for (var i = 0; i < maxNumObjectss; i++) {
        pos.set(Math.random(), 2 *i, Math.random());
        quat.set(0, 0, 0, 1);
        createRondomObject(pos, quat, Math.ceil(Math.random() * 3));
    }
```

&#160; &#160; &#160; &#160;现在应该就可以看见随机的几何体从天而降了。

# 生成地形

&#160; &#160; &#160; &#160;生成地形的话，最主要的就是需要地形起伏的数据。对于一个平面，可以想象成Y坐标均相同的一个物体。只要修改每个点的Y坐标，就能产生起伏。

&#160; &#160; &#160; &#160;目标：接下来我要创建两个辅助函数generateHeight用来生成起伏用的Y坐标，createTerrainShape则是利用刚才生成的Y坐标在物理空间生成起伏。之后修改createObjects函数，在物理空间和绘制空间添加地形。

&#160; &#160; &#160; &#160;新增的全局变量如下：

```js
    // 高度场相关
    var terrainWidthExtents = 100;
    var terrainDepthExtents = 100;
    var terrainWidth = 128;
    var terrainDepth = 128;
    var terrainHalfWidth = terrainWidth / 2;
    var terrainHalfDepth = terrainDepth / 2;
    var terrainMaxHeight = 8;
    var terrainMinHeight = -2;
    var heightData = null;
    var ammoHeightData = null;
```

## 生成起伏

&#160; &#160; &#160; &#160;如果要生成连续的波纹状起伏，最简单的方法就是使用正弦函数。根据距离中心点的远近的sin值改变Y坐标，是非常简单生成正弦波形状的地形的方法：

```js
    // 生成连续起伏(利用正弦函数)
    function generateHeight( width, depth, minHeight, maxHeight ) {
        // 使用正弦函数生成凹凸不平的场(sinus wave)
        var size = width * depth;
        var data = new Float32Array(size);
        var hRange = maxHeight - minHeight;
        var w2 = width / 2;
        var d2 = depth / 2;
        var phaseMult = 12;
        var p = 0;
        for (var  i = 0; i < depth; i++) {
            for (var j = 0; j < width; j++) {
                var radius = Math.sqrt(
                        Math.pow((j - w2)/w2, 2.0) +
                        Math.pow((i - d2)/d2, 2.0)
                );
//                var height = (Math.sin(radius * phaseMult) + 1) * 0.5 * hRange + minHeight;
                var height = ( Math.sin( radius * phaseMult ) + 1 ) * 0.5 * hRange + minHeight;
                data[p] = height;
                p++;
            }
        }
        return data;
    }
```

&#160; &#160; &#160; &#160;记得在init函数中添加`heightData = generateHeight(terrainWidth, terrainDepth, terrainMinHeight, terrainMaxHeight);`来初始化高度数据

## 显示地形

&#160; &#160; &#160; &#160;有了hightData其实就可以在场景中显示地形了，在createObjects先删除之前用来生成地面的代码，然后添加如下：

```js
        // 渲染用
        var geometry = new THREE.PlaneBufferGeometry( 100, 100, terrainWidth - 1, terrainDepth - 1 );
        geometry.rotateX( -Math.PI / 2 );
        var vertices = geometry.attributes.position.array;
        for ( var i = 0, j = 0, l = vertices.length; i < l; i++, j += 3 ) {
            // j + 1 because it is the y component that we modify
            vertices[ j + 1 ] = heightData[ i ];
        }
        geometry.computeVertexNormals();
        var groundMaterial = new THREE.MeshPhongMaterial( { color: 0xC7C7C7 } );
        terrainMesh = new THREE.Mesh( geometry, groundMaterial );
        terrainMesh.receiveShadow = true;
        terrainMesh.castShadow = true;
        scene.add( terrainMesh );
        var textureLoader = new THREE.TextureLoader();
        textureLoader.load("textures/grid.png", function ( texture ) {
            texture.wrapS = THREE.RepeatWrapping;
            texture.wrapT = THREE.RepeatWrapping;
            texture.repeat.set( terrainWidth - 1, terrainDepth - 1 );
            groundMaterial.map = texture;
            groundMaterial.needsUpdate = true;
        });
```

## 利用高度数据，在物理空间生成地形

&#160; &#160; &#160; &#160;添加createTerrainShape函数，

```js
// 生成物理引擎用高度场
    function createTerrainShape(heightData) {
        // This parameter is not really used, since we are using PHY_FLOAT height data type and hence it is ignored
        var heightScale = 1;
        // Up axis = 0 for X, 1 for Y, 2 for Z. Normally 1 = Y is used.
        var upAxis = 1;
        // hdt, height data type. "PHY_FLOAT" is used. Possible values are "PHY_FLOAT", "PHY_UCHAR", "PHY_SHORT"
        var hdt = "PHY_FLOAT";
        // Set this to your needs (inverts the triangles)
        var flipQuadEdges = false;
        // Creates height data buffer in Ammo heap
        ammoHeightData = Ammo._malloc(4 * terrainWidth * terrainDepth);
        // Copy the javascript height data array to the Ammo one.
        var p = 0;
        var p2 = 0;
        for ( var j = 0; j < terrainDepth; j++ ) {
            for ( var i = 0; i < terrainWidth; i++ ) {
                // write 32-bit float data to memory
                Ammo.HEAPF32[ ammoHeightData + p2 >> 2 ] = heightData[ p ];
                p++;
                // 4 bytes/float
                p2 += 4;
            }
        }
        // Creates the heightfield physics shape
        var heightFieldShape = new Ammo.btHeightfieldTerrainShape(
                terrainWidth,
                terrainDepth,
                ammoHeightData,
                heightScale,
                terrainMinHeight,
                terrainMaxHeight,
                upAxis,
                hdt,
                flipQuadEdges
        );
        // Set horizontal scale
        var scaleX = terrainWidthExtents / ( terrainWidth - 1 );
        var scaleZ = terrainDepthExtents / ( terrainDepth - 1 );
        heightFieldShape.setLocalScaling( new Ammo.btVector3( scaleX, 1, scaleZ ) );
        heightFieldShape.setMargin( 0.05 );
        return heightFieldShape;
    }
```

&#160; &#160; &#160; &#160;主要的部分就是btHeightfieldTerrainShape函数，参数很多，主要讲解一下ammoHeightData参数的生成吧

&#160; &#160; &#160; &#160;地形一共有terrainWidth * terrainDepth个顶点，保存其Y坐标需要用sizeof(float32) * terrainWidth * terrainDepth字节即4 * terrainWidth * terrainDepth字节内存。Ammo._malloc(4 * terrainWidth * terrainDepth)分配内存后，用Ammo.HEAPF32[ ammoHeightData + p2 >> 2 ] = heightData[ p ]来填充内存。HEAPF32是asm.js中用来保存连续buffer的一种数据结构(HEAP指堆，F31指float32，意思是float32的堆)，一个HEAPF32是32位即四个字节Ammo.HEAPF32[ ammoHeightData + p2 >> 2 ]等价于Ammo.HEAPF32[ ammoHeightData + p2 / 4 ]，即从ammoHeightData开始的内存地址开始，偏移p2 / 4字节内存，即之后第p2 / 4个float的位置。将高度数据依次放入，生成高度缓冲。

## 在物理空间添加地形

&#160; &#160; &#160; &#160;在createObjects函数中添加：

```js
    // 物理计算用
    var groundShape = createTerrainShape(heightData);
    var groundTransform = new Ammo.btTransform();
    groundTransform.setIdentity();
    // 设置bullet计算时物体中心
    groundTransform.setOrigin(new Ammo.btVector3( 0, ( terrainMaxHeight + terrainMinHeight ) / 2, 0 ));
    var groundMass = 0;
    var groundLocalInertia = new Ammo.btVector3( 0, 0, 0 );
    var groundMotionState = new Ammo.btDefaultMotionState( groundTransform );
    var groundBody = new Ammo.btRigidBody(new Ammo.btRigidBodyConstructionInfo(groundMass, groundMotionState, groundShape, groundLocalInertia));
    physicsWorld.addRigidBody(groundBody);
```

&#160; &#160; &#160; &#160;之后运行即可。如果觉得摄像机离地面太过靠近，请修改相机的位置。该程序的[完整源代码](https://github.com/THISISAGOODNAME/learn-ammojs/blob/master/Demo2-terrain.html)，[运行效果](http://aicdg.com/learn-ammojs/Demo2-terrain.html),依旧鼠标点击可以发射小球。
---
layout: post
title: "用threejs和ammojs制作有体积的柔体"
description: "用threejs和ammojs制作有体积的柔体"
category: html5
tags: [html5,bullet]
---

&#160; &#160; &#160; &#160;三维空间中的物体，从简单到复杂，依次是点、线、面、体。柔体的线和面都已经完成，接下来，就要制作体。

<!-- more -->

1. bullet引擎和OpenGL结合创建简单的场景
2. three.js和ammo.js创建简单的场景
3. 创建地形
4. 制作车辆
5. 柔体-绳索
6. 柔体-布料
7. **柔体-有体积的柔体**
8. 使用blender引擎模拟物理场景

* Table of Contents
{:toc}

# 有体积的柔体场景

&#160; &#160; &#160; &#160;本节目标：在空间中生成两个带体积的柔体，一个球，一个长方体。

&#160; &#160; &#160; &#160;和布料相同，复制一份[绳索场景](https://github.com/THISISAGOODNAME/learn-ammojs/blob/master/Demo4-rope.html)作为工作的模板，不过这次要先把绳索和支架相关的部分先删干净。

# 需要的全局变量

&#160; &#160; &#160; &#160;添加三个全局变量

```js
var softBodySolver;
var softBodies = [];
var softBodyHelpers = new Ammo.btSoftBodyHelpers();
```

&#160; &#160; &#160; &#160;**说明：**softBodies数组用来保存柔体，和rigidBodies作用相同。把softBodyHelpers提取出来也是因为柔体不止一个。

# 创建柔体的函数createSoftVolume

&#160; &#160; &#160; &#160;该过程非常复杂，相比绳索和布料复杂的多得多。我们接下来创建一系列辅助函数。为了内存使用的高效，要把顶点的xyz坐标展开后依次放入一个数组，所以需要一个索引标记每个向量开始的位置。

## 1.  isEqual

```js
//fun1 判断误差范围内两向量相等的函数
function isEqual(x1, y1, z1, x2, y2, z2) {
    var delta = 0.000001;
    return  Math.abs(x1 - x2) < delta &&
        Math.abs(y1 - y2) < delta &&
        Math.abs(z1 - z2) < delta;
}
```

&#160; &#160; &#160; &#160;因为bullet引擎迭代次数有限，所以运算前后bufferGeom和indexedBufferGeom坐标可能会有误差，这个函数用来关联顶点坐标和纹理坐标

## 2. createIndexedBufferGeometryFromGeometry

```js
function createIndexedBufferGeometryFromGeometry(geometry) {
    var numVertices = geometry.vertices.length;
    var numFaces = geometry.faces.length;

    var bufferGeom = new THREE.BufferGeometry();
    var vertices = new Float32Array(numVertices * 3);
    var indices = new ( numFaces * 3 > 65535 ? Uint32Array : Uint16Array )(numFaces * 3);

    for (var i = 0; i < numVertices; i++) {
        var p = geometry.vertices[i];

        var i3 = i * 3;

        vertices[i3] = p.x;
        vertices[i3 + 1] = p.y;
        vertices[i3 + 2] = p.z;
    }

    for (var i = 0; i < numFaces; i++) {
        var f = geometry.faces[i];

        var i3 = i * 3;

        indices[i3] = f.a;
        indices[i3 + 1] = f.b;
        indices[i3 + 2] = f.c;
    }

    bufferGeom.setIndex(new THREE.BufferAttribute(indices, 1));
    bufferGeom.addAttribute('position', new THREE.BufferAttribute(vertices, 3));

    return bufferGeom;
}
```

&#160; &#160; &#160; &#160;addAttribute用来设置顶点着色器的position属性，顶点为了正确渲染到屏幕上，需要经过MVP变换，具体可以参考计算机图形学不赘述。postion属性就是模型点在模型局部空间内的相对位置。

## 3. mapIndices

```js
function mapIndices(bufGeometry, indexedBufferGeom) {
    // Creates ammoVertices, ammoIndices and ammoIndexAssociation in bufGeometry
    var vertices = bufGeometry.attributes.position.array;
    var idxVertices = indexedBufferGeom.attributes.position.array;
    var indices = indexedBufferGeom.index.array;

    var numIdxVertices = idxVertices.length / 3;
    var numVertices = vertices.length / 3;

    bufGeometry.ammoVertices = idxVertices;
    bufGeometry.ammoIndices = indices;
    bufGeometry.ammoIndexAssociation = [];

    for (var i = 0; i < numIdxVertices; i++) {
        var association = [];
        bufGeometry.ammoIndexAssociation.push(association);

        var i3 = i * 3;

        for (var j = 0; j < numVertices; j++) {
            var j3 = j * 3;
            if (isEqual(idxVertices[i3], idxVertices[i3 + 1], idxVertices[i3 + 2],
                    vertices[j3], vertices[j3 + 1], vertices[j3 + 2])) {
                association.push(j3);
            }
        }
    }
}
```

&#160; &#160; &#160; &#160;ammoIndexAssociation[i]表示数组中第i个向量开始的位置。

## 4. processGeometry

```js
function processGeometry(bufGeometry) {
    // Obtain a Geometry
//        var geometry = new THREE.Geometry().fromBufferGeometry(bufGeometry);
    var geometry = new THREE.Geometry().fromBufferGeometry( bufGeometry );

    // Merge the vertices so the triangle soup is converted to indexed triangles
//        var vertsDiff = geometry.mergeVertices();
    geometry.mergeVertices();

    // Convert again to BufferGeometry, indexed
    var indexedBufferGeom = createIndexedBufferGeometryFromGeometry(geometry);

    // Create index arrays mapping the indexed vertices to bufGeometry vertices
    mapIndices(bufGeometry, indexedBufferGeom);
}
```

&#160; &#160; &#160; &#160;mergeVertices函数是THREE.Geometry()的一个功能，可以优化物体的顶点和索引，使之渲染性能更好。

## 5. createSoftVolume

```js
function createSoftVolume(bufferGeom, mass, pressure) {
    processGeometry(bufferGeom);

    // 渲染空间中的柔体
    var volume = new THREE.Mesh(bufferGeom, new THREE.MeshPhongMaterial({color: 0x00EE00}));
    volume.castShadow = true;
    volume.receiveShadow = true;
    volume.frustumCulled = false;
    scene.add(volume);

    // 物理空间中的柔体
    var volumeSoftBody = softBodyHelpers.CreateFromTriMesh(
        physicsWorld.getWorldInfo(),
        bufferGeom.ammoVertices,
        bufferGeom.ammoIndices,
        bufferGeom.ammoIndices.length / 3,
        true);

    var sbConfig = volumeSoftBody.get_m_cfg();
    sbConfig.set_viterations(40);
    sbConfig.set_piterations(40);

    // Soft-soft and soft-rigid collisions
    sbConfig.set_collisions(0x11);

    // Friction 摩擦因数
    sbConfig.set_kDF(0.1);
    // Damping 阻尼
    sbConfig.set_kDP(0.01);
    // Pressure 压力
    sbConfig.set_kPR(pressure);
    // Stiffness 强度
    volumeSoftBody.get_m_materials().at(0).set_m_kLST(0.9);
    volumeSoftBody.get_m_materials().at(0).set_m_kAST(0.9);

    // Soft-soft and soft-rigid collisions
    sbConfig.set_collisions(0x11);

    volumeSoftBody.setTotalMass(mass, false);
    Ammo.castObject(volumeSoftBody, Ammo.btCollisionObject).getCollisionShape().setMargin(margin);
    physicsWorld.addSoftBody(volumeSoftBody, 1, -1);
    volume.userData.physicsBody = volumeSoftBody;

    // Disable deactivation
    // Ammo.DISABLE_DEACTIVATION = 4
    volumeSoftBody.setActivationState(4);

    softBodies.push(volume);
}
```

&#160; &#160; &#160; &#160;createSoftVolume和创建车辆系统很类似，要配置大量物理状态。

## 修改createObjects函数

&#160; &#160; &#160; &#160;在场景中保留砖墙，去掉绳索和小球，添加一个坡道和两个柔体，两个柔体一个球一个长方体。

```js
// 创建坡道
pos.set(3, 1, 0);
quat.setFromAxisAngle(new THREE.Vector3(0, 0, 1), 30 * Math.PI / 180);
var obstacle = createParallellepiped(10, 1, 4, 0, pos, quat, new THREE.MeshPhongMaterial({color: 0x606060}));
obstacle.castShadow = true;
obstacle.receiveShadow = true;

// 创建拥有体积的柔体
var volumeMass = 15;

var sphereGeometry = new THREE.SphereBufferGeometry(1.5, 40, 25);
sphereGeometry.translate(5, 5, 0);
createSoftVolume(sphereGeometry, volumeMass, 250);
var boxGeometry = new THREE.BufferGeometry().fromGeometry(new THREE.BoxGeometry(1, 1, 5, 4, 4, 20));
boxGeometry.translate(-2, 5, 0);
createSoftVolume(boxGeometry, volumeMass, 120);
```

# 修改updatePhysics函数

&#160; &#160; &#160; &#160;在updatePhysics函数中添加：

```js
// 更新柔体状态
for (var i = 0, iL = softBodies.length; i < iL; i++) {
    var volume = softBodies[i];
    var geometry = volume.geometry;
    var softBody = volume.userData.physicsBody;
    var volumePositions = geometry.attributes.position.array;
    var volumeNormals = geometry.attributes.normal.array;
    var association = geometry.ammoIndexAssociation;
    var numVerts = association.length;
    var nodes = softBody.get_m_nodes();

    for (var j = 0; j < numVerts; j++) {
        var node = nodes.at(j);
        var nodePos = node.get_m_x();
        var x = nodePos.x();
        var y = nodePos.y();
        var z = nodePos.z();
        var nodeNormal = node.get_m_n();
        var nx = nodeNormal.x();
        var ny = nodeNormal.y();
        var nz = nodeNormal.z();
        var assocVertex = association[j];
        for (var k = 0, kl = assocVertex.length; k < kl; k++) {
            var indexVertex = assocVertex[k];
            volumePositions[indexVertex] = x;
            volumeNormals[indexVertex] = nx;
            indexVertex++;
            volumePositions[indexVertex] = y;
            volumeNormals[indexVertex] = ny;
            indexVertex++;
            volumePositions[indexVertex] = z;
            volumeNormals[indexVertex] = nz;
        }
    }

    geometry.attributes.position.needsUpdate = true;
    geometry.attributes.normal.needsUpdate = true;
}
```

&#160; &#160; &#160; &#160;到此为止，整个场景完全完成。该程序的[完整源代码](https://github.com/THISISAGOODNAME/learn-ammojs/blob/master/Demo6-softboy-volume.html)，[运行效果](http://aicdg.com/learn-ammojs/Demo6-softboy-volume.html)。别忘了按鼠标可以发射小球。建议把发射小球的质量减小，如果柔体的前表面穿到了后表面之后，会发生各种奇怪的错误。柔体经常发生一些破面之类的问题，这是因为求解迭代次数过少导致。
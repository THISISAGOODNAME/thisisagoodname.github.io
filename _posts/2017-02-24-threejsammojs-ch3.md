---
layout: post
title: "用threejs和ammojs制作绳索"
description: "用threejs和ammojs制作绳索"
category: html5
tags: [html5,bullet]
---

&#160; &#160; &#160; &#160;bullet2物理引擎的核心模块一共有4个，分别是LinearMath、BulletCollision、BulletDynamics、BulletSoftBody(后来还加入了BulletInverseDynamics)。经历了车辆系统之后，刚体物理应该没什么能继续阻挡我们了，所以接下来，开始柔体物理的学习。

<!-- more -->

1. bullet引擎和OpenGL结合创建简单的场景
2. three.js和ammo.js创建简单的场景
3. 创建地形
4. 制作车辆
5. **柔体-绳索**
6. 柔体-布料
7. 柔体-有体积的柔体
8. 使用blender引擎模拟物理场景

* Table of Contents
{:toc}

# 绳索链接支架和小球的场景

&#160; &#160; &#160; &#160;本节的主要目的就是创建一个支架和一个小球，支架和小球之间由绳索链接；支架本身带有铰链关节可以用ZX键控制转动；场景中还有一个砖墙。

&#160; &#160; &#160; &#160;依旧先复制一份[生成地形场景的代码](https://github.com/THISISAGOODNAME/learn-ammojs/blob/master/Demo2-terrain.html)，删去heightData相关的所以内容，然后重新添加40x40的地面到场景中。

# 准备工作

## 添加全局变量

&#160; &#160; &#160; &#160;添加绳索和支架旋转相关的全局变量，需要的全局变量并不多：

```js
// 绳索相关
var hinge;
var rope;
var softBodySolver;

// 支架旋转
var armMovement = 0;
```

## 要点：修改initPhysics，使之支持柔体动力学

&#160; &#160; &#160; &#160;全新的initPhysics函数如下：

```js
function initPhysics() {
  /**
   * 启动支持柔体的物理引擎
   * config，solver和physicsWorld要用softbody版本
   * 还要单独创建softBodySolver
   */
  collisionConfiguration = new Ammo.btSoftBodyRigidBodyCollisionConfiguration();
  dispatcher = new Ammo.btCollisionDispatcher(collisionConfiguration);
  broadphase = new Ammo.btDbvtBroadphase();
  solver = new Ammo.btSequentialImpulseConstraintSolver();
  softBodySolver = new Ammo.btDefaultSoftBodySolver();
  physicsWorld = new Ammo.btSoftRigidDynamicsWorld(dispatcher, broadphase, solver, collisionConfiguration, softBodySolver);
  physicsWorld.setGravity(new Ammo.btVector3(0, gravityConstant, 0));

  physicsWorld.getWorldInfo().set_m_gravity(new Ammo.btVector3(0, gravityConstant, 0));
}
```

&#160; &#160; &#160; &#160;**注意：**set_m_gravity是asm.js的扩展方法，可以给c++代码中的值属性添加get/set方法进行调用。

## 在场景中添加砖墙、小球和支架

&#160; &#160; &#160; &#160;在createObjects添加如下代码：

```js
// 创建一个球
var ballMass =1.2;
var ballRadius = 0.6;

var ball = new THREE.Mesh(new THREE.SphereGeometry(ballRadius, 20, 20), new THREE.MeshPhongMaterial( { color: 0x202020 } ));
ball.castShadow = true;
ball.receiveShadow = true;
var ballShape = new Ammo.btSphereShape( ballRadius );
ballShape.setMargin( margin );
pos.set( -3, 2, 0 );
quat.set( 0, 0, 0, 1 );
createRigidBody( ball, ballShape, ballMass, pos, quat );
ball.userData.physicsBody.setFriction( 0.5 );

// 创建砖墙
var brickMass = 0.5;
var brickLength = 1.2;
var brickDepth = 0.6;
var brickHeight = brickLength * 0.5;
var numBricksLength = 6;
var numBricksHeight = 8;
var z0 = - numBricksLength * brickLength * 0.5;
pos.set( 0, brickHeight * 0.5, z0 );
quat.set( 0, 0, 0, 1 );
for ( var j = 0; j < numBricksHeight; j ++ ) {
  var oddRow = ( j % 2 ) == 1;
  pos.z = z0;
  if ( oddRow ) {
    pos.z -= 0.25 * brickLength;
  }
  var nRow = oddRow? numBricksLength + 1 : numBricksLength;
  for ( var i = 0; i < nRow; i ++ ) {
    var brickLengthCurrent = brickLength;
    var brickMassCurrent = brickMass;
    if ( oddRow && ( i == 0 || i == nRow - 1 ) ) {
      brickLengthCurrent *= 0.5;
      brickMassCurrent *= 0.5;
    }
    var brick = createParallellepiped( brickDepth, brickHeight, brickLengthCurrent, brickMassCurrent, pos, quat, createRendomColorObjectMeatrial() );
    brick.castShadow = true;
    brick.receiveShadow = true;
    if ( oddRow && ( i == 0 || i == nRow - 2 ) ) {
      pos.z += 0.75 * brickLength;
    }
    else {
      pos.z += brickLength;
    }
  }
  pos.y += brickHeight;
}

// 支架
var armMass = 2;
var armLength = 3;
var pylonHeight = ropePos.y + ropeLength;
var baseMaterial = new THREE.MeshPhongMaterial( { color: 0x606060 } );
pos.set( ropePos.x, 0.1, ropePos.z - armLength );
quat.set( 0, 0, 0, 1 );
var base = createParallellepiped( 1, 0.2, 1, 0, pos, quat, baseMaterial );
base.castShadow = true;
base.receiveShadow = true;
pos.set( ropePos.x, 0.5 * pylonHeight, ropePos.z - armLength );
var pylon = createParallellepiped( 0.4, pylonHeight, 0.4, 0, pos, quat, baseMaterial );
pylon.castShadow = true;
pylon.receiveShadow = true;
pos.set( ropePos.x, pylonHeight + 0.2, ropePos.z - 0.5 * armLength );
var arm = createParallellepiped( 0.4, 0.4, armLength + 0.4, armMass, pos, quat, baseMaterial );
arm.castShadow = true;
arm.receiveShadow = true;
```

# 在绘图空间和物理空间初始化小球

&#160; &#160; &#160; &#160;所谓绳索，其实就是由一系列点产生的折线，细分值越高，绳子的仿真程度就越好，但是计算代价也越大，我这里以10作为绳子的段数：

```js
// 创建绳索

// 绳索的绘制部分
var ropeNumSegments = 10;
var ropeLength = 4;
var ropeMass = 3;
var ropePos = ball.position.clone();
ropePos.y += ballRadius;

var segmentLength = ropeLength / ropeNumSegments;
var ropeGeometry = new THREE.BufferGeometry();
var ropeMaterial = new THREE.LineBasicMaterial( { color: 0x000000 } );
var ropePositions = [];
var ropeIndices = [];

for (var i = 0; i < ropeNumSegments + 1; i++) {
  ropePositions.push(ropePos.x, ropePos.y + i * segmentLength, ropePos.z);
}

for (var i = 0; i < ropeNumSegments; i++) {
  ropeIndices.push(i, i+1);
}

ropeGeometry.setIndex(new THREE.BufferAttribute(new Uint16Array(ropeIndices), 1));
ropeGeometry.addAttribute('position', new THREE.BufferAttribute(new Float32Array(ropePositions), 3));
ropeGeometry.computeBoundingSphere();
rope = new THREE.LineSegments( ropeGeometry, ropeMaterial );
rope.castShadow = true;
rope.receiveShadow = true;
scene.add(rope);

// 物理空间中的绳索
var softBodyHelpers = new Ammo.btSoftBodyHelpers();
var ropeStart = new Ammo.btVector3(ropePos.x, ropePos.y, ropePos.z);
var ropeEnd = new Ammo.btVector3(ropePos.x, ropePos.y + ropeLength, ropePos.z);
var ropeSoftBody = softBodyHelpers.CreateRope(physicsWorld.getWorldInfo(), ropeStart, ropeEnd, ropeNumSegments -1, 0);
var sbConfig = ropeSoftBody.get_m_cfg();
sbConfig.set_viterations(10);
sbConfig.set_piterations(10);
ropeSoftBody.setTotalMass(ropeMass, false);
Ammo.castObject(ropeSoftBody, Ammo.btCollisionObject).getCollisionShape().setMargin(margin * 3);
physicsWorld.addSoftBody(ropeSoftBody, 1, -1);
rope.userData.physicsBody = ropeSoftBody;

// Disable deactivation
var DISABLE_DEACTIVATION = 4;
ropeSoftBody.setActivationState(DISABLE_DEACTIVATION);
```

&#160; &#160; &#160; &#160;**说明：**CreateRope函数设置绳子首尾两端点坐标，再传入中间辅助分段点的个数，即分段数-1。最后别忘记取消绳子的运动衰减。

# 同步绳索在绘图空间中的形状

&#160; &#160; &#160; &#160;在updatePhysics中，添加如下代码：

```js
// 更新绳索状态
var softBody = rope.userData.physicsBody;
var ropePositions = rope.geometry.attributes.position.array;
var numVerts = ropePositions.length / 3;
var nodes = softBody.get_m_nodes();
var indexFloat = 0;
for (var i = 0; i <numVerts; i++) {
  var node = nodes.at(i);
  var nodePos = node.get_m_x();
  ropePositions[ indexFloat++ ] = nodePos.x();
  ropePositions[ indexFloat++ ] = nodePos.y();
  ropePositions[ indexFloat++ ] = nodePos.z();
}
rope.geometry.attributes.position.needsUpdate = true;
```

&#160; &#160; &#160; &#160;到此为止，已经获得了一个从天坠落的绳索。

# 用绳索链接支架和小球

&#160; &#160; &#160; &#160;用appendAnchor方法，可以添加柔体和刚体之间的锚点，即将两个物体链接起来。在createObjects函数中添加：

```js
// 将球和支架用绳索链接
var influence = 1;
ropeSoftBody.appendAnchor(0, ball.userData.physicsBody, true, influence);
ropeSoftBody.appendAnchor( ropeNumSegments, arm.userData.physicsBody, true, influence );
```

&#160; &#160; &#160; &#160;现在运行场景，就可以看到支架臂和小球被连在一起坠落。

# 为支架添加铰链约束

&#160; &#160; &#160; &#160;在createObjects函数中，添加如下代码：

```js
// 给支架臂添加铰链约束
var pivotA = new Ammo.btVector3(0, pylonHeight * 0.5, 0);
var pivotB = new Ammo.btVector3(0, -0.2, -armLength * 0.5);
var axis = new Ammo.btVector3(0, 1, 0);
hinge = new Ammo.btHingeConstraint(pylon.userData.physicsBody, arm.userData.physicsBody, pivotA, pivotB, axis, axis, true);
physicsWorld.addConstraint(hinge, true);
```

&#160; &#160; &#160; &#160;**注意：**铰链约束的在两个物体上的约束点没有要求，不一定要是同一个点，两个物体也并不要求接触。

## 添加ZX键控制铰链转动

&#160; &#160; &#160; &#160;在initInput中添加ZX的事件绑定：

```js
window.addEventListener( 'keydown', function( event ) {
  switch ( event.code ) {
    // Z
    case 'KeyZ':
      armMovement = 1;
      break;
    // X
    case 'KeyX':
      armMovement = - 1;
      break;
  }
}, false );
window.addEventListener( 'keyup', function( event ) {
  armMovement = 0;
}, false );
```

&#160; &#160; &#160; &#160;然后再在updatePhysics函数中，添加

```js
// ZX控制铰链转动
hinge.enableAngularMotor(true, 1.5 * armMovement, 50);
```

&#160; &#160; &#160; &#160;到此为止，整个场景完全完成。该程序的[完整源代码](https://github.com/THISISAGOODNAME/learn-ammojs/blob/master/Demo4-rope.html)，[运行效果](http://aicdg.com/learn-ammojs/Demo4-rope.html)
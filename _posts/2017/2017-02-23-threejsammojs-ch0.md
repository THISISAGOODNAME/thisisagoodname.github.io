---
layout: post
title: "用threejs和ammojs制作一个简单场景"
description: "用threejs和ammojs制作一个简单场景"
category: html5
tags: [html5,bullet]
---

&#160; &#160; &#160; &#160;本文的主要内容就是用three.js制作简单场景，并引入ammo.js物理引擎。

<!-- more -->

* Table of Contents
{:toc}

1. bullet引擎和OpenGL结合创建简单的场景
2. **three.js和ammo.js创建简单的场景**
3. 创建地形
4. 制作车辆
5. 柔体-绳索
6. 柔体-布料
7. 柔体-有体积的柔体
8. 使用blender引擎模拟物理场景

&#160; &#160; &#160; &#160;本节目标是制作一个模板代码，后续开发都在这个模板的基础上继续开发。这一系列教程都是bullet物理引擎简易使用教程，不会涉及很多原理(比如函数的每个参数的具体含义)，只有简单的使用方法。

&#160; &#160; &#160; &#160;原因：我以前学编程，总是先知其然再探其所以然。能粗略使用，对于初学者的信心提升是非常高的。我自己也是初学者，我以前学东西的时候，总是在这种基础关卡住，而向别人求教的时候，总是直接给你丢个文档过来说读去吧。一开始就读文档对于初学者来说并不是什么好的入门方法，相反，通过一些简单的实例的模仿迅速拾起信心，才是关键。

# three.js制作简单场景

&#160; &#160; &#160; &#160;整体思路：主函数一共两个，分别是init和animate，作用类似于unity3d中的Start()和Update()。

&#160; &#160; &#160; &#160;全局变量一共有如下

```js
	// 绘图相关变量
    var container, stats;
    var camera, controls, scene, renderer;
    var textureLoader;
    var clock = new THREE.Clock();
```

&#160; &#160; &#160; &#160;init中只有一个子函数initGraphics，其创建一个空场景，并添加一个环境光和一个线性光，线性光可以产生阴影。initGraphics也是为了和后文中的initPhysics区分开。

&#160; &#160; &#160; &#160;animate函数的主要作用是更新场景，绘制下一帧，更新相机位置，顺便用stats工具统计一下帧数、每帧需要的时间、内存占用什么的。

&#160; &#160; &#160; &#160;本场景除了three.js之外，还引用了OrbitControls.js，stats.min.js，Detector.js三个文件，分别用来添加鼠标控制，状态统计和检测浏览器对webgl的支持。

&#160; &#160; &#160; &#160;[本场景源码](https://github.com/THISISAGOODNAME/learn-ammojs/blob/master/boilerplate/basicScene.html)，[运行效果](http://aicdg.com/learn-ammojs/boilerplate/basicScene.html),如果你的程序也是一片浅蓝，帧数显示正确，恭喜你运行成功。

# 添加物理场景

&#160; &#160; &#160; &#160;整体思路：在上一个场景的基础上添加初始化物理场景函数initPhysics、物理场景更新函数updatePhysics、将物理场景中的物体和绘图空间中的场景关联起来并添加到两个场景中的函数createRigidBody(前几个demo中物理场景只支持刚体)，向场景中添加平行六面体的函数createParallellepiped，以及初始化绘图空间和物理空间中物体的函数createObjects。

&#160; &#160; &#160; &#160;需要添加的新全局变量如下。

```js
// 物理引擎相关变量
    var gravityConstant = -9.8;
    var collisionConfiguration;
    var dispatcher;
    var broadphase;
    var solver;
    var physicsWorld;
    var rigidBodies = [];
    var margin = 0.05;
    var transformAux1 = new Ammo.btTransform();
```

&#160; &#160; &#160; &#160;**说明：**rigidBodies同时保存刚体在绘图空间和物理空间中的形状。其实就是把物体的物理形状保存在THREEObject的userData属性中。

&#160; &#160; &#160; &#160;**注意：**别忘了把initPhysics还有createObjects加到init，吧updatePhysics加到render函数中。

&#160; &#160; &#160; &#160;列举几个关键函数的源代码：

## 初始化物理空间函数initPhysics

```js
    function initPhysics() {
        // bullet基本场景配置
        collisionConfiguration = new Ammo.btDefaultCollisionConfiguration();
        dispatcher = new Ammo.btCollisionDispatcher(collisionConfiguration);
        broadphase = new Ammo.btDbvtBroadphase();
        solver = new Ammo.btSequentialImpulseConstraintSolver();
        physicsWorld = new Ammo.btDiscreteDynamicsWorld(dispatcher, broadphase, solver, collisionConfiguration);
        physicsWorld.setGravity(new Ammo.btVector3(0, gravityConstant, 0));
    }
```

## 物理场景更新函数

```js
    function updatePhysics(deltaTime) {
        physicsWorld.stepSimulation(deltaTime);

        // 更新物体位置
        for (var i = 0, iL = rigidBodies.length; i <iL; i++ ){
            var objThree = rigidBodies[i];
            var objPhys = objThree.userData.physicsBody;
            var ms = objPhys.getMotionState();
            if (ms) {
                ms.getWorldTransform(transformAux1);
                var p = transformAux1.getOrigin();
                var q = transformAux1.getRotation();
                objThree.position.set(p.x(), p.y(), p.z());
                objThree.quaternion.set(q.x(), q.y(), q.z(), q.w());
            }
        }
    }
```

## 刚体生成函数

```js
    function createRigidBody(threeObject, physicsShape, mass, pos, quat) {
        threeObject.position.copy(pos);
        threeObject.quaternion.copy(quat);

        var transform = new Ammo.btTransform();
        transform.setIdentity();
        transform.setOrigin(new Ammo.btVector3(pos.x, pos.y, pos.z));
        transform.setRotation(new Ammo.btQuaternion(quat.x, quat.y, quat.z, quat.w));
        var motionState = new Ammo.btDefaultMotionState(transform);

        var localInertia = new Ammo.btVector3(0, 0, 0);
        physicsShape.calculateLocalInertia(mass, localInertia);

        var rbInfo = new Ammo.btRigidBodyConstructionInfo(mass, motionState, physicsShape, localInertia);
        var body = new Ammo.btRigidBody(rbInfo);

        threeObject.userData.physicsBody = body;

        scene.add(threeObject);

        if (mass > 0) {
            rigidBodies.push(threeObject);

            // Disable deactivation
            // 防止物体弹力过快消失

            // Ammo.DISABLE_DEACTIVATION = 4
            body.setActivationState(4);
        }

        physicsWorld.addRigidBody(body);

    }
```

&#160; &#160; &#160; &#160;该程序的[完整源代码](https://github.com/THISISAGOODNAME/learn-ammojs/blob/master/boilerplate/basicBulletScene.html)，[运行效果](http://aicdg.com/learn-ammojs/boilerplate/basicBulletScene.html),我在createObject函数中创建了一个40x40m^2的地面，每个格子边长为1m

# 为场景添加鼠标点击发射功能

&#160; &#160; &#160; &#160;目标：为场景添加点击鼠标发射小球的功能。

```js
    function initInput() {

        window.addEventListener( 'mousedown', function( event ) {

            mouseCoords.set(
                    ( event.clientX / window.innerWidth ) * 2 - 1,
                    - ( event.clientY / window.innerHeight ) * 2 + 1
            );

            raycaster.setFromCamera( mouseCoords, camera );

            // Creates a ball and throws it
            var ballMass = 35;
            var ballRadius = 0.4;

            var ball = new THREE.Mesh( new THREE.SphereGeometry( ballRadius, 14, 10 ), ballMaterial );
            ball.castShadow = true;
            ball.receiveShadow = true;
            var ballShape = new Ammo.btSphereShape( ballRadius );
            ballShape.setMargin( margin );
            var pos = new THREE.Vector3();
            var quat = new THREE.Quaternion();
            pos.copy( raycaster.ray.direction );
            pos.add( raycaster.ray.origin );
            quat.set( 0, 0, 0, 1 );
            var ballBody = createRigidBody( ball, ballShape, ballMass, pos, quat );

            pos.copy( raycaster.ray.direction );
            pos.multiplyScalar( 24 );
            ballBody.setLinearVelocity( new Ammo.btVector3( pos.x, pos.y, pos.z ) );

        }, false );

    }
   
```

&#160; &#160; &#160; &#160; initInput添加到init函数中即可。

&#160; &#160; &#160; &#160;html的坐标系统是左上角为原点，向右和向下为xy轴正方向，单位为设备坐标(px默认)；而webgl坐标系统则是正中心为坐标原点，向右向上为xy正方向，坐标范围[-1,1]的规范化坐标系。

&#160; &#160; &#160; &#160;这个函数的作用是在鼠标点击位置沿着摄像机视线方向发射初速度24m/s，质量35kg，半径0.4m的小球(其实是直径0.8m的超级炮弹)

&#160; &#160; &#160; &#160;该程序的[完整源代码](https://github.com/THISISAGOODNAME/learn-ammojs/blob/master/boilerplate/BasicBulletInputScene.html)，[运行效果](http://aicdg.com/learn-ammojs/boilerplate/BasicBulletInputScene.html),鼠标点击可以发射小球。

# 第一个实例

&#160; &#160; &#160; &#160;在完成上面的例子后，就有了一个基础模板，以后的开发完全都基于此模板。

&#160; &#160; &#160; &#160;开发目标：添加一个30个方块从天而降的场景。

&#160; &#160; &#160; &#160;实现方法，添加一个辅助函数createRendomColorObjectMeatrial。

```js
// 生成随机颜色材质
    function createRendomColorObjectMeatrial() {
        var color = Math.floor(Math.random() * (1 << 24));
        return new THREE.MeshPhongMaterial({color: color});
    }
```

&#160; &#160; &#160; &#160;Math.floor(Math.random() * (1 << 24))可以生成一个随机的24位整数(即6位16进制整数)

&#160; &#160; &#160; &#160;然后在createObjects函数中添加

```js
        // 随机创建30个箱子
        for (var i = 0; i < 30; i++) {
            pos.set(Math.random(), 2 *i, Math.random());
            quat.set(0, 0, 0, 1);

            createParallellepiped(1, 1, 1, 1, pos, quat, createRendomColorObjectMeatrial());
        }
```

&#160; &#160; &#160; &#160;该程序的[完整源代码](https://github.com/THISISAGOODNAME/learn-ammojs/blob/master/Demo1-Simple%20Scene.html)，[运行效果](http://aicdg.com/learn-ammojs/Demo1-Simple%20Scene.html),依旧鼠标点击可以发射小球。


# 知识补充：六自由度刚体物理

&#160; &#160; &#160; &#160;bullet中的刚体物理碰撞是六自由度物理碰撞。六自由度的意思是可以同时产生位移(xyz轴各一个)和旋转(xyz轴各一个)。

&#160; &#160; &#160; &#160;bullet中表示刚体运动的类为btTransform，在updatePhysics函数中，从物体的montionState()中获取变化之后，可以用getOrigin和getRotation两个方法获取位置(btVector3类型)和旋转(btQuaternion类型)。

&#160; &#160; &#160; &#160;Quaternion的意思是四元数，一个1x4的向量，只靠四个数，就能很好的描述三维空间中物体绕任意轴旋转，而且还可以有效预防物体在空间中旋转的欧拉万向锁问题。四元数描述旋转网上有不少资料，我也不在赘述，大家只要知道一个1x4向量就能描述物体在三维空间中旋转即可。

&#160; &#160; &#160; &#160;刚体物体的自由度为6，柔体更多，因为柔体自身可以发生形变，相关柔体的部分日后再说。

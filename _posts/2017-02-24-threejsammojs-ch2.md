---
layout: post
title: "用threejs和ammojs制作车辆"
description: "用threejs和ammojs制作车辆"
category: html5
tags: [html5,bullet]
---

&#160; &#160; &#160; &#160;接下来就是刚体物理的中最有意思的部分，车辆物理，也是刚体运动学中的难点。[运行效果](http://aicdg.com/learn-ammojs/Demo3-vehicle.html)

<!-- more -->

1. bullet引擎和OpenGL结合创建简单的场景
2. three.js和ammo.js创建简单的场景
3. 创建地形
4. **制作车辆**
5. 柔体-绳索
6. 柔体-布料
7. 柔体-有体积的柔体
8. 使用blender引擎模拟物理场景


&#160; &#160; &#160; &#160;和之前的demo不太一样，一开始就展示了运行场景。一个用WSAD控制小车运动的场景，应该还是比较有趣的。

&#160; &#160; &#160; &#160;这个场景和之前的场景比较有很多不同，之前的demo，整个场景也就200来行，而本场景长度超过600行，createVehicle一个函数的长度就有差不多150行。对于初学者来说应该是个不小的挑战。先打一下气。慢慢来，别着急，程序运行不起来的时候一定要心平气和，铭记：气浮如流水不安,心静似高山不动。一旦跨过这道坎，相信你的信心一定能大幅度提高。

* Table of Contents
{:toc}

# 概念说明

&#160; &#160; &#160; &#160;bullet引擎中的物理车辆的核心，就是btRaycastVehicle类。

&#160; &#160; &#160; &#160;raycast是光线投射。光线投射顾名思义，就是在某一点沿着某个方向发射一条射线，得到这条射线与物体穿入穿出的结果的算法。光线投射算法应用非常多，不仅仅是在实感渲染方面，可以通过光线穿过物体表面的次数是奇数还是偶数判断空间中的点在物体内部还是外部，进而判断多个物体之间的遮挡和重合关系。

&#160; &#160; &#160; &#160;RaycastVehicle的意思，就是车轮能获得沿着车轮轴线的力，进而控制车体运行。这是一个高度简化的模型，现实世界中的车辆前进的过程比这个复杂的多，汽车仿真软件，也没一个用bullet/phsyX/newton仿真的，少数用ODE的也是魔改的ODE，追加了大量物理模型。

# 整体设计

&#160; &#160; &#160; &#160;模板准备：将[生成地形](https://github.com/THISISAGOODNAME/learn-ammojs/blob/master/Demo2-terrain.html)的代码复制一份，删除generateHeight和createTerrainShape两个函数声明，然后修改createObjects将地形创建的部分也删除。吧heightData相关的变量删除。

&#160; &#160; &#160; &#160;核心思路：添加syncList变量，该变量可以存储一系列函数，函数的作用是更新某个特定的物体在物理空间和绘图空间中的位置。

# 事前练手

&#160; &#160; &#160; &#160;首先先完成一个修改版的createBox函数，作用和之前生成平行六面体的函数作用相同，区别是这一次并不向rigidBodies数组中添加物体，而是将更新本物体在两个空间中位置的函数放入syncList数组。

&#160; &#160; &#160; &#160;首先创建几个辅助用的全局变量

```js
// bullet内置宏
    var DISABLE_DEACTIVATION = 4;
    var TRANSFORM_AUX = new Ammo.btTransform();
    var ZERO_QUATERNION = new THREE.Quaternion(0, 0, 0, 1);

// 车辆系统辅助
    var syncList = []; // 车辆系统用syncList保存事件列表,不再使用rigidBodies变量.存放用于绘制和同步物理场景的方法
```

## 为syncList修改过的createBox函数

&#160; &#160; &#160; &#160;完整的createBox函数如下：

```js
    function createBox(pos, quat, w, l, h, mass, friction) {
        var material = createRendomColorObjectMeatrial();
        var shape = new THREE.BoxGeometry(w, l, h, 1, 1, 1);
        var geometry = new Ammo.btBoxShape(new Ammo.btVector3(w * 0.5, l * 0.5, h * 0.5));
        if(!mass) mass = 0;
        if(!friction) friction = 1;
        var mesh = new THREE.Mesh(shape, material);
        mesh.position.copy(pos);
        mesh.quaternion.copy(quat);
        scene.add( mesh );
        var transform = new Ammo.btTransform();
        transform.setIdentity();
        transform.setOrigin(new Ammo.btVector3(pos.x, pos.y, pos.z));
        transform.setRotation(new Ammo.btQuaternion(quat.x, quat.y, quat.z, quat.w));
        var motionState = new Ammo.btDefaultMotionState(transform);
        var localInertia = new Ammo.btVector3(0, 0, 0);
        geometry.calculateLocalInertia(mass, localInertia);
        var rbInfo = new Ammo.btRigidBodyConstructionInfo(mass, motionState, geometry, localInertia);
        var body = new Ammo.btRigidBody(rbInfo);
        body.setFriction(friction);

        physicsWorld.addRigidBody(body);
        if (mass > 0) {
            body.setActivationState(DISABLE_DEACTIVATION);
            // 同步物理场景和绘图空间
            function sync(dt) {
                var ms = body.getMotionState();
                if (ms) {
                    ms.getWorldTransform(TRANSFORM_AUX);
                    var p = TRANSFORM_AUX.getOrigin();
                    var q = TRANSFORM_AUX.getRotation();
                    mesh.position.set(p.x(), p.y(), p.z());
                    mesh.quaternion.set(q.x(), q.y(), q.z(), q.w());
                }
            }
            syncList.push(sync);
        }
    }
```

&#160; &#160; &#160; &#160;多了一个添加摩擦力的功能，然后返回的是一个sync函数，作用是更新绘图空间中的坐标，使之和物理空间相同。

## 测试createBox函数

&#160; &#160; &#160; &#160;在createObjects函数中，创建一个地面和一个砖墙：

```js
	createBox(new THREE.Vector3(0, -0.5, 0), ZERO_QUATERNION, 75, 1, 75, 0, 2);
	var quaternion = new THREE.Quaternion(0, 0, 0, 1);
	quaternion.setFromAxisAngle(new THREE.Vector3(1, 0, 0), -Math.PI / 18);
	createBox(new THREE.Vector3(0, -1.5, 0), quaternion, 8, 4, 10, 0);
	var size = .75;
	var nw = 8;
	var nh = 6;
	for (var j = 0; j < nw; j++)
		for (var i = 0; i < nh; i++)
			createBox(new THREE.Vector3(size * j - (size * (nw - 1)) / 2, size * i, 10), ZERO_QUATERNION, size, size, size, 10);
```

&#160; &#160; &#160; &#160;别忘了把updatePhysics函数也修改一下，添加：

```js
for (var i = 0; i < syncList.length; i++)
	syncList[i](deltaTime);
```

&#160; &#160; &#160; &#160;还可以更新`physicsWorld.stepSimulation(deltaTime)`为`physicsWorld.stepSimulation( deltaTime, 10 )`，这样可以限制物理引擎线性方程求解器的迭代次数为10次，已损失精度为代价，提高运行性能。

# 开始解决车辆问题

## 解决输入问题

&#160; &#160; &#160; &#160;为了解决输入问题，先添加两个全局变量，用来关联WSAD按键和加速、减速、左转、右转四个事件:

```js
// 键盘相关
    var actions = {
        'acceleration': false,
        'braking': false,
        'left': false,
        'right': false
    };
    var keysActions = {
        "KeyW":'acceleration',
        "KeyS":'braking',
        "KeyA":'left',
        "KeyD":'right'
    };
```

&#160; &#160; &#160; &#160;之后修改initInput函数，添加键盘按下和松开相关的两个事件绑定:

```js
window.addEventListener( 'keydown', function (e) {
    if(keysActions[e.code]) {
        actions[keysActions[e.code]] = true;
        e.preventDefault();
        e.stopPropagation();
        return false;
    }
});

window.addEventListener( 'keyup', function (e) {
    if(keysActions[e.code]) {
        actions[keysActions[e.code]] = false;
        e.preventDefault();
        e.stopPropagation();
        return false;
    }
})
```

## 车辆绘制的辅助函数

&#160; &#160; &#160; &#160;在车辆创建正式开始之前，先完成两个用来生成车轮和车身mesh的函数:

```js
// 绘制车轮
    function createWheelMesh(radius, width) {
        var t = new THREE.CylinderGeometry(radius, radius, width, 24, 1);
        t.rotateZ(Math.PI / 2);
        var mesh = new THREE.Mesh(t, createRendomColorObjectMeatrial());
        mesh.add(new THREE.Mesh(new THREE.BoxGeometry(width * 1.5, radius * 1.75, radius*.25, 1, 1, 1), createRendomColorObjectMeatrial()));
        scene.add(mesh);
        return mesh;
    }

// 绘制底盘
    function createChassisMesh(w, l, h) {
        var shape = new THREE.BoxGeometry(w, l, h, 1, 1, 1);
        var mesh = new THREE.Mesh(shape, createRendomColorObjectMeatrial());
        scene.add(mesh);
        return mesh;
    }
```

&#160; &#160; &#160; &#160;车轮是圆柱上嵌了个方块，用来让车轮转动的显示更加明显

## 完成createVehicle函数

&#160; &#160; &#160; &#160;createVehicle(pos, quat)函数创建大致分成四部分，

1. 创建车辆要用的局部变量
2. 创建车身
3. 添加车轮
4. 生成操作相关的sync(dt)函数并添加到syncList

### 1. 创建车辆要用的局部变量

&#160; &#160; &#160; &#160;先在createVehicle创建如下常量：

```js
// 车辆常量
var chassisWidth = 1.8;
var chassisHeight = .6;
var chassisLength = 4;        
var massVehicle = 800;        
var wheelAxisPositionBack = -1;        
var wheelRadiusBack = .4;        
var wheelWidthBack = .3;        
var wheelHalfTrackBack = 1;        
var wheelAxisHeightBack = .3;        
var wheelAxisFrontPosition = 1.7;        
var wheelHalfTrackFront = 1;        
var wheelAxisHeightFront = .3;        
var wheelRadiusFront = .35;        
var wheelWidthFront = .2;        
var friction = 1000;        
var suspensionStiffness = 20.0; // 悬挂刚性        
var suspensionDamping = 2.3;    // 悬挂阻尼        
var suspensionCompression = 4.4;// 悬挂压缩        
var suspensionRestLength = 0.6; // 悬挂能恢复的长度        
var rollInfluence = 0.2;        
var steeringIncrement = .04;        
var steeringClamp = .5;        
var maxEngineForce = 2000;        
var maxBreakingForce = 100;
```

### 2. 创建车身

&#160; &#160; &#160; &#160;之后是创建车身并在物理空间注册车辆对象：

```js
// 底盘
var geometry = new Ammo.btBoxShape(new Ammo.btVector3(chassisWidth * .5, chassisHeight * .5, chassisLength * .5));        
var transform = new Ammo.btTransform();        
transform.setIdentity();        
transform.setOrigin(new Ammo.btVector3(pos.x, pos.y, pos.z));        
transform.setRotation(new Ammo.btQuaternion(quat.x, quat.y, quat.z, quat.w));        
var motionState = new Ammo.btDefaultMotionState(transform);        
var localInertia = new Ammo.btVector3(0, 0, 0);        
geometry.calculateLocalInertia(massVehicle, localInertia);        
var body = new Ammo.btRigidBody(new Ammo.btRigidBodyConstructionInfo(massVehicle, motionState, geometry, localInertia));        
body.setActivationState(DISABLE_DEACTIVATION);        
physicsWorld.addRigidBody(body);        
var chassisMesh = createChassisMesh(chassisWidth, chassisHeight, chassisLength);

// Raycast Vehicle        
var engineForce = 0;        
var vehicleSteering = 0;        
var breakingForce = 0;        
var tuning = new Ammo.btVehicleTuning(); // 保存车辆参数配置        
var rayCaster = new Ammo.btDefaultVehicleRaycaster(physicsWorld);        
var vehicle = new Ammo.btRaycastVehicle(tuning, body, rayCaster);        
vehicle.setCoordinateSystem(0, 1, 2);        
physicsWorld.addAction(vehicle);
```

### 3. 添加车轮

&#160; &#160; &#160; &#160;添加车轮：

```js
// 车轮
var FRONT_LEFT = 0;
var FRONT_RIGHT = 1;
var BACK_LEFT = 2;
var BACK_RIGHT = 3;

var wheelMeshes = [];
var wheelDirectionCS0 = new Ammo.btVector3(0, -1, 0);
var wheelAxleCS = new Ammo.btVector3(-1, 0, 0);

function addWheel(isFront, pos, radius, width, index) {
  var wheelInfo = vehicle.addWheel(
    pos,
    wheelDirectionCS0,
    wheelAxleCS,
    suspensionRestLength,
    radius,
    tuning,
    isFront
  );
  wheelInfo.set_m_suspensionStiffness(suspensionStiffness);
  wheelInfo.set_m_wheelsDampingRelaxation(suspensionDamping);
  wheelInfo.set_m_wheelsDampingCompression(suspensionCompression);
  wheelInfo.set_m_frictionSlip(friction);
  wheelInfo.set_m_rollInfluence(rollInfluence);
  wheelMeshes[index] = createWheelMesh(radius, width);
}

addWheel(true, new Ammo.btVector3(wheelHalfTrackFront, wheelAxisHeightFront, wheelAxisFrontPosition), wheelRadiusFront, wheelWidthFront, FRONT_LEFT);
addWheel(true, new Ammo.btVector3(-wheelHalfTrackFront, wheelAxisHeightFront, wheelAxisFrontPosition), wheelRadiusFront, wheelWidthFront, FRONT_RIGHT);
addWheel(false, new Ammo.btVector3(-wheelHalfTrackBack, wheelAxisHeightBack, wheelAxisPositionBack), wheelRadiusBack, wheelWidthBack, BACK_LEFT);
addWheel(false, new Ammo.btVector3(wheelHalfTrackBack, wheelAxisHeightBack, wheelAxisPositionBack), wheelRadiusBack, wheelWidthBack, BACK_RIGHT);
```

&#160; &#160; &#160; &#160;**注意：**vehicle已经添加到physicsWorld，所以车轮添加到vehicle即可，不用再次显示调用addRigidBody将车轮添。

### 4. 生成操作相关的sync(dt)函数并添加到syncList

&#160; &#160; &#160; &#160;在createVehicle函数中添加

```js
// 将键盘输入,物理和绘制同步
function sync(dt) {
  var speed = vehicle.getCurrentSpeedKmHour();
  breakingForce = 0;
  engineForce = 0;
  if (actions.acceleration) {
    if (speed < -1)
      breakingForce = maxBreakingForce;
    else engineForce = maxEngineForce;
  }
  if (actions.braking) {
    if (speed > 1)
      breakingForce = maxBreakingForce;
    else engineForce = -maxEngineForce / 2;
  }
  if (actions.left) {
    if (vehicleSteering < steeringClamp)
      vehicleSteering += steeringIncrement;
  }
  else {
    if (actions.right) {
      if (vehicleSteering > -steeringClamp)
        vehicleSteering -= steeringIncrement;
    }
    else {
      if (vehicleSteering < -steeringIncrement)
        vehicleSteering += steeringIncrement;
      else {
        if (vehicleSteering > steeringIncrement)
          vehicleSteering -= steeringIncrement;
        else {
          vehicleSteering = 0;
        }
      }
    }
  }

  vehicle.applyEngineForce(engineForce, BACK_LEFT);
  vehicle.applyEngineForce(engineForce, BACK_RIGHT);
  vehicle.setBrake(breakingForce / 2, FRONT_LEFT);
  vehicle.setBrake(breakingForce / 2, FRONT_RIGHT);
  vehicle.setBrake(breakingForce, BACK_LEFT);
  vehicle.setBrake(breakingForce, BACK_RIGHT);
  vehicle.setSteeringValue(vehicleSteering, FRONT_LEFT);
  vehicle.setSteeringValue(vehicleSteering, FRONT_RIGHT);
  var tm, p, q, i;
  var n = vehicle.getNumWheels();
  for (i = 0; i < n; i++) {
    vehicle.updateWheelTransform(i, true);
    tm = vehicle.getWheelTransformWS(i);
    p = tm.getOrigin();
    q = tm.getRotation();
    wheelMeshes[i].position.set(p.x(), p.y(), p.z());
    wheelMeshes[i].quaternion.set(q.x(), q.y(), q.z(), q.w());
  }

  console.log(vehicle);

  tm = vehicle.getChassisWorldTransform();
  vehicle.getChassisWorldTransform();
  p = tm.getOrigin();
  q = tm.getRotation();
  chassisMesh.position.set(p.x(), p.y(), p.z());
  chassisMesh.quaternion.set(q.x(), q.y(), q.z(), q.w());

}

syncList.push(sync);
```

&#160; &#160; &#160; &#160;到此为止，车辆创建已经完成，在createObjects中添加`createVehicle(new THREE.Vector3(0, 4, -20), ZERO_QUATERNION);`，然后运行场景，就可以开始飙车了。

# 最后的锦上添花：显示车速

&#160; &#160; &#160; &#160;在html的body标签中，添加`<div id="speedometer">0.0 km/h</div>`，然后在initGraphics最后添加`speedometer = document.getElementById( 'speedometer' );`,然后修改createVehicle函数中的sync函数，添加`speedometer.innerHTML = (speed < 0 ? '(R) ' : '') + Math.abs(speed).toFixed(1) + ' km/h';`，然后再修改css调整速度表的显示位置

```css
#speedometer {
	position: absolute;
	color: white;
	background: #900;
	bottom: 0;
	padding: 5px;
}
```

&#160; &#160; &#160; &#160;到此为止再运行场景，就可以看到提示老司机飙车状态的时速显示了。

# 额外的JavaScript小知识

## JavaScript函数传值和传引用

&#160; &#160; &#160; &#160;相信熟悉c++的人，对于值传递和引用传递都不陌生。JS是动态语言，传值和传引用，就是靠判断参数类型决定的。

&#160; &#160; &#160; &#160;JS中简单类型一共有5种，分别为

1. Undefined类型
2. Null类型
3. Boolean类型
4. Number类型
5. String类型

&#160; &#160; &#160; &#160;5中简单类型都是传值调用，注意，Number类型有5个特殊常量，分别为：

1. 最大值： Number.MAX_VALUE
2. 最小值： Number.MIN_VALUE
3. 正无穷大：Infinity
4. 负无穷大：-Infinity
5. 非数字：NaN

&#160; &#160; &#160; &#160;JS中的复杂数据类型也有5种，分别是

1. Object类型
2. Array类型
3. Date类型
4. 函数类型
5. RegExp类型(正则表达式)

&#160; &#160; &#160; &#160;复杂数据类型的传递，均为传引用。

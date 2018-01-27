---
layout: post
title: "bullet物理引擎简单场景"
description: "bullet物理引擎简单场景"
category: c++
tags: [c++,bullet]
---

# 简介

&#160; &#160; &#160; &#160;[bullet物理引擎](http://bulletphysics.org/)是一个优秀的物理引擎。有开源、高效、小巧的特点。但是bullet引擎教程匮乏，官方教程说实话写的也不怎么样，对于想学习这个引擎的初学者来说，确实不太容易。

&#160; &#160; &#160; &#160;所以我在此再挖一个坑，就是编写一些bullet引擎相关的入门指引。不会很深入，就是入个门。不讲原理只讲实现，实用主义为主，希望能帮助大家在bullet使用上入门。

<!-- more -->

* Table of Contents
{:toc}

&#160; &#160; &#160; &#160;这个系列教程大概分成下面几课：

1. bullet引擎和OpenGL结合创建简单的场景
2. three.js和ammo.js创建简单的场景
3. 创建地形
4. 制作车辆
5. 柔体-绳索
6. 柔体-布料
7. 柔体-有体积的柔体
8. 使用blender引擎模拟物理场景

&#160; &#160; &#160; &#160;上面的教程1.是使用OpenGL(c++)进行开发，2~7均使用webgl(three.js)和ammo.js(bullet引擎的webgl版本)进行开发。教程8.则不是编程教程，反而更像blender的教程，希望能通过blender中使用bullet物理引擎的演示，来教大家一些物理模拟的相关知识。我本来是希望使用OpenGL完成全部教程，作为learnopengl系列教程的非官方扩展存在的。结果发生了很多蛋疼的问题。

> 以前用固定管线，glu真心是个好东西。现代OpenGL没有立即渲染模式(固定功能管线)，调试bullet很费劲。

&#160; &#160; &#160; &#160;所有demo在bullet官方的example中都能找到，希望通过这系列教程，帮大家在实用bullet引擎上入门。如果有可能，我也希望能把这系列教程制作视频教程，上传到网上。

# bullet引擎和OpenGL结合创建简单的场景

&#160; &#160; &#160; &#160;在简介里我已经说明，本系列教程是接着[learnopengl](https://learnopengl.com/)([中文版](https://learnopengl-cn.github.io/))走的。learnOpenGL是非常棒的现代openGL教程(Nehe的教程有些过时了)，通过这个系列能很好的学习可编程管线还有很多现代openGL技术。本教程假设你已经粗略掌握了可编程管线的相关技能，可以独自完成物体绘制。我在这个例子中使用了很多learnOpenGL系列教程的编写的辅助类，比如camera，model，shader。

## bullet引擎初始化

&#160; &#160; &#160; &#160;这部分我想稍微简单一点，因为网上关于bullet引擎的hello world的教程还是蛮多的(虽然大部分也就仅此而已了/(ㄒoㄒ)/~~)。网上bullet的hello world基本会把本教程中用到的bullet函数讲解一遍，我就不再赘述了。

&#160; &#160; &#160; &#160;创建几个物理引擎相关的全局变量

```c++
btDynamicsWorld* world;
btDispatcher* dispatcher;
btCollisionConfiguration* collisionConfiguration;
btBroadphaseInterface* broadphase;
btConstraintSolver* solver;
```

&#160; &#160; &#160; &#160;初始化物理引擎的btInit函数

```c++
void btInit()
{
    collisionConfiguration = new btDefaultCollisionConfiguration();
    dispatcher = new btCollisionDispatcher(collisionConfiguration);
    broadphase = new btDbvtBroadphase();
    solver = new btSequentialImpulseConstraintSolver();
    world = new btDiscreteDynamicsWorld(dispatcher, broadphase, solver, collisionConfiguration);
    world->setGravity(btVector3(0, -10, 0));
}
```

&#160; &#160; &#160; &#160;**注意：**btInit()需要在渲染循环之前进行，希望和最好和渲染准备分开。

## bullet引擎退出

&#160; &#160; &#160; &#160;bullet引擎退出的话，其实把几个物理引擎相关的实例析构即可。

```c++
void btExit()
{
    delete dispatcher;
    delete collisionConfiguration;
    delete solver;
    delete world;
    delete broadphase;
}
```

&#160; &#160; &#160; &#160;**注意：**在程序退出前执行

## 创建地面

&#160; &#160; &#160; &#160;在btInit函数后面添加如下代码

```c++
	btTransform t;
	t.setIdentity();
	t.setOrigin(btVector3(0, 0, 0));
	btStaticPlaneShape* plane = new btStaticPlaneShape(btVector3(0, 1, 0), 0);
    btMotionState* motion = new btDefaultMotionState(t);
    btRigidBody::btRigidBodyConstructionInfo info(0.0, motion, plane);
    btRigidBody* body = new btRigidBody(info);
    body->setRestitution(btScalar(0.5));
    world->addRigidBody(body);
```
&#160; &#160; &#160; &#160;**重要概念解释**

1. btRigidBodyConstructionInfo实例info的第一个参数质量设置为0表示该物体是一个静态物理
2. `btStaticPlaneShape`的第一个参数是地面的朝向，第二个参数0表示无限延伸

## 创建小球

&#160; &#160; &#160; &#160;我们接下来编写一个用来创建小球的函数。大体上可以参考创建地面的方法。

```c++
btRigidBody* addSphere(float radius, float x, float y, float z, float mass)
{
    btTransform t;
    t.setIdentity();
    t.setOrigin(btVector3(x, y, z));
    btSphereShape* sphere = new btSphereShape(radius);
    btVector3 inertia(0, 0, 0);
    if (mass != 0.0) {
        sphere->calculateLocalInertia(mass, inertia);
    }

    btMotionState* motion = new btDefaultMotionState(t);
    btRigidBody::btRigidBodyConstructionInfo info(mass, motion, sphere);
    btRigidBody* body = new btRigidBody(info);
    body->setRestitution(btScalar(0.5));
    world->addRigidBody(body);

    return body;
}
```

&#160; &#160; &#160; &#160;**重要概念解释**

1. `calculateLocalInertia`函数可以根据物体的质量和固有惯性计算在力场中的实际惯性。比如在垂直向下的重力场下，物体都有下落的趋势
2. `setRestitution`表示物体的恢复系数，范围[0,1]，越大弹力越强

&#160; &#160; &#160; &#160;接下来在btInit函数最后添加一行，用来向场景里添加小球。

```c++
addSphere(1.0, 0, 20, 0, 1.0);
```

## 场景绘制

&#160; &#160; &#160; &#160;到此为止，物理世界的准备就全部完成。依靠这些数据就已经可以在命令行中将各个物体的运动情况显示出来。但如果要在openGL中显示，就要考虑绘制的问题。也就是bullet空间和绘制空间的同步。

&#160; &#160; &#160; &#160;地面是静止的物体，用openGL正常的方法绘制一个平面即可。但是小球的绘制不太一样。

&#160; &#160; &#160; &#160;创建小球渲染函数，大概如下：

```c++
void renderSphere(btRigidBody* sphere, Shader &_shader)
{
    if (sphere->getCollisionShape()->getShapeType() != SPHERE_SHAPE_PROXYTYPE){
//        std::cout << RED << "sphere error" << RESET << std::endl;
        return;
    }
    float r = ((btSphereShape*)sphere->getCollisionShape())->getRadius();
    btTransform t;
    sphere->getMotionState()->getWorldTransform(t);

    float mat[16];
    t.getOpenGLMatrix(mat);

    glm::mat4 positionTrans = glm::make_mat4(mat);

    glm::mat4 model;
//    model = glm::translate(model, glm::vec3(0.0f, -1.75f, 0.0f));
    model = positionTrans * model;
    model = glm::scale(model, glm::vec3(r));
    glUniformMatrix4fv(glGetUniformLocation(_shader.Program, "model"), 1, GL_FALSE, glm::value_ptr(model));

    sphereModel->Draw(_shader);
}
```

&#160; &#160; &#160; &#160;**解释：**openGL中并没有直接绘制一个球体的函数，这个相当不爽，这也是之后的教程我准备用three.js完成的原因，openGL基础设施实在匮乏。其实openGL固定管线也没有球体相机等常用的辅助函数，那是glu的功能(⊙﹏⊙)b。在这里我很简单，就是使用learnopenGL中第三章中的模型加载类，直接加载一个球体的obj模型。利用Draw方法就可以绘制出小球。

&#160; &#160; &#160; &#160;其实用数学方法计算球体各个顶点的方法也是有的，球面上一点可以表示为

$$
x = r·cos{\alpha}cos{\beta} \\
y = r·sin{\alpha} \\
z = r·cos{\alpha}sin{\beta}
$$

&#160; &#160; &#160; &#160;alpha和beta两个角按照球面细分情况一次增加，就能求出球面各个顶点的坐标。

&#160; &#160; &#160; &#160;`t.getOpenGLMatrix`可以求出物体在空间物理空间中的位置的变换矩阵，和模型坐标相乘，就可以得到位移矩阵。

&#160; &#160; &#160; &#160;一定不要忘记在渲染循环中调用rednerSphere方法，不然是不会显示的。

## 按空格发射小球

&#160; &#160; &#160; &#160;实现的功能是按下空格键，就在摄像机位置沿着视线的方向，发射一个半径为1质量为1的小球，初速度为20。

&#160; &#160; &#160; &#160;先创建一个用来保存所有物理世界中物体的全局变量。

```c++
std::vector<btRigidBody*> bodies;
```

> 记得修改btInit和addSphere函数，把地面和每个小球都加到bodies变量中

&#160; &#160; &#160; &#160;在key_callback方法中添加如下代码：

```c++
    // 按空格发射小球
    if (key == GLFW_KEY_SPACE && action == GLFW_PRESS)
    {
        std::cout << "camera.Front: " << camera.Front.x << "," << camera.Front.y << "," << camera.Front.z << std::endl;
        btRigidBody* sphere = addSphere(1.0, camera.Position.x, camera.Position.y, camera.Position.z, 1.0);
        glm::vec3 look = camera.Front;
        sphere->setLinearVelocity(btVector3(20 * look.x, 20 * look.y, 20 * look.z));
    }
```

&#160; &#160; &#160; &#160;在渲染循环中修改绘制部分

```c++
    redBallShader.Use();
    glUniformMatrix4fv(glGetUniformLocation(redBallShader.Program, "view"), 1, GL_FALSE, glm::value_ptr(view));
    glUniformMatrix4fv(glGetUniformLocation(redBallShader.Program, "projection"), 1, GL_FALSE, glm::value_ptr(projection));
    for (int i = 0; i < bodies.size(); i++)
    {
        if (bodies[i]->getCollisionShape()->getShapeType() == SPHERE_SHAPE_PROXYTYPE) {
            renderSphere(bodies[i], redBallShader);
        }
    }
```

> 地面已经单独绘制了，所以判断绘制体是小球之后才绘制

## 运行效果示意图

![运行效果](https://github.com/THISISAGOODNAME/learnopengl-glitter/raw/master/other/bullet%20physics/Hello%20RigidBody/GIF.gif)
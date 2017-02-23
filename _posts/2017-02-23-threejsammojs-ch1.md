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
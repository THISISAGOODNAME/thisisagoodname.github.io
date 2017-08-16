---
layout: post
title: "PMX format"
description: "PMX读取测试"
category: html5
tags: [html5,webGL]
---

&nbsp; &nbsp; &nbsp; &nbsp;好久没写博客了，今天加班很晚，睡前水一发吧，而且关于pmx格式解析的文章很早以前就想写了。

<!-- more -->

* Table of Contents
{:toc}

# PMX格式简述

&nbsp; &nbsp; &nbsp; &nbsp;估计看标题点进来的人应该没有不知道的了吧，pmx是MMD使用的第二代模型格式，相比上一代格式PMD，有几个显著的特征：

1. 支持顶点数量更多，PMD只支持65535个顶点，而PMX支持2^32个顶点，也就是42亿+个顶点
2. 采用新的变形方式SDEF(以前是BDEF)，关节扭转更自然，不容易发生扭曲

# PMX文件结构

Header |	|	Ver
------|-----|-------
Model information	| 	 | 2.0
[int] Vertex count	| Vertices	| 2.0
[int] Surface count	| Surfaces	| 2.0
[int] Texture count	| Textures	| 2.0
[int] Material count	| Materials	| 2.0
[int] Bone count	| Bones	| 2.0
[int] Morph count	| Morphs	| 2.0
[int] Displayframe count	| Displayframes	| 2.0
[int] Rigidbody count	| Rigidbodies	| 2.0
[int] Joint count	| Joints	| 2.0
[int] SoftBody count	| SoftBodies	| 2.1

> pmx一共有两个版本，分别是pmx2.0和pmx2.1，pmx2.1。pmx2.1比2.0版本添加了bullet柔体支持(X摇视频就是靠这玩意)等特性

# PMX文件简单读取

&nbsp; &nbsp; &nbsp; &nbsp;想显示pmx文件，最简单的情况，只需要读取出面信息即可，也就是需要顶点信息

```javascript
function PMXHeader(buffer){
    this.signature = buffer.readByte(4);
    this.version = buffer.readFloat();
    this.globalsCount = buffer.readByte(1)[0];
    this.globals = buffer.readByte(this.globalsCount);
    this.nameLocal = new Text();
    this.nameLocal.readFromFile(buffer);
    this.nameUniversal = new Text();
    this.nameUniversal.readFromFile(buffer);
    this.commentsLocal = new Text();
    this.commentsLocal.readFromFile(buffer);
    this.commentsUniversal = new Text();
    this.commentsUniversal.readFromFile(buffer);
}

function PMXVertex(buffer, addVec4, boneIndex){
    this.position = new Vec3();
    this.position.readFromFile(buffer);
    this.normal = new Vec3();
    this.normal.readFromFile(buffer);
    this.uv = new Vec2();
    this.uv.readFromFile(buffer);
    
	...
}

function PMXFile(buffer){
    this.header = new PMXHeader(buffer);

    this.vertexCount = buffer.readUint();
    this.vertices = [];
    this.vertexData = new Float32Array(this.vertexCount*3);
    this.uvData = new Float32Array(this.vertexCount*2);
    for(var i = 0; i < this.vertexCount; i++){
        var vertex = new PMXVertex(buffer,this.header.globals[1], this.header.globals[5]);
        this.vertices.push(vertex);
        var vertexData = vertex.position.getData();
        this.vertexData[3*i] = vertexData[0];
        this.vertexData[3*i+1] = vertexData[1];
        this.vertexData[3*i+2] = vertexData[2];
        this.uvData[2*i] = vertex.uv.x;
        this.uvData[2*i+1] = vertex.uv.y;
    }
}
```

> `PMXFile.vertexData`即为顶点xyz坐标，`PMXFile.uvData`即为顶点纹理uv坐标。`PMXFile.vertexData.length/3`和`PMXFile.uvData。length/2`相等，就是顶点数量。

&nbsp; &nbsp; &nbsp; &nbsp;[pmx完整的加载源码](https://github.com/THISISAGOODNAME/PMX-viewer-webgl2/blob/gh-pages/pmx.js)

# 项目地址和demo

## 项目地址

&nbsp; &nbsp; &nbsp; &nbsp;[项目地址](https://github.com/THISISAGOODNAME/PMX-viewer-webgl2)

## demo

&nbsp; &nbsp; &nbsp; &nbsp;[webgl2版](http://aicdg.com/PMX-viewer-webgl2/index.html) (需要支持webgl2的浏览器，chrome 58+、firefox 53+)

&nbsp; &nbsp; &nbsp; &nbsp;[webgl版](http://aicdg.com/PMX-viewer-webgl2/index-webgl.html)

> 加载比较慢，可以打开控制终端看是否开始加载以及加载情况

# pmx文件格式完整文档

<script src="https://gist.github.com/felixjones/f8a06bd48f9da9a4539f.js"></script>
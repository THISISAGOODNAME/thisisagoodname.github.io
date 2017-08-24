---
layout: post
title: "webgpu小试"
description: "webgpu小试"
category: html5
tags: [html5,前端技术]
---

&nbsp; &nbsp; &nbsp; &nbsp;WebGL2从出生到落伍。

<!-- more -->

* Table of Contents
{:toc}

&nbsp; &nbsp; &nbsp; &nbsp;我知道WebGPU是几个月之前，那时候对WebGPU的理解，是Web版的Metal。我曾经狂吹过一波Metal Shading Language,理由很弱智：MSL的语法比GLSL高级些(类似C++相比C的进步)，带模块化系统，离线编译，泛型/类/命名空间/位运算符支持。。。基于C++11(MSL2基于C++14)设计的新shader怎么看怎么屌，但是Metal的API就不太亲民了，只有objc的接口，虽然只在苹果家平台上能用，不用考虑跨平台的问题，但是就是觉得有些蛋疼。

&nbsp; &nbsp; &nbsp; &nbsp;不过，WebGPU出现了，JS接口，终于看着舒服点了，也让我有机会切身体验一下MSL的魅力。

&nbsp; &nbsp; &nbsp; &nbsp;WebGPU现阶段只能在[Safari技术预览版](https://developer.apple.com/safari/technology-preview/)上使用，没有文档，接口也随时会改变，所以本文就是个尝个鲜，很可能过个两三天Demo就用不了了。

# Metal执行过程，和Vulkan比较

&nbsp; &nbsp; &nbsp; &nbsp;还是使用图形学最喜闻乐见的画三角来讲述一下最简单WebGPU程序的的执行过程

## 运行vulkan，大致要准备如下

1. Instance and Devices
2. Image Presentation
3. Command Buffers and Synchronization
4. Resources and Memory
5. Descriptor Sets
6. Render Passes and Framebuffers
7. Shaders
8. Graphics/Compute Pipelines
9. Command Recording and Drawing

&nbsp; &nbsp; &nbsp; &nbsp;以上也是Vulkan Cookbook的前9章目录.Vulkan Cookbook是个非常非常好的Vulkan入门教程，想要入门Vulkan的小伙伴一定不要错过。

## Metal执行过程

### 1.获取上下文

&nbsp; &nbsp; &nbsp; &nbsp;和绝大部分图形API一样，需要先获取context。这一步其实相当于浏览器底层帮你完成了Instance and Devices的准备工作(比vulkan方便太多啊，如果自己写个Function Loader。。。全是泪)。

```javascript
let gpu;

let canvas = document.querySelector("canvas");
let canvasSize = canvas.getBoundingClientRect();
canvas.width = canvasSize.width;
canvas.height = canvasSize.height;
        
gpu = canvas.getContext("webgpu");
```

### 2. 创建CommandQueue

&nbsp; &nbsp; &nbsp; &nbsp;然后创建commandQueue

```javascript
let commandQueue;
commandQueue = gpu.createCommandQueue();
```

### 3.编译链接Shaders

&nbsp; &nbsp; &nbsp; &nbsp;编译并连接Shaders，说一下，WebGPU的编译连接shader的步骤确实比webGL(openGL)简单太多了

```javascript
let library = gpu.createLibrary(document.getElementById("shader").text);
let vertexFunction = library.functionWithName("vertex_main");
let fragmentFunction = library.functionWithName("fragment_main");
if (!library || !fragmentFunction || !vertexFunction) {
	return;
}
```

### 4.生成PipelineDescriptor

&nbsp; &nbsp; &nbsp; &nbsp;生成PipelineDescriptor

```javascript
let pipelineDescriptor = new WebGPURenderPipelineDescriptor();
pipelineDescriptor.vertexFunction = vertexFunction;
pipelineDescriptor.fragmentFunction = fragmentFunction;
// NOTE: Our API proposal has these values as enums, not constant numbers.
// We haven't got around to implementing the enums yet.
pipelineDescriptor.colorAttachments[0].pixelFormat = gpu.PixelFormatBGRA8Unorm;
```

### 5.生成renderPassDescriptor

&nbsp; &nbsp; &nbsp; &nbsp;生成renderPassDescriptor

```javascript
let renderPassDescriptor;
renderPassDescriptor = new WebGPURenderPassDescriptor();
// NOTE: Our API proposal has some of these values as enums, not constant numbers.
// We haven't got around to implementing the enums yet.
renderPassDescriptor.colorAttachments[0].loadAction = gpu.LoadActionClear;
renderPassDescriptor.colorAttachments[0].storeAction = gpu.StoreActionStore;
renderPassDescriptor.colorAttachments[0].clearColor = [0.0, 0.0, 0.0, 1.0];
```

### 6.生成renderPipelineState

&nbsp; &nbsp; &nbsp; &nbsp;生成renderPipelineState，和vulkan设置pipeline states作用相同

```javascript
let renderPipelineState;
renderPipelineState = gpu.createRenderPipelineState(pipelineDescriptor);
```

### 7.创建VAO、VBO

&nbsp; &nbsp; &nbsp; &nbsp;创建VAO、VBO向顶点着色器传递数据。

```javascript
let vertexData, vertexBuffer;
vertexData = new Float32Array([
// x y z 1 r g b 1
	    0,  0.75, 0, 1, 1, 0, 0, 1,
	-0.75, -0.75, 0, 1, 0, 1, 0, 1,
	 0.75, -0.75, 0, 1, 0, 0, 1, 1
]);
vertexBuffer = gpu.createBuffer(vertexData);
```

### 8.生成commandEncoder执行Command Buffer

&nbsp; &nbsp; &nbsp; &nbsp;接下来就需要讲Command Buffer执行即可(对应vulkan的Command Recording and Drawing步骤)

```javascript
let commandBuffer = commandQueue.createCommandBuffer();
let drawable = gpu.nextDrawable();
// Similat to Vulkan image
renderPassDescriptor.colorAttachments[0].texture = drawable.texture;

let commandEncoder = commandBuffer.createRenderCommandEncoderWithDescriptor(renderPassDescriptor);
commandEncoder.setRenderPipelineState(renderPipelineState);
        
// to use for the geometry.
commandEncoder.setVertexBuffer(vertexBuffer, 0, 0);
        
// NOTE: Our API proposal uses the enum value "triangle" here. We haven't got around to implementing the enums yet.
commandEncoder.drawPrimitives(gpu.PrimitiveTypeTriangle, 0, 3);

commandEncoder.endEncoding();
commandBuffer.presentDrawable(drawable);
commandBuffer.commit();
```

&nbsp; &nbsp; &nbsp; &nbsp;到此为止，最简单的Metal程序就算完成了，[完整源码](https://github.com/THISISAGOODNAME/fuckFrontEnd2017/blob/master/webgpu/HelloTriangle/HelloTriangle.html)，记住，暂时只能在Safari Technology Preview上运行。

&nbsp; &nbsp; &nbsp; &nbsp;我还扒了[Webkit博客上的demo](https://webkit.org/demos/webgpu/)，[项目地址](https://github.com/THISISAGOODNAME/fuckFrontEnd2017)

# 一些题外话

&nbsp; &nbsp; &nbsp; &nbsp;图形API今年真是井喷啊，3D Portable API，Open XR(面向厂商而非开发者)，NXT，webgpu，gpuweb。其中面向web的就有三个，分别是NXT、webgpu、gpuweb，三者的背后各是谷歌、苹果、W3C&Khronos。在WebGL2.0进入chrome和firefox主线版本之后，没过两个月就进入了下代标准的制定了，只能感叹web对于高性能图形的需求还真是不低。

&nbsp; &nbsp; &nbsp; &nbsp;今年先是WebAssembly，然后就是下代web图形标准，js社区稍微消停点了结果前端界还是迎来了重大变革啊。我创建了一个repo，叫[fuckFrontEnd2017](https://github.com/THISISAGOODNAME/fuckFrontEnd2017)，用来记录17年前端界的一些新坑。也希望小伙伴们能和我一起，丰富这个repo，一起记录17年前端的新气象。
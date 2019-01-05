---
layout: post
title: "openGL渲染"
description: "openGL渲染"
image: ""
date: 2019-01-05 15:40:08
categories: 研究
tags: [研究]
---

&nbsp; &nbsp; &nbsp; &nbsp;最近美术想了一个需求，想要实现全屏同时播放上千个粒子系统。这个需求确实挺疯狂的，因为对于CPU粒子来说，改动很大；而对于GPU粒子来说基本不可能(?)。老大对于这个需求非常质疑，我本人倒是很感兴趣，在寻找渲染实例化对象的实例化的(InstancedBaseInstance)方法的时候，发现高版本openGL的很多有意思的地方。在本文，总结一下高版本openGL draw call的类型。

<!-- more -->
* Table of Contents
{:toc}

# Direct rendering

&nbsp; &nbsp; &nbsp; &nbsp;Direct rendering和其他drawcall类型最显著的区别，就是Direct rendering的所有数据都是从CPU传来的，而其他的渲染方式都有数据是从openGl object(GPU)传来。

## Basic Drawing

```c++
void glDrawArrays( GLenum mode​, GLint first​, GLsizei count​ );
void glDrawElements( GLenum mode​, GLsizei count​, GLenum type​, void * indices​ );
```

## Multi-Draw

&nbsp; &nbsp; &nbsp; &nbsp;创建或者修改VAO都是一个很费的操作，因此有了这个功能，在一个VAO中存储多个mesh，然后批量绘制(batch)

```c++
void glMultiDrawArrays( GLenum mode​, GLint *first​, GLsizei *count​, GLsizei primcount​);
void glMultiDrawElements( GLenum mode​, GLsizei *count​, GLenum type​, void **indices​, GLsizei primcount​ );
```

&nbsp; &nbsp; &nbsp; &nbsp;实际上相当于

```c++
void glMultiDrawArrays( GLenum mode, GLint *first, GLsizei *count, GLsizei primcount )
{
	for (int i = 0; i < primcount; i++)
	{
		if (count[i] > 0)
			glDrawArrays(mode, first[i], count[i]);
	}
}

void glMultiDrawElements( GLenum mode, GLsizei *count, GLenum type, void **indices, GLsizei primcount )
{
	for (int i = 0; i < primcount; i++)
	{
		if (count[i]) > 0)
			glDrawElements(mode, count[i], type, indices[i]);
	}
}
```

> Multi-Draw所有物体使用相同材质

## Base Index

&nbsp; &nbsp; &nbsp; &nbsp;将多个模型存在同一个VBO中，如下所示

```c++
[A00 A01 A02 A03 A04... Ann B00 B01 B02... Bmm]
```

&nbsp; &nbsp; &nbsp; &nbsp;`glDrawArrays`的话指定start index和count就能画出指定的模型了，`glDrawElements`却存在一些小问题，就是B的索引并非*m*，而是*m + nn*。调整一下其实没多麻烦，但openGL还是提供了接口，可以选择一个basevertex作为索引基础(nn)，其他的索引自动加上这个basevertex的索引

```c++
void glDrawElementsBaseVertex( GLenum mode​, GLsizei count​,
    GLenum type​, void *indices​, GLint basevertex​);
```

## Instancing

&nbsp; &nbsp; &nbsp; &nbsp;这个应该都很熟悉了，不多介绍了

```c++
void glDrawArraysInstanced( GLenum mode​, GLint first​,
    GLsizei count​, GLsizei instancecount​ );
void glDrawElementsInstanced( GLenum mode​, GLsizei count​, 
    GLenum type​, const void *indices​, GLsizei instancecount​ );
```

> `gl_InstanceID`还有[Instanced_Array](https://www.khronos.org/opengl/wiki/Instanced_Array)是实例之间的唯一差距。

&nbsp; &nbsp; &nbsp; &nbsp;在 openGL 4.2 版本或者支持扩展 [ARB_base_instance](http://www.opengl.org/registry/specs/ARB/base_instance.txt)，实例化的实例化已经成为可能！

```c++
void glDrawArraysInstancedBaseInstance( GLenum mode​, GLint first​,
    GLsizei count​, GLsizei instancecount​, GLuint baseinstance​​ );
void glDrawElementsInstancedBaseInstance( GLenum mode​, GLsizei count​, 
    GLenum type​, const void *indices​, GLsizei instancecount​, GLuint baseinstance​​ );
```

> `gl_InstanceID`不随*baseinstance​​*变化，因此每实例差异只能靠[Instanced_Array](https://www.khronos.org/opengl/wiki/Instanced_Array)来实现。在 openGL 4.6 或者 [ARB_shader_draw_parameters](http://www.opengl.org/registry/specs/ARB/shader_draw_parameters.txt)扩展的情况下，可以通过`gl_BaseInstance`访问函数中的*baseinstance*参数

## Range

&nbsp; &nbsp; &nbsp; &nbsp;“Range”版本的`glDrawElements`也用来处理多个模型存储在同一个VBO的情况，但是不会有索引数组越界的错误

```c++
void glDrawRangeElements( GLenum mode​, GLuint start​, 
    GLuint end​, GLsizei count​, GLenum type​, void *indices​ );
```

## Combinations

&nbsp; &nbsp; &nbsp; &nbsp;Combinations和openGL的primitive restart特性紧密相关。

&nbsp; &nbsp; &nbsp; &nbsp;Base vertex可以和MultiDraw, Range, 或 Instancing结合

```c++
void glMultiDrawElementsBaseVertex( GLenum mode​, 
    GLsizei *count​, GLenum type​, void **indices​, 
    GLsizei primcount​, GLint *basevertex​ );
 
void glDrawRangeElementsBaseVertex( GLenum mode​, 
    GLuint start​, GLuint end​, GLsizei count​, GLenum type​, 
    void *indices​, GLint basevertex​ );

void glDrawElementsInstancedBaseVertex( GLenum mode​, 
    GLsizei count​, GLenum type​, const void *indices​, 
    GLsizei instancecount​, GLint basevertex​ );
```

&nbsp; &nbsp; &nbsp; &nbsp;在openGL 4.2版本或者[ARB_base_instance](http://www.opengl.org/registry/specs/ARB/base_instance.txt)，BaseVertex 也能和 BaseInstance 结合

```c++
void glDrawElementsInstancedBaseVertexBaseInstance( GLenum mode​, 
    GLsizei count​, GLenum type​, const void *indices​, 
    GLsizei instancecount​, GLint basevertex​, GLuint baseinstance​);
```

# Transform feedback rendering

&nbsp; &nbsp; &nbsp; &nbsp;可以使用[Transform Feedback](https://www.khronos.org/opengl/wiki/Transform_Feedback)来生成顶点信息用于渲染。通常的数据流程是GPU->CPU->GPU，但有一些API允许在CPU读取确切数据之前直接绘制。

```c++
void glDrawTransformFeedback(GLenum mode​, GLuint id​);
void glDrawTransformFeedbackStream(GLenum mode​, GLuint id​, GLuint stream​);
```

&nbsp; &nbsp; &nbsp; &nbsp;在 openGL 4.2版本或者支持ARB_transform_feedback_instanced 的设备上，上述两个函数的实例化版本也可以使用

```c++
void glDrawTransformFeedbackInstanced(GLenum mode​, GLuint id​, GLsizei instancecount​);
void glDrawTransformFeedbackStreamInstanced(GLenum mode​, GLuint id​, GLuint stream​, GLsizei instancecount​);
```

# Indirect rendering

&nbsp; &nbsp; &nbsp; &nbsp;该特性的目的是允许直接渲染GPU中的数据，省去GPU->CPU->GPU的拷贝过程。运行时数据来源可能是Compute Shader，也可能是Geometry Shader + Transform Feedback，甚至可能是openCL/CUDA。

&nbsp; &nbsp; &nbsp; &nbsp;所有Indirect rendering都适用于下列情况：

- Indexed rendering
    - Base vertex (for indexed rendering)
- Instanced rendering
- Base instance (if OpenGL 4.2 or ARB_base_instance is available). Does not require rendering more than one instance

```c++
void glDrawArraysIndirect(GLenum mode​, const void *indirect​);
```

&nbsp; &nbsp; &nbsp; &nbsp;此函数功能上等价于

```c++
typedef  struct {
   GLuint  count;
   GLuint  instanceCount;
   GLuint  first;
   GLuint  baseInstance;
} DrawArraysIndirectCommand;

glDrawArraysInstancedBaseInstance(mode, cmd->first, cmd->count, cmd->instanceCount, cmd->baseInstance);
```

&nbsp; &nbsp; &nbsp; &nbsp;在openGL 4.3或者[ARB_multi_draw_indirect](http://www.opengl.org/registry/specs/ARB/multi_draw_indirect.txt)扩展下，也可以用于multi draw

```c++
void glMultiDrawArraysIndirect(GLenum mode​, const void *indirect​, GLsizei drawcount​, GLsizei stride​);
```

&nbsp; &nbsp; &nbsp; &nbsp;对于索引渲染

```c++
void glDrawElementsIndirect(GLenum mode​, GLenum type​, const void *indirect​);
```

&nbsp; &nbsp; &nbsp; &nbsp;在功能上等价于

```c++
typedef  struct {
    GLuint  count;
    GLuint  instanceCount;
    GLuint  firstIndex;
    GLuint  baseVertex;
    GLuint  baseInstance;
} DrawElementsIndirectCommand;

glDrawElementsInstancedBaseVertexBaseInstance(mode, cmd->count, type,
  cmd->firstIndex * size-of-type, cmd->instanceCount, cmd->baseVertex, cmd->baseInstance);
```
&nbsp; &nbsp; &nbsp; &nbsp;在openGL 4.3或者[ARB_multi_draw_indirect](http://www.opengl.org/registry/specs/ARB/multi_draw_indirect.txt)扩展下，也可以用于multi draw

```c++
void glMultiDrawElementsIndirect(GLenum mode​, GLenum type​, const void *indirect​, GLsizei drawcount​, GLsizei stride​);
```

# Conditional rendering

&nbsp; &nbsp; &nbsp; &nbsp;条件渲染就是只在某些条件下渲染，在某些条件下就不渲染。最经典的运用就是Occlusion Query。

```c++
glBeginConditionalRender(GLuint id​, GLenum mode​);
glEndConditionalRender();
```

&nbsp; &nbsp; &nbsp; &nbsp;支持的命令如下

- Every function previously mentioned. IE, all functions of the form `glDraw*` or `glMultiDraw*`.
- `glClear` and `glClearBuffer`.
- `glDispatchCompute` and `glDispatchComputeIndirect`.

参数中*mode*可以是如下选项：

- **GL_QUERY_WAIT**​: OpenGL will wait until the query result is returned, then decide whether to execute the rendering command. This ensures that the rendering commands will only be executed if the query fails. Note that it is OpenGL that's waiting, not (necessarily) the CPU.
- **GL_QUERY_NO_WAIT​**: OpenGL may execute the rendering commands anyway. It will not wait to see if the query test is true or not. This is used to prevent pipeline stalls if the time between the query test and the execution of the rendering commands is too short.
- **GL_QUERY_BY_REGION_WAIT**: OpenGL will wait until the query result is returned, then decide whether to execute the rendering command. However, the rendered results will be clipped to the samples that were actually rasterized in the occlusion query. Thus, the rendered result can never appear outside of the occlusion query area.
- **GL_QUERY_BY_REGION_NO_WAIT**: As above, except that it may not wait until the occlusion query is finished. The region clipping still holds.


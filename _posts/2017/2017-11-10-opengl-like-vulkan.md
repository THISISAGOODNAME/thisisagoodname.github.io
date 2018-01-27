---
layout: post
title: "[转载]How to make OpenGL usage Vulkan like"
description: "[转载]How to make OpenGL usage Vulkan like"
category: C++
tags: [c++,图形学,OpenGL,Vulkan]
---

&nbsp; &nbsp; &nbsp; &nbsp;现代openGL有很多新式扩展，通过一些高版本的openGL扩展的使用，可以让openGL的开发非常vulkan like(不光开发起来像，性能当然也是有长足的提高)，本文节选了API相关的部分，[原文链接](https://developer.nvidia.com/opengl-vulkan)

<!-- more -->

# 1. Object Management

Vulkan follows an object-oriented design and we always manipulate objects directly with their handles.

- **ARB\_direct\_state\_access** (DSA), OpenGL 4.5 core, allows direct manipulation of objects (textures, buffers, fbos…) rather than the classic “bind to edit”. Using DSA makes object manipulation cleaner as it doesn’t affect the binding state used for rendering, therefore is also middleware friendly.
- Within the **rendering hot loop** the new set of binding calls is encouraged, as they only affect rendering state:

	- glBindBuffersBase / glBindBuffersRange for UNIFORM, SHADER\_STORAGE, ATOMIC\_COUNTER, TRANSFORM\_FEEDBACK
	- glBindVertexBuffer(s)
	- glBindFramebuffer
	- glBindTextures / glBindTextureUnit
	- glBindBuffer would most likely only be used with the ELEMENT\_ARRAY, DRAW\_INDIRECT in the rendering loop

“Bind to Edit” can still be beneficial in the hot loop, for example when the same buffer is updated very often by glBufferSubData, or glUniforms for the active program are changed. In this case the “indirection” by passing the object handle, may cause additional work. Some of these functions may also be available under EXT\_direct\_state\_access, the primary difference between ARB and EXT is that ARB requires the use of glCreate”Resource” rather than working from glGen”Resource” object handles.

# 2. Resource Management

- **ARB\_buffer\_storage** (core 4.4) should be preferred over the classic glBufferData, as it gives better usage hints than glBufferData. Ideally the developer uses persistent mapped buffers for all buffers that he expects to for reading or writing access on the CPU. This would allow an identical workflow to Vulkan. We also encourage the use of few but rather large buffers as suggested in the memory management blog post.
- **ARB\_texture\_storage** (core 4.2) provides immutable textures whose definition is complete up-front. The classic glTexImage never “knew” how many mip-maps would actually be specified, often causing lazy allocation. glTextureStorage is the better alternative for the driver.
- **ARB\_texture\_view** (core 4.3) introduced the concept of casting texture formats to different types as long as they have the same texel size. The developer can also use it as view on a sub-resource, for example individual texture layer or mipmap in an array texture. Similar to using larger buffers and sub-allocating from those, a developer could use texture arrays for popular dimensions and re-use individual layers. It is not as flexible as the Vulkan texture memory management, but it can reduce run-time texture creation costs.

# 3. Command Buffer / Faster Rendering

Even in a single-threaded scenario Vulkan’s command buffers provide efficient ways to render data: recording a command-buffer is faster than traditional OpenGL commands and command buffers can be re-used. In theory OpenGL’s display lists allow re-use, in practice they are only beneficial for a limited functionality. However, there is still some other ways to speed up rendering in OpenGL.

- **ARB\_multi\_draw\_indirect** (MDI), OpenGL 4.3 core, allows accelerating draw-call submission. Compared to instancing we can draw any sub-range of vertex/index-buffers. To make most use of this feature use the previous encouraged larger buffers for geometry data, as it will allow drawing many geometries at once via MDI. The MDI buffer can also be filled from different threads using persistent mapped memory, whose pointers are passed to threads that don’t need an OpenGL context. The buffer content does not have to be filled every frame, but can be re-used and even manipulated by the GPU directly, for example for level of detail or culling purposes.

NVIDIA provides various extensions to make the hot-loop faster

- **NV\_command\_list** is the latest extension and comes closest to Vulkan’s command buffers and provides very fast ways to record and submit commands that are most common in the rendering hot-loop. Just like in Vulkan, it is possible to re-use those command-buffers as well. 
We encourage you to have a look at the GTC presentation as well as samples on github.

	The draw-indirect like buffers encode tokens for typical binding state (UBO, VBO, EBO) and draw-calls as well as minor state changes such as front-face or viewport dimensions. Due to its binary nature it is the fastest way to record commands (can also be used threaded) and provides two alternate render modes, either as pre-compiled object, or interpreted data from buffer objects. Due to state inheritance and the ability to stitch sequences from arbitrary buffer addresses, it can actually be faster than Vulkan’s secondary command buffers.

- **Bindless Rendering** the NV\_command\_list builds on top of previous extensions that improved the rendering hot-loop performance by using native GPU addresses, and therefore avoiding per-object handle lookups and validation.

	- NV\_uniform\_buffer\_unified_memory
	- NV\_vertex\_buffer\_unified_memory
	- NV\_shader\_buffer\_load/store

	The core OpenGL alternative to “bindless” is “binding less” by using large buffers and array textures and manually managing the sub-allocations. The bindless extensions let the driver still manage the allocations and objects, while avoiding some of the negatives at rendering time.

# 4. Multi-Threading

- **Threaded Validation and Submission** is an optimization that NVIDIA’s OpenGL driver may use to off-load work on a dedicated thread. The application thread remains very fast in this scenario, and the OpenGL commands are forwarded for processing to the worker thread. Applications should avoid commands that need synchronization between the treads, such as glGets or map/unmap (prefer persistent mapped, or BufferSubData).
- **Bindless** or **Binding Less** on NVIDIA drivers will also improve the performance in threaded shared OpenGL contexts. To ensure consistency of objects across the threads, the OpenGL driver has to use locks more conservatively. In Vulkan this is the application’s responsibility. Doing less binds means a reduction of the impact of these locks, with bindless we can further avoid them.
- **Persistent Mapped Buffers** (**ARB\_buffer\_storage**, core 4.4) allow us to pass buffer pointers to threads for writing and not using OpenGL at all. This is useful for both raw data and commands being encoded for ARB\_multi\_draw\_indirect or NV\_command\_list buffers.
OpenGL however does not provide many of the other explicit Vulkan features around threading, especially fast submission to the same graphics queue.

# 5. Shader Resource/Uniform Binding

- **OpenGL 4.3 explicit bindings** are instructions inside GLSL that allow us to manage our bindings up-front. Vulkan’s bindings are also hard-coded with SPIR-V and Vulkan does not allow querying binding information, so preparing yourself for the management of this is mandatory. Vulkan supports faster bindings and shader changes when the bindings use increasing units. Low binding units should reflect data that is shared for many operations and shaders.

```glsl
// define vertex attribute locations
layout(location=0) in vec3 attr_position;

// ubo/ssbo bindings
layout(binding=1,std140) uniform objectInfo {
  mat4 worldMatrix;
};
// texture/image units
layout(binding=0) uniform samplerCube envCube;
layout(binding=1) uniform sampler2D   objectLightmap;
```

- **ARB\_uniform\_buffer\_object** are the de-facto standard now to pass uniforms. Therefore migrate your uniform usage to UBOs, and ideally group uniforms by frequency of change into dedicated uniform blocks to maximize re-use. By using larger buffer and glBindBufferRange with offsets you can improve performance to switch between them. This method is also encouraged for Vulkan.

```glsl
// commonly used
layout(binding=0,std140) uniform viewInfo {
  mat4 viewProjectionMatrix;
  vec2 viewportSize;
};

// for shaderA
// changed content depending on material
layout(binding=1,std140) uniform shaderAInfo {
  vec4  someControlValue;
  float andAnother;
  ...
};

// for shader B a different definition using same binding lost
layout(binding=1,std140) uniform shaderBInfo {
  float blah;
  int   blubb;
  ...
};
```

- **NV/ARB\_bindless\_texture** speeds up the binding of textures by using native handles instead of texture-objects. UBOs and bindless textures work fairly similar to Vulkan’s DescriptorSets.

```glsl
layout(bindless_sampler,location=1) uniform samplerCube envReflectionProbe;
// updated via glUniformHandleui64ARB(1, texHandle);

// directly store texture as 64-bit value in the UBO
layout(binding=1,std140) uniform shaderAInfo {
    sampler2D  texAlbedo; 
    sampler2D  texNormal;
    float      bumpIntensity;
};
```

- **Uniform Arrays** could be useful for small data changes, and allow us to mimic Vulkan’s “PushConstants”

```glsl
// pack variables in a single array
layout(location=0) uniform vec4 pushConstants[2];
// wrap accessors
#define myScale pushConstants[0]
#define myIdx   floatBitsToInt(pushConstants[1].x)
void main(){
  ...
  ... texelFetch(someDataBuffer, myIdx);
  ...
}
// alternatively use a single small UBO and glBufferSubData its content
// NVIDIA optimizes glBufferSubData for buffers that are only used as UBO
```

# 6. State Management

Vulkan will not offer most of the generic state querying capabilities (other than the hardware capabilities), even for OpenGL relying on glGets or glIsEnabled is not good for performance. It is best if the application manages its own state in a compact way that is CPU cache-friendly. Unextended OpenGL doesn’t have that much in terms of pre-validated state either, however there is still some ways to minimize validation costs.

- **ARB\_vertex\_attrib\_binding** allows to define vertex format specification independent of vertex buffer bindings. Due to the expected higher frequency as well as “offset” usage, we encourage changing bindings via glBindVertexBuffers or their bindless equivalent glBufferAddressRangeNV rather than using vertex array objects. Changing the vertex-format specification is typically rarely done, as result the classic glVertexAttribPointer approach of coupling both data and format doesn’t really represent the real-world usage well.
- **NV\_command\_list** is fairly close to Vulkan in this regard as well. It provides a “StateObject” analog to Vulkan’s pipeline object. The emulation layer of the command list samples provides [some details](https://github.com/nvpro-samples/gl_commandlist_basic/blob/master/statesystem.hpp) what state is captured in a state object. This allows pre-validation of the state just like in Vulkan and then have much faster state transitions at render time.

```glsl
// setup time
glUseProgram(blah)
glEnable(GL_BLEND)
glStateCapture(GL_TRIANGLES, stateBlah);

glUseProgram(blubb)
glDisable(GL_BLEND)
glStateCapture(GL_TRIANGLES, stateBlubb);

// rendering time
glDrawCommandsStatesAddressNV(... {fbo0,fbo0,fbo1},{stateBlah,stateBlubb,stateBlah}, ... 3)
```

- **ARB\_separate\_shader\_objects** can be useful if there is a lot of shader permutations. Keep in mind this extension is only useful if you intend to mix different stages a lot. For example you have a set of N vertex shaders that are all used with varying M fragment shaders, giving a total of NxM active pairings. In a lot of applications the fragment shaders alone are causing the lion’s share of the active pairings. When this is the case separate shader objects will not be beneficial, it may even be lower performing as cross-stage optimizations cannot be done by the compiler.
- **ARB\_shader\_subroutine** has the capability to greatly speed up shader switches between a certain “family” of shaders. Vulkan’s SPIR-V introduces the concept of “specialization constants”, we can create sub-versions of the same shader by setting some hard-coded values at run-time. This can be achieved (although not as cleanly) with shader subroutines as well.

```glsl
// assumes mostly common bindings
// especially shader input/outputs should be same for
// all permutations of a "family"

void genericMain(int state){
  ...
}

// auto generate this code based on required permutations...
subroutine void fn_entry();
subroutine(fn_entry) void specializedMain_0(){
  genericMain(0);
}
subroutine(fn_entry) void specializedMain_1(){
  genericMain(1);
}
subroutine uniform fn_entry used_entry;
void main(){
  used_entry();
}
// toggle via glUniformSubroutines​(GL_FRAGMENT_SHADER, 1, {main0/main1});

// be aware that the array usage as below is not advised,
// as the worst-case register usage of the array
// becomes active for the program
uniform int used_idx;
subroutine uniform fn_entry entries[2]; // filled with both functions
void main(){
  entries[used_idx](); 
  // likely running with more allocated registers than needed
}
```

- **Pre-validating a program under render conditions** will cause the driver to generate the state-dependent version of the program. This can help to avoid later stuttering. One can achieve this by recreating the rendering state with important metrics (framebuffer configuration, multisampling state, vertex enable state…) and issuing a dummy draw-call that doesn’t actually trigger work, but triggers validation. By using the KHR_debug_output extension the NVIDIA driver will also trigger performance warnings when a program had to be recompiled because a different state was active when the program is used. While the process is not as clean as Vulkan’s pre-validation of pipeline objects, it can be an improvement over the general situation.

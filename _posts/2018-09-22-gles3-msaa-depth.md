---
layout: post
title: "opengl es 3.0 MSAA depth resolve"
description: "opengl es 3.0 MSAA depth resolve"
image: ""
date: 2018-09-22 10:00:04
categories: 研究
tags: [研究]
---

<!-- more -->
* Table of Contents
{:toc}

# 从一个显存泄漏说起

&nbsp; &nbsp; &nbsp; &nbsp;我以前有一篇文章，已经写了一篇UE4 MSAA depth resolve相关的[blog](http://aicdg.com/ue4-msaa-depth/)。如果说当时只是一个小bug的话，那这次的这个就是重大bug，显存泄漏，相信大家也是闻所未闻的bug吧。

&nbsp; &nbsp; &nbsp; &nbsp;UE4在Android上，如果反复切换MSAA的倍率，就会导致显存急剧升高，直到最后崩溃。而且说实话，显存高也就罢了，Android平台内存显存是共享内存的，显存占用过高不仅会影响应用程序所能使用内存的量，而且，还会占用地址空间。实际情况中很多崩溃，不是因为MSAA显存泄漏导致内存不足崩溃，而是32位内存地址空间不足了，new无法进行导致的崩溃。

&nbsp; &nbsp; &nbsp; &nbsp;这个bug虽然是Epic留的坑，但是其实本身影响并不大。因为不修改引擎代码的话，移动平台是无法运行时修改 `r.MobileMSAA` 的。在开启MSAA的状态下，会为depthbuffer(和stencilbuffer，如果有)分配一个 `renderbuffer`。不过这个 `renderbuffer` 只创建了，没有删除。如果运行时修改 `r.MobileMSAA` ，每次修改都会产生新的 `renderbuffer` ，就产生了内存泄漏。这个内存泄漏其实不太难处理，只要每次创建新的时候删掉旧的就行了。

> 在 OpenGLRenderTarget.cpp 中，在 glGenRenderbuffers 前调用 glDeleteRenderbuffers

# 败犬renderbuffer

&nbsp; &nbsp; &nbsp; &nbsp;这个bug解决了到还好，但是 `renderbuffer` 引起了我的兴趣，没见过这货啊。这货是不是 `framebuffer` 的马甲啊。

&nbsp; &nbsp; &nbsp; &nbsp;重新复习了一遍 fbo 之后，我可以负责任的说，`renderbuffer` 和 `framebuffer` 完全不是一个东西。在显存统计中，fbo被算作 `texture`。`renderbuffer` 和 `texture` 完全不是一个东西。

&nbsp; &nbsp; &nbsp; &nbsp;高通提供了kgsl工具用来查看各进程缩使用显存的具体情况。

```
# cat /d/kgsl/proc/<proc id>/mem  
```

&nbsp; &nbsp; &nbsp; &nbsp;查阅Khronos的文档，可以知道，texture是一个可读可写的buffer。而renderbuffer是只读不写的。WTF！！！不能读取有个毛用啊。可以说，renderbuffer是个不折不扣的败犬啊。

# renderbuffer的唯一作用

&nbsp; &nbsp; &nbsp; &nbsp;如果不使用MSAA，那么安卓平台将会完全没有 `renderbuffer` 创建。所以，可以说，`renderbuffer` 是和MSAA相关的。

&nbsp; &nbsp; &nbsp; &nbsp;查看khronos的WIKI，豁然开朗。

**Renderbuffer Objects** are OpenGL Objects that contain images. They are created and used specifically with Framebuffer Objects. They are optimized for use as render targets, while Textures may not be, and are the logical choice when you do not need to sample (i.e. in a post-pass shader) from the produced image. If you need to resample (such as when reading depth back in a second shader pass), use Textures instead. Renderbuffer objects also natively accommodate Multisampling (MSAA).

# renderbuffer的使用

&nbsp; &nbsp; &nbsp; &nbsp;renderbuffer和其他的opeGL对象一样，有 `glGenRenderbuffers/glDeleteRenderbuffers` 用于创建和删除，使用之前需要 `glBindRenderbuffer(GL_RENDERBUFFER)` 绑定。

&nbsp; &nbsp; &nbsp; &nbsp;之后使用

```
void glRenderbufferStorage(GLenum target​, GLenum internalformat​, GLsizei width​, GLsizei height​);

// openGL 3+
void glRenderbufferStorageMultisample(GLenum target​, GLsizei samples​, GLenum internalformat​, GLsizei width​, GLsizei height​);

// openGL ES extension
void glRenderbufferStorageMultisampleEXT(GLenum target​, GLsizei samples​, GLenum internalformat​, GLsizei width​, GLsizei height​);

void glFramebufferRenderbuffer(	GLenum target, GLenum attachment, GLenum renderbuffertarget, GLuint renderbuffer);
```

> openGL ES API没有MSAA，gles上的MSAA都是依靠扩展。`glRenderbufferStorageMultisampleEXT` 就是openGL ES正确resolve MSAA depth的方法，并不需要framebuffer fetch或者shader local storgae扩展(当然效率要低一些)




---
layout: post
title: "vulkan在shader中打log"
description: "vulkan在shader中打log"
image: ""
date: 2021-01-10 18:31:20
categories: vulkan
tags: [vulkan]
---
<!-- more -->
* Table of Contents
{:toc}

# VK_VALIDATION_FEATURE_ENABLE_DEBUG_PRINTF_EXT

&nbsp; &nbsp; &nbsp; &nbsp;在较新的vulkan版本中，添加了对[debug_printf](https://github.com/KhronosGroup/Vulkan-ValidationLayers/blob/master/docs/debug_printf.md)的支持，也就是说，现在可以在shader中打log了。

# 开启vulkan debug_printf

&nbsp; &nbsp; &nbsp; &nbsp;debug_printf在vulkan中是**VALIDATION_FEATURE**，也就是说可以通过和其他验证层一样的方式来开启和关闭，非常方便。



## 最终效果

![debug_printf](http://aicdg.com/assets/img/blogimg/vulkan/debugprintf/R$CZOW05{{{ZW~TS2ZIHGYS.jpg)

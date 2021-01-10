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

> 注意，vulkan中开启debug_printf需要同时开启device扩展**VK_KHR_SHADER_NON_SEMANTIC_INFO_EXTENSION_NAME**

```c++
const std::vector<const char*> deviceExtensions = {
        ...
        VK_KHR_SHADER_NON_SEMANTIC_INFO_EXTENSION_NAME
		...
};
```

## (推荐)使用Vulkan Configurator

&nbsp; &nbsp; &nbsp; &nbsp;在最近的vulkan sdk中，新添加了名为[Vulkan Configurator](https://vulkan.lunarg.com/doc/sdk/1.2.162.0/windows/vkconfig.html)的工具，可以很方便的在UI界面下配置vulkan的validation layer

## 使用环境变量

&nbsp; &nbsp; &nbsp; &nbsp;和vulkan旧版本相兼容，debug_printf也可以通过配置环境变量开启

```c++
VK_LAYER_ENABLES=VK_VALIDATION_FEATURE_ENABLE_DEBUG_PRINTF_EXT
```

&nbsp; &nbsp; &nbsp; &nbsp;通过设置环境变量**VK_LAYER_ENABLES**到**VK_VALIDATION_FEATURE_ENABLE_DEBUG_PRINTF_EXT**来全局开启debug_printf功能

## 在代码中配置

&nbsp; &nbsp; &nbsp; &nbsp;vulkan在代码中配置debug_printf会稍微复杂一些，但是可以对每个程序实现更精细的控制

```c++
const std::vector<VkValidationFeatureEnableEXT> enabledValidationLayers = {
        VK_VALIDATION_FEATURE_ENABLE_DEBUG_PRINTF_EXT
};

...

// debugPrintf
VkValidationFeaturesEXT validationFeatures{};
validationFeatures.sType = VK_STRUCTURE_TYPE_VALIDATION_FEATURES_EXT;
validationFeatures.pNext = nullptr;
validationFeatures.pEnabledValidationFeatures = enabledValidationLayers.data();
validationFeatures.enabledValidationFeatureCount = enabledValidationLayers.size();
validationFeatures.pDisabledValidationFeatures = nullptr;
validationFeatures.disabledValidationFeatureCount = 0;

...

instanceCreateInfo.pNext = &validationFeatures;
```

# shader中打log





&nbsp; &nbsp; &nbsp; &nbsp; GLSL中不要忘记`#extension GL_EXT_debug_printf : enable`,HLSL dxc编译器如果开启优化编译器会crash，现在如果要在hlsl打log，请先关闭优化

```bash
dxc -spirv -T vs_6_0 -E vert hlsl/shader.hlsl -Fo vert.spv -O0
```

## 最终效果

![debug_printf](http://aicdg.com/assets/img/blogimg/vulkan/debugprintf/R$CZOW05{{{ZW~TS2ZIHGYS.jpg)

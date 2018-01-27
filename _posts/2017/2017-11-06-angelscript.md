---
layout: post
title: "angelscript，极具潜力的脚本语言"
description: "angelscript，极具潜力的脚本语言"
category: c++
tags: [c++]
---

&nbsp; &nbsp; &nbsp; &nbsp;[angelscript](http://www.angelcode.com/angelscript/)说他极具潜力可能确实不太好，因为这货往上能追溯到PS2时代，是久经考验，证明有生产力的语言。和Squirrel一样，在游戏开发领域虽然小众单并不十分冷门。我是接触urho3D的时候才接触到的angelscript，并不算早，但并不妨碍我对angelscript表达有种的赞赏和钦佩。对于C++开发，这货的体验比lua强多了。

<!-- more -->

&nbsp; &nbsp; &nbsp; &nbsp;angelscript有很多特性。比如
支持接口，继承，多态，操作符重载，多线程，垃圾回收，枚举，typedef，函数指针，模块插件，单步执行调试，动态编译加载……作为脚本语言，这货的特性太像编译型语言了，而且还真的支持二进制编译，既能保护代码，又能提高运行效率。

&nbsp; &nbsp; &nbsp; &nbsp;在native方法到脚本语言注册的时候，更是令人倍感亲切。C++中导出方法给lua用的时候，想必另不少人深恶痛绝吧，虽然lua的FFI也不少，但是内置一个给你应该还是令人非常感激。anglescript绑定C++的方法只需要一行，甚至能注册C++模板，非常厉害。

&nbsp; &nbsp; &nbsp; &nbsp;anglescript是一种强类型语言，这个在脚本语言里也比较罕见。


AngelScript |	C++ |	Size (bits)
------------|----|------------
void |	void	|0
int8	|signed char|	8
int16	|signed short	|16
int	|signed int (*)	|32
int64	|signed int64_t|	64
uint8	|unsigned char|	8
uint16|	unsigned short|	16
uint	|unsigned int (*)	|32
uint64|	unsigned uint64_t|	64
float	|float|	32
double|	double|	64
bool	|bool|	8 (**)

> (*)int型的长度其实和平台相关，但是32位是大多数情况


> (**)在32为PowerPC上bool的长度为8

&nbsp; &nbsp; &nbsp; &nbsp;不过个人认为anglescript的类型系统，有两个遗憾：

1. 不支持String
2. 不支持Dynamic Array

&nbsp; &nbsp; &nbsp; &nbsp;这两点原作者倒是都有解释，对于可以从宿主语言导出数据类型的语言，String和Array都可以由宿主语言来提供。而且作者也在文档中提出了[String](http://www.angelcode.com/angelscript/sdk/docs/manual/doc_strings.html)还有[Dynamic Array](http://www.angelcode.com/angelscript/sdk/docs/manual/doc_arrays.html)绑定方面的一些建议，并且还是以[addon的形式提供了array](http://www.angelcode.com/angelscript/sdk/docs/manual/doc_addon_array.html)

&nbsp; &nbsp; &nbsp; &nbsp;说了anglescript不太爽的一点，就来说个特别爽的一点吧。这货的Object handles。AngelScript object handles的实质就是对象的指针，但是又有自动管理内存的能力，是智能指针。(不过话说回来，有内存自动管理能力不是脚本语言的标配吗，除了个别“小而美”的语言)

&nbsp; &nbsp; &nbsp; &nbsp;anglescript有类C的开发风格，和C++协同开发有着出色的体验，作为嵌入式脚本语言回避了JavaScript，Python和lua的很多问题，是一个出色的备胎。很遗憾他没有流行开来，这货在游戏这种热更新需求较大的行当本应该大有可为的，不过anglescript依然是一种出色的脚本语言，而且是罕见的强类型脚本语言。
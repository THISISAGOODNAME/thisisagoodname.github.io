---
layout: post
title: "emscripten meshlabJS移植步骤"
description: "emscripten meshlabJS移植步骤"
category: html5
tags: [html5,c++,emscripten]
---
* Table of Contents
{:toc}

&#160; &#160; &#160; &#160;MeshLab， 是一个开源，方便携带，和可扩展的系统，用于处理和非结构化编辑 3D 三角形网格。该系统发布于2005年年底。旨在帮助在 3D 扫描、 编辑、 清洗、 愈合、 检查、 呈现和转换这种网格提供一套工具所产生的典型不让小非结构化模型的处理。

&#160; &#160; &#160; &#160;这样优秀的网格处理软件，如果能移植到web上，必然会给建模人员带来极大方便。

<!-- more -->

# 环境搭建

&#160; &#160; &#160; &#160;首先，当然需要搭建emscripten开发环境，不在赘述。

&#160; &#160; &#160; &#160;然后，就是配置meshlab源码了。meshlabJS编译的时候，需要[VCG library](http://vcg.isti.cnr.it/vcglib/)。VCG library是一个开源的图形操作，处理和现实库。

&#160; &#160; &#160; &#160;新建一个文件夹，并在这个文件夹下面执行下面的命令

{% highlight bash %}
git clone https://github.com/cnr-isti-vclab/meshlabjs.git
svn checkout svn://svn.code.sf.net/p/vcg/code/trunk/vcglib vcglib
{% endhighlight %}

&#160; &#160; &#160; &#160;之后，该文件夹下的文件路径大概是这个样子。

{% highlight bash %}
├── meshlabjs
│   ├── LICENSE
│   ├── css
│   ├── doc
│   ├── img
│   ├── js
│   ├── mesh
│   ├── sources
│   │   ├── MakefileJS
│   │   └── build.bat
│   └── test
└── vcglib
    ├── apps
    ├── docs
    ├── eigenlib
    ├── img
    ├── vcg
    └── wrap
{% endhighlight %}

> 没有把所有文件列出

# 编译

&#160; &#160; &#160; &#160;编译过程很简单

## windows

- 进入`MeshLabJS/sources/` 并运行 `build.bat`

## linux&MacOS

- 在命令行下进入 `MeshLabJS/sources/`
- 执行下面的命令

{% highlight bash %}
make -f MakefileJS clean
make -f MakefileJS
make -f MakefileJS install
{% endhighlight %}

&#160; &#160; &#160; &#160;之后，`meshlabJS/meshlabjs/js`文件夹中应该会出现_MeshLabCppCore.js_和_MeshLabCppCore.js.mem_两个文件。

# 运行

&#160; &#160; &#160; &#160;运行方法很简单，在`meshlabJS`的根目录下启动server即可，以python为例。

运行下面的命令：

{% highlight bash %}
python -m SimpleHTTPServer
{% endhighlight %}

然后在浏览器中访问`http://localhost:8000/`,可以看到和[这个网页](http://www.meshlabjs.net/)类似的网页

# 技术关键:Javascript & C++ Interaction

&#160; &#160; &#160; &#160;所有本来在C++中执行的网格处理程序，都被emscripten编译成了asm.js，在C++函数中，指定一个emscripten绑定用于指定导出的函数是有必要的。

{% highlight cpp %}
#ifdef __EMSCRIPTEN__
//Binding code
EMSCRIPTEN_BINDINGS(MLCreatePlugin) {
    emscripten::function("Function1Name", &Function1Pointer);
    ...
    emscripten::function("FunctionNName", &FunctionNPointer);
}
#endif
{% endhighlight %}

&#160; &#160; &#160; &#160;之后，这个_Module_对象(一个全局javascript对象，他的各个属性是绑定的函数)可以在需要使用这个绑定函数时用下面的方法调用

{% highlight js %}
Module.Function1Name(parameter_1, ..., parameter_n);
{% endhighlight %}


# 总结

&#160; &#160; &#160; &#160;通过meshlabJS的移植过程，可以看出，移植过程非常顺畅(想想，unreal engine只用了4天就移植到了web上)，说明emscripten应该还是非常过硬的，关键点还是在于**makefile**的编写上。

&#160; &#160; &#160; &#160;接下来，我也许会分析一下[sqlite](http://www.sqlite.org/)([sql.js](https://github.com/kripken/sql.js)),[Bullet physics engine](http://bulletphysics.org/wordpress/)([ammo.js](https://github.com/kripken/ammo.js))和[Box2D](http://box2d.org/)([box2d.js](https://github.com/kripken/box2d.js))的移植过程
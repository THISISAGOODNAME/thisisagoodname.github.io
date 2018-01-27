---
layout: post
title: "openTK开发环境搭建"
description: "openTK开发环境搭建"
category: CG
tags: [CG,图形学,OpenGL]
---

&nbsp; &nbsp; &nbsp; &nbsp;上周我们讨论了一个问题：关于Unity支持平台的问题。我当时提出了一个观点，就是只要能运行[MonoGame](http://www.monogame.net/)，就能运行Unity引擎。而MonoGame图形底层就是[openTK](https://opentk.github.io/)(openTK又基于[SDL2](https://www.libsdl.org/))。openGL的C# binding其实不少，甚至[FNA](https://github.com/FNA-XNA/FNA)还提供了一个[单文件版](https://github.com/FNA-XNA/FNA/blob/master/src/FNAPlatform/OpenGLDevice_GL.cs)。不过个人还是觉得openTK开发体验比较好。

<!-- more -->

&nbsp; &nbsp; &nbsp; &nbsp;顺便来句题外话，MonoGame和FNA两个项目最初的目的也都是提供一个XNA的开源跨平台实现。两者也都不约而同的使用了SDL2作为底层。欧美独立游戏开发者使用XNA的比例其实很高，比如最近大火的《星露谷物语》，就是作者耗时4年完成。从NX消息披露，到NS发布会上公布独立游戏名单，也不过数月，星露谷就在NS可以运行，也足见XNA移植MonoGame可能真的非常简单。

* Table of Contents
{:toc}

# 1. 新建工程

&nbsp; &nbsp; &nbsp; &nbsp;打开rider，在C#的IDE方面，个人更偏向于rider而非蓝星最强IDE VS，不光因为我是JB粉，更因为Mac版的VS没那么好用(其实还是MonoDevelop)。

&nbsp; &nbsp; &nbsp; &nbsp;新建一个ConsoleApplication

![新建一个ConsoleApplication](http://7xqrar.com1.z0.glb.clouddn.com/opentkQQ20171029-222721@2x.png)

&nbsp; &nbsp; &nbsp; &nbsp;然后写个Hello world测试一下C#环境是不是OK。

![Hello World c#](http://7xqrar.com1.z0.glb.clouddn.com/opentkQQ20171029-222805@2x.png)

# 2. 为项目添加openTK依赖

&nbsp; &nbsp; &nbsp; &nbsp;要说C#项目添加外部依赖，最方便的方法自然是使用NuGet。无论Rider，Visual Studio还是Mono Develop都已经集成了NuGet的支持。

![NuGet 添加openTK](http://7xqrar.com1.z0.glb.clouddn.com/opentkQQ20171029-224748@2x.png)

&nbsp; &nbsp; &nbsp; &nbsp;用图中方法，为项目添加openTK，openTK.GLControl两个依赖。接下来就开始正戏了。

# 3. 创建窗口

&nbsp; &nbsp; &nbsp; &nbsp;新建一个名为MainWindow的类，内容改为如下：

```csharp
using OpenTK;
using OpenTK.Graphics.OpenGL4;
namespace OGLApplication1
{
    public sealed class MainWindow : GameWindow
    {
    }
}
```

&nbsp; &nbsp; &nbsp; &nbsp;修改Program.cs为如下内容：

```csharp
using System;
using System.Collections.Generic;

namespace OGLApplication1
{
    internal class Program
    {
        [STAThread]
        public static void Main(string[] args)
        {
            new MainWindow().Run(60);
        }
    }
}
```

&nbsp; &nbsp; &nbsp; &nbsp;点击运行，看见一个空窗口。那么恭喜成功。

# 4. 初始化OpenGL上下文

&nbsp; &nbsp; &nbsp; &nbsp;修改MainWindow的构造函数如下：

```csharp
 public MainWindow() : base(800, // initial width
            600, // initial height
            GraphicsMode.Default,
            "dreamstatecoding", // initial title
            GameWindowFlags.Default,
            DisplayDevice.Default,
            4, // OpenGL major version
            0, // OpenGL minor version
            GraphicsContextFlags.ForwardCompatible)
{
    Title += ": OpenGL Version: " + GL.GetString(StringName.Version);
}
```

&nbsp; &nbsp; &nbsp; &nbsp;点击运行，如果看到窗口标题为dreamstatecoding加上你的OpenGL版本(还有显卡驱动版本)，那么恭喜，这步也成功。

# 5. 一个完整示例

&nbsp; &nbsp; &nbsp; &nbsp;一个清屏&显示FPS&ESC退出的示例代码

```csharp
using System;
using OpenTK;
using OpenTK.Graphics;
using OpenTK.Graphics.OpenGL4;
using OpenTK.Input;

namespace OGLApplication1
{
    public sealed class MainWindow : GameWindow
    {
        public MainWindow() : base(800, // initial width
            600, // initial height
            GraphicsMode.Default,
            "dreamstatecoding", // initial title
            GameWindowFlags.Default,
            DisplayDevice.Default,
            4, // OpenGL major version
            0, // OpenGL minor version
            GraphicsContextFlags.ForwardCompatible)
        {
            Title += ": OpenGL Version: " + GL.GetString(StringName.Version);
        }
        
        protected override void OnResize(EventArgs e)
        {
            GL.Viewport(0, 0, Width, Height);
        }
    
        protected override void OnLoad(EventArgs e)
        {
            CursorVisible = true;
        }
        
        protected override void OnUpdateFrame(FrameEventArgs e)
        {
            HandleKeyboard();
        }
        
        private void HandleKeyboard()
        {
            var keyState = Keyboard.GetState();

            if (keyState.IsKeyDown(Key.Escape))
            {
                Exit();
            }
        }
    
        protected override void OnRenderFrame(FrameEventArgs e)
        {
            Title = $"(Vsync: {VSync}) FPS: {1f / e.Time:0}";

            GL.ClearColor(0.3f, 0.1f, 0.1f, 1.0f);
            GL.Clear(ClearBufferMask.ColorBufferBit | ClearBufferMask.DepthBufferBit);

            SwapBuffers();
        }
    }
}
```

&nbsp; &nbsp; &nbsp; &nbsp;点击运行，屏幕被清理为棕红色，窗口的标题显示帧数，以及垂直同步是否开启。运行效果如下：

![运行效果](http://7xqrar.com1.z0.glb.clouddn.com/opentkQQ20171029-230441@2x.png)



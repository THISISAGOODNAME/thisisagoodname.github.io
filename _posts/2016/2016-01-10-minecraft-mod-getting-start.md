---
layout: post
title: "minecraft mod 开发环境配置"
description: "minecraft mod 开发环境配置"
category: minecraft
tags: [minecraft,mod]
---

&#160; &#160; &#160; &#160;在正文开始之前，照例推荐一些私货

<p data-height="400" data-theme-id="0" data-slug-hash="yebqvd" data-default-tab="result" data-user="THISISAGOODNAME" class='codepen'>See the Pen <a href='http://codepen.io/THISISAGOODNAME/pen/yebqvd/'>SPHERE: REVISITED </a> by 攻伤菊菊长 (<a href='http://codepen.io/THISISAGOODNAME'>@THISISAGOODNAME</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

&#160; &#160; &#160; &#160;顺便吐槽一下[run.js](http://runjs.cn/)，曾经一直是run.js的忠实用户，但现在看run.js在嵌入展示的时候体验实在不佳。本人最近完成了run.js向codepin.io的项目迁移，[个人codepin主页](http://codepen.io/THISISAGOODNAME)

<!-- more -->

* Table of Contents
{:toc}

# 写作背景

&#160; &#160; &#160; &#160;写作背景请直接略过，没营养。

&#160; &#160; &#160; &#160;作为一个资深装逼实践家和知名喷子，我跟某minecraft mod群中某人杠上了，"You can you up"，好多时候觉得，未必I can't,but I just don‘t want up.

&#160; &#160; &#160; &#160;而且，我觉得minecraft开发不应该是个特别困难的事情，特别是有了forge API之后。感觉现在国内mod界(可能还有mmd界，mad界，手绘界，建模界等)，都是一群小白围观少数大神装逼。而真的要自己上的时候，反而一堆客观条件限制，比如时间不够，硬件跟不上；当然最多的还是没有基础，“没有基础”也是我个人最反感的理由，没有基础应该是学习的动力，因为“没有基础”成为了不去学习的借口，太可笑了。

# 安装minecraft forge开发环境

&#160; &#160; &#160; &#160;在安装forge API之前必须确保已经安装了[ Java SDK Standard Edition](http://www.oracle.com/technetwork/java/javase/downloads/index.html)，并配置好了环境变量。

## Step 1

&#160; &#160; &#160; &#160;下载forge API的源代码，[下载地址](http://files.minecraftforge.net/),在11.14.3.1502版本之前，叫src；而在11.14.3.1503版本以后，叫Mdk(估计是Mod develop kit)的缩写。

&#160; &#160; &#160; &#160;我自己的minecraft安装的forge版本是10.13.4.1448，所以下载10.13.4.1448的src。大家开发的时候就最新吧，反正新手们只有跌过几个跟头之后才能明白(也希望以后大家别吐槽我blender还在用2.63版本了，装的时候也是很新的)

&#160; &#160; &#160; &#160;解压下载的源文件，建议选择一个好找且干净的地方，下载之后应该是如下的文件结构

{% highlight bash %}
.
├── CREDITS-fml.txt
├── LICENSE-fml.txt
├── MinecraftForge-Credits.txt
├── MinecraftForge-License.txt
├── README.txt
├── build.gradle
├── eclipse
├── forge-1.7.10-10.13.4.1448-1.7.10-changelog.txt
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
└── src
    └── main
        ├── java
        │   └── com
        │       └── example
        │           └── examplemod
        │               └── ExampleMod.java
        └── resources
            └── mcmod.info
{% endhighlight %}

## Step 2

&#160; &#160; &#160; &#160;在命令行下，进入源代码文件所在的文件夹。然后执行下面的命令

- windows

{% highlight bash %}
gradlew setupDecompWorkspace --refresh-dependencies
{% endhighlight %}

- Linux/Mac OS

{% highlight bash %}
./gradlew setupDecompWorkspace --refresh-dependencies
{% endhighlight %}

&#160; &#160; &#160; &#160;如果在linux/Mac OS上禁止运行此命令，可以先执行`chmod +x gradlew`，然后重试一次。

&#160; &#160; &#160; &#160;此步骤极其浪费时间，该步作用如下：

- 从Maven仓库下载大部分依赖 (Minecraft, MCP(Mod Coder Pack), Java Libraries, Gradle, Forge, and FML)
- 设置 Forge, MCP 和 Gradle.

> 看到这里我要说一下了，默认的gradle脚本使用的是[maven中心仓库](http://repo1.maven.org/maven2)，国内极其慢，可以自行替换为国内的maven镜像源(修改build.gradle)。如果你不知道maven是啥，我也没办法

![编译过程截图](http://img17.poco.cn/mypoco/myphoto/20160110/10/17800049220160110104013074.png)

<center>编译过程截图</center>

![构建成功](http://img17.poco.cn/mypoco/myphoto/20160110/10/17800049220160110104116069.png)

<center>构建成功截图</center>

## Step 3

&#160; &#160; &#160; &#160;配置IDE，这次我不推荐intellij IDEA，而是推荐Eclipse。Forge 的大部分开发者都使用Eclipse，eclipse的配置已经完全傻瓜化，而IDEA配置非常复杂，因为中间涉及到了maven仓库的问题，IDEA的配置将更加费时费力。因此，这次我更推荐傻瓜化配置的Eclipse。

&#160; &#160; &#160; &#160;在forge 源代码文件夹下，运行

- windows

{% highlight bash %}
gradlew eclipse
{% endhighlight %}

- Linux/Mac OS

{% highlight bash %}
./gradlew eclipse
{% endhighlight %}

## Step 4

&#160; &#160; &#160; &#160;打开eclipse，在选择workspace的界面，将workspace目录选择为forge源代码下面的eclipse文件夹。

![eclipse配置成功界面](http://img17.poco.cn/mypoco/myphoto/20160110/10/17800049220160110104139088.png)

<center>eclipse界面</center>

> Eclipse默认提示比较差，因为eclipse默认设置只有敲击"."后才会进行提示，可以通过

1. 打开“ Window －－ Preferences”
2. 选择“Java－－Editor－－Content Assist”，在右侧的“Auto-Activation”找到“Auto Activation triggers for java”选项
3. 把"."修改为“.abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ(, ”

![Auto Activation triggers for java](http://img17.poco.cn/mypoco/myphoto/20160110/11/17800049220160110111356057.png)

<center>Auto Activation triggers for java位置</center>

&#160; &#160; &#160; &#160;之后eclipse提示能好一点，但和intellij IDEA的智能提示还是没有可比性。

# Hello forge demo

## 编写

&#160; &#160; &#160; &#160;新建一个`tutorial.generic`包，在包种新建一个`Generic`类，在其中添加如下代码

{% highlight java %}
package tutorial.generic;

import net.minecraftforge.fml.common.Mod;
import net.minecraftforge.fml.common.Mod.EventHandler;
import net.minecraftforge.fml.common.Mod.Instance;

import net.minecraftforge.fml.SidedProxy;

import net.minecraftforge.fml.common.event.FMLInitializationEvent;
import net.minecraftforge.fml.common.event.FMLPostInitializationEvent;
import net.minecraftforge.fml.common.event.FMLPreInitializationEvent;

@Mod(modid=Generic.MODID, name=Generic.MODNAME, version=Generic.MODVER) //Tell forge "Oh hey, there's a new mod here to load."
public class Generic //Start the class Declaration
{
    //Set the ID of the mod (Should be lower case).
    public static final String MODID = "generic";
    //Set the "Name" of the mod.
    public static final String MODNAME = "Generic Mod";
    //Set the version of the mod.
    public static final String MODVER = "0.0.0";

    @Instance(value = Generic.MODID) //Tell Forge what instance to use.
    public static Generic instance;
        
    @EventHandler
    public void preInit(FMLPreInitializationEvent event)
    {
        
    }
        
    @EventHandler
    public void load(FMLInitializationEvent event)
    {

    }
        
    @EventHandler
    public void postInit(FMLPostInitializationEvent event)
    {
    }
}
{% endhighlight %}

然后保存

## 发布mod

&#160; &#160; &#160; &#160;在forge源代码文件夹下执行

- windows

{% highlight bash %}
gradlew build
{% endhighlight %}

- Linux/Mac OS

{% highlight bash %}
./gradlew build
{% endhighlight %}

之后，在`build/libs`下生产的jar文件就是打包成的mod，可以拷贝给别人使用

## 测试mod

&#160; &#160; &#160; &#160;在forge源代码文件夹下执行

- windows

{% highlight bash %}
gradlew runClient
{% endhighlight %}

- Linux/Mac OS

{% highlight bash %}
./gradlew runClient
{% endhighlight %}

然后会打开minecraft，并加载了你刚刚编写的mod，如下图所示

![mod demo](http://img17.poco.cn/mypoco/myphoto/20160110/10/17800049220160110104203036.png)

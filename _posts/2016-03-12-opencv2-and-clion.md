---
layout: post
title: "mac os opencv2开发环境简易配置"
description: "mac os opencv2开发环境简易配置"
category: opencv
tags: [c++,opencv]
---

&#160; &#160; &#160; &#160;写作正文之前，依旧推荐一个大神的作品

<p data-height="268" data-theme-id="0" data-slug-hash="ONRLQN" data-default-tab="result" data-user="THISISAGOODNAME" data-preview="true" class="codepen">See the Pen <a href="http://codepen.io/THISISAGOODNAME/pen/ONRLQN/">3D Random Walkers</a> by 攻伤菊菊长 (<a href="http://codepen.io/THISISAGOODNAME">@THISISAGOODNAME</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

<!-- more -->

* Table of Contents
{:toc}

&#160; &#160; &#160; &#160;为了省事，使用homebrew安装openCV2，然后出于习惯，我使用Clion作为IDE(Jetbrains家族软件大爱)

# 替换homebrew源

&#160; &#160; &#160; &#160;其实homebrew貌似不被wallout，就是homebrew官方bootles比较慢，而且经常抽风逼你然后自动下源码编译，所以换源还是有助于提升生活水平的。

## 替换homebrew默认源

{% highlight bash %}
cd /usr/local
git remote set-url origin git://mirrors.ustc.edu.cn/homebrew.git
{% endhighlight %}

## 替换homebrew bottles默认源

{% highlight bash %}
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.bashrc
{% endhighlight %}

&#160; &#160; &#160; &#160;zsh用户向**.zshrc**中添加

# 使用homebrew安装openCV2

{% highlight bash %}
brew update
brew install homebrew/science/opencv
{% endhighlight %}

&#160; &#160; &#160; &#160;如果安装openCV3，则使用如下命令

{% highlight bash %}
brew update
brew install homebrew/science/opencv3
{% endhighlight %}

# 在Clion中的CmakeLists.txt中配置openCV

&#160; &#160; &#160; &#160;在Clion中新建一个项目，然后修改CmakeLists.txt如下

<pre><code>cmake_minimum_required(VERSION 3.3)
#//openCVlearn是我的工程名,可以替换为你自己的,Clion的话创建项目时应该已经添加好了
project(openCVlearn)

#//这里将会使用OpenCV 2,所以是OpenCV,如果使用openCV 3,则为OpenCV3
find_package( OpenCV )
#//添加OpenCV头文件目录
include_directories( ${OpenCV_INCLUDE_DIRS} )

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

set(SOURCE_FILES main.cpp)
add_executable(openCVlearn ${SOURCE_FILES})

#//openCVlearn是工程名,添加动态库
target_link_libraries( openCVlearn ${OpenCV_LIBS} )
</code></pre>

# 测试

&#160; &#160; &#160; &#160;修改main.cpp如下

{% highlight c++ %}
#include <opencv2/opencv.hpp>

using namespace cv;

int main() {
    Mat srcImage = imread("PATH/TO/AN/IMAGE");
    imshow("[原始图]", srcImage);
    waitKey(0);

    return 0;
}
{% endhighlight %}


&#160; &#160; &#160; &#160;imread中给一个图像的路径，建议是绝对路径，因为在不做修改的情况下，clion会在一个稀奇古怪的地方产生二进制文件。当然，可以在Clion的设置面板，Build,Execution,Deployment -> Cmake中，修改build output path的值的方法来指定生成二进制文件的路径。比如我希望在工程目录下的bin文件夹下生成二进制文件，那么直接将build output path的值填写为bin即可。

![配置Camke文件生成路径](http://7xqrar.com1.z0.glb.clouddn.com/QQ20160312-0@2x.png)

&#160; &#160; &#160; &#160;这样配置的话，就会在项目目录的bin文件夹下生成可执行文件了

![配置后bin文件路径](http://7xqrar.com1.z0.glb.clouddn.com/QQ20160312-1@2x.png)


---
layout: post
title: "[转载]一种语言, 适合任何时候使用 -- Haxe特性杂谈"
description: "[转载]一种语言, 适合任何时候使用 -- Haxe特性杂谈"
category: haxe
tags: [haxe]
---

&#160; &#160; &#160; &#160;转述正文之前，先夹杂一些私或，顺便推荐[codepen.io](http://codepen.io/)

<p data-height="500" data-theme-id="0" data-slug-hash="pgRVvR" data-default-tab="result" data-user="THISISAGOODNAME" class='codepen'>See the Pen <a href='http://codepen.io/THISISAGOODNAME/pen/pgRVvR/'>pgRVvR</a> by 攻伤菊菊长 (<a href='http://codepen.io/THISISAGOODNAME'>@THISISAGOODNAME</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

<!-- more -->

&#160; &#160; &#160; &#160;正文开始，[原文链接](http://www.jtianling.com/one-language-everywhere.html),作者的文章是针对haxe 2.x 版本写的，我可能会做出一定的修改

* Table of Contents
{:toc}

# 前言

&#160; &#160; &#160; &#160;这个世界有很多的语言, 不同的语言适合不同的情境, Ruby因为很适合开发领域语言(DSL), 所以被大卫选择用于开发Rails, JavaScript因为常用异步模型和方便的回调处理, 被Ryan Dah用于开发node, 而JAVA期望通过JVM一次编写, 随处运行. 各种语言因为不同的目标场景, 也使得语言本身的特性趋于目标场景, 决定了这些语言应该是什么样子, 比如C++因为效率的考虑和兼容C的需求, 决定了C++之所以成为现在的C++.

&#160; &#160; &#160; &#160;同时也有很多语言是为生成另外一种语言而生的, 最典型的就是[CoffeeScript](http://coffeescript.org/), 我知道我的语言要更加优美, 但是我也知道我无法彻底替代你, 这是这个世界中习惯和惯性带来的悲哀, 所以, 不能替代你, 那我首先通过生成你来取代你. 这就是Coffee这样语言的思路. 因为JavaScript是如此的不好用和难以简单替代, 所以成为了这类语言的竞技场.

&#160; &#160; &#160; &#160;但是, Haxe就更加夸张了, 他也是一种通过生成其他语言而生的语言, 只不过, 他可以生成多达7种不同的语言(现在是9种语言).

# Haxe简单介绍

> One language, everywhere.
>        
> -- Haxe's slogan

&#160; &#160; &#160; &#160;Haxe可能是我学习的最冷门的语言了, 不过因为Haxe的特性, 他再冷门, 我在实际中用到他的可能性都应该比Lisp要强的多. 我可以这么说, Haxe可能是世界上最有野心的语言, 并且它还真的部分做到了. 其实Haxe与我还真有些渊源, 因为我以前的leader出来创业, 最后选择的就是Haxe. 并且对它的适用性和成熟程度都很认可, 甚至还向我推荐过.

&#160; &#160; &#160; &#160;Haxe起源于flash社区, 号称'multiplatform language', 据说是一个喜欢OCaml但是又知道OCaml实在无法推广的法国人Nicolas Cannasse开发的, 可以生成的语言如下: ([具体的见Haxe的intro](http://haxe.org/documentation/introduction/))

1. JavaScript: 直接生成单个的.js文件, 支持DOM API.
2. Flash: 直接可以生成swf, 兼容Flash Players 6到11.
3. NekoVM: 可以直接编译成NekoVM的字节码. NekoVM本来就挺冷门的.
4. PHP: 很难想象哪个真的Phper会准备用Haxe替代掉自己的语言.
5. C++: 生成源代码并且包含Makefiles, 另外, 提到了一个名为NME的2D跨平台游戏库. 简单的看了一下, 从特性上看, 似乎是除了Cocos2D-X以外一个较为不错的选择, 而从一些介绍看, 这个库本似乎是一个模仿Flash的库, 因为使用了Haxe这个神奇的语言, 所以现在已经支持了从iOS到Android到Windows等众多平台了.
6. C#和JAVA: 2.1以后开始支持, 但是还在实验中, 官方称在3.0时才算成熟.

> NOTES，以下是官方文档中的编译参数使用

- `-js file_name` Generates [JavaScript](http://haxe.org/manual/target-javascript.html) source code in specified file.
- `-as3 directory` Generates <span style="color:red">ActionScript 3</span> source code in specified directory.
- `-swf file_name` Generates the specified file as [Flash](http://haxe.org/manual/target-flash.html) .swf.
- `-neko file_name` Generates <span style="color:red">Neko</span> binary as specified file.
- `-php directory` Generates [PHP](http://haxe.org/manual/target-php.html) source code in specified directory.
- `-cpp directory` Generates [C++](http://haxe.org/manual/target-cpp.html) source code in specified directory and compiles it using native C++ compiler.
- `-cs directory` Generates <span style="color:red">C#</span> source code in specified directory.
- `-java directory` Generates <span style="color:red">Java</span> source code in specified directory and compiles it using the Java Compiler.
- `-python file_name` Generates <span style="color:red">Python</span> source code in the specified file.

&#160; &#160; &#160; &#160;因为能生成这么多语言, 所以Haxe可以支持一些我们经常用到的框架和库, 比如NodeJS, 比如直接生成PHP代码结合Apache做后台, 比如生成C++代码以直接支持跨平台的游戏开发.

&#160; &#160; &#160; &#160;以上的事实已经足够让我震惊了, 更逆天的是Haxe竟然还有自己的着色器语言解决方法, [hxsl](http://old.haxe.org/manual/hxsl), 什么叫征服世界, 那就是不仅要征服CPU上运行的语言, 还要征服GPU上运行的语言~~~

&#160; &#160; &#160; &#160;总的来说, Haxe就是希望自己能干任何事情, 而按它的理念, 它还真的有可能有一天能做到, 语言的世界很残酷, 各种语言都有他最适合的情景, 没有一种语言能统一世界, 但是假如真有的话, Haxe一定是最可能的那一个. 事实上, 我说Haxe的野心大并不是贬义, 这是事实上Nicolas脑海中存在的概念, 他自己就做过一个讲座, 名字就叫[Plans for World Domination](https://docs.google.com/viewer?url=http%3A%2F%2Fncannasse.fr%2Ffile%2Fwwx2012.pdf)([youtube视频](https://www.youtube.com/watch?v=wOAIRzaD7Wc) -- 请自备梯子), Haxe就是他占领世界的计划~~

&#160; &#160; &#160; &#160;在官方的[Why user Haxe文档](http://haxe.org/doc/why)(原文链接已经移除，会重定向到[Use Cases](http://haxe.org/use-cases/))中提到了用Haxe的以下理由:

- 适用于客户端, 服务端和桌面程序开发的ECMA风格的程序语言.
- 极为快速的编译(这似乎不是理由, 因为不用Haxe都不用多出这道编译流程, 更何况, 我实际感受是其实也没有那么快, 特别是编译成C++的时候)
- 类型检测的好处(事实上喜欢动态语言的人就是因为喜欢没有类型检测带来的自由)
- 为你的目标平台增加需要的语言特性.
- 开源

&#160; &#160; &#160; &#160;其中提到的干货不多, 其实我觉得支持编译到多种语言, 所以支持多种平台, 使得只需要学习一种语言就能适用于几乎任何环境, 这就是最大并且足够强大的理由了.

> NOTES：官方使用场景说明中，haXe在下面六个都能得心应手

- [**Games**](http://haxe.org/use-cases/games/) Haxe is popular with game creators because it is fast, has many useful libraries, and can target iOS, Android, Web and Desktop easily.
- [**Web**](http://haxe.org/use-cases/web/) Haxe gives you a powerful, type-safe language that can target JavaScript on the client and PHP, NodeJS or Neko on the server. Share code and APIs between the client and server seamlessly.
- [**Mobile**](http://haxe.org/use-cases/mobile/) Share code between key platforms. Access native functionality without sacrificing performance.
- [**Desktop**](http://haxe.org/use-cases/desktop/) Build cross platform desktop apps using WX Widgets, Node Webkit, Java Swing or custom UI libraries.
- [**Command Line**](http://haxe.org/use-cases/cli/) Take advantage of easy-to-use libraries to write powerful, cross platform CLI applications.
- [**Cross-Platform APIs**](http://haxe.org/use-cases/cross-platform-apis/) Write cross platform APIs in Haxe that can be exported and shared with other languages and environments.

&#160; &#160; &#160; &#160;在[Who Uses Haxe
](http://haxe.org/use-cases/who-uses-haxe.html)中可以看到haXe 3 的实际使用情况，BBC、丰田、可口可乐、迪士尼等知名企业都使用了haXe，而且[openFl](http://www.openfl.org/)的出现，让haXe不再是玩具语言，而是真的具有生产力的工具，知名引擎[stencyl](http://stencyl.com/)的runtime就是基于openFl的

# 环境

&#160; &#160; &#160; &#160;Haxe 2.1(本文写作时，已经更新到3.2.1版本), 我会通过生成C++代码和JavaScript代码来观察结果. 其中JavaScript我用的不是HTML, 而是我可能稍微熟悉一点的Node, 所以还用了Hx-node(已经被[nodejs](http://lib.haxe.org/p/nodejs)库取代)这个haxe对nodejs的支持库.

&#160; &#160; &#160; &#160;另外, 有Haxe的REPL环境[ihx](https://github.com/ianxm/ihx), 和官方提供的在网页上尝试Haxe的[http://try.haxe.org/](http://try.haxe.org/)

# 生成C++初体验

&#160; &#160; &#160; &#160;从简单的HelloWorld开始吧:

{% highlight haxe %}
// HelloWorld.hx:

class HelloWorld {
    public static function main() {
        trace("Hello World!");
    }
}
{% endhighlight %}

&#160; &#160; &#160; &#160;从例子中能够看到, 原来所谓的ECMA风格就是类似C#和JAVA的风格...

&#160; &#160; &#160; &#160;我用来编译成cpp的.hxml配置是cpp.hxml:

<pre><code>
-main HelloWorld
-cpp cpp
-debug
-D HXCPP_M64
</code></pre>

通过`haxe cpp.hxml`命令给我的生成结果是在cpp目录下的一堆文件, 其中cpp/src目录下看到有100多行C++代码, 我甚至都不想现在就全都弄懂. 不过还好的是给我的执行文件的确输出了HelloWorld.

# 生成JavaScript With nodejs

&#160; &#160; &#160; &#160;首先需要通过haxelib install nodejs命令来安装haxe的nodejs库. 然后通过以下配置(我命名为nodejs.hxml)编译上例中的HelloWorld.hx生成想要的.js文件.

<pre><code>
-main HelloWorld
-js helloworld.js
-debug
-lib nodejs
</code></pre>

&#160; &#160; &#160; &#160;通过`haxe nodejs.hxml`命令会生成两个文件, 一个是helloworld.js, 一个是helloworld.js.map(用于debug), 此时通过node helloworld.js能看到输出结果正常. 当然, 首先你得把node安装好.

# Haxe入门

## 变量和类型

支持以下类型:

- Integer(int): 整数类型, 不支持更进一步详细的整数类型,比如short, long啥的, 类似动态语言的做法.
- Float: 浮点数, 也是一个浮点走天下.
- Boolean(Bool): 布尔, 代表的是true和false.
- String: 字符串, 不解释.
- Void: 用于表示没有类型和值.
- Dynamic: 类似C#中的dynamic类型, 意图在静态语言中加入动态的效果, 个人很欣赏的尝试.
- Unknown: 类似javascript中的undefined.
另外, 还有null关键字, 类型是Unknown<0>.

&#160; &#160; &#160; &#160;定义一个变量的语法和[UnityScript](http://www.jtianling.com/articles/179.html)(Unity中的JavaScript)类似(或者说JScript), 我甚至怀疑javascript2将来就会是这个样子(现在来看应该是不会了，不过[TypeScript](http://www.typescriptlang.org/)倒是这个样子).

	var i : Int = 5;

不过还是有个区别, 那就是当同样使用简化的语法时:

	var i = 5;

&#160; &#160; &#160; &#160;Haxe使用的是类似C++和C#的类型推导技术, 也就是说类型会在编译器决定, 不影响执行效率, 而UnityScript则不一样, 会影响执行效率. 就这点来说, Haxe是足够先进的.

同时, 作为现代语言, String的连接操作符是没啥问题的.

{% highlight haxe %}
class HelloWorld {
    public static function main() {
        var hello = "Hello,";
        var world = "World!";
        trace(hello + world);
    }
}
{% endhighlight %}

&#160; &#160; &#160; &#160;自从发现javascript和C#的扭曲特性后, 我总是喜欢用1 + "1"的操作来实验一个语言, 很不幸的是, Haxe就是这样中招的语言, 会无错误的输出11.

## 函数

&#160; &#160; &#160; &#160;用的还是类似UnityScript的方式, 使用`function`关键字, 同时用`str:String`这样的方式来表示参数类型, 通过把`:Int`加到函数定义语句的结尾来表示返回值, 熟悉UnityScript的XD是无障碍了.

{% highlight haxe %}
class HelloWorld {
    static function add(n1 : Int, n2 : Int) : Int {
        return n1 + n2;
    }

    public static function main() {
        trace( add(3,4) );
    }
}
{% endhighlight %}

## 操作符

&#160; &#160; &#160; &#160;没有看到操作符重载的内容, 但是支持常见操作符, 包括>>>这个javascript特有的(修正，其实无符号位移JAVA也有)和C系的++, --等.

## 流程控制

&#160; &#160; &#160; &#160;基本上该有的都有, while, do-while, for-in, switch. 不过也没有啥亮点. 有些诡异的是去掉了常规的for, 即`for(int i=0; i<length; ++i)`形式的for语句. 不知道是为什么. 倒是避免了javascript中for块变量作用域的问题(控制下标的循环遍历被认为是落伍的，比如go和python都不能这么循环控制).

&#160; &#160; &#160; &#160;稍微值得一提的是switch的自动break, case支持常量字符串和用逗号分隔的多路匹配, 现在的语言一般都是从C那儿来的, C特别别扭的switch也就成为了各大语言改进的对象, 有意思的是, 改进的效果都不一样, Haxe就是这样改的:

{% highlight haxe %}
class HelloWorld {
    public static function main() {

        var choice : String = "";
        while (true) {
            choice = Sys.stdin().readLine();

            switch( choice ) {
                case "y", "Y":
                    Sys.println("You surely want it!");
                case "n", "N":
                    Sys.println("you don't want it?!");
                    break;
                default:
                    Sys.println("Not a valid choice.");
            }
        }
    }
}

{% endhighlight %}

&#160; &#160; &#160; &#160;我没有在`case "y"`后面使用`break`, 但是不会执行到下面的case, 而事实上, 后面的那个`break`是用来从`while`死循环中退出的.

&#160; &#160; &#160; &#160;上面的代码没法在js下运行, 因为读了命令行, 我们得使用nodejs中读命令行的process模块, 上面的代码经过处理就成了下面这样了:

{% highlight haxe %}
import js.Node;

class HelloWorld {
    public static function main() {

        Node.process.stdin.resume();
        Node.process.stdin.setEncoding('utf8');
        Node.process.stdin.on('data', function (chunk) {
            var choice : String = chunk.trim();
            switch( choice ) {
                case "y", "Y":
                    Node.console.log("You surely want it!");
                case "n", "N":
                    Node.console.log("you don't want it?!");
                default:
                    Node.console.log("Not a valid choice.");
            }
        });
    }
}
{% endhighlight %}

&#160; &#160; &#160; &#160;首先, 因为没有了while死循环, 所以可能得自己加个条件作为退出(这里没有), 其次, 为了使用js的模块, 可能得稍微费点周折.

1. 在预先设为Haxe lib的地址`git clone https://github.com/cloudshift/hx-node.git nodejs`
2. 修改nodejs.hxml, 添加一个-cp YOUR_PATH. 比如我的地址是~haxe_libs, 于是就是-cp ~/haxe_libs/nodejs.

> NOTES: 用haxelib安装haxe lib的话只要运行`haxelib install nodejs`
 
## 作用域

&#160; &#160; &#160; &#160;类作用域和函数作用域不用说了, 好消息是Haxe有块作用域, 也就是说, {}范围内定义的变量只在大括号范围内有效, 这个很有很有用. 假如不能这样的话, 需要用javascript扭曲的匿名函数hack, 太扭曲了.


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

##变量和类型

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

# Haxe特性

&#160; &#160; &#160; &#160;因为Haxe是如此的冷门, 所以明明本文是讲Haxe特性的, 结果也拉入了一堆简介. 好了， 以下是正题. 我选择的特性列表直接来自于官方列出的列表[Language Features](http://old.haxe.org/doc/features#language-features), 没有什么比这个更有代表性了.

## 基于模版(class-based)的类和接口模型(类似JAVA)

> Classic Object-Oriented [class + interface model](http://haxe.org/manual/types-class-instance.html) (similar to Java)

1. 类的访问限制只有public和private两层, 相对一些语言(比如JAVA)还在C++经典的三权限中去增加一个默认的package不同, Haxe甚至去掉了通常的private, 而把protected称为private, 并且默认状态就是private.
2. 用new()表示构造函数.
3. 用super来访问父类
4. interface可以implement一个interface.
5. 覆盖父类的函数时要求必须使用override关键字标志.(和C#一样)
6. 允许在一个文件中定义多个类. 可以用private限定一个类只在当前文件中可用.

&#160; &#160; &#160; &#160;下例我尽量多用了更多特性, 不过一个例子还是有些难, 看个风格吧, 反正和JAVA, C#很像.

{% highlight haxe %}
interface PointProto {
    function length() : Float;
}

class Point {
    var x_ : Float;
    var y_ : Float;

    public function new(x : Float, y : Float) {
        this.x_ = x;
        this.y_ = y;
    }

    public function x() : Float {
        return x_;
    }

    public function y() : Float {
        return y_;
    }
}

class Point3D extends Point, implements PointProto{
    var z_ : Float;

    public function new(x : Float, y : Float, z : Float) {
        super(x, y);
        this.z_ = z;
    }

    public function z() : Float {
        return z_;
    }

    public function length() : Float {
        return Math.sqrt(x_ * x_ + y_ * y_ + z_ * z_);
    }
}

class HelloWorld {
    public static function main() {
        var p = new Point3D(1.0, 2.0, 3.0);
        trace(p.length());
    }
}
{% endhighlight %}

## 静态类型但是有对动态类型的支持

>Strict typing but with [Dynamic]([http://haxe.org/ref/dynamic](http://haxe.org/ref/dynamic) support

&#160; &#160; &#160; &#160;这点很像C#, 我觉得也就就应该这样, 为啥非要需要区分动态语言和静态语言?

这个世界上的语言有些很有意思的现象:

1. 对于动态语言来说, 没有静态类型可以选择, 但是对于静态语言来说, 就算有动态语言的选择时, 一定是推荐你尽量使用静态类型, 因为效率和编译期的静态检查. 没有人会推荐你把一个同时有动态类型和静态类型的语言当作动态类型的来用, 比如C#, UnityScript, 还有Haxe.
2. 对于语句必须以;结尾或者必须用{}的情况, 你没得选择. 对于语句完全不需要;和{}时, 也没得选择. 但是对于语句可以以;结尾也可以不以;结尾, 语句可以用{}也可以不用时, 一定是推荐你用;和{}, 因为更加严谨, 不容易出现错误. 比如javascript的;和类C语言的单行if语句.

上面的例子可能还有一些, 不过总得来说, 开发社区还是有保守的倾向和对运行效率的追求.

&#160; &#160; &#160; &#160;Dynamic除了可以把Haxe当作动态语言来用, 还有个好处就是不用模版来实现模版的功能.

{% highlight haxe %}
class HelloWorld {

    public static function add(left_param : Dynamic, right_param : Dynamic) {
        return left_param + right_param;
    }

    public static function main() {
        trace( add( 1, 2.0) );
    }
}
{% endhighlight %}

&#160; &#160; &#160; &#160;事实上, 上例要是用模版的话, add函数还需要是双模版(即T1, T2两个类型) 才能实现整数对浮点数的操作. 当然, 上例太弱智了, 但是真实情况下对多种类型进行一样的操作是完全可能的, 特别是你准备用duck typing的时候.

&#160; &#160; &#160; &#160;有趣的是, Haxe在动态化上走的非常远, 一般情况下, 即使是Dynamic的对象, 也不能访问一个不存在的成员变量和成员函数, 只是把检测推迟到运行时了, 但是Haxe允许动态为Dynamic类型的对象添加成员变量, 甚至函数, 也就是说, 就像javascript里面的Object那样, 对于一个静态类型的语言加入这个特性大大的超乎我的想象. 不过回头想想, Haxe起源于flash社区的Action Script(javascript的方言), 也就没有那么奇怪了. 只是因为Haxe的静态特性又太深入, 使得我有些都忘了它的起源, 这点和UnityScript很不一样.

{% highlight haxe %}
class HelloWorld {

    public static function main() {
        var obj : Dynamic = {};
        obj.name = "Simon";
        obj.hello = function() {
            trace("hello," + obj.name);
        }

        obj.hello();
    }
}
{% endhighlight %}

## 参数化动态类型

> Parameterized Dynamic Variables

&#160; &#160; &#160; &#160;在Dynamic如此自由后, 作为静态类型的语言, Haxe加入了参数化动态类型这个新的特性, 作用如下:

{% highlight haxe %}
class HelloWorld {

    public static function main() {
        var dyn : Dynamic<String> = cast {};
        dyn.name = "Paul";
        dyn.age = "28";   // can't be dyn.age = 28;

        trace( dyn.name + ":" + dyn.age );
    }
}
{% endhighlight %}

&#160; &#160; &#160; &#160;而使用了参数化的动态类型后, (比如dyn变量被设定为Dynamic<String>)一个变量能在参数化的范围内, 随意的访问成员变量, 但是, 仍然是有限制的, 比如上例中的dyn, 就不能将其成员变量赋值成字符串意外的类型.

&#160; &#160; &#160; &#160;更觉的是, 这个参数化类型甚至能被继承... 具体的就还是看[官方的介绍](http://haxe.org/manual/types-dynamic.html#implementing-dynamic)吧, 因为这样的特性已经超出我的想象了, 所以不知道什么时候能用上.

## 包和模块的直接支持

> Packages and [modules](http://haxe.org/manual/type-system-modules-and-paths.html)

&#160; &#160; &#160; &#160;默认情况下一个.hx文件就是一个module, 此时是以文件名为模块名的, 比如下例, 我讲上面例子中的add函数拆分到一个新的文件中去:

{% highlight haxe %}
// file: utility.hx
class Utility {

    public static function add(left_param : Dynamic, right_param : Dynamic) {
        return left_param + right_param;
    }
}

// file: helloworld.hx
import Utility;

class HelloWorld {

    public static function main() {
        trace( Utility.add( 1, 2.0) );
    }
}
{% endhighlight %}

值得注意的有以下几点:

1. 一个模块中可以有多个类.
2. 默认时每个类都是public的, 外部都可以访问, 设定为private时, 该类为模块内的internal内, 只能在模块内访问.
3. 模块名只与文件名有关, 和文件里面的类没有关系. 同时, 就算文件名为全小写, import的时候仍然要求首字符大写, 并且可以正常import.
4. 导入后, 实际已经将导入模块的所有类型都加入到了全局空间, 不需要加模块名即可访问, 假如有冲突的话, 仍然可以通过模块名的限定来取得指定模块的类型.

另外, 有可选的关键字package.

## 泛型

> Generics ([type parameters](http://haxe.org/manual/type-system-type-parameters.html)) with one or several constraints, but not [variance](http://haxe.org/manual/type-system-variance.html)

&#160; &#160; &#160; &#160;在静态类型上走远了, 总会需要泛型的, 不然一些基础的容器会很累人, Haxe也又泛型, 基本的语法如下:

{% highlight haxe %}
class Array<T> {

    function new() {
        // ...
    }

    function get( pos : Int ) : T {
        // ...
    }

    function set( pos : Int, val : T ) : Void {
        // ...
    }
}
{% endhighlight %}

&#160; &#160; &#160; &#160;单纯函数的泛型官网上没有说明, 可能实际上就没有, 因为正如上面动态类型的例子中所演示的, 用Dynamic就能模拟出泛型的效果, 还能通过参数限定.

## 泛型参数限制

> [Constraint Parameters](http://haxe.org/manual/type-system-type-parameters.html#constraint-parameters)

&#160; &#160; &#160; &#160;一般意义的泛型往往太过灵活, 所以Haxe加入了泛型参数限制的特性. 很类似C#的约束类型. 概念上类似, 就不写例子了, 只是语法上有差异:

{% highlight haxe %}
class EvtQueue<T : (Event, EventDispatcher)> {
    var evt : T;
    // ...
}
{% endhighlight %}

## 对所有变量适用的先进类型推导, 包括函数参数, 返回值(除了类成员变量)

> Advanced [Type Inference](http://haxe.org/manual/type-system-type-inference.html) for all variables including methods arguments and return types (except member variables)

&#160; &#160; &#160; &#160;这个在上面最开始讲类型的地方就介绍过了, 不再重复.

&#160; &#160; &#160; &#160;只在这里介绍一个辅助的标记`$type`, 在编译期这个标记会移除, 但是具有这个标记的表达式会保留, 同时通过Warning信息输出推导出来的类型, 作为方便调试的一个手段. 比如下面的代码:

	var x : Int = $type(0);

## 具有结构化继承的匿名结构

> Anonymous [Structures](http://haxe.org/manual/types-anonymous-structure.html) with structural subtyping

&#160; &#160; &#160; &#160;这是个很让人惊叹的特性, 同样的也是因为Haxe其实源于Action Script. Haxe允许以以下的格式来定义新的对象, 这在Haxe被称作匿名结构(Anonymous Structures).

{% highlight %}
var point = { x : 1, y : -5};

var user = {
    name : "Nicolas",
    age : 32,
    pos : [ {x:0, y:0}, {x:0, y:0}],
};
{% endhighlight %}

&#160; &#160; &#160; &#160;了解javascript的人, 一眼就能看出, 这就是javascript Object的一种定义方式. 到这还没有什么, 作为静态语言, Haxe还真的给上面这种对象定义了类型, 比如, 上面的point类似就是`{ x : Int, y : Int }`, 而user的类型是`{ name : String, age : Int, pos : Array<{ x : Int, y : Int }> }`, 这是个很神奇的事情. 可以通过刚提到的$type来验证.

&#160; &#160; &#160; &#160;既然是类型, 你甚至就可以直接使用, 只是要是多次使用会稍微麻烦一点, 所以Haxe提供了C/C++里面的typedef来简化这种操作. 见下面的代码:

{% highlight %}
typedef Point = { x : Int, y : Int }

class Path {
        var start : Point;
        var target : Point;
        var current : Point;
}
{% endhighlight %}

&#160; &#160; &#160; &#160;这就是Haxe中定义一个对象简便的办法, 假如需要的是一个临时使用的对象, 的确不需要兴师动众的动用class来做了. 特别的, 虽然官方的例子中没有演示, 我上面已经演示了, Haxe的这种结构实际上也是支持函数成员的, 使得它的应用可以更加广泛.

&#160; &#160; &#160; &#160;具体还有些细节, 这里就不一一列举了.

&#160; &#160; &#160; &#160;需要稍微注意一点的是在Haxe中结构是按javascript那么动态实现的, 所以运行效率会低于静态实现的class类型. 但是带来一个好处, 就是Haxe官方所谓的*Structural Subtyping*, 其实又没有subtyping的语法, 只是当一个对象拥有另一个对象的所有成员时, 可以完全当作另一个对象使用, 这个有些类似Duck Typing.

&#160; &#160; &#160; &#160;更进一步的是, 可以不指定函数参数的类型(似乎默认就是Dynamic), 然后自动适配structural.

{% highlight haxe %}
public static function getLength(pt) {
    return pt.x + pt.y;
}
{% endhighlight %}

&#160; &#160; &#160; &#160;此时就完全可以接受上面typedef的Point类型对象. 更加神奇的是, 假如你真的传入了一个不对的对象, Haxe能在编译期就发现错误... 这简直逆天了. 在具有极为动态特性的时候, 还能有强大的编译期类型检测, 你还能说什么...

## 严格静态的函数类型, 闭包和偏函数

> Strictly typed function types, functions closures and partial applications

{% highlight haxe %}
class HelloWorld {

    public static function makeIncrementor(base : Int) {
        var count = base;
        return function(num : Int) {
            count += num;
            return count;
        }
    }

    public static function main() {

        var obj1 = makeIncrementor(10);
        var obj2 = makeIncrementor(20);

        trace(obj1(1));
        trace(obj1(1));

        trace(obj2(2));
        trace(obj2(2));
    }
}
{% endhighlight %}

&#160; &#160; &#160; &#160;偏函数是一个很有用的特性, 在很早的时候C++为了支持这个特性鼓捣出了bind1st, bind2nd等恶心的函数, 在C++11中由bind统一了, 主要的作用就是可以让一个函数在经过少量的适配代码后就能应用到需要函数作为参数, 但是参数个数对不上的地方.

&#160; &#160; &#160; &#160;其实本质上就算没有, 我们也可以通过新建一个函数, 然后调用原有函数实现, 只是有了原生的支持后会变得简单很多.

{% highlight haxe %}
function add( x: Float, y: Float ) {
    return x + y;
}

var addOne = callback( add, 1 );

addOne( 5 ); // returns 6
{% endhighlight %}

callback就是实现偏函数的函数, 有个遗憾是, 暂时没有(2.09版本)发现通过常见的通过占位符实现任意参数的bind, 目前只能依照参数从左至右的实现partical function. 在3.x版本中, callback将被废弃, 引入了通过占位符_实现的任意参数bind.

届时, 以上的代码将会是类似`var addOne = add(1, _)`;

> NOTES:3.x版本已经推出了

## 有限制的多态函数

> Polymorphic Methods (per-method type parameters), with constraints

&#160; &#160; &#160; &#160;恕我愚钝, 官方也没有直接提供例子, 光看文本, 我不知道说的是什么, 函数重载? 函数调用时的多态性?

## 函数的可选参数和默认参数

> [Optional](http://haxe.org/manual/types-function-optional-arguments.html) and constant default value function arguments

&#160; &#160; &#160; &#160;概念上很简单, 就不举例子了, 需要稍微注意一下的是Haxe奇怪的可选参数语法, 是在参数的最前面加?.

## 指定的内联函数和常量的内联

>Explicit [Inline](http://haxe.org/manual/class-field-inline.html) methods and constant inlined variables

&#160; &#160; &#160; &#160;这是纯粹的运行效率考虑了, 用函数的inline来减少函数的调用开销, 通过常量替换实现减少中间变量. 这两个特性可能只有需要编译的语言才能实现了. 用的也是inline关键字, 用法和C++基本一样, 限制也是一样的, 那就是要求在编译器的确能够决定调用的函数内容, 才能形成内联, 比如动态绑定的情况, 就没法实现.

## 可以使用this的匿名函数

> Local function declarations with this capturing

&#160; &#160; &#160; &#160;Haxe中可以用类似javascript的通过变量来定义函数, 毕竟起源于AS啊, 比如:

	var fun = function() { };
	
上面的形式可以在一个局部定义, 此时函数像普通变量一样, 只在作用域内有效, 在外部无法访问.

至于with this capturing的意思, 大概是指在这种情况下, 匿名函数还能调用this吧, 比如下面:

{% highlight haxe %}
class Point {
    var x_ : Float;
    var y_ : Float;
    public function new(x : Float, y : Float) {
        x_ = x;
        y_ = y;

        var fun = function() {
            // this capturing
            this.x_ = 10.0;
            this.y_ = 20.0;
        };

        fun();
    }

    public function x() {
        return x_;
    }

    public function y() {
        return y_;
    }

}

class HelloWorld {

    public static function main() {
        var pt = new Point(20.0, 10.0);

        trace("x:" + pt.x());
        trace("y:" + pt.y());
    }
}
{% endhighlight %}

要是没有this捕获, 那匿名函数的作用会大打折扣.

## 自动闭包创建

> Automatic closure creation

&#160; &#160; &#160; &#160;就是闭包被, 难道哪个语言的闭包还不是自动创建的? 不明白. 指的是Objective-C中Blocks创建的时候需要手动指定哪些局部变量需要进行捕获才能更改吗?

## 强大的枚举类型, 有构造函数参数和模型匹配

> Powerful [Enums](http://haxe.org/manual/types-enum-instance.html) (with constructor parameters and pattern matching)

&#160; &#160; &#160; &#160;不知道是何种原因, Nicolas对枚举似乎有强大的兴趣和偏好, 现代语言中的确对原来C/C++的枚举很反感, 所以一般都会让枚举变得强类型一些, 但是Haxe中就不仅仅是让枚举变的强类型而已, 而是极大的复杂化了枚举(或者说极大的强大?) 比如, 在Haxe中枚举支持构造函数参数:

{% highlight haxe %}
enum Color {
        Red;
        Green;
        Blue;
        Grey( v : Int );
        Rgb( r : Int, g : Int, b : Int );
        Alpha( a : Int, col : Color );
}
{% endhighlight %}

&#160; &#160; &#160; &#160;上面例子中的Grey, Rgb, Alpha就是带构造函数参数的枚举, 甚至在Alpha中递归引用了Color这个枚举类型. 问题是, 我没有发现这个到底有什么用, 因为枚举都是常量, 你可以如下定义Alpha:

	Alpha( 127, Red );
	Alpha( 255, Rgb(0,0,0) );

&#160; &#160; &#160; &#160;定义后就不能更改, 然后呢? 我不知道然后怎么样了. 官方的文档中似乎是想用switch中, 在case语句中使用枚举的构造函数参数, 一则这种用处非常之小, 二则真的要用到, 定义一个枚举和一个class, 差异也不大了.

## 没有声明, 只有表达式

> No statements : only expressions

&#160; &#160; &#160; &#160;没有具体的文档, 意思是说没有C++/Objective-C中那样所谓的接口和实现分离吗? 那样的做法的确很不人道, 我在C#的文章中就提到过, 看来碰到知音了. 不过稍微现代一点的语言, 比如JAVA, C#都早就是这样了, 说明持这样看法的人不是一个两个了.

## 异常(try/catch)

> Exceptions (try/catch)

&#160; &#160; &#160; &#160;只有try catch没有finally的异常机制根本就不值得作为优点提出来(就像C++中的那样), 那是不完善的设计. 有意思的是在Haxe用catch Dynamic类型的异常作为捕获所有的异常, 因为内部没有为异常建立一个完整的树壮继承对象体系?
